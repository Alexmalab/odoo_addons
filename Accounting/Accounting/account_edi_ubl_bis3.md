# Odoo Module: account_edi_ubl_bis3

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Import/Export invoices with UBL (BIS3)',
    'description': '''
    Support for Export/Import in UBL format (BIS3).
    ''',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'depends': ['account_edi_ubl'],
    'data': [
        'data/bis3_templates.xml',
    ],
    'installable': True,
    'application': False,
    'license': 'LGPL-3',
}

```

## File: data\bis3_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!--
            res.partner
        -->
        <template id="export_bis3_invoice_partner">
            <cac:Party
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <t t-set="partner" t-value="partner_vals['partner']"/>
                <cbc:EndpointID
                        t-if="partner_vals.get('bis3_endpoint')"
                        t-att-schemeID="partner_vals['bis3_endpoint_scheme']"
                        t-esc="partner_vals['bis3_endpoint']"/>
                <cac:PartyIdentification t-if="partner_vals.get('partner_identification')">
                    <cbc:ID t-esc="partner_vals['partner_identification']"/>
                </cac:PartyIdentification>
                <cac:PartyName>
                    <cbc:Name t-esc="partner.display_name"/>
                </cac:PartyName>
                <cac:PostalAddress>
                    <cbc:StreetName
                        t-if="partner.street"
                        t-esc="partner.street"/>
                    <cbc:AdditionalStreetName
                        t-if="partner.street2"
                        t-esc="partner.street2"/>
                    <cbc:CityName
                        t-if="partner.city"
                        t-esc="partner.city"/>
                    <cbc:PostalZone
                        t-if="partner.zip"
                        t-esc="partner.zip"/>
                    <cbc:CountrySubentity
                        t-if="partner.state_id"
                        t-esc="partner.state_id.name"/>
                    <cac:Country
                        t-if="partner.country_id">
                        <cbc:IdentificationCode t-esc="partner.country_id.code"/>
                    </cac:Country>
                </cac:PostalAddress>
                <cac:PartyTaxScheme t-if="partner.commercial_partner_id.vat">
                    <cbc:CompanyID t-esc="partner.commercial_partner_id.vat"/>
                    <cac:TaxScheme>
                        <cbc:ID>VAT</cbc:ID>
                    </cac:TaxScheme>
                </cac:PartyTaxScheme>
                <cac:PartyLegalEntity>
                    <cbc:RegistrationName t-esc="partner.name"/>
                    <cbc:CompanyID
                            t-if="partner_vals.get('legal_entity')"
                            t-att-schemeID="partner_vals['legal_entity_scheme']"
                            t-esc="partner_vals['legal_entity']"/>
                </cac:PartyLegalEntity>
                <cac:Contact>
                    <cbc:Name t-esc="partner.name"/>
                    <cbc:Telephone t-if="partner.phone" t-esc="partner.phone"/>
                    <cbc:ElectronicMail t-if="partner.email" t-esc="partner.email"/>
                  </cac:Contact>
            </cac:Party>
        </template>

        <!--
        account.move.line
        -->
        <template id="export_bis3_invoice_line">
            <cac:InvoiceLine
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <t t-set="line" t-value="line_vals['line']"/>
                <cbc:ID t-esc="line_vals['index']"/>
                <cbc:Note t-if="line.discount">Discount (<t t-esc='line.discount'/> %)</cbc:Note>
                <cbc:InvoicedQuantity unitCode="ZZ" t-esc="line.quantity"/>
                <cbc:LineExtensionAmount
                    t-att-currencyID="record.currency_id.name"
                    t-esc="format_monetary(line_vals['price_subtotal_with_no_tax_closing'])"/>
                <cac:Item>
                    <cbc:Description t-esc="line.name.replace('\n', ', ')"/>
                    <cbc:Name t-esc="(line.name or '').replace('\n', ', ')"/>
                    <cac:SellersItemIdentification t-if="line.product_id.default_code">
                        <cbc:ID t-esc="line.product_id.default_code"/>
                    </cac:SellersItemIdentification>
                    <cac:ClassifiedTaxCategory
                            t-foreach="tax_details['invoice_line_tax_details'][line]['tax_details'].values()"
                            t-as="tax_detail_vals">
                        <cbc:ID t-esc="tax_detail_vals['tax_category']"/>
                        <cbc:Percent t-esc="tax_detail_vals['tax_percent']"/>
                        <cac:TaxScheme>
                            <cbc:ID>VAT</cbc:ID>
                        </cac:TaxScheme>
                    </cac:ClassifiedTaxCategory>
                </cac:Item>
                <cac:Price>
                    <cbc:PriceAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(line_vals['price_subtotal_with_no_tax_closing'] / line.quantity)"/>
                    <cbc:BaseQuantity t-esc="line.quantity"/>
                </cac:Price>
            </cac:InvoiceLine>
        </template>

        <!--
        invoice
        -->
        <template id="export_bis3_invoice">
            <Invoice
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:CustomizationID t-esc="customization_id"/>
                <cbc:ProfileID t-esc="profile_id"/>
                <cbc:ID t-esc="record.name"/>
                <cbc:IssueDate t-esc="record.invoice_date"/>
                <cbc:DueDate t-esc="record.invoice_date_due"/>
                <cbc:InvoiceTypeCode t-esc="type_code"/>
                <cbc:Note t-if="note" t-esc="note"/>
                <cbc:DocumentCurrencyCode t-esc="record.currency_id.name"/>
                <cbc:TaxCurrencyCode t-if="record.currency_id != record.company_currency_id"
                                     t-esc="record.company_currency_id.name"/>
                <cbc:BuyerReference t-esc="record.commercial_partner_id.name"/>
                <cac:OrderReference t-if="record.ref">
                    <cbc:ID t-esc="record.ref"/>
                </cac:OrderReference>
                <cac:AccountingSupplierParty t-call="account_edi_ubl_bis3.export_bis3_invoice_partner">
                    <t t-set="partner_vals" t-value="supplier_vals"/>
                </cac:AccountingSupplierParty>
                <cac:AccountingCustomerParty t-call="account_edi_ubl_bis3.export_bis3_invoice_partner">
                    <t t-set="partner_vals" t-value="customer_vals"/>
                </cac:AccountingCustomerParty>
                <cac:PaymentMeans>
                    <cbc:PaymentMeansCode t-esc="payment_means_code"/>
                    <cac:PayeeFinancialAccount t-if="bank_account">
                        <cbc:ID t-esc="bank_account.acc_number"/>
                    </cac:PayeeFinancialAccount>
                </cac:PaymentMeans>
                <cac:PaymentTerms t-if="record.invoice_payment_term_id">
                    <cbc:Note t-esc="record.invoice_payment_term_id.name"/>
                </cac:PaymentTerms>
                <cac:TaxTotal>
                    <cbc:TaxAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(balance_multiplicator * tax_details['tax_amount_currency'])"/>
                    <cac:TaxSubtotal t-foreach="tax_details['tax_details'].values()" t-as="tax_detail_vals">
                        <cbc:TaxableAmount
                                t-att-currencyID="record.currency_id.name"
                                t-esc="format_monetary(balance_multiplicator * tax_detail_vals['base_amount_currency'])"/>
                        <cbc:TaxAmount
                                t-att-currencyID="record.currency_id.name"
                                t-esc="format_monetary(balance_multiplicator * tax_detail_vals['tax_amount_currency'])"/>
                        <cac:TaxCategory>
                            <cbc:ID t-esc="tax_detail_vals['tax_category']"/>
                            <cbc:Percent t-esc="tax_detail_vals['tax_percent']"/>
                            <cac:TaxScheme>
                                <cbc:ID>VAT</cbc:ID>
                            </cac:TaxScheme>
                        </cac:TaxCategory>
                    </cac:TaxSubtotal>
                </cac:TaxTotal>
                <cac:TaxTotal t-if="record.currency_id != record.company_currency_id">
                    <cbc:TaxAmount
                        t-att-currencyID="record.company_currency_id.name"
                        t-esc="format_monetary(balance_multiplicator * tax_details['tax_amount'])"/>
                </cac:TaxTotal>
                <cac:LegalMonetaryTotal>
                    <cbc:LineExtensionAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(total_untaxed_amount)"/>
                    <cbc:TaxExclusiveAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(total_untaxed_amount)"/>
                    <cbc:TaxInclusiveAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(record.amount_total)"/>
                    <cbc:PrepaidAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(record.amount_total - record.amount_residual)"/>
                    <cbc:PayableAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(record.amount_residual)"/>
                </cac:LegalMonetaryTotal>
                <t t-foreach="invoice_line_vals_list" t-as="line_vals">
                    <t t-call="account_edi_ubl_bis3.export_bis3_invoice_line"/>
                </t>
            </Invoice>
        </template>
    </data>
</odoo>

```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.exceptions import UserError
from odoo.tests.common import Form


COUNTRY_EAS = {
    'HU': 9910,

    'AD': 9922,
    'AL': 9923,
    'BA': 9924,
    'BE': 9925,
    'BG': 9926,
    'CH': 9927,
    'CY': 9928,
    'CZ': 9929,
    'DE': 9930,
    'EE': 9931,
    'UK': 9932,
    'GR': 9933,
    'HR': 9934,
    'IE': 9935,
    'LI': 9936,
    'LT': 9937,
    'LU': 9938,
    'LV': 9939,
    'MC': 9940,
    'ME': 9941,
    'MK': 9942,
    'MT': 9943,
    'NL': 9944,
    'PL': 9945,
    'PT': 9946,
    'RO': 9947,
    'RS': 9948,
    'SI': 9949,
    'SK': 9950,
    'SM': 9951,
    'TR': 9952,
    'VA': 9953,

    'SE': '0007',

    'FR': 9957
}


class AccountEdiFormat(models.Model):
    ''' This edi_format is "abstract" meaning that it provides an additional layer for similar edi_format (formats
    deriving from EN16931) that share some functionalities but needs to be extended to be used.
    '''
    _inherit = 'account.edi.format'

    ####################################################
    # Export
    ####################################################

    def _get_bis3_values(self, invoice):
        values = super()._get_ubl_values(invoice)
        values.update({
            'customization_id': 'urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0',
            'profile_id': 'urn:fdc:peppol.eu:2017:poacc:billing:01:1.0',
        })

        # Tax details.

        def grouping_key_generator(tax_values):
            tax = tax_values['tax_id']
            return {
                'tax_percent': tax.amount,
                'tax_category': 'S' if tax.amount else 'Z',
            }

        values['tax_details'] = invoice._prepare_edi_tax_details(
            filter_to_apply=lambda x: x['tax_repartition_line_id'].use_in_tax_closing,
            grouping_key_generator=grouping_key_generator,
        )
        for line_vals in values['invoice_line_vals_list']:
            if len(values['tax_details']['invoice_line_tax_details'][line_vals['line']]['tax_details']) > 1:
                raise UserError("Multiple vat percentage not supported on the same invoice line")

        tax_details_no_tax_closing = invoice._prepare_edi_tax_details(
            filter_to_apply=lambda x: not x['tax_repartition_line_id'].use_in_tax_closing,
        )
        for line_vals in values['invoice_line_vals_list']:
            line_vals['price_subtotal_with_no_tax_closing'] = line_vals['line'].price_subtotal
            for tax_detail in tax_details_no_tax_closing['invoice_line_tax_details'][line_vals['line']]['tax_details'].values():
                line_vals['price_subtotal_with_no_tax_closing'] += tax_detail['tax_amount_currency']

        values['total_untaxed_amount'] = sum(x['price_subtotal_with_no_tax_closing'] for x in values['invoice_line_vals_list'])

        # Misc.

        for partner_vals in (values['customer_vals'], values['supplier_vals']):
            partner = partner_vals['partner'].commercial_partner_id
            if partner.country_id.code in COUNTRY_EAS:
                partner_vals['bis3_endpoint'] = partner.vat
                partner_vals['bis3_endpoint_scheme'] = COUNTRY_EAS[partner.country_id.code]

        return values

    ####################################################
    # Import
    ####################################################

    def _get_bis3_namespaces(self):
        return {
            'cac': 'urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2',
            'cbc': 'urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2',
        }

    def _bis3_get_extra_partner_domains(self, tree):
        """ Returns an additional domain to find the partner of the invoice based on specific implementation of BIS3.
        TO OVERRIDE

        :returns: a list of domains
        """
        return []

    def _decode_bis3(self, tree, invoice):
        """ Decodes an EN16931 invoice into an invoice.
        :param tree:    the UBL (EN16931) tree to decode.
        :param invoice: the invoice to update or an empty recordset.
        :returns:       the invoice where the UBL (EN16931) data was imported.
        """
        def _find_value(path, root=tree):
            element = root.find(path)
            return element.text if element is not None else None

        element = tree.find('./{*}InvoiceTypeCode')
        if element is not None:
            type_code = element.text
            move_type = 'in_refund' if type_code == '381' else 'in_invoice'
        else:
            move_type = 'in_invoice'

        default_journal = invoice.with_context(default_move_type=move_type)._get_default_journal()

        with Form(invoice.with_context(default_move_type=move_type, default_journal_id=default_journal.id)) as invoice_form:
            # Reference
            element = tree.find('./{*}ID')
            if element is not None:
                invoice_form.ref = element.text

            # Dates
            element = tree.find('./{*}IssueDate')
            if element is not None:
                invoice_form.invoice_date = element.text
            element = tree.find('./{*}DueDate')
            if element is not None:
                invoice_form.invoice_date_due = element.text

            # Currency
            currency = self._retrieve_currency(_find_value('./{*}DocumentCurrencyCode'))
            if currency and currency.active:
                invoice_form.currency_id = currency

            # Partner
            specific_domain = self._bis3_get_extra_partner_domains(tree)
            invoice_form.partner_id = self._retrieve_partner(
                name=_find_value('./{*}AccountingSupplierParty/{*}Party/*/{*}Name'),
                phone=_find_value('./{*}AccountingSupplierParty/{*}Party/*/{*}Telephone'),
                mail=_find_value('./{*}AccountingSupplierParty/{*}Party/*/{*}ElectronicMail'),
                vat=_find_value('./{*}AccountingSupplierParty/{*}Party/{*}PartyTaxScheme/{*}CompanyID'),
                domain=specific_domain,
            )

            # Lines
            for eline in tree.findall('.//{*}InvoiceLine'):
                with invoice_form.invoice_line_ids.new() as invoice_line_form:
                    # Product
                    invoice_line_form.product_id = self._retrieve_product(
                        default_code=_find_value('./{*}Item/{*}SellersItemIdentification/{*}ID', eline),
                        name=_find_value('./{*}Item/{*}Name', eline),
                        barcode=_find_value('./{*}Item/{*}StandardItemIdentification/{*}ID[@schemeID=\'0160\']', eline)
                    )

                    # Quantity
                    element = eline.find('./{*}InvoicedQuantity')
                    quantity = element is not None and float(element.text) or 1.0
                    invoice_line_form.quantity = quantity

                    # Price Unit
                    element = eline.find('./{*}Price/{*}PriceAmount')
                    price_unit = element is not None and float(element.text) or 0.0
                    line_extension_amount = element is not None and float(element.text) or 0.0
                    invoice_line_form.price_unit = price_unit or line_extension_amount / invoice_line_form.quantity or 0.0

                    # Name
                    element = eline.find('./{*}Item/{*}Description')
                    invoice_line_form.name = element is not None and element.text or ''

                    # Taxes
                    tax_elements = eline.findall('./{*}Item/{*}ClassifiedTaxCategory')
                    invoice_line_form.tax_ids.clear()
                    for tax_element in tax_elements:
                        invoice_line_form.tax_ids.add(self._retrieve_tax(
                            amount=_find_value('./{*}Percent', tax_element),
                            type_tax_use=invoice_form.journal_id.type
                        ))

        return invoice_form.save()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_edi_format

```

