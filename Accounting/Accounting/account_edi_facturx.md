# Odoo Module: account_edi_facturx

Category: Accounting/Accounting

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
    'name': 'Import/Export invoices with Factur-X',
    'description': '''
    Support for invoice Export/Import in Factur-x format (1.0.04).
    ''',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'depends': ['account_edi'],
    'data': [
        'data/account_edi_data.xml',
        'data/facturx_templates.xml',
    ],
    'installable': True,
    'post_init_hook': '_post_init_hook',
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="edi_facturx_1_0_05" model="account.edi.format">
            <field name="name">Factur-X (FR)</field>
            <field name="code">facturx_1_0_05</field>
        </record>

    </data>
</odoo>
```

## File: data\facturx_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="account_invoice_line_facturx_export">
            <t t-set="line" t-value="line_vals['line']"/>
            <t  xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100"
                xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100"
                xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100">
                <ram:IncludedSupplyChainTradeLineItem>
                    <!-- Line number. -->
                    <ram:AssociatedDocumentLineDocument>
                        <ram:LineID t-esc="line_vals['index']"/>
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
                        <!-- TODO:
                        <ram:Name t-esc="line.product_id.name or line.name"/>
                        <ram:Description t-esc="line.name" t-if="line.name != line.product_id.name"/>
                        -->
                        <ram:Name t-esc="line.name"/>
                    </ram:SpecifiedTradeProduct>

                    <!-- Amounts. -->
                    <ram:SpecifiedLineTradeAgreement>
                        <!-- Line information, with discount and unit price separate -->
                        <ram:GrossPriceProductTradePrice>
                            <ram:ChargeAmount
                                t-esc="format_monetary(line_vals['gross_price_total_unit'], record.currency_id)"/>

                            <!-- Discount. -->
                            <ram:AppliedTradeAllowanceCharge t-if="line.discount">
                                <ram:ChargeIndicator>
                                    <udt:Indicator>false</udt:Indicator>
                                </ram:ChargeIndicator>
                                <ram:ActualAmount t-esc="format_monetary(line_vals['price_discount_unit'], record.currency_id)"/>
                            </ram:AppliedTradeAllowanceCharge>
                        </ram:GrossPriceProductTradePrice>
                        <!-- Line unit price, with discount applied -->
                        <ram:NetPriceProductTradePrice>
                            <ram:ChargeAmount
                                t-esc="format_monetary(line_vals['price_subtotal_unit'], record.currency_id)"/>
                        </ram:NetPriceProductTradePrice>
                    </ram:SpecifiedLineTradeAgreement>

                    <!-- Quantity. -->
                    <ram:SpecifiedLineTradeDelivery>
                        <ram:BilledQuantity t-att-unitCode="line_vals['unece_uom_code']" t-esc="line.quantity"/>
                    </ram:SpecifiedLineTradeDelivery>

                    <ram:SpecifiedLineTradeSettlement>
                        <t t-foreach="tax_details['invoice_line_tax_details'][line]['tax_details'].values()"
                           t-as="tax_detail_vals">
                            <ram:ApplicableTradeTax t-if="tax_detail_vals['amount_type'] == 'percent'">
                                <ram:TypeCode>VAT</ram:TypeCode>
                                <ram:CategoryCode t-esc="tax_detail_vals['unece_tax_category_code']"/>
                                <ram:RateApplicablePercent t-esc="tax_detail_vals['amount']"/>
                            </ram:ApplicableTradeTax>
                        </t>
                        <!-- Subtotal. -->
                        <ram:SpecifiedTradeSettlementLineMonetarySummation>
                            <ram:LineTotalAmount
                                t-esc="format_monetary(line.price_subtotal, record.currency_id)"/>
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
                <ram:Name t-if="partner.display_name" t-esc="partner.display_name"/>

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
                    <ram:TypeCode t-esc="'381' if 'refund' in record.move_type else '380'"/>
                    <ram:IssueDateTime>
                        <udt:DateTimeString format="102" t-esc="format_date(record.invoice_date)"/>
                    </ram:IssueDateTime>
                    <ram:IncludedNote t-if="not is_html_empty(record.narration)">
                        <ram:Content t-esc="record.narration"/>
                    </ram:IncludedNote>
                </rsm:ExchangedDocument>

                <rsm:SupplyChainTradeTransaction>
                    <!-- Invoice lines. -->
                    <t t-foreach="invoice_line_vals_list" t-as="line_vals">
                        <t t-call="account_edi_facturx.account_invoice_line_facturx_export"/>
                    </t>

                    <!-- Partners. -->
                    <ram:ApplicableHeaderTradeAgreement>
                        <ram:BuyerReference t-esc="buyer_reference or record.ref"/>

                        <!-- Seller. -->
                        <ram:SellerTradeParty>
                            <!-- Address. -->
                            <t t-call="account_edi_facturx.account_invoice_partner_facturx_export">
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
                            <t t-call="account_edi_facturx.account_invoice_partner_facturx_export">
                                <t t-set="partner" t-value="record.partner_id"/>
                                <t t-set="SpecifiedLegalOrganization" t-value="buyer_specified_legal_organization"/>
                            </t>

                            <!-- VAT. -->
                            <ram:SpecifiedTaxRegistration t-if="record.commercial_partner_id.vat">
                                <ram:ID schemeID="VA" t-esc="record.commercial_partner_id.vat"/>
                            </ram:SpecifiedTaxRegistration>
                        </ram:BuyerTradeParty>

                        <!-- Reference. -->
                        <ram:BuyerOrderReferencedDocument>
                            <ram:IssuerAssignedID t-esc="purchase_order_reference or record.payment_reference or record.name"/>
                        </ram:BuyerOrderReferencedDocument>

                        <ram:ContractReferencedDocument t-if="contract_reference">
                            <ram:IssuerAssignedID t-esc="contract_reference"/>
                        </ram:ContractReferencedDocument>
                    </ram:ApplicableHeaderTradeAgreement>

                    <!-- Delivery. Don't make a dependency with sale only for one field. -->
                    <ram:ApplicableHeaderTradeDelivery>
                        <ram:ShipToTradeParty>
                            <t t-call="account_edi_facturx.account_invoice_partner_facturx_export">
                                <t t-if="'partner_shipping_id' in record._fields and record.partner_shipping_id" t-set="partner" t-value="record.partner_shipping_id"/>
                                <t t-else="" t-set="partner" t-value="record.commercial_partner_id"/>
                            </t>
                        </ram:ShipToTradeParty>
                    </ram:ApplicableHeaderTradeDelivery>

                    <!-- Taxes. -->
                    <ram:ApplicableHeaderTradeSettlement>
                        <ram:InvoiceCurrencyCode t-esc="record.currency_id.name"/>

                        <!-- Bank account. -->
                        <ram:SpecifiedTradeSettlementPaymentMeans t-if="record.partner_bank_id.acc_type == 'iban'">
                        <ram:TypeCode>42</ram:TypeCode>
                            <ram:PayeePartyCreditorFinancialAccount>
                                <ram:IBANID t-esc="record.partner_bank_id.sanitized_acc_number"/>
                            </ram:PayeePartyCreditorFinancialAccount>
                        </ram:SpecifiedTradeSettlementPaymentMeans>

                        <!-- Tax Summary. -->
                        <t t-foreach="tax_details['tax_details'].values()" t-as="tax_detail_vals">
                            <ram:ApplicableTradeTax>
                                <ram:CalculatedAmount
                                    t-esc="format_monetary(balance_multiplicator * tax_detail_vals['tax_amount_currency'], record.currency_id)"/>
                                <ram:TypeCode>VAT</ram:TypeCode>
                                <ram:ExemptionReason t-if="tax_detail_vals['unece_tax_category_code'] == 'E'" t-esc="tax_detail_vals['exemption_reason']"/>
                                <ram:ExemptionReason t-if="tax_detail_vals['unece_tax_category_code'] == 'G'" t-esc="'Export outside the EU'"/>
                                <ram:ExemptionReason t-if="tax_detail_vals['unece_tax_category_code'] == 'K'" t-esc="'Intra-community supply'"/>
                                <ram:BasisAmount
                                    t-esc="format_monetary(balance_multiplicator * tax_detail_vals['base_amount_currency'], record.currency_id)"/>
                                <ram:CategoryCode t-esc="tax_detail_vals['unece_tax_category_code']"/>
                                <ram:RateApplicablePercent
                                    t-if="tax_detail_vals['amount_type'] == 'percent'"
                                    t-esc="tax_detail_vals['amount']"/>
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
                                t-esc="format_monetary(record.amount_untaxed, record.currency_id)"/>
                            <ram:TaxBasisTotalAmount
                                t-esc="format_monetary(record.amount_untaxed, record.currency_id)"/>
                            <ram:TaxTotalAmount
                                t-att-currencyID="record.currency_id.name"
                                t-esc="format_monetary(record.amount_tax, record.currency_id)"/>
                            <ram:GrandTotalAmount
                                t-esc="format_monetary(record.amount_total, record.currency_id)"/>
                            <ram:TotalPrepaidAmount
                                t-esc="format_monetary(record.amount_total - record.amount_residual, record.currency_id)"/>
                            <ram:DuePayableAmount
                                t-esc="format_monetary(record.amount_residual, record.currency_id)"/>
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

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models, fields, tools, _
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT, float_repr, is_html_empty, str2bool
from odoo.tests.common import Form
from odoo.exceptions import RedirectWarning, UserError

from datetime import datetime
from lxml import etree
from PyPDF2 import PdfFileReader
import base64
import markupsafe

import io

import logging

_logger = logging.getLogger(__name__)


DEFAULT_FACTURX_DATE_FORMAT = '%Y%m%d'


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    def _post_invoice_edi(self, invoices):
        self.ensure_one()
        if self.code != 'facturx_1_0_05' or self._is_account_edi_ubl_cii_available():
            return super()._post_invoice_edi(invoices)
        res = {}
        for invoice in invoices:
            attachment = self._export_facturx(invoice)
            res[invoice] = {'success': True, 'attachment': attachment}
        return res

    def _is_embedding_to_invoice_pdf_needed(self):
        # OVERRIDE
        self.ensure_one()
        return True if self.code == 'facturx_1_0_05' else super()._is_embedding_to_invoice_pdf_needed()

    def _get_embedding_to_invoice_pdf_values(self, invoice):
        values = super()._get_embedding_to_invoice_pdf_values(invoice)
        if values and self.code == 'facturx_1_0_05':
            values['name'] = 'factur-x.xml'
        return values

    def _prepare_invoice_report(self, pdf_writer, edi_document):
        self.ensure_one()
        if self.code != 'facturx_1_0_05' or self._is_account_edi_ubl_cii_available():
            return super()._prepare_invoice_report(pdf_writer, edi_document)
        if not edi_document.attachment_id:
            return

        pdf_writer.embed_odoo_attachment(edi_document.attachment_id, subtype='text/xml')
        if not pdf_writer.is_pdfa and str2bool(self.env['ir.config_parameter'].sudo().get_param('edi.use_pdfa', 'False')):
            try:
                pdf_writer.convert_to_pdfa()
            except Exception as e:
                _logger.exception("Error while converting to PDF/A: %s", e)
            metadata_template = self.env.ref('account_edi_facturx.account_invoice_pdfa_3_facturx_metadata', raise_if_not_found=False)
            if metadata_template:
                pdf_writer.add_file_metadata(metadata_template._render({
                    'title': edi_document.move_id.name,
                    'date': fields.Date.context_today(self),
                }).encode())

    def _export_facturx(self, invoice):

        def format_date(dt):
            # Format the date in the Factur-x standard.
            dt = dt or datetime.now()
            return dt.strftime(DEFAULT_FACTURX_DATE_FORMAT)

        def format_monetary(number, currency):
            # Format the monetary values to avoid trailing decimals (e.g. 90.85000000000001).
            if currency.is_zero(number):  # Ensure that we never return -0.0
                number = 0.0
            return float_repr(number, currency.decimal_places)

        self.ensure_one()
        # Create file content.
        seller_siret = 'siret' in invoice.company_id._fields and invoice.company_id.siret or invoice.company_id.company_registry
        buyer_siret = 'siret' in invoice.commercial_partner_id._fields and invoice.commercial_partner_id.siret
        tax_detail_vals = invoice._prepare_edi_tax_details(
                grouping_key_generator=lambda tax_values: {
                    'unece_tax_category_code': tax_values['tax_id']._get_unece_category_code(invoice.commercial_partner_id, invoice.company_id),
                    'amount': tax_values['tax_id'].amount,
                    'amount_type': tax_values['tax_id'].amount_type,  # We need to check the type
                }
            )
        # For compatibility with the old template that was not using a grouped tax details. Only used to get the tax amount.
        # Done here to avoid adding a key in the base method that shouldn't be used. (the tax_amount key should be used)
        tax_details = list(tax_detail_vals['tax_details'].values())
        for line in tax_detail_vals['invoice_line_tax_details'].values():
            tax_details.extend(list(line['tax_details'].values()))
        for tax_detail in tax_details:
            tax_detail['tax'] = tax_detail['group_tax_details'][0]['tax_id']
        template_values = {
            **invoice._prepare_edi_vals_to_export(),
            'tax_details': tax_detail_vals,
            'format_date': format_date,
            'format_monetary': format_monetary,
            'is_html_empty': is_html_empty,
            'seller_specified_legal_organization': seller_siret,
            'buyer_specified_legal_organization': buyer_siret,
            # Chorus PRO fields
            'buyer_reference': 'buyer_reference' in invoice._fields and invoice.buyer_reference or '',
            'contract_reference': 'contract_reference' in invoice._fields and invoice.contract_reference or '',
            'purchase_order_reference': 'purchase_order_reference' in invoice._fields and invoice.purchase_order_reference or '',
        }

        xml_content = markupsafe.Markup("<?xml version='1.0' encoding='UTF-8'?>")
        xml_content += self.env.ref('account_edi_facturx.account_invoice_facturx_export')._render(template_values)
        return self.env['ir.attachment'].create({
            'name': 'factur-x.xml',
            'raw': xml_content.encode(),
            'mimetype': 'application/xml'
        })

    def _is_facturx(self, filename, tree):
        return self.code == 'facturx_1_0_05' and tree.tag == '{urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100}CrossIndustryInvoice'

    def _create_invoice_from_xml_tree(self, filename, tree, journal=None):
        self.ensure_one()
        if self._is_facturx(filename, tree) and not self._is_account_edi_ubl_cii_available():
            return self._import_facturx(tree, self.env['account.move'])
        return super()._create_invoice_from_xml_tree(filename, tree, journal=journal)

    def _update_invoice_from_xml_tree(self, filename, tree, invoice):
        self.ensure_one()
        if self._is_facturx(filename, tree) and not self._is_account_edi_ubl_cii_available():
            return self._import_facturx(tree, invoice)
        return super()._update_invoice_from_xml_tree(filename, tree, invoice)

    def _import_facturx(self, tree, invoice):
        """ Decodes a factur-x invoice into an invoice.

        :param tree:    the factur-x tree to decode.
        :param invoice: the invoice to update or an empty recordset.
        :returns:       the invoice where the factur-x data was imported.
        """

        def _find_value(xpath, element=tree):
            return self._find_value(xpath, element, tree.nsmap)

        amount_total_import = None

        default_move_type = False
        if invoice._context.get('default_journal_id'):
            journal = self.env['account.journal'].browse(self.env.context['default_journal_id'])
            default_move_type = 'out_invoice' if journal.type == 'sale' else 'in_invoice'
        elif invoice._context.get('default_move_type'):
            default_move_type = self._context['default_move_type']
        elif invoice.move_type in self.env['account.move'].get_invoice_types(include_receipts=True):
            # in case an attachment is saved on a draft invoice previously created, we might
            # have lost the default value in context but the type was already set
            default_move_type = invoice.move_type

        if not default_move_type:
            raise UserError(_("No information about the journal or the type of invoice is passed"))
        if default_move_type == 'entry':
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

        default_move_type.replace('_refund', '_invoice')
        if type_code == '381':
            default_move_type = 'out_refund' if default_move_type == 'out_invoice' else 'in_refund'
            refund_sign = -1
        else:
            # Handle 'b' refund mode.
            if total_amount < 0:
                default_move_type = 'out_refund' if default_move_type == 'out_invoice' else 'in_refund'
            refund_sign = -1 if 'refund' in default_move_type else 1

        # Write the type as the journal entry is already created.
        invoice.move_type = default_move_type

        # self could be a single record (editing) or be empty (new).
        with Form(invoice.with_context(default_move_type=default_move_type)) as invoice_form:
            partner_type = invoice_form.journal_id.type == 'purchase' and 'SellerTradeParty' or 'BuyerTradeParty'
            invoice_form.partner_id = self._retrieve_partner(
                name=_find_value(f"//ram:{partner_type}/ram:Name"),
                mail=_find_value(f"//ram:{partner_type}//ram:URIID[@schemeID='SMTP']"),
                vat=_find_value(f"//ram:{partner_type}/ram:SpecifiedTaxRegistration/ram:ID"),
            )

            # Reference.
            elements = tree.xpath('//rsm:ExchangedDocument/ram:ID', namespaces=tree.nsmap)
            if elements:
                invoice_form.ref = elements[0].text

            # Name.
            elements = tree.xpath('//ram:BuyerOrderReferencedDocument/ram:IssuerAssignedID', namespaces=tree.nsmap)
            if elements:
                invoice_form.payment_reference = elements[0].text

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
                    currency_str = elements[0].attrib.get('currencyID', None)

            # Currency.
            if currency_str:
                currency = self._retrieve_currency(currency_str)
                if currency and not currency.active:
                    error_msg = _('The currency (%s) of the document you are uploading is not active in this database.\n'
                                  'Please activate it before trying again to import.', currency.name)
                    error_action = {
                        'view_mode': 'form',
                        'res_model': 'res.currency',
                        'type': 'ir.actions.act_window',
                        'target': 'new',
                        'res_id': currency.id,
                        'views': [[False, 'form']]
                    }
                    raise RedirectWarning(error_msg, error_action, _('Display the currency'))
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
                        name = _find_value('.//ram:SpecifiedTradeProduct/ram:Name', element)
                        invoice_line_form.product_id = self._retrieve_product(
                            default_code=_find_value('.//ram:SpecifiedTradeProduct/ram:SellerAssignedID', element),
                            name=_find_value('.//ram:SpecifiedTradeProduct/ram:Name', element),
                            barcode=_find_value('.//ram:SpecifiedTradeProduct/ram:GlobalID', element)
                        )
                        # force original line description instead of the one copied from product's Sales Description
                        if name:
                            invoice_line_form.name = name

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
                                        invoice_line_form.discount = (invoice_line_form.price_unit - net_unit_price) / invoice_line_form.price_unit * 100.
                        else:
                            line_elements = element.xpath('.//ram:NetPriceProductTradePrice/ram:ChargeAmount', namespaces=tree.nsmap)
                            if line_elements:
                                quantity_elements = element.xpath('.//ram:NetPriceProductTradePrice/ram:BasisQuantity', namespaces=tree.nsmap)
                                if quantity_elements:
                                    invoice_line_form.price_unit = float(line_elements[0].text) / float(quantity_elements[0].text)
                                else:
                                    invoice_line_form.price_unit = float(line_elements[0].text)

                        # Taxes
                        tax_element = element.xpath('.//ram:SpecifiedLineTradeSettlement/ram:ApplicableTradeTax/ram:RateApplicablePercent', namespaces=tree.nsmap)
                        invoice_line_form.tax_ids.clear()
                        for eline in tax_element:
                            tax = self._retrieve_tax(
                                amount=eline.text,
                                type_tax_use=invoice_form.journal_id.type
                            )
                            if tax:
                                invoice_line_form.tax_ids.add(tax)

            elif amount_total_import:
                # No lines in BASICWL.
                with invoice_form.invoice_line_ids.new() as invoice_line_form:
                    invoice_line_form.name = invoice_form.comment or '/'
                    invoice_line_form.quantity = 1
                    invoice_line_form.price_unit = amount_total_import

        return invoice_form.save()

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

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-

from . import account_edi_format
from . import account_tax

```

## File: test_file\test_facturx.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<rsm:CrossIndustryInvoice xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100" 
    xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100" 
    xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100">
    <rsm:ExchangedDocumentContext>
        <ram:GuidelineSpecifiedDocumentContextParameter>
            <ram:ID>urn:cen.eu:en16931:2017</ram:ID>
        </ram:GuidelineSpecifiedDocumentContextParameter>
    </rsm:ExchangedDocumentContext>

    <rsm:ExchangedDocument>
        <ram:TypeCode>380</ram:TypeCode>
        <ram:IssueDateTime>
            <udt:DateTimeString format="102">20200501</udt:DateTimeString>
        </ram:IssueDateTime>
    </rsm:ExchangedDocument>

    <rsm:SupplyChainTradeTransaction>
        <ram:IncludedSupplyChainTradeLineItem>
            <ram:AssociatedDocumentLineDocument>
                <ram:LineID>1</ram:LineID>
            </ram:AssociatedDocumentLineDocument>

            <ram:SpecifiedTradeProduct>
                <ram:SellerAssignedID>FURN_6741</ram:SellerAssignedID>
                <ram:Name>[FURN_6741] Large Meeting Table
Conference room table</ram:Name>
            </ram:SpecifiedTradeProduct>

            <ram:SpecifiedLineTradeAgreement>
                <ram:GrossPriceProductTradePrice>
                    <ram:ChargeAmount currencyID="USD">642.00</ram:ChargeAmount>
                </ram:GrossPriceProductTradePrice>
            </ram:SpecifiedLineTradeAgreement>

            <ram:SpecifiedLineTradeDelivery>
                <ram:BilledQuantity>5.0</ram:BilledQuantity>
            </ram:SpecifiedLineTradeDelivery>

            <ram:SpecifiedLineTradeSettlement>

                <ram:SpecifiedTradeSettlementLineMonetarySummation>
                    <ram:LineTotalAmount currencyID="USD">3210.00</ram:LineTotalAmount>
                </ram:SpecifiedTradeSettlementLineMonetarySummation>
            </ram:SpecifiedLineTradeSettlement>

        </ram:IncludedSupplyChainTradeLineItem>

        <ram:IncludedSupplyChainTradeLineItem>
            <ram:AssociatedDocumentLineDocument>
                <ram:LineID>2</ram:LineID>
            </ram:AssociatedDocumentLineDocument>

            <ram:SpecifiedTradeProduct>
                <ram:SellerAssignedID>FURN_8220</ram:SellerAssignedID>
                <ram:Name>[FURN_8220] Four Person Desk
Four person modern office workstation</ram:Name>
            </ram:SpecifiedTradeProduct>

            <ram:SpecifiedLineTradeAgreement>
                <ram:GrossPriceProductTradePrice>
                    <ram:ChargeAmount currencyID="USD">280.00</ram:ChargeAmount>
                </ram:GrossPriceProductTradePrice>
            </ram:SpecifiedLineTradeAgreement>

            <ram:SpecifiedLineTradeDelivery>
                <ram:BilledQuantity>5.0</ram:BilledQuantity>
            </ram:SpecifiedLineTradeDelivery>

            <ram:SpecifiedLineTradeSettlement>
                <ram:ApplicableTradeTax>
                    <ram:RateApplicablePercent>0.0</ram:RateApplicablePercent>
                </ram:ApplicableTradeTax>
                <ram:SpecifiedTradeSettlementLineMonetarySummation>
                    <ram:LineTotalAmount currencyID="USD">1400.00</ram:LineTotalAmount>
                </ram:SpecifiedTradeSettlementLineMonetarySummation>
            </ram:SpecifiedLineTradeSettlement>

        </ram:IncludedSupplyChainTradeLineItem>

        <ram:ApplicableHeaderTradeAgreement>
            <ram:SellerTradeParty>
                <ram:Name>YourCompany</ram:Name>
                <ram:DefinedTradeContact>
                    <ram:PersonName>YourCompany</ram:PersonName>
                    <ram:TelephoneUniversalCommunication>
                        <ram:CompleteNumber>+1 (650) 691-3277 </ram:CompleteNumber>
                    </ram:TelephoneUniversalCommunication>
                    <ram:EmailURIUniversalCommunication>
                        <ram:URIID schemeID="SMTP">info@yourcompany.example.com</ram:URIID>
                    </ram:EmailURIUniversalCommunication>
                </ram:DefinedTradeContact>

                <ram:PostalTradeAddress>
                    <ram:PostcodeCode>94134</ram:PostcodeCode>
                    <ram:LineOne>250 Executive Park Blvd, Suite 3400</ram:LineOne>

                    <ram:CityName>San Francisco</ram:CityName>
                    <ram:CountryID>US</ram:CountryID>
                </ram:PostalTradeAddress>
                <ram:SpecifiedTaxRegistration>
                    <ram:ID schemeID="VA">VAT789</ram:ID>
                </ram:SpecifiedTaxRegistration>
            </ram:SellerTradeParty>

            <ram:BuyerTradeParty>
                <ram:Name>Azure Interior</ram:Name>
                <ram:DefinedTradeContact>
                    <ram:PersonName>Azure Interior</ram:PersonName>
                    <ram:TelephoneUniversalCommunication>
                        <ram:CompleteNumber>(870)-931-0505</ram:CompleteNumber>
                    </ram:TelephoneUniversalCommunication>
                    <ram:EmailURIUniversalCommunication>
                        <ram:URIID schemeID="SMTP">azure.Interior24@example.com</ram:URIID>
                    </ram:EmailURIUniversalCommunication>
                </ram:DefinedTradeContact>

                <ram:PostalTradeAddress>
                    <ram:PostcodeCode>94538</ram:PostcodeCode>
                    <ram:LineOne>4557 De Silva St</ram:LineOne>

                    <ram:CityName>Fremont</ram:CityName>
                    <ram:CountryID>US</ram:CountryID>
                </ram:PostalTradeAddress>
                <ram:SpecifiedTaxRegistration>
                    <ram:ID schemeID="VA">VAT12345</ram:ID>
                </ram:SpecifiedTaxRegistration>
            </ram:BuyerTradeParty>

            <ram:BuyerOrderReferencedDocument>
                <ram:IssuerAssignedID>INV/2020/05/0001: INV/2020/05/0001</ram:IssuerAssignedID>
            </ram:BuyerOrderReferencedDocument>
        </ram:ApplicableHeaderTradeAgreement>

        <ram:ApplicableHeaderTradeDelivery>
        </ram:ApplicableHeaderTradeDelivery>

        <ram:ApplicableHeaderTradeSettlement>
            <ram:SpecifiedTradePaymentTerms>
                <ram:Description>End of Following Month</ram:Description>
                <ram:DueDateDateTime>
                    <udt:DateTimeString>20200630</udt:DateTimeString>
                </ram:DueDateDateTime>
            </ram:SpecifiedTradePaymentTerms>

            <ram:SpecifiedTradeSettlementHeaderMonetarySummation>
                <ram:LineTotalAmount currencyID="USD">4610.00</ram:LineTotalAmount>
                <ram:TaxBasisTotalAmount currencyID="USD">0.00</ram:TaxBasisTotalAmount>
                <ram:TaxTotalAmount currencyID="USD">0.00</ram:TaxTotalAmount>
                <ram:GrandTotalAmount currencyID="USD">4610.00</ram:GrandTotalAmount>
                <ram:TotalPrepaidAmount currencyID="USD">0.00</ram:TotalPrepaidAmount>
                <ram:DuePayableAmount currencyID="USD">4610.00</ram:DuePayableAmount>
            </ram:SpecifiedTradeSettlementHeaderMonetarySummation>
        </ram:ApplicableHeaderTradeSettlement>

    </rsm:SupplyChainTradeTransaction>
</rsm:CrossIndustryInvoice>

```

