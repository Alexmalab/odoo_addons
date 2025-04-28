# Odoo Module: account_facturx

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name' : 'Import Bills/Invoices From XML',
    'version' : '1.0',
    'category': 'Accounting/Accounting',
    'depends' : ['account'],
    'data': [
        'data/facturx_templates.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\facturx_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="account_invoice_line_facturx_export">
            <t t-set="line" t-value="line_values['line']"/>
            <t  xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100"
                xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100"
                xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100">
                <ram:IncludedSupplyChainTradeLineItem>
                    <!-- Line number. -->
                    <ram:AssociatedDocumentLineDocument>
                        <ram:LineID t-esc="line_values['index']"/>
                    </ram:AssociatedDocumentLineDocument>

                    <!-- Product. -->
                    <ram:SpecifiedTradeProduct>
                        <ram:GlobalID
                            schemeID="0160"
                            t-if="line.product_id and line.product_id.barcode"
                            t-esc="line.product_id.barcode"/>
                        <ram:SellerAssignedID
                            t-if="line.product_id and line.product_id.default_code"
                            t-esc="line.product_id.default_code"/>
                        <ram:Name t-esc="line.name"/>
                    </ram:SpecifiedTradeProduct>

                    <!-- Amounts. -->
                    <ram:SpecifiedLineTradeAgreement>
                        <!-- Line information, with discount and unit price separate -->
                        <t t-set="net_amount" t-value="line.price_subtotal/line.quantity if line.quantity else 0"/>
                        <ram:GrossPriceProductTradePrice>
                            <ram:ChargeAmount
                                t-esc="format_monetary(line.price_unit, currency)"/>

                            <!-- Discount. -->
                            <ram:AppliedTradeAllowanceCharge t-if="line.discount">
                                <ram:ChargeIndicator>
                                    <udt:Indicator>false</udt:Indicator>
                                </ram:ChargeIndicator>
                                <ram:ActualAmount t-esc="format_monetary(line_values['price_discount_unit'], currency)"/>
                            </ram:AppliedTradeAllowanceCharge>
                        </ram:GrossPriceProductTradePrice>
                        <!-- Line unit price, with discount applied -->
                        <ram:NetPriceProductTradePrice>
                            <ram:ChargeAmount
                                t-esc="format_monetary(net_amount, currency)"/>
                        </ram:NetPriceProductTradePrice>
                    </ram:SpecifiedLineTradeAgreement>

                    <!-- Quantity. -->
                    <ram:SpecifiedLineTradeDelivery>
                        <ram:BilledQuantity t-att-unitCode="line_values['unece_uom_code']" t-esc="line.quantity"/>
                    </ram:SpecifiedLineTradeDelivery>

                    <ram:SpecifiedLineTradeSettlement>
                        <t t-foreach="line_values['tax_details']" t-as="tax_vals">
                            <t t-set="tax" t-value="tax_vals['tax']"/>
                            <ram:ApplicableTradeTax t-if="tax.amount_type == 'percent'">
                                <ram:TypeCode>VAT</ram:TypeCode>
                                <ram:CategoryCode t-esc="tax_vals['unece_tax_category_code']"/>
                                <ram:RateApplicablePercent t-esc="tax.amount"/>
                            </ram:ApplicableTradeTax>
                        </t>
                        <!-- Subtotal. -->
                        <ram:SpecifiedTradeSettlementLineMonetarySummation>
                            <ram:LineTotalAmount
                                t-esc="format_monetary(line.price_subtotal, currency)"/>
                        </ram:SpecifiedTradeSettlementLineMonetarySummation>
                    </ram:SpecifiedLineTradeSettlement>

                </ram:IncludedSupplyChainTradeLineItem>
            </t>
        </template>

        <template id="account_invoice_partner_facturx_export">
            <t  xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100"
                xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100"
                xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100">
                <!-- Contact. -->
                <ram:Name t-if="partner.name" t-esc="partner.name"/>

                <ram:SpecifiedLegalOrganization t-if="SpecifiedLegalOrganization">
                    <ram:ID t-esc="SpecifiedLegalOrganization"/>
                </ram:SpecifiedLegalOrganization>

                <ram:DefinedTradeContact>
                    <ram:PersonName t-esc="partner.name"/>
                    <ram:TelephoneUniversalCommunication t-if="partner.phone or partner.mobile">
                        <ram:CompleteNumber t-esc="partner.phone or partner.mobile"/>
                    </ram:TelephoneUniversalCommunication>
                    <ram:EmailURIUniversalCommunication t-if="partner.email">
                        <ram:URIID schemeID='SMTP' t-esc="partner.email"/>
                    </ram:EmailURIUniversalCommunication>
                </ram:DefinedTradeContact>

                <!-- Address. -->
                <ram:PostalTradeAddress>
                    <ram:PostcodeCode t-if="partner.zip" t-esc="partner.zip"/>
                    <ram:LineOne t-if="partner.street" t-esc="partner.street"/>
                    <ram:LineTwo t-if="partner.street2" t-esc="partner.street2"/>
                    <ram:CityName t-if="partner.city" t-esc="partner.city"/>
                    <ram:CountryID t-if="partner.country_id" t-esc="partner.country_id.code"/>
                </ram:PostalTradeAddress>
            </t>
        </template>

        <template id="account_invoice_facturx_export">
            <t t-set="currency" t-value="record.currency_id or record.company_currency_id"/>
            <rsm:CrossIndustryInvoice
                xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100"
                xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100"
                xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100">
                <!-- Factur-x level:
                    * minimum or basicwl:   urn:factur-x.eu:1p0...
                    * basic:                urn:cen.eu:en16931:2017:compliant:factur-x.eu:1p0:basic
                    * en16931:              urn:cen.eu:en16931:2017
                -->
                <rsm:ExchangedDocumentContext>
                    <ram:GuidelineSpecifiedDocumentContextParameter>
                        <ram:ID>urn:cen.eu:en16931:2017#conformant#urn:factur-x.eu:1p0:extended</ram:ID>
                    </ram:GuidelineSpecifiedDocumentContextParameter>
                </rsm:ExchangedDocumentContext>

                <!-- Document Headers. -->
                <rsm:ExchangedDocument>
                    <ram:ID t-esc="record.name"/>
                    <ram:TypeCode t-esc="'381' if 'refund' in record.type else '380'"/>
                    <ram:IssueDateTime>
                        <udt:DateTimeString format="102" t-esc="format_date(record.invoice_date)"/>
                    </ram:IssueDateTime>
                    <ram:IncludedNote t-if="record.narration">
                        <ram:Content t-esc="record.narration"/>
                    </ram:IncludedNote>
                </rsm:ExchangedDocument>

                <rsm:SupplyChainTradeTransaction>
                    <!-- Invoice lines. -->
                    <t t-foreach="invoice_line_values" t-as="line_values">
                        <t t-call="account_facturx.account_invoice_line_facturx_export"/>
                    </t>

                    <!-- Partners. -->
                    <ram:ApplicableHeaderTradeAgreement>
                        <!-- Seller. -->
                        <ram:SellerTradeParty>
                            <!-- Address. -->
                            <t t-call="account_facturx.account_invoice_partner_facturx_export">
                                <t t-set="partner" t-value="record.company_id.partner_id"/>
                                <t t-set="SpecifiedLegalOrganization" t-value="seller_specified_legal_organization"/>
                            </t>

                            <!-- VAT. -->
                            <ram:SpecifiedTaxRegistration t-if="record.company_id.vat">
                                <ram:ID schemeID="VA" t-esc="record.company_id.vat"/>
                            </ram:SpecifiedTaxRegistration>
                        </ram:SellerTradeParty>

                        <!-- Customer. -->
                        <ram:BuyerTradeParty>
                            <!-- Address. -->
                            <t t-call="account_facturx.account_invoice_partner_facturx_export">
                                <t t-set="partner" t-value="record.commercial_partner_id"/>
                                <t t-set="SpecifiedLegalOrganization" t-value="buyer_specified_legal_organization"/>
                            </t>

                            <!-- VAT. -->
                            <ram:SpecifiedTaxRegistration t-if="record.commercial_partner_id.vat">
                                <ram:ID schemeID="VA" t-esc="record.commercial_partner_id.vat"/>
                            </ram:SpecifiedTaxRegistration>
                        </ram:BuyerTradeParty>

                        <!-- Reference. -->
                        <ram:BuyerOrderReferencedDocument>
                            <ram:IssuerAssignedID t-esc="record.invoice_payment_ref if record.invoice_payment_ref else record.name"/>
                        </ram:BuyerOrderReferencedDocument>
                    </ram:ApplicableHeaderTradeAgreement>

                    <!-- Delivery. Don't make a dependency with sale only for one field. -->
                    <ram:ApplicableHeaderTradeDelivery>
                        <ram:ShipToTradeParty>
                            <t t-call="account_facturx.account_invoice_partner_facturx_export">
                                <t t-if="'partner_shipping_id' in record._fields and record.partner_shipping_id" t-set="partner" t-value="record.partner_shipping_id"/>
                                <t t-else="" t-set="partner" t-value="record.commercial_partner_id"/>
                            </t>
                        </ram:ShipToTradeParty>
                    </ram:ApplicableHeaderTradeDelivery>

                    <!-- Taxes. -->
                    <ram:ApplicableHeaderTradeSettlement>
                        <ram:InvoiceCurrencyCode t-esc="record.currency_id.name"/>

                        <!-- Bank account. -->
                        <ram:SpecifiedTradeSettlementPaymentMeans t-if="record.invoice_partner_bank_id.acc_type == 'iban'">
                            <ram:TypeCode>42</ram:TypeCode>
                            <ram:PayeePartyCreditorFinancialAccount>
                                <ram:IBANID t-esc="record.invoice_partner_bank_id.sanitized_acc_number"/>
                            </ram:PayeePartyCreditorFinancialAccount>
                        </ram:SpecifiedTradeSettlementPaymentMeans>

                        <!-- Tax Summary. -->
                        <t t-foreach="tax_details" t-as="tax_vals">
                            <ram:ApplicableTradeTax>
                                <ram:CalculatedAmount
                                    t-esc="format_monetary(tax_vals['tax_amount'], currency)"/>
                                <ram:TypeCode>VAT</ram:TypeCode>
                                <ram:ExemptionReason t-if="tax_vals['unece_tax_category_code'] == 'E'" t-esc="tax_vals['exemption_reason']"/>
                                <ram:ExemptionReason t-if="tax_vals['unece_tax_category_code'] == 'G'" t-esc="'Export outside the EU'"/>
                                <ram:ExemptionReason t-if="tax_vals['unece_tax_category_code'] == 'K'" t-esc="'Intra-community supply'"/>
                                <ram:BasisAmount
                                    t-esc="format_monetary(tax_vals['tax_base_amount'], currency)"/>
                                <ram:CategoryCode t-esc="tax_vals['unece_tax_category_code']"/>
                                <ram:RateApplicablePercent
                                    t-esc="tax_vals['amount']"/>
                            </ram:ApplicableTradeTax>
                        </t>

                        <ram:BillingSpecifiedPeriod>
                            <ram:StartDateTime>
                                <udt:DateTimeString format="102" t-esc="format_date(record.invoice_date)"/>
                            </ram:StartDateTime>
                        </ram:BillingSpecifiedPeriod>

                        <!-- Payment Term. -->
                        <ram:SpecifiedTradePaymentTerms>
                            <ram:Description t-if="record.invoice_payment_term_id" t-esc="record.invoice_payment_term_id.name"/>
                            <ram:DueDateDateTime t-if="record.invoice_date_due">
                                <udt:DateTimeString format="102" t-esc="format_date(record.invoice_date_due)"/>
                            </ram:DueDateDateTime>
                        </ram:SpecifiedTradePaymentTerms>

                        <!-- Summary. -->
                        <ram:SpecifiedTradeSettlementHeaderMonetarySummation>
                            <ram:LineTotalAmount
                                t-esc="format_monetary(record.amount_untaxed, currency)"/>
                            <ram:TaxBasisTotalAmount
                                t-esc="format_monetary(record.amount_untaxed, currency)"/>
                            <ram:TaxTotalAmount
                                t-att-currencyID="currency.name"
                                t-esc="format_monetary(record.amount_tax, currency)"/>
                            <ram:GrandTotalAmount
                                t-esc="format_monetary(record.amount_total, currency)"/>
                            <ram:TotalPrepaidAmount
                                t-esc="format_monetary(record.amount_total - record.amount_residual, currency)"/>
                            <ram:DuePayableAmount
                                t-esc="format_monetary(record.amount_residual, currency)"/>
                        </ram:SpecifiedTradeSettlementHeaderMonetarySummation>
                    </ram:ApplicableHeaderTradeSettlement>
                </rsm:SupplyChainTradeTransaction>
            </rsm:CrossIndustryInvoice>
        </template>

        <template id="account_invoice_pdfa_3_facturx_metadata">
            <x:xmpmeta xmlns:x="adobe:ns:meta/">
                <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
                    <rdf:Description xmlns:pdfaid="http://www.aiim.org/pdfa/ns/id/" rdf:about="">
                        <pdfaid:part>3</pdfaid:part>
                        <pdfaid:conformance>B</pdfaid:conformance>
                    </rdf:Description>
                    <rdf:Description xmlns:dc="http://purl.org/dc/elements/1.1/" rdf:about="">
                        <dc:title>
                            <rdf:Alt>
                                <rdf:li t-att="{'xml:lang': 'x-default'}" t-esc="title"/>
                            </rdf:Alt>
                        </dc:title>
                        <dc:creator>
                            <rdf:Seq>
                                <rdf:li>Odoo</rdf:li>
                            </rdf:Seq>
                        </dc:creator>
                        <dc:description>
                            <rdf:Alt>
                                <rdf:li t-att="{'xml:lang': 'x-default'}">Invoice generated by Odoo</rdf:li>
                            </rdf:Alt>
                        </dc:description>
                    </rdf:Description>
                    <rdf:Description xmlns:pdf="http://ns.adobe.com/pdf/1.3/" rdf:about="">
                        <pdf:Producer>Odoo</pdf:Producer>
                    </rdf:Description>
                    <rdf:Description xmlns:xmp="http://ns.adobe.com/xap/1.0/" rdf:about="">
                        <xmp:CreatorTool>Odoo</xmp:CreatorTool>
                        <xmp:CreateDate t-esc="date"/>
                        <xmp:ModifyDate t-esc="date"/>
                    </rdf:Description>
                    <rdf:Description xmlns:pdfaExtension="http://www.aiim.org/pdfa/ns/extension/"
                                     xmlns:pdfaSchema="http://www.aiim.org/pdfa/ns/schema#"
                                     xmlns:pdfaProperty="http://www.aiim.org/pdfa/ns/property#" rdf:about="">
                        <pdfaExtension:schemas>
                            <rdf:Bag>
                                <rdf:li rdf:parseType="Resource">
                                    <pdfaSchema:schema>Factur-X PDFA Extension Schema</pdfaSchema:schema>
                                    <pdfaSchema:namespaceURI>urn:factur-x:pdfa:CrossIndustryDocument:invoice:1p0#</pdfaSchema:namespaceURI>
                                    <pdfaSchema:prefix>fx</pdfaSchema:prefix>
                                    <pdfaSchema:property>
                                        <rdf:Seq>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name>DocumentFileName</pdfaProperty:name>
                                                <pdfaProperty:valueType>Text</pdfaProperty:valueType>
                                                <pdfaProperty:category>external</pdfaProperty:category>
                                                <pdfaProperty:description>name of the embedded XML invoice file</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name>DocumentType</pdfaProperty:name>
                                                <pdfaProperty:valueType>Text</pdfaProperty:valueType>
                                                <pdfaProperty:category>external</pdfaProperty:category>
                                                <pdfaProperty:description>INVOICE</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name>Version</pdfaProperty:name>
                                                <pdfaProperty:valueType>Text</pdfaProperty:valueType>
                                                <pdfaProperty:category>external</pdfaProperty:category>
                                                <pdfaProperty:description>The actual version of the Factur-X XML schema</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name>ConformanceLevel</pdfaProperty:name>
                                                <pdfaProperty:valueType>Text</pdfaProperty:valueType>
                                                <pdfaProperty:category>external</pdfaProperty:category>
                                                <pdfaProperty:description>The conformance level of the embedded Factur-X data</pdfaProperty:description>
                                            </rdf:li>
                                        </rdf:Seq>
                                    </pdfaSchema:property>
                                </rdf:li>
                            </rdf:Bag>
                        </pdfaExtension:schemas>
                    </rdf:Description>
                    <rdf:Description xmlns:fx="urn:factur-x:pdfa:CrossIndustryDocument:invoice:1p0#" rdf:about="">
                        <fx:ConformanceLevel>EN 16931</fx:ConformanceLevel>
                        <fx:DocumentFileName>factur-x.xml</fx:DocumentFileName>
                        <fx:DocumentType>INVOICE</fx:DocumentType>
                        <fx:Version>1.0</fx:Version>
                    </rdf:Description>
                </rdf:RDF>
            </x:xmpmeta>
        </template>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models, fields, tools, _
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT, float_repr
from odoo.tests.common import Form
from odoo.exceptions import UserError, except_orm

from collections import defaultdict
from datetime import datetime
from lxml import etree
from PyPDF2 import PdfFileReader

import io
import base64

import logging
_logger = logging.getLogger(__name__)


DEFAULT_FACTURX_DATE_FORMAT = '%Y%m%d'


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _export_as_facturx_xml(self):
        ''' Create the Factur-x xml file content.
        :return: The XML content as str.
        '''
        self.ensure_one()

        def format_date(dt):
            # Format the date in the Factur-x standard.
            dt = dt or datetime.now()
            return dt.strftime(DEFAULT_FACTURX_DATE_FORMAT)

        def format_monetary(number, currency):
            # Format the monetary values to avoid trailing decimals (e.g. 90.85000000000001).
            if currency.is_zero(number):  # Ensure that we never return -0.0
                number = 0.0
            return float_repr(number, currency.decimal_places)

        # Create file content.
        seller_siret = 'siret' in self.company_id._fields and self.company_id.siret or self.company_id.company_registry
        buyer_siret = 'siret' in self.commercial_partner_id._fields and self.commercial_partner_id.siret
        template_values = {
            'record': self,
            'format_date': format_date,
            'format_monetary': format_monetary,
            'invoice_line_values': [],
            'seller_specified_legal_organization': seller_siret,
            'buyer_specified_legal_organization': buyer_siret,
        }
        # Tax lines.
        # The old system was making one total "line" per tax in the xml, by using the tax_line_id.
        # The new one is making one total "line" for each tax category and rate group.
        aggregated_taxes_details = self._prepare_edi_tax_details(
            grouping_key_generator=lambda tax_values: {
                'unece_tax_category_code': tax_values['tax_id']._get_unece_category_code(self.commercial_partner_id, self.company_id),
                'amount': tax_values['tax_id'].amount
            }
        )['tax_details']

        balance_multiplicator = -1 if self.is_inbound() else 1
        # Map the new keys from the backported _prepare_edi_tax_details into the old ones for compatibility with the old
        # template. Also apply the multiplication here for consistency between the old and new template.
        for tax_detail in aggregated_taxes_details.values():
            tax_detail['tax_base_amount'] = balance_multiplicator * tax_detail['base_amount_currency']
            tax_detail['tax_amount'] = balance_multiplicator * tax_detail['tax_amount_currency']

            # The old template would get the amount from a tax line given to it, while
            # the new one would get the amount from the dictionary returned by _prepare_edi_tax_details directly.
            # As the line was only used to get the tax amount, giving it any line with the same amount will give a correct
            # result even if the tax line doesn't make much sense as this is a total that is not linked to a specific tax.
            # I don't have a solution for 0% taxes, we will give an empty line that will allow to render the xml, but it
            # won't be completely correct. (The RateApplicablePercent will be missing for that line)
            tax_detail['line'] = self.line_ids.filtered(lambda l: l.tax_line_id and l.tax_line_id.amount == tax_detail['amount'])[:1]

        # Invoice lines.
        for i, line in enumerate(self.invoice_line_ids.filtered(lambda l: not l.display_type)):
            price_unit_with_discount = line.price_unit * (1 - (line.discount / 100.0))
            taxes_res = line.tax_ids.with_context(force_sign=line.move_id._get_tax_force_sign()).compute_all(
                price_unit_with_discount,
                currency=line.currency_id,
                quantity=line.quantity,
                product=line.product_id,
                partner=self.partner_id,
                is_refund=line.move_id.type in ('in_refund', 'out_refund'),
            )

            if line.discount == 100.0:
                gross_price_subtotal = line.always_set_currency_id.round(line.price_unit * line.quantity)
            else:
                gross_price_subtotal = line.always_set_currency_id.round(line.price_subtotal / (1 - (line.discount / 100.0)))
            line_template_values = {
                'line': line,
                'index': i + 1,
                'tax_details': [],
                'net_price_subtotal': taxes_res['total_excluded'],
                'price_discount_unit': (gross_price_subtotal - line.price_subtotal) / line.quantity if line.quantity else 0.0,
                'unece_uom_code': line.product_id.product_tmpl_id.uom_id._get_unece_code(),
            }

            for tax_res in taxes_res['taxes']:
                tax = self.env['account.tax'].browse(tax_res['id'])
                tax_category_code = tax._get_unece_category_code(self.commercial_partner_id, self.company_id)
                line_template_values['tax_details'].append({
                    'tax': tax,
                    'tax_amount': tax_res['amount'],
                    'tax_base_amount': tax_res['base'],
                    'unece_tax_category_code': tax_category_code,
                })

            template_values['invoice_line_values'].append(line_template_values)

        template_values['tax_details'] = list(aggregated_taxes_details.values())

        content = self.env.ref('account_facturx.account_invoice_facturx_export').render(template_values)
        return b"<?xml version='1.0' encoding='UTF-8'?>" + content

    def _import_facturx_invoice(self, tree):
        ''' Extract invoice values from the Factur-x xml tree passed as parameter.

        :param tree: The tree of the Factur-x xml file.
        :return: A dictionary containing account.invoice values to create/update it.
        '''
        def find_partner(partner_type):
            elements = tree.xpath('//ram:%s/ram:SpecifiedTaxRegistration/ram:ID' % partner_type, namespaces=tree.nsmap)
            partner = elements and self.env['res.partner'].search([('vat', '=', elements[0].text)], limit=1)
            if not partner:
                elements = tree.xpath('//ram:%s/ram:Name' % partner_type, namespaces=tree.nsmap)
                partner_name = elements and elements[0].text
                partner = elements and self.env['res.partner'].search([('name', 'ilike', partner_name)], limit=1)
            if not partner:
                elements = tree.xpath('//ram:%s//ram:URIID[@schemeID=\'SMTP\']' % partner_type, namespaces=tree.nsmap)
                partner = elements and self.env['res.partner'].search([('email', '=', elements[0].text)], limit=1)
            return partner or self.env["res.partner"]

        amount_total_import = None

        default_type = False
        if self._context.get('default_journal_id'):
            journal = self.env['account.journal'].browse(self.env.context['default_journal_id'])
            default_type = 'out_invoice' if journal.type == 'sale' else 'in_invoice'
        elif self._context.get('default_type'):
            default_type = self._context['default_type']
        elif self.type in self.env['account.move'].get_invoice_types(include_receipts=True):
            # in case an attachment is saved on a draft invoice previously created, we might
            # have lost the default value in context but the type was already set
            default_type = self.type

        if not default_type:
            raise UserError(_("No information about the journal or the type of invoice is passed"))
        if default_type == 'entry':
            return

        # Total amount.
        elements = tree.xpath('//ram:GrandTotalAmount', namespaces=tree.nsmap)
        total_amount = elements and float(elements[0].text) or 0.0

        # Refund type.
        # There is two modes to handle refund in Factur-X:
        # a) type_code == 380 for invoice, type_code == 381 for refund, all positive amounts.
        # b) type_code == 380, negative amounts in case of refund.
        # To handle both, we consider the 'a' mode and switch to 'b' if a negative amount is encountered.
        elements = tree.xpath('//rsm:ExchangedDocument/ram:TypeCode', namespaces=tree.nsmap)
        type_code = elements[0].text

        default_type.replace('_refund', '_invoice')
        if type_code == '381':
            default_type = 'out_refund' if default_type == 'out_invoice' else 'in_refund'
            refund_sign = -1
        else:
            # Handle 'b' refund mode.
            if total_amount < 0:
                default_type = 'out_refund' if default_type == 'out_invoice' else 'in_refund'
            refund_sign = -1 if 'refund' in default_type else 1

        # Write the type as the journal entry is already created.
        self.type = default_type

        # self could be a single record (editing) or be empty (new).
        with Form(self.with_context(default_type=default_type)) as invoice_form:
            # Partner (first step to avoid warning 'Warning! You must first select a partner.').
            partner_type = invoice_form.journal_id.type == 'purchase' and 'SellerTradeParty' or 'BuyerTradeParty'
            invoice_form.partner_id = find_partner(partner_type)

            # Reference.
            elements = tree.xpath('//rsm:ExchangedDocument/ram:ID', namespaces=tree.nsmap)
            if elements:
                invoice_form.ref = elements[0].text

            # Name.
            elements = tree.xpath('//ram:BuyerOrderReferencedDocument/ram:IssuerAssignedID', namespaces=tree.nsmap)
            if elements:
                invoice_form.invoice_payment_ref = elements[0].text

            # Comment.
            elements = tree.xpath('//ram:IncludedNote/ram:Content', namespaces=tree.nsmap)
            if elements:
                invoice_form.narration = elements[0].text

            # Get currency string for new invoices, or invoices coming from outside
            elements = tree.xpath('//ram:InvoiceCurrencyCode', namespaces=tree.nsmap)
            if elements:
                currency_str = elements[0].text
            # Fallback for old invoices from odoo where the InvoiceCurrencyCode was not present
            else:
                elements = tree.xpath('//ram:TaxTotalAmount', namespaces=tree.nsmap)
                if elements:
                    currency_str = elements[0].attrib['currencyID']

            # Set the invoice currency
            if currency_str:
                currency = self.env.ref('base.%s' % currency_str.upper(), raise_if_not_found=False)
                if currency and not currency.active:
                    raise UserError(
                        _('The currency (%s) of the document you are uploading is not active in this database.\n'
                          'Please activate it before trying again to import.') % currency.name
                    )
                if currency != self.env.company.currency_id and currency.active:
                    invoice_form.currency_id = currency

            # Store xml total amount.
            amount_total_import = total_amount * refund_sign

            # Date.
            elements = tree.xpath('//rsm:ExchangedDocument/ram:IssueDateTime/udt:DateTimeString', namespaces=tree.nsmap)
            if elements:
                date_str = elements[0].text
                date_obj = datetime.strptime(date_str, DEFAULT_FACTURX_DATE_FORMAT)
                invoice_form.invoice_date = date_obj.strftime(DEFAULT_SERVER_DATE_FORMAT)

            # Due date.
            elements = tree.xpath('//ram:SpecifiedTradePaymentTerms/ram:DueDateDateTime/udt:DateTimeString', namespaces=tree.nsmap)
            if elements:
                date_str = elements[0].text
                date_obj = datetime.strptime(date_str, DEFAULT_FACTURX_DATE_FORMAT)
                invoice_form.invoice_date_due = date_obj.strftime(DEFAULT_SERVER_DATE_FORMAT)

            # Invoice lines.
            elements = tree.xpath('//ram:IncludedSupplyChainTradeLineItem', namespaces=tree.nsmap)
            if elements:
                for element in elements:
                    with invoice_form.invoice_line_ids.new() as invoice_line_form:

                        # Sequence.
                        line_elements = element.xpath('.//ram:AssociatedDocumentLineDocument/ram:LineID', namespaces=tree.nsmap)
                        if line_elements:
                            invoice_line_form.sequence = int(line_elements[0].text)

                        # Product.
                        line_elements = element.xpath('.//ram:SpecifiedTradeProduct/ram:Name', namespaces=tree.nsmap)
                        if line_elements:
                            invoice_line_form.name = line_elements[0].text
                        line_elements = element.xpath('.//ram:SpecifiedTradeProduct/ram:SellerAssignedID', namespaces=tree.nsmap)
                        if line_elements and line_elements[0].text:
                            product = self.env['product.product'].search([('default_code', '=', line_elements[0].text)])
                            if product:
                                invoice_line_form.product_id = product
                        if not invoice_line_form.product_id:
                            line_elements = element.xpath('.//ram:SpecifiedTradeProduct/ram:GlobalID', namespaces=tree.nsmap)
                            if line_elements and line_elements[0].text:
                                product = self.env['product.product'].search([('barcode', '=', line_elements[0].text)])
                                if product:
                                    invoice_line_form.product_id = product

                        # Quantity.
                        line_elements = element.xpath('.//ram:SpecifiedLineTradeDelivery/ram:BilledQuantity', namespaces=tree.nsmap)
                        if line_elements:
                            invoice_line_form.quantity = float(line_elements[0].text)

                        # Price Unit.
                        line_elements = element.xpath('.//ram:GrossPriceProductTradePrice/ram:ChargeAmount', namespaces=tree.nsmap)
                        if line_elements:
                            quantity_elements = element.xpath('.//ram:GrossPriceProductTradePrice/ram:BasisQuantity', namespaces=tree.nsmap)
                            if quantity_elements:
                                invoice_line_form.price_unit = float(line_elements[0].text) / float(quantity_elements[0].text)
                            else:
                                invoice_line_form.price_unit = float(line_elements[0].text)
                            # For Gross price, we need to check if a discount must be taken into account
                            discount_elements = element.xpath('.//ram:AppliedTradeAllowanceCharge',
                                                          namespaces=tree.nsmap)
                            if discount_elements:
                                discount_percent_elements = element.xpath(
                                    './/ram:AppliedTradeAllowanceCharge/ram:CalculationPercent', namespaces=tree.nsmap)
                                if discount_percent_elements:
                                    invoice_line_form.discount = float(discount_percent_elements[0].text)
                                else:
                                    # if discount not available, it will be computed from the gross and net prices.
                                    net_price_elements = element.xpath('.//ram:NetPriceProductTradePrice/ram:ChargeAmount',
                                                                  namespaces=tree.nsmap)
                                    if net_price_elements:
                                        quantity_elements = element.xpath(
                                            './/ram:NetPriceProductTradePrice/ram:BasisQuantity', namespaces=tree.nsmap)
                                        net_unit_price = float(net_price_elements[0].text) / float(quantity_elements[0].text) \
                                            if quantity_elements else float(net_price_elements[0].text)
                                        invoice_line_form.discount = (invoice_line_form.price_unit - net_unit_price) / invoice_line_form.price_unit * 100.0
                        else:
                            line_elements = element.xpath('.//ram:NetPriceProductTradePrice/ram:ChargeAmount', namespaces=tree.nsmap)
                            if line_elements:
                                quantity_elements = element.xpath('.//ram:NetPriceProductTradePrice/ram:BasisQuantity', namespaces=tree.nsmap)
                                if quantity_elements:
                                    invoice_line_form.price_unit = float(line_elements[0].text) / float(quantity_elements[0].text)
                                else:
                                    invoice_line_form.price_unit = float(line_elements[0].text)

                        # Taxes
                        line_elements = element.xpath('.//ram:SpecifiedLineTradeSettlement/ram:ApplicableTradeTax/ram:RateApplicablePercent', namespaces=tree.nsmap)
                        invoice_line_form.tax_ids.clear()
                        for tax_element in line_elements:
                            percentage = float(tax_element.text)

                            tax = self.env['account.tax'].search([
                                ('company_id', '=', invoice_form.company_id.id),
                                ('amount_type', '=', 'percent'),
                                ('type_tax_use', '=', invoice_form.journal_id.type),
                                ('amount', '=', percentage),
                            ], limit=1)

                            if tax:
                                invoice_line_form.tax_ids.add(tax)
            elif amount_total_import:
                # No lines in BASICWL.
                with invoice_form.invoice_line_ids.new() as invoice_line_form:
                    invoice_line_form.name = invoice_form.comment or '/'
                    invoice_line_form.quantity = 1
                    invoice_line_form.price_unit = amount_total_import

        return invoice_form.save()

    def _message_post_process_attachments(self, attachments, attachment_ids, message_values):
        # OVERRIDE
        # /!\ 'default_res_id' in self._context is used to don't process attachment when using a form view.
        return_values = super()._message_post_process_attachments(attachments, attachment_ids, message_values)
        if not self.env.context.get('no_new_invoice') and len(self) == 1 and self.state == 'draft' and (
            self.env.context.get('default_type', self.type) in self.env['account.move'].get_invoice_types(include_receipts=True)
            or self.env['account.journal'].browse(self.env.context.get('default_journal_id')).type in ('sale', 'purchase')
        ):
            attachments = self.env['ir.attachment'].browse([c[1] for c in return_values['attachment_ids']])
            for attachment in attachments:
                self._create_invoice_from_attachment(attachment)
        return return_values

    def _create_invoice_from_attachment(self, attachment):
        if 'pdf' in attachment.mimetype:
            for move in self:
                move._create_invoice_from_pdf(attachment)
        if 'xml' in attachment.mimetype:
            for move in self:
                move._create_invoice_from_xml(attachment)

    def _create_invoice_from_pdf(self, attachment):
        content = base64.b64decode(attachment.datas)

        with io.BytesIO(content) as buffer:
            try:
                reader = PdfFileReader(buffer)

                # Search for Factur-x embedded file.
                if reader.trailer['/Root'].get('/Names') and reader.trailer['/Root']['/Names'].get('/EmbeddedFiles'):
                    # N.B: embedded_files looks like:
                    # ['file.xml', {'/Type': '/Filespec', '/F': 'file.xml', '/EF': {'/F': IndirectObject(22, 0)}}]
                    embedded_files = reader.trailer['/Root']['/Names']['/EmbeddedFiles']['/Names']
                    # '[::2]' because it's a list [fn_1, content_1, fn_2, content_2, ..., fn_n, content_2]
                    for filename_obj, content_obj in list(zip(embedded_files, embedded_files[1:]))[::2]:
                        content = content_obj.getObject()['/EF']['/F'].getData()
                        try:
                            tree = etree.fromstring(content)
                        except Exception:
                            continue
                        if tree.tag == '{urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100}CrossIndustryInvoice':
                            self._import_facturx_invoice(tree)
                            self._remove_ocr_option()
                            buffer.close()
            except except_orm as e:
                raise e
            except Exception as e:
                # Malformed pdf
                _logger.exception(e)

    @api.model
    def _get_xml_decoders(self):
        ''' List of usable decoders to extract invoice from attachments.

        :return: a list of triplet (xml_type, check_func, decode_func)
            * xml_type: The format name, e.g 'UBL 2.1'
            * check_func: A function taking an etree and a file name as parameter and returning a dict:
                * flag: The etree is part of this format.
                * error: Error message.
            * decode_func: A function taking an etree as parameter and returning an invoice record.
        '''
        # TO BE OVERWRITTEN
        return []

    def _create_invoice_from_xml(self, attachment):
        decoders = self._get_xml_decoders()

        # Convert attachment -> etree
        content = base64.b64decode(attachment.datas)
        try:
            tree = etree.fromstring(content)
        except Exception:
            _logger.exception('The xml file is badly formatted : {}'.format(attachment.name))

        for xml_type, check_func, decode_func in decoders:
            check_res = check_func(tree, attachment.name)

            if check_res.get('flag') and not check_res.get('error'):
                invoice_ids = decode_func(tree)
                if invoice_ids:
                    invoice_ids._remove_ocr_option()
                    break

        try:
            return invoice_ids
        except UnboundLocalError:
            _logger.exception('No decoder was found for the xml file: {}. The file is badly formatted, not supported or the decoder is not installed'.format(attachment.name))

    def _remove_ocr_option(self):
        if 'extract_state' in self:
            self.write({'extract_state': 'done'})

    @api.model
    def _add_edi_tax_values(self, results, grouping_key, serialized_grouping_key, tax_values):
        # Add to global results.
        results['tax_amount'] += tax_values['tax_amount']
        results['tax_amount_currency'] += tax_values['tax_amount_currency']

        # Add to tax details.
        tax_details = results['tax_details'][serialized_grouping_key]
        tax_details.update(grouping_key)
        if tax_values['base_line_id'] not in set(x['base_line_id'] for x in tax_details['group_tax_details']):
            tax_details['base_amount'] += tax_values['base_amount']
            tax_details['base_amount_currency'] += tax_values['base_amount_currency']

        tax_details['tax_amount'] += tax_values['tax_amount']
        tax_details['tax_amount_currency'] += tax_values['tax_amount_currency']
        tax_details['exemption_reason'] = tax_values['tax_id'].name
        tax_details['group_tax_details'].append(tax_values)

    def _prepare_edi_tax_details(self, filter_to_apply=None, filter_invl_to_apply=None, grouping_key_generator=None):
        ''' Compute amounts related to taxes for the current invoice.
        :param filter_to_apply:         Optional filter to exclude some tax values from the final results.
                                        The filter is defined as a method getting a dictionary as parameter
                                        representing the tax values for a single repartition line.
                                        This dictionary contains:
            'base_line_id':             An account.move.line record.
            'tax_id':                   An account.tax record.
            'tax_repartition_line_id':  An account.tax.repartition.line record.
            'base_amount':              The tax base amount expressed in company currency.
            'tax_amount':               The tax amount expressed in company currency.
            'base_amount_currency':     The tax base amount expressed in foreign currency.
            'tax_amount_currency':      The tax amount expressed in foreign currency.
                                        If the filter is returning False, it means the current tax values will be
                                        ignored when computing the final results.
        :param grouping_key_generator:  Optional method used to group tax values together. By default, the tax values
                                        are grouped by tax. This parameter is a method getting a dictionary as parameter
                                        (same signature as 'filter_to_apply').
                                        This method must returns a dictionary where values will be used to create the
                                        grouping_key to aggregate tax values together. The returned dictionary is added
                                        to each tax details in order to retrieve the full grouping_key later.
        :return:                        The full tax details for the current invoice and for each invoice line
                                        separately. The returned dictionary is the following:
            'base_amount':              The total tax base amount in company currency for the whole invoice.
            'tax_amount':               The total tax amount in company currency for the whole invoice.
            'base_amount_currency':     The total tax base amount in foreign currency for the whole invoice.
            'tax_amount_currency':      The total tax amount in foreign currency for the whole invoice.
            'tax_details':              A mapping of each grouping key (see 'grouping_key_generator') to a dictionary
                                        containing:
                'base_amount':              The tax base amount in company currency for the current group.
                'tax_amount':               The tax amount in company currency for the current group.
                'base_amount_currency':     The tax base amount in foreign currency for the current group.
                'tax_amount_currency':      The tax amount in foreign currency for the current group.
                'group_tax_details':        The list of all tax values aggregated into this group.
            'invoice_line_tax_details': A mapping of each invoice line to a dictionary containing:
                'base_amount':          The total tax base amount in company currency for the whole invoice line.
                'tax_amount':           The total tax amount in company currency for the whole invoice line.
                'base_amount_currency': The total tax base amount in foreign currency for the whole invoice line.
                'tax_amount_currency':  The total tax amount in foreign currency for the whole invoice line.
                'tax_details':          A mapping of each grouping key (see 'grouping_key_generator') to a dictionary
                                        containing:
                    'base_amount':          The tax base amount in company currency for the current group.
                    'tax_amount':           The tax amount in company currency for the current group.
                    'base_amount_currency': The tax base amount in foreign currency for the current group.
                    'tax_amount_currency':  The tax amount in foreign currency for the current group.
                    'group_tax_details':    The list of all tax values aggregated into this group.
        '''
        self.ensure_one()

        def _serialize_python_dictionary(vals):
            return '-'.join(str(vals[k]) for k in sorted(vals.keys()))

        def default_grouping_key_generator(tax_values):
            return {'tax': tax_values['tax_id']}

        # Compute the taxes values for each invoice line.

        invoice_lines = self.invoice_line_ids.filtered(lambda line: not line.display_type)
        if filter_invl_to_apply:
            invoice_lines = invoice_lines.filtered(filter_invl_to_apply)

        invoice_lines_tax_values_dict = {}
        sign = -1 if self.is_inbound() else 1
        for invoice_line in invoice_lines:
            taxes_res = invoice_line.tax_ids.compute_all(
                invoice_line.price_unit * (1 - (invoice_line.discount / 100.0)),
                currency=invoice_line.currency_id,
                quantity=invoice_line.quantity,
                product=invoice_line.product_id,
                partner=invoice_line.partner_id,
                is_refund=invoice_line.move_id.type in ('in_refund', 'out_refund'),
            )
            tax_values_list = invoice_lines_tax_values_dict[invoice_line] = []
            rate = abs(invoice_line.balance) / abs(
                invoice_line.amount_currency) if invoice_line.amount_currency else 0.0
            for tax_res in taxes_res['taxes']:
                tax_values_list.append({
                    'base_line_id': invoice_line,
                    'tax_id': self.env['account.tax'].browse(tax_res['id']),
                    'tax_repartition_line_id': self.env['account.tax.repartition.line'].browse(
                        tax_res['tax_repartition_line_id']),
                    'base_amount': sign * invoice_line.company_currency_id.round(tax_res['base'] * rate),
                    'tax_amount': sign * invoice_line.company_currency_id.round(tax_res['amount'] * rate),
                    'base_amount_currency': sign * tax_res['base'],
                    'tax_amount_currency': sign * tax_res['amount'],
                })
        grouping_key_generator = grouping_key_generator or default_grouping_key_generator

        # Apply 'filter_to_apply'.

        if filter_to_apply:
            invoice_lines_tax_values_dict = {
                invoice_line: [x for x in tax_values_list if filter_to_apply(x)]
                for invoice_line, tax_values_list in invoice_lines_tax_values_dict.items()
            }

        # Initialize the results dict.

        invoice_global_tax_details = {
            'base_amount': 0.0,
            'tax_amount': 0.0,
            'base_amount_currency': 0.0,
            'tax_amount_currency': 0.0,
            'tax_details': defaultdict(lambda: {
                'base_amount': 0.0,
                'tax_amount': 0.0,
                'base_amount_currency': 0.0,
                'tax_amount_currency': 0.0,
                'group_tax_details': [],
            }),
            'invoice_line_tax_details': defaultdict(lambda: {
                'base_amount': 0.0,
                'tax_amount': 0.0,
                'base_amount_currency': 0.0,
                'tax_amount_currency': 0.0,
                'tax_details': defaultdict(lambda: {
                    'base_amount': 0.0,
                    'tax_amount': 0.0,
                    'base_amount_currency': 0.0,
                    'tax_amount_currency': 0.0,
                    'group_tax_details': [],
                }),
            }),
        }

        # Apply 'grouping_key_generator' to 'invoice_lines_tax_values_list' and add all values to the final results.

        for invoice_line in invoice_lines:
            tax_values_list = invoice_lines_tax_values_dict[invoice_line]

            # Add to invoice global tax amounts.
            invoice_global_tax_details['base_amount'] += invoice_line.balance
            invoice_global_tax_details['base_amount_currency'] += invoice_line.amount_currency

            for tax_values in tax_values_list:
                grouping_key = grouping_key_generator(tax_values)
                serialized_grouping_key = _serialize_python_dictionary(grouping_key)

                # Add to invoice line global tax amounts.
                if serialized_grouping_key not in invoice_global_tax_details['invoice_line_tax_details'][invoice_line]:
                    invoice_line_global_tax_details = invoice_global_tax_details['invoice_line_tax_details'][
                        invoice_line]
                    invoice_line_global_tax_details.update({
                        'base_amount': invoice_line.balance,
                        'base_amount_currency': invoice_line.amount_currency,
                    })
                else:
                    invoice_line_global_tax_details = invoice_global_tax_details['invoice_line_tax_details'][
                        invoice_line]

                self._add_edi_tax_values(invoice_global_tax_details, grouping_key, serialized_grouping_key, tax_values)
                self._add_edi_tax_values(invoice_line_global_tax_details, grouping_key, serialized_grouping_key,
                                         tax_values)

        return invoice_global_tax_details

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    def _get_unece_category_code(self, customer, supplier):
        """ By default, this method will try to compute the tax category (used by EDI for example) based on the amount
        and the tax repartition lines. This is hack-ish~ but a valid solution to get a default value in stable.

        In master, the Category selection field should be by default on taxes and filled for each tax in the demo data
        if possible.

        See https://unece.org/fileadmin/DAM/trade/untdid/d16b/tred/tred5305.htm for the codes.
        """
        self.ensure_one()
        # Defaulting to standard tax.
        category = 'S'
        if self.type_tax_use == 'sale':
            eu_countries = self.env.ref('base.europe').country_ids
            if supplier.country_id in eu_countries and customer.country_id not in eu_countries:
                category = 'G'
            else:
                if customer.country_id != supplier.country_id \
                        and customer.country_id in eu_countries \
                        and supplier.country_id in eu_countries:
                    category = 'K'
                # Taxes with a Zero amount will get the E code. (Exempt)
                elif self.amount == 0:
                    category = 'E'

        return category

```

## File: models\ir_actions_report.py

```python
# -*- coding: utf-8 -*-
from io import BytesIO
from logging import getLogger
from PyPDF2 import PdfFileReader

from odoo import fields, models
from odoo import tools
from odoo.tools.pdf import OdooPdfFileWriter

_logger = getLogger(__name__)


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _post_pdf(self, save_in_attachment, pdf_content=None, res_ids=None):
        # OVERRIDE
        if self.model == 'account.move' and res_ids and len(res_ids) == 1:
            invoice = self.env['account.move'].browse(res_ids)
            if invoice.is_sale_document() and invoice.state != 'draft':
                xml_content = invoice._export_as_facturx_xml()

                reader_buffer = BytesIO(pdf_content)
                reader = PdfFileReader(reader_buffer)
                writer = OdooPdfFileWriter()
                writer.cloneReaderDocumentRoot(reader)

                if tools.str2bool(self.env['ir.config_parameter'].sudo().get_param('edi.use_pdfa', 'False')):
                    try:
                        writer.convert_to_pdfa()
                    except Exception as e:
                        _logger.exception("Error while converting to PDF/A: %s", e)

                    metadata_template = self.env.ref('account_facturx.account_invoice_pdfa_3_facturx_metadata', False)
                    if metadata_template:
                        metadata_content = metadata_template.render({
                            'title': invoice.name,
                            'date': fields.Date.context_today(self),
                        })
                        writer.add_file_metadata(metadata_content)

                writer.addAttachment('factur-x.xml', xml_content, '/text#2Fxml')

                buffer = BytesIO()
                writer.write(buffer)
                pdf_content = buffer.getvalue()
                buffer.close()
                reader_buffer.close()

        return super(IrActionsReport, self)._post_pdf(save_in_attachment, pdf_content=pdf_content, res_ids=res_ids)

```

## File: models\uom.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class UoM(models.Model):
    _inherit = 'uom.uom'

    def _get_unece_code(self):
        """ Returns the UNECE code used for international trading for corresponding to the UoM as per
        https://unece.org/fileadmin/DAM/cefact/recommendations/rec20/rec20_rev3_Annex2e.pdf"""
        mapping = {
            'uom.product_uom_unit': 'C62',
            'uom.product_uom_dozen': 'DZN',
            'uom.product_uom_kgm': 'KGM',
            'uom.product_uom_gram': 'GRM',
            'uom.product_uom_day': 'DAY',
            'uom.product_uom_hour': 'HUR',
            'uom.product_uom_ton': 'TNE',
            'uom.product_uom_meter': 'MTR',
            'uom.product_uom_km': 'KTM',
            'uom.product_uom_cm': 'CMT',
            'uom.product_uom_litre': 'LTR',
            'uom.product_uom_lb': 'LBR',
            'uom.product_uom_oz': 'ONZ',
            'uom.product_uom_inch': 'INH',
            'uom.product_uom_foot': 'FOT',
            'uom.product_uom_mile': 'SMI',
            'uom.product_uom_floz': 'OZA',
            'uom.product_uom_qt': 'QT',
            'uom.product_uom_gal': 'GLL',
            'uom.product_uom_cubic_meter': 'MTQ',
            'uom.product_uom_cubic_inch': 'INQ',
            'uom.product_uom_cubic_foot': 'FTQ',
        }
        xml_ids = self._get_external_ids().get(self.id, [])
        matches = list(set(xml_ids) & set(mapping.keys()))
        return matches and mapping[matches[0]] or 'C62'

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-

from . import account_move
from . import account_tax
from . import ir_actions_report
from . import uom

```

