# Odoo Module: l10n_nl_edi

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
    'name': 'Netherlands - E-Invoicing (NLCIUS 1.0.3)',
    'icon': '/l10n_nl/static/description/icon.png',
    'version': '0.1',
    'category': 'Accounting/Localizations/EDI',
    'summary': 'E-Invoicing, Universal Business Language (NLCIUS 1.0.3), nlc',
    'description': """
NLCIUS is the Dutch implementation of EN 16931 norm. Both for UBL and UN / CEFACT XML CII.
    """,
    'depends': ['l10n_nl', 'account_edi_ubl_bis3'],
    'data': [
        'data/account_edi_data.xml',
        'data/nlcius_template.xml',
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

        <record id="edi_nlcius_1" model="account.edi.format">
            <field name="name">NLCIUS (Netherlands)</field>
            <field name="code">nlcius_1</field>
        </record>

    </data>
</odoo>

```

## File: data\nlcius_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <template id="export_nlcius_invoice_partner"
                  inherit_id="account_edi_ubl_bis3.export_bis3_invoice_partner"
                  primary="True">
            <xpath expr="//*[local-name()='CountrySubentity']" position="replace"/>
        </template>

        <template id="export_nlcius_invoice"
                  inherit_id="account_edi_ubl_bis3.export_bis3_invoice"
                  primary="True">
            <xpath expr="//*[local-name()='AccountingSupplierParty']" position="replace">
                <cac:AccountingSupplierParty t-call="l10n_nl_edi.export_nlcius_invoice_partner"
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                    <t t-set="partner_vals" t-value="supplier_vals"/>
                </cac:AccountingSupplierParty>
            </xpath>
            <xpath expr="//*[local-name()='AccountingCustomerParty']" position="replace">
                <cac:AccountingCustomerParty t-call="l10n_nl_edi.export_nlcius_invoice_partner"
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
            xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                    <t t-set="partner_vals" t-value="customer_vals"/>
                </cac:AccountingCustomerParty>
            </xpath>
        </template>

    </data>
</odoo>

```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-

import markupsafe
from odoo.addons.account_edi_ubl_bis3.models.account_edi_format import COUNTRY_EAS

from odoo import models, _


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    ####################################################
    # Import
    ####################################################

    def _is_ubl(self, filename, tree):
        """ OVERRIDE so that the generic ubl parser does not parse BIS3 any longer.
        """
        is_ubl = super()._is_ubl(filename, tree)
        return is_ubl and not self._is_nlcius(filename, tree)

    def _is_nlcius(self, filename, tree):
        profile_id = tree.find('./{*}ProfileID')
        customization_id = tree.find('./{*}CustomizationID')
        return tree.tag == '{urn:oasis:names:specification:ubl:schema:xsd:Invoice-2}Invoice' and \
            profile_id is not None and 'peppol' in profile_id.text and \
            customization_id is not None and 'nlcius' in customization_id.text

    def _bis3_get_extra_partner_domains(self, tree):
        if self.code == 'nlcius_1':
            endpoint = tree.find('./{*}AccountingSupplierParty/{*}Party/{*}EndpointID')
            if endpoint is not None:
                scheme = endpoint.attrib['schemeID']
                if scheme == '0106' and endpoint.text:
                    return [('l10n_nl_kvk', '=', endpoint.text)]
                elif scheme == '0190' and endpoint.text:
                    return [('l10n_nl_oin', '=', endpoint.text)]
        return super()._bis3_get_extra_partner_domains(tree)

    ####################################################
    # Export
    ####################################################

    def _get_nlcius_values(self, invoice):
        values = super()._get_bis3_values(invoice)
        values.update({
            'customization_id': 'urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0',
            'payment_means_code': 30,
        })

        for partner_vals in (values['customer_vals'], values['supplier_vals']):
            partner = partner_vals['partner']
            endpoint = partner.l10n_nl_oin or partner.l10n_nl_kvk
            if partner.country_code == 'NL' and endpoint:
                scheme = '0190' if partner.l10n_nl_oin else '0106'
                partner_vals.update({
                    'bis3_endpoint': endpoint,
                    'bis3_endpoint_scheme': scheme,
                    'legal_entity': endpoint,
                    'legal_entity_scheme': scheme,
                    'partner_identification': endpoint,
                })

        return values

    def _export_nlcius(self, invoice):
        self.ensure_one()
        # Create file content.
        xml_content = markupsafe.Markup("<?xml version='1.0' encoding='UTF-8'?>")
        xml_content += self.env.ref('l10n_nl_edi.export_nlcius_invoice')._render(self._get_nlcius_values(invoice))
        vat = invoice.company_id.partner_id.commercial_partner_id.vat
        xml_name = 'nlcius-%s%s%s.xml' % (vat or '', '-' if vat else '', invoice.name.replace('/', '_'))
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

        if self.code != 'nlcius_1' or self._is_account_edi_ubl_cii_available():
            return errors

        supplier = invoice.company_id.partner_id.commercial_partner_id
        if not supplier.street or not supplier.zip or not supplier.city:
            errors.append(_("The supplier's address must include street, zip and city (%s).", supplier.display_name))
        if supplier.country_code == 'NL' and not supplier.l10n_nl_kvk and not supplier.l10n_nl_oin:
            errors.append(_("The supplier %s must have a KvK-nummer or OIN.", supplier.display_name))
        if not supplier.vat:
            errors.append(_("Please define a VAT number for '%s'.", supplier.display_name))

        customer = invoice.commercial_partner_id
        if customer.country_code == 'NL' and (not customer.street or not customer.zip or not customer.city):
            errors.append(_("Customer's address must include street, zip and city (%s).", customer.display_name))
        if customer.country_code == 'NL' and not customer.l10n_nl_kvk and not customer.l10n_nl_oin:
            errors.append(_("The customer %s must have a KvK-nummer or OIN.", customer.display_name))

        if not invoice.partner_bank_id:
            errors.append(_("The supplier %s must have a bank account.", supplier.display_name))

        if invoice.invoice_line_ids.filtered(lambda l: not (l.product_id.name or l.name)):
            errors.append(_('Each invoice line must have a product or a label.'))

        if invoice.invoice_line_ids.tax_ids.invoice_repartition_line_ids.filtered(lambda r: r.use_in_tax_closing) and \
           not supplier.vat:
            errors.append(_("When vat is present, the supplier must have a vat number."))

        return errors

    def _is_compatible_with_journal(self, journal):
        self.ensure_one()
        if self.code != 'nlcius_1' or self._is_account_edi_ubl_cii_available():
            return super()._is_compatible_with_journal(journal)
        return journal.type == 'sale' and journal.country_code == 'NL'

    def _post_invoice_edi(self, invoices):
        self.ensure_one()

        if self.code != 'nlcius_1' or self._is_account_edi_ubl_cii_available():
            return super()._post_invoice_edi(invoices)

        invoice = invoices  # no batch ensure that there is only one invoice
        attachment = self._export_nlcius(invoice)
        return {invoice: {'success': True, 'attachment': attachment}}

    def _create_invoice_from_xml_tree(self, filename, tree, journal=None):
        self.ensure_one()
        if self.code == 'nlcius_1' and self._is_nlcius(filename, tree) and not self._is_account_edi_ubl_cii_available():
            return self._decode_bis3(tree, self.env['account.move'])
        return super()._create_invoice_from_xml_tree(filename, tree, journal=journal)

    def _update_invoice_from_xml_tree(self, filename, tree, invoice):
        self.ensure_one()
        if self.code == 'nlcius_1' and self._is_nlcius(filename, tree) and not self._is_account_edi_ubl_cii_available():
            return self._decode_bis3(tree, invoice)
        return super()._update_invoice_from_xml_tree(filename, tree, invoice)

    def _is_required_for_invoice(self, invoice):
        self.ensure_one()
        if self.code != 'nlcius_1' or self._is_account_edi_ubl_cii_available():
            return super()._is_required_for_invoice(invoice)

        return invoice.commercial_partner_id.country_code in COUNTRY_EAS

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-

from . import account_edi_format

```

## File: test_xml_file\nlcius_test.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
            <Invoice xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2" xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2" xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2">
                <cbc:CustomizationID>urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0</cbc:CustomizationID>
                <cbc:ProfileID>urn:fdc:peppol.eu:2017:poacc:billing:01:1.0</cbc:ProfileID>
                <cbc:ID>INV/2020/11/0001</cbc:ID>
                <cbc:IssueDate>2020-11-02</cbc:IssueDate>
                <cbc:DueDate>2020-11-02</cbc:DueDate>
                <cbc:InvoiceTypeCode>380</cbc:InvoiceTypeCode>
                
                <cbc:DocumentCurrencyCode>EUR</cbc:DocumentCurrencyCode>
                
                <cbc:BuyerReference>Azure Interior</cbc:BuyerReference>
                
                <cac:AccountingSupplierParty>
            <cac:Party>
                <cbc:EndpointID schemeID="0106">77777677</cbc:EndpointID>
                <cac:PartyIdentification>
                    <cbc:ID>77777677</cbc:ID>
                </cac:PartyIdentification>
                <cac:PostalAddress>
                    <cbc:StreetName>De Silva St 4557</cbc:StreetName>
                    
                    <cbc:CityName>Fremont</cbc:CityName>
                    <cbc:PostalZone>94538</cbc:PostalZone>
                    <cac:Country>
                        <cbc:IdentificationCode>NL</cbc:IdentificationCode>
                    </cac:Country>
                </cac:PostalAddress>
                <cac:PartyTaxScheme>
                    <cbc:CompanyID>NL000021504B09</cbc:CompanyID>
                    <cac:TaxScheme>
                        <cbc:ID>VAT</cbc:ID>
                    </cac:TaxScheme>
                </cac:PartyTaxScheme>
                <cac:PartyLegalEntity>
                    <cbc:RegistrationName>partner_a</cbc:RegistrationName>
                    <cbc:CompanyID schemeID="0106">77777677</cbc:CompanyID>
                </cac:PartyLegalEntity>
            </cac:Party>
        </cac:AccountingSupplierParty>
                <cac:AccountingCustomerParty>
            <cac:Party>
                <cbc:EndpointID schemeID="0106">12345678</cbc:EndpointID>
                <cac:PartyIdentification>
                    <cbc:ID>12345678</cbc:ID>
                </cac:PartyIdentification>
                <cac:PartyName>
                    <cbc:Name>NL Company</cbc:Name>
                </cac:PartyName>
                <cac:PostalAddress>
                    <cbc:StreetName>Bloemstraat 42</cbc:StreetName>
                    
                    <cbc:CityName>Groningen</cbc:CityName>
                    <cbc:PostalZone>6700</cbc:PostalZone>
                    <cac:Country>
                        <cbc:IdentificationCode>NL</cbc:IdentificationCode>
                    </cac:Country>
                </cac:PostalAddress>
                <cac:PartyTaxScheme>
                    <cbc:CompanyID>NL000043603B11</cbc:CompanyID>
                    <cac:TaxScheme>
                        <cbc:ID>VAT</cbc:ID>
                    </cac:TaxScheme>
                </cac:PartyTaxScheme>
                <cac:PartyLegalEntity>
                    <cbc:RegistrationName>NL Company</cbc:RegistrationName>
                    <cbc:CompanyID schemeID="0106">12345678</cbc:CompanyID>
                </cac:PartyLegalEntity>
                <cac:Contact>
                    <cbc:Name>NL Company</cbc:Name>
                    <cbc:Telephone>+31 6 12345678</cbc:Telephone>
                    <cbc:ElectronicMail>info@company.nlexample.com</cbc:ElectronicMail>
                  </cac:Contact>
            </cac:Party>
        </cac:AccountingCustomerParty>
                <cac:PaymentMeans>
                    <cbc:PaymentMeansCode>30</cbc:PaymentMeansCode>
                    <cac:PayeeFinancialAccount>
                        <cbc:ID>BE68 5390 0754 7034</cbc:ID>
                    </cac:PayeeFinancialAccount>
                </cac:PaymentMeans>
                
                <cac:TaxTotal>
                    <cbc:TaxAmount currencyID="EUR">67.20</cbc:TaxAmount>
                    
                        <cac:TaxSubtotal>
                            <cbc:TaxableAmount currencyID="EUR">320.00</cbc:TaxableAmount>
                            <cbc:TaxAmount currencyID="EUR">67.20</cbc:TaxAmount>
                            <cac:TaxCategory>
                                <cbc:ID>S</cbc:ID>
                                <cbc:Percent>21.0</cbc:Percent>
                                <cac:TaxScheme>
                                    <cbc:ID>VAT</cbc:ID>
                                </cac:TaxScheme>
                            </cac:TaxCategory>
                        </cac:TaxSubtotal>
                    
                </cac:TaxTotal>
                
                <cac:LegalMonetaryTotal>
                    <cbc:LineExtensionAmount currencyID="EUR">320.00</cbc:LineExtensionAmount>
                    <cbc:TaxExclusiveAmount currencyID="EUR">320.00</cbc:TaxExclusiveAmount>
                    <cbc:TaxInclusiveAmount currencyID="EUR">387.20</cbc:TaxInclusiveAmount>
                    <cbc:PrepaidAmount currencyID="EUR">0.00</cbc:PrepaidAmount>
                    <cbc:PayableAmount currencyID="EUR">387.20</cbc:PayableAmount>
                </cac:LegalMonetaryTotal>
                
                
            <cac:InvoiceLine>
                <cbc:ID>1</cbc:ID>
                
                
                <cbc:InvoicedQuantity unitCode="ZZ">1.0</cbc:InvoicedQuantity>
                <cbc:LineExtensionAmount currencyID="EUR">320.00</cbc:LineExtensionAmount>
                <cac:Item>
                    <cbc:Description>[E-COM07] Large Cabinet</cbc:Description>
                    <cbc:Name>Large Cabinet</cbc:Name>
                    <cac:SellersItemIdentification>
                        <cbc:ID>E-COM07</cbc:ID>
                    </cac:SellersItemIdentification>
                    
                        <cac:ClassifiedTaxCategory>
                            <cbc:ID>S</cbc:ID>
                            <cbc:Percent>21.0</cbc:Percent>
                            <cac:TaxScheme>
                                <cbc:ID>VAT</cbc:ID>
                            </cac:TaxScheme>
                        </cac:ClassifiedTaxCategory>
                    
                    
                </cac:Item>
                <cac:Price>
                    <cbc:PriceAmount currencyID="EUR">320.00</cbc:PriceAmount>
                    <cbc:BaseQuantity>1.0</cbc:BaseQuantity>
                </cac:Price>
            </cac:InvoiceLine>
        
            </Invoice>

```

