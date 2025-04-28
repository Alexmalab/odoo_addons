# Odoo Module: l10n_es_edi_facturae

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: xml_utils.py

```python
import base64
import hashlib
from base64 import b64encode
from copy import deepcopy

from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding
from lxml import etree

from odoo.exceptions import UserError

NS_MAP = {'ds': "http://www.w3.org/2000/09/xmldsig#"}


def _canonicalize_node(node):
    """
    Returns the canonical (C14N 1.0, without comments, non exclusive) representation of node.
    Speficied in: https://www.w3.org/TR/2001/REC-xml-c14n-20010315
    Required for computing digests and signatures.
    Returns an UTF-8 encoded bytes string.
    """

    return etree.tostring(node, method="c14n", with_comments=False, exclusive=False)


def _get_uri(uri, reference, base_uri=""):
    """
    Returns the content within `reference` that is identified by `uri`.
    Canonicalization is used to convert node reference to an octet stream.
    - URIs starting with # are same-document references
    https://www.w3.org/TR/xmldsig-core/#sec-URI
    - Empty URIs point to the whole document tree, without the signature
    https://www.w3.org/TR/xmldsig-core/#sec-EnvelopedSignature
    Returns an UTF-8 encoded bytes string.
    """
    node = deepcopy(reference.getroottree().getroot())
    if uri == base_uri:
        # Base URI: whole document, without signature (default is empty URI)
        for signature in node.xpath('ds:Signature', namespaces=NS_MAP):
            if signature.tail:
                # move the tail to the previous node or to the parent
                if (previous := signature.getprevious()) is not None:
                    previous.tail = "".join([previous.tail or "", signature.tail or ""])
                else:
                    signature.getparent().text = "".join([signature.getparent().text or "", signature.tail or ""])
            node.remove(signature)
        return _canonicalize_node(node)

    if uri.startswith("#"):
        path = "//*[@*[local-name() = '{}' ]=$uri]"
        results = node.xpath(path.format("Id"), uri=uri.lstrip("#"))  # case-sensitive 'Id'
        if len(results) == 1:
            return _canonicalize_node(results[0])
        if len(results) > 1:
            raise UserError(f"Ambiguous reference URI {uri} resolved to {len(results)} nodes")

    raise UserError(f'URI {uri} not found')


def _reference_digests(node, base_uri=""):
    """
    Processes the references from node and computes their digest values as specified in
    https://www.w3.org/TR/xmldsig-core/#sec-DigestMethod
    https://www.w3.org/TR/xmldsig-core/#sec-DigestValue
    """
    for reference in node.findall("ds:Reference", namespaces=NS_MAP):
        ref_node = _get_uri(reference.get("URI", ""), reference, base_uri=base_uri)
        lib = hashlib.new("sha256", ref_node)
        reference.find("ds:DigestValue", namespaces=NS_MAP).text = b64encode(lib.digest())


def _fill_signature(node, private_key):
    """
    Uses private_key to sign the SignedInfo sub-node of `node`, as specified in:
    https://www.w3.org/TR/xmldsig-core/#sec-SignatureValue
    https://www.w3.org/TR/xmldsig-core/#sec-SignedInfo
    """
    signed_info_xml = node.find("ds:SignedInfo", namespaces=NS_MAP)

    # During signature generation, the digest is computed over the canonical form of the document
    signature = private_key.sign(_canonicalize_node(signed_info_xml), padding.PKCS1v15(), hashes.SHA256())
    node.find("ds:SignatureValue", namespaces=NS_MAP).text = base64.encodebytes(signature)


def _int_to_bytes(number):
    """ Converts an integer to a byte string (in smallest big-endian form). """
    return number.to_bytes((number.bit_length() + 7) // 8, byteorder='big')

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import csv

from odoo.tools import file_open
from . import models
from . import wizard


def _l10n_es_edi_facturae_post_init_hook(env):
    """
    We need to replace the existing spanish taxes following the template so the new fields are set properly
    """
    for company in env['res.company'].search([('chart_template', '=like', 'es_%'), ('parent_id', '=', False)]):
        Template = env['account.chart.template'].with_company(company)
        Template._load_data({
            'account.tax': Template._get_es_facturae_account_tax(),
        })

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Spain - Facturae EDI',
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'website': 'https://www.facturae.gob.es/face/Paginas/FACE.aspx',
    'description': """
This module create the Facturae file required to send the invoices information to the General State Administrations.
It allows the export and signature of the signing of Facturae files.
The current version of Facturae supported is the 3.2.2

for more informations, see https://www.facturae.gob.es/face/Paginas/FACE.aspx
    """,
    'depends': [
        'l10n_es',
    ],
    'data': [
        'data/uom.uom.csv',
        'data/facturae_templates.xml',
        'data/signature_templates.xml',

        'security/ir.model.access.csv',
        'security/l10n_es_edi_certificate.xml',

        'views/l10n_es_edi_facturae_views.xml',
        'views/res_company_views.xml',
        'views/account_tax_views.xml',
        'views/uom_uom_views.xml',
        'views/account_menuitem.xml',

        'wizard/account_move_send_views.xml',
        'wizard/account_move_reversal_view.xml',
    ],
    'demo': [
        'demo/l10n_es_edi_facturae_demo.xml',
    ],
    'post_init_hook': '_l10n_es_edi_facturae_post_init_hook',
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\facturae_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Sub-template used for every instance of AddressType -->
        <template id="address_type">
            <t t-if="partner_country_code == 'ESP'">
                <AddressInSpain>
                    <Address t-out="', '.join(val for val in (partner.street, partner.street2) if val)[:80]"/>
                    <PostCode t-out="str(partner.zip)"/>
                    <Town t-out="(partner.city or '')[:50]"/>
                    <Province t-out="partner.state_id.name[:20] if partner.state_id else partner.country_id.name[:20]"/>
                    <CountryCode t-out="partner_country_code"/>
                </AddressInSpain>
            </t>
            <t t-else="">
                <OverseasAddress>
                    <Address t-out="', '.join(val for val in (partner.street, partner.street2) if val)[:80]"/>
                    <PostCodeAndTown t-out="' '.join(var for var in (str(partner.zip), partner.city) if var)[:50]"/>
                    <Province t-out="partner.state_id.name[:20] if partner.state_id else partner.country_id.name[:20]"/>
                    <CountryCode t-out="partner_country_code"/>
                </OverseasAddress>
            </t>
        </template>

        <!-- Sub-template used for every instance of ContactDetails -->
        <template id="contact_details_type">
            <ContactDetails>
                <Telephone t-out="(partner_phone or '')[:15]"/>
                <WebAddress t-out="(partner.website or '')[:60]"/>
                <ElectronicMail t-out="(partner.email or '')[:60]"/>
            </ContactDetails>
        </template>

        <!-- Sub-template used for every instance of LegalEntityType -->
        <template id="legal_entity_type">
            <LegalEntity>
                <CorporateName t-out="partner.name[:80]"/>
                <TradeName t-out="partner.display_name[:40]"/>
                <t t-call="l10n_es_edi_facturae.address_type"/>
                <t t-call="l10n_es_edi_facturae.contact_details_type"/>
            </LegalEntity>
        </template>

        <!-- Sub-template used for every instance of IndividualType -->
        <template id="individual_type">
            <Individual>
                <Name t-out="(partner_name.get('firstname') or '')[:40]"/>
                <FirstSurname t-out="(partner_name.get('surname') or '')[:40]"/>
                <SecondSurname t-out="(partner_name.get('surname2') or '')[:40]"/>
                <t t-call="l10n_es_edi_facturae.address_type"/>
                <t t-call="l10n_es_edi_facturae.contact_details_type"/>
            </Individual>
        </template>

        <!-- Sub-template used for every instance of TaxIdentificationType -->
        <template id="tax_identification_type">
            <TaxIdentification>
                <PersonTypeCode t-out="'J' if partner.is_company else 'F'"/>
                <ResidenceTypeCode t-out="partner.l10n_es_edi_facturae_residence_type"/>
                <TaxIdentificationNumber t-out="partner.vat"/>
            </TaxIdentification>
        </template>

        <!-- Sub-template used for every instance of BusinessType -->
        <template id="business_type">
            <t t-call="l10n_es_edi_facturae.tax_identification_type"/>
            <t t-if="partner.is_company">
                <t t-call="l10n_es_edi_facturae.legal_entity_type"/>
            </t>
            <t t-else="">
                <t t-call="l10n_es_edi_facturae.individual_type"/>
            </t>
        </template>

        <!-- Sub-template used for every instance of TaxType -->
        <template id="tax_type">
            <Tax>
                <TaxTypeCode t-out="tax['tax_record'].l10n_es_edi_facturae_tax_type"/>
                <TaxRate t-out="tax['TaxRate']"/>
                <TaxableBase>
                    <TotalAmount t-out="float_repr(refund_multiplier*tax['TaxableBase']['TotalAmount'], file_currency.decimal_places)"/>
                    <EquivalentInEuros t-if="need_conversion" t-out="float_repr(refund_multiplier*tax['TaxableBase']['EquivalentInEuros'], eur.decimal_places)"/>
                </TaxableBase>
                <TaxAmount t-if="tax.get('TaxAmount')">
                    <TotalAmount t-out="float_repr(refund_multiplier*tax['TaxAmount']['TotalAmount'], file_currency.decimal_places)"/>
                    <EquivalentInEuros t-if="need_conversion" t-out="float_repr(refund_multiplier*tax['TaxAmount']['EquivalentInEuros'], eur.decimal_places)"/>
                </TaxAmount>
                <SpecialTaxableBase t-if="tax.get('SpecialTaxableBase')">
                    <TotalAmount t-out="float_repr(refund_multiplier*tax['SpecialTaxableBase']['TotalAmount'], file_currency.decimal_places)"/>
                    <EquivalentInEuros t-if="need_conversion" t-out="float_repr(refund_multiplier*tax['SpecialTaxableBase']['EquivalentInEuros'], eur.decimal_places)"/>
                </SpecialTaxableBase>
                <SpecialTaxAmount t-if="tax.get('SpecialTaxAmount')">
                    <TotalAmount t-out="float_repr(refund_multiplier*tax['SpecialTaxAmount']['TotalAmount'], file_currency.decimal_places)"/>
                    <EquivalentInEuros t-if="need_conversion" t-out="float_repr(refund_multiplier*tax['SpecialTaxAmount']['EquivalentInEuros'], eur.decimal_places)"/>
                </SpecialTaxAmount>
                <EquivalenceSurcharge t-out="tax.get('EquivalenceSurcharge')"/>
                <EquivalenceSurchargeAmount t-if="tax.get('EquivalenceSurchargeAmount')">
                    <TotalAmount t-out="float_repr(refund_multiplier*tax['EquivalenceSurchargeAmount']['TotalAmount'], file_currency.decimal_places)"/>
                    <EquivalentInEuros t-if="need_conversion" t-out="float_repr(refund_multiplier*tax['EquivalenceSurchargeAmount']['EquivalentInEuros'], eur.decimal_places)"/>
                </EquivalenceSurchargeAmount>
            </Tax>
        </template>

        <!-- Sub-template used for every instance of InvoiceLineType -->
        <template id="invoice_line_type">
            <InvoiceLine>
                <ReceiverTransactionReference t-out="line.get('ReceiverTransactionReference')"/>
                <FileReference t-out="line.get('FileReference')"/>
                <ReceiverContractReference t-out="line.get('ReceiverContractReference')"/>
                <FileDate t-out="line.get('FileDate')"/>
                <SequenceNumber t-out="line.get('SequenceNumber')"/>
                <ItemDescription t-out="line['ItemDescription']"/>
                <Quantity t-out="line['Quantity']"/>
                <UnitOfMeasure t-out="line.get('UnitOfMeasure')"/>
                <UnitPriceWithoutTax t-out="float_repr(refund_multiplier*line['UnitPriceWithoutTax'], file_currency.decimal_places)"/>
                <TotalCost t-out="float_repr(refund_multiplier*line['TotalCost'], file_currency.decimal_places)"/>
                <DiscountsAndRebates t-if="line.get('DiscountsAndRebates')">
                    <t t-foreach="line['DiscountsAndRebates']" t-as="discount">
                        <Discount>
                            <DiscountReason t-out="discount['DiscountReason']"/>
                            <DiscountRate t-out="discount.get('DiscountRate')"/>
                            <DiscountAmount t-out="float_repr(refund_multiplier*discount['DiscountAmount'], file_currency.decimal_places)"/>
                        </Discount>
                    </t>
                </DiscountsAndRebates>
                <Charges t-if="line.get('Charges')">
                    <t t-foreach="line['Charges']" t-as="charge">
                        <Charge>
                            <ChargeReason t-out="charge['ChargeReason']"/>
                            <ChargeRate t-out="charge.get('ChargeRate')"/>
                            <ChargeAmount t-out="float_repr(refund_multiplier*charge['ChargeAmount'], file_currency.decimal_places)"/>
                        </Charge>
                    </t>
                </Charges>
                <GrossAmount t-out="float_repr(refund_multiplier*line['GrossAmount'], file_currency.decimal_places)"/>
                <TaxesWithheld t-if="line.get('TaxesWithheld')">
                    <t t-foreach="line['TaxesWithheld']" t-as="tax"><t t-call="l10n_es_edi_facturae.tax_type"/></t>
                </TaxesWithheld>
                <TaxesOutputs>
                    <t t-foreach="line['TaxesOutputs']" t-as="tax"><t t-call="l10n_es_edi_facturae.tax_type"/></t>
                </TaxesOutputs>
                <LineItemPeriod t-if="line.get('LineItemPeriod')">
                    <StartDate t-out="line['LineItemPeriod']['StartDate']"/>
                    <EndDate t-out="line['LineItemPeriod']['EndDate']"/>
                </LineItemPeriod>
                <TransactionDate t-out="line.get('TransactionDate')"/>
            </InvoiceLine>
        </template>

        <template id="corrective_type">
            <Corrective>
                <InvoiceNumber t-out="invoice['Corrective']['refunded_invoice_record'].name"/>
                <ReasonCode t-out="invoice['Corrective']['ReasonCode']"/>
                <ReasonDescription t-out="invoice['Corrective']['Reason']"/>
                <TaxPeriod>
                    <StartDate t-out="invoice['Corrective']['TaxPeriod']['StartDate']"/>
                    <EndDate t-out="invoice['Corrective']['TaxPeriod']['EndDate']"/>
                </TaxPeriod>
                <CorrectionMethod>01</CorrectionMethod>
                <CorrectionMethodDescription>Rectificación modelo íntegro.</CorrectionMethodDescription>
            </Corrective>
        </template>

        <!-- Sub-template used for every instance of InvoiceType -->
        <template id="invoice_type">
            <Invoice>
                <InvoiceHeader>
                    <InvoiceNumber t-out="invoice['invoice_record'].name"/>
                    <InvoiceDocumentType t-out="invoice['InvoiceDocumentType']"/>
                    <InvoiceClass t-out="invoice['InvoiceClass']"/>
                    <t t-if="refund_multiplier == -1" t-call="l10n_es_edi_facturae.corrective_type"/>
                </InvoiceHeader>
                <InvoiceIssueData>
                    <IssueDate t-out="invoice['invoice_record'].invoice_date.isoformat()"/>
                    <OperationDate t-out="invoice['InvoiceIssueData'].get('OperationDate')"/>
                    <InvoicingPeriod t-if="invoice['InvoiceIssueData'].get('InvoicingPeriod')">
                        <StartDate t-out="invoice['InvoiceIssueData']['InvoicingPeriod']['StartDate']"/>
                        <EndDate t-out="invoice['InvoiceIssueData']['InvoicingPeriod']['EndDate']"/>
                    </InvoicingPeriod>
                    <InvoiceCurrencyCode t-out="invoice['invoice_currency'].name"/>
                    <ExchangeRateDetails t-if="invoice['InvoiceIssueData'].get('ExchangeRateDetails')">
                        <ExchangeRate t-out="invoice['InvoiceIssueData']['ExchangeRate']"/>
                        <ExchangeRateDate t-out="(invoice['invoice_record'].invoice_date or invoice['invoice_record'].date).isoformat()"/>
                    </ExchangeRateDetails>
                    <TaxCurrencyCode t-out="invoice['invoice_currency'].name"/>
                    <LanguageName t-out="invoice['InvoiceIssueData']['LanguageName']"/>
                    <ReceiverTransactionReference t-out="invoice['InvoiceIssueData']['ReceiverTransactionReference']"/>
                    <FileReference t-out="invoice['InvoiceIssueData']['FileReference']"/>
                    <ReceiverContractReference t-out="invoice['InvoiceIssueData']['ReceiverContractReference']"/>
                </InvoiceIssueData>
                <TaxesOutputs>
                    <t t-foreach="invoice['TaxOutputs']" t-as="tax"><t t-call="l10n_es_edi_facturae.tax_type"/></t>
                </TaxesOutputs>
                <TaxesWithheld t-if="invoice.get('TaxesWithheld')">
                    <t t-foreach="invoice['TaxesWithheld']" t-as="tax"><t t-call="l10n_es_edi_facturae.tax_type"/></t>
                </TaxesWithheld>
                <InvoiceTotals>
                    <TotalGrossAmount t-out="float_repr(refund_multiplier*invoice['TotalGrossAmount'], file_currency.decimal_places)"/>
                    <GeneralDiscounts t-if="invoice.get('GeneralDiscounts')">
                        <t t-foreach="invoice['GeneralDiscounts']" t-as="discount">
                            <Discount>
                                <DiscountReason t-out="discount['DiscountReason']"/>
                                <DiscountRate t-out="discount.get('DiscountRate')"/>
                                <DiscountAmount t-out="float_repr(refund_multiplier*discount['DiscountAmount'], file_currency.decimal_places)"/>
                            </Discount>
                        </t>
                    </GeneralDiscounts>
                    <GeneralSurcharges t-if="invoice.get('GeneralSurcharges')">
                        <t t-foreach="invoice['GeneralSurcharges']" t-as="charge">
                            <Charge>
                                <ChargeReason t-out="charge['ChargeReason']"/>
                                <ChargeRate t-out="charge.get('ChargeRate')"/>
                                <ChargeAmount t-out="float_repr(refund_multiplier*charge['ChargeAmount'], file_currency.decimal_places)"/>
                            </Charge>
                        </t>
                    </GeneralSurcharges>
                    <TotalGeneralDiscounts t-if="invoice.get('TotalGeneralDiscounts')" t-out="float_repr(refund_multiplier*invoice['TotalGeneralDiscounts'], file_currency.decimal_places)"/>
                    <TotalGeneralSurcharges t-if="invoice.get('TotalGeneralSurcharges')" t-out="float_repr(refund_multiplier*invoice['TotalGeneralSurcharges'], file_currency.decimal_places)"/>
                    <TotalGrossAmountBeforeTaxes t-out="float_repr(refund_multiplier*invoice['TotalGrossAmountBeforeTaxes'], file_currency.decimal_places)"/>
                    <TotalTaxOutputs t-out="float_repr(refund_multiplier*invoice['TotalTaxOutputs'], file_currency.decimal_places)"/>
                    <TotalTaxesWithheld t-out="float_repr(refund_multiplier*invoice['TotalTaxesWithheld'], file_currency.decimal_places)"/>
                    <InvoiceTotal t-out="float_repr(refund_multiplier*invoice['InvoiceTotal'], file_currency.decimal_places)"/>
                    <PaymentsOnAccount t-if="invoice.get('PaymentsOnAccount')">
                        <t t-foreach="invoice['PaymentsOnAccount']" t-as="payment">
                            <PaymentOnAccount>
                                <PaymentOnAccountDate t-out="payment['PaymentOnAccountDate']"/>
                                <PaymentOnAccountAmount t-out="float_repr(refund_multiplier*payment['PaymentOnAccountAmount'], file_currency.decimal_places)"/>
                            </PaymentOnAccount>
                        </t>
                    </PaymentsOnAccount>
                    <TotalFinancialExpenses t-if="invoice.get('ReimbursableExpensesAmount')" t-out="float_repr(refund_multiplier*invoice['ReimbursableExpensesAmount'], file_currency.decimal_places)"/>
                    <TotalOutstandingAmount t-out="float_repr(refund_multiplier*invoice['TotalOutstandingAmount'], file_currency.decimal_places)"/>
                    <TotalPaymentsOnAccount t-if="invoice.get('TotalPaymentsOnAccount')" t-out="float_repr(refund_multiplier*invoice['TotalPaymentsOnAccount'], file_currency.decimal_places)"/>
                    <AmountsWithheld t-if="invoice.get('AmountsWithheld')">
                        <WithholdingReason t-out="invoice['AmountsWithheld']['WithholdingReason']"/>
                        <WithholdingRate t-out="invoice['AmountsWithheld'].get('WithholdingRate')"/>
                        <WithholdingAmount t-out="float_repr(refund_multiplier*invoice['AmountsWithheld']['WithholdingAmount'], file_currency.decimal_places)"/>
                    </AmountsWithheld>
                    <TotalExecutableAmount t-out="float_repr(refund_multiplier*invoice['TotalExecutableAmount'], file_currency.decimal_places)"/>
                    <TotalReimbursableExpenses t-if="invoice.get('TotalReimbursableExpenses')" t-out="float_repr(refund_multiplier*invoice['TotalReimbursableExpenses'], file_currency.decimal_places)"/>
                </InvoiceTotals>
                <Items>
                    <t t-foreach="invoice['Items']" t-as="line"><t t-call="l10n_es_edi_facturae.invoice_line_type"/></t>
                </Items>
                <PaymentDetails t-if="invoice['PaymentDetails']">
                    <Installment t-foreach="invoice['PaymentDetails']" t-as="installment">
                        <InstallmentDueDate t-out="installment['InstallmentDueDate']"/>
                        <InstallmentAmount t-out="float_repr(installment['InstallmentAmount'], 2)"/>
                        <PaymentMeans t-out="installment['PaymentMeans']"/>
                        <AccountToBeCredited>
                            <IBAN t-out="installment['AccountToBeCredited']['IBAN']"/>
                            <BIC t-out="installment['AccountToBeCredited']['BIC']"/>
                        </AccountToBeCredited>
                    </Installment>
                </PaymentDetails>
                <LegalLiterals t-if="invoice.get('LegalLiterals')">
                    <t t-foreach="invoice['LegalLiterals']" t-as="reference"><LegalReference t-out="reference[:250]"/></t>
                </LegalLiterals>
            </Invoice>
        </template>

        <!-- Main template used for the EDI -->
        <template id="account_invoice_facturae_export">
            <fac:Facturae xmlns:fac="http://www.facturae.gob.es/formato/Versiones/Facturaev3_2_2.xml">
                <FileHeader>
                    <SchemaVersion>3.2.2</SchemaVersion>
                    <Modality t-out="Modality"/>
                    <InvoiceIssuerType t-out="'EM' if is_outstanding else 'RE'"/>
                    <Batch>
                        <BatchIdentifier t-out="BatchIdentifier"/>
                        <InvoicesCount t-out="InvoicesCount"/>
                        <TotalInvoicesAmount>
                            <TotalAmount t-out="float_repr(refund_multiplier*TotalInvoicesAmount['TotalAmount'], file_currency.decimal_places)"/>
                            <EquivalentInEuros t-if="conversion_needed" t-out="float_repr(refund_multiplier*TotalInvoicesAmount['EquivalentInEuros'], eur.decimal_places)"/>
                        </TotalInvoicesAmount>
                        <TotalOutstandingAmount>
                            <TotalAmount t-out="float_repr(refund_multiplier*TotalOutstandingAmount['TotalAmount'], file_currency.decimal_places)"/>
                            <EquivalentInEuros t-if="conversion_needed" t-out="float_repr(refund_multiplier*TotalOutstandingAmount['EquivalentInEuros'], eur.decimal_places)"/>
                        </TotalOutstandingAmount>
                        <TotalExecutableAmount>
                            <TotalAmount t-out="float_repr(refund_multiplier*TotalExecutableAmount['TotalAmount'], file_currency.decimal_places)"/>
                            <EquivalentInEuros t-if="conversion_needed" t-out="float_repr(refund_multiplier*TotalExecutableAmount['EquivalentInEuros'], eur.decimal_places)"/>
                        </TotalExecutableAmount>
                        <InvoiceCurrencyCode t-out="InvoiceCurrencyCode"/>
                    </Batch>
                </FileHeader>
                <Parties>
                    <SellerParty>
                        <t t-call="l10n_es_edi_facturae.business_type">
                            <t t-set="partner" t-value="self_party if is_outstanding else other_party"/>
                            <t t-set="partner_country_code" t-value="self_party_country_code if is_outstanding else other_party_country_code"/>
                            <t t-set="partner_phone" t-value="False if is_outstanding else other_party_phone"/>
                            <t t-set="partner_name" t-value="self_party_name if is_outstanding else other_party_name"/>
                        </t>
                    </SellerParty>
                    <BuyerParty>
                        <t t-call="l10n_es_edi_facturae.business_type">
                            <t t-set="partner" t-value="other_party if is_outstanding else self_party"/>
                            <t t-set="partner_country_code" t-value="other_party_country_code if is_outstanding else self_party_country_code"/>
                            <t t-set="partner_phone" t-value="other_party_phone if is_outstanding else False"/>
                            <t t-set="partner_name" t-value="other_party_name if is_outstanding else self_party_name"/>
                        </t>
                    </BuyerParty>
                </Parties>
                <Invoices>
                    <t t-foreach="Invoices" t-as="invoice"><t t-call="l10n_es_edi_facturae.invoice_type"/></t>
                </Invoices>
            </fac:Facturae>
        </template>
    </data>
</odoo>

```

## File: data\signature_templates.xml

```xml
<odoo>
    <data>
        <template id="template_xades_signature">
            <ds:Signature t-att-Id="signature_id" xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
                <ds:SignedInfo>
                    <ds:CanonicalizationMethod Algorithm="http://www.w3.org/TR/2001/REC-xml-c14n-20010315"/>
                    <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
                    <ds:Reference t-att-Id="reference_uri" Type="http://www.w3.org/2000/09/xmldsig#Object" URI="">
                        <ds:Transforms>
                            <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
                        </ds:Transforms>
                        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                        <ds:DigestValue></ds:DigestValue>
                    </ds:Reference>
                    <ds:Reference Type="http://uri.etsi.org/01903#SignedProperties" t-attf-URI="##{sigproperties_id}">
                        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                        <ds:DigestValue></ds:DigestValue>
                    </ds:Reference>
                    <ds:Reference t-attf-URI="##{keyinfo_id}">
                        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                        <ds:DigestValue></ds:DigestValue>
                    </ds:Reference>
                </ds:SignedInfo>
                <ds:SignatureValue></ds:SignatureValue>
                <ds:KeyInfo t-att-Id="keyinfo_id">
                    <ds:X509Data>
                        <ds:X509Certificate t-out="x509_certificate"/>
                    </ds:X509Data>
                    <ds:KeyValue>
                        <ds:RSAKeyValue>
                            <ds:Modulus t-out="public_modulus"/>
                            <ds:Exponent t-out="public_exponent"/>
                        </ds:RSAKeyValue>
                    </ds:KeyValue>
                </ds:KeyInfo>
                <ds:Object>
                    <xades:QualifyingProperties xmlns:xades="http://uri.etsi.org/01903/v1.3.2#" t-attf-Target="##{signature_id}">
                        <xades:SignedProperties t-att-Id="sigproperties_id">
                            <xades:SignedSignatureProperties>
                                <xades:SigningTime t-out="iso_now"/>
                                <xades:SigningCertificate>
                                    <xades:Cert>
                                        <xades:CertDigest>
                                            <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                                            <ds:DigestValue t-out="sigcertif_digest"/>
                                        </xades:CertDigest>
                                        <xades:IssuerSerial>
                                            <ds:X509IssuerName t-out="x509_issuer_description"/>
                                            <ds:X509SerialNumber t-out="x509_serial_number"/>
                                        </xades:IssuerSerial>
                                    </xades:Cert>
                                </xades:SigningCertificate>
                                <xades:SignaturePolicyIdentifier>
                                    <xades:SignaturePolicyId>
                                        <xades:SigPolicyId>
                                            <xades:Identifier t-out="sigpolicy_url"/>
                                            <xades:Description t-out="sigpolicy_description"/>
                                        </xades:SigPolicyId>
                                        <xades:SigPolicyHash>
                                            <ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
                                            <ds:DigestValue>Ohixl6upD6av8N7pEvDABhEL6hM=</ds:DigestValue>
                                        </xades:SigPolicyHash>
                                        <xades:SigPolicyQualifiers>
                                            <xades:SigPolicyQualifier>
                                                <xades:SPURI t-out="sigpolicy_url"/>
                                            </xades:SigPolicyQualifier>
                                        </xades:SigPolicyQualifiers>
                                    </xades:SignaturePolicyId>
                                </xades:SignaturePolicyIdentifier>
                            </xades:SignedSignatureProperties>
                            <xades:SignedDataObjectProperties>
                                <xades:DataObjectFormat t-attf-ObjectReference="##{reference_uri}">
                                    <xades:Description/>
                                    <xades:ObjectIdentifier>
                                        <!-- Identifier below is Uniform Resource Number for MimeType "text/xml" (no specific version) -->
                                        <!-- 1.2.840.10003.5: see http://www.oid-info.com/cgi-bin/display?oid=1.2.840.10003.5&a=display -->
                                        <!-- 109.10: see https://www.loc.gov/z3950/agency/defns/oids.html#5 -->
                                        <xades:Identifier Qualifier="OIDAsURN">urn:oid:1.2.840.10003.5.109.10</xades:Identifier>
                                        <xades:Description/>
                                    </xades:ObjectIdentifier>
                                    <xades:MimeType>text/xml</xades:MimeType>
                                    <xades:Encoding/>
                                </xades:DataObjectFormat>
                            </xades:SignedDataObjectProperties>
                        </xades:SignedProperties>
                    </xades:QualifyingProperties>
                </ds:Object>
            </ds:Signature>
        </template>
    </data>
</odoo>

```

## File: data\uom.uom.csv

```csv
"id","l10n_es_edi_facturae_uom_code"
uom.product_uom_unit,01
uom.product_uom_dozen,18
uom.product_uom_hour,02
uom.product_uom_meter,25
uom.product_uom_millimeter,26
uom.product_uom_km,22
uom.product_uom_cm,16
uom.uom_square_meter,05
uom.product_uom_litre,04
uom.product_uom_cubic_meter,33
uom.product_uom_kgm,03
uom.product_uom_gram,21
uom.product_uom_ton,05

```

## File: data\template\account.tax-es_common.csv

```csv
"id","l10n_es_edi_facturae_tax_type"
"account_tax_template_s_iva21b","01"
"account_tax_template_p_iva4_bc","01"
"account_tax_template_p_iva4_ibc","01"
"account_tax_template_p_iva4_sc","01"
"account_tax_template_p_iva4_sp_ex","01"
"account_tax_template_p_iva4_bi","01"
"account_tax_template_p_iva4_ibi","01"
"account_tax_template_p_iva10_bc","01"
"account_tax_template_p_iva10_ibc","01"
"account_tax_template_p_iva10_sc","01"
"account_tax_template_p_iva10_sp_ex","01"
"account_tax_template_p_iva10_bi","01"
"account_tax_template_p_iva10_ibi","01"
"account_tax_template_p_iva21_bc","01"
"account_tax_template_p_iva21_ibc","01"
"account_tax_template_p_iva21_sc","01"
"account_tax_template_p_iva21_sp_ex","01"
"account_tax_template_p_iva21_bi","01"
"account_tax_template_p_iva21_ibi","01"
"account_tax_template_s_iva4b","01"
"account_tax_template_s_iva0_e","01"
"account_tax_template_s_iva4s","01"
"account_tax_template_s_iva_e","01"
"account_tax_template_s_iva10b","01"
"account_tax_template_s_iva10s","01"
"account_tax_template_s_iva21s","01"
"account_tax_template_p_iva4_ic_bc","01"
"account_tax_template_p_iva4_sp_in","01"
"account_tax_template_p_iva4_ic_bi","01"
"account_tax_template_p_iva10_ic_bc","01"
"account_tax_template_p_iva10_sp_in","01"
"account_tax_template_p_iva10_ic_bi","01"
"account_tax_template_p_iva21_ic_bc","01"
"account_tax_template_p_iva21_sp_in","01"
"account_tax_template_p_iva21_ic_bi","01"
"account_tax_template_s_iva0_ic","01"
"account_tax_template_s_iva0_sp_i","01"
"account_tax_template_p_iva0_ns_b","01"
"account_tax_template_p_iva0_ns","01"
"account_tax_template_s_iva_ns_b","01"
"account_tax_template_s_iva_ns","01"
"account_tax_template_s_req05","05"
"account_tax_template_s_req014","05"
"account_tax_template_s_req52","05"
"account_tax_template_p_req05","05"
"account_tax_template_p_req014","05"
"account_tax_template_p_req52","05"
"account_tax_template_s_iva0_isp","01"
"account_tax_template_s_irpf19a","04"
"account_tax_template_p_irpf19a","04"
"account_tax_template_s_iva0","01"
"account_tax_template_p_iva0_bc","01"
"account_tax_template_s_irpf195a","04"
"account_tax_template_p_irpf195a","04"
"account_tax_template_s_irpf20a","04"
"account_tax_template_p_irpf20a","04"
"account_tax_template_s_irpf21a","04"
"account_tax_template_p_irpf21a","04"
"account_tax_template_s_irpf21","04"
"account_tax_template_s_iva0_ns","01"
"account_tax_template_p_irpf21p","04"
"account_tax_template_s_irpf20","04"
"account_tax_template_p_irpf20","04"
"account_tax_template_s_irpf24","04"
"account_tax_template_p_irpf24","04"
"account_tax_template_s_irpf15","04"
"account_tax_template_p_irpf15","04"
"account_tax_template_s_irpf18","04"
"account_tax_template_p_irpf18","04"
"account_tax_template_s_irpf19","04"
"account_tax_template_p_irpf19","04"
"account_tax_template_s_irpf9","04"
"account_tax_template_p_irpf9","04"
"account_tax_template_s_irpf7","04"
"account_tax_template_p_irpf7","04"
"account_tax_template_s_irpf2","04"
"account_tax_template_p_irpf2","04"
"account_tax_template_s_irpf1","04"
"account_tax_template_p_irpf1","04"
"account_tax_template_p_iva4_isp","01"
"account_tax_template_p_iva4_isp_bi","01"
"account_tax_template_p_iva10_isp","01"
"account_tax_template_p_iva10_isp_bi","01"
"account_tax_template_p_iva21_isp","01"
"account_tax_template_p_iva21_isp_bi","01"
"account_tax_template_p_iva12_agr","01"
"account_tax_template_p_iva105_gan","01"
"account_tax_template_s_iva0_g_i","01"
"account_tax_template_s_iva0_g_e","01"

```

## File: models\account_chart_template.py

```python
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('es_common', 'account.tax')
    def _get_es_facturae_account_tax(self):
        taxes = self._parse_csv('es_common', 'account.tax', module='l10n_es_edi_facturae')
        # only return existing taxes
        taxes = {
            key: value
            for key, value in taxes.items()
            if self.env['account.chart.template'].ref(key, raise_if_not_found=False)
        }
        return taxes

```

## File: models\account_move.py

```python
import re

from collections import defaultdict
from markupsafe import Markup

from odoo import fields, models, api, _, Command
from odoo.exceptions import UserError
from odoo.tools import float_repr, date_utils
from odoo.tools.xml_utils import cleanup_xml_node, find_xml_value

PHONE_CLEAN_TABLE = str.maketrans({" ": None, "-": None, "(": None, ")": None, "+": None})
COUNTRY_CODE_MAP = {
    "BD": "BGD", "BE": "BEL", "BF": "BFA", "BG": "BGR", "BA": "BIH", "BB": "BRB", "WF": "WLF", "BL": "BLM", "BM": "BMU",
    "BN": "BRN", "BO": "BOL", "BH": "BHR", "BI": "BDI", "BJ": "BEN", "BT": "BTN", "JM": "JAM", "BV": "BVT", "BW": "BWA",
    "WS": "WSM", "BQ": "BES", "BR": "BRA", "BS": "BHS", "JE": "JEY", "BY": "BLR", "BZ": "BLZ", "RU": "RUS", "RW": "RWA",
    "RS": "SRB", "TL": "TLS", "RE": "REU", "TM": "TKM", "TJ": "TJK", "RO": "ROU", "TK": "TKL", "GW": "GNB", "GU": "GUM",
    "GT": "GTM", "GS": "SGS", "GR": "GRC", "GQ": "GNQ", "GP": "GLP", "JP": "JPN", "GY": "GUY", "GG": "GGY", "GF": "GUF",
    "GE": "GEO", "GD": "GRD", "GB": "GBR", "GA": "GAB", "SV": "SLV", "GN": "GIN", "GM": "GMB", "GL": "GRL", "GI": "GIB",
    "GH": "GHA", "OM": "OMN", "TN": "TUN", "JO": "JOR", "HR": "HRV", "HT": "HTI", "HU": "HUN", "HK": "HKG", "HN": "HND",
    "HM": "HMD", "VE": "VEN", "PR": "PRI", "PS": "PSE", "PW": "PLW", "PT": "PRT", "SJ": "SJM", "PY": "PRY", "IQ": "IRQ",
    "PA": "PAN", "PF": "PYF", "PG": "PNG", "PE": "PER", "PK": "PAK", "PH": "PHL", "PN": "PCN", "PL": "POL", "PM": "SPM",
    "ZM": "ZMB", "EH": "ESH", "EE": "EST", "EG": "EGY", "ZA": "ZAF", "EC": "ECU", "IT": "ITA", "VN": "VNM", "SB": "SLB",
    "ET": "ETH", "SO": "SOM", "ZW": "ZWE", "SA": "SAU", "ES": "ESP", "ER": "ERI", "ME": "MNE", "MD": "MDA", "MG": "MDG",
    "MF": "MAF", "MA": "MAR", "MC": "MCO", "UZ": "UZB", "MM": "MMR", "ML": "MLI", "MO": "MAC", "MN": "MNG", "MH": "MHL",
    "MK": "MKD", "MU": "MUS", "MT": "MLT", "MW": "MWI", "MV": "MDV", "MQ": "MTQ", "MP": "MNP", "MS": "MSR", "MR": "MRT",
    "IM": "IMN", "UG": "UGA", "TZ": "TZA", "MY": "MYS", "MX": "MEX", "IL": "ISR", "FR": "FRA", "IO": "IOT", "SH": "SHN",
    "FI": "FIN", "FJ": "FJI", "FK": "FLK", "FM": "FSM", "FO": "FRO", "NI": "NIC", "NL": "NLD", "NO": "NOR", "NA": "NAM",
    "VU": "VUT", "NC": "NCL", "NE": "NER", "NF": "NFK", "NG": "NGA", "NZ": "NZL", "NP": "NPL", "NR": "NRU", "NU": "NIU",
    "CK": "COK", "XK": "XKX", "CI": "CIV", "CH": "CHE", "CO": "COL", "CN": "CHN", "CM": "CMR", "CL": "CHL", "CC": "CCK",
    "CA": "CAN", "CG": "COG", "CF": "CAF", "CD": "COD", "CZ": "CZE", "CY": "CYP", "CX": "CXR", "CR": "CRI", "CW": "CUW",
    "CV": "CPV", "CU": "CUB", "SZ": "SWZ", "SY": "SYR", "SX": "SXM", "KG": "KGZ", "KE": "KEN", "SS": "SSD", "SR": "SUR",
    "KI": "KIR", "KH": "KHM", "KN": "KNA", "KM": "COM", "ST": "STP", "SK": "SVK", "KR": "KOR", "SI": "SVN", "KP": "PRK",
    "KW": "KWT", "SN": "SEN", "SM": "SMR", "SL": "SLE", "SC": "SYC", "KZ": "KAZ", "KY": "CYM", "SG": "SGP", "SE": "SWE",
    "SD": "SDN", "DO": "DOM", "DM": "DMA", "DJ": "DJI", "DK": "DNK", "VG": "VGB", "DE": "DEU", "YE": "YEM", "DZ": "DZA",
    "US": "USA", "UY": "URY", "YT": "MYT", "UM": "UMI", "LB": "LBN", "LC": "LCA", "LA": "LAO", "TV": "TUV", "TW": "TWN",
    "TT": "TTO", "TR": "TUR", "LK": "LKA", "LI": "LIE", "LV": "LVA", "TO": "TON", "LT": "LTU", "LU": "LUX", "LR": "LBR",
    "LS": "LSO", "TH": "THA", "TF": "ATF", "TG": "TGO", "TD": "TCD", "TC": "TCA", "LY": "LBY", "VA": "VAT", "VC": "VCT",
    "AE": "ARE", "AD": "AND", "AG": "ATG", "AF": "AFG", "AI": "AIA", "VI": "VIR", "IS": "ISL", "IR": "IRN", "AM": "ARM",
    "AL": "ALB", "AO": "AGO", "AQ": "ATA", "AS": "ASM", "AR": "ARG", "AU": "AUS", "AT": "AUT", "AW": "ABW", "IN": "IND",
    "AX": "ALA", "AZ": "AZE", "IE": "IRL", "ID": "IDN", "UA": "UKR", "QA": "QAT", "MZ": "MOZ"
}
REVERSED_COUNTRY_CODE = {v: k for k, v in COUNTRY_CODE_MAP.items()}

class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_es_edi_facturae_xml_id = fields.Many2one(
        comodel_name='ir.attachment',
        string="Facturae Attachment",
        compute=lambda self: self._compute_linked_attachment_id('l10n_es_edi_facturae_xml_id', 'l10n_es_edi_facturae_xml_file'),
        depends=['l10n_es_edi_facturae_xml_file']
    )
    l10n_es_edi_facturae_xml_file = fields.Binary(
        attachment=True,
        string="Facturae File",
        copy=False,
    )
    l10n_es_edi_facturae_reason_code = fields.Selection(
        selection=[
            ('01', "Invoice number"),
            ('02', "Invoice serial number"),
            ('03', "Issue date"),
            ('04', "Name and surnames/Corporate name - Issuer (Sender)"),
            ('05', "Name and surnames/Corporate name - Receiver"),
            ('06', "Issuer's Tax Identification Number"),
            ('07', "Receiver's Tax Identification Number"),
            ('08', "Issuer's address"),
            ('09', "Receiver's address"),
            ('10', "Item line"),
            ('11', "Applicable Tax Rate"),
            ('12', "Applicable Tax Amount"),
            ('13', "Applicable Date/Period"),
            ('14', "Invoice Class"),
            ('15', "Legal literals"),
            ('16', "Taxable Base"),
            ('80', "Calculation of tax outputs"),
            ('81', "Calculation of tax inputs"),
            ('82', "Taxable Base modified due to return of packages and packaging materials"),
            ('83', "Taxable Base modified due to discounts and rebates"),
            ('84', "Taxable Base modified due to firm court ruling or administrative decision"),
            ('85', "Taxable Base modified due to unpaid outputs where there is a judgement opening insolvency proceedings"),
        ], string='Spanish Facturae EDI Reason Code', default='10')

    def _l10n_es_edi_facturae_get_default_enable(self):
        self.ensure_one()
        return not self.invoice_pdf_report_id \
            and not self.l10n_es_edi_facturae_xml_id \
            and not self.l10n_es_is_simplified \
            and self.is_invoice(include_receipts=True) \
            and (self.partner_id.is_company or self.partner_id.vat) \
            and self.company_id.country_code == 'ES' \
            and self.company_id.currency_id.name == 'EUR'

    def _l10n_es_edi_facturae_get_filename(self):
        self.ensure_one()
        return f'{self.name.replace("/", "_")}_facturae_signed.xml'

    def _l10n_es_edi_facturae_get_tax_period(self):
        self.ensure_one()
        if self.env['res.company'].fields_get(['account_tax_periodicity']):
            period_start, period_end = self.company_id._get_tax_closing_period_boundaries(self.date)
        else:
            period_start = date_utils.start_of(self.date, 'month')
            period_end = date_utils.end_of(self.date, 'month')

        return {'start':period_start, 'end':period_end}

    def _l10n_es_edi_facturae_get_refunded_invoices(self):
        self.env['account.partial.reconcile'].flush_model()
        invoices_refunded_mapping = {invoice.id: invoice.reversed_entry_id.id for invoice in self}

        queries = []
        for source_field, counterpart_field in (('debit', 'credit'), ('credit', 'debit')):
            queries.append(f'''
                SELECT
                    source_line.move_id AS source_move_id,
                    counterpart_line.move_id AS counterpart_move_id
                FROM account_partial_reconcile part
                JOIN account_move_line source_line ON source_line.id = part.{source_field}_move_id
                JOIN account_move_line counterpart_line ON counterpart_line.id = part.{counterpart_field}_move_id
                WHERE source_line.move_id IN %s AND counterpart_line.move_id != source_line.move_id
                GROUP BY source_move_id, counterpart_move_id
            ''')
        self._cr.execute(' UNION ALL '.join(queries), [tuple(self.ids)] * 2)
        payment_data = defaultdict(lambda: [])
        for row in self._cr.dictfetchall():
            payment_data[row['source_move_id']].append(row)

        for invoice in self:
            if not invoice.move_type.endswith('refund'):
                # We only want to map refunds
                continue

            for move_id in (record_dict['counterpart_move_id'] for record_dict in payment_data.get(invoice.id, [])):
                invoices_refunded_mapping[invoice.id] = move_id
        return invoices_refunded_mapping

    def _l10n_es_edi_facturae_get_corrective_data(self):
        self.ensure_one()
        if self.move_type.endswith('refund'):
            if not self.reversed_entry_id:
                raise UserError(_("The credit note/refund appears to have been issued manually. For the purpose of "
                                  "generating a Facturae document, it's necessary that the credit note/refund is created "
                                  "directly from the associated invoice/bill."))

            refunded_invoice = self.env['account.move'].browse(self._l10n_es_edi_facturae_get_refunded_invoices()[self.id])
            tax_period = refunded_invoice._l10n_es_edi_facturae_get_tax_period()

            reason_code = self.l10n_es_edi_facturae_reason_code or '10'
            reason_description = [label for code, label in self._fields['l10n_es_edi_facturae_reason_code'].selection
                                  if code == reason_code][0]
            return {
                'refunded_invoice_record': refunded_invoice,
                'ReasonCode': reason_code,
                'Reason': reason_description,
                'TaxPeriod': {
                    'StartDate': tax_period.get('start'),
                    'EndDate': tax_period.get('end'),
                }
            }
        return {}

    @api.model
    def _l10n_es_edi_facturae_convert_computed_tax_to_template(self, computed_tax_dict):
        """ Helper to convert the tax dict from a _compute_taxes() into a dict usable in the template """
        tax = self.env["account.tax"].browse(computed_tax_dict["tax_id"])
        return {
            "tax_record": tax,
            "TaxRate": f'{abs(tax.amount):.3f}',
            "TaxableBase": {
                'TotalAmount': computed_tax_dict["base_amount_currency"],
                'EquivalentInEuros': computed_tax_dict["base_amount"],
            },
            "TaxAmount": {
                "TotalAmount": abs(computed_tax_dict["tax_amount_currency"]),
                "EquivalentInEuros": abs(computed_tax_dict["tax_amount"]),
            },
        }

    def _l10n_es_edi_facturae_convert_payment_terms_to_installments(self):
        """
        Convert the payments terms to a list of <Installment> elements to be used in the
        <PaymentDetails> node of the Facturae XML generation.

        For now we only use the hardcoded '04' value (Credit Transfer).
        """
        self.ensure_one()
        installments = []
        if self.is_inbound() and self.partner_bank_id:
            for payment_term in self.line_ids.filtered(lambda l: l.display_type == 'payment_term').sorted('date_maturity'):
                installments.append({
                    'InstallmentDueDate': payment_term.date_maturity,
                    'InstallmentAmount': payment_term.amount_residual_currency,
                    'PaymentMeans': '04',  # Credit Transfer
                    'AccountToBeCredited': {
                        'IBAN': self.partner_bank_id.sanitized_acc_number,
                        'BIC': self.partner_bank_id.bank_bic,
                    },
                })
        return installments

    def _l10n_es_edi_facturae_inv_lines_to_items(self, conversion_rate=None):
        """
        Convert the invoice lines to a list of items required for the Facturae xml generation

        :param float conversion_rate: Conversion rate of the invoice, if needed
        :return: A tuple containing the Face items, the taxes and the invoice totals data.
        """
        self.ensure_one()
        items = []
        totals = {
            'total_gross_amount': 0.,
            'total_general_discounts': 0.,
            'total_general_surcharges': 0.,
            'total_taxes_withheld': 0.,
            'total_tax_outputs': 0.,
            'total_payments_on_account': 0.,
            'amounts_withheld': 0.,
        }
        taxes = []
        taxes_withheld = []
        invoice_ref = self.ref[:20] if self.ref else False
        for line in self.invoice_line_ids:
            if line.display_type in {'line_section', 'line_note'}:
                continue
            invoice_line_values = {}

            tax_base_before_discount = self.env['account.tax']._convert_to_tax_base_line_dict(
                base_line=line,
                currency=line.currency_id,
                taxes=line.tax_ids,
                price_unit=line.price_unit,
                quantity=line.quantity,
            )
            tax_before_discount = self.env['account.tax']._compute_taxes([tax_base_before_discount])
            price_before_discount = sum(to_update['price_subtotal'] for _dummy, to_update in tax_before_discount['base_lines_to_update'])
            discount = max(0., (price_before_discount - line.price_subtotal))
            surcharge = abs(min(0., (price_before_discount - line.price_subtotal)))
            totals['total_gross_amount'] += line.price_subtotal
            base_line = self.env['account.tax']._convert_to_tax_base_line_dict(
                line, partner=line.partner_id, currency=line.currency_id, product=line.product_id, taxes=line.tax_ids,
                price_unit=line.price_unit, quantity=line.quantity, discount=line.discount, account=line.account_id,
                price_subtotal=line.price_subtotal, is_refund=line.is_refund, rate=conversion_rate
            )

            taxes_computed = self.env['account.tax']._compute_taxes([base_line])
            taxes_withheld_computed = [tax for tax in taxes_computed["tax_lines_to_add"] if tax["tax_amount"] < 0]
            taxes_normal_computed = [tax for tax in taxes_computed["tax_lines_to_add"] if tax["tax_amount"] >= 0]

            taxes_output = [self._l10n_es_edi_facturae_convert_computed_tax_to_template(tax) for tax in taxes_normal_computed]
            totals['total_tax_outputs'] += sum((abs(tax["tax_amount"]) for tax in taxes_normal_computed))

            tax_withheld_output = [self._l10n_es_edi_facturae_convert_computed_tax_to_template(tax) for tax in taxes_withheld_computed]
            totals['total_taxes_withheld'] += sum((abs(tax["tax_amount"]) for tax in taxes_withheld_computed))

            receiver_transaction_reference = (
                line.sale_line_ids.order_id.client_order_ref[:20]
                if 'sale_line_ids' in line._fields and line.sale_line_ids.order_id.client_order_ref
                else invoice_ref
            )

            invoice_line_values.update({
                'ReceiverTransactionReference': receiver_transaction_reference,
                'FileReference': invoice_ref,
                'ReceiverContractReference': invoice_ref,
                'FileDate': fields.Date.context_today(self),
                'ItemDescription': line.name,
                'Quantity': line.quantity,
                'UnitOfMeasure': line.product_uom_id.l10n_es_edi_facturae_uom_code,
                'UnitPriceWithoutTax': line.currency_id.round(price_before_discount / line.quantity if line.quantity else 0.),
                'TotalCost': price_before_discount,
                'DiscountsAndRebates': [{
                    'DiscountReason': '/',
                    'DiscountRate': f'{line.discount:.2f}',
                    'DiscountAmount': discount
                }, ] if discount != 0. else [],
                'Charges': [{
                    'ChargeReason': '/',
                    'ChargeRate': f'{max(0, -line.discount):.2f}',
                    'ChargeAmount': surcharge,
                }, ] if surcharge != 0. else [],
                'GrossAmount': line.price_subtotal,
                'TaxesOutputs': taxes_output,
                'TaxesWithheld': tax_withheld_output,
            })
            items.append(invoice_line_values)
            taxes += taxes_output
            taxes_withheld += tax_withheld_output
        return items, taxes, taxes_withheld, totals

    def _l10n_es_edi_facturae_export_facturae(self):
        """
        Produce the Facturae XML data for the invoice.

        :return: (data needed to render the full template, data needed to render the signature template)
        """
        def extract_party_name(party):
            name = {'firstname': 'UNKNOWN', 'surname': 'UNKNOWN', 'surname2': ''}
            if not party.is_company:
                name_split = [part for part in party.name.replace(', ', ' ').split(' ') if part]
                if len(name_split) > 2:
                    name['firstname'] = ' '.join(name_split[:-2])
                    name['surname'], name['surname2'] = name_split[-2:]
                elif len(name_split) == 2:
                    name['firstname'] = ' '.join(name_split[:-1])
                    name['surname'] = name_split[-1]
            return name

        self.ensure_one()
        company = self.company_id
        partner = self.commercial_partner_id

        if not company.vat:
            raise UserError(_('The company needs a set tax identification number or VAT number'))
        if not partner.vat:
            raise UserError(_('The partner needs a set tax identification number or VAT number'))
        if not partner.country_id:
            raise UserError(_("The partner needs a set country"))
        if self.move_type == "entry":
            return False

        operation_date = None
        if self.delivery_date and self.delivery_date != self.invoice_date:
            operation_date = self.delivery_date.isoformat()

        eur_curr = self.env['res.currency'].search([('name', '=', 'EUR')])
        inv_curr = self.currency_id
        legal_literals = self.narration.striptags() if self.narration else False
        legal_literals = legal_literals.split(";") if legal_literals else False

        invoice_issuer_signature_type = 'supplier' if self.move_type == 'out_invoice' else 'customer'
        need_conv = bool(inv_curr != eur_curr)
        conversion_rate = abs(self.amount_total_in_currency_signed / self.amount_total_signed) if self.amount_total_signed else 0.
        total_outst_am_in_currency = abs(self.amount_total_in_currency_signed)
        total_outst_am = abs(self.amount_total_signed)
        total_exec_am_in_currency = abs(self.amount_total_in_currency_signed)
        total_exec_am = abs(self.amount_total_signed)
        items, taxes, taxes_withheld, totals = self._l10n_es_edi_facturae_inv_lines_to_items(conversion_rate)
        template_values = {
            'self_party': company.partner_id,
            'self_party_country_code': COUNTRY_CODE_MAP[company.country_id.code],
            'self_party_name': extract_party_name(company.partner_id),
            'other_party': partner,
            'other_party_country_code': COUNTRY_CODE_MAP[partner.country_id.code],
            'other_party_phone': partner.phone.translate(PHONE_CLEAN_TABLE) if partner.phone else False,
            'other_party_name': extract_party_name(partner),
            'is_outstanding': self.move_type.startswith('out_'),
            'float_repr': float_repr,
            'file_currency': inv_curr,
            'eur': eur_curr,
            'conversion_needed': need_conv,
            'refund_multiplier': -1 if self.move_type.endswith('refund') else 1,

            'Modality': 'I',
            'BatchIdentifier': self.name,
            'InvoicesCount': 1,
            'TotalInvoicesAmount': {
                'TotalAmount': abs(self.amount_total_in_currency_signed),
                'EquivalentInEuros': abs(self.amount_total_signed),
            },
            'TotalOutstandingAmount': {
                'TotalAmount': abs(total_outst_am_in_currency),
                'EquivalentInEuros': abs(total_outst_am),
            },
            'TotalExecutableAmount': {
                'TotalAmount': total_exec_am_in_currency,
                'EquivalentInEuros': total_exec_am,
            },
            'InvoiceCurrencyCode': inv_curr.name,
            'Invoices': [{
                'invoice_record': self,
                'invoice_currency': inv_curr,
                'InvoiceDocumentType': 'FC',
                'InvoiceClass': 'OO',
                'Corrective': self._l10n_es_edi_facturae_get_corrective_data(),
                'InvoiceIssueData': {
                    'OperationDate': operation_date,
                    'ExchangeRateDetails': need_conv,
                    'ExchangeRate': f"{round(conversion_rate, 4):.4f}",
                    'LanguageName': self._context.get('lang', 'en_US').split('_')[0],
                    'ReceiverTransactionReference': self.ref[:20] if self.ref else False,
                    'FileReference': self.ref[:20] if self.ref else False,
                    'ReceiverContractReference': self.ref[:20] if self.ref else False,
                },
                'TaxOutputs': taxes,
                'TaxesWithheld': taxes_withheld,
                'TotalGrossAmount': totals['total_gross_amount'],
                'TotalGeneralDiscounts': totals['total_general_discounts'],
                'TotalGeneralSurcharges': totals['total_general_surcharges'],
                'TotalGrossAmountBeforeTaxes': totals['total_gross_amount'] - totals['total_general_discounts'] + totals['total_general_surcharges'],
                'TotalTaxOutputs': totals['total_tax_outputs'],
                'TotalTaxesWithheld': totals['total_taxes_withheld'],
                'PaymentsOnAccount': [],
                'TotalOutstandingAmount': total_outst_am_in_currency,
                'InvoiceTotal': abs(self.amount_total_in_currency_signed),
                'TotalPaymentsOnAccount': totals['total_payments_on_account'],
                'AmountsWithheld': {
                    'WithholdingReason': '',
                    'WithholdingRate': False,
                    'WithholdingAmount': totals['amounts_withheld'],
                } if totals['amounts_withheld'] else False,
                'TotalExecutableAmount': total_exec_am_in_currency,
                'Items': items,
                'PaymentDetails': self._l10n_es_edi_facturae_convert_payment_terms_to_installments(),
                'LegalLiterals': legal_literals,
            }],
        }
        signature_values = {'SigningTime': '', 'SignerRole': invoice_issuer_signature_type, }
        return template_values, signature_values

    def _l10n_es_edi_facturae_render_facturae(self):
        """
        Produce the Facturae XML file for the invoice.

        :return: rendered xml file string.
        :rtype:  str
        """
        self.ensure_one()
        company = self.company_id
        template_values, signature_values = self._l10n_es_edi_facturae_export_facturae()
        xml_content = cleanup_xml_node(self.env['ir.qweb']._render('l10n_es_edi_facturae.account_invoice_facturae_export', template_values))
        certificate = self.env['l10n_es_edi_facturae.certificate'].search([("company_id", '=', company.id)], limit=1)

        errors = []
        try:
            xml_content = certificate._sign_xml(xml_content, signature_values)
        except ValueError:
            errors.append(_('No valid certificate found for this company, Facturae EDI file will not be signed.\n'))
        return xml_content, errors

    # -------------------------------------------------------------------------
    # IMPORT
    # -------------------------------------------------------------------------

    def _get_edi_decoder(self, file_data, new=False):
        def is_facturae(tree):
            return tree.tag in [
                '{http://www.facturae.es/Facturae/2014/v3.2.1/Facturae}Facturae',
                '{http://www.facturae.gob.es/formato/Versiones/Facturaev3_2_2.xml}Facturae',
            ]

        if file_data['type'] == 'xml' and is_facturae(file_data['xml_tree']):
            return self._import_invoice_facturae

        return super()._get_edi_decoder(file_data, new=new)

    def _import_invoice_facturae(self, invoice, file_data, new=False):
        tree = file_data['xml_tree']
        is_bill = invoice.move_type.startswith('in_')
        partner = self._import_get_partner(tree, is_bill)
        self._import_invoice_facturae_invoices(invoice, partner, tree)

    def _import_get_partner(self, tree, is_bill):
        # If we're dealing with a vendor bill, then the partner is the seller party, if an invoice then it's the buyer.
        party = tree.xpath('//SellerParty') if is_bill else tree.xpath('//BuyerParty')
        if party:
            partner_vals = self._import_extract_partner_values(party[0])
            return self._import_create_or_retrieve_partner(partner_vals)
        return None

    def _import_extract_partner_values(self, party_node):
        name = find_xml_value('.//CorporateName|.//Name', party_node)
        first_surname = find_xml_value('.//FirstSurname', party_node)
        second_surname = find_xml_value('.//SecondSurname', party_node)
        phone = find_xml_value('.//Telephone', party_node)
        mail = find_xml_value('.//ElectronicMail', party_node)
        country_code = find_xml_value('.//CountryCode', party_node)
        vat = find_xml_value('.//TaxIdentificationNumber', party_node)

        full_name = ' '.join(part for part in [name, first_surname, second_surname] if part)

        return {'name': full_name, 'vat': vat, 'phone': phone, 'email': mail, 'country_code': country_code}

    def _import_create_or_retrieve_partner(self, partner_vals):
        name = partner_vals['name']
        vat = partner_vals['vat']
        phone = partner_vals['phone']
        mail = partner_vals['email']
        country_code = partner_vals['country_code']

        partner = self.env['res.partner']._retrieve_partner(name=name, vat=vat, phone=phone, mail=mail)

        if not partner and name:
            partner_vals = {'name': name, 'email': mail, 'phone': phone}
            country_code = REVERSED_COUNTRY_CODE.get(country_code)
            country = self.env['res.country'].search([('code', '=', country_code)]) if country_code else False
            if country:
                partner_vals['country_id'] = country.id
            partner = self.env['res.partner'].create(partner_vals)
            if vat and self.env['res.partner']._run_vat_test(vat, country):
                partner.vat = vat

        return partner

    def _import_invoice_facturae_invoices(self, invoice, partner, tree):
        invoices = tree.xpath('//Invoice')
        if not invoices:
            return

        self._import_invoice_facturae_invoice(invoice, partner, invoices[0])

        # There might be other invoices inside the facturae.
        for node in invoices[1:]:
            other_invoice = invoice.create({
                'journal_id': invoice.journal_id.id,
                'move_type': invoice.move_type
            })
            with other_invoice._get_edi_creation():
                self._import_invoice_facturae_invoice(other_invoice, partner, node)
                other_invoice.message_post(body=_("Created from attachment in %s", invoice._get_html_link()))

    def _import_invoice_facturae_invoice(self, invoice, partner, tree):
        logs = []

        # ==== move_type ====
        invoice_total = find_xml_value('.//InvoiceTotal', tree)
        is_refund = float(invoice_total) < 0 if invoice_total else False
        if is_refund:
            invoice.move_type = "in_refund" if invoice.move_type.startswith("in_") else "out_refund"
        ref_multiplier = -1.0 if is_refund else 1.0

        # ==== partner_id ====
        if partner:
            invoice.partner_id = partner
        else:
            logs.append(_("Customer/Vendor could not be found and could not be created due to missing data in the XML."))

        # ==== currency_id ====
        invoice_currency_code = find_xml_value('.//InvoiceCurrencyCode', tree)
        if invoice_currency_code:
            currency = self.env['res.currency'].search([('name', '=', invoice_currency_code)], limit=1)
            if currency:
                invoice.currency_id = currency
            else:
                logs.append(_("Could not retrieve currency: %s. Did you enable the multicurrency option "
                              "and activate the currency?", invoice_currency_code))

        # ==== invoice date ====
        if issue_date := find_xml_value('.//IssueDate', tree):
            invoice.invoice_date = issue_date

        # ==== invoice_date_due ====
        if end_date := find_xml_value('.//InstallmentDueDate', tree):
            invoice.invoice_date_due = end_date

        # ==== ref ====
        if invoice_number := find_xml_value('.//InvoiceNumber', tree):
            invoice.ref = invoice_number

        # ==== narration ====
        invoice.narration = "\n".join(
            ref.text
            for ref in tree.xpath('.//LegalReference')
            if ref.text
        )

        # === invoice_line_ids ===
        logs += self._import_invoice_fill_lines(invoice, tree, ref_multiplier)

        body = Markup("<strong>%s</strong>") % _("Invoice imported from Factura-E XML file.")

        if logs:
            body += Markup("<ul>%s</ul>") \
                    % Markup().join(Markup("<li>%s</li>") % log for log in logs)

        invoice.message_post(body=body)

        return logs

    def _import_invoice_fill_lines(self, invoice, tree, ref_multiplier):
        lines = tree.xpath('.//InvoiceLine')
        logs = []
        vals_list = []
        for line in lines:
            line_vals = {'move_id': invoice.id}

            # ==== name ====
            if item_description := find_xml_value('.//ItemDescription', line):
                product = self._search_product_for_import(item_description)
                if product:
                    line_vals['product_id'] = product.id
                else:
                    logs.append(_("The product '%s' could not be found.", item_description))
                line_vals['name'] = item_description

            # ==== quantity ====
            line_vals['quantity'] = find_xml_value('.//Quantity', line) or 1

            # ==== price_unit ====
            price_unit = find_xml_value('.//UnitPriceWithoutTax', line)
            line_vals['price_unit'] = ref_multiplier * float(price_unit) if price_unit else 1.0

            # ==== discount ====
            discounts = line.xpath('.//DiscountRate')
            discount_rate = 0.0
            for discount in discounts:
                discount_rate += float(discount.text)

            charges = line.xpath('.//ChargeRate')
            charge_rate = 0.0
            for charge in charges:
                charge_rate += float(charge.text)

            discount_rate -= charge_rate
            line_vals['discount'] = discount_rate

            # ==== tax_ids ====
            taxes_withheld_nodes = line.xpath('.//TaxesWithheld/Tax')
            taxes_outputs_nodes = line.xpath('.//TaxesOutputs/Tax')
            is_purchase = invoice.move_type.startswith('in')
            tax_ids = []
            logs += self._import_fill_invoice_line_taxes(invoice, line_vals, tax_ids, taxes_outputs_nodes, False, is_purchase)
            logs += self._import_fill_invoice_line_taxes(invoice, line_vals, tax_ids, taxes_withheld_nodes, True, is_purchase)
            line_vals['tax_ids'] = [Command.set(tax_ids)]
            vals_list.append(line_vals)

        invoice.invoice_line_ids = self.env['account.move.line'].create(vals_list)
        return logs

    def _import_fill_invoice_line_taxes(self, invoice, line_vals, tax_ids, tax_nodes, is_withheld, is_purchase):
        logs = []
        for tax_node in tax_nodes:
            tax_rate = find_xml_value('.//TaxRate', tax_node)
            if tax_rate:
                # Since the 'TaxRate' node isn't guaranteed to be a percentage, we can find out by
                # applying the tax rate on the taxable base, and if it's equal to the tax amount
                # then we can say this is a percentage, otherwise a fixed amount.
                taxable_base = find_xml_value('.//TaxableBase/TotalAmount', tax_node)
                tax_amount = find_xml_value('.//TaxAmount/TotalAmount', tax_node)
                is_fixed = False

                if taxable_base and tax_amount and invoice.currency_id.compare_amounts(float(taxable_base) * (float(tax_rate) / 100), float(tax_amount)) != 0:
                    is_fixed = True

                tax_excl = self._search_tax_for_import(invoice.company_id, float(tax_rate), is_fixed, is_withheld, is_purchase, price_included=False)

                if tax_excl:
                    tax_ids.append(tax_excl.id)
                elif tax_incl := self._search_tax_for_import(invoice.company_id, float(tax_rate), is_fixed, is_withheld, is_purchase, price_included=True):
                    tax_ids.append(tax_incl)
                    line_vals['price_unit'] *= (1.0 + float(tax_rate) / 100.0)
                else:
                    logs.append(_("Could not retrieve the tax: %s %% for line '%s'.", tax_rate, line_vals.get('name', "")))

        return logs

    def _search_tax_for_import(self, company, amount, is_fixed, is_withheld, is_purchase, price_included):
        taxes = self.env['account.tax'].search([
            ('company_id', '=', company.id),
            ('amount', '=', -1.0 * amount if is_withheld else amount),
            ('amount_type', '=', 'fixed' if is_fixed else 'percent'),
            ('type_tax_use', '=', 'purchase' if is_purchase else 'sale'),
            ('price_include', '=', price_included),
        ], limit=1)

        return taxes

    def _search_product_for_import(self, item_description):
        # Exported Odoo XML will have item_description = "[default_code] name".
        # We can check if it follows the same format and search for the product with the default code and the name.
        code_and_name = re.match(r"(\[(?P<default_code>.*?)\]\s)?(?P<name>.*)", item_description).groupdict()
        product = self.env['product.product']._retrieve_product(**code_and_name)
        return product

    def _generate_pdf_and_send_invoice(self, template, force_synchronous=True, allow_fallback_pdf=True, bypass_download=False, **kwargs):
        if self.company_id.country_code == "ES" and not self.company_id.l10n_es_edi_facturae_certificate_id:
            kwargs['l10n_es_edi_facturae_checkbox_xml'] = False
        return super()._generate_pdf_and_send_invoice(template, force_synchronous, allow_fallback_pdf, bypass_download, **kwargs)

```

## File: models\account_tax.py

```python
from odoo import fields, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_es_edi_facturae_tax_type = fields.Selection([
        ('01', 'Value-Added Tax'),
        ('02', 'Taxes on production, services and imports in Ceuta and Melilla'),
        ('03', 'IGIC: Canaries General Indirect Tax'),
        ('04', 'IRPF: Personal Income Tax'),
        ('05', 'Other'),
        ('06', 'ITPAJD: Tax on wealth transfers and stamp duty'),
        ('07', 'IE: Excise duties and consumption taxes'),
        ('08', 'RA: Customs duties'),
        ('09', 'IGTECM: Sales tax in Ceuta and Melilla'),
        ('10', 'IECDPCAC: Excise duties on oil derivates in Canaries'),
        ('11', 'IIIMAB: Tax on premises that affect the environment in the Balearic Islands'),
        ('12', 'ICIO: Tax on construction, installation and works'),
        ('13', 'IMVDN: Local tax on unoccupied homes in Navarre'),
        ('14', 'IMSN: Local tax on building plots in Navarre'),
        ('15', 'IMGSN: Local sumptuary tax in Navarre'),
        ('16', 'IMPN: Local tax on advertising in Navarre'),
        ('17', 'REIVA: Special VAT for travel agencies'),
        ('18', 'REIGIC: Special IGIC: for travel agencies'),
        ('19', 'REIPSI: Special IPSI for travel agencies'),
        ('20', 'IPS: Insurance premiums Tax'),
        ('21', 'SWUA: Surcharge for Winding Up Activity'),
        ('22', 'IVPEE: Tax on the value of electricity generation'),
        ('23', 'Tax on the production of spent nuclear fuel and radioactive waste from the generation of nuclear electric power'),
        ('24', 'Tax on the storage of spent nuclear energy and radioactive waste in centralised facilities'),
        ('25', 'IDEC: Tax on bank deposits'),
        ('26', 'Excise duty applied to manufactured tobacco in Canaries'),
        ('27', 'IGFEI: Tax on Fluorinated Greenhouse Gases'),
        ('28', 'IRNR: Non-resident Income Tax'),
        ('29', 'Corporation Tax'),
    ], string='Spanish Facturae EDI Tax Type', default='01')

```

## File: models\l10n_es_edi_facturae_certificate.py

```python
from base64 import b64decode, b64encode, encodebytes
from copy import deepcopy
from hashlib import sha1

from cryptography.hazmat.primitives import hashes, serialization
from lxml import etree

from odoo import _, api, fields, models
from odoo.addons.account.tools.certificate import load_key_and_certificates
from odoo.addons.l10n_es_edi_facturae import xml_utils
from odoo.exceptions import UserError
from odoo.tools import cleanup_xml_node

class Certificate(models.Model):
    _name = 'l10n_es_edi_facturae.certificate'
    _description = 'Facturae Digital Certificate'
    _order = 'date_start desc, id desc'
    _rec_name = 'serial_number'

    content = fields.Binary(string="PFX Certificate", required=True, help="PFX Certificate")
    password = fields.Char(help="Passphrase for the PFX certificate")
    serial_number = fields.Char(readonly=True, index=True, help="The serial number to add to electronic documents")
    date_start = fields.Datetime(readonly=True, help="The date on which the certificate starts to be valid")
    date_end = fields.Datetime(readonly=True, help="The date on which the certificate expires")
    company_id = fields.Many2one(comodel_name='res.company', default=lambda self: self.env.company, required=True, readonly=True)

    def _decode_certificate(self):
        """
        Return certificate data

        :return tuple: private_key, certificate
        """
        self.ensure_one()
        content, password = b64decode(self.with_context(bin_size=False).content), self.password.encode() if self.password else None
        return load_key_and_certificates(content, password)

    # -------------------------------------------------------------------------
    # LOW-LEVEL METHODS
    # -------------------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        certificates = super().create(vals_list)
        for certificate in certificates:
            try:
                _key, certif = certificate._decode_certificate()
            except ValueError:
                raise UserError(_('There has been a problem with the certificate, some usual problems can be:\n'
                                  '\t- The password given or the certificate are not valid.\n'
                                  '\t- The certificate content is invalid.'))
            if fields.datetime.now() > certif.not_valid_after:
                raise UserError(_('The certificate is expired since %s', certif.not_valid_after))
            # Assign extracted values from the certificate
            certificate.write({'serial_number': certif.serial_number, 'date_start': certif.not_valid_before, 'date_end': certif.not_valid_after})
        return certificates

    # -------------------------------------------------------------------------
    # BUSINESS METHODS                                                        #
    # -------------------------------------------------------------------------
    def _sign_xml(self, edi_data, signature_data):
        """
        Signs the given XML data with the certificate and private key.

        :param etree._Element edi_data: The XML data to sign.
        :param dict signature_data: The signature data to use.
        :return: The signed XML data string.
        :rtype: str
        """
        self.ensure_one()
        if not (self.date_start < fields.Datetime.now() < self.date_end):
            raise UserError('Facturae certificate date is not valid, its validity has probably expired')
        root = deepcopy(edi_data)
        cert_private, cert_public = self._decode_certificate()
        public_key_numbers = cert_public.public_key().public_numbers()

        rfc4514_attr = dict(element.rfc4514_string().split("=", 1) for element in cert_public.issuer.rdns)

        # The 'Organizational Unit' field is optional
        issuer = f"CN={rfc4514_attr.pop('CN')}, "
        if 'OU' in rfc4514_attr:
            issuer += f"OU={rfc4514_attr.pop('OU')}, "
        issuer += f"O={rfc4514_attr.pop('O')}, C={rfc4514_attr.pop('C')}"

        # Add remaining certificate fields (not all certificates have other fields)
        issuer += "".join([f", {key}={value}" for key, value in rfc4514_attr.items()])

        # Identifiers
        document_id = f"Document-{sha1(etree.tostring(edi_data)).hexdigest()}"
        signature_id = f"Signature-{document_id}"
        keyinfo_id = f"KeyInfo-{document_id}"
        sigproperties_id = f"SignatureProperties-{document_id}"

        signature_data.update({
            'document_id': document_id,
            'x509_certificate': encodebytes(cert_public.public_bytes(encoding=serialization.Encoding.DER)).decode(),
            'public_modulus': encodebytes(xml_utils._int_to_bytes(public_key_numbers.n)).decode(),
            'public_exponent': encodebytes(xml_utils._int_to_bytes(public_key_numbers.e)).decode(),
            'iso_now': fields.datetime.now().isoformat(),
            'keyinfo_id': keyinfo_id,
            'signature_id': signature_id,
            'sigproperties_id': sigproperties_id,
            'reference_uri': "Reference-" + document_id,
            'sigpolicy_url': "http://www.facturae.es/politica_de_firma_formato_facturae/politica_de_firma_formato_facturae_v3_1.pdf",
            'sigpolicy_description': "Política de firma electrónica para facturación electrónica con formato Facturae",
            'sigcertif_digest': b64encode(cert_public.fingerprint(hashes.SHA256())).decode(),
            'x509_issuer_description': issuer,
            'x509_serial_number': cert_public.serial_number,
        })
        signature = self.env['ir.qweb']._render('l10n_es_edi_facturae.template_xades_signature', signature_data)
        signature = cleanup_xml_node(signature, remove_blank_nodes=False)
        root.append(signature)
        xml_utils._reference_digests(signature.find("ds:SignedInfo", namespaces=xml_utils.NS_MAP))
        xml_utils._fill_signature(signature, cert_private)

        return etree.tostring(root, xml_declaration=True, encoding='UTF-8', standalone=True)

```

## File: models\res_company.py

```python
from odoo import fields, models


class Company(models.Model):
    _inherit = 'res.company'

    l10n_es_edi_facturae_residence_type = fields.Char(string='Facturae EDI Residency Type Code', related='partner_id.l10n_es_edi_facturae_residence_type')
    l10n_es_edi_facturae_certificate_id = fields.One2many(string='Facturae EDI signing certificate',
        comodel_name='l10n_es_edi_facturae.certificate', inverse_name='company_id')

```

## File: models\res_partner.py

```python
from odoo import api, fields, models


class Partner(models.Model):
    _inherit = 'res.partner'

    l10n_es_edi_facturae_residence_type = fields.Char(string='Facturae EDI Residency Type Code',
        compute='_compute_l10n_es_edi_facturae_residence_type', store=False, readonly=True,)

    @api.depends('country_id')
    def _compute_l10n_es_edi_facturae_residence_type(self):
        eu_country_ids = self.env.ref('base.europe').country_ids.ids
        for partner in self:
            country = partner.country_id
            if country.code == 'ES':
                partner.l10n_es_edi_facturae_residence_type = 'R'
            elif country.id in eu_country_ids:
                partner.l10n_es_edi_facturae_residence_type = 'U'
            else:
                partner.l10n_es_edi_facturae_residence_type = 'E'

```

## File: models\uom_uom.py

```python
from odoo import fields, models


class UoM(models.Model):
    _inherit = 'uom.uom'

    l10n_es_edi_facturae_uom_code = fields.Selection(
        selection=[
            ('01', 'Units'),
            ('02', 'Hours'),
            ('03', 'Kilograms'),
            ('04', 'Liters'),
            ('05', 'Other'),
            ('06', 'Boxes'),
            ('07', 'Trays, one layer no cover, plastic'),
            ('08', 'Barrels'),
            ('09', 'Jerricans, cylindrical'),
            ('10', 'Bags'),
            ('11', 'Carboys, non-protected'),
            ('12', 'Bottles, non-protected, cylindrical'),
            ('13', 'Canisters'),
            ('14', 'Tetra Briks'),
            ('15', 'Centiliters'),
            ('16', 'Centimeters'),
            ('17', 'Bins'),
            ('18', 'Dozens'),
            ('19', 'Cases'),
            ('20', 'Demijohns, non-protected'),
            ('21', 'Grams'),
            ('22', 'Kilometers'),
            ('23', 'Cans, rectangular'),
            ('24', 'Bunches'),
            ('25', 'Meters'),
            ('26', 'Millimeters'),
            ('27', '6-Packs'),
            ('28', 'Packages'),
            ('29', 'Portions'),
            ('30', 'Rolls'),
            ('31', 'Envelopes'),
            ('32', 'Tubs'),
            ('33', 'Cubic meter'),
            ('34', 'Second'),
            ('35', 'Watt'),
            ('36', 'Kilowatt-hour')
    ], string='Spanish EDI Units', default="05", required=True)

```

## File: models\__init__.py

```python
from . import account_tax
from . import l10n_es_edi_facturae_certificate
from . import res_company
from . import res_partner
from . import uom_uom
from . import account_move
from . import account_chart_template

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
l10n_es_edi_facturae.access_l10n_es_edi_facturae_certificate_manager,access_l10n_es_edi_facturae_certificate,l10n_es_edi_facturae.model_l10n_es_edi_facturae_certificate,account.group_account_manager,1,0,0,0
l10n_es_edi_facturae.access_l10n_es_edi_facturae_certificate_system,access_l10n_es_edi_facturae_certificate,l10n_es_edi_facturae.model_l10n_es_edi_facturae_certificate,base.group_system,1,1,1,1

```

## File: security\l10n_es_edi_certificate.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Allow read certificates of my company-->
    <record id="facturae_digital_certificate" model="ir.rule">
        <field name="name">Facturae Digital Certificate</field>
        <field name="model_id" ref="l10n_es_edi_facturae.model_l10n_es_edi_facturae_certificate"/>
        <field name="groups" eval="[Command.link(ref('account.group_account_user')), Command.link(ref('account.group_account_manager'))]"/>
        <field name="domain_force">[('company_id', 'in', user.company_ids.ids)]</field>
    </record>
</odoo>

```

## File: views\account_menuitem.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <menuitem id="menu_l10n_es_edi_facturae_root" name="Spain Facturae EDI" sequence="110" groups="base.group_system"
                  parent="account.menu_finance_configuration">
            <menuitem id="menu_l10n_es_edi_facturae_root_certificates" name="Certificates" action="l10n_es_edi_facturae_certificate_action"
                  sequence="100" groups="base.group_system"/>
        </menuitem>
    </data>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_tax_tree_inherit_l10n_es_edi_facturae" model="ir.ui.view">
            <field name="name">account.tax.tree.inherit.l10n_es_edi_facturae</field>
            <field name="inherit_id" ref="account.view_tax_tree"/>
            <field name="model">account.tax</field>
            <field name="arch" type="xml">
                <field name="country_id" position="after">
                    <field name="country_code" column_invisible="True"/>
                    <field name="l10n_es_edi_facturae_tax_type" string="Spanish Tax Type" optional="hide"
                        invisible="country_code != 'ES'"/>
                </field>
            </field>
        </record>

        <record id="view_tax_form_inherit_l10n_es_edi_facturae" model="ir.ui.view">
            <field name="name">account.tax.form.inherit.l10n_es_edi_facturae</field>
            <field name="model">account.tax</field>
            <field name="inherit_id" ref="account.view_tax_form"/>
            <field name="arch" type="xml">
                <field name="country_id" position="after">
                    <field name="l10n_es_edi_facturae_tax_type" invisible="country_code != 'ES'"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\l10n_es_edi_facturae_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="l10n_es_edi_facturae_certificate_form" model="ir.ui.view">
            <field name="name">l10n_es_edi_facturae.certificate.form</field>
            <field name="model">l10n_es_edi_facturae.certificate</field>
            <field name="arch" type="xml">
                <form>
                    <sheet>
                        <group>
                            <field name="content"/>
                            <field name="password" password="True"/>
                            <label for="date_start" string="Validity"/>
                            <div>
                                <field name="date_start"/> -
                                <field name="date_end"/>
                            </div>
                            <field name="company_id" groups="base.group_multi_company" readonly="True"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="l10n_es_edi_facturae_certificate_tree" model="ir.ui.view">
            <field name="name">l10n_es_edi_facturae.certificate.tree</field>
            <field name="model">l10n_es_edi_facturae.certificate</field>
            <field name="arch" type="xml">
                <tree>
                    <field name="date_start"/>
                    <field name="date_end"/>
                    <field name="company_id" groups="base.group_multi_company" readonly="True"/>
                </tree>
            </field>
        </record>

        <record id="l10n_es_edi_facturae_certificate_action" model="ir.actions.act_window">
            <field name="name">Certificates for Facturae EDI invoices on Spain</field>
            <field name="res_model">l10n_es_edi_facturae.certificate</field>
            <field name="view_mode">tree,form</field>
            <field name="context" eval="{'search_default_my_certificates': 1}"/>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">Create the first certificate</p>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_company_form_inherit_l10n_es_edi_facturae" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_es_edi_facturae</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="model">res.company</field>
        <field name="arch" type="xml">
            <field name="currency_id" position="after">
                <field name="l10n_es_edi_facturae_certificate_id" string="Facturae signature certificate"
                       invisible="country_code != 'ES'"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\uom_uom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_uom_tree_view_inherit_l10n_es_edi_facturae" model="ir.ui.view">
        <field name="name">uom.uom.tree.inherit.l10n_es_edi_facturae</field>
        <field name="model">uom.uom</field>
        <field name="inherit_id" ref="uom.product_uom_tree_view"/>
        <field name="arch" type="xml">
            <field name="category_id" position="after">
                <field name="l10n_es_edi_facturae_uom_code" string="Spanish Facturae EDI type"/>
            </field>
        </field>
    </record>
    <record id="product_uom_form_view_inherit_l10n_es_edi_facturae" model="ir.ui.view">
        <field name="name">uom.uom.form.inherit.l10n_es_edi_facturae</field>
        <field name="model">uom.uom</field>
        <field name="inherit_id" ref="uom.product_uom_form_view"/>
        <field name="arch" type="xml">
            <field name="category_id" position="after">
                <field name="l10n_es_edi_facturae_uom_code" string="Spanish Facturae EDI type"/>
            </field>
        </field>
    </record>
    <record id="product_uom_categ_form_view_inherit_l10n_es_edi_facturae" model="ir.ui.view">
        <field name="name">uom.category.form.inherit.l10n_es_edi_facturae</field>
        <field name="model">uom.category</field>
        <field name="inherit_id" ref="uom.product_uom_categ_form_view"/>
        <field name="arch" type="xml">
            <tree position="inside">
                <field name="l10n_es_edi_facturae_uom_code" string="Spanish Facturae EDI type"/>
            </tree>
        </field>
    </record>
</odoo>

```

## File: wizard\account_move_reversal.py

```python
from odoo import models, fields


class AccountMoveReversal(models.TransientModel):
    _inherit = 'account.move.reversal'

    l10n_es_edi_facturae_reason_code = fields.Selection(
        selection=lambda self: self.env['account.move']._fields['l10n_es_edi_facturae_reason_code']._description_selection(self.env),
        string='Spanish Facturae EDI Reason Code',
        default='10'
    )

    def reverse_moves(self, is_modify=False):
        # Extends account_account
        res = super(AccountMoveReversal, self).reverse_moves(is_modify)
        self.new_move_ids.l10n_es_edi_facturae_reason_code = self.l10n_es_edi_facturae_reason_code
        return res

```

## File: wizard\account_move_reversal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_tax_tree_inherit_l10n_es_edi_facturae" model="ir.ui.view">
            <field name="name">account.tax.tree.inherit.l10n_es_edi_facturae</field>
            <field name="inherit_id" ref="account.view_tax_tree"/>
            <field name="model">account.tax</field>
            <field name="arch" type="xml">
                <field name="country_id" position="after">
                    <field name="country_code" column_invisible="True"/>
                    <field name="l10n_es_edi_facturae_tax_type" string="Spanish Tax Type" optional="hide"
                           invisible="country_code != 'ES'"/>
                </field>
            </field>
        </record>
        <record id="view_account_move_reversal_inherit_l10n_es_edi_facturae" model="ir.ui.view">
            <field name="name">account.move.reversal.form.inherit.l10n_es_edi_facturae</field>
            <field name="inherit_id" ref="account.view_account_move_reversal"/>
            <field name="model">account.move.reversal</field>
            <field name="arch" type="xml">
                <field name="reason" position="replace">
                    <field name="country_code" invisible="1"/>
                    <field name="l10n_es_edi_facturae_reason_code" string="Reason" invisible="country_code != 'ES'"/>
                    <field name="reason" string="Reason" invisible="move_type == 'entry' and country_code == 'ES'"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\account_move_send.py

```python
import logging

from odoo import _, api, fields, models, SUPERUSER_ID
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class AccountMoveSend(models.TransientModel):
    _inherit = 'account.move.send'

    l10n_es_edi_facturae_enable_xml = fields.Boolean(compute='_compute_l10n_es_edi_facturae_enable_xml')
    l10n_es_edi_facturae_checkbox_xml = fields.Boolean(
        string="Generate Facturae edi file",
        default=True,
        company_dependent=True,
    )

    def _get_wizard_values(self):
        # EXTENDS 'account'
        values = super()._get_wizard_values()
        values['l10n_es_edi_facturae_xml'] = self.l10n_es_edi_facturae_checkbox_xml
        return values

    @api.model
    def _get_wizard_vals_restrict_to(self, only_options):
        # EXTENDS 'account'
        values = super()._get_wizard_vals_restrict_to(only_options)
        return {
            'l10n_es_edi_facturae_checkbox_xml': False,
            **values,
        }

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('move_ids')
    def _compute_l10n_es_edi_facturae_enable_xml(self):
        for wizard in self:
            wizard.l10n_es_edi_facturae_enable_xml = any(move._l10n_es_edi_facturae_get_default_enable() for move in wizard.move_ids)

    @api.depends('l10n_es_edi_facturae_enable_xml')
    def _compute_l10n_es_edi_facturae_checkbox_xml(self):
        for wizard in self:
            wizard.l10n_es_edi_facturae_checkbox_xml = wizard.l10n_es_edi_facturae_enable_xml

    @api.depends('l10n_es_edi_facturae_checkbox_xml')
    def _compute_mail_attachments_widget(self):
        # EXTENDS 'account' - add depends
        super()._compute_mail_attachments_widget()

    # -------------------------------------------------------------------------
    # ATTACHMENTS
    # -------------------------------------------------------------------------

    @api.model
    def _get_invoice_extra_attachments(self, move):
        # EXTENDS 'account'
        return super()._get_invoice_extra_attachments(move) + move.l10n_es_edi_facturae_xml_id

    def _get_placeholder_mail_attachments_data(self, move):
        # EXTENDS 'account'
        results = super()._get_placeholder_mail_attachments_data(move)

        if self.mode == 'invoice_single' and self.l10n_es_edi_facturae_enable_xml and self.l10n_es_edi_facturae_checkbox_xml:
            filename = f'{move.name.replace("/", "_")}_facturae_signed.xml'
            results.append({
                'id': f'placeholder_{filename}',
                'name': filename,
                'mimetype': 'application/xml',
                'placeholder': True,
            })

        return results

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    @api.model
    def _hook_invoice_document_before_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_before_pdf_report_render(invoice, invoice_data)

        if invoice_data.get('l10n_es_edi_facturae_xml') and invoice._l10n_es_edi_facturae_get_default_enable():
            try:
                xml_content, errors = invoice._l10n_es_edi_facturae_render_facturae()
                if errors:
                    invoice_data['error'] = {
                        'error_title': _("Errors occurred while creating the EDI document (format: %s):", "Facturae"),
                        'errors': errors,
                    }
                else:
                    invoice_data['l10n_es_edi_facturae_attachment_values'] = {
                        'name': invoice._l10n_es_edi_facturae_get_filename(),
                        'raw': xml_content,
                        'mimetype': 'application/xml',
                        'res_model': invoice._name,
                        'res_id': invoice.id,
                        'res_field': 'l10n_es_edi_facturae_xml_file',  # Binary field
                    }
            except UserError as e:
                if self.env.context.get('forced_invoice'):
                    _logger.warning(
                        'An error occured during generation of Facturae EDI of %s: %s',
                        invoice.name,
                        e.args[0]
                    )
                else:
                    raise

    @api.model
    def _link_invoice_documents(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._link_invoice_documents(invoice, invoice_data)

        attachment_vals = invoice_data.get('l10n_es_edi_facturae_attachment_values')
        if attachment_vals:
            self.env['ir.attachment'].with_user(SUPERUSER_ID).create(attachment_vals)
            invoice.invalidate_recordset(fnames=['l10n_es_edi_facturae_xml_id', 'l10n_es_edi_facturae_xml_file'])

```

## File: wizard\account_move_send_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_move_send_form" model="ir.ui.view">
        <field name="name">account.move.send.form</field>
        <field name="model">account.move.send</field>
        <field name="inherit_id" ref="account.account_move_send_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='advanced_options']" position="inside">
                <field name="l10n_es_edi_facturae_enable_xml" invisible="1"/>
                <div name="option_xml"
                     invisible="not l10n_es_edi_facturae_enable_xml">
                    <!-- Use one field as the label for another -->
                    <field name="l10n_es_edi_facturae_checkbox_xml" style="display: inline-block"/>
                    <b><label for="l10n_es_edi_facturae_checkbox_xml"></label></b>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
from . import account_move_reversal
from . import account_move_send

```

