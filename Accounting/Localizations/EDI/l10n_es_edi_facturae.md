# Odoo Module: l10n_es_edi_facturae

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: xml_utils.py

```python
import base64
import hashlib
from copy import deepcopy
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
        reference.find("ds:DigestValue", namespaces=NS_MAP).text = base64.b64encode(lib.digest())

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
        'certificate',
        'l10n_es',
    ],
    'data': [
        'data/uom.uom.csv',
        'data/facturae_templates.xml',
        'data/l10n_es_edi_facturae.ac_role_type.csv',
        'data/signature_templates.xml',

        'security/ir.model.access.csv',

        'views/l10n_es_edi_facturae_views.xml',
        'views/res_partner_views.xml',
        'views/account_tax_views.xml',
        'views/account_move_views.xml',
        'views/uom_uom_views.xml',
        'views/account_menuitem.xml',

        'wizard/account_move_reversal_view.xml',
    ],
    'demo': [
        'demo/l10n_es_edi_facturae_demo.xml',
    ],
    'post_init_hook': '_l10n_es_edi_facturae_post_init_hook',
    'installable': True,
    'auto_install': ['l10n_es'],
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

        <!-- Sub-template used for every instance of AdministrativeCentresType -->
        <template id="administrative_centers_type">
            <AdministrativeCentres>
                <AdministrativeCentre t-foreach="administrative_centers" t-as="ac">
                    <CentreCode t-out="(ac.get('center_code') or '')[:10]"/>
                    <RoleTypeCode t-out="ac.get('role_type_code')"/>
                    <Name t-out="(ac.get('name') or '')[:40]"/>
                    <t t-call="l10n_es_edi_facturae.address_type">
                        <t t-set="partner" t-value="ac.get('partner')"/>
                        <t t-set="partner_country_code" t-value="ac.get('partner_country_code')"/>
                    </t>
                    <t t-call="l10n_es_edi_facturae.contact_details_type">
                        <t t-set="partner" t-value="ac.get('partner')"/>
                        <t t-set="partner_phone" t-value="ac.get('partner_phone')"/>
                    </t>
                    <PhysicalGLN t-out="(ac.get('physical_gln') or '')[:14]"/>
                    <LogicalOperationalPoint t-out="(ac.get('logical_operational_point') or '')[:14]"/>
                </AdministrativeCentre>
            </AdministrativeCentres>
        </template>

        <!-- Sub-template used for every instance of BusinessType -->
        <template id="business_type">
            <t t-call="l10n_es_edi_facturae.tax_identification_type"/>
            <t t-call="l10n_es_edi_facturae.administrative_centers_type"/>
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
                            <t t-set="administrative_centers" t-value="self_party_administrative_centers if is_outstanding else other_party_administrative_centers"/>
                        </t>
                    </SellerParty>
                    <BuyerParty>
                        <t t-call="l10n_es_edi_facturae.business_type">
                            <t t-set="partner" t-value="other_party if is_outstanding else self_party"/>
                            <t t-set="partner_country_code" t-value="other_party_country_code if is_outstanding else self_party_country_code"/>
                            <t t-set="partner_phone" t-value="other_party_phone if is_outstanding else False"/>
                            <t t-set="partner_name" t-value="other_party_name if is_outstanding else self_party_name"/>
                            <t t-set="administrative_centers" t-value="other_party_administrative_centers if is_outstanding else self_party_administrative_centers"/>
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

## File: data\l10n_es_edi_facturae.ac_role_type.csv

```csv
"id","code","name"
"ac_role_type_01","01","Fiscal"
"ac_role_type_02","02","Receiver"
"ac_role_type_03","03","Payer"
"ac_role_type_04","04","Buyer"
"ac_role_type_05","05","Collector"
"ac_role_type_06","06","Seller"
"ac_role_type_07","07","Payment Receiver"
"ac_role_type_08","08","Collection Receiver"
"ac_role_type_09","09","Issuer"

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

## File: data\template\account.tax-es_common_mainland.csv

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
import base64
import re
from collections import defaultdict
from copy import deepcopy
from hashlib import sha1

from lxml import etree
from markupsafe import Markup

from odoo import Command, _, api, fields, models
from odoo.exceptions import UserError
from odoo.tools import float_round, float_repr, date_utils, SQL
from odoo.tools.xml_utils import cleanup_xml_node, find_xml_value
from odoo.addons.l10n_es_edi_facturae.xml_utils import (
    NS_MAP,
    _canonicalize_node,
    _reference_digests,
)

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
    l10n_es_invoicing_period_start_date = fields.Date(string="Invoice Period Start Date")
    l10n_es_invoicing_period_end_date = fields.Date(string="Invoice Period End Date")
    l10n_es_payment_means = fields.Selection(
        selection=[
            ('01', "In cash"),
            ('02', "Direct debit"),
            ('03', "Receipt"),
            ('04', "Credit transfer"),
            ('05', "Accepted bill of exchange"),
            ('06', "Documentary credit"),
            ('07', "Contract award"),
            ('08', "Bill of exchange"),
            ('09', "Transferable promissory note"),
            ('10', "Non transferable promissory note"),
            ('11', "Cheque"),
            ('12', "Open account reimbursement"),
            ('13', "Special payment"),
            ('14', "Set-off by reciprocal credits"),
            ('15', "Payment by postgiro"),
            ('16', "Certified cheque"),
            ('17', "Banker’s draft"),
            ('18', "Cash on delivery"),
            ('19', "Payment by card"),
        ], string="Payment Means", default='04')

    def _get_fields_to_detach(self):
        # EXTENDS account
        fields_list = super()._get_fields_to_detach()
        fields_list.append("l10n_es_edi_facturae_xml_file")
        return fields_list

    def _l10n_es_edi_facturae_get_default_enable(self):
        self.ensure_one()
        return not self.invoice_pdf_report_id \
            and not self.l10n_es_edi_facturae_xml_id \
            and not self.l10n_es_is_simplified \
            and self.is_invoice(include_receipts=True) \
            and (self.partner_id.is_company or self.partner_id.vat) \
            and self.company_id.country_code == 'ES' \
            and self.company_id.currency_id.name == 'EUR' \
            and self.company_id.sudo().l10n_es_edi_facturae_certificate_ids  # We only enable Facturae if a certificate is valid or has been valid (which will raise an error)

    def _l10n_es_edi_facturae_get_filename(self):
        self.ensure_one()
        return f'{self.name.replace("/", "_")}_facturae_signed.xml'

    def _l10n_es_edi_facturae_get_tax_period(self):
        self.ensure_one()
        if self.env['res.company'].fields_get(['account_tax_periodicity']):
            period_start, period_end = self.company_id._get_tax_closing_period_boundaries(self.date, self.env.ref('l10n_es.mod_303'))
        else:
            period_start = date_utils.start_of(self.date, 'month')
            period_end = date_utils.end_of(self.date, 'month')

        return {'start':period_start, 'end':period_end}

    def _l10n_es_edi_facturae_get_refunded_invoices(self):
        self.env['account.partial.reconcile'].flush_model()
        invoices_refunded_mapping = {invoice.id: invoice.reversed_entry_id.id for invoice in self}

        stored_ids = tuple(self.ids)
        queries = []
        for source_field, counterpart_field in (
            ('debit_move_id', 'credit_move_id'),
            ('credit_move_id', 'debit_move_id'),
        ):
            queries.append(SQL('''
                SELECT
                    source_line.move_id AS source_move_id,
                    counterpart_line.move_id AS counterpart_move_id
                FROM account_partial_reconcile part
                JOIN account_move_line source_line ON source_line.id = part.%s
                JOIN account_move_line counterpart_line ON counterpart_line.id = part.%s
                WHERE source_line.move_id IN %s AND counterpart_line.move_id != source_line.move_id
                GROUP BY source_move_id, counterpart_move_id
            ''', SQL.identifier(source_field), SQL.identifier(counterpart_field), stored_ids))
        payment_data = defaultdict(list)
        for row in self.env.execute_query_dict(SQL(" UNION ALL ").join(queries)):
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

    def _l10n_es_edi_facturae_get_administrative_centers(self, partner):
        self.ensure_one()
        administrative_centers = []
        for ac in partner.child_ids.filtered(lambda p: p.type == 'facturae_ac'):
            ac_template = {
                'center_code': ac.l10n_es_edi_facturae_ac_center_code,
                'name': ac.name,
                'partner': ac,
                'partner_country_code': COUNTRY_CODE_MAP[ac.country_code],
                'partner_phone': ac.phone.translate(PHONE_CLEAN_TABLE) if ac.phone else False,
                'physical_gln': ac.l10n_es_edi_facturae_ac_physical_gln,
                'logical_operational_point': ac.l10n_es_edi_facturae_ac_logical_operational_point,
            }
            # An administrative center can have multiple roles, each of which should be reported separately.
            for role in ac.l10n_es_edi_facturae_ac_role_type_ids or [self.env['l10n_es_edi_facturae.ac_role_type']]:
                administrative_centers.append({
                    **ac_template,
                    'role_type_code': role.code,
                })
        return administrative_centers

    def _l10n_es_edi_facturae_get_tax_node_from_tax_data(self, values):
        self.ensure_one()
        tax = values['grouping_key']
        return {
            'tax_record': tax,
            'TaxRate': f'{abs(tax.amount):.3f}',
            'TaxableBase': {
                'TotalAmount': self.currency_id.round(values['raw_base_amount_currency']),
                'EquivalentInEuros': self.company_currency_id.round(values['raw_base_amount']),
            },
            'TaxAmount': {
                'TotalAmount': self.currency_id.round(abs(values['raw_tax_amount_currency'])),
                'EquivalentInEuros': self.company_currency_id.round(abs(values['raw_tax_amount'])),
            },
        }

    def _l10n_es_edi_facturae_convert_payment_terms_to_installments(self):
        """
        Convert the payments terms to a list of <Installment> elements to be used in the
        <PaymentDetails> node of the Facturae XML generation.
        """
        self.ensure_one()
        installments = []
        if self.is_inbound() and self.partner_bank_id:
            for payment_term in self.line_ids.filtered(lambda l: l.display_type == 'payment_term').sorted('date_maturity'):
                installments.append({
                    'InstallmentDueDate': payment_term.date_maturity,
                    'InstallmentAmount': payment_term.amount_residual_currency,
                    'PaymentMeans': self.l10n_es_payment_means,
                    'AccountToBeCredited': {
                        'IBAN': self.partner_bank_id.sanitized_acc_number,
                        'BIC': self.partner_bank_id.bank_bic,
                    },
                })
        return installments

    def _l10n_es_edi_facturae_prepare_inv_line(self, base_line, aggregated_values):
        """
        Convert the invoice lines to a list of items required for the Facturae xml generation

        :return: A tuple containing the Face items, the taxes and the invoice totals data.
        """
        self.ensure_one()
        extended_dp = 6 if self.company_id.tax_calculation_rounding_method == 'round_globally' else 2
        invoice_ref = self.ref and self.ref[:20]
        line = base_line['record']
        tax_details = base_line['tax_details']

        receiver_transaction_reference = (
            line.sale_line_ids.order_id.client_order_ref[:20]
            if 'sale_line_ids' in line._fields and line.sale_line_ids.order_id.client_order_ref
            else invoice_ref
        )

        xml_values = {
            'ReceiverTransactionReference': receiver_transaction_reference,
            'FileReference': invoice_ref,
            'ReceiverContractReference': invoice_ref,
            'FileDate': fields.Date.context_today(self),
            'ItemDescription': line.name,
            'Quantity': line.quantity,
            'UnitOfMeasure': line.product_uom_id.l10n_es_edi_facturae_uom_code,
            'DiscountsAndRebates': [],
            'Charges': [],
            'GrossAmount': line.price_subtotal,
        }

        if line.discount == 100.0:
            raw_total_cost = line.price_unit * line.quantity
        else:
            raw_total_cost = tax_details['total_excluded_currency'] / (1 - (line.discount / 100.0))
        xml_values['TotalCost'] = line.currency_id.round(raw_total_cost)

        if line.quantity:
            xml_values['UnitPriceWithoutTax'] = float_round(raw_total_cost / line.quantity, precision_digits=extended_dp)
        else:
            xml_values['UnitPriceWithoutTax'] = 0.0

        raw_discount_amount = xml_values['TotalCost'] - line.price_subtotal
        discount_amount = max(raw_discount_amount, 0.0)
        if discount_amount:
            xml_values['DiscountsAndRebates'].append({
                'DiscountReason': '/',
                'DiscountRate': f'{line.discount:.2f}',
                'DiscountAmount': discount_amount,
            })

        surcharge_amount = -min(0.0, raw_discount_amount)
        if surcharge_amount:
            xml_values['Charges'].append({
                'ChargeReason': '/',
                'ChargeRate': f'{-line.discount:.2f}',
                'ChargeAmount': surcharge_amount,
            })

        xml_values['TaxesOutputs'] = [
            self._l10n_es_edi_facturae_get_tax_node_from_tax_data(values)
            for values in aggregated_values.values()
            if values['grouping_key'] and values['grouping_key'].amount >= 0.0
        ]
        xml_values['TaxesWithheld'] = [
            self._l10n_es_edi_facturae_get_tax_node_from_tax_data(values)
            for values in aggregated_values.values()
            if values['grouping_key'] and values['grouping_key'].amount < 0.0
        ]

        return xml_values

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

        # Multi-currencies.
        eur_curr = self.env['res.currency'].search([('name', '=', 'EUR')])
        inv_curr = self.currency_id
        conversion_needed = inv_curr != eur_curr

        # Invoice xml values.
        invoice_ref = self.ref and self.ref[:20]
        legal_literals = self.narration and self.narration.striptags()
        legal_literals = legal_literals.split(";") if legal_literals else False
        invoice_values = {
            'invoice_record': self,
            'invoice_currency': inv_curr,
            'InvoiceDocumentType': 'FC',
            'InvoiceClass': 'OO',
            'Corrective': self._l10n_es_edi_facturae_get_corrective_data(),
            'InvoiceIssueData': {
                'OperationDate': operation_date,
                'ExchangeRateDetails': conversion_needed,
                'ExchangeRate': f"{round(self.invoice_currency_rate, 4):.4f}",
                'LanguageName': self._context.get('lang', 'en_US').split('_')[0],
                'InvoicingPeriod': None,
                'ReceiverTransactionReference': invoice_ref,
                'FileReference': invoice_ref,
                'ReceiverContractReference': invoice_ref,
            },
            'TaxOutputs': [],
            'TaxesWithheld': [],
            'TotalGrossAmount': 0.0,
            'TotalGeneralDiscounts': 0.0,
            'TotalGeneralSurcharges': 0.0,
            'TotalGrossAmountBeforeTaxes': 0.0,
            'TotalTaxOutputs': 0.0,
            'TotalTaxesWithheld': 0.0,
            'PaymentsOnAccount': [],
            'TotalOutstandingAmount': abs(self.amount_total_in_currency_signed),
            'InvoiceTotal': abs(self.amount_total_in_currency_signed),
            'TotalPaymentsOnAccount': 0.0,
            'AmountsWithheld': None,
            'TotalExecutableAmount': abs(self.amount_total_in_currency_signed),
            'Items': [],
            'PaymentDetails': self._l10n_es_edi_facturae_convert_payment_terms_to_installments(),
            'LegalLiterals': legal_literals,
        }

        # Taxes.
        AccountTax = self.env['account.tax']
        base_amls = self.invoice_line_ids.filtered(lambda line: line.display_type == 'product')
        base_lines = [self._prepare_product_base_line_for_taxes_computation(line) for line in base_amls]
        AccountTax._add_tax_details_in_base_lines(base_lines, self.company_id)
        tax_amls = self.line_ids.filtered('tax_repartition_line_id')
        tax_lines = [self._prepare_tax_line_for_taxes_computation(tax_line) for tax_line in tax_amls]
        AccountTax._round_base_lines_tax_details(base_lines, self.company_id, tax_lines=tax_lines)

        def grouping_function(base_line, tax_data):
            return tax_data['tax']

        base_lines_aggregated_values = AccountTax._aggregate_base_lines_tax_details(base_lines, grouping_function)
        for base_line, aggregated_values in base_lines_aggregated_values:
            invoice_line_values = self._l10n_es_edi_facturae_prepare_inv_line(base_line, aggregated_values)
            invoice_values['TotalGrossAmount'] += invoice_line_values['GrossAmount']
            invoice_values['Items'].append(invoice_line_values)

            for values in aggregated_values.values():
                tax = values['grouping_key']
                if not tax:
                    continue

                tax_data = self._l10n_es_edi_facturae_get_tax_node_from_tax_data(values)
                if tax.amount < 0.0:
                    invoice_values['TaxesWithheld'].append(tax_data)
                    invoice_values['TotalTaxesWithheld'] += tax_data['TaxAmount']['TotalAmount']
                else:
                    invoice_values['TaxOutputs'].append(tax_data)
                    invoice_values['TotalTaxOutputs'] += tax_data['TaxAmount']['TotalAmount']

        invoice_values['TotalGrossAmountBeforeTaxes'] = (
            invoice_values['TotalGrossAmount']
            - invoice_values['TotalGeneralDiscounts']
            + invoice_values['TotalGeneralSurcharges']
        )

        template_values = {
            'self_party': company.partner_id,
            'self_party_country_code': COUNTRY_CODE_MAP[company.country_id.code],
            'self_party_name': extract_party_name(company.partner_id),
            'self_party_administrative_centers': self._l10n_es_edi_facturae_get_administrative_centers(company.partner_id),
            'other_party': partner,
            'other_party_country_code': COUNTRY_CODE_MAP[partner.country_id.code],
            'other_party_phone': partner.phone.translate(PHONE_CLEAN_TABLE) if partner.phone else False,
            'other_party_name': extract_party_name(partner),
            'other_party_administrative_centers': self._l10n_es_edi_facturae_get_administrative_centers(partner),
            'is_outstanding': self.move_type.startswith('out_'),
            'float_repr': float_repr,
            'file_currency': inv_curr,
            'eur': eur_curr,
            'conversion_needed': conversion_needed,
            'refund_multiplier': -1 if self.move_type in ('out_refund', 'in_refund') else 1,

            'Modality': 'I',
            'BatchIdentifier': self.name,
            'InvoicesCount': 1,
            'TotalInvoicesAmount': {
                'TotalAmount': abs(self.amount_total_in_currency_signed),
                'EquivalentInEuros': abs(self.amount_total_signed),
            },
            'TotalOutstandingAmount': {
                'TotalAmount': abs(self.amount_total_in_currency_signed),
                'EquivalentInEuros': abs(self.amount_total_signed),
            },
            'TotalExecutableAmount': {
                'TotalAmount': abs(self.amount_total_in_currency_signed),
                'EquivalentInEuros': abs(self.amount_total_signed),
            },
            'InvoiceCurrencyCode': inv_curr.name,
            'Invoices': [invoice_values],
        }
        if self.l10n_es_invoicing_period_start_date and self.l10n_es_invoicing_period_end_date:
            template_values['Invoices'][0]['InvoiceIssueData']['InvoicingPeriod'] = {
                'StartDate': self.l10n_es_invoicing_period_start_date,
                'EndDate': self.l10n_es_invoicing_period_end_date,
            }

        invoice_issuer_signature_type = 'supplier' if self.move_type == 'out_invoice' else 'customer'
        signature_values = {'SigningTime': '', 'SignerRole': invoice_issuer_signature_type}
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

        errors = []
        try:
            xml_content = self._l10n_es_facturae_sign_xml(xml_content, signature_values)
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
        email = partner_vals['email']
        country_code = partner_vals['country_code']

        partner = self.env['res.partner']._retrieve_partner(name=name, vat=vat, phone=phone, email=email)

        if not partner and name:
            partner_vals = {'name': name, 'email': email, 'phone': phone}
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
                    logs.append(_("Could not retrieve the tax: %(tax_rate)s %% for line '%(line)s'.", tax_rate=tax_rate, line=line_vals.get('name', "")))

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

    # -------------------------------------------------------------------------
    # BUSINESS METHODS                                                        #
    # -------------------------------------------------------------------------
    def _l10n_es_facturae_sign_xml(self, edi_data, signature_data):
        """
        Signs the given XML data with the certificate and private key.

        :param etree._Element edi_data: The XML data to sign.
        :param dict signature_data: The signature data to use.
        :return: The signed XML data string.
        :rtype: str
        """
        self.ensure_one()
        certificates_sudo = self.company_id.sudo().l10n_es_edi_facturae_certificate_ids.filtered("is_valid")
        if not certificates_sudo:
            raise UserError(_('No valid certificate found'))

        certificate_sudo = certificates_sudo[0]

        root = deepcopy(edi_data)
        e, n = certificate_sudo._get_public_key_numbers_bytes()
        issuer = certificate_sudo._l10n_es_edi_facturae_get_issuer()

        # Identifiers
        document_id = f"Document-{sha1(etree.tostring(edi_data)).hexdigest()}"
        signature_id = f"Signature-{document_id}"
        keyinfo_id = f"KeyInfo-{document_id}"
        sigproperties_id = f"SignatureProperties-{document_id}"

        signature_data.update({
            'document_id': document_id,
            'x509_certificate': base64.encodebytes(base64.b64decode(certificate_sudo._get_der_certificate_bytes())).decode(),
            'public_modulus': n.decode(),
            'public_exponent': e.decode(),
            'iso_now': fields.datetime.now().isoformat(),
            'keyinfo_id': keyinfo_id,
            'signature_id': signature_id,
            'sigproperties_id': sigproperties_id,
            'reference_uri': f"Reference-{document_id}",
            'sigpolicy_url': "http://www.facturae.es/politica_de_firma_formato_facturae/politica_de_firma_formato_facturae_v3_1.pdf",
            'sigpolicy_description': "Política de firma electrónica para facturación electrónica con formato Facturae",
            'sigcertif_digest': certificate_sudo._get_fingerprint_bytes(formatting='base64').decode(),
            'x509_issuer_description': issuer,
            'x509_serial_number': int(certificate_sudo.serial_number),
        })
        signature = self.env['ir.qweb']._render('l10n_es_edi_facturae.template_xades_signature', signature_data)
        signature = cleanup_xml_node(signature, remove_blank_nodes=False)
        root.append(signature)
        _reference_digests(signature.find("ds:SignedInfo", namespaces=NS_MAP))

        signed_info_xml = signature.find("ds:SignedInfo", namespaces=NS_MAP)
        signature.find("ds:SignatureValue", namespaces=NS_MAP).text = certificate_sudo._sign(_canonicalize_node(signed_info_xml)).decode()

        return etree.tostring(root, xml_declaration=True, encoding='UTF-8', standalone=True)

```

## File: models\account_move_send.py

```python
import logging

from odoo import _, models, SUPERUSER_ID
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class AccountMoveSend(models.AbstractModel):
    _inherit = 'account.move.send'

    # -------------------------------------------------------------------------
    # ATTACHMENTS
    # -------------------------------------------------------------------------

    def _get_invoice_extra_attachments(self, move):
        # EXTENDS 'account'
        return super()._get_invoice_extra_attachments(move) + move.l10n_es_edi_facturae_xml_id

    def _get_placeholder_mail_attachments_data(self, move, invoice_edi_format=None, extra_edis=None):
        # EXTENDS 'account'
        results = super()._get_placeholder_mail_attachments_data(move, invoice_edi_format=invoice_edi_format, extra_edis=extra_edis)

        if invoice_edi_format == 'es_facturae' and move._l10n_es_edi_facturae_get_default_enable():
            filename = f'{move.name.replace("/", "_")}_facturae_signed.xml'
            results.append({
                'id': f'placeholder_{filename}',
                'name': filename,
                'mimetype': 'application/xml',
                'placeholder': True,
            })

        return results

    # -------------------------------------------------------------------------
    # SENDING METHODS
    # -------------------------------------------------------------------------

    def _hook_invoice_document_before_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_before_pdf_report_render(invoice, invoice_data)

        if invoice_data['invoice_edi_format'] == 'es_facturae' and invoice._l10n_es_edi_facturae_get_default_enable():
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

    def _link_invoice_documents(self, invoices_data):
        # EXTENDS 'account'
        super()._link_invoice_documents(invoices_data)

        attachments_vals = [
            invoice_data.get('l10n_es_edi_facturae_attachment_values')
            for invoice_data in invoices_data.values()
            if invoice_data.get('l10n_es_edi_facturae_attachment_values')
        ]
        if attachments_vals:
            attachments = self.env['ir.attachment'].with_user(SUPERUSER_ID).create(attachments_vals)
            res_ids = attachments.mapped('res_id')
            self.env['account.move'].browse(res_ids).invalidate_recordset(fnames=['l10n_es_edi_facturae_xml_id', 'l10n_es_edi_facturae_xml_file'])

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

## File: models\certificate.py

```python
import base64

from cryptography import x509

from odoo import fields, models


class Certificate(models.Model):
    _inherit = 'certificate.certificate'

    scope = fields.Selection(
        selection_add=[
            ('facturae', 'Facturae')
        ],
    )

    def _l10n_es_edi_facturae_get_issuer(self):
        self.ensure_one()

        cert = x509.load_pem_x509_certificate(base64.b64decode(self.pem_certificate))
        rfc4514_attr = dict(element.rfc4514_string().split("=", 1) for element in cert.issuer.rdns)

        # The 'Organizational Unit' field is optional
        issuer = f"CN={rfc4514_attr.pop('CN')}, "
        if 'OU' in rfc4514_attr:
            issuer += f"OU={rfc4514_attr.pop('OU')}, "
        issuer += f"O={rfc4514_attr.pop('O')}, C={rfc4514_attr.pop('C')}"

        # Add remaining certificate fields (not all certificates have other fields)
        return issuer + "".join([f", {key}={value}" for key, value in rfc4514_attr.items()])

```

## File: models\res_company.py

```python
from odoo import fields, models


class Company(models.Model):
    _inherit = 'res.company'

    l10n_es_edi_facturae_residence_type = fields.Char(string='Facturae EDI Residency Type Code', related='partner_id.l10n_es_edi_facturae_residence_type')
    l10n_es_edi_facturae_certificate_ids = fields.One2many(string='Facturae EDI signing certificate',
        comodel_name='certificate.certificate', inverse_name='company_id', domain=[('scope', '=', 'facturae')])

```

## File: models\res_partner.py

```python
from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools import check_barcode_encoding


class AcRoleType(models.Model):
    _name = 'l10n_es_edi_facturae.ac_role_type'
    _description = 'Administrative Center Role Type'

    code = fields.Char(required=True)
    name = fields.Char(required=True, translate=True)


class Partner(models.Model):
    _inherit = 'res.partner'

    invoice_edi_format = fields.Selection(selection_add=[('es_facturae', 'Facturae')])
    type = fields.Selection(selection_add=[('facturae_ac', 'FACe Center'), ('other',)])
    l10n_es_edi_facturae_ac_center_code = fields.Char(string='Code', size=10, help="Code of the issuing department.")
    l10n_es_edi_facturae_ac_role_type_ids = fields.Many2many(
        string='Roles',
        comodel_name='l10n_es_edi_facturae.ac_role_type',
        help="It indicates the role played by the Operational Point defined as a Workplace/Department.\n"
             "These functions are:\n"
             "- Receiver: Workplace associated to the recipient's tax identification number where the invoice will be received.\n"
             "- Payer: Workplace associated to the recipient's tax identification number responsible for paying the invoice.\n"
             "- Buyer: Workplace associated to the recipient's tax identification number who issued the purchase order.\n"
             "- Collector: Workplace associated to  the issuer's tax identification number responsible for handling the collection.\n"
             "- Fiscal: Workplace associated to the recipient's tax identification number, where an Operational Point mailbox is shared "
             "by different client companies with different tax identification numbers and it is necessary to differentiate between "
             "where the message is received (shared letterbox) and the workplace where it must be stored (recipient company).",
    )
    l10n_es_edi_facturae_ac_physical_gln = fields.Char(
        string='Physical GLN',
        size=14,
        help="Identification of the connection point to the VAN EDI (Global Location Number). Barcode of 13 standard positions. "
        "Codes are registered in Spain by AECOC. The code is made up of the country code (2 positions) Spain is '84' "
        "+ Company code (5 positions) + the remaining positions. The last one is the product + check digit."
    )
    l10n_es_edi_facturae_ac_logical_operational_point = fields.Char(
        string='Logical Operational Point',
        size=14,
        help="Code identifying the company. Barcode of 13 standard positions. Codes are registered in Spain by AECOC. "
        "The code is made up of the country code (2 positions) Spain is '84' + Company code (5 positions) + the remaining positions. "
        "The last one is the product + check digit.",
    )
    l10n_es_edi_facturae_residence_type = fields.Char(string='Facturae EDI Residency Type Code',
        compute='_compute_l10n_es_edi_facturae_residence_type', store=False, readonly=True,)

    @api.constrains('l10n_es_edi_facturae_ac_physical_gln')
    def _validate_l10n_es_edi_facturae_ac_physical_gln(self):
        for p in self:
            if not p.l10n_es_edi_facturae_ac_physical_gln:
                continue
            if not check_barcode_encoding(p.l10n_es_edi_facturae_ac_physical_gln, 'ean13'):
                raise ValidationError(_('The Physical GLN entered is not valid.'))

    @api.constrains('l10n_es_edi_facturae_ac_logical_operational_point')
    def _validate_l10n_es_edi_facturae_ac_logical_operational_point(self):
        for p in self:
            if not p.l10n_es_edi_facturae_ac_logical_operational_point:
                continue
            if not check_barcode_encoding(p.l10n_es_edi_facturae_ac_logical_operational_point, 'ean13'):
                raise ValidationError(_('The Logical Operational Point entered is not valid.'))

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
from . import account_move_send
from . import account_tax
from . import certificate
from . import res_company
from . import res_partner
from . import uom_uom
from . import account_move
from . import account_chart_template

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
l10n_es_edi_facturae.access_l10n_es_edi_facturae_ac_role_type_invoice,access_l10n_es_edi_facturae_ac_role_type,l10n_es_edi_facturae.model_l10n_es_edi_facturae_ac_role_type,account.group_account_invoice,1,0,0,0
l10n_es_edi_facturae.access_l10n_es_edi_facturae_ac_role_type_readonly,access_l10n_es_edi_facturae_ac_role_type,l10n_es_edi_facturae.model_l10n_es_edi_facturae_ac_role_type,account.group_account_readonly,1,0,0,0

```

## File: views\account_menuitem.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <menuitem id="menu_l10n_es_edi_facturae_root" name="Spain Facturae EDI" sequence="110"
                  parent="account.menu_finance_configuration">
            <menuitem id="menu_l10n_es_edi_facturae_root_certificates" name="Certificates" action="l10n_es_edi_facturae_certificate_action"
                  sequence="100"/>
        </menuitem>
    </data>
</odoo>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_move_form" model="ir.ui.view">
            <field name="name">account.move.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='other_info']" position="inside">
                        <group string="Factura-e"
                               name="l10n_es_facturae_invoicing_period"
                               invisible="country_code != 'ES'">
                            <field name="l10n_es_invoicing_period_start_date"/>
                            <field name="l10n_es_invoicing_period_end_date"/>
                            <field name="l10n_es_payment_means"/>
                        </group>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_tax_tree_inherit_l10n_es_edi_facturae" model="ir.ui.view">
            <field name="name">account.tax.list.inherit.l10n_es_edi_facturae</field>
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
    <record id="certificate_certificate_view_search" model="ir.ui.view">
        <field name="name">certificate_certificate_view_search.inherit.l10n_es_edi_facturae</field>
        <field name="model">certificate.certificate</field>
        <field name="inherit_id" ref="certificate.certificate_certificate_view_search"/>
        <field name="arch" type="xml">
            <filter name="scope_general" position="after">
                <filter string="Facturae" name="scope_facturae" domain="[('scope','=','facturae')]" help="Facturae certificates"/>
            </filter>
        </field>
    </record>

    <record id="l10n_es_edi_facturae_certificate_action" model="ir.actions.act_window">
        <field name="name">Certificates for Facturae EDI invoices on Spain</field>
        <field name="res_model">certificate.certificate</field>
        <field name="view_mode">list,form</field>
        <field name="context">{'scope': 'facturae', 'search_default_scope_facturae': 1}</field>
        <field name="help" type="html">
            <p class="oe_view_nocontent_create">Create the first certificate</p>
        </field>
    </record>

    <record id="certificate_certificate_view_form" model="ir.ui.view">
        <field name="name">certificate_certificate_view_form.inherit.l10n_es_edi_facturae</field>
        <field name="model">certificate.certificate</field>
        <field name="inherit_id" ref="certificate.certificate_certificate_view_form"/>
        <field name="arch" type="xml">
            <field name="scope" position="attributes">
                <attribute name="invisible">False</attribute>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_es_edi_facturae" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_es_edi_facturae</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='child_ids']//div[hasclass('oe_edit_only')]" position="inside">
                <p class="mb-0" invisible="type != 'facturae_ac'">
                    <span>Administrative Center for Spain Public Administrations. Used in Spanish electronic invoices.</span>
                </p>
            </xpath>
            <xpath expr="//field[@name='child_ids']//form//field[@name='name']" position="after">
                <field name="l10n_es_edi_facturae_ac_center_code" invisible="type != 'facturae_ac'"/>
                <field name="l10n_es_edi_facturae_ac_role_type_ids" widget="many2many_tags" invisible="type != 'facturae_ac'"/>
                <field name="l10n_es_edi_facturae_ac_physical_gln" invisible="type != 'facturae_ac'"/>
                <field name="l10n_es_edi_facturae_ac_logical_operational_point" invisible="type != 'facturae_ac'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\uom_uom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_uom_tree_view_inherit_l10n_es_edi_facturae" model="ir.ui.view">
        <field name="name">uom.uom.list.inherit.l10n_es_edi_facturae</field>
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
            <list position="inside">
                <field name="l10n_es_edi_facturae_uom_code" string="Spanish Facturae EDI type"/>
            </list>
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
            <field name="name">account.tax.list.inherit.l10n_es_edi_facturae</field>
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

## File: wizard\__init__.py

```python
from . import account_move_reversal

```

