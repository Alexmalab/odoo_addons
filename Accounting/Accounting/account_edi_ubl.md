# Odoo Module: account_edi_ubl

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
    'name': 'Import/Export invoices with generic UBL',
    'description': '''
    Support for Export/Import in UBL format (2.1).
    ''',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'depends': ['account_edi'],
    'data': [
        'data/ubl_templates.xml',
        'data/account_edi_data.xml',
    ],
    'installable': True,
    'application': False,
    'license': 'LGPL-3',
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="edi_ubl_2_1" model="account.edi.format">
            <field name="name">UBL 2.1</field>
            <field name="code">ubl_2_1</field>
        </record>

    </data>
</odoo>

```

## File: data\ubl_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!--
        Render a res.partner record to be added to the UBL xml document.
        -->
        <template id="export_ubl_invoice_partner">
            <cac:Party
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <t t-set="partner" t-value="partner_vals['partner']"/>
                <cbc:WebsiteURI
                    t-if="partner.website"
                    t-esc="partner.website"/>
                <cac:PartyName>
                    <cbc:Name t-esc="partner.display_name"/>
                </cac:PartyName>
                <cac:Language t-if="partner.lang">
                    <cbc:LocaleCode t-esc="partner.lang"/>
                </cac:Language>
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
                    <cbc:CountrySubentityCode
                        t-if="partner.state_id"
                        t-esc="partner.state_id.code"/>
                    <cac:Country
                        t-if="partner.country_id">
                        <cbc:IdentificationCode t-esc="partner.country_id.code"/>
                        <cbc:Name t-esc="partner.country_id.name"/>
                    </cac:Country>
                </cac:PostalAddress>
                <cac:PartyTaxScheme t-if="partner.commercial_partner_id.vat">
                    <cbc:RegistrationName t-esc="partner.commercial_partner_id.name"/>
                    <cbc:CompanyID t-esc="partner.commercial_partner_id.vat"/>
                    <cac:TaxScheme>
                        <cbc:ID
                            schemeID="UN/ECE 5153"
                            schemeAgencyID="6"
                            >VAT</cbc:ID>
                    </cac:TaxScheme>
                </cac:PartyTaxScheme>
                <cac:Contact>
                    <cbc:Name t-esc="partner.name"/>
                    <cbc:Telephone
                        t-if="partner.phone"
                        t-esc="partner.phone"/>
                    <cbc:ElectronicMail
                        t-if="partner.email"
                        t-esc="partner.email"/>
                </cac:Contact>
            </cac:Party>
        </template>

        <!--
        Render an invoice's line to be added to the UBL xml document.
        -->
        <template id="export_ubl_invoice_line">
            <cac:InvoiceLine
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <t t-set="line" t-value="line_vals['line']"/>
                <cbc:ID t-esc="line.id"/>
                <cbc:Note t-if="line.discount">Discount (<t t-esc='line.discount'/> %)</cbc:Note>
                <cbc:InvoicedQuantity t-esc="line.quantity"/>
                <cbc:LineExtensionAmount
                    t-att-currencyID="line.currency_id.name"
                    t-esc="format_monetary(line.price_subtotal)"/>
                <cac:TaxTotal
                    t-foreach="tax_details['invoice_line_tax_details'][line]['tax_details'].values()"
                    t-as="tax_detail_vals">
                    <cbc:TaxAmount
                        t-att-currencyID="line.currency_id.name"
                        t-esc="format_monetary(-tax_detail_vals['tax_amount_currency'])"/>
                </cac:TaxTotal>
                <cac:Item>
                    <cbc:Description t-if="line.name" t-esc="line.name.replace('\n', ', ')"/>
                    <cbc:Name t-esc="line.product_id.name"/>
                    <cac:SellersItemIdentification t-if="line.product_id.default_code">
                        <cbc:ID t-esc="line.product_id.default_code"/>
                    </cac:SellersItemIdentification>
                </cac:Item>
                <cac:Price>
                    <cbc:PriceAmount
                        t-att-currencyID="line.currency_id.name"
                        t-esc="format_monetary(line.price_unit)"/>
                </cac:Price>
            </cac:InvoiceLine>
        </template>

        <!--
        Render an invoice to be added to the UBL xml document.
        -->
        <template id="export_ubl_invoice">
            <Invoice
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:CustomizationID t-if="customization_id" t-esc="customization_id"/>
                <cbc:ProfileID t-if="profile_id" t-esc="profile_id"/>
                <cbc:UBLVersionID t-if="ubl_version" t-esc="ubl_version"/>
                <cbc:ID t-esc="record.name"/>
                <cbc:IssueDate t-esc="record.invoice_date"/>
                <cbc:InvoiceTypeCode t-esc="type_code"/>
                <cbc:Note t-if="note" t-esc="note"/>
                <cbc:DocumentCurrencyCode t-esc="record.currency_id.name"/>
                <cbc:BuyerReference t-esc="record.commercial_partner_id.name"/>
                <cac:OrderReference t-if="record.ref">
                    <cbc:ID t-esc="record.ref"/>
                </cac:OrderReference>
                <cac:AccountingSupplierParty t-call="account_edi_ubl.export_ubl_invoice_partner">
                    <t t-set="partner_vals" t-value="supplier_vals"/>
                </cac:AccountingSupplierParty>
                <cac:AccountingCustomerParty t-call="account_edi_ubl.export_ubl_invoice_partner">
                    <t t-set="partner_vals" t-value="customer_vals"/>
                </cac:AccountingCustomerParty>
                <cac:PaymentMeans>
                    <cbc:PaymentMeansCode listID="UN/ECE 4461" t-esc="payment_means_code"/>
                    <cbc:PaymentDueDate t-esc="record.invoice_date_due"/>
                    <cbc:InstructionID t-esc="record.payment_reference"/>
                    <cac:PayeeFinancialAccount t-if="record.journal_id.bank_account_id">
                        <cbc:ID schemeName="IBAN" t-esc="record.journal_id.bank_account_id.acc_number"/>
                        <cac:FinancialInstitutionBranch t-if="record.journal_id.bank_account_id.bank_bic">
                            <cac:FinancialInstitution>
                                <cbc:ID schemeName="BIC" t-esc="record.journal_id.bank_account_id.bank_bic"/>
                            </cac:FinancialInstitution>
                        </cac:FinancialInstitutionBranch>
                    </cac:PayeeFinancialAccount>
                </cac:PaymentMeans>
                <cac:PaymentTerms t-if="record.invoice_payment_term_id">
                    <cbc:Note t-esc="record.invoice_payment_term_id.name"/>
                </cac:PaymentTerms>
                <cac:TaxTotal>
                    <cbc:TaxAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(record.amount_tax)"/>
                </cac:TaxTotal>
                <cac:LegalMonetaryTotal>
                    <cbc:LineExtensionAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(record.amount_untaxed)"/>
                    <cbc:TaxExclusiveAmount
                        t-att-currencyID="record.currency_id.name"
                        t-esc="format_monetary(record.amount_untaxed)"/>
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
                    <t t-call="account_edi_ubl.export_ubl_invoice_line"/>
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
from odoo import models, fields
from odoo.tools import float_repr, html2plaintext
from odoo.tests.common import Form

from pathlib import PureWindowsPath

import base64
import logging
import markupsafe

_logger = logging.getLogger(__name__)


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    ####################################################
    # Helpers
    ####################################################

    def _is_ubl(self, filename, tree):
        return tree.tag == '{urn:oasis:names:specification:ubl:schema:xsd:Invoice-2}Invoice'

    ####################################################
    # Import
    ####################################################
    def _create_invoice_from_ubl(self, tree):
        invoice = self.env['account.move']
        journal = invoice._get_default_journal()

        move_type = 'out_invoice' if journal.type == 'sale' else 'in_invoice'
        element = tree.find('.//{*}InvoiceTypeCode')
        if element is not None and element.text == '381':
            move_type = 'in_refund' if move_type == 'in_invoice' else 'out_refund'

        invoice = invoice.with_context(default_move_type=move_type, default_journal_id=journal.id)
        return self._import_ubl(tree, invoice)

    def _update_invoice_from_ubl(self, tree, invoice):
        invoice = invoice.with_context(default_move_type=invoice.move_type, default_journal_id=invoice.journal_id.id)
        return self._import_ubl(tree, invoice)

    def _import_ubl(self, tree, invoice):
        """ Decodes an UBL invoice into an invoice.

        :param tree:    the UBL tree to decode.
        :param invoice: the invoice to update or an empty recordset.
        :returns:       the invoice where the UBL data was imported.
        """

        def _get_ubl_namespaces():
            ''' If the namespace is declared with xmlns='...', the namespaces map contains the 'None' key that causes an
            TypeError: empty namespace prefix is not supported in XPath
            Then, we need to remap arbitrarily this key.

            :param tree: An instance of etree.
            :return: The namespaces map without 'None' key.
            '''
            namespaces = tree.nsmap
            namespaces['inv'] = namespaces.pop(None)
            return namespaces

        namespaces = _get_ubl_namespaces()

        def _find_value(xpath, element=tree):
            return self._find_value(xpath, element, namespaces)

        with Form(invoice) as invoice_form:
            # Reference
            elements = tree.xpath('//cbc:ID', namespaces=namespaces)
            if elements:
                invoice_form.ref = elements[0].text
            elements = tree.xpath('//cbc:InstructionID', namespaces=namespaces)
            if elements:
                invoice_form.payment_reference = elements[0].text

            # Dates
            elements = tree.xpath('//cbc:IssueDate', namespaces=namespaces)
            if elements:
                invoice_form.invoice_date = elements[0].text
            elements = tree.xpath('//cbc:PaymentDueDate', namespaces=namespaces)
            if elements:
                invoice_form.invoice_date_due = elements[0].text
            # allow both cbc:PaymentDueDate and cbc:DueDate
            elements = tree.xpath('//cbc:DueDate', namespaces=namespaces)
            invoice_form.invoice_date_due = invoice_form.invoice_date_due or elements and elements[0].text

            # Currency
            currency = self._retrieve_currency(_find_value('//cbc:DocumentCurrencyCode'))
            if currency and currency.active:
                invoice_form.currency_id = currency

            # Incoterm
            elements = tree.xpath('//cbc:TransportExecutionTerms/cac:DeliveryTerms/cbc:ID', namespaces=namespaces)
            if elements:
                invoice_form.invoice_incoterm_id = self.env['account.incoterms'].search([('code', '=', elements[0].text)], limit=1)

            # Partner
            counterpart = 'Customer' if invoice_form.move_type in ('out_invoice', 'out_refund') else 'Supplier'
            invoice_form.partner_id = self._retrieve_partner(
                name=_find_value(f'//cac:Accounting{counterpart}Party/cac:Party//cbc:Name'),
                phone=_find_value(f'//cac:Accounting{counterpart}Party/cac:Party//cbc:Telephone'),
                mail=_find_value(f'//cac:Accounting{counterpart}Party/cac:Party//cbc:ElectronicMail'),
                vat=_find_value(f'//cac:Accounting{counterpart}Party/cac:Party//cbc:CompanyID'),
            )

            # Lines
            lines_elements = tree.xpath('//cac:InvoiceLine', namespaces=namespaces)
            for eline in lines_elements:
                with invoice_form.invoice_line_ids.new() as invoice_line_form:
                    # Product
                    invoice_line_form.product_id = self._retrieve_product(
                        default_code=_find_value('cac:Item/cac:SellersItemIdentification/cbc:ID', eline),
                        name=_find_value('cac:Item/cbc:Name', eline),
                        barcode=_find_value('cac:Item/cac:StandardItemIdentification/cbc:ID[@schemeID=\'0160\']', eline)
                    )

                    # Quantity
                    elements = eline.xpath('cbc:InvoicedQuantity', namespaces=namespaces)
                    quantity = elements and float(elements[0].text) or 1.0
                    invoice_line_form.quantity = quantity

                    # Price Unit
                    elements = eline.xpath('cac:Price/cbc:PriceAmount', namespaces=namespaces)
                    price_unit = elements and float(elements[0].text) or 0.0
                    elements = eline.xpath('cbc:LineExtensionAmount', namespaces=namespaces)
                    line_extension_amount = elements and float(elements[0].text) or 0.0
                    invoice_line_form.price_unit = price_unit or line_extension_amount / invoice_line_form.quantity or 0.0

                    # Name
                    elements = eline.xpath('cac:Item/cbc:Description', namespaces=namespaces)
                    if elements and elements[0].text:
                        invoice_line_form.name = elements[0].text
                        invoice_line_form.name = invoice_line_form.name.replace('%month%', str(fields.Date.to_date(invoice_form.invoice_date).month))  # TODO: full name in locale
                        invoice_line_form.name = invoice_line_form.name.replace('%year%', str(fields.Date.to_date(invoice_form.invoice_date).year))
                    else:
                        partner_name = _find_value('//cac:AccountingSupplierParty/cac:Party//cbc:Name')
                        invoice_line_form.name = "%s (%s)" % (partner_name or '', invoice_form.invoice_date)

                    # Taxes
                    tax_element = eline.xpath('cac:TaxTotal/cac:TaxSubtotal', namespaces=namespaces)
                    invoice_line_form.tax_ids.clear()
                    for eline in tax_element:
                        tax = self._retrieve_tax(
                            amount=_find_value('cbc:Percent', eline),
                            type_tax_use=invoice_form.journal_id.type
                        )
                        if tax:
                            invoice_line_form.tax_ids.add(tax)
        invoice = invoice_form.save()

        # Regenerate PDF
        attachments = self.env['ir.attachment']
        elements = tree.xpath('//cac:AdditionalDocumentReference', namespaces=namespaces)
        for element in elements:
            attachment_name = element.xpath('cbc:ID', namespaces=namespaces)
            attachment_data = element.xpath('cac:Attachment//cbc:EmbeddedDocumentBinaryObject', namespaces=namespaces)
            if attachment_name and attachment_data:
                text = attachment_data[0].text
                # Normalize the name of the file : some e-fff emitters put the full path of the file
                # (Windows or Linux style) and/or the name of the xml instead of the pdf.
                # Get only the filename with a pdf extension.
                name = PureWindowsPath(attachment_name[0].text).stem + '.pdf'
                attachments |= self.env['ir.attachment'].create({
                    'name': name,
                    'res_id': invoice.id,
                    'res_model': 'account.move',
                    'datas': text + '=' * (len(text) % 3),  # Fix incorrect padding
                    'type': 'binary',
                    'mimetype': 'application/pdf',
                })
        if attachments:
            invoice.with_context(no_new_invoice=True).message_post(attachment_ids=attachments.ids)

        return invoice

    ####################################################
    # Export
    ####################################################

    def _get_ubl_values(self, invoice):
        ''' Get the necessary values to generate the XML. These values will be used in the qweb template when
        rendering. Needed values differ depending on the implementation of the UBL, as (sub)template can be overriden
        or called dynamically.
        :returns:   a dictionary with the value used in the template has key and the value as value.
        '''
        def format_monetary(amount):
            # Format the monetary values to avoid trailing decimals (e.g. 90.85000000000001).
            return float_repr(amount, invoice.currency_id.decimal_places)

        return {
            **invoice._prepare_edi_vals_to_export(),
            'tax_details': invoice._prepare_edi_tax_details(),
            'ubl_version': 2.1,
            'type_code': 380 if invoice.move_type == 'out_invoice' else 381,
            'payment_means_code': 42 if invoice.journal_id.bank_account_id else 31,
            'bank_account': invoice.partner_bank_id,
            'note': html2plaintext(invoice.narration) if invoice.narration else False,
            'format_monetary': format_monetary,
            'customer_vals': {'partner': invoice.partner_id},
            'supplier_vals': {'partner': invoice.company_id.partner_id.commercial_partner_id},
        }

    def _export_ubl(self, invoice):
        self.ensure_one()
        # Create file content.
        xml_content = markupsafe.Markup("<?xml version='1.0' encoding='UTF-8'?>")
        xml_content += self.env.ref('account_edi_ubl.export_ubl_invoice')._render(self._get_ubl_values(invoice))
        xml_name = '%s_ubl_2_1.xml' % (invoice.name.replace('/', '_'))
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

    def _create_invoice_from_xml_tree(self, filename, tree, journal=None):
        # OVERRIDE
        self.ensure_one()
        if self.code == 'ubl_2_1' and self._is_ubl(filename, tree) and not self._is_account_edi_ubl_cii_available():
            return self._create_invoice_from_ubl(tree)
        return super()._create_invoice_from_xml_tree(filename, tree, journal=journal)

    def _update_invoice_from_xml_tree(self, filename, tree, invoice):
        # OVERRIDE
        self.ensure_one()
        if self.code == 'ubl_2_1' and self._is_ubl(filename, tree) and not self._is_account_edi_ubl_cii_available():
            return self._update_invoice_from_ubl(tree, invoice)
        return super()._update_invoice_from_xml_tree(filename, tree, invoice)

    def _is_compatible_with_journal(self, journal):
        # OVERRIDE
        self.ensure_one()
        if self.code != 'ubl_2_1':
            return super()._is_compatible_with_journal(journal)
        return journal.type == 'sale'

    def _is_enabled_by_default_on_journal(self, journal):
        # OVERRIDE
        # UBL is disabled by default to prevent conflict with other implementations of UBL.
        self.ensure_one()
        if self.code != 'ubl_2_1':
            return super()._is_enabled_by_default_on_journal(journal)
        return False

    def _post_invoice_edi(self, invoices):
        # OVERRIDE
        self.ensure_one()
        if self.code != 'ubl_2_1' or self._is_account_edi_ubl_cii_available():
            return super()._post_invoice_edi(invoices)
        res = {}
        for invoice in invoices:
            attachment = self._export_ubl(invoice)
            res[invoice] = {'success': True, 'attachment': attachment}
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_edi_format

```

