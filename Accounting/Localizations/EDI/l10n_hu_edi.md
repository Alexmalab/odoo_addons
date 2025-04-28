# Odoo Module: l10n_hu_edi

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard


def post_init(env):
    for company in env['res.company'].search([('chart_template', '=', 'hu')], order="parent_path"):
        # Apply default cash rounding configuration
        company._l10n_hu_edi_configure_company()

        # Set Hungarian fields on taxes
        env['account.chart.template'].with_company(company)._load_data({
            'account.tax': {
                xmlid: vals
                for xmlid, vals in env['account.chart.template']._get_hu_account_tax().items()
                if env['account.chart.template'].with_company(company).ref(xmlid, raise_if_not_found=False)
            }
        })

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Hungary - E-invoicing',
    'version': '1.0.0',
    'category': 'Accounting/Localizations/EDI',
    'author': 'DO Tech (OdooTech Zrt.), BDSC Business Consulting Kft. & Odoo S.A.',
    'description': """
* Electronically report invoices to the NAV (Hungarian Tax Agency) when issuing physical (paper) invoices.
* Perform the Tax Audit Export (Adóhatósági Ellenőrzési Adatszolgáltatás) in NAV 3.0 format.
    """,
    'website': 'https://www.odootech.hu',
    'depends': ['account_debit_note', 'base_iban', 'l10n_hu'],
    'data': [
        'security/ir.model.access.csv',
        'data/uom.uom.csv',
        'data/account_cash_rounding.xml',
        'data/template_requests.xml',
        'data/template_invoice.xml',
        'data/ir_cron.xml',
        'views/report_templates.xml',
        'views/report_invoice.xml',
        'views/account_move_views.xml',
        'views/product_template_views.xml',
        'views/account_tax_views.xml',
        'views/uom_uom_views.xml',
        'views/res_partner_views.xml',
        'views/res_company_views.xml',
        'views/res_config_settings_views.xml',
        'wizard/l10n_hu_edi_cancellation.xml',
        'wizard/l10n_hu_edi_tax_audit_export.xml',
    ],
    'demo': [
        'demo/demo_partner.xml',
    ],
    'post_init_hook': 'post_init',
    'auto_install': ['l10n_hu'],
    'license': 'LGPL-3',
}

```

## File: data\account_cash_rounding.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="cash_rounding_1_huf" model="account.cash.rounding">
        <field name="name">Rounding to 1.00</field>
        <field name="rounding">1.0</field>
        <field name="strategy">add_invoice_line</field>
    </record>
</odoo>

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <record id="ir_cron_update_status" model="ir.cron">
            <field name="name">NAV 3.0: Update status of pending invoices</field>
            <field name="model_id" ref="account.model_account_move"/>
            <field name="state">code</field>
            <field name="code">
    env['account.move.send']._l10n_hu_edi_cron_update_status()
            </field>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="nextcall" eval="DateTime.now().strftime('%Y-%m-%d 22:00:00')"/>
        </record>
     </data>
</odoo>

```

## File: data\neutralize.sql

```sql
-- Disable production mode for Hungary EDI
UPDATE res_company
   SET l10n_hu_edi_server_mode = 'test',
       l10n_hu_edi_username = '',
       l10n_hu_edi_password = '',
       l10n_hu_edi_signature_key = '',
       l10n_hu_edi_replacement_key = ''
 WHERE l10n_hu_edi_server_mode = 'production';

```

## File: data\template_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="nav_online_invoice_xml_3_0">
        <InvoiceData xmlns="http://schemas.nav.gov.hu/OSA/3.0/data" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://schemas.nav.gov.hu/OSA/3.0/data invoiceData.xsd"
            xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common" xmlns:base="http://schemas.nav.gov.hu/OSA/3.0/base">
            <invoiceNumber t-out="invoice.name"/>
            <invoiceIssueDate t-out="invoiceIssueDate"/>
            <completenessIndicator t-out="format_bool(completenessIndicator)"/>
            <invoiceMain>
                <invoice>
                    <invoiceReference t-if="base_invoice">
                        <originalInvoiceNumber t-out="base_invoice.name"/>
                        <modifyWithoutMaster t-out="format_bool(modifyWithoutMaster)"/>
                        <modificationIndex t-out="invoice.l10n_hu_invoice_chain_index"/>
                    </invoiceReference>
                    <invoiceHead>
                        <supplierInfo>
                            <supplierTaxNumber>
                                <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_tax_number">
                                    <t t-set="vat" t-value="supplier_vat_data['tax_number']"/>
                                </t>
                            </supplierTaxNumber>
                            <groupMemberTaxNumber t-if="supplier_vat_data.get('group_member_tax_number')">
                                <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_tax_number">
                                    <t t-set="vat" t-value="supplier_vat_data.get('group_member_tax_number')"/>
                                </t>
                            </groupMemberTaxNumber>
                            <supplierName t-out="supplier.name"/>
                            <supplierAddress>
                                <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_simple_address">
                                    <t t-set="partner" t-value="supplier"/>
                                </t>
                            </supplierAddress>
                            <supplierBankAccountNumber t-out="supplierBankAccountNumber"/>
                            <individualExemption t-out="format_bool(individualExemption)"/>
                        </supplierInfo>
                        <customerInfo>
                            <customerVatStatus t-out="customerVatStatus"/>
                            <customerVatData t-if="customer.is_company">
                                <customerTaxNumber t-if="customer_vat_data.get('tax_number')">
                                    <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_tax_number">
                                        <t t-set="vat" t-value="customer_vat_data.get('tax_number')"/>
                                    </t>
                                    <groupMemberTaxNumber t-if="customer_vat_data.get('group_member_tax_number')">
                                        <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_tax_number">
                                            <t t-set="vat" t-value="customer_vat_data.get('group_member_tax_number')"/>
                                        </t>
                                    </groupMemberTaxNumber>
                                </customerTaxNumber>
                                <communityVatNumber t-out="customer_vat_data.get('community_vat_number')"/>
                                <thirdStateTaxId t-out="customer_vat_data.get('third_state_tax_id')"/>
                            </customerVatData>
                            <customerName t-if="customer.is_company" t-out="customer.name"/>
                            <customerAddress t-if="customer.is_company">
                                <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_simple_address">
                                    <t t-set="partner" t-value="customer"/>
                                </t>
                            </customerAddress>
                            <customerBankAccountNumber t-out="customerBankAccountNumber"/>
                        </customerInfo>
                        <invoiceDetail>
                            <invoiceCategory t-translation="off">NORMAL</invoiceCategory>
                            <invoiceDeliveryDate t-out="invoice.delivery_date or invoice.invoice_date"/>
                            <smallBusinessIndicator t-out="format_bool(smallBusinessIndicator)"/>
                            <currencyCode t-out="invoice.currency_id.name"/>
                            <exchangeRate t-out="float_repr(exchangeRate, 6)"/>
                            <paymentMethod t-if="invoice.l10n_hu_payment_mode" t-out="invoice.l10n_hu_payment_mode"/>
                            <paymentDate t-if="invoice.invoice_date_due" t-out="invoice.invoice_date_due"/>
                            <cashAccountingIndicator t-out="format_bool(cashAccountingIndicator)"/>
                            <invoiceAppearance t-translation="off">PAPER</invoiceAppearance>
                        </invoiceDetail>
                    </invoiceHead>
                    <invoiceLines>
                        <mergedItemIndicator t-out="format_bool(mergedItemIndicator)"/>
                        <line t-foreach="lines_values" t-as="line_values">
                            <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_line"/>
                        </line>
                    </invoiceLines>
                    <invoiceSummary>
                        <summaryNormal>
                            <summaryByVatRate t-foreach="tax_summary" t-as="tax_vals">
                                <vatRate>
                                    <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_vat_rate">
                                        <t t-set="tax" t-value="tax_vals['vat_tax']"/>
                                        <t t-set="vatPercentage" t-value="tax_vals['vatPercentage']"/>
                                    </t>
                                </vatRate>
                                <vatRateNetData>
                                    <vatRateNetAmount t-out="float_repr(tax_vals['vatRateNetAmount'], 2)"/>
                                    <vatRateNetAmountHUF t-out="float_repr(tax_vals['vatRateNetAmountHUF'], 2)"/>
                                </vatRateNetData>
                                <vatRateVatData>
                                    <vatRateVatAmount t-out="float_repr(tax_vals['vatRateVatAmount'], 2)"/>
                                    <vatRateVatAmountHUF t-out="float_repr(tax_vals['vatRateVatAmountHUF'], 2)"/>
                                </vatRateVatData>
                            </summaryByVatRate>
                            <invoiceNetAmount t-out="float_repr(invoiceNetAmount, 2)"/>
                            <invoiceNetAmountHUF t-out="float_repr(invoiceNetAmountHUF, 2)"/>
                            <invoiceVatAmount t-out="float_repr(invoiceVatAmount, 2)"/>
                            <invoiceVatAmountHUF t-out="float_repr(invoiceVatAmountHUF, 2)"/>
                        </summaryNormal>
                        <summaryGrossData>
                            <invoiceGrossAmount t-out="float_repr(invoiceGrossAmount, 2)"/>
                            <invoiceGrossAmountHUF t-out="float_repr(invoiceGrossAmountHUF, 2)"/>
                        </summaryGrossData>
                    </invoiceSummary>
                </invoice>
            </invoiceMain>
        </InvoiceData>
    </template>

    <template id="nav_online_invoice_xml_3_0_simple_address" xmlns:base="http://schemas.nav.gov.hu/OSA/3.0/base">
        <base:simpleAddress>
            <base:countryCode t-out="partner.country_code"/>
            <base:region t-if="partner.state_id" t-out="partner.state_id.name"/>
            <base:postalCode t-out="partner.zip"/>
            <base:city t-out="partner.city"/>
            <base:additionalAddressDetail t-out="partner.street"/>
        </base:simpleAddress>
    </template>

    <template id="nav_online_invoice_xml_3_0_tax_number" xmlns:base="http://schemas.nav.gov.hu/OSA/3.0/base">
        <t t-if="'-' in vat">
            <base:taxpayerId t-out="vat[:8]"/>
            <base:vatCode t-out="vat[9:10]"/>
            <base:countyCode t-out="vat[11:13]"/>
        </t>
        <t t-else="else">
            <base:taxpayerId t-out="vat[:8]"/>
            <base:vatCode t-out="vat[8:9]"/>
            <base:countyCode t-out="vat[9:11]"/>
        </t>
    </template>

    <template id="nav_online_invoice_xml_3_0_vat_rate">
        <vatPercentage t-if="tax.l10n_hu_tax_type == 'VAT'" t-out="float_repr(vatPercentage, 4)"/>
        <vatExemption t-if="tax.l10n_hu_tax_type in ['AAM','TAM','KBAET','KBAUK','EAM','NAM']">
            <case t-out="tax.l10n_hu_tax_type"/>
            <reason t-out="tax.l10n_hu_tax_reason"/>
        </vatExemption>
        <vatOutOfScope t-if="tax.l10n_hu_tax_type in ['ATK','EUFAD37','EUFADE','EUE','HO']">
            <case t-out="tax.l10n_hu_tax_type"/>
            <reason t-out="tax.l10n_hu_tax_reason"/>
        </vatOutOfScope>
        <vatDomesticReverseCharge t-if="tax.l10n_hu_tax_type == 'DOMESTIC_REVERSE'" t-translation="off">true</vatDomesticReverseCharge>
        <marginSchemeIndicator t-if="tax.l10n_hu_tax_type in ['TRAVEL_AGENCY', 'SECOND_HAND', 'ARTWORK', 'ANTIQUES']" t-out="tax.l10n_hu_tax_type"/>
        <vatAmountMismatch t-if="tax.l10n_hu_tax_type in ['REFUNDABLE_VAT', 'NONREFUNDABLE_VAT']">
            <vatRate>
                <vatPercentage t-out="float_repr(vatPercentage, 4)"/>
            </vatRate>
        </vatAmountMismatch>
        <noVatCharge t-if="tax.l10n_hu_tax_type == 'NO_VAT'" t-translation="off">true</noVatCharge>
    </template>
    
    <template id="nav_online_invoice_xml_3_0_line">
        <t t-set="line" t-value="line_values['line']"/>
        <lineNumber t-out="line_values['lineNumber']"/>
        <lineModificationReference t-if="line_values['lineNumberReference']">
            <lineNumberReference t-out="line_values['lineNumberReference']"/>
            <lineOperation t-translation="off">CREATE</lineOperation>
        </lineModificationReference>
        <advanceData t-if="line_values.get('advanceIndicator')">
            <advanceIndicator t-translation="off">true</advanceIndicator>
            <advancePaymentData t-if="line_values.get('advanceOriginalInvoice')">
                <advanceOriginalInvoice t-out="line_values.get('advanceOriginalInvoice')"/>
                <advancePaymentDate t-out="line_values.get('advancePaymentDate')"/>
                <advanceExchangeRate t-out="float_repr(line_values.get('advanceExchangeRate'), 6)"/>
            </advancePaymentData>
        </advanceData>
        <productCodes t-if="line.product_id.l10n_hu_product_code_type or line.product_id.default_code">
            <productCode t-if="line.product_id.l10n_hu_product_code_type">
                <productCodeCategory t-out="line.product_id.l10n_hu_product_code_type"/>
                <productCodeValue t-out="line.product_id.l10n_hu_product_code"/>
            </productCode>
            <productCode t-if="line.product_id.default_code">
                <productCodeCategory t-translation="off">OWN</productCodeCategory>
                <productCodeOwnValue t-out="line.product_id.default_code"/>
            </productCode>
        </productCodes>
        <lineExpressionIndicator t-out="format_bool(line_values['lineExpressionIndicator'])"/>
        <lineNatureIndicator t-out="line_values['lineNatureIndicator']"/>
        <lineDescription t-out="line_values['lineDescription']"/>
        <t t-if="line_values['lineExpressionIndicator']">
            <quantity t-out="line_values['quantity']"/>
            <unitOfMeasure t-out="line.product_uom_id.l10n_hu_edi_code or 'OWN'"/>
            <unitOfMeasureOwn t-if="not line.product_uom_id.l10n_hu_edi_code" t-out="line.product_uom_id.name"/>
            <unitPrice t-out="float_repr(line_values['unitPrice'], 6)"/>
            <unitPriceHUF t-out="float_repr(line_values['unitPriceHUF'], 6)"/>
        </t>
        <lineDiscountData t-if="line.discount">
            <discountValue t-out="float_repr(line_values['discountValue'], 2)"/>
            <discountRate t-out="float_repr(line_values['discountRate'], 4)"/>
        </lineDiscountData>
        <lineAmountsNormal>
            <lineNetAmountData>
                <lineNetAmount t-out="float_repr(line_values['lineNetAmount'], 2)"/>
                <lineNetAmountHUF t-out="float_repr(line_values['lineNetAmountHUF'], 2)"/>
            </lineNetAmountData>
            <lineVatRate>
                <t t-call="l10n_hu_edi.nav_online_invoice_xml_3_0_vat_rate">
                    <t t-set="tax" t-value="line_values['vat_tax']"/>
                    <t t-set="vatPercentage" t-value="line_values['vatPercentage']"/>
                </t>
            </lineVatRate>
            <lineVatData t-if="line_values['lineVatData']">
                <lineVatAmount t-out="float_repr(line_values['lineVatAmount'], 2)"/>
                <lineVatAmountHUF t-out="float_repr(line_values['lineVatAmountHUF'], 2)"/>
            </lineVatData>
            <lineGrossAmountData>
                <lineGrossAmountNormal t-out="float_repr(line_values['lineGrossAmountNormal'], 2)"/>
                <lineGrossAmountNormalHUF t-out="float_repr(line_values['lineGrossAmountNormalHUF'], 2)"/>
            </lineGrossAmountData>
        </lineAmountsNormal>
    </template>
</odoo>

```

## File: data\template_requests.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="request_header" xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common">
        <common:header>
            <common:requestId t-out="requestId"/>
            <common:timestamp t-out="timestamp"/>
            <common:requestVersion t-translation="off">3.0</common:requestVersion>
            <common:headerVersion t-translation="off">1.0</common:headerVersion>
        </common:header>
        <common:user>
            <common:login t-out="login"/>
            <common:passwordHash cryptoType="SHA-512" t-out="passwordHash"/>
            <common:taxNumber t-out="taxNumber"/>
            <common:requestSignature cryptoType="SHA3-512" t-out="requestSignature"/>
        </common:user>
        <software>
            <softwareId t-out="softwareId"/>
            <softwareName t-out="softwareName"/>
            <softwareOperation t-out="softwareOperation"/>
            <softwareMainVersion t-out="softwareMainVersion"/>
            <softwareDevName t-out="softwareDevName"/>
            <softwareDevContact t-out="softwareDevContact"/>
            <softwareDevCountryCode t-out="softwareDevCountryCode"/>
            <softwareDevTaxNumber t-out="softwareDevTaxNumber"/>
        </software>
    </template>

    <template id="token_exchange_request">
        <TokenExchangeRequest xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common" xmlns="http://schemas.nav.gov.hu/OSA/3.0/api">
            <t t-call="l10n_hu_edi.request_header"/>
        </TokenExchangeRequest>
    </template>

    <template id="manage_invoice_request">
        <ManageInvoiceRequest xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common" xmlns="http://schemas.nav.gov.hu/OSA/3.0/api">
            <t t-call="l10n_hu_edi.request_header"/>
            <exchangeToken t-out="exchangeToken"/>
            <invoiceOperations>
                <compressedContent t-out="format_bool(compressedContent)"/>
                <invoiceOperation t-foreach="invoices" t-as="invoice_data">
                    <index t-out="invoice_data['index']"/>
                    <invoiceOperation t-out="invoice_data['invoiceOperation']"/>
                    <invoiceData t-out="invoice_data['invoiceData']"/>
                </invoiceOperation>
            </invoiceOperations>
        </ManageInvoiceRequest>
    </template>

    <template id="query_transaction_status_request">
        <QueryTransactionStatusRequest xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common" xmlns="http://schemas.nav.gov.hu/OSA/3.0/api">
            <t t-call="l10n_hu_edi.request_header"/>
            <transactionId t-out="transactionId"/>
            <returnOriginalRequest t-out="format_bool(returnOriginalRequest)"/>
        </QueryTransactionStatusRequest>
    </template>

    <template id="query_transaction_list_request">
        <QueryTransactionListRequest xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common" xmlns="http://schemas.nav.gov.hu/OSA/3.0/api">
            <t t-call="l10n_hu_edi.request_header"/>
            <page t-out="page"/>
            <insDate>
                <dateTimeFrom t-out="dateTimeFrom"/>
                <dateTimeTo t-out="dateTimeTo"/>
            </insDate>
        </QueryTransactionListRequest>
    </template>

    <template id="manage_annulment_request">
        <ManageAnnulmentRequest xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common" xmlns="http://schemas.nav.gov.hu/OSA/3.0/api">
            <t t-call="l10n_hu_edi.request_header"/>
            <exchangeToken t-out="exchangeToken"/>
            <annulmentOperations>
                <annulmentOperation t-foreach="annulments" t-as="annulment_data">
                    <index t-out="annulment_data['index']"/>
                    <annulmentOperation t-out="annulment_data['annulmentOperation']"/>
                    <invoiceAnnulment t-out="annulment_data['invoiceAnnulment']"/>
                </annulmentOperation>
            </annulmentOperations>
        </ManageAnnulmentRequest>
    </template>

    <template id="invoice_annulment">
        <InvoiceAnnulment
                xmlns="http://schemas.nav.gov.hu/OSA/3.0/annul"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://schemas.nav.gov.hu/OSA/3.0/data invoiceAnnulment.xsd"
                xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common"
                xmlns:base="http://schemas.nav.gov.hu/OSA/3.0/base">
            <annulmentReference t-out="annulmentReference"/>
            <annulmentTimestamp t-out="annulmentTimestamp"/>
            <annulmentCode t-out="annulmentCode"/>
            <annulmentReason t-out="annulmentReason"/>
        </InvoiceAnnulment>
    </template>

</odoo>

```

## File: data\uom.uom.csv

```csv
"id","l10n_hu_edi_code"
"uom.product_uom_unit","PIECE"
"uom.product_uom_day","DAY"
"uom.product_uom_hour","HOUR"
"uom.product_uom_meter","METER"
"uom.product_uom_km","KILOMETER"
"uom.product_uom_litre","LITER"
"uom.product_uom_cubic_meter","CUBIC_METER"
"uom.product_uom_kgm","KILOGRAM"
"uom.product_uom_ton","TON"

```

## File: data\template\account.tax-hu.csv

```csv
"id","l10n_hu_tax_type"
"F27","VAT"
"F18","VAT"
"F5","VAT"
"FA","AAM"
"FT","TAM"
"FKKS","ATK"
"FKKT","ATK"
"FF","DOMESTIC_REVERSE"
"FEUSZ","KBAET"
"FEUT","KBAET"
"FEXS","NAM"
"FEXT","EAM"
"V27","VAT"
"V27TE","VAT"
"V18","VAT"
"V5","VAT"
"VKOMP7","VAT"
"VKOMP12","VAT"
"VA","AAM"
"VT","TAM"
"VKKS","ATK"
"VKKT","ATK"
"VF","DOMESTIC_REVERSE"
"VEU27S","EUFAD37"
"VEU27T","EUFAD37"
"VEU27TE","EUFAD37"
"VEUM","EUFADE"
"VIMS","HO"
"VIMK","HO"
"VIM","HO"

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import math
import base64
import logging
import re

from lxml import etree
from psycopg2.errors import LockNotAvailable

from odoo import fields, models, api, _
from odoo.http import request
from odoo.exceptions import UserError, ValidationError
from odoo.tools import formatLang, float_compare, float_is_zero, float_round, float_repr, cleanup_xml_node, groupby
from odoo.tools.misc import split_every
from odoo.addons.base_iban.models.res_partner_bank import normalize_iban
from odoo.addons.l10n_hu_edi.models.l10n_hu_edi_connection import format_bool, L10nHuEdiConnection, L10nHuEdiConnectionError

_logger = logging.getLogger(__name__)


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_hu_payment_mode = fields.Selection(
        [
            ("TRANSFER", "Transfer"),
            ("CASH", "Cash"),
            ("CARD", "Credit/debit card"),
            ("VOUCHER", "Voucher"),
            ("OTHER", "Other"),
        ],
        string="Payment mode",
        help="NAV expected payment mode of the invoice.",
    )

    # === EDI Fields === #
    l10n_hu_edi_state = fields.Selection(
        ######################################################################################################################
        # STATE DIAGRAM
        # * False, rejected, cancelled --[upload]--> False, sent, send_timeout
        # * sent --[query_status]--> sent, confirmed, confirmed_warning, rejected
        # * confirmed, confirmed_warning --[request_cancel]--> cancel_sent, cancel_timeout
        # * cancel_sent, cancel_pending --[query_status]--> confirmed_warning, cancel_pending, cancelled
        # * send_timeout --[recover_timeout]--> False, send_timeout, confirmed, confirmed_warning, rejected,
        # * cancel_timeout --[recover_timeout]--> confirmed_warning, cancel_sent, cancel_timeout, cancel_pending, cancelled
        ######################################################################################################################
        selection=[
            ('sent', 'Sent, waiting for response'),
            ('send_timeout', 'Timeout when sending'),
            ('confirmed', 'Confirmed'),
            ('confirmed_warning', 'Confirmed with warnings'),
            ('rejected', 'Rejected'),
            ('cancel_sent', 'Cancellation request sent'),
            ('cancel_timeout', 'Timeout when requesting cancellation'),
            ('cancel_pending', 'Cancellation request pending'),
            ('cancelled', 'Cancelled'),
        ],
        string='NAV 3.0 status',
        copy=False,
        index='btree_not_null',
    )
    l10n_hu_edi_batch_upload_index = fields.Integer(
        string='Index of invoice within a batch upload',
        copy=False,
    )
    l10n_hu_edi_attachment = fields.Binary(
        string='Invoice XML file',
        attachment=True,
        copy=False,
    )
    l10n_hu_edi_send_time = fields.Datetime(
        string='Invoice upload time',
        copy=False,
    )
    l10n_hu_edi_transaction_code = fields.Char(
        string='Transaction Code',
        index='trigram',
        copy=False,
        tracking=True,
    )

    # A dict with the following structure:
    # {
    #     'error_title': the main heading of the message
    #     'errors': a list of message items
    #     'blocking_level': {'error' | 'warning' | None}
    #         directs which blocking behaviour to adopt in the Send and Print:
    #         * error: blocks PDF generation and sending by e-mail
    #         * warning: PDF is generated and sent by e-mail, but a warning appears in the banner
    #         * None: PDF is generated and sent by e-mail, no warning appears
    # }
    l10n_hu_edi_messages = fields.Json(
        string='Transaction messages (JSON)',
        copy=False,
    )

    l10n_hu_invoice_chain_index = fields.Integer(
        string='Invoice Chain Index',
        help="""
            Index in the chain of modification invoices:
                -1 for a base invoice;
                1, 2, 3, ... for modification invoices;
                0 for rejected/cancelled invoices or if it has not yet been set.
            """,
        copy=False,
    )
    l10n_hu_edi_attachment_filename = fields.Char(
        string='Invoice XML filename',
        compute='_compute_l10n_hu_edi_attachment_filename',
    )
    l10n_hu_edi_message_html = fields.Html(
        string='Transaction messages',
        compute='_compute_message_html',
    )

    # === Constraints === #

    @api.constrains('l10n_hu_edi_state', 'state')
    def _check_posted_if_active(self):
        """ Enforce the constraint that you cannot reset to draft / cancel a posted invoice if it was already sent to NAV. """
        for move in self:
            if move.state in ['draft', 'cancel'] and move.l10n_hu_edi_state not in [False, 'rejected', 'cancelled']:
                raise ValidationError(_('Cannot reset to draft or cancel invoice %s because an electronic document was already sent to NAV!', move.name))

    # === Computes === #
    @api.depends('delivery_date')
    def _compute_invoice_currency_rate(self):
        # In Hungary, the currency rate should be based on the delivery date.
        super()._compute_invoice_currency_rate()

    def _get_invoice_currency_rate_date(self):
        self.ensure_one()
        if self.country_code == 'HU' and self.delivery_date:
            return self.delivery_date
        return super()._get_invoice_currency_rate_date()

    @api.depends('l10n_hu_edi_messages')
    def _compute_message_html(self):
        for move in self:
            if move.l10n_hu_edi_messages:
                move.l10n_hu_edi_message_html = self.env['account.move.send']._format_error_html(move.l10n_hu_edi_messages)
            else:
                move.l10n_hu_edi_message_html = False

    @api.depends('l10n_hu_edi_state', 'state')
    def _compute_show_reset_to_draft_button(self):
        super()._compute_show_reset_to_draft_button()
        self.filtered(lambda m: m.l10n_hu_edi_state not in [False, 'rejected', 'cancelled']).show_reset_to_draft_button = False

    @api.depends('l10n_hu_edi_state')
    def _compute_need_cancel_request(self):
        # EXTEND 'account' to add dependencies
        return super()._compute_need_cancel_request()

    @api.depends('name')
    def _compute_l10n_hu_edi_attachment_filename(self):
        for move in self:
            move.l10n_hu_edi_attachment_filename = f'{move.name.replace("/", "_")}.xml' if move.name else 'nav30.xml'

    # === Overrides === #

    def _need_cancel_request(self):
        # EXTEND account
        # Technical annulment should be available only in debug mode
        return super()._need_cancel_request() or (self.l10n_hu_edi_state in ['confirmed', 'confirmed_warning'] and request and request.session.debug)

    def button_request_cancel(self):
        # EXTEND 'account'
        if self._need_cancel_request() and self.l10n_hu_edi_state in ['confirmed', 'confirmed_warning']:
            return {
                "name": _("Technical Annulment"),
                "type": "ir.actions.act_window",
                "view_type": "form",
                "view_mode": "form",
                "res_model": "l10n_hu_edi.cancellation",
                "target": "new",
                "context": {"default_invoice_id": self.id},
            }

        return super().button_request_cancel()

    # === Actions === #

    def l10n_hu_edi_button_update_status(self, from_cron=False):
        """ Attempt to update the status of the invoices in `self` """
        invoices_to_query = self.filtered(lambda m: 'query_status' in m._l10n_hu_edi_get_valid_actions())

        with L10nHuEdiConnection(self.env) as connection:
            # Call `query_status` on the invoices.
            invoices_to_query._l10n_hu_edi_query_status(connection)

            # Attempt to recover missing transactions, if any invoice is missing a transaction code
            # or has a duplicate error.
            recover_transactions_error = False
            if any(
                not m.l10n_hu_edi_transaction_code
                or any(
                    'INVOICE_NUMBER_NOT_UNIQUE' in error or 'ANNULMENT_IN_PROGRESS' in error
                    for error in m.l10n_hu_edi_messages['errors']
                )
                for m in self
            ):
                recover_transactions_error = self.company_id._l10n_hu_edi_recover_transactions(connection)

        # Error handling.
        for invoice in invoices_to_query:
            # Log invoice status in chatter.
            formatted_message = self.env['account.move.send']._format_error_html(invoice.l10n_hu_edi_messages)
            invoice.with_context(no_new_invoice=True).message_post(body=formatted_message)

        if self.env['account.move.send']._can_commit():
            self.env.cr.commit()

        # If blocking errors, raise UserError, or log if we are in a cron.
        for invoice in invoices_to_query:
            if invoice.l10n_hu_edi_messages.get('blocking_level') == 'error' or recover_transactions_error:
                if invoice.l10n_hu_edi_messages.get('blocking_level') == 'error':
                    error_text = self.env['account.move.send']._format_error_text(invoice.l10n_hu_edi_messages)
                else:
                    error_text = self.env['account.move.send']._format_error_text(recover_transactions_error)
                if not from_cron:
                    raise UserError(error_text)
                else:
                    _logger.error(error_text)

    def l10n_hu_edi_button_hide_banner(self):
        messages = self.l10n_hu_edi_messages
        if messages:
            messages['hide_banner'] = True
            self.l10n_hu_edi_messages = messages

    # === Helpers === #

    def _l10n_hu_edi_get_valid_actions(self):
        """ If any NAV 3.0 flows are applicable to the given invoice, return them, else None. """
        self.ensure_one()
        valid_actions = []
        if (
            self.country_code == 'HU'
            and self.is_sale_document()
            and self.state == 'posted'
        ):
            if self.l10n_hu_edi_state in [False, 'rejected', 'cancelled']:
                valid_actions.append('upload')
            if self.l10n_hu_edi_transaction_code:
                valid_actions.append('query_status')
            if self.l10n_hu_edi_state in ['confirmed', 'confirmed_warning']:
                valid_actions.append('request_cancel')
            if not valid_actions:
                # Placeholder to denote that the invoice was already processed with a NAV flow
                valid_actions.append(True)
        return valid_actions

    def _l10n_hu_get_chain_base(self):
        """ Get the base invoice of the invoice chain. """
        modification_invoices = self
        base_invoices = self.env['account.move']
        while modification_invoices:
            base_invoices |= modification_invoices.filtered(lambda m: not m.reversed_entry_id and not m.debit_origin_id)
            modification_invoices = modification_invoices.reversed_entry_id | modification_invoices.debit_origin_id
        return base_invoices

    def _l10n_hu_get_chain_invoices(self):
        """ Given base invoices, get all invoices in the chain. """
        chain_invoices = self
        next_invoices = self
        while (next_invoices := next_invoices.reversal_move_ids | next_invoices.debit_note_ids):
            chain_invoices |= next_invoices
        return chain_invoices

    def _l10n_hu_get_currency_rate(self):
        """ Get the invoice currency / HUF rate.

        If the company currency is HUF, we estimate this based on the invoice lines
        (or if this is not an invoice, based on the AMLs), using a MMSE estimator.

        If the company currency is not HUF (e.g. Hungarian companies that do their accounting in euro),
        we get the rate from the currency rates.
        """
        if self.currency_id.name == 'HUF':
            return 1
        if self.company_id.currency_id.name == 'HUF':
            squared_amount_currency = sum(line.amount_currency ** 2 for line in (self.invoice_line_ids or self.line_ids))
            squared_balance = sum(line.balance ** 2 for line in self.invoice_line_ids)
            return math.sqrt(squared_balance / squared_amount_currency)
        return self.env['res.currency']._get_conversion_rate(
            from_currency=self.currency_id,
            to_currency=self.env.ref('base.HUF'),
            company=self.company_id,
            date=self.invoice_date,
        )

    def _l10n_hu_edi_set_chain_index(self):
        """ Set the l10n_hu_invoice_chain_index field. """
        self.ensure_one()
        base_invoice = self._l10n_hu_get_chain_base()
        if base_invoice == self:
            self.l10n_hu_invoice_chain_index = -1  # -1 indicates a base invoice (0 indicates the chain index was not set).
        else:
            # Lock base invoice to prevent concurrent updates, ensuring sequence integrity.
            base_invoice._l10n_hu_edi_acquire_lock()

            chain_indexes_already_sent = base_invoice._l10n_hu_get_chain_invoices().filtered(
                lambda m: m.l10n_hu_edi_state not in ['rejected', 'cancelled']
            ).mapped('l10n_hu_invoice_chain_index')
            for i in range(1, len(chain_indexes_already_sent) + 1):
                if i not in chain_indexes_already_sent:
                    self.l10n_hu_invoice_chain_index = i
                    break

    def _l10n_hu_edi_acquire_lock(self):
        """ Acquire a write lock on the invoices in self. """
        if not self:
            return
        try:
            with self.env.cr.savepoint(flush=False):
                self.env.cr.execute('SELECT * FROM account_move WHERE id = ANY(%s) FOR UPDATE NOWAIT', [self.ids])
        except LockNotAvailable:
            raise UserError(_('Could not acquire lock on invoices - is another user performing operations on them?')) from None

    # === EDI: Flow === #

    def _l10n_hu_edi_check_invoices(self):
        hu_vat_regex = re.compile(r'\d{8}-[1-5]-\d{2}')
        hu_bank_account_regex = re.compile(r'\d{8}-\d{8}-\d{8}|\d{8}-\d{8}|[A-Z]{2}\d{2}[0-9A-Za-z]{11,30}')

        # This contains all the advance invoices that correspond to final invoices in `self`.
        advance_invoices = self.filtered(lambda m: not m._is_downpayment()).invoice_line_ids._get_downpayment_lines().mapped('move_id')

        checks = {
            'company_vat_missing': {
                'records': self.company_id.filtered(lambda c: not c.vat),
                'message': _('Please set company VAT number!'),
                'action_text': _('View Company/ies'),
            },
            'company_vat_invalid': {
                'records': self.company_id.filtered(
                    lambda c: (
                        (c.vat and not hu_vat_regex.fullmatch(c.vat))
                        or (c.l10n_hu_group_vat and not hu_vat_regex.fullmatch(c.l10n_hu_group_vat))
                    )
                ),
                'message': _('Please enter the Hungarian VAT (and/or Group VAT) number in 12345678-1-12 format!'),
                'action_text': _('View Company/ies'),
            },
            'company_address_missing': {
                'records': self.company_id.filtered(lambda c: not c.country_id or not c.zip or not c.city or not c.street),
                'message': _('Please set company Country, Zip, City and Street!'),
                'action_text': _('View Company/ies'),
            },
            'company_not_huf': {
                'records': self.company_id.filtered(lambda c: c.currency_id.name != 'HUF'),
                'message': _('Please use HUF as company currency!'),
                'action_text': _('View Company/ies'),
            },
            'partner_bank_account_invalid': {
                'records': self.partner_bank_id.filtered(lambda p: not hu_bank_account_regex.fullmatch(p.acc_number)),
                'message': _('Please set a valid recipient bank account number!'),
                'action_text': _('View partner(s)'),
            },
            'partner_vat_missing': {
                'records': self.partner_id.commercial_partner_id.filtered(
                    lambda p: p.is_company and not p.vat
                ),
                'message': _('Please set partner Tax ID on company partners!'),
                'action_text': _('View partner(s)'),
            },
            'partner_vat_invalid': {
                'records': self.partner_id.commercial_partner_id.filtered(
                    lambda p: (
                        p.is_company and p.country_code == 'HU'
                        and (
                            (p.vat and not hu_vat_regex.fullmatch(p.vat))
                            or (p.l10n_hu_group_vat and not hu_vat_regex.fullmatch(p.l10n_hu_group_vat))
                        )
                    )
                ),
                'message': _('Please enter the Hungarian VAT (and/or Group VAT) number in 12345678-1-12 format!'),
                'action_text': _('View partner(s)'),
            },
            'partner_address_missing': {
                'records': self.partner_id.commercial_partner_id.filtered(
                    lambda p: p.is_company and (not p.country_id or not p.zip or not p.city or not p.street),
                ),
                'message': _('Please set partner Country, Zip, City and Street!'),
                'action_text': _('View partner(s)'),
            },
            'invoice_date_not_today': {
                'records': self.filtered(lambda m: m.invoice_date != fields.Date.context_today(m)),
                'message': _('Please set invoice date to today!'),
                'action_text': _('View invoice(s)'),
            },
            'invoice_chain_not_confirmed': {
                'records': self.env['account.move'].union(*[
                    move._l10n_hu_get_chain_base()._l10n_hu_get_chain_invoices().filtered(
                        lambda m: (
                            m.id < move.id
                            and m.l10n_hu_edi_state in [False, 'rejected', 'cancelled']
                            and m not in self
                        )
                    )
                    for move in self
                ]),
                'message': _('The following invoices appear to be earlier in the chain, but have not yet been sent. Please send them first.'),
                'action_text': _('View invoice(s)'),
            },
            'invoice_advance_not_paid': {
                'records': advance_invoices.filtered(
                    lambda m: (
                        m.payment_state not in ['in_payment', 'paid', 'partial']
                        or m.l10n_hu_edi_state in [False, 'rejected', 'cancelled']
                            and m not in self  # It's okay to send an advance and a final invoice together, as we sort by id before sending.
                    )
                ),
                'message': _('All advance invoices must be paid and sent to NAV before the final invoice is issued.'),
                'action_text': _('View advance invoice(s)'),
            },
            'invoice_line_not_one_vat_tax': {
                'records': self.filtered(
                    lambda m: any(
                        len(l.tax_ids.filtered(lambda t: t.l10n_hu_tax_type)) != 1
                        for l in m.invoice_line_ids.filtered(lambda l: l.display_type == 'product')
                    )
                ),
                'message': _('Please set exactly one VAT tax on each invoice line!'),
                'action_text': _('View invoice(s)'),
            },
            'invoice_line_non_vat_taxes_misconfigured': {
                'records': self.invoice_line_ids.tax_ids.filtered(
                    lambda t: not t.l10n_hu_tax_type and (not t.price_include or not t.include_base_amount)
                ),
                'message': _("Please set any non-VAT (excise) taxes to be 'Included in Price' and 'Affects subsequent taxes'!"),
                'action_text': _('View tax(es)'),
            },
            'invoice_line_vat_taxes_misconfigured': {
                'records': self.invoice_line_ids.tax_ids.filtered(
                    lambda t: t.l10n_hu_tax_type and not t.is_base_affected
                ),
                'message': _("Please set any VAT taxes to be 'Affected by previous taxes'!"),
                'action_text': _('View tax(es)'),
            },
        }

        errors = {
            f"l10n_hu_edi_{check}": {
                'message': values['message'],
                'action_text': values['action_text'],
                'action': values['records']._get_records_action(name=values['action_text']),
            }
            for check, values in checks.items()
            if values['records']
        }

        if companies_missing_credentials := self.company_id.filtered(lambda c: not c.l10n_hu_edi_server_mode):
            errors['l10n_hu_edi_company_credentials_missing'] = {
                'message': _('Please set NAV credentials in the Accounting Settings!'),
                'action_text': _('Open Accounting Settings'),
                'action': self.env.ref('account.action_account_config').with_company(companies_missing_credentials[0])._get_action_dict(),
            }

        return errors

    def _l10n_hu_edi_upload(self, connection):
        """ Generate invoice XMLs and send to NAV. """
        invoices_sorted = self.sorted(lambda m: m.id)
        for invoice in invoices_sorted:
            # If we come from the 'cancelled' state, this means the previous XML had been confirmed
            # before it was cancelled.
            # In that case, we want to keep it as a regular invoice attachment, for future reference.
            if invoice.l10n_hu_edi_state == 'cancelled':
                self.env['ir.attachment'].search([
                    ('res_model', '=', self._name),
                    ('res_id', '=', invoice.id),
                    ('res_field', '=', 'l10n_hu_edi_attachment'),
                ]).write({
                    'res_field': False,
                    'name': f'{invoice.name.replace("/", "_")}_cancelled_{invoice.l10n_hu_edi_transaction_code}.xml',
                })

            # Set chain index
            invoice._l10n_hu_edi_set_chain_index()

            # Generate XML
            invoice.l10n_hu_edi_attachment = base64.b64encode(invoice._l10n_hu_edi_generate_xml())

            # Set name & mimetype on newly-created attachment.
            attachment = self.env['ir.attachment'].search([
                ('res_model', '=', self._name),
                ('res_id', '=', invoice.id),
                ('res_field', '=', 'l10n_hu_edi_attachment'),
            ])
            attachment.write({
                'name': invoice.l10n_hu_edi_attachment_filename,
                'mimetype': 'application/xml',
            })

        # Batch by company, with max 100 invoices per batch.
        for __, batch_company in groupby(invoices_sorted, lambda m: m.company_id):
            for batch in split_every(100, batch_company):
                self.env['account.move'].union(*batch)._l10n_hu_edi_upload_single_batch(connection)

    def _l10n_hu_edi_upload_single_batch(self, connection):
        try:
            token_result = connection.do_token_exchange(self.company_id.sudo()._l10n_hu_edi_get_credentials_dict())
        except L10nHuEdiConnectionError as e:
            return self.write({
                'l10n_hu_edi_state': 'rejected',
                'l10n_hu_edi_transaction_code': False,
                'l10n_hu_edi_messages': {
                    'error_title': _('Could not authenticate with NAV. Check your credentials and try again.'),
                    'errors': e.errors,
                    'blocking_level': 'error',
                },
            })

        for i, invoice in enumerate(self, start=1):
            invoice.l10n_hu_edi_batch_upload_index = i

        def get_operation_type(invoice):
            operation_type = 'MODIFY'
            base_invoice = invoice._l10n_hu_get_chain_base()
            if invoice == base_invoice:
                operation_type = 'CREATE'
            elif base_invoice.amount_residual == 0:
                operation_type = 'STORNO'
            return operation_type

        invoice_operations = [
            {
                'index': invoice.l10n_hu_edi_batch_upload_index,
                'operation': get_operation_type(invoice),
                'invoice_data': base64.b64decode(invoice.l10n_hu_edi_attachment),
            }
            for invoice in self
        ]

        self.write({'l10n_hu_edi_send_time': fields.Datetime.now()})

        try:
            transaction_code = connection.do_manage_invoice(
                self.company_id.sudo()._l10n_hu_edi_get_credentials_dict(),
                token_result['token'],
                invoice_operations,
            )
        except L10nHuEdiConnectionError as e:
            if e.code == 'timeout':
                return self.write({
                    'l10n_hu_edi_state': 'send_timeout',
                    'l10n_hu_edi_transaction_code': False,
                    'l10n_hu_edi_messages': {
                        'error_title': _('Invoice submission timed out. Please wait at least 6 minutes, then update the status.'),
                        'errors': e.errors,
                        'blocking_level': 'warning',
                    },
                })
            return self.write({
                'l10n_hu_edi_state': 'rejected',
                'l10n_hu_edi_transaction_code': False,
                'l10n_hu_invoice_chain_index': 0,
                'l10n_hu_edi_messages': {
                    'error_title': _('Invoice submission failed.'),
                    'errors': e.errors,
                    'blocking_level': 'error',
                },
            })

        self.write({
            'l10n_hu_edi_state': 'sent',
            'l10n_hu_edi_transaction_code': transaction_code,
            'l10n_hu_edi_messages': {
                'error_title': _('Invoice submitted, waiting for response.'),
                'errors': [],
            }
        })

    def _l10n_hu_edi_query_status(self, connection):
        """ Check the NAV invoice status. """
        # We should update all invoices with the same company and transaction code at once.
        invoices = self | self.search([
            ('company_id', 'in', self.company_id.ids),
            ('l10n_hu_edi_transaction_code', 'in', self.mapped('l10n_hu_edi_transaction_code')),
            ('l10n_hu_edi_state', 'in', ['sent', 'cancel_sent']),
        ])

        # Querying status should be grouped by company and transaction code
        for __, invoices_grouped in groupby(invoices, lambda m: (m.company_id, m.l10n_hu_edi_transaction_code)):
            self.env['account.move'].union(*invoices_grouped)._l10n_hu_edi_query_status_single_batch(connection)

    def _l10n_hu_edi_query_status_single_batch(self, connection):
        """ Check the NAV status for invoices that share the same transaction code (uploaded in a single batch). """
        try:
            results = connection.do_query_transaction_status(
                self.company_id.sudo()._l10n_hu_edi_get_credentials_dict(),
                self[0].l10n_hu_edi_transaction_code,
            )
        except L10nHuEdiConnectionError as e:
            if self.l10n_hu_edi_state == 'sent':
                return self.write({
                    'l10n_hu_edi_messages': {
                        'error_title': _('The invoice was sent to the NAV, but there was an error querying its status.'),
                        'errors': e.errors,
                        'blocking_level': 'warning',
                    },
                })
            else:
                return self.write({
                    'l10n_hu_edi_messages': {
                        'error_title': _('The annulment was sent to the NAV, but there was an error querying its status.'),
                        'errors': e.errors,
                        'blocking_level': 'warning',
                    },
                })

        for processing_result in results['processing_results']:
            invoice = self.filtered(lambda m: str(m.l10n_hu_edi_batch_upload_index) == processing_result['index'])
            if not invoice:
                _logger.error(_('Could not match NAV transaction_code %(code)s, index %(index)s to an invoice in Odoo',
                                code=self[0].l10n_hu_edi_transaction_code,
                                index=processing_result['index']))
                continue

            invoice._l10n_hu_edi_process_query_transaction_result(processing_result, results['annulment_status'])

    def _l10n_hu_edi_process_query_transaction_result(self, processing_result, annulment_status):
        def get_errors_from_processing_result(processing_result):
            return [
                f'({message["validation_result_code"]}) {message["validation_error_code"]}: {message["message"]}'
                for message in processing_result.get('business_validation_messages', []) + processing_result.get('technical_validation_messages', [])
            ]

        self.ensure_one()

        if processing_result['invoice_status'] in ['RECEIVED', 'PROCESSING', 'SAVED']:
            # The invoice/annulment has not been processed yet.
            if self.l10n_hu_edi_state in ['sent', 'send_timeout']:
                self.write({
                    'l10n_hu_edi_state': 'sent',
                    'l10n_hu_edi_messages': {
                        'error_title': _('The invoice was received by the NAV, but has not been confirmed yet.'),
                        'errors': get_errors_from_processing_result(processing_result),
                        'blocking_level': 'warning',
                    },
                })
            elif self.l10n_hu_edi_state in ['cancel_sent', 'cancel_timeout']:
                self.write({
                    'l10n_hu_edi_state': 'cancel_sent',
                    'l10n_hu_edi_messages': {
                        'error_title': _('The annulment request was received by the NAV, but has not been confirmed yet.'),
                        'errors': get_errors_from_processing_result(processing_result),
                        'blocking_level': 'warning',
                    },
                })

        elif processing_result['invoice_status'] == 'DONE':
            if self.l10n_hu_edi_state in ['sent', 'send_timeout']:
                if not processing_result['business_validation_messages'] and not processing_result['technical_validation_messages']:
                    self.write({
                        'l10n_hu_edi_state': 'confirmed',
                        'l10n_hu_edi_messages': {
                            'error_title': _('The invoice was successfully accepted by the NAV.'),
                            'errors': get_errors_from_processing_result(processing_result),
                        },
                    })
                else:
                    self.write({
                        'l10n_hu_edi_state': 'confirmed_warning',
                        'l10n_hu_edi_messages': {
                            'error_title': _(
                                'The invoice was accepted by the NAV, but warnings were reported. '
                                'To reverse, create a credit note / debit note.'
                            ),
                            'errors': get_errors_from_processing_result(processing_result),
                            'blocking_level': 'warning',
                        },
                    })
            elif self.l10n_hu_edi_state in ['cancel_sent', 'cancel_timeout', 'cancel_pending']:
                if annulment_status == 'NOT_VERIFIABLE':
                    self.write({
                        'l10n_hu_edi_state': 'confirmed_warning',
                        'l10n_hu_edi_messages': {
                            'error_title': _('The annulment request was rejected by NAV.'),
                            'errors': get_errors_from_processing_result(processing_result),
                            'blocking_level': 'error',
                        },
                    })
                elif annulment_status == 'VERIFICATION_PENDING':
                    self.write({
                        'l10n_hu_edi_state': 'cancel_pending',
                        'l10n_hu_edi_messages': {
                            'error_title': _('The annulment request is pending, please confirm it on the OnlineSzámla portal.'),
                            'errors': get_errors_from_processing_result(processing_result),
                            'blocking_level': 'warning',
                        }
                    })
                elif annulment_status == 'VERIFICATION_DONE':
                    # Annulling a base invoice will also annul all its modification invoices on NAV.
                    to_cancel = self if self.reversed_entry_id or self.debit_origin_id else self._l10n_hu_get_chain_invoices().filtered(lambda m: m.l10n_hu_edi_state)
                    to_cancel.write({
                        'l10n_hu_edi_state': 'cancelled',
                        'l10n_hu_invoice_chain_index': 0,
                        'l10n_hu_edi_messages': {
                            'error_title': _('The annulment request has been approved by the user on the OnlineSzámla portal.'),
                            'errors': get_errors_from_processing_result(processing_result),
                        }
                    })
                    to_cancel.button_cancel()
                elif annulment_status == 'VERIFICATION_REJECTED':
                    self.write({
                        'l10n_hu_edi_state': 'confirmed_warning',
                        'l10n_hu_edi_messages': {
                            'error_title': _('The annulment request was rejected by the user on the OnlineSzámla portal.'),
                            'errors': get_errors_from_processing_result(processing_result),
                            'blocking_level': 'error',
                        }
                    })

        elif processing_result['invoice_status'] == 'ABORTED':
            if self.l10n_hu_edi_state in ['sent', 'send_timeout']:
                self.write({
                    'l10n_hu_edi_state': 'rejected',
                    'l10n_hu_invoice_chain_index': 0,
                    'l10n_hu_edi_messages': {
                        'error_title': _('The invoice was rejected by the NAV.'),
                        'errors': get_errors_from_processing_result(processing_result),
                        'blocking_level': 'error',
                    },
                })
            elif self.l10n_hu_edi_state in ['cancel_sent', 'cancel_timeout', 'cancel_pending']:
                self.write({
                    'l10n_hu_edi_state': 'confirmed_warning',
                    'l10n_hu_edi_messages': {
                        'error_title': _('The cancellation request could not be performed.'),
                        'errors': get_errors_from_processing_result(processing_result),
                        'blocking_level': 'error',
                    },
                })

    def _l10n_hu_edi_request_cancel(self, connection, code, reason):
        """ Send a cancellation request for all invoices in `self`. """
        # Batch by company, with max 100 annulment requests per batch.
        for __, batch_company in groupby(self, lambda m: m.company_id):
            for batch in split_every(100, batch_company):
                self.env['account.move'].union(*batch)._l10n_hu_edi_request_cancel_single_batch(connection, code, reason)

    def _l10n_hu_edi_request_cancel_single_batch(self, connection, code, reason):
        for i, invoice in enumerate(self, start=1):
            invoice.l10n_hu_edi_batch_upload_index = i

        annulment_operations = [
            {
                'index': invoice.l10n_hu_edi_batch_upload_index,
                'annulmentReference': invoice.name,
                'annulmentCode': code,
                'annulmentReason': reason,
            }
            for invoice in self
        ]

        try:
            token_result = connection.do_token_exchange(self.company_id.sudo()._l10n_hu_edi_get_credentials_dict())
        except L10nHuEdiConnectionError as e:
            return self.write({
                'l10n_hu_edi_messages': {
                    'error_title': _('Could not authenticate with NAV. Check your credentials and try again.'),
                    'errors': e.errors,
                    'blocking_level': 'error',
                },
            })

        self.write({'l10n_hu_edi_send_time': fields.Datetime.now()})

        try:
            transaction_code = connection.do_manage_annulment(
                self.company_id.sudo()._l10n_hu_edi_get_credentials_dict(),
                token_result['token'],
                annulment_operations,
            )
        except L10nHuEdiConnectionError as e:
            if e.code == 'timeout':
                return self.write({
                    'l10n_hu_edi_state': 'cancel_timeout',
                    'l10n_hu_edi_messages': {
                        'error_title': _('Cancellation request timed out. Please wait at least 6 minutes, then update the status.'),
                        'errors': e.errors,
                        'blocking_level': 'warning',
                    },
                })
            return self.write({
                'l10n_hu_edi_messages': {
                    'error_title': _('Cancellation request failed.'),
                    'errors': e.errors,
                    'blocking_level': 'error',
                },
            })

        self.write({
            'l10n_hu_edi_state': 'cancel_sent',
            'l10n_hu_edi_transaction_code': transaction_code,
            'l10n_hu_edi_messages': {
                'error_title': _('Cancellation request submitted, waiting for response.'),
                'errors': [],
            }
        })

    # === EDI: XML generation === #

    def _l10n_hu_edi_generate_xml(self):
        invoice_data = self.env['ir.qweb']._render(
            self._l10n_hu_edi_get_electronic_invoice_template(),
            self._l10n_hu_edi_get_invoice_values(),
        )
        return etree.tostring(cleanup_xml_node(invoice_data, remove_blank_nodes=False), xml_declaration=True, encoding='UTF-8')

    def _l10n_hu_edi_get_electronic_invoice_template(self):
        """ For feature extensibility. """
        return 'l10n_hu_edi.nav_online_invoice_xml_3_0'

    def _l10n_hu_edi_get_invoice_values(self):
        eu_country_codes = set(self.env.ref('base.europe').country_ids.mapped('code'))

        def get_vat_data(partner, force_vat=None):
            if partner.country_code == 'HU' or force_vat:
                return {
                    'tax_number': partner.l10n_hu_group_vat or (force_vat or partner.vat),
                    'group_member_tax_number': partner.l10n_hu_group_vat and (force_vat or partner.vat),
                }
            elif partner.country_code in eu_country_codes:
                return {'community_vat_number': partner.vat}
            else:
                return {'third_state_tax_id': partner.vat}

        def format_bank_account_number(bank_account):
            # Normalize IBANs (no spaces!)
            if bank_account.acc_type == 'iban':
                return normalize_iban(bank_account.acc_number)
            else:
                return bank_account.acc_number

        supplier = self.company_id.partner_id
        customer = self.partner_id.commercial_partner_id

        currency_huf = self.env.ref('base.HUF')
        currency_rate = self._l10n_hu_get_currency_rate()

        base_invoice = self._l10n_hu_get_chain_base()

        invoice_values = {
            'invoice': self,
            'invoiceIssueDate': self.invoice_date,
            'completenessIndicator': False,
            'modifyWithoutMaster': False,
            'base_invoice': base_invoice if base_invoice != self else None,
            'supplier': supplier,
            'supplier_vat_data': get_vat_data(supplier, self.fiscal_position_id.foreign_vat),
            'supplierBankAccountNumber': format_bank_account_number(self.partner_bank_id or supplier.bank_ids[:1]),
            'individualExemption': self.company_id.l10n_hu_tax_regime == 'ie',
            'customer': customer,
            'customerVatStatus': (not customer.is_company and 'PRIVATE_PERSON') or (customer.country_code == 'HU' and 'DOMESTIC') or 'OTHER',
            'customer_vat_data': get_vat_data(customer) if customer.is_company else None,
            'customerBankAccountNumber': format_bank_account_number(customer.bank_ids[:1]),
            'smallBusinessIndicator': self.company_id.l10n_hu_tax_regime == 'sb',
            'exchangeRate': currency_rate,
            'cashAccountingIndicator': self.company_id.l10n_hu_tax_regime == 'ca',
            'shipping_partner': self.partner_shipping_id,
            'sales_partner': self.user_id,
            'mergedItemIndicator': False,
            'format_bool': format_bool,
            'float_repr': float_repr,
            'lines_values': [],
        }

        sign = 1.0 if self.is_inbound() else -1.0

        prev_chain_invoices = base_invoice._l10n_hu_get_chain_invoices().filtered(
            lambda m: m.l10n_hu_invoice_chain_index and m.l10n_hu_invoice_chain_index < self.l10n_hu_invoice_chain_index
        )
        first_line_number = sum(
            len(move.line_ids.filtered(lambda l: l.display_type in ['product', 'rounding']))
            for move in prev_chain_invoices
        ) + 1

        for (line_number, line) in enumerate(
            self.line_ids.filtered(lambda l: l.display_type in ['product', 'rounding']).sorted(lambda l: l.display_type),
            start=first_line_number,
        ):
            line_values = {
                'line': line,
                'lineNumber': line_number - first_line_number + 1,
                'lineNumberReference': base_invoice != self and line_number,
                'lineExpressionIndicator': line.product_id and line.product_uom_id,
                'lineNatureIndicator': {False: 'OTHER', 'service': 'SERVICE'}.get(line.product_id.type, 'PRODUCT'),
                'lineDescription': line.name.replace('\n', ' '),
            }

            if 'is_downpayment' in line and line.is_downpayment:
                # Advance and final invoices.
                line_values['advanceIndicator'] = True

                if not self._is_downpayment():
                    # This is a final invoice that deducts one or more advance invoices.
                    # In this case, we add a reference to the *last-paid* advance invoice (NAV only allows us to report one) if one exists,
                    # otherwise we don't add anything.

                    advance_invoices = line._get_downpayment_lines().mapped('move_id').filtered(lambda m: m.state == 'posted')
                    reconciled_moves = advance_invoices._get_reconciled_amls().move_id
                    last_reconciled_payment = reconciled_moves.filtered(lambda m: m.origin_payment_id or m.statement_line_id).sorted('date', reverse=True)[:1]

                    if last_reconciled_payment:
                        line_values.update({
                            'advanceOriginalInvoice': advance_invoices.filtered(lambda m: last_reconciled_payment in m._get_reconciled_amls().move_id)[0].name,
                            'advancePaymentDate': last_reconciled_payment.date,
                            'advanceExchangeRate': last_reconciled_payment._l10n_hu_get_currency_rate(),
                        })

            if line.display_type == 'product':
                vat_tax = line.tax_ids.filtered(lambda t: t.l10n_hu_tax_type)

                if line.quantity == 0.0 or line.discount == 100.0:
                    price_unit_signed = 0.0
                else:
                    price_unit_signed = sign * line.price_subtotal / (1 - line.discount / 100) / line.quantity

                price_net_signed = self.currency_id.round(price_unit_signed * line.quantity * (1 - line.discount / 100.0))
                discount_value_signed = self.currency_id.round(price_unit_signed * line.quantity - price_net_signed)
                price_total_signed = sign * line.price_total
                vat_amount_signed = self.currency_id.round(price_total_signed - price_net_signed)

                line_values.update({
                    'vat_tax': vat_tax,
                    'vatPercentage': float_round(vat_tax.amount / 100.0, 4),
                    'quantity': line.quantity,
                    'unitPrice': price_unit_signed,
                    'unitPriceHUF': currency_huf.round(price_unit_signed * currency_rate),
                    'discountValue': discount_value_signed,
                    'discountRate': line.discount / 100.0,
                    'lineNetAmount': price_net_signed,
                    'lineNetAmountHUF': currency_huf.round(price_net_signed * currency_rate),
                    'lineVatData': not self.currency_id.is_zero(vat_amount_signed),
                    'lineVatAmount': vat_amount_signed,
                    'lineVatAmountHUF': currency_huf.round(vat_amount_signed * currency_rate),
                    'lineGrossAmountNormal': price_total_signed,
                    'lineGrossAmountNormalHUF': currency_huf.round(price_total_signed * currency_rate),
                })

            elif line.display_type == 'rounding':
                atk_tax = self.env['account.tax'].search(
                    [
                        ('type_tax_use', '=', 'sale'),
                        ('l10n_hu_tax_type', '=', 'ATK'),
                        ('company_id', '=', self.company_id.id),
                    ],
                    limit=1,
                )
                if not atk_tax:
                    raise UserError(_('Please create a sales tax with type ATK (outside the scope of the VAT Act).'))

                amount_huf = line.balance if self.company_id.currency_id == currency_huf else currency_huf.round(line.amount_currency * currency_rate)
                line_values.update({
                    'vat_tax': atk_tax,
                    'vatPercentage': float_round(atk_tax.amount / 100.0, 4),
                    'quantity': 1.0,
                    'unitPrice': -line.amount_currency,
                    'unitPriceHUF': -amount_huf,
                    'lineNetAmount': -line.amount_currency,
                    'lineNetAmountHUF': -amount_huf,
                    'lineVatData': False,
                    'lineGrossAmountNormal': -line.amount_currency,
                    'lineGrossAmountNormalHUF': -amount_huf,
                })
            line_values['lineDescription'] = line_values['lineDescription'] or line.product_id.display_name
            invoice_values['lines_values'].append(line_values)

        is_company_huf = self.company_id.currency_id == currency_huf
        tax_amounts_by_tax = {
            line.tax_line_id: {
                'vatRateVatAmount': -line.amount_currency,
                'vatRateVatAmountHUF': -line.balance if is_company_huf else currency_huf.round(-line.amount_currency * currency_rate),
            }
            for line in self.line_ids.filtered(lambda l: l.tax_line_id.l10n_hu_tax_type)
        }

        invoice_values['tax_summary'] = [
            {
                'vat_tax': vat_tax,
                'vatPercentage': float_round(vat_tax.amount / 100.0, 4),
                'vatRateNetAmount': self.currency_id.round(sum(l['lineNetAmount'] for l in lines_values_by_tax)),
                'vatRateNetAmountHUF': currency_huf.round(sum(l['lineNetAmountHUF'] for l in lines_values_by_tax)),
                'vatRateVatAmount': tax_amounts_by_tax.get(vat_tax, {}).get('vatRateVatAmount', 0.0),
                'vatRateVatAmountHUF': tax_amounts_by_tax.get(vat_tax, {}).get('vatRateVatAmountHUF', 0.0),
            }
            for vat_tax, lines_values_by_tax in groupby(invoice_values['lines_values'], lambda l: l['vat_tax'])
        ]

        total_vat = self.currency_id.round(sum(tax_vals['vatRateVatAmount'] for tax_vals in invoice_values['tax_summary']))
        total_vat_huf = currency_huf.round(sum(tax_vals['vatRateVatAmountHUF'] for tax_vals in invoice_values['tax_summary']))

        total_gross = self.amount_total_in_currency_signed
        total_gross_huf = self.amount_total_signed if is_company_huf else currency_huf.round(self.amount_total_in_currency_signed * currency_rate)

        total_net = self.currency_id.round(total_gross - total_vat)
        total_net_huf = currency_huf.round(total_gross_huf - total_vat_huf)

        invoice_values.update({
            'invoiceNetAmount': total_net,
            'invoiceNetAmountHUF': total_net_huf,
            'invoiceVatAmount': total_vat,
            'invoiceVatAmountHUF': total_vat_huf,
            'invoiceGrossAmount': total_gross,
            'invoiceGrossAmountHUF': total_gross_huf,
        })

        return invoice_values

    # === PDF generation === #

    def _get_name_invoice_report(self):
        self.ensure_one()
        return 'l10n_hu_edi.report_invoice_document' if self.country_code == 'HU' else super()._get_name_invoice_report()

    def _l10n_hu_get_invoice_totals_for_report(self):
        """ In Hungary, tax amounts should appear negative on credit notes.
            We therefore apply a post-processing to the tax totals to make them negative. """

        def invert_dict(dictionary, keys_to_invert):
            """ Replace the values of keys_to_invert by their negative. """
            dictionary.update({
                key: -value
                for key, value in dictionary.items()
                if key in keys_to_invert
            })

        self.ensure_one()
        tax_totals = self.tax_totals
        if not tax_totals:
            return tax_totals

        fields_to_reverse = (
            'base_amount_currency', 'base_amount',
            'display_base_amount_currency', 'display_base_amount',
            'tax_amount_currency', 'tax_amount',
            'total_amount_currency', 'total_amount',
            'cash_rounding_base_amount_currency', 'cash_rounding_base_amount',
        )

        if self.move_type in ('out_refund', 'in_refund'):
            invert_dict(tax_totals, fields_to_reverse)
            for subtotal in tax_totals['subtotals']:
                invert_dict(subtotal, fields_to_reverse)
                for tax_group in subtotal['tax_groups']:
                    invert_dict(tax_group, fields_to_reverse)

        currency_huf = self.env.ref('base.HUF')
        tax_totals['total_vat_amount_in_huf'] = sum(
            line.balance for line in self.line_ids.filtered(lambda l: l.tax_line_id.l10n_hu_tax_type)
        ) * (1 if self.is_purchase_document() else -1)
        tax_totals['formatted_total_vat_amount_in_huf'] = formatLang(
            self.env, tax_totals['total_vat_amount_in_huf'], currency_obj=currency_huf
        )

        return tax_totals

```

## File: models\account_move_send.py

```python
import time
from datetime import timedelta

from odoo import api, fields, models, _
from odoo.addons.l10n_hu_edi.models.l10n_hu_edi_connection import L10nHuEdiConnection


class AccountMoveSend(models.AbstractModel):
    _inherit = 'account.move.send'

    @api.model
    def _is_hu_edi_applicable(self, move):
        return 'upload' in move._origin._l10n_hu_edi_get_valid_actions()

    def _get_all_extra_edis(self) -> dict:
        # EXTENDS 'account'
        res = super()._get_all_extra_edis()
        res.update({'hu_nav_30': {'label': _("NAV 3.0"), 'is_applicable': self._is_hu_edi_applicable}})
        return res

    # -------------------------------------------------------------------------
    # ALERTS
    # -------------------------------------------------------------------------

    def _get_alerts(self, moves, moves_data):
        # EXTENDS 'account'
        alerts = super()._get_alerts(moves, moves_data)
        hu_moves = moves.filtered(lambda m: 'hu_nav_30' in moves_data[m]['extra_edis'])
        enabled_moves = moves.filtered(lambda m: 'upload' in m._l10n_hu_edi_get_valid_actions())._origin
        if hu_moves - enabled_moves:
            alerts['l10n_hu_edi_checkbox_not_ticked'] = {
                'message': _("Invoices issued in Hungary must, with few exceptions, be reported to the NAV's Online-Invoice system.")
            }
        else:
            alerts.update(enabled_moves._l10n_hu_edi_check_invoices())
        return alerts

    # -------------------------------------------------------------------------
    # SENDING METHODS
    # -------------------------------------------------------------------------

    @api.model
    def _l10n_hu_edi_cron_update_status(self):
        final_states = [False, 'confirmed', 'confirmed_warning', 'rejected', 'cancel_pending', 'cancelled']
        invoices_pending = self.env['account.move'].search([('l10n_hu_edi_state', 'not in', final_states)])
        invoices_pending.l10n_hu_edi_button_update_status(from_cron=True)

        if any(m.state not in final_states for m in invoices_pending):
            # Trigger cron again in 10 minutes.
            self.env.ref('l10n_hu_edi.ir_cron_update_status')._trigger(at=fields.Datetime.now() + timedelta(minutes=10))

    def _call_web_service_before_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_before_invoice_pdf_render(invoices_data)

        invoices_hu = self.env['account.move'].browse([
            invoice.id
            for invoice, invoice_data in invoices_data.items()
            if 'hu_nav_30' in invoice_data['extra_edis']
               and 'upload' in invoice._l10n_hu_edi_get_valid_actions()
        ])

        if not invoices_hu:
            return

        # Pre-emptively acquire write lock on all invoices to be processed
        # Otherwise, we will get a serialization error later
        # (bad, because Odoo will try to retry the entire request, leading to duplicate sending to NAV)
        invoices_hu._l10n_hu_edi_acquire_lock()

        # STEP 1: Generate and send the invoice XMLs.
        invoices_to_upload = invoices_hu.filtered(lambda m: 'upload' in m._l10n_hu_edi_get_valid_actions())

        # If we need to re-generate the PDF, break the link between the existing attachment and the 'invoice_pdf_report_file' field.
        # The existing PDF will remain linked to the invoice, but no longer as primary attachment.
        invoices_to_upload.invoice_pdf_report_id.write({'res_field': False})
        invoices_to_upload.invalidate_recordset(fnames=['invoice_pdf_report_id', 'invoice_pdf_report_file'])

        with L10nHuEdiConnection(self.env) as connection:
            invoices_to_upload._l10n_hu_edi_upload(connection)
            if self._can_commit():
                self.env.cr.commit()

            if any(m.l10n_hu_edi_state == 'sent' for m in invoices_hu):
                # If any invoices were just sent, wait so that NAV has enough time to process them
                time.sleep(2)

            # STEP 2: Query status
            invoices_hu.filtered(lambda m: 'query_status' in m._l10n_hu_edi_get_valid_actions())._l10n_hu_edi_query_status(connection)

        # STEP 3: Schedule update status of pending invoices in 10 minutes.
        if any(m.l10n_hu_edi_state not in [False, 'confirmed', 'confirmed_warning', 'rejected'] for m in invoices_hu):
            self.env.ref('l10n_hu_edi.ir_cron_update_status')._trigger(at=fields.Datetime.now() + timedelta(minutes=10))

        # STEP 4: Error / success handling.
        for invoice in invoices_hu:
            # Log outcome in chatter
            formatted_message = self._format_error_html(invoice.l10n_hu_edi_messages)
            invoice.with_context(no_new_invoice=True).message_post(body=formatted_message)

            # Update invoice_data with errors
            blocking_level = invoice.l10n_hu_edi_messages.get('blocking_level')
            if blocking_level == 'error':
                invoices_data[invoice]['error'] = invoice.l10n_hu_edi_messages

        if self._can_commit():
            self.env.cr.commit()

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.translate import LazyTranslate

_lt = LazyTranslate(__name__)


_SELECTION_TAX_TYPE = [
    ('VAT', 'Normal VAT (percent based)'),
    ('AAM', 'AAM - Personal tax exemption'),
    ('TAM', 'TAM - tax-exempt activity or tax-exempt due to being in public interest or special in nature'),
    ('KBAET', 'KBAET - intra-Community exempt supply, without new means of transport'),
    ('KBAUK', 'KBAUK - tax-exempt, intra-Community sales of new means of transport'),
    ('EAM', 'EAM - tax-exempt, extra-Community sales of goods (export of goods to a non-EU country)'),
    ('NAM', 'NAM - tax-exempt on other grounds related to international transactions'),
    ('ATK', 'ATK - Outside the scope of VAT'),
    ('EUFAD37', 'EUFAD37 - Based on section 37 of the VAT Act, a reverse charge transaction carried out in another Member State'),
    ('EUFADE', 'EUFADE - Reverse charge transaction carried out in another Member State, not subject to Section 37 of the VAT Act'),
    ('EUE', 'EUE - Non-reverse charge transaction performed in another Member State'),
    ('HO', 'HO - Transaction in a third country'),
    ('DOMESTIC_REVERSE', 'DOMESTIC_REVERSE - Domestic reverse-charge regime'),
    ('TRAVEL_AGENCY', 'TRAVEL_AGENCY - Profit-margin based regime for travel agencies'),
    ('SECOND_HAND', 'SECOND_HAND - Profit-margin based regime for second-hand sales'),
    ('ARTWORK', 'ARTWORK - Profit-margin based regime for artwork sales'),
    ('ANTIQUES', 'ANTIQUES - Profit-margin based regime for antique sales'),
    ('REFUNDABLE_VAT', 'REFUNDABLE_VAT - VAT incurred under sections 11 or 14, without an agreement from the beneficiary to reimburse VAT'),
    ('NONREFUNDABLE_VAT', 'NONREFUNDABLE_VAT - VAT incurred under sections 11 or 14, with an agreement from the beneficiary to reimburse VAT'),
    ('NO_VAT', 'VAT not applicable pursuant to section 17 of the VAT Act'),
]

_DEFAULT_TAX_REASONS = {
    'AAM': _lt('AAM Tax exempt'),
    'TAM': _lt('TAM Exempt property'),
    'KBAET': _lt('KBAET sale to EU - VAT tv.§ 89.'),
    'KBAUK': _lt('KBAUK New means of transport within the EU - VAT tv.§ 89.§(2)'),
    'EAM': _lt('EAM Product export to 3rd country - VAT tv.98-109.§'),
    'NAM': _lt('NAM other export transaction VAT law § 110-118'),
    'ATK': _lt('ATK Outside the scope of VAT - VAT tv.2-3.§'),
    'EUFAD37': _lt('EUFAD37 § 37 (1) Reverse VAT in another EU country'),
    'EUFADE': _lt('EUFADE Reverse charge of VAT in another EU country not VAT tv. § 37 (1)'),
    'EUE': _lt('EUE Sales made in a 2nd EU country'),
    'HO': _lt('HO Service to 3rd country'),
}


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_hu_tax_type = fields.Selection(
        _SELECTION_TAX_TYPE,
        string='NAV VAT Tax Type',
        help='Precise identification of the VAT tax for the Hungarian authority.',
    )
    l10n_hu_tax_reason = fields.Char(
        string='NAV VAT Tax Exemption Reason',
        help='May be used to provide support for the use of a VAT-exempt VAT tax type.',
        compute='_compute_l10n_hu_tax_reason',
        readonly=False,
    )

    @api.depends('l10n_hu_tax_type')
    def _compute_l10n_hu_tax_reason(self):
        for tax in self:
            reason = _DEFAULT_TAX_REASONS.get(tax.l10n_hu_tax_type, '')
            tax.l10n_hu_tax_reason = self.env._(reason)  # pylint: disable=gettext-variable

```

## File: models\l10n_hu_edi_connection.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode, b64encode
import binascii
from datetime import datetime, timedelta, timezone
import hashlib
import logging
import secrets

from cryptography.hazmat.primitives import padding
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import dateutil.parser
from lxml import etree
import requests

from odoo import _, release
from odoo.tools import cleanup_xml_node


_logger = logging.getLogger(__name__)


XML_NAMESPACES = {
    'api': 'http://schemas.nav.gov.hu/OSA/3.0/api',
    'common': 'http://schemas.nav.gov.hu/NTCA/1.0/common',
    'base': 'http://schemas.nav.gov.hu/OSA/3.0/base',
    'data': 'http://schemas.nav.gov.hu/OSA/3.0/data',
}


def format_bool(value):
    return 'true' if value else 'false'


def format_timestamp(value):
    return value.strftime('%Y-%m-%dT%H:%M:%S.%f')[:-3] + 'Z'


def decrypt_aes128(key, encrypted_token):
    """ Decrypt AES-128-ECB encrypted bytes.
    :param key bytes: the 128-bit key
    :param encrypted_token bytes: the bytes to decrypt
    :return: the decrypted bytes
    """
    decryptor = Cipher(algorithms.AES(key), modes.ECB()).decryptor()
    decrypted_token = decryptor.update(encrypted_token) + decryptor.finalize()
    unpadder = padding.PKCS7(128).unpadder()
    unpadded_token = unpadder.update(decrypted_token) + unpadder.finalize()
    return unpadded_token


class L10nHuEdiConnectionError(Exception):
    def __init__(self, errors, code=None):
        if not isinstance(errors, list):
            errors = [errors]
        self.errors = errors
        self.code = code
        super().__init__('\n'.join(errors))


class L10nHuEdiConnection:
    def __init__(self, env):
        """ Methods to call NAV API endpoints.
        Use this as a context manager (`with L10nHuEdiConnection(...) as connection`)
        to ensure the TCP connection is closed when you are finished calling endpoints.

        :param env: the Odoo environment
        """
        self.env = env
        self.session = requests.Session()

    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.session.close()

    def do_token_exchange(self, credentials):
        """ Request a token for invoice submission.

        :param credentials: a dictionary
            {
                'vat': str,
                'mode': 'production' || 'test',
                'username': str,
                'password': str,
                'signature_key': str,
                'replacement_key': str
            }
        :return: a dictionary {'token': str, 'token_validity_to': datetime}
        :raise: L10nHuEdiConnectionError
        """
        if credentials['mode'] == 'demo':
            return {'token': 'token', 'token_validity_to': datetime.utcnow() + timedelta(minutes=5)}

        template_values = self._get_header_values(credentials)
        request_data = self.env['ir.qweb']._render('l10n_hu_edi.token_exchange_request', template_values)
        request_data = etree.tostring(cleanup_xml_node(request_data, remove_blank_nodes=False), xml_declaration=True, encoding='UTF-8')

        response_xml = self._call_nav_endpoint(credentials['mode'], 'tokenExchange', request_data)
        self._parse_error_response(response_xml)

        encrypted_token = response_xml.findtext('api:encodedExchangeToken', namespaces=XML_NAMESPACES)
        token_validity_to = response_xml.findtext('api:tokenValidityTo', namespaces=XML_NAMESPACES)
        try:
            # Convert into a naive UTC datetime, since Odoo can't store timezone-aware datetimes
            token_validity_to = dateutil.parser.isoparse(token_validity_to).astimezone(timezone.utc).replace(tzinfo=None)
        except ValueError:
            _logger.warning('Could not parse token validity end timestamp!')
            token_validity_to = datetime.utcnow() + timedelta(minutes=5)

        if not encrypted_token:
            raise L10nHuEdiConnectionError(_('Missing token in response from NAV.'))

        try:
            token = decrypt_aes128(credentials['replacement_key'].encode(), b64decode(encrypted_token.encode())).decode()
        except ValueError as e:
            raise L10nHuEdiConnectionError(_('Error during decryption of ExchangeToken.')) from e

        return {'token': token, 'token_validity_to': token_validity_to}

    def do_manage_invoice(self, credentials, token, invoice_operations):
        """ Submit one or more invoices.

        :param token: a token obtained via `do_token_exchange`
        :param invoice_operations: a list of dictionaries:
            {
                'index': <index given to invoice>,
                'operation': 'CREATE' or 'MODIFY' or 'STORNO',
                'invoice_data': <XML data of the invoice as bytes>
            }
        :return str: The transaction code issued by NAV.
        :raise: L10nHuEdiConnectionError, with code='timeout' if a timeout occurred.
        """
        if credentials['mode'] == 'demo':
            return secrets.token_hex(8).upper()

        template_values = {
            'exchangeToken': token,
            'compressedContent': False,
            'invoices': [],
        }
        invoice_hashes = []
        for invoice_operation in invoice_operations:
            invoice_data_b64 = b64encode(invoice_operation['invoice_data']).decode('utf-8')
            template_values['invoices'].append({
                'index': invoice_operation['index'],
                'invoiceOperation': invoice_operation['operation'],
                'invoiceData': invoice_data_b64,
            })
            invoice_hashes.append(self._calculate_invoice_hash(invoice_operation['operation'] + invoice_data_b64))

        template_values.update(self._get_header_values(credentials, invoice_hashs=invoice_hashes))

        request_data = self.env['ir.qweb']._render('l10n_hu_edi.manage_invoice_request', template_values)
        request_data = etree.tostring(cleanup_xml_node(request_data, remove_blank_nodes=False), xml_declaration=True, encoding='UTF-8')

        response_xml = self._call_nav_endpoint(credentials['mode'], 'manageInvoice', request_data, timeout=60)
        self._parse_error_response(response_xml)

        transaction_code = response_xml.findtext('api:transactionId', namespaces=XML_NAMESPACES)
        if not transaction_code:
            raise L10nHuEdiConnectionError(_('Invoice Upload failed: NAV did not return a Transaction ID.'))

        return transaction_code

    def do_query_transaction_status(self, credentials, transaction_code, return_original_request=False):
        """ Query the status of a transaction.

        :param transaction_code: the code of the transaction to query
        :param return_original_request: whether to request the submitted invoice XML.
        :return: a list of dicts {'index': str, 'invoice_status': str, 'business_validation_messages', 'technical_validation_messages'}
        :raise: L10nHuEdiConnectionError
        """
        if credentials['mode'] == 'demo':
            invoices = self.env['account.move'].search([('l10n_hu_edi_transaction_code', '=', transaction_code)])
            if any(m.l10n_hu_edi_state.startswith('cancel') for m in invoices):
                return {
                    'processing_results': [
                        {
                            'index': str(i + 1),
                            'invoice_status': 'DONE',
                            'business_validation_messages': [{
                                'validation_result_code': 'INFO',
                                'validation_error_code': 'INFO_SINGLE_INVOICE_ANNULMENT',
                                'message': 'Egyedi számla technikai érvénytelenítése',
                            }],
                            'technical_validation_messages': [],
                        }
                        for i in range(len(invoices))
                    ],
                    'annulment_status': 'VERIFICATION_DONE',
                }
            else:
                return {
                    'processing_results': [
                        {
                            'index': str(i + 1),
                            'invoice_status': 'DONE',
                            'business_validation_messages': [],
                            'technical_validation_messages': [],
                        }
                        for i in range(len(invoices))
                    ],
                    'annulment_status': None,
                }

        template_values = {
            **self._get_header_values(credentials),
            'transactionId': transaction_code,
            'returnOriginalRequest': return_original_request,
        }
        request_data = self.env['ir.qweb']._render('l10n_hu_edi.query_transaction_status_request', template_values)
        request_data = etree.tostring(cleanup_xml_node(request_data, remove_blank_nodes=False), xml_declaration=True, encoding='UTF-8')

        response_xml = self._call_nav_endpoint(credentials['mode'], 'queryTransactionStatus', request_data)
        self._parse_error_response(response_xml)

        results = {
            'processing_results': [],
            'annulment_status': response_xml.findtext('api:processingResults/api:annulmentData/api:annulmentVerificationStatus', namespaces=XML_NAMESPACES),
        }
        for processing_result_xml in response_xml.iterfind('api:processingResults/api:processingResult', namespaces=XML_NAMESPACES):
            processing_result = {
                'index': processing_result_xml.findtext('api:index', namespaces=XML_NAMESPACES),
                'invoice_status': processing_result_xml.findtext('api:invoiceStatus', namespaces=XML_NAMESPACES),
                'business_validation_messages': [],
                'technical_validation_messages': [],
            }
            for message_xml in processing_result_xml.iterfind('api:businessValidationMessages', namespaces=XML_NAMESPACES):
                processing_result['business_validation_messages'].append({
                    'validation_result_code': message_xml.findtext('api:validationResultCode', namespaces=XML_NAMESPACES),
                    'validation_error_code': message_xml.findtext('api:validationErrorCode', namespaces=XML_NAMESPACES),
                    'message': message_xml.findtext('api:message', namespaces=XML_NAMESPACES),
                })
            for message_xml in processing_result_xml.iterfind('api:technicalValidationMessages', namespaces=XML_NAMESPACES):
                processing_result['technical_validation_messages'].append({
                    'validation_result_code': message_xml.findtext('common:validationResultCode', namespaces=XML_NAMESPACES),
                    'validation_error_code': message_xml.findtext('common:validationErrorCode', namespaces=XML_NAMESPACES),
                    'message': message_xml.findtext('common:message', namespaces=XML_NAMESPACES),
                })
            if return_original_request:
                try:
                    original_file = b64decode(processing_result_xml.findtext('api:originalRequest', namespaces=XML_NAMESPACES))
                    original_xml = etree.fromstring(original_file)
                except binascii.Error as e:
                    raise L10nHuEdiConnectionError(str(e)) from e
                except etree.ParserError as e:
                    raise L10nHuEdiConnectionError(str(e)) from e

                processing_result.update({
                    'original_file': original_file.decode(),
                    'original_xml': original_xml,
                })

            results['processing_results'].append(processing_result)

        return results

    def do_query_transaction_list(self, credentials, datetime_from, datetime_to, page=1):
        """ Query the transactions that were submitted in a given time interval.

        :param datetime_from: start of the time interval to query
        :param datetime_to: end of the time interval to query
        :return: a dict {'transaction_codes': list[str], 'available_pages': int}
        :raise: L10nHuEdiConnectionError
        """
        if credentials['mode'] == 'demo':
            return {"transactions": [], "available_pages": 1}

        template_values = {
            **self._get_header_values(credentials),
            'page': page,
            'dateTimeFrom': format_timestamp(datetime_from),
            'dateTimeTo': format_timestamp(datetime_to),
        }
        request_data = self.env['ir.qweb']._render('l10n_hu_edi.query_transaction_list_request', template_values)
        request_data = etree.tostring(cleanup_xml_node(request_data, remove_blank_nodes=False), xml_declaration=True, encoding='UTF-8')

        response_xml = self._call_nav_endpoint(credentials['mode'], 'queryTransactionList', request_data)
        self._parse_error_response(response_xml)

        available_pages = response_xml.findtext('api:transactionListResult/api:availablePage', namespaces=XML_NAMESPACES)
        try:
            available_pages = int(available_pages)
        except ValueError:
            available_pages = 1

        transactions = []
        for transaction_xml in response_xml.iterfind('api:transactionListResult/api:transaction', namespaces=XML_NAMESPACES):
            try:
                send_time = datetime.fromisoformat(transaction_xml.findtext('api:insDate', namespaces=XML_NAMESPACES).replace('Z', ''))
            except ValueError as e:
                raise L10nHuEdiConnectionError(_('Could not parse time of previous transaction')) from e
            transactions.append({
                'transaction_code': transaction_xml.findtext('api:transactionId', namespaces=XML_NAMESPACES),
                'annulment': transaction_xml.findtext('api:technicalAnnulment', namespaces=XML_NAMESPACES) == 'true',
                'username': transaction_xml.findtext('api:insCusUser', namespaces=XML_NAMESPACES),
                'source': transaction_xml.findtext('api:source', namespaces=XML_NAMESPACES),
                'send_time': send_time,
            })

        return {"transactions": transactions, "available_pages": available_pages}

    def do_manage_annulment(self, credentials, token, annulment_operations):
        """ Request technical annulment of one or more invoices.

        :param token: a token obtained via `do_token_exchange`
        :param annulment_operations: a list of dictionaries:
            {
                'index': <index given to invoice>,
                'annulmentReference': the name of the invoice to annul,
                'annulmentCode': one of ('ERRATIC_DATA', 'ERRATIC_INVOICE_NUMBER', 'ERRATIC_INVOICE_ISSUE_DATE', 'ERRATIC_ELECTRONIC_HASH_VALUE'),
                'annulmentReason': a plain-text explanation of the reason for annulment,
            }
        :return str: The transaction code issued by NAV.
        :raise: L10nHuEdiConnectionError, with code='timeout' if a timeout occurred.
        """
        if credentials['mode'] == 'demo':
            return secrets.token_hex(8).upper()

        template_values = {
            'exchangeToken': token,
            'annulments': []
        }

        annulment_hashes = []
        for annulment_operation in annulment_operations:
            annulment_operation['annulmentTimestamp'] = format_timestamp(datetime.utcnow())
            annulment_data = self.env['ir.qweb']._render('l10n_hu_edi.invoice_annulment', annulment_operation)
            annulment_data_b64 = b64encode(annulment_data.encode()).decode('utf-8')
            template_values['annulments'].append({
                'index': annulment_operation['index'],
                'annulmentOperation': 'ANNUL',
                'invoiceAnnulment': annulment_data_b64,
            })
            annulment_hashes.append(self._calculate_invoice_hash('ANNUL' + annulment_data_b64))

        template_values.update(self._get_header_values(credentials, invoice_hashs=annulment_hashes))

        request_data = self.env['ir.qweb']._render('l10n_hu_edi.manage_annulment_request', template_values)
        request_data = etree.tostring(cleanup_xml_node(request_data, remove_blank_nodes=False), xml_declaration=True, encoding='UTF-8')

        response_xml = self._call_nav_endpoint(credentials['mode'], 'manageAnnulment', request_data, timeout=60)
        self._parse_error_response(response_xml)

        transaction_code = response_xml.findtext('api:transactionId', namespaces=XML_NAMESPACES)
        if not transaction_code:
            raise L10nHuEdiConnectionError(_('Invoice Upload failed: NAV did not return a Transaction ID.'))

        return transaction_code

    # === Helpers: XML generation === #

    def _get_header_values(self, credentials, invoice_hashs=None):
        timestamp = datetime.utcnow()
        request_id = 'ODOO' + secrets.token_hex(13)
        request_signature = self._calculate_request_signature(credentials['signature_key'], request_id, timestamp, invoice_hashs=invoice_hashs)
        odoo_version = release.version
        module_version = self.env['ir.module.module'].get_module_info('l10n_hu_edi').get('version').replace('saas~', '').replace('.', '')

        return {
            'requestId': request_id,
            'timestamp': format_timestamp(timestamp),
            'login': credentials['username'],
            'passwordHash': self._calculate_password_hash(credentials['password']),
            'taxNumber': credentials['vat'][:8],
            'requestSignature': request_signature,
            'softwareId': f'BE477472701-{module_version}'[:18],
            'softwareName': 'Odoo Enterprise',
            'softwareOperation': 'ONLINE_SERVICE',
            'softwareMainVersion': odoo_version,
            'softwareDevName': 'Odoo SA',
            'softwareDevContact': 'andu@odoo.com',
            'softwareDevCountryCode': 'BE',
            'softwareDevTaxNumber': '477472701',
            'format_bool': format_bool,
        }

    def _calculate_password_hash(self, password):
        return hashlib.sha512(password.encode()).hexdigest().upper()

    def _calculate_invoice_hash(self, value):
        return hashlib.sha3_512(value.encode()).hexdigest().upper()

    def _calculate_request_signature(self, key_sign, reqid, reqdate, invoice_hashs=None):
        strings = [reqid, reqdate.strftime('%Y%m%d%H%M%S'), key_sign]

        # merge the invoice CRCs if we got
        if invoice_hashs:
            strings += invoice_hashs

        # return back the uppered hexdigest
        return self._calculate_invoice_hash(''.join(strings))

    # === Helpers: HTTP Post === #

    def _call_nav_endpoint(self, mode, service, data, timeout=20):
        if mode == 'production':
            url = 'https://api.onlineszamla.nav.gov.hu/invoiceService/v3/'
        elif mode == 'test':
            url = 'https://api-test.onlineszamla.nav.gov.hu/invoiceService/v3/'
        else:
            raise L10nHuEdiConnectionError(_('Mode should be Production or Test!'))

        headers = {'content-type': 'application/xml', 'accept': 'application/xml'}
        try:
            response_object = self.session.post(f'{url}{service}', data=data, headers=headers, timeout=timeout)
        except requests.Timeout as e:
            raise L10nHuEdiConnectionError(_('Connection to NAV servers timed out.'), code='timeout') from e
        except requests.RequestException as e:
            raise L10nHuEdiConnectionError(str(e)) from e

        try:
            response_xml = etree.fromstring(response_object.text.encode())
        except etree.ParseError as e:
            raise L10nHuEdiConnectionError(_('Invalid NAV response!')) from e

        return response_xml

    # === Helpers: Response parsing === #

    def _parse_error_response(self, response_xml):
        error_code = response_xml.findtext('common:result/common:errorCode', namespaces=XML_NAMESPACES)
        message = response_xml.findtext('common:result/common:message', namespaces=XML_NAMESPACES)
        if error_code:
            errors = [f'{error_code}: {message}']
            for message_xml in response_xml.iterfind('api:technicalValidationMessages', namespaces=XML_NAMESPACES):
                message = message_xml.findtext('api:message', namespaces=XML_NAMESPACES)
                error_code = message_xml.findtext('api:validationErrorCode', namespaces=XML_NAMESPACES)
                errors.append(f'{error_code}: {message}')

            raise L10nHuEdiConnectionError(errors)

```

## File: models\product.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    l10n_hu_product_code_type = fields.Selection(
        selection=[
            ('VTSZ', 'VTSZ - Customs Code'),
            ('TESZOR', 'TESZOR - CPA 2.1 Code'),
            ('KN', 'KN - Combined Nomenclature Code'),
            ('AHK', 'AHK - e-TKO Excise Duty Code'),
            ('KT', 'KT - Environmental Product Code'),
            ('CSK', 'CSK - Packaging Catalogue Code'),
            ('EJ', 'EJ - Building Registry Number'),
            ('OTHER', 'Other'),
        ],
        string='Product Code Type',
        help='If your product has a code in a standard nomenclature, you can indicate which nomenclature here.',
    )
    l10n_hu_product_code = fields.Char(
        string='Product Code Value',
        help='If your product has a code in a standard nomenclature, you can indicate its code here.',
    )

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
from datetime import timedelta
from itertools import islice

from lxml import etree

from odoo import models, fields, _
from odoo.exceptions import UserError
from odoo.addons.l10n_hu_edi.models.l10n_hu_edi_connection import L10nHuEdiConnection, L10nHuEdiConnectionError, XML_NAMESPACES


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_hu_group_vat = fields.Char(
        related='partner_id.l10n_hu_group_vat',
        readonly=False,
    )
    l10n_hu_tax_regime = fields.Selection(
        selection=[
            ('ie', 'Individual Exemption'),
            ('ca', 'Cash Accounting'),
            ('sb', 'Small Business'),
        ],
        string='NAV Tax Regime',
    )
    l10n_hu_edi_server_mode = fields.Selection(
        selection=[
            ('production', 'Production'),
            ('test', 'Test'),
            ('demo', 'Demo'),
        ],
        string='Server Mode',
        help="""
            - Production: Sends invoices to the NAV's production system.
            - Test: Sends invoices to the NAV's test system.
            - Demo: Mocks the NAV system (does not require credentials).
        """
    )
    l10n_hu_edi_username = fields.Char(
        string='NAV Username',
        groups='base.group_system',
    )
    l10n_hu_edi_password = fields.Char(
        string='NAV Password',
        groups='base.group_system',
    )
    l10n_hu_edi_signature_key = fields.Char(
        string='NAV Signature Key',
        groups='base.group_system',
    )
    l10n_hu_edi_replacement_key = fields.Char(
        string='NAV Replacement Key',
        groups='base.group_system',
    )
    l10n_hu_edi_last_transaction_recovery = fields.Datetime(
        string='Last transaction recovery (in production mode)',
        default=lambda self: fields.Datetime.now(),
    )

    def _l10n_hu_edi_configure_company(self):
        """ Single-time configuration for companies, to be applied when l10n_hu_edi is installed
        or a new company is created.
        """
        for company in self:
            # Set profit/loss accounts on cash rounding method
            profit_account = self.env['account.chart.template'].with_company(company).ref('l10n_hu_969', raise_if_not_found=False)
            loss_account = self.env['account.chart.template'].with_company(company).ref('l10n_hu_869', raise_if_not_found=False)
            rounding_method = self.env.ref('l10n_hu_edi.cash_rounding_1_huf', raise_if_not_found=False)
            if profit_account and loss_account and rounding_method:
                rounding_method.with_company(company).write({
                    'profit_account_id': profit_account.id,
                    'loss_account_id': loss_account.id,
                })

            # Activate cash rounding on the company
            res_config_id = self.env['res.config.settings'].create({
                'company_id': company.id,
                'group_cash_rounding': True,
            })
            res_config_id.execute()

    def _l10n_hu_edi_get_credentials_dict(self):
        self.ensure_one()
        credentials_dict = {
            'vat': self.vat,
            'mode': self.l10n_hu_edi_server_mode,
            'username': self.l10n_hu_edi_username,
            'password': self.l10n_hu_edi_password,
            'signature_key': self.l10n_hu_edi_signature_key,
            'replacement_key': self.l10n_hu_edi_replacement_key,
        }
        if self.l10n_hu_edi_server_mode != 'demo' and not all(credentials_dict.values()):
            raise UserError(_('Missing NAV credentials for company %s', self.name))
        return credentials_dict

    def _l10n_hu_edi_test_credentials(self):
        with L10nHuEdiConnection(self.env) as connection:
            for company in self:
                if not company.vat:
                    raise UserError(_('NAV Credentials: Please set the hungarian vat number on the company first!'))
                try:
                    connection.do_token_exchange(company._l10n_hu_edi_get_credentials_dict())
                except L10nHuEdiConnectionError as e:
                    raise UserError(
                        _('Incorrect NAV Credentials! Check that your company VAT number is set correctly. \nError details: %s', e)
                    ) from e

    def _l10n_hu_edi_recover_transactions(self, connection):
        """ Recover transactions that are in force but for some reason are not matched to the company's
        invoices, and update the invoice state correspondingly.

        This can happen, for example, if the invoice sending timed out: in that case, we don't have a
        transaction ID for the invoice. It can also happen if for some reason the transaction ID was
        overwritten by a new request, but the new request fails with a 'duplicate invoice' error.

        To do this, we request a list of all transactions made since l10n_hu_edi_last_transaction_recovery,
        and then we query the last 10 transactions whose transaction IDs are unknown by Odoo. We try to
        match them to invoices in Odoo, and if successful, update the invoice state.
        """

        for company in self:
            # We use the l10n_hu_edi_last_transaction_recovery time only in production mode
            # to indicate which transactions to request.
            # In test mode (where we expect far fewer invoices), we just take the last 24 hours.
            recovery_end_time = fields.Datetime.now()
            if company.l10n_hu_edi_server_mode == 'production':
                recovery_start_time = company.l10n_hu_edi_last_transaction_recovery
            else:
                recovery_start_time = recovery_end_time - timedelta(hours=24)

            # Old invoices are already up-to-date - no need to re-check them.
            invoices_to_check = self.env['account.move'].search([
                ('company_id', '=', company.id),
                ('l10n_hu_edi_send_time', '>=', recovery_start_time),
                ('l10n_hu_edi_state', '!=', False),
            ])
            # Step 1: Request a list of all transactions made during the specified time interval.
            page = 1
            available_pages = 1
            transactions = []
            while page <= available_pages:
                try:
                    transaction_list = connection.do_query_transaction_list(
                        company.sudo()._l10n_hu_edi_get_credentials_dict(),
                        recovery_start_time,
                        recovery_end_time,
                        page,
                    )
                except L10nHuEdiConnectionError as e:
                    return {
                        'error_title': _('Error listing transactions while attempting transaction recovery.'),
                        'errors': e.errors,
                    }

                available_pages = transaction_list['available_pages']
                transactions += transaction_list['transactions']
                page += 1

            # Step 2: Query unknown transactions in reverse order (latest first) and update invoice states accordingly.
            # If there are too many, we should only query the last 10, to avoid pointlessly making huge numbers of requests.
            transactions_to_query = (
                t for t in reversed(transactions)
                if t['username'] == company.sudo().l10n_hu_edi_username
                    and t['source'] == 'MGM'
                    and t['transaction_code'] not in invoices_to_check.mapped('l10n_hu_edi_transaction_code')
            )

            for transaction in islice(transactions_to_query, 10):
                try:
                    results = connection.do_query_transaction_status(
                        company.sudo()._l10n_hu_edi_get_credentials_dict(),
                        transaction['transaction_code'],
                        return_original_request=True,
                    )
                except L10nHuEdiConnectionError as e:
                    return {
                        'error_title': _('Error querying transaction while attempting transaction recovery.'),
                        'errors': e.errors,
                    }

                for processing_result in results['processing_results']:
                    invoice_name = processing_result['original_xml'].findtext('data:invoiceNumber', namespaces=XML_NAMESPACES)
                    canonicalized_attachment = etree.canonicalize(processing_result['original_file'])
                    annulment_invoice_name = processing_result['original_xml'].findtext('data:annulmentReference', namespaces=XML_NAMESPACES)

                    matched_invoice = invoices_to_check.filtered(
                        lambda m: (
                            # 1. Match invoice if the entire XML matches.
                            # For performance, we first check the invoice name before trying to match the whole XML.
                            (
                                m.name == invoice_name
                                and etree.canonicalize(base64.b64decode(m.l10n_hu_edi_attachment).decode())
                                    == canonicalized_attachment
                            )
                            or m.name == annulment_invoice_name
                        ) and (
                            # 2. We update the invoice state only if:
                            # - the invoice doesn't have a transaction code, or
                            # - it currently has a duplicate error, or
                            # - the current transaction is more recent than the latest transaction on the invoice
                            #   and is not a duplicate error (this avoid overwriting the state with a previous, obsolete one).
                            not m.l10n_hu_edi_transaction_code
                            or any(
                                'INVOICE_NUMBER_NOT_UNIQUE' in error or 'ANNULMENT_IN_PROGRESS' in error
                                for error in m.l10n_hu_edi_messages['errors']
                            )
                            or (
                                transaction['send_time'] >= m.l10n_hu_edi_send_time
                                and not (
                                    processing_result['technical_validation_messages']
                                    or any(
                                        message['validation_error_code'] in ['INVOICE_NUMBER_NOT_UNIQUE', 'ANNULMENT_IN_PROGRESS']
                                        for message in processing_result['business_validation_messages']
                                    )
                                )
                            )
                        )
                    )

                    if matched_invoice:
                        # Set the correct transaction code on the matched invoice
                        matched_invoice.l10n_hu_edi_transaction_code = transaction['transaction_code']
                        matched_invoice._l10n_hu_edi_process_query_transaction_result(processing_result, results['annulment_status'])

            # The server might still be processing transactions from the last 6 minutes,
            # so we should keep open the possibility of re-querying them.
            recovery_close_time = recovery_end_time - timedelta(minutes=6)
            if company.l10n_hu_edi_server_mode == 'production':
                company.l10n_hu_edi_last_transaction_recovery = recovery_close_time

            # Any invoices still in a 'timeout' state that are more than 6 minutes old and could not be matched should be considered not received.
            invoices_to_check.filtered(
                lambda m: m.l10n_hu_edi_state == 'send_timeout' and m.l10n_hu_edi_send_time < recovery_close_time
            ).write({
                'l10n_hu_invoice_chain_index': 0,
                'l10n_hu_edi_state': 'rejected',
            })

            invoices_to_check.filtered(
                lambda m: m.l10n_hu_edi_state == 'cancel_timeout' and m.l10n_hu_edi_send_time < recovery_close_time
            ).write({
                'l10n_hu_edi_state': 'confirmed_warning',
            })

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_hu_tax_regime = fields.Selection(
        related='company_id.l10n_hu_tax_regime',
        readonly=False,
    )
    l10n_hu_edi_server_mode = fields.Selection(
        related='company_id.l10n_hu_edi_server_mode',
        readonly=False,
    )
    l10n_hu_edi_username = fields.Char(
        related='company_id.l10n_hu_edi_username',
        readonly=False,
    )
    l10n_hu_edi_password = fields.Char(
        related='company_id.l10n_hu_edi_password',
        readonly=False,
    )
    l10n_hu_edi_signature_key = fields.Char(
        related='company_id.l10n_hu_edi_signature_key',
        readonly=False,
    )
    l10n_hu_edi_replacement_key = fields.Char(
        related='company_id.l10n_hu_edi_replacement_key',
        readonly=False,
    )
    # Technical field to control display of the "Authentication with NAV 3.0 successful" banner
    l10n_hu_edi_is_active = fields.Boolean(
        compute='_compute_l10n_hu_edi_is_active',
    )

    @api.depends('company_id.l10n_hu_edi_server_mode')
    def _compute_l10n_hu_edi_is_active(self):
        for record in self:
            record.l10n_hu_edi_is_active = record.company_id.l10n_hu_edi_server_mode in ['production', 'test']

    @api.model_create_multi
    def create(self, vals_list):
        records = super().create(vals_list)
        for record in records:
            if record.company_id.l10n_hu_edi_server_mode in ['production', 'test']:
                record.company_id._l10n_hu_edi_test_credentials()
        return records

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_hu_group_vat = fields.Char(
        string='Group Tax ID',
        size=13,
        help="If this company belongs to a VAT group, indicate the group's VAT number here.",
        index=True,
    )

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + [
            'l10n_hu_group_vat',
        ]

    @api.model
    def _run_vies_test(self, vat_number, default_country):
        """Convert back the hungarian format to EU format: 12345678-1-12 => HU12345678"""
        if default_country and default_country.code == 'HU' and not vat_number.startswith('HU'):
            vat_number = f'HU{vat_number[:8]}'
        return super()._run_vies_test(vat_number, default_country)

```

## File: models\template_hu.py

```python
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    def _load(self, template_code, company, install_demo, force_create=True):
        res = super()._load(template_code, company, install_demo, force_create)
        if template_code == 'hu':
            company._l10n_hu_edi_configure_company()
        return res

    @template('hu', 'account.tax')
    def _get_hu_account_tax(self):
        data = self._parse_csv('hu', 'account.tax', module='l10n_hu_edi')
        return data

```

## File: models\uom_uom.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProductUoM(models.Model):
    _inherit = 'uom.uom'

    l10n_hu_edi_code = fields.Selection(
        selection=[
            ('PIECE', 'Piece'),
            ('KILOGRAM', 'Kilogram'),
            ('TON', 'Ton'),
            ('KWH', 'Kilowatt hour'),
            ('DAY', 'Day'),
            ('HOUR', 'Hour'),
            ('MINUTE', 'Minute'),
            ('MONTH', 'Month'),
            ('LITER', 'Liter'),
            ('KILOMETER', 'Kilometer'),
            ('CUBIC_METER', 'Cubic meter'),
            ('METER', 'Meter'),
            ('LINEAR_METER', 'Linear meter'),
            ('CARTON', 'Carton'),
            ('PACK', 'Package'),
        ],
        string='NAV UoM code',
        help='Choose the corresponding code, or leave blank if none correspond.',
    )

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import account_move_send
from . import account_tax
from . import l10n_hu_edi_connection
from . import product
from . import res_partner
from . import res_company
from . import res_config_settings
from . import template_hu
from . import uom_uom

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_hu_edi_cancellation_user,l10n_hu_edi.cancellation group_account_invoice,model_l10n_hu_edi_cancellation,account.group_account_invoice,1,1,1,1
l10n_hu_edi_tax_audit_export_user,l10n_hu_edi.tax_audit_export group_account_manager,model_l10n_hu_edi_tax_audit_export,account.group_account_manager,1,1,1,1

```

## File: static\img\logo-mol-colorful.88751645.svg

```svg
<svg width="98" height="60" viewBox="0 0 98 60" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M87.1214 0H13.2897C3.96582 0 0 6.09244 0 16.3866C0 31.1765 12.91 60 29.2373 60C32.3594 60 36.6627 58.1092 40.1644 56.1765C51.8509 49.8319 97.5 24.1176 97.5 8.78151C97.5 4.53782 94.6733 0 87.1214 0Z" fill="white"/>
<path d="M87.1214 0H13.2897C3.96582 0 0 6.09244 0 16.3866C0 31.1765 12.91 60 29.2373 60C32.3594 60 36.6627 58.1092 40.1644 56.1765C51.8509 49.8319 97.5 24.1176 97.5 8.78151C97.5 4.53782 94.6733 0 87.1214 0ZM29.2373 55.7983C14.0913 55.7983 1.51882 29.4538 1.51882 15.7983C1.51882 8.9916 3.37516 4.7479 7.21441 2.77311C5.27369 4.07563 4.17676 5.96639 3.58611 7.68908C2.70013 10.2941 2.36261 13.3613 2.70013 16.4286C4.72523 34.5798 17.6352 45.4622 25.145 50.3782C28.5623 52.605 31.6421 53.9496 34.4267 54.4958C32.275 55.2941 30.4608 55.7983 29.2373 55.7983Z" fill="#A0CF67"/>
<path d="M27.0435 47.9832C19.0697 43.3193 6.24406 32.1428 4.4721 15.9244C3.71269 8.94956 5.90654 3.36133 13.2897 3.36133H87.1214C93.2389 3.36133 94.1248 6.76469 94.1248 8.7815C94.1248 11.2185 91.8888 18.4454 71.2581 33.1092C63.7484 38.4454 52.8635 45.084 43.6662 49.5378C39.1097 51.7647 34.5954 52.3529 27.0435 47.9832Z" fill="#EF4138"/>
<path d="M25.6924 18.3619L25.6502 8.99219H19.6593L13.373 28.614H19.2374L21.5578 19.7905L21.9375 28.614H25.5236L29.2363 19.7905L28.2238 28.614H34.1303L35.2694 8.99219H29.2785L25.6924 18.3619Z" fill="white"/>
<path d="M50.3313 9.66389C46.4498 7.35297 40.8808 9.53784 37.8853 14.5378C34.8899 19.5799 35.6493 25.5042 39.5307 27.8572C43.4122 30.1681 48.9812 27.9832 51.9766 22.9832C54.9299 17.9412 54.2127 12.0168 50.3313 9.66389ZM48.0952 18.7815C47.6733 20.9244 45.9435 22.6891 44.2138 22.6891C42.484 22.6891 41.4293 20.9664 41.8512 18.7815C42.2731 16.6387 44.0028 14.874 45.7326 14.874C47.4624 14.916 48.5171 16.6387 48.0952 18.7815Z" fill="white"/>
<path d="M60.3295 22.6477L62.9452 8.99219H56.9121L53.1572 28.614H59.8232L64.3797 22.6477H60.3295Z" fill="white"/>
</svg>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_invoice_tree_inherit_l10n_hu_edi" model="ir.ui.view">
        <field name="name">account.invoice.list.inherit.l10n_hu_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_invoice_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//list" position="inside">
                <field name="l10n_hu_edi_state" optional="hide"/>
            </xpath>
        </field>
    </record>

    <record id="view_move_form_inherit_l10n_hu_edi" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n_hu_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button string="Update Status"
                        name="l10n_hu_edi_button_update_status"
                        type="object"
                        groups="account.group_account_invoice"
                        invisible="not l10n_hu_edi_state"/>
            </xpath>

            <xpath expr="//sheet" position="before">
                <div class="alert alert-warning mb-0" role="alert"
                     invisible="not l10n_hu_edi_messages or not l10n_hu_edi_messages.get('blocking_level') or l10n_hu_edi_messages.get('hide_banner')">
                    <field name="l10n_hu_edi_messages" invisible="1"/>
                    <field name="l10n_hu_edi_message_html" readonly="1"/>
                    <button name="l10n_hu_edi_button_hide_banner"
                            type="object"
                            class="btn btn-link p-0"
                            string="Hide this message"/>
                </div>
            </xpath>

            <xpath expr="//label[@for='journal_id']" position="before">
                <field name="l10n_hu_payment_mode" invisible="country_code != 'HU'" readonly="state != 'draft'"/>
            </xpath>

            <!-- Documents tab -->
            <xpath expr="//page[@id='other_tab_entry']" position="after">
                <page id="l10n_hu_edi"
                      string="NAV 3.0"
                      invisible="not l10n_hu_edi_state">
                    <group>
                        <group>
                            <field name="l10n_hu_edi_state" readonly="1"/>
                            <field name="l10n_hu_invoice_chain_index" readonly="1"/>
                        </group>
                        <group>
                            <field name="l10n_hu_edi_send_time" readonly="1"/>
                            <field name="l10n_hu_edi_transaction_code" readonly="l10n_hu_edi_state == 'rejected'"/>
                        </group>
                    </group>
                    <group>
                        <field name="l10n_hu_edi_message_html" readonly="1"/>
                        <field name="l10n_hu_edi_attachment" widget="binary" filename="l10n_hu_edi_attachment_filename" readonly="1"/>
                        <field name="l10n_hu_edi_attachment_filename" invisible="1"/>
                    </group>
                </page>
            </xpath>
        </field>
    </record>
</odoo>
```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_tax_form_l10n_hu_edi" model="ir.ui.view">
        <field name="name">account.tax.form.l10n_hu_edi</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="l10n_hu_tax_type" invisible="country_code != 'HU'"/>
                <field name="l10n_hu_tax_reason"
                       invisible="country_code != 'HU' or l10n_hu_tax_type == 'VAT'"
                       required="country_code == 'HU' and l10n_hu_tax_type not in [False, 'VAT']"/>
            </field>
        </field>
    </record>
</odoo>
```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_template_form_view_l10n_hu_edi" model="ir.ui.view">
        <field name="name">product.template.common.form.l10n_hu_edi</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='accounting']" position="inside">
                <group name="l10n_hu_edi" string="Hungary" invisible="'HU' not in fiscal_country_codes">
                    <label for="l10n_hu_product_code_type" string="Product Code"/>
                    <div class="d-flex">
                        <field name="l10n_hu_product_code_type"/>
                        <span class="oe_inline o_form_label mx-3"> : </span>
                        <field name="l10n_hu_product_code" required="l10n_hu_product_code_type"/>
                    </div>
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-if="o._get_name_invoice_report() == 'l10n_hu_edi.report_invoice_document'" t-call="l10n_hu_edi.report_invoice_document" t-lang="lang"/>
        </xpath>
    </template>

    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
        <xpath expr="//t[@t-set='forced_vat']" position="replace"/>
        <xpath expr="//t[@t-set='o']" position="after">
            <t t-set="sign" t-value="-1 if 'refund' in o.move_type else 1"/>
            <t t-set="custom_header" t-value="'l10n_hu_edi.custom_header'"/>
            <t t-set="information_block">
            <div class="col-6">
                <div><strong>Supplier:</strong></div>
                <div class="float-start company_address">
                    <ul class="list-unstyled">
                        <li><t t-out="o.company_id.partner_id" t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": true}'/></li>
                        <li>
                            <t t-if="not o.fiscal_position_id.foreign_vat and o.company_id.partner_id.l10n_hu_group_vat">Group Tax ID</t>
                            <t t-else="else" t-out="o.company_id.account_fiscal_country_id.vat_label or 'Tax ID'"/>:
                            <span t-if="o.fiscal_position_id.foreign_vat" t-out="o.fiscal_position_id.foreign_vat"/>
                            <span t-else="else" t-out="o.company_id.vat"/>
                        </li>
                        <li t-if="not o.fiscal_position_id.foreign_vat and o.company_id.partner_id.l10n_hu_group_vat">Group Member Tax ID: <span t-field="o.company_id.partner_id.l10n_hu_group_vat"/></li>
                        <li t-if="o.partner_id.commercial_partner_id.country_id and o.partner_id.commercial_partner_id.country_id.code!='HU'">
                            EU Tax ID: <span t-out="'HU%s' % o.company_id.vat[:8]"/>
                        </li>
                        <li t-if="o.move_type in ['out_invoice', 'out_receipt'] and o.partner_bank_id">Bank Account: <span t-field="o.partner_bank_id.acc_number"/></li>
                        <li t-if="o.company_id.l10n_hu_tax_regime=='ie'">The issuer of the invoice is <u>Exempt from VAT</u>.</li>
                        <li t-if="o.company_id.l10n_hu_tax_regime=='ca'">The issuer of the invoice is <u>Cash accounting</u>.</li>
                        <li t-if="o.company_id.l10n_hu_tax_regime=='sb'">The issuer of the invoice is <u>Small taxpayer</u>.</li>
                    </ul>
                </div>
            </div>
            </t>
        </xpath>
        <xpath expr="//div[@name='invoice_date']/t[1]" position="attributes">
            <attribute name="t-if">o.move_type == 'out_invoice' or (o.move_type == 'out_refund' and not o.reversed_entry_id)</attribute>
        </xpath>
        <xpath expr="//div[@name='address_not_same_as_shipping']//address" position="before">
            <strong>Customer:</strong>
        </xpath>
        <xpath expr="//div[@name='shipping_address_block']/../.." position="replace"/>
        <xpath expr="//div[@name='address_not_same_as_shipping']//span[@t-field='o.partner_id.vat']/.." position="after">
            <div t-if="o.partner_id.l10n_hu_group_vat">Group Member Tax ID: <span t-field="o.partner_id.l10n_hu_group_vat"/></div>
            <div groups="account.group_delivery_invoice_address" name="shipping_address_block">
                <div/>
                <strong>Shipping Address:</strong>
                <div t-field="o.partner_shipping_id" t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
            </div>
        </xpath>
        <xpath expr="//div[@name='address_same_as_shipping']//address" position="before">
            <strong>Customer:</strong>
        </xpath>
        <xpath expr="//div[@name='address_same_as_shipping']//span[@t-field='o.partner_id.vat']/.." position="after">
            <div t-if="o.partner_id.l10n_hu_group_vat">Group Member Tax ID: <span t-field="o.partner_id.l10n_hu_group_vat"/></div>
            <div t-if="o.move_type == 'out_refund' and o.partner_bank_id">Bank Account: <span t-field="o.partner_bank_id.acc_number"/></div>
        </xpath>
        <xpath expr="//div[@name='no_shipping']//address" position="before">
            <strong>Customer:</strong>
        </xpath>
        <xpath expr="//div[@name='no_shipping']//span[@t-field='o.partner_id.vat']/.." position="after">
            <div t-if="o.partner_id.l10n_hu_group_vat">Group Member Tax ID: <span t-field="o.partner_id.l10n_hu_group_vat"/></div>
        </xpath>
        <xpath expr="//div[@name='due_date']" position="after">
            <div class="col" t-if="o.l10n_hu_payment_mode" name="l10n_hu_payment_mode">
                <strong>Payment Mode</strong>
                <div t-field="o.l10n_hu_payment_mode"/>
            </div>
        </xpath>
        <xpath expr="//table[@name='invoice_line_table']//span[@t-field='line.quantity']" position="attributes">
            <attribute name="t-field"/>
            <attribute name="t-out">line.quantity * sign if line.quantity != 0 else line.quantity</attribute>
        </xpath>
        <xpath expr="//table[@name='invoice_line_table']//span[@t-field='line.price_subtotal']" position="attributes">
            <attribute name="t-field"/>
            <attribute name="t-out">line.price_subtotal * sign</attribute>
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </xpath>
        <xpath expr="//table[@name='invoice_line_table']//span[@t-out='current_subtotal']" position="attributes">
            <attribute name="t-out">current_subtotal * sign</attribute>
        </xpath>
        <xpath expr="//div[hasclass('clearfix')]//span[@t-field='o.amount_residual']" position="attributes">
            <attribute name="t-field"/>
            <attribute name="t-out">o.amount_residual * sign</attribute>
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </xpath>
        <xpath expr="//div[hasclass('clearfix')]//t[@t-call='account.document_tax_totals']" position="attributes">
            <attribute name="t-call">l10n_hu_edi.document_tax_totals</attribute>
        </xpath>
        <xpath expr="//div[hasclass('clearfix')]//t[@t-set='tax_totals']" position="attributes">
            <attribute name="t-value">o._l10n_hu_get_invoice_totals_for_report()</attribute>
        </xpath>
        <xpath expr="//td[@name='account_invoice_line_name']/span" position="after">
            <div t-if="line.product_id.l10n_hu_product_code_type and line.product_id.l10n_hu_product_code">
                <span t-if="line.product_id.l10n_hu_product_code_type == 'OTHER'">Other Product Code</span>
                <span t-else="else" t-out="line.product_id.l10n_hu_product_code_type"/>:
                <span t-out="line.product_id.l10n_hu_product_code"/>
            </div>
        </xpath>
    </template>

    <template id="document_tax_totals" inherit_id="account.document_tax_totals" primary="True">
        <xpath expr="//tr[hasclass('o_total')]" position="after">
            <tr t-if="o.currency_id.name != 'HUF'">
                <td><strong>Total VAT amount in HUF</strong></td>
                <td class="text-end text-nowrap">
                    <span t-out="tax_totals['formatted_total_vat_amount_in_huf']"/>
                </td>
            </tr>
        </xpath>
    </template>

    <template id="custom_header">
        <div class="row mb8">
            <div class="col-6">
                <img t-if="company.logo" class="o_company_logo_small" t-att-src="image_data_uri(company.logo)" alt="Logo"/>
            </div>
            <div class="col-6 text-end mb4">
                <div class="mt0 h4" t-field="company.report_header"/>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="external_layout_standard" inherit_id="web.external_layout_standard" >
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>

        <xpath expr="//img/../.." position="after">
            <div t-attf-class="header o_company_#{company.id}_layout"
                t-if="custom_header and company.account_fiscal_country_id.code == 'HU'">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content d-flex border-top pt-2" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>

        <div class="o_footer_content d-flex border-top pt-2" position="after">
            <div class="o_footer_content border-top pt-2"
                 t-if="custom_footer and company.account_fiscal_country_id.code == 'HU'">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_bold" inherit_id="web.external_layout_bold" >
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>

        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company.account_fiscal_country_id.code == 'HU'">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content row border-top pt-2" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div class="o_footer_content row border-top pt-2" position="after">
            <div class="o_footer_content row border-top pt-2"
                 t-if="custom_footer and company.account_fiscal_country_id.code == 'HU'">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_boxed" inherit_id="web.external_layout_boxed" >
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>

        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company.account_fiscal_country_id.code == 'HU'">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content row border-top pt-2" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div class="o_footer_content row border-top pt-2" position="after">
            <div class="o_footer_content row border-top pt-2"
                 t-if="custom_footer and company.account_fiscal_country_id.code == 'HU'">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_striped" inherit_id="web.external_layout_striped" >
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>

        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company.account_fiscal_country_id.code == 'HU'">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content border-top pt-2 text-center" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div class="o_footer_content border-top pt-2 text-center" position="after">
            <div class="o_footer_content border-top pt-2 text-center"
                 t-if="custom_footer and company.account_fiscal_country_id.code == 'HU'">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_bubble" inherit_id="web.external_layout_bubble">
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>

        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company.account_fiscal_country_id.code == 'HU'">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-4 text-center" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-4 text-center" position="after">
            <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-4 text-center"
                 t-if="custom_footer and company.account_fiscal_country_id.code == 'HU'">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_wave" inherit_id="web.external_layout_wave">
        <!-- support for custom header -->
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>

        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company.account_fiscal_country_id.code == 'HU'">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-5 text-center" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-5 text-center" position="after">
            <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-5 text-center"
                 t-if="custom_footer and company.account_fiscal_country_id.code == 'HU'">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_folder" inherit_id="web.external_layout_folder">
        <!-- support for custom header -->
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>

        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company.account_fiscal_country_id.code == 'HU'">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content d-flex border-top pt-2" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div class="o_footer_content d-flex border-top pt-2" position="after">
            <div class="o_footer_content border-top pt-2"
                 t-if="custom_footer and company.account_fiscal_country_id.code == 'HU'">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_company_form_l10n_hu_edi" model="ir.ui.view">
        <field name="name">res.company.form.l10n_hu_edi</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="account.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='company_registry']" position="after">
                <field name="account_fiscal_country_id" invisible="1"/>
                <field name="l10n_hu_group_vat" invisible="account_fiscal_country_id != %(base.hu)d"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_form_inherit_l10n_hu_edi" model="ir.ui.view">
        <field name="name">res.config.settings.form.inherit.l10n.hu.edi</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@id='invoicing_settings']" position="after">
                <block title="Hungarian Electronic Invoicing"
                       invisible="country_code != 'HU'"
                       groups="account.group_account_manager"
                       name="l10n_hu_edi_settings">
                    <setting company_dependent="1" string="NAV Credentials"
                             help="Enter your e-invoicing credentials given by the Hungarian Authority."
                             name="l10n_hu_edi_nav_credentials">
                        <div class="row">
                            <field name="l10n_hu_edi_is_active" invisible="1"/>
                            <div class="alert alert-success text-center ms-3" role="alert"
                                 invisible="not l10n_hu_edi_is_active">
                                Authentication with NAV 3.0 successful.
                            </div>
                        </div>
                        <div class="row">
                            <label string="Mode" for="l10n_hu_edi_server_mode" class="col-lg-3 o_light_label"/>
                            <field name="l10n_hu_edi_server_mode"/>
                        </div>
                        <div class="row">
                            <label string="Username" for="l10n_hu_edi_username" class="col-lg-3 o_light_label"/>
                            <field name="l10n_hu_edi_username"/>
                        </div>
                        <div class="row">
                            <label string="Password" for="l10n_hu_edi_password" class="col-lg-3 o_light_label"/>
                            <field name="l10n_hu_edi_password"/>
                        </div>
                        <div class="row">
                            <label string="Signature Key" for="l10n_hu_edi_signature_key" class="col-lg-3 o_light_label"/>
                            <field name="l10n_hu_edi_signature_key"/>
                        </div>
                        <div class="row">
                            <label string="Replacement Key" for="l10n_hu_edi_replacement_key" class="col-lg-3 o_light_label"/>
                            <field name="l10n_hu_edi_replacement_key"/>
                        </div>
                    </setting>
                    <setting company_dependent="1"
                             help="Your company's specific tax arrangements, if any of these apply to your company."
                             name="l10n_hu_edi_specials">
                        <field name="l10n_hu_tax_regime"/>
                    </setting>
                </block>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_partner_form_l10n_hu_edi" model="ir.ui.view">
        <field name="name">res.partner.form.l10n_hu_edi</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='credit_limits']" position="after">
                <group string="Hungary"
                       name="hungary"
                       invisible="country_code != 'HU'">
                    <field name="l10n_hu_group_vat"/>
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\uom_uom_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="uom_uom_form_inherit_l10n_hu_edi" model="ir.ui.view">
        <field name="name">uom.uom.inherit.l10n_hu_edi</field>
        <field name="model">uom.uom</field>
        <field name="inherit_id" ref="uom.product_uom_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='uom_type']" position="after">
                <field name="l10n_hu_edi_code" invisible="'HU' not in fiscal_country_codes"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\account_move_reversal.py

```python
from odoo import models


class AccountMoveReversal(models.TransientModel):
    _inherit = 'account.move.reversal'

    def reverse_moves(self, is_modify=False):
        action = super().reverse_moves(is_modify=is_modify)
        if is_modify:
            # In Hungary, if we do `Reverse and Create Invoice`, the new invoice should have a debit_origin_id pointing to the old invoice.
            # Match new invoices to old invoices based on (move_type, journal_id, partner_id, amount_total_in_currency_signed).
            for origin in self.move_ids.filtered(lambda m: m.l10n_hu_edi_state):
                matched_new_move = self.new_move_ids.filtered(
                    lambda m: (
                        (m.move_type, m.journal_id, m.partner_id, m.amount_total_in_currency_signed)
                        == (origin.move_type, origin.journal_id, origin.partner_id, origin.amount_total_in_currency_signed)
                    )
                )
                matched_new_move.write({'debit_origin_id': origin.id})
        return action

```

## File: wizard\l10n_hu_edi_cancellation.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import time

from odoo import models, fields
from odoo.exceptions import UserError
from odoo.addons.l10n_hu_edi.models.l10n_hu_edi_connection import L10nHuEdiConnection


class L10nHuEdiCancellation(models.TransientModel):
    _name = 'l10n_hu_edi.cancellation'
    _description = 'Technical Annulment Wizard'

    invoice_id = fields.Many2one(
        comodel_name='account.move',
        string='Invoice to cancel',
    )
    code = fields.Selection(
        selection=[
            ('ERRATIC_DATA', 'ERRATIC_DATA - Erroneous data'),
            ('ERRATIC_INVOICE_NUMBER', 'ERRATIC_INVOICE_NUMBER - Erroneous invoice number'),
            ('ERRATIC_INVOICE_ISSUE_DATE', 'ERRATIC_INVOICE_ISSUE_DATE - Erroneous issue date'),
        ],
        string='Annulment Code',
        required=True,
    )
    reason = fields.Char(
        string='Annulment Reason',
        required=True,
    )

    def button_request_cancel(self):
        with L10nHuEdiConnection(self.env) as connection:
            self.invoice_id._l10n_hu_edi_acquire_lock()
            self.invoice_id._l10n_hu_edi_request_cancel(connection, self.code, self.reason)

            if 'query_status' in self.invoice_id._l10n_hu_edi_get_valid_actions():
                time.sleep(2)
                self.invoice_id._l10n_hu_edi_query_status(connection)

        formatted_message = self.env['account.move.send']._format_error_html(self.invoice_id.l10n_hu_edi_messages)
        self.invoice_id.with_context(no_new_invoice=True).message_post(body=formatted_message)

        if self.env['account.move.send']._can_commit():
            self.env.cr.commit()

        if self.invoice_id.l10n_hu_edi_messages.get('blocking_level') == 'error':
            raise UserError(self.env['account.move.send']._format_error_text(self.invoice_id.l10n_hu_edi_messages))

```

## File: wizard\l10n_hu_edi_cancellation.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="l10n_hu_edi_cancellation_form" model="ir.ui.view">
        <field name="name">l10n_hu_edi.cancellation.form</field>
        <field name="model">l10n_hu_edi.cancellation</field>
        <field name="arch" type="xml">
            <form>
                <div class="alert alert-warning" role="alert">
                    Technical Annulment should only be used when an error in the software caused an incorrect data report.<br/>
                    To cancel an invoice / credit note in a normal business flow, please create a credit note / debit note.
                </div>
                <group>
                    <field name="code"/>
                    <field name="reason"/>
                </group>
                <footer>
                    <button string="Request Annulment" name="button_request_cancel" type="object" default_focus="1" class="btn-primary"/>
                    <button string="Close" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>
```

## File: wizard\l10n_hu_edi_tax_audit_export.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import contextlib
import io
import zipfile

from odoo import api, models, fields, _
from odoo.exceptions import UserError


class L10nHuEdiTaxAuditExport(models.TransientModel):
    _name = 'l10n_hu_edi.tax_audit_export'
    _description = 'Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás'

    selection_mode = fields.Selection(
        string='Selection mode',
        selection=[
            ('date', 'By date'),
            ('name', 'By serial number'),
        ],
        default='date',
    )
    date_from = fields.Date(
        string='Date From'
    )
    date_to = fields.Date(
        string='Date To'
    )
    name_from = fields.Char(
        string='Name From'
    )
    name_to = fields.Char(
        string='Name To'
    )
    filename = fields.Char(
        string='File name',
        compute='_compute_filename'
    )
    export_file = fields.Binary(
        string='Generated File',
        readonly=True
    )

    @api.depends('selection_mode', 'date_from', 'date_to', 'name_from', 'name_to')
    def _compute_filename(self):
        domain = [
            ('move_type', 'in', ('out_invoice', 'out_refund')),
            ('state', '=', 'posted'),
            ('country_code', '=', 'HU'),
        ]
        if self.selection_mode == 'date':
            date_from = self.date_from
            if not date_from:
                first_invoice = self.env['account.move'].search(domain, order='date', limit=1)
                date_from = first_invoice.date
            date_to = self.date_to or fields.Date.today()
            self.filename = f'export_{date_from}_{date_to}.zip'

        else:
            name_from = self.name_from
            if not name_from:
                first_invoice = self.env['account.move'].search(domain, order='name', limit=1)
                name_from = first_invoice.name
            name_to = self.name_to
            if not name_to:
                last_invoice = self.env['account.move'].search(domain, order='name desc', limit=1)
                name_to = last_invoice.name
            self.filename = f'export_{name_from.replace("/", "")}_{name_to.replace("/", "")}.zip'

    def action_export(self):
        self.ensure_one()
        domain = [
            ('move_type', 'in', ('out_invoice', 'out_refund')),
            ('state', '=', 'posted'),
            ('country_code', '=', 'HU'),
        ]
        if self.selection_mode == 'date':
            if self.date_from:
                domain.append(('date', '>=', self.date_from))
            if self.date_to:
                domain.append(('date', '<=', self.date_to))
        else:
            if self.name_from:
                domain.append(('name', '>=', self.name_from))
            if self.name_to:
                domain.append(('name', '<=', self.name_to))

        invoices = self.env['account.move'].search(domain)
        if not invoices:
            raise UserError(_('No invoice to export!'))

        with io.BytesIO() as buf:
            with zipfile.ZipFile(buf, mode='w', compression=zipfile.ZIP_DEFLATED, allowZip64=False) as zf:
                # To correctly generate the XML for invoices created before l10n_hu_edi was installed,
                # we need to temporarily set the chain index and line numbers, so we do this in a savepoint.
                with contextlib.closing(self.env.cr.savepoint(flush=False)):
                    for invoice in invoices.sorted(lambda i: i.create_date):
                        if invoice.l10n_hu_edi_state:
                            # Case 1: An XML was already generated for this invoice.
                            invoice_xml = base64.b64decode(invoice.l10n_hu_edi_attachment)
                        else:
                            # Case 2: No XML was generated for this invoice.
                            if not invoice.l10n_hu_invoice_chain_index:
                                invoice._l10n_hu_edi_set_chain_index()
                            invoice_xml = invoice._l10n_hu_edi_generate_xml()

                        filename = f'{invoice.name.replace("/", "_")}.xml'
                        zf.writestr(filename, invoice_xml)
            self.export_file = base64.b64encode(buf.getvalue())

        return {
            'type': 'ir.actions.act_window',
            'res_model': self._name,
            'view_mode': 'form',
            'res_id': self.id,
            'views': [(False, 'form')],
            'target': 'new',
        }

```

## File: wizard\l10n_hu_edi_tax_audit_export.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="l10n_hu_edi_tax_audit_export_form" model="ir.ui.view">
        <field name="name">l10n_hu_edi.tax_audit_export.form</field>
        <field name="model">l10n_hu_edi.tax_audit_export</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="selection_mode" widget="radio"/>
                    <group>
                        <field name="date_from" invisible="selection_mode != 'date'"/>
                        <field name="name_from" invisible="selection_mode != 'name'"/>
                    </group>
                    <group>
                        <field name="date_to" invisible="selection_mode != 'date'"/>
                        <field name="name_to" invisible="selection_mode != 'name'"/>
                    </group>
                </group>
                <div invisible="not export_file">
                    <field name="export_file" widget="binary" filename="filename" readonly="1"/>
                    <field name="filename" invisible="1"/>
                </div>
                <footer>
                    <button string="Export" name="action_export" type="object" default_focus="1" class="btn-primary"/>
                    <button string="Close" special="cancel" default_focus="1" class="btn-primary"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_l10n_hu_edi_tax_audit_export_form" model="ir.actions.act_window">
        <field name="name">Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás</field>
        <field name="res_model">l10n_hu_edi.tax_audit_export</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="l10n_hu_edi.l10n_hu_edi_tax_audit_export_form"/>
        <field name="target">new</field>
    </record>

    <menuitem id="menu_finance_reports_hu" name="Hungary" parent="account.menu_finance_reports" sequence="30"/>
    <menuitem id="menu_hu_tax_audit_export"
              name="Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás"
              parent="menu_finance_reports_hu"
              sequence="40"
              action="action_l10n_hu_edi_tax_audit_export_form"/>

</odoo>
```

## File: wizard\__init__.py

```python
from . import account_move_reversal
from . import l10n_hu_edi_cancellation
from . import l10n_hu_edi_tax_audit_export

```

