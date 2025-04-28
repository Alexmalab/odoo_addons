# Odoo Module: l10n_no_edi

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from odoo import api, SUPERUSER_ID


def _post_init_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    modules = env['ir.module.module'].search([('name', '=', 'account_edi_ubl_cii'), ('state', '=', 'uninstalled')])
    modules.sudo().button_install()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Norway - E-Invoicing (EHF 3)',
    'icon': '/l10n_no/static/description/icon.png',
    'version': '0.1',
    'category': 'Accounting/Localizations/EDI',
    'summary': 'E-Invoicing, Universal Business Language (EHF 3)',
    'description': """
EHF 3 is the Norwegian implementation of EN 16931 norm.
    """,
    'depends': ['l10n_no', 'account_edi_ubl_bis3'],
    'data': [
        'data/account_edi_data.xml',
        'data/ehf_3_template.xml',
    ],
    'post_init_hook': '_post_init_hook',
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="edi_ehf_3" model="account.edi.format">
            <field name="name">EHF 3 (Norway)</field>
            <field name="code">ehf_3</field>
        </record>

    </data>
</odoo>

```

## File: data\ehf_3_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="export_ehf_3_invoice_partner" inherit_id="account_edi_ubl_bis3.export_bis3_invoice_partner" primary="True">
            <xpath expr="//*[local-name()='PartyTaxScheme']" position="after">
                <cac:PartyTaxScheme xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <cbc:CompanyID>Foretaksregisteret</cbc:CompanyID>
                    <cac:TaxScheme>
                        <cbc:ID>TAX</cbc:ID>
                    </cac:TaxScheme>
                </cac:PartyTaxScheme>
            </xpath>
        </template>

        <template id="export_ehf_3_invoice" inherit_id="account_edi_ubl_bis3.export_bis3_invoice" primary="True">
            <!-- Only for the supplier, append Foretaksregisteret, see rule NO-R-002 -->
            <xpath expr="//*[local-name()='AccountingSupplierParty']" position="replace">
                <cac:AccountingSupplierParty t-call="l10n_no_edi.export_ehf_3_invoice_partner"
                    xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                    <t t-set="partner_vals" t-value="supplier_vals"/>
                </cac:AccountingSupplierParty>
            </xpath>
        </template>

    </data>
</odoo>

```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-

from odoo import models, _
from odoo.addons.account_edi_ubl_bis3.models.account_edi_format import COUNTRY_EAS


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    ####################################################
    # Import
    ####################################################

    def _is_ubl(self, filename, tree):
        """ OVERRIDE so that the generic ubl parser does not parse BIS3 any longer.
        """
        is_ubl = super()._is_ubl(filename, tree)
        return is_ubl and not self._is_ehf_3(filename, tree)

    def _is_ehf_3(self, filename, tree):
        ns = self._get_bis3_namespaces()
        return tree.tag == '{urn:oasis:names:specification:ubl:schema:xsd:Invoice-2}Invoice' \
            and 'peppol' in tree.findtext('./cbc:ProfileID', '', namespaces=ns) \
            and tree.xpath(
                "./cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme/cbc:CompanyID[text()='Foretaksregisteret']",
                namespaces=ns) is not None

    def _bis3_get_extra_partner_domains(self, tree):
        if self.code == 'ehf_3':
            ns = self._get_bis3_namespaces()
            bronnoysund = tree.xpath('./cac:AccountingSupplierParty/cac:Party/cbc:EndpointID[@schemeID="0192"]/text()', namespaces=ns)
            if bronnoysund:
                return [('l10n_no_bronnoysund_number', '=', bronnoysund[0])]
        return super()._bis3_get_extra_partner_domains(tree)

    ####################################################
    # Export
    ####################################################

    def _get_ehf_3_values(self, invoice):
        values = super()._get_bis3_values(invoice)
        for partner_vals in (values['customer_vals'], values['supplier_vals']):
            partner = partner_vals['partner']
            if partner.country_code == 'NO':
                partner_vals.update(
                    bis3_endpoint=partner.l10n_no_bronnoysund_number,
                    bis3_endpoint_scheme='0192',
                )

        return values

    def _export_ehf_3(self, invoice):
        self.ensure_one()
        # Create file content.
        xml_content = self.env.ref('l10n_no_edi.export_ehf_3_invoice')._render(self._get_ehf_3_values(invoice))
        vat = invoice.company_id.partner_id.commercial_partner_id.vat
        xml_name = 'ehf-%s%s%s.xml' % (vat or '', '-' if vat else '', invoice.name.replace('/', '_'))
        return self.env['ir.attachment'].create({
            'name': xml_name,
            'raw': xml_content.encode(),
            'res_model': 'account.move',
            'res_id': invoice.id,
            'mimetype': 'application/xml'
        })

    ####################################################
    # Account.edi.format override
    ####################################################

    def _check_move_configuration(self, invoice):
        errors = super()._check_move_configuration(invoice)
        if self.code != 'ehf_3' or self._is_account_edi_ubl_cii_available():
            return errors

        supplier = invoice.company_id.partner_id.commercial_partner_id
        if supplier.country_code == 'NO' and not supplier.l10n_no_bronnoysund_number:
            errors.append(_("The supplier %r must have a Bronnoysund company registry.", supplier.display_name))

        if supplier.country_code != 'NO' and supplier.country_code not in COUNTRY_EAS:
            errors.append(_("The supplier %r is from a country that is not supported for EHF (Bis3)", supplier.display_name))

        customer = invoice.commercial_partner_id
        if customer.country_code == 'NO' and not customer.l10n_no_bronnoysund_number:
            errors.append(_("The customer %r must have a Bronnoysund company registry.", customer.display_name))

        if customer.country_code != 'NO' and customer.country_code not in COUNTRY_EAS:
            errors.append(_("The customer %r is from a country that is not supported for EHF (Bis3)", customer.display_name))

        return errors

    def _is_compatible_with_journal(self, journal):
        self.ensure_one()
        if self.code != 'ehf_3' or self._is_account_edi_ubl_cii_available():
            return super()._is_compatible_with_journal(journal)
        return journal.type == 'sale' and journal.country_code == 'NO'

    def _post_invoice_edi(self, invoices):
        self.ensure_one()
        if self.code != 'ehf_3' or self._is_account_edi_ubl_cii_available():
            return super()._post_invoice_edi(invoices)

        invoice = invoices  # no batch ensure that there is only one invoice
        attachment = self._export_ehf_3(invoice)
        return {invoice: {'attachment': attachment}}

    def _create_invoice_from_xml_tree(self, filename, tree, journal=None):
        self.ensure_one()
        if self.code == 'ehf_3' and self._is_ehf_3(filename, tree) and not self._is_account_edi_ubl_cii_available():
            return self._decode_bis3(tree, self.env['account.move'])
        return super()._create_invoice_from_xml_tree(filename, tree, journal=journal)

    def _update_invoice_from_xml_tree(self, filename, tree, invoice):
        self.ensure_one()
        if self.code == 'ehf_3' and self._is_ehf_3(filename, tree) and not self._is_account_edi_ubl_cii_available():
            return self._decode_bis3(tree, invoice)
        return super()._update_invoice_from_xml_tree(filename, tree, invoice)

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-

from . import account_edi_format

```

## File: test_xml_file\ehf_test.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<Invoice xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
    <cbc:CustomizationID>urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0</cbc:CustomizationID>
    <cbc:ProfileID>urn:fdc:peppol.eu:2017:poacc:billing:01:1.0</cbc:ProfileID>
    <cbc:ID>TOSL108</cbc:ID>
    <cbc:IssueDate>2013-06-30</cbc:IssueDate>
    <cbc:DueDate>2013-07-20</cbc:DueDate>
    <cbc:InvoiceTypeCode>380</cbc:InvoiceTypeCode>
    <cbc:Note>Ordered in our booth at the convention.</cbc:Note>
    <cbc:TaxPointDate>2013-06-30</cbc:TaxPointDate>
    <cbc:DocumentCurrencyCode>NOK</cbc:DocumentCurrencyCode>
    <cbc:AccountingCost>Project cost code 123</cbc:AccountingCost>
    <cbc:BuyerReference>3150bdn</cbc:BuyerReference>
    <cac:InvoicePeriod>
        <cbc:StartDate>2013-06-01</cbc:StartDate>
        <cbc:EndDate>2013-06-30</cbc:EndDate>
    </cac:InvoicePeriod>
    <cac:OrderReference>
        <cbc:ID>123</cbc:ID>
    </cac:OrderReference>
    <cac:ContractDocumentReference>
        <cbc:ID>Contract321</cbc:ID>
    </cac:ContractDocumentReference>
    <cac:AdditionalDocumentReference>
        <cbc:ID>Doc1</cbc:ID>
        <cbc:DocumentDescription>Timesheet</cbc:DocumentDescription>
        <cac:Attachment>
            <cac:ExternalReference>
                <cbc:URI>http://www.suppliersite.eu/sheet001.html</cbc:URI>
            </cac:ExternalReference>
        </cac:Attachment>
    </cac:AdditionalDocumentReference>
    <cac:AdditionalDocumentReference>
        <cbc:ID>Doc2</cbc:ID>
        <cbc:DocumentDescription>EHF specification</cbc:DocumentDescription>
        <cac:Attachment>
            <cbc:EmbeddedDocumentBinaryObject mimeCode="application/pdf" filename="Hours-spent.csv">VGVzdCBiYXNlIDY0IGVuY29kaW5n</cbc:EmbeddedDocumentBinaryObject>
        </cac:Attachment>
    </cac:AdditionalDocumentReference>
    <cac:AccountingSupplierParty>
        <cac:Party>
            <cbc:EndpointID schemeID="0192">864234232</cbc:EndpointID>
            <cac:PartyIdentification>
                <cbc:ID schemeID="0088">1238764941386</cbc:ID>
            </cac:PartyIdentification>
            <cac:PartyName>
                <cbc:Name>partner_a</cbc:Name>
            </cac:PartyName>
            <cac:PostalAddress>
                <cbc:StreetName>Main street 34</cbc:StreetName>
                <cbc:AdditionalStreetName>Suite 123</cbc:AdditionalStreetName>
                <cbc:CityName>Big city</cbc:CityName>
                <cbc:PostalZone>54321</cbc:PostalZone>
                <cbc:CountrySubentity>RegionA</cbc:CountrySubentity>
                <cac:Country>
                    <cbc:IdentificationCode>NO</cbc:IdentificationCode>
                </cac:Country>
            </cac:PostalAddress>
            <cac:PartyTaxScheme>
                <cbc:CompanyID>NO864234232MVA</cbc:CompanyID>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:PartyTaxScheme>
            <cac:PartyTaxScheme>
                <cbc:CompanyID>Foretaksregisteret</cbc:CompanyID>
                <cac:TaxScheme>
                    <cbc:ID>TAX</cbc:ID>
                </cac:TaxScheme>
            </cac:PartyTaxScheme>
            <cac:PartyLegalEntity>
                <cbc:RegistrationName>The Sellercompany ASA</cbc:RegistrationName>
                <cbc:CompanyID schemeID="0192">864234232</cbc:CompanyID>
            </cac:PartyLegalEntity>
            <cac:Contact>
                <cbc:Name>Antonio Salesmacher</cbc:Name>
                <cbc:Telephone>46211230</cbc:Telephone>
                <cbc:ElectronicMail>antonio@salescompany.no</cbc:ElectronicMail>
            </cac:Contact>
        </cac:Party>
    </cac:AccountingSupplierParty>
    <cac:AccountingCustomerParty>
        <cac:Party>
            <cbc:EndpointID schemeID="0192">987654325</cbc:EndpointID>
            <cac:PartyIdentification>
                <cbc:ID schemeID="0088">3456789012098</cbc:ID>
            </cac:PartyIdentification>
            <cac:PartyName>
                <cbc:Name>The Buyercompany</cbc:Name>
            </cac:PartyName>
            <cac:PostalAddress>
                <cbc:StreetName>Anystreet 8</cbc:StreetName>
                <cbc:AdditionalStreetName>Back door</cbc:AdditionalStreetName>
                <cbc:CityName>Anytown</cbc:CityName>
                <cbc:PostalZone>101</cbc:PostalZone>
                <cbc:CountrySubentity>RegionB</cbc:CountrySubentity>
                <cac:Country>
                    <cbc:IdentificationCode>NO</cbc:IdentificationCode>
                </cac:Country>
            </cac:PostalAddress>
            <cac:PartyTaxScheme>
                <cbc:CompanyID>NO987654325MVA</cbc:CompanyID>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:PartyTaxScheme>
            <cac:PartyLegalEntity>
                <cbc:RegistrationName>Buyercompany ASA</cbc:RegistrationName>
                <cbc:CompanyID schemeID="0192">987654325</cbc:CompanyID>
            </cac:PartyLegalEntity>
            <cac:Contact>
                <cbc:Name>John Doe</cbc:Name>
                <cbc:Telephone>5121230</cbc:Telephone>
                <cbc:ElectronicMail>john@buyercompany.no</cbc:ElectronicMail>
            </cac:Contact>
        </cac:Party>
    </cac:AccountingCustomerParty>
    <cac:PayeeParty>
        <cac:PartyIdentification>
            <cbc:ID schemeID="0088">2298740918237</cbc:ID>
        </cac:PartyIdentification>
        <cac:PartyName>
            <cbc:Name>Ebeneser Scrooge AS</cbc:Name>
        </cac:PartyName>
        <cac:PartyLegalEntity>
            <cbc:CompanyID schemeID="0192">999999999</cbc:CompanyID>
        </cac:PartyLegalEntity>
    </cac:PayeeParty>
    <cac:TaxRepresentativeParty>
        <cac:PartyName>
            <cbc:Name>Tax handling company AS</cbc:Name>
        </cac:PartyName>
        <cac:PostalAddress>
            <cbc:StreetName>Regent street</cbc:StreetName>
            <cbc:AdditionalStreetName>Front door</cbc:AdditionalStreetName>
            <cbc:CityName>Newtown</cbc:CityName>
            <cbc:PostalZone>101</cbc:PostalZone>
            <cbc:CountrySubentity>RegionC</cbc:CountrySubentity>
            <cac:Country>
                <cbc:IdentificationCode>NO</cbc:IdentificationCode>
            </cac:Country>
        </cac:PostalAddress>
        <cac:PartyTaxScheme>
            <cbc:CompanyID>NO999999999MVA</cbc:CompanyID>
            <cac:TaxScheme>
                <cbc:ID>VAT</cbc:ID>
            </cac:TaxScheme>
        </cac:PartyTaxScheme>
    </cac:TaxRepresentativeParty>
    <cac:Delivery>
        <cbc:ActualDeliveryDate>2013-06-15</cbc:ActualDeliveryDate>
        <cac:DeliveryLocation>
            <cbc:ID schemeID="0088">6754238987643</cbc:ID>
        </cac:DeliveryLocation>
    </cac:Delivery>
    <cac:PaymentMeans>
        <cbc:PaymentMeansCode>31</cbc:PaymentMeansCode>
        <cbc:PaymentID>0003434323213231</cbc:PaymentID>
        <cac:PayeeFinancialAccount>
            <cbc:ID>86011117947</cbc:ID>
            <cac:FinancialInstitutionBranch>
                <cbc:ID>DNBANOKK</cbc:ID>
            </cac:FinancialInstitutionBranch>
        </cac:PayeeFinancialAccount>
    </cac:PaymentMeans>
    <cac:PaymentTerms>
        <cbc:Note>2 % discount if paid within 2 days. Penalty percentage 10% from due date</cbc:Note>
    </cac:PaymentTerms>
    <cac:AllowanceCharge>
        <cbc:ChargeIndicator>true</cbc:ChargeIndicator>
        <cbc:AllowanceChargeReasonCode>FC</cbc:AllowanceChargeReasonCode>
        <cbc:AllowanceChargeReason>Freight</cbc:AllowanceChargeReason>
        <cbc:Amount currencyID="NOK">100</cbc:Amount>
        <cac:TaxCategory>
            <cbc:ID>S</cbc:ID>
            <cbc:Percent>25</cbc:Percent>
            <cac:TaxScheme>
                <cbc:ID>VAT</cbc:ID>
            </cac:TaxScheme>
        </cac:TaxCategory>
    </cac:AllowanceCharge>
    <cac:AllowanceCharge>
        <cbc:ChargeIndicator>false</cbc:ChargeIndicator>
        <cbc:AllowanceChargeReasonCode>95</cbc:AllowanceChargeReasonCode>
        <cbc:AllowanceChargeReason>Promotion discount</cbc:AllowanceChargeReason>
        <cbc:Amount currencyID="NOK">100</cbc:Amount>
        <cac:TaxCategory>
            <cbc:ID>S</cbc:ID>
            <cbc:Percent>25</cbc:Percent>
            <cac:TaxScheme>
                <cbc:ID>VAT</cbc:ID>
            </cac:TaxScheme>
        </cac:TaxCategory>
    </cac:AllowanceCharge>
    <cac:TaxTotal>
        <cbc:TaxAmount currencyID="NOK">365.28</cbc:TaxAmount>
        <cac:TaxSubtotal>
            <cbc:TaxableAmount currencyID="NOK">1460.5</cbc:TaxableAmount>
            <cbc:TaxAmount currencyID="NOK">365.13</cbc:TaxAmount>
            <cac:TaxCategory>
                <cbc:ID>S</cbc:ID>
                <cbc:Percent>25</cbc:Percent>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:TaxCategory>
        </cac:TaxSubtotal>
        <cac:TaxSubtotal>
            <cbc:TaxableAmount currencyID="NOK">1</cbc:TaxableAmount>
            <cbc:TaxAmount currencyID="NOK">0.15</cbc:TaxAmount>
            <cac:TaxCategory>
                <cbc:ID>S</cbc:ID>
                <cbc:Percent>15</cbc:Percent>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:TaxCategory>
        </cac:TaxSubtotal>
        <cac:TaxSubtotal>
            <cbc:TaxableAmount currencyID="NOK">-25</cbc:TaxableAmount>
            <cbc:TaxAmount currencyID="NOK">0</cbc:TaxAmount>
            <cac:TaxCategory>
                <cbc:ID>E</cbc:ID>
                <cbc:Percent>0</cbc:Percent>
                <cbc:TaxExemptionReason>Exempt New Means of Transport</cbc:TaxExemptionReason>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:TaxCategory>
        </cac:TaxSubtotal>
    </cac:TaxTotal>
    <cac:LegalMonetaryTotal>
        <cbc:LineExtensionAmount currencyID="NOK">1436.5</cbc:LineExtensionAmount>
        <cbc:TaxExclusiveAmount currencyID="NOK">1436.5</cbc:TaxExclusiveAmount>
        <cbc:TaxInclusiveAmount currencyID="NOK">1801.78</cbc:TaxInclusiveAmount>
        <cbc:AllowanceTotalAmount currencyID="NOK">100</cbc:AllowanceTotalAmount>
        <cbc:ChargeTotalAmount currencyID="NOK">100</cbc:ChargeTotalAmount>
        <cbc:PrepaidAmount currencyID="NOK">1000</cbc:PrepaidAmount>
        <cbc:PayableRoundingAmount currencyID="NOK">0.22</cbc:PayableRoundingAmount>
        <cbc:PayableAmount currencyID="NOK">802.00</cbc:PayableAmount>
    </cac:LegalMonetaryTotal>
    <cac:InvoiceLine>
        <cbc:ID>1</cbc:ID>
        <cbc:Note>Scratch on box</cbc:Note>
        <cbc:InvoicedQuantity unitCode="NAR">1</cbc:InvoicedQuantity>
        <cbc:LineExtensionAmount currencyID="NOK">1273</cbc:LineExtensionAmount>
        <cbc:AccountingCost>BookingCode001</cbc:AccountingCost>
        <cac:InvoicePeriod>
            <cbc:StartDate>2013-06-01</cbc:StartDate>
            <cbc:EndDate>2013-06-30</cbc:EndDate>
        </cac:InvoicePeriod>
        <cac:OrderLineReference>
            <cbc:LineID>1</cbc:LineID>
        </cac:OrderLineReference>
        <cac:AllowanceCharge>
            <cbc:ChargeIndicator>false</cbc:ChargeIndicator>
            <cbc:AllowanceChargeReason>Damage</cbc:AllowanceChargeReason>
            <cbc:Amount currencyID="NOK">12</cbc:Amount>
        </cac:AllowanceCharge>
        <cac:AllowanceCharge>
            <cbc:ChargeIndicator>true</cbc:ChargeIndicator>
            <cbc:AllowanceChargeReason>Testing</cbc:AllowanceChargeReason>
            <cbc:Amount currencyID="NOK">12</cbc:Amount>
        </cac:AllowanceCharge>
        <cac:Item>
            <cbc:Description>Processor: Intel Core 2 Duo SU9400 LV (1.4GHz). RAM: 3MB. Screen 1440x900</cbc:Description>
            <cbc:Name>Laptop computer</cbc:Name>
            <cac:SellersItemIdentification>
                <cbc:ID>JB007</cbc:ID>
            </cac:SellersItemIdentification>
            <cac:StandardItemIdentification>
                <cbc:ID schemeID="0088">1234567890124</cbc:ID>
            </cac:StandardItemIdentification>
            <cac:OriginCountry>
                <cbc:IdentificationCode>DE</cbc:IdentificationCode>
            </cac:OriginCountry>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="MP">12344321</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="STI">65434568</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:ClassifiedTaxCategory>
                <cbc:ID>S</cbc:ID>
                <cbc:Percent>25</cbc:Percent>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:ClassifiedTaxCategory>
            <cac:AdditionalItemProperty>
                <cbc:Name>Color</cbc:Name>
                <cbc:Value>Black</cbc:Value>
            </cac:AdditionalItemProperty>
        </cac:Item>
        <cac:Price>
            <cbc:PriceAmount currencyID="NOK">1273</cbc:PriceAmount>
            <cbc:BaseQuantity>1</cbc:BaseQuantity>
            <cac:AllowanceCharge>
                <cbc:ChargeIndicator>false</cbc:ChargeIndicator>
                <cbc:Amount currencyID="NOK">227</cbc:Amount>
                <cbc:BaseAmount currencyID="NOK">1500</cbc:BaseAmount>
            </cac:AllowanceCharge>
        </cac:Price>
    </cac:InvoiceLine>
    <cac:InvoiceLine>
        <cbc:ID>2</cbc:ID>
        <cbc:Note>Cover is slightly damaged.</cbc:Note>
        <cbc:InvoicedQuantity unitCode="NAR">-1</cbc:InvoicedQuantity>
        <cbc:LineExtensionAmount currencyID="NOK">-3.96</cbc:LineExtensionAmount>
        <cbc:AccountingCost>BookingCode002</cbc:AccountingCost>
        <cac:OrderLineReference>
            <cbc:LineID>5</cbc:LineID>
        </cac:OrderLineReference>
        <cac:Item>
            <cbc:Name>Returned "Advanced computing" book</cbc:Name>
            <cac:SellersItemIdentification>
                <cbc:ID>JB008</cbc:ID>
            </cac:SellersItemIdentification>
            <cac:StandardItemIdentification>
                <cbc:ID schemeID="0088">1234567890125</cbc:ID>
            </cac:StandardItemIdentification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="MP">32344324</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="STI">65434567</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:ClassifiedTaxCategory>
                <cbc:ID schemeID="UNCL5305">S</cbc:ID>
                <cbc:Percent>15</cbc:Percent>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:ClassifiedTaxCategory>
        </cac:Item>
        <cac:Price>
            <cbc:PriceAmount currencyID="NOK">3.96</cbc:PriceAmount>
            <cbc:BaseQuantity>1</cbc:BaseQuantity>
        </cac:Price>
    </cac:InvoiceLine>
    <cac:InvoiceLine>
        <cbc:ID>3</cbc:ID>
        <cbc:InvoicedQuantity unitCode="NAR">2</cbc:InvoicedQuantity>
        <cbc:LineExtensionAmount currencyID="NOK">4.96</cbc:LineExtensionAmount>
        <cbc:AccountingCost>BookingCode003</cbc:AccountingCost>
        <cac:OrderLineReference>
            <cbc:LineID>3</cbc:LineID>
        </cac:OrderLineReference>
        <cac:Item>
            <cbc:Name>"Computing for dummies" book</cbc:Name>
            <cac:SellersItemIdentification>
                <cbc:ID>JB009</cbc:ID>
            </cac:SellersItemIdentification>
            <cac:StandardItemIdentification>
                <cbc:ID schemeID="0088">1234567890126</cbc:ID>
            </cac:StandardItemIdentification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="MP">32344324</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="STI">65434566</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:ClassifiedTaxCategory>
                <cbc:ID schemeID="UNCL5305">S</cbc:ID>
                <cbc:Percent>15</cbc:Percent>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:ClassifiedTaxCategory>
        </cac:Item>
        <cac:Price>
            <cbc:PriceAmount currencyID="NOK">2.48</cbc:PriceAmount>
            <cbc:BaseQuantity>1</cbc:BaseQuantity>
            <cac:AllowanceCharge>
                <cbc:ChargeIndicator>false</cbc:ChargeIndicator>
                <cbc:Amount currencyID="NOK">0.27</cbc:Amount>
                <cbc:BaseAmount currencyID="NOK">2.75</cbc:BaseAmount>
            </cac:AllowanceCharge>
        </cac:Price>
    </cac:InvoiceLine>
    <cac:InvoiceLine>
        <cbc:ID>4</cbc:ID>
        <cbc:InvoicedQuantity unitCode="NAR">-1</cbc:InvoicedQuantity>
        <cbc:LineExtensionAmount currencyID="NOK">-25</cbc:LineExtensionAmount>
        <cbc:AccountingCost>BookingCode004</cbc:AccountingCost>
        <cac:OrderLineReference>
            <cbc:LineID>2</cbc:LineID>
        </cac:OrderLineReference>
        <cac:Item>
            <cbc:Name>Returned IBM 5150 desktop</cbc:Name>
            <cac:SellersItemIdentification>
                <cbc:ID>JB010</cbc:ID>
            </cac:SellersItemIdentification>
            <cac:StandardItemIdentification>
                <cbc:ID schemeID="0088">1234567890127</cbc:ID>
            </cac:StandardItemIdentification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="MP">12344322</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="STI">65434565</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:ClassifiedTaxCategory>
                <cbc:ID schemeID="UNCL5305">E</cbc:ID>
                <cbc:Percent>0</cbc:Percent>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:ClassifiedTaxCategory>
        </cac:Item>
        <cac:Price>
            <cbc:PriceAmount currencyID="NOK">25</cbc:PriceAmount>
            <cbc:BaseQuantity>1</cbc:BaseQuantity>
        </cac:Price>
    </cac:InvoiceLine>
    <cac:InvoiceLine>
        <cbc:ID>5</cbc:ID>
        <cbc:InvoicedQuantity unitCode="MTR">250</cbc:InvoicedQuantity>
        <cbc:LineExtensionAmount currencyID="NOK">187.5</cbc:LineExtensionAmount>
        <cbc:AccountingCost>BookingCode005</cbc:AccountingCost>
        <cac:OrderLineReference>
            <cbc:LineID>4</cbc:LineID>
        </cac:OrderLineReference>
        <cac:Item>
            <cbc:Name>Network cable</cbc:Name>
            <cac:SellersItemIdentification>
                <cbc:ID>JB011</cbc:ID>
            </cac:SellersItemIdentification>
            <cac:StandardItemIdentification>
                <cbc:ID schemeID="0088">1234567890128</cbc:ID>
            </cac:StandardItemIdentification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="MP">12344325</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:CommodityClassification>
                <cbc:ItemClassificationCode listID="STI">65434564</cbc:ItemClassificationCode>
            </cac:CommodityClassification>
            <cac:ClassifiedTaxCategory>
                <cbc:ID schemeID="UNCL5305">S</cbc:ID>
                <cbc:Percent>25</cbc:Percent>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:ClassifiedTaxCategory>
            <cac:AdditionalItemProperty>
                <cbc:Name>Type</cbc:Name>
                <cbc:Value>Cat5</cbc:Value>
            </cac:AdditionalItemProperty>
        </cac:Item>
        <cac:Price>
            <cbc:PriceAmount currencyID="NOK">0.75</cbc:PriceAmount>
            <cbc:BaseQuantity>1</cbc:BaseQuantity>
        </cac:Price>
    </cac:InvoiceLine>
</Invoice>
```

