# Odoo Module: l10n_be_edi

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Belgium - E-Invoicing (UBL 2.0, e-fff)',
    'version': '0.1',
    'category': 'Hidden',
    'summary': 'E-Invoicing, Universal Business Language (UBL 2.0), e-fff protocol',
    'description': """
Universal Business Language (UBL <http://ubl.xml.org/>`_) is a library of standard electronic XML business documents such as
invoices. The UBL standard became the `ISO/IEC 19845
<http://www.iso.org/iso/catalogue_detail.htm?csnumber=66370>`_ standard in January 2016
(cf the `official announce <http://www.prweb.com/releases/2016/01/prweb13186919.htm>`_).
Belgian e-invoicing uses the UBL 2.0 using the e-fff protocol.
    """,
    'depends': ['account', 'l10n_be', 'account_facturx'],
    'data': [
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, tools, _
from odoo.tests.common import Form
from odoo.osv import expression

import logging
_logger = logging.getLogger(__name__)


class AccountMove(models.Model):
    _inherit = 'account.move'

    @api.model
    def _get_ubl_namespaces(self, tree):
        ''' If the namespace is declared with xmlns='...', the namespaces map contains the 'None' key that causes an
        TypeError: empty namespace prefix is not supported in XPath
        Then, we need to remap arbitrarily this key.

        :param tree: An instance of etree.
        :return: The namespaces map without 'None' key.
        '''
        namespaces = tree.nsmap
        namespaces['inv'] = namespaces.pop(None)
        return namespaces

    @api.model
    def _detect_ubl_2_1(self, tree, file_name):
        # Quick check the tree looks like an UBL 2.1 file.
        flag = tree.tag == '{urn:oasis:names:specification:ubl:schema:xsd:Invoice-2}Invoice'
        error = None

        return {'flag': flag, 'error': error}

    @api.model
    def _decode_ubl_2_1(self, tree):
        self.ensure_one()
        namespaces = self._get_ubl_namespaces(tree)

        elements = tree.xpath('//cbc:InvoiceTypeCode', namespaces=namespaces)
        if elements:
            type_code = elements[0].text
            type = 'in_refund' if type_code == '381' else 'in_invoice'
        else:
            type = 'in_invoice'

        default_journal = self.with_context(default_type=type)._get_default_journal()

        with Form(self.with_context(default_type=type, default_journal_id=default_journal.id)) as invoice_form:
            # Reference
            elements = tree.xpath('//cbc:ID', namespaces=namespaces)
            if elements:
                invoice_form.ref = elements[0].text
            elements = tree.xpath('//cbc:InstructionID', namespaces=namespaces)
            if elements:
                invoice_form.invoice_payment_ref = elements[0].text

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
            elements = tree.xpath('//cbc:DocumentCurrencyCode', namespaces=namespaces)
            currency_code = elements and elements[0].text or ''
            currency = self.env['res.currency'].search([('name', '=', currency_code.upper())], limit=1)
            if elements:
                invoice_form.currency_id = currency

            # Incoterm
            elements = tree.xpath('//cbc:TransportExecutionTerms/cac:DeliveryTerms/cbc:ID', namespaces=namespaces)
            if elements:
                invoice_form.invoice_incoterm_id = self.env['account.incoterms'].search([('code', '=', elements[0].text)], limit=1)

            # Partner
            partner_element = tree.xpath('//cac:AccountingSupplierParty/cac:Party', namespaces=namespaces)
            if partner_element:
                domains = []
                partner_element = partner_element[0]
                elements = partner_element.xpath('//cac:AccountingSupplierParty/cac:Party//cbc:Name', namespaces=namespaces)
                if elements:
                    partner_name = elements[0].text
                    domains.append([('name', 'ilike', partner_name)])
                elements = partner_element.xpath('//cac:AccountingSupplierParty/cac:Party//cbc:Telephone', namespaces=namespaces)
                if elements:
                    partner_telephone = elements[0].text
                    domains.append([('phone', '=', partner_telephone), ('mobile', '=', partner_telephone)])
                elements = partner_element.xpath('//cac:AccountingSupplierParty/cac:Party//cbc:ElectronicMail', namespaces=namespaces)
                if elements:
                    partner_mail = elements[0].text
                    domains.append([('email', '=', partner_mail)])
                elements = partner_element.xpath('//cac:AccountingSupplierParty/cac:Party//cbc:ID', namespaces=namespaces)
                if elements:
                    partner_id = elements[0].text
                    domains.append([('vat', 'like', partner_id)])

                if domains:
                    partner = self.env['res.partner'].search(expression.OR(domains), limit=1)
                    if partner:
                        invoice_form.partner_id = partner
                    else:
                        invoice_form.partner_id = self.env['res.partner']

            # Regenerate PDF
            attachments = self.env['ir.attachment']
            elements = tree.xpath('//cac:AdditionalDocumentReference', namespaces=namespaces)
            for element in elements:
                attachment_name = element.xpath('cbc:ID', namespaces=namespaces)
                attachment_data = element.xpath('cac:Attachment//cbc:EmbeddedDocumentBinaryObject', namespaces=namespaces)
                if attachment_name and attachment_data:
                    attachments |= self.env['ir.attachment'].create({
                        'name': attachment_name[0].text,
                        'res_id': self.id,
                        'res_model': 'account.move',
                        'datas': attachment_data[0].text,
                        'type': 'binary',
                    })
            if attachments:
                self.with_context(no_new_invoice=True).message_post(attachment_ids=attachments.ids)

            # Lines
            lines_elements = tree.xpath('//cac:InvoiceLine', namespaces=namespaces)
            for eline in lines_elements:
                with invoice_form.invoice_line_ids.new() as invoice_line_form:
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
                    invoice_line_form.name = elements and elements[0].text or ''
                    invoice_line_form.name = invoice_line_form.name.replace('%month%', str(fields.Date.to_date(invoice_form.invoice_date).month))  # TODO: full name in locale
                    invoice_line_form.name = invoice_line_form.name.replace('%year%', str(fields.Date.to_date(invoice_form.invoice_date).year))

                    # Product
                    elements = eline.xpath('cac:Item/cac:SellersItemIdentification/cbc:ID', namespaces=namespaces)
                    domains = []
                    if elements:
                        product_code = elements[0].text
                        domains.append([('default_code', '=', product_code)])
                    elements = eline.xpath('cac:Item/cac:StandardItemIdentification/cbc:ID[@schemeID=\'GTIN\']', namespaces=namespaces)
                    if elements:
                        product_ean13 = elements[0].text
                        domains.append([('barcode', '=', product_ean13)])
                    if domains:
                        product = self.env['product.product'].search(expression.OR(domains), limit=1)
                        if product:
                            invoice_line_form.product_id = product

                    # Taxes
                    taxes_elements = eline.xpath('cac:TaxTotal/cac:TaxSubtotal', namespaces=namespaces)
                    invoice_line_form.tax_ids.clear()
                    for etax in taxes_elements:
                        elements = etax.xpath('cbc:Percent', namespaces=namespaces)
                        if elements:
                            tax = self.env['account.tax'].search([
                                ('company_id', '=', self.env.company.id),
                                ('amount', '=', float(elements[0].text)),
                                ('type_tax_use', '=', invoice_form.journal_id.type),
                            ], order='sequence ASC', limit=1)
                            if tax:
                                invoice_line_form.tax_ids.add(tax)

        return invoice_form.save()

    @api.model
    def _get_xml_decoders(self):
        # Override
        ubl_decoders = [('UBL 2.1', self._detect_ubl_2_1, self._decode_ubl_2_1)]
        return super(AccountMove, self)._get_xml_decoders() + ubl_decoders

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_invoice

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Note for later versions, the group Employee Manager should become contract manager -->
    <record id="hr.group_hr_manager" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('fleet.fleet_group_manager'))]"/>
    </record>
</odoo>

```

## File: test_xml_file\efff_test.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Invoice xmlns:qdt="urn:oasis:names:specification:ubl:schema:xsd:QualifiedDatatypes-2" xmlns:ccts="urn:oasis:names:specification:ubl:schema:xsd:CoreComponentParameters-2" xmlns:stat="urn:oasis:names:specification:ubl:schema:xsd:DocumentStatusCode-1.0" xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2" xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2" xmlns:udt="urn:un:unece:uncefact:data:draft:UnqualifiedDataTypesSchemaModule:2" xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2">
    <cbc:UBLVersionID>2.0</cbc:UBLVersionID>
    <cbc:CustomizationID>1.0</cbc:CustomizationID>
    <cbc:ProfileID>FFF.BE KOALABOOX 1.0.0</cbc:ProfileID>
    <cbc:ID>2019/045</cbc:ID>
    <cbc:CopyIndicator>0</cbc:CopyIndicator>
    <cbc:IssueDate>2019-01-31</cbc:IssueDate>
    <cbc:InvoiceTypeCode listURI="http://www.FFF.be/ubl/2.0/cl/gc/BE-InvoiceCode-1.0.gc">380</cbc:InvoiceTypeCode>
    <cbc:Note></cbc:Note>
    <cbc:TaxPointDate>2019-01-31</cbc:TaxPointDate>
    <cbc:DocumentCurrencyCode>EUR</cbc:DocumentCurrencyCode>
    <cbc:LineCountNumeric>1</cbc:LineCountNumeric>
    <cac:AccountingSupplierParty>
        <cac:Party>
            <cbc:EndpointID schemeID="GLN">BE0477472701</cbc:EndpointID>
            <cac:PartyIdentification>
                <cbc:ID schemeAgencyName="KBO" schemeAgencyID="BE">477472701</cbc:ID>
            </cac:PartyIdentification>
            <cac:PartyName>
                <cbc:Name>The best supplier</cbc:Name>
            </cac:PartyName>
            <cac:PostalAddress>
                <cbc:StreetName>Chemin de Odoo 34</cbc:StreetName>
                <cbc:CityName>Odoo Ville</cbc:CityName>
                <cbc:PostalZone>7777</cbc:PostalZone>
                <cac:Country>
                    <cbc:IdentificationCode>BE</cbc:IdentificationCode>
                </cac:Country>
            </cac:PostalAddress>
            <cac:PartyLegalEntity>
                <cbc:RegistrationName>The best supplier</cbc:RegistrationName>
                <cbc:CompanyID>12345689</cbc:CompanyID>
            </cac:PartyLegalEntity>
        </cac:Party>
    </cac:AccountingSupplierParty>
    <cac:AccountingCustomerParty>
        <cbc:CustomerAssignedAccountID></cbc:CustomerAssignedAccountID>
        <cac:Party>
            <cac:PartyIdentification>
                <cbc:ID schemeAgencyName="Just a customer" schemeAgencyID=""/>
            </cac:PartyIdentification>
            <cac:PartyName>
                <cbc:Name>Just a customer</cbc:Name>
            </cac:PartyName>
            <cac:PostalAddress>
                <cbc:StreetName>Customer Street 5 </cbc:StreetName>
                <cbc:CityName>Customer City</cbc:CityName>
                <cbc:PostalZone>6666</cbc:PostalZone>
                <cac:Country>
                    <cbc:IdentificationCode>BE</cbc:IdentificationCode>
                </cac:Country>
            </cac:PostalAddress>
            <cac:Contact>
                <cbc:Name>Just a customer</cbc:Name>
                <cbc:Telephone></cbc:Telephone>
                <cbc:ElectronicMail/>
            </cac:Contact>
        </cac:Party>
    </cac:AccountingCustomerParty>
    <cac:PaymentMeans>
        <cbc:PaymentMeansCode listURI="http://docs.oasis-open.org/ubl/os-UBL-2.0-update/cl/gc/default/PaymentMeansCode-2.0.gc" listName="Payment Means" listID="UN/ECE 4461">1</cbc:PaymentMeansCode>
        <cbc:PaymentDueDate>2019-02-07</cbc:PaymentDueDate>
        <cbc:InstructionID>+++000/0000/00505+++</cbc:InstructionID>
        <cac:PayeeFinancialAccount>
            <cbc:ID></cbc:ID>
        </cac:PayeeFinancialAccount>
    </cac:PaymentMeans>
    <cac:TaxTotal>
        <cbc:TaxAmount currencyID="EUR">115.67</cbc:TaxAmount>
        <cac:TaxSubtotal>
            <cbc:TaxableAmount currencyID="EUR">550.83</cbc:TaxableAmount>
            <cbc:TaxAmount currencyID="EUR">115.67</cbc:TaxAmount>
            <cbc:Percent>21.00</cbc:Percent>
            <cac:TaxCategory>
                <cbc:ID schemeName="Duty or tax or fee category" schemeID="UN/EDIFACT 5305">S</cbc:ID>
                <cbc:Name>03</cbc:Name>
                <cac:TaxScheme>
                    <cbc:ID>VAT</cbc:ID>
                </cac:TaxScheme>
            </cac:TaxCategory>
        </cac:TaxSubtotal>
    </cac:TaxTotal>
    <cac:LegalMonetaryTotal>
        <cbc:LineExtensionAmount currencyID="EUR">550.83</cbc:LineExtensionAmount>
        <cbc:TaxExclusiveAmount currencyID="EUR">550.83</cbc:TaxExclusiveAmount>
        <cbc:TaxInclusiveAmount currencyID="EUR">666.50</cbc:TaxInclusiveAmount>
        <cbc:PayableAmount currencyID="EUR">666.50</cbc:PayableAmount>
    </cac:LegalMonetaryTotal>
    <cac:InvoiceLine>
        <cbc:ID>1</cbc:ID>
        <cbc:Note></cbc:Note>
        <cbc:InvoicedQuantity>1.0000</cbc:InvoicedQuantity>
        <cbc:LineExtensionAmount currencyID="EUR">550.83</cbc:LineExtensionAmount>
        <cac:TaxTotal>
            <cbc:TaxAmount currencyID="EUR">115.67</cbc:TaxAmount>
            <cac:TaxSubtotal>
                <cbc:TaxableAmount currencyID="EUR">550.83</cbc:TaxableAmount>
                <cbc:TaxAmount currencyID="EUR">115.67</cbc:TaxAmount>
                <cbc:Percent>21.00</cbc:Percent>
                <cac:TaxCategory>
                    <cbc:ID schemeName="Duty or tax or fee category" schemeID="UN/EDIFACT 5305">S</cbc:ID>
                    <cbc:Name>03</cbc:Name>
                    <cac:TaxScheme>
                        <cbc:ID>VAT</cbc:ID>
                    </cac:TaxScheme>
                </cac:TaxCategory>
            </cac:TaxSubtotal>
        </cac:TaxTotal>
        <cac:Item>
            <cbc:Description>Loyer du %month% / %year%</cbc:Description>
            <cbc:Name></cbc:Name>
        </cac:Item>
        <cac:Price>
            <cbc:PriceAmount currencyID="EUR">550.83</cbc:PriceAmount>
        </cac:Price>
    </cac:InvoiceLine>
</Invoice>

```

