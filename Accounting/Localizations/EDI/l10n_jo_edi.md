# Odoo Module: l10n_jo_edi

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import wizard

```

## File: __manifest__.py

```python
{
    'name': 'Jordan E-Invoicing',
    'countries': ['jo'],
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'summary': 'Electronic Invoicing for Jordan UBL 2.1',
    'author': 'Odoo S.A., Smart Way Business Solutions',
    'description': """
       Allows the users to integrate with JoFotara.
    """,
    'depends': ['account_edi_ubl_cii', 'l10n_jo'],
    'data': [
        'data/ubl_jo_templates.xml',
        'views/account_move_views.xml',
        'views/report_invoice.xml',
        'views/res_config_settings_views.xml',
        'wizard/account_move_send_views.xml',
    ],
    'installable': True,
    'auto_install': ['l10n_jo'],
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
UPDATE res_company
   SET l10n_jo_edi_secret_key = 'dummy_secret',
       l10n_jo_edi_client_identifier = 'dummy_id';


```

## File: data\ubl_jo_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="ubl_jo_InvoiceType" inherit_id="account_edi_ubl_cii.ubl_20_InvoiceType" primary="True">
        <xpath expr="//*[local-name()='DocumentCurrencyCode']" position="attributes">
            <attribute name="t-out">vals['document_currency_code']</attribute>
        </xpath>

        <xpath expr="//*[local-name()='BillingReference']//*[local-name()='InvoiceDocumentReference']" position="inside">
            <cbc:UUID
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-out="billing_reference_vals.get('uuid')"/>
            <cbc:DocumentDescription
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-out="billing_reference_vals.get('document_description')"/>
        </xpath>

        <xpath expr="//*[local-name()='AdditionalDocumentReference']//*[local-name()='ID']" position="after">
            <cbc:UUID
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-out="foreach_vals['uuid']"/>
        </xpath>

        <xpath expr="//*[local-name()='AccountingCustomerParty']" position="inside">
            <cac:AccountingContact
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:Telephone t-out="accounting_customer_vals['accounting_contact']['telephone']"/>
            </cac:AccountingContact>
        </xpath>

        <xpath expr="//*[local-name()='AccountingCustomerParty']" position="after">
            <cac:SellerSupplierParty
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <t t-set="seller_supplier_vals" t-value="vals['seller_supplier_party_vals']"/>
                <cac:Party>
                    <t t-call="{{PartyType_template}}">
                        <t t-set="vals" t-value="seller_supplier_vals['party_vals']"/>
                    </t>
                </cac:Party>
            </cac:SellerSupplierParty>
        </xpath>
    </template>

    <template id="ubl_jo_Invoice">
        <Invoice
            xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
            xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
            xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
            xmlns:ext="urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2">
            <t t-call="{{InvoiceType_template}}"/>
        </Invoice>
    </template>

    <template id="ubl_jo_PaymentMeansType" inherit_id="account_edi_ubl_cii.ubl_20_PaymentMeansType" primary="True">
        <xpath expr="//*[local-name()='PaymentMeansCode']" position="after">
            <cbc:InstructionNote
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-out="vals['instruction_note']"/>
        </xpath>
    </template>

    <template id="ubl_jo_InvoiceLineType" inherit_id="account_edi_ubl_cii.ubl_20_InvoiceLineType" primary="True">
        <xpath expr="//*[local-name()='AllowanceCharge']" position="replace"/>

        <xpath expr="//*[local-name()='Price']//*[local-name()='BaseQuantity']" position="after">
            <cac:AllowanceCharge
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                t-foreach="price_vals['allowance_charge_vals']" t-as="foreach_vals">
                <t t-call="{{AllowanceChargeType_template}}">
                    <t t-set="vals" t-value="foreach_vals"/>
                </t>
            </cac:AllowanceCharge>
        </xpath>

        <xpath expr="//*[local-name()='Price']//*[local-name()='PriceAmount']" position="attributes">
            <attribute name="t-out">
                format_float(price_vals['price_amount'], price_vals['product_price_dp'])
            </attribute>
        </xpath>
    </template>

    <template id="ubl_jo_TaxTotalType" inherit_id="account_edi_ubl_cii.ubl_20_TaxTotalType" primary="True">
        <xpath expr="//*[local-name()='TaxAmount']" position="after">
            <cbc:RoundingAmount
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('rounding_amount'), vals.get('currency_dp'))"/>
        </xpath>
    </template>
</odoo>

```

## File: models\account_edi_xml_ubl_21_jo.py

```python
from types import SimpleNamespace

from odoo import models
from odoo.tools import float_repr
from odoo.tools.float_utils import float_round


# There is a need for this dummy currency because:
# 1. In `ubl_20_templates.xml`, the currency name is read from currency like `vals['currency'].name`
# 2. In Jordanian EDI XML documentation, certain locations expect the currency name to be `JO` not `JOD`.
JO_CURRENCY = SimpleNamespace(name='JO')

JO_MAX_DP = 9

PAYMENT_CODES_MAP = {
    'income': {
        'cash': '011',
        'receivable': '021',
    },
    'sales': {
        'cash': '012',
        'receivable': '022',
    },
    'special': {
        'cash': '013',
        'receivable': '023',
    }
}


class AccountEdiXmlUBL21JO(models.AbstractModel):
    _name = "account.edi.xml.ubl_21.jo"
    _inherit = 'account.edi.xml.ubl_21'
    _description = "UBL 2.1 (JoFotara)"

    ####################################################
    # helper functions
    ####################################################

    def _round_max_dp(self, value):
        return float_round(value, JO_MAX_DP)

    def _get_line_amount_before_discount_jod(self, line):
        amount_after_discount = abs(line.balance)
        return (amount_after_discount / (1 - line.discount / 100)) \
            if line.discount < 100 else line.currency_id._convert(
            from_amount=line.price_unit * line.quantity,
            to_currency=self.env.ref('base.JOD'),
            company=line.company_id,
            date=line.date,
        )

    def _get_line_discount_jod(self, line):
        return self._get_line_amount_before_discount_jod(line) * line.discount / 100

    def _get_unit_price_jod(self, line):
        return self._get_line_amount_before_discount_jod(line) / line.quantity

    def _get_line_taxable_amount(self, line):
        return self._round_max_dp(self._get_unit_price_jod(line)) * self._round_max_dp(line.quantity) - self._round_max_dp(self._get_line_discount_jod(line))

    def _get_payment_method_code(self, invoice):
        return PAYMENT_CODES_MAP[invoice.company_id.l10n_jo_edi_taxpayer_type]['receivable']

    def _aggregate_totals(self, vals):
        """
        This method is needed to ensure that units sum up to total values.
        ===================================================================================================
        Problem statement can be found inside tests/test_jo_edi_precision.py in the docstring of _validate_jo_edi_numbers
        -------------------------------------------------------------------------------
        Solution:
        taxes_vals (calculated from invoice._prepare_invoice_aggregated_taxes() in `account_edi_xml_ubl_20` module)
        is calculated to generate units with no rounding ensuring that these when aggregated sum up to the totals stored in Odoo.
        This method here uses these unit to calculate the totals again so that JoFotara validations don't fail.
        The difference between reported totals and Odoo stored totals in this case is < 0.001 JOD.
        """
        tax_inclusive_amount = 0
        tax_exclusive_amount = 0
        for line_val in vals['line_vals']:
            price_unit = self._round_max_dp(line_val['price_vals']['price_amount'])
            quantity = self._round_max_dp(line_val['line_quantity'])
            discount = self._round_max_dp(line_val['price_vals']['allowance_charge_vals'][0]['amount'])

            total_tax_amount = 0
            if line_val['tax_total_vals']:
                total_tax_amount = self._round_max_dp(sum(self._round_max_dp(subtotal['tax_amount']) for subtotal in line_val['tax_total_vals'][0]['tax_subtotal_vals']))
                line_val['tax_total_vals'][0]['rounding_amount'] = self._round_max_dp(line_val['line_extension_amount']) + total_tax_amount

            tax_exclusive_amount += self._round_max_dp(price_unit * quantity)
            tax_inclusive_amount += self._round_max_dp((price_unit * quantity) - discount + total_tax_amount)

        vals['monetary_total_vals']['tax_inclusive_amount'] = vals['monetary_total_vals']['payable_amount'] = tax_inclusive_amount
        vals['monetary_total_vals']['tax_exclusive_amount'] = tax_exclusive_amount

    ########################################################
    # overriding vals methods of account_edi_xml_ubl_20 file
    ########################################################

    def _get_country_vals(self, country):
        return {
            'identification_code': country.code,
        }

    def _get_partner_party_identification_vals_list(self, partner):
        return [{
            'id_attrs': {'schemeID': 'TN' if not partner.country_code or partner.country_code == 'JO' else 'PN'},
            'id': partner.vat if partner.vat and partner.vat != '/' else '',
        }]

    def _get_partner_address_vals(self, partner):
        return {
            'postal_zone': partner.zip,
            'country_subentity_code': partner.state_id.code,
            'country_vals': self._get_country_vals(partner.country_id),
        }

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        return [{
            'company_id': partner.vat,
            'tax_scheme_vals': {'id': 'VAT'},
        }]

    def _get_partner_party_legal_entity_vals_list(self, partner):
        return [{
            'registration_name': partner.name,
        }]

    def _get_partner_contact_vals(self, partner):
        return {}

    def _get_empty_party_vals(self):
        return {
            'postal_address_vals': {'country_vals': {'identification_code': 'JO'}},
            'party_tax_scheme_vals': [{'tax_scheme_vals': {'id': 'VAT'}}],
        }

    def _get_partner_party_vals(self, partner, role):
        vals = super()._get_partner_party_vals(partner, role)
        vals['party_name_vals'] = []
        if role == 'supplier':
            vals['party_identification_vals'] = []
        return vals

    def _get_delivery_vals_list(self, invoice):
        return []

    def _get_invoice_payment_means_vals_list(self, invoice):
        if invoice.move_type == 'out_refund':
            return [{
                'payment_means_code': 10,
                'payment_means_code_attrs': {'listID': "UN/ECE 4461"},
                'instruction_note': invoice.ref.replace('/', '_') if invoice.ref else '',
            }]
        else:
            return []

    def _get_invoice_payment_terms_vals_list(self, invoice):
        return []

    def _get_invoice_tax_totals_vals_helper(self, taxes_vals, special_tax_amount_per_line):
        tax_totals_vals = {
            'currency': JO_CURRENCY,
            'currency_dp': self._get_currency_decimal_places(),
            'tax_amount': 0,
            'tax_subtotal_vals': [],
        }
        for grouping_key, vals in taxes_vals['tax_details'].items():
            if grouping_key['tax_amount_type'] != 'fixed':
                special_tax_amount = sum(special_tax_amount_per_line.get(line, 0) for line in vals['records'])
                taxable_amount = sum(self._round_max_dp(self._get_line_taxable_amount(line)) for line in vals['records']) + special_tax_amount
                subtotal = {
                    'currency': JO_CURRENCY,
                    'currency_dp': self._get_currency_decimal_places(),
                    'taxable_amount': taxable_amount,
                    'tax_amount': self._round_max_dp(taxable_amount) * vals['tax_category_percent'] / 100,
                    'tax_category_vals': vals['_tax_category_vals_'],
                }
                tax_totals_vals['tax_subtotal_vals'].append(subtotal)
                tax_totals_vals['tax_amount'] += subtotal['tax_amount']

        return tax_totals_vals

    def _get_invoice_tax_totals_vals_list(self, invoice, taxes_vals):
        # Tax unregistered companies should have no tax values
        if invoice.company_id.l10n_jo_edi_taxpayer_type == 'income':
            return []

        special_tax_amount_per_line = {
            line: line_tax['tax_amount']
            for line, line_vals in taxes_vals['tax_details_per_record'].items()
            for line_tax in line_vals['tax_details'].values()
            if 'tax_amount_type' in line_tax and line_tax['tax_amount_type'] == 'fixed'
        }
        vals = self._get_invoice_tax_totals_vals_helper(taxes_vals, special_tax_amount_per_line)
        if not invoice._is_sales_refund():
            vals['tax_subtotal_vals'] = []
        return [vals]

    def _get_invoice_line_item_vals(self, line, taxes_vals):
        product = line.product_id
        description = line.name and line.name.replace('\n', ', ')
        return {
            'name': product.name or description,
        }

    def _get_document_allowance_charge_vals_list(self, invoice):
        """ For JO UBL the document allowance charge vals needs to be the sum of the line discounts. """
        discount_amount = 0
        invoice_lines = invoice.invoice_line_ids.filtered(lambda line: line.display_type not in ('line_note', 'line_section'))
        for line in invoice_lines:
            discount_amount += self._round_max_dp(self._get_line_discount_jod(line))
        return [{
            'charge_indicator': 'false',
            'allowance_charge_reason': 'discount',
            'currency_name': JO_CURRENCY.name,
            'currency_dp': self._get_currency_decimal_places(),
            'amount': discount_amount,
        }]

    def _get_invoice_line_allowance_vals_list(self, line, taxes_vals):
        return [{
            'charge_indicator': 'false',
            'allowance_charge_reason': 'DISCOUNT',
            'currency_name': JO_CURRENCY.name,
            'currency_dp': self._get_currency_decimal_places(),
            'amount': self._get_line_discount_jod(line),
        }]

    def _get_invoice_line_price_vals(self, line, taxes_vals):
        return {
            'currency': JO_CURRENCY,
            'currency_dp': self._get_currency_decimal_places(),
            'price_amount': self._get_unit_price_jod(line),
            'product_price_dp': self._get_currency_decimal_places(),
            'allowance_charge_vals': self._get_invoice_line_allowance_vals_list(line, taxes_vals),
            'base_quantity': None,
            'base_quantity_attrs': {'unitCode': self._get_uom_unece_code()},
        }

    def _get_invoice_line_tax_totals_vals_list(self, line, taxes_vals):
        # Tax unregistered companies should have no tax values
        if line.move_id.company_id.l10n_jo_edi_taxpayer_type == 'income':
            return []

        special_tax_amount = 0
        special_tax_subtotal = None
        taxable_amount = 0
        for grouping_key, tax_details_vals in taxes_vals['tax_details'].items():
            if grouping_key['tax_amount_type'] == 'fixed':
                taxable_amount = sum(self._round_max_dp(self._get_line_taxable_amount(line)) for line in tax_details_vals['records'])
                special_tax_amount = tax_details_vals['tax_amount']
                special_tax_subtotal = {
                    'currency': JO_CURRENCY,
                    'currency_dp': self._get_currency_decimal_places(),
                    'taxable_amount': taxable_amount,
                    'tax_amount': special_tax_amount,
                    'tax_category_vals': tax_details_vals['_tax_category_vals_'],
                }
        vals = self._get_invoice_tax_totals_vals_helper(taxes_vals, special_tax_amount_per_line={line: special_tax_amount})
        if special_tax_subtotal:
            vals['tax_subtotal_vals'].insert(0, special_tax_subtotal)
            vals['tax_subtotal_vals'][1]['taxable_amount'] = taxable_amount
        return [vals]

    def _get_invoice_line_vals(self, line, line_id, taxes_vals):
        return {
            'currency': JO_CURRENCY,
            'currency_dp': self._get_currency_decimal_places(),
            'id': line_id + 1,
            'line_quantity': line.quantity,
            'line_quantity_attrs': {'unitCode': self._get_uom_unece_code()},
            'line_extension_amount': self._get_line_taxable_amount(line),
            'tax_total_vals': self._get_invoice_line_tax_totals_vals_list(line, taxes_vals),
            'item_vals': self._get_invoice_line_item_vals(line, taxes_vals),
            'price_vals': self._get_invoice_line_price_vals(line, taxes_vals),
        }

    def _get_invoice_monetary_total_vals(self, invoice, taxes_vals, line_extension_amount, allowance_total_amount, charge_total_amount):
        return {
            'currency': JO_CURRENCY,
            'currency_dp': self._get_currency_decimal_places(),
            'allowance_total_amount': allowance_total_amount,
            'prepaid_amount': 0 if invoice._is_sales_refund() else None,
        }

    ####################################################
    # overriding vals methods of account_edi_common file
    ####################################################

    def format_float(self, amount, precision_digits):
        if amount is None:
            return None

        def get_decimal_places(number):
            return len(f'{float(number)}'.split('.')[1])

        rounded_amount = float_repr(self._round_max_dp(amount), JO_MAX_DP).rstrip('0').rstrip('.')
        decimal_places = get_decimal_places(rounded_amount)
        if decimal_places < precision_digits:
            rounded_amount = float_repr(float(rounded_amount), precision_digits)
        return rounded_amount

    def _get_currency_decimal_places(self, currency_id=None):
        # Invoices are always reported in JOD
        return self.env.ref('base.JOD').decimal_places

    def _get_uom_unece_code(self, line=None):
        return "PCE"

    def _get_tax_category_list(self, invoice, taxes):
        def get_tax_jo_ubl_code(tax):
            if tax._l10n_jo_is_exempt_tax():
                return "Z"
            if tax.amount:
                return "S"
            return "O"

        def get_jo_tax_type(tax):
            if tax.amount_type == 'percent':
                return 'general'
            elif tax.amount_type == 'fixed':
                return 'special'

        res = []
        for tax in taxes:
            tax_type = get_jo_tax_type(tax)
            tax_code = get_tax_jo_ubl_code(tax)
            res.append({
                'id': tax_code,
                'id_attrs': {'schemeAgencyID': '6', 'schemeID': 'UN/ECE 5305'},
                'percent': tax.amount if tax_type == 'general' else '',
                'tax_scheme_vals': {
                    'id': 'VAT' if tax_type == 'general' else 'OTH',
                    'id_attrs': {
                        'schemeAgencyID': '6',
                        'schemeID': 'UN/ECE 5153',
                    },
                },
            })
        return res

    ####################################################
    # vals methods of account.edi.xml.ubl_21.jo
    ####################################################

    def _get_billing_reference_vals(self, invoice):
        if not invoice.reversed_entry_id:
            return {}

        return {
            'id': invoice.reversed_entry_id.name.replace('/', '_'),
            'uuid': invoice.reversed_entry_id.l10n_jo_edi_uuid,
            'document_description': self.format_float(abs(invoice.reversed_entry_id.amount_total_signed), self._get_currency_decimal_places()),
        }

    def _get_additional_document_reference_list(self, invoice):
        return [{
            'id': 'ICV',
            'uuid': invoice.id,
        }]

    def _get_seller_supplier_party_vals(self, invoice):
        return {
            'party_identification_vals': [{'id': invoice.company_id.l10n_jo_edi_sequence_income_source}],
        }

    ####################################################
    # export methods
    ####################################################

    def _export_invoice_vals(self, invoice):
        vals = super()._export_invoice_vals(invoice)

        vals.update({
            'main_template': 'l10n_jo_edi.ubl_jo_Invoice',
            'InvoiceType_template': 'l10n_jo_edi.ubl_jo_InvoiceType',
            'PaymentMeansType_template': 'l10n_jo_edi.ubl_jo_PaymentMeansType',
            'InvoiceLineType_template': 'l10n_jo_edi.ubl_jo_InvoiceLineType',
            'TaxTotalType_template': 'l10n_jo_edi.ubl_jo_TaxTotalType',
        })

        customer = invoice.partner_id
        is_refund = invoice.move_type == 'out_refund'

        vals['vals'].update({
            'ubl_version_id': '',
            'order_reference': '',
            'sales_order_id': '',
            'profile_id': 'reporting:1.0',
            'id': invoice.name.replace('/', '_'),
            'uuid': invoice.l10n_jo_edi_uuid,
            'document_currency_code': 'JOD',
            'tax_currency_code': 'JOD',
            'document_type_code_attrs': {'name': self._get_payment_method_code(invoice)},
            'document_type_code': "381" if is_refund else "388",
            'accounting_customer_party_vals': {
                'party_vals': self._get_empty_party_vals() if is_refund else self._get_partner_party_vals(customer, role='customer'),
                'accounting_contact': {
                    'telephone': '' if is_refund else invoice.partner_id.phone or invoice.partner_id.mobile,
                },
            },
            'seller_supplier_party_vals': {
                'party_vals': self._get_seller_supplier_party_vals(invoice),
            },
            'billing_reference_vals': self._get_billing_reference_vals(invoice),
            'additional_document_reference_list': self._get_additional_document_reference_list(invoice),
        })

        self._aggregate_totals(vals['vals'])

        return vals

```

## File: models\account_move.py

```python
import base64
import requests
import uuid
from werkzeug.urls import url_encode

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

JOFOTARA_URL = "https://backend.jofotara.gov.jo/core/invoices/"


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_jo_edi_uuid = fields.Char(string="Invoice UUID", copy=False, compute="_compute_l10n_jo_edi_uuid", store=True)
    l10n_jo_edi_qr = fields.Char(string="QR", copy=False)

    l10n_jo_edi_is_needed = fields.Boolean(
        compute="_compute_l10n_jo_edi_is_needed",
        help="Jordan: technical field to determine if this invoice is eligible to be e-invoiced.",
    )
    l10n_jo_edi_state = fields.Selection(
        selection=[('to_send', 'To Send'), ('sent', 'Sent')],
        string="JoFotara State",
        copy=False)
    l10n_jo_edi_error = fields.Text(
        string="JoFotara Error",
        copy=False,
        readonly=True,
        help="Jordan: Error details.",
    )
    l10n_jo_edi_computed_xml = fields.Binary(
        string="Jordan E-Invoice computed XML File",
        compute="_compute_l10n_jo_edi_computed_xml",
        help="Jordan: technical field computing e-invoice XML data, useful at submission failure scenarios.",
    )
    l10n_jo_edi_xml_attachment_file = fields.Binary(
        string="Jordan E-Invoice XML File",
        copy=False,
        attachment=True,
        help="Jordan: technical field holding the e-invoice XML data.",
    )
    l10n_jo_edi_xml_attachment_id = fields.Many2one(
        comodel_name="ir.attachment",
        string="Jordan E-Invoice XML",
        compute=lambda self: self._compute_linked_attachment_id(
            "l10n_jo_edi_xml_attachment_id", "l10n_jo_edi_xml_attachment_file"
        ),
        depends=["l10n_jo_edi_xml_attachment_file"],
        help="Jordan: e-invoice XML.",
    )

    @api.depends("country_code", "move_type")
    def _compute_l10n_jo_edi_is_needed(self):
        for move in self:
            move.l10n_jo_edi_is_needed = (
                move.country_code == "JO"
                and move.move_type in ("out_invoice", "out_refund")
            )

    @api.depends("l10n_jo_edi_state")
    def _compute_show_reset_to_draft_button(self):
        # EXTENDS 'account'
        super()._compute_show_reset_to_draft_button()
        self.filtered(lambda move: move.l10n_jo_edi_state == 'sent').show_reset_to_draft_button = False

    @api.depends("l10n_jo_edi_is_needed")
    def _compute_l10n_jo_edi_uuid(self):
        for invoice in self:
            if invoice.l10n_jo_edi_is_needed and not invoice.l10n_jo_edi_uuid:
                invoice.l10n_jo_edi_uuid = uuid.uuid4()

    def _compute_l10n_jo_edi_computed_xml(self):
        for invoice in self:
            xml_content = self.env['account.edi.xml.ubl_21.jo']._export_invoice(invoice)[0]
            invoice.l10n_jo_edi_computed_xml = base64.b64encode(xml_content)

    def download_l10n_jo_edi_computed_xml(self):
        if error_message := self._l10n_jo_validate_config() or self._l10n_jo_validate_fields():
            raise ValidationError(_("The following errors have to be fixed in order to create an XML:\n") + error_message)
        params = url_encode({
            'model': self._name,
            'id': self.id,
            'field': 'l10n_jo_edi_computed_xml',
            'filename': self._l10n_jo_edi_get_xml_attachment_name(),
            'mimetype': 'application/xml',
            'download': 'true',
        })
        return {'type': 'ir.actions.act_url', 'url': '/web/content/?' + params, 'target': 'new'}

    def _l10n_jo_qr_code_src(self):
        self.ensure_one()
        encoded_params = url_encode({
            'barcode_type': 'QR',
            'value': self.l10n_jo_edi_qr,
            'width': 200,
            'height': 200,
        })
        return f'/report/barcode/?{encoded_params}'

    def _is_sales_refund(self):
        self.ensure_one()
        return self.company_id.l10n_jo_edi_taxpayer_type == 'sales' and self.move_type == 'out_refund'

    def button_draft(self):
        # EXTENDS 'account'
        self.write(
            {
                "l10n_jo_edi_error": False,
                "l10n_jo_edi_state": False,
            }
        )
        return super().button_draft()

    def _post(self, soft=True):
        # EXTENDS 'account'
        for invoice in self.filtered('l10n_jo_edi_is_needed'):
            invoice.l10n_jo_edi_state = 'to_send'
        return super()._post(soft)

    def _get_name_invoice_report(self):
        # EXTENDS account
        self.ensure_one()
        if self.l10n_jo_edi_state == 'sent' and self.l10n_jo_edi_xml_attachment_id:
            return 'l10n_jo_edi.report_invoice_document'
        return super()._get_name_invoice_report()

    def _l10n_jo_build_jofotara_headers(self):
        self.ensure_one()
        return {
            'Client-Id': self.sudo().company_id.l10n_jo_edi_client_identifier,
            'Secret-Key': self.sudo().company_id.l10n_jo_edi_secret_key,
        }

    def _submit_to_jofotara(self):
        self.ensure_one()
        headers = self._l10n_jo_build_jofotara_headers()
        xml_invoice = self.env['account.edi.xml.ubl_21.jo']._export_invoice(self)[0]
        params = {'invoice': base64.b64encode(xml_invoice).decode()}

        try:
            response = requests.post(JOFOTARA_URL, json=params, headers=headers, timeout=50)
        except requests.exceptions.Timeout:
            return _("Request time out! Please try again.")
        except requests.exceptions.RequestException as e:
            return _("Invalid request: %s", e)

        if not response.ok:
            return _("Request failed: %s", response.content.decode())
        dict_response = response.json()
        self.l10n_jo_edi_qr = str(dict_response.get('EINV_QR', ''))
        self.invoice_pdf_report_id.res_field = False
        self.env["ir.attachment"].create(
            {
                "res_model": "account.move",
                "res_id": self.id,
                "res_field": "l10n_jo_edi_xml_attachment_file",
                "name": self._l10n_jo_edi_get_xml_attachment_name(),
                "raw": xml_invoice,
            }
        )
        self.invalidate_recordset(
            fnames=[
                "l10n_jo_edi_xml_attachment_id",
                "l10n_jo_edi_xml_attachment_file",
                "invoice_pdf_report_id",
                "invoice_pdf_report_file",
            ]
        )

    def _l10n_jo_edi_get_xml_attachment_name(self):
        return f"{self.name.replace('/', '_')}_edi.xml"

    def _l10n_jo_validate_config(self):
        error_msg = ''
        if not self.sudo().company_id.l10n_jo_edi_client_identifier:
            error_msg += _("Client ID is missing.\n")
        if not self.sudo().company_id.l10n_jo_edi_secret_key:
            error_msg += _("Secret key is missing.\n")
        if not self.company_id.l10n_jo_edi_taxpayer_type:
            error_msg += _("Taxpayer type is missing.\n")
        if not self.company_id.l10n_jo_edi_sequence_income_source:
            error_msg += _("Activity number (Sequence of income source) is missing.\n")

        if error_msg:
            return _("%s To set: Configuration > Settings > Electronic Invoicing (Jordan)", error_msg)

    def _l10n_jo_validate_fields(self):
        def has_non_digit_vat(partner, partner_type):
            if partner.vat and not partner.vat.isdigit():
                return _("JoFotara portal cannot process %s VAT with non-digit characters in it\n", partner_type)
            return ""
        error_msg = ''

        customer = self.partner_id
        error_msg += has_non_digit_vat(customer, 'customer')

        supplier = self.company_id.partner_id.commercial_partner_id
        error_msg += has_non_digit_vat(supplier, 'supplier')

        if any(
            line.display_type not in ('line_note', 'line_section')
            and (line.quantity < 0 or line.price_unit < 0)
            for line in self.invoice_line_ids
        ):
            error_msg += _("JoFotara portal cannot process negative quantity nor negative price on invoice lines")

        for line in self.invoice_line_ids.filtered(lambda line: line.display_type not in ('line_note', 'line_section')):
            if self.company_id.l10n_jo_edi_taxpayer_type == 'income' and len(line.tax_ids) != 0:
                error_msg += _("No taxes are allowed on invoice lines for taxpayers unregistered in the sales tax")
            elif self.company_id.l10n_jo_edi_taxpayer_type == 'sales' and len(line.tax_ids) != 1:
                error_msg += _("One general tax per invoice line is expected for taxpayers registered in the sales tax")
            elif self.company_id.l10n_jo_edi_taxpayer_type == 'special' and len(line.tax_ids) != 2:
                error_msg += _("One special and one general tax per invoice line is expected for taxpayers registered in the special tax")

        return error_msg

    def _l10n_jo_edi_send(self):
        self.ensure_one()
        # == Lock ==
        self.env['res.company']._with_locked_records(self)
        # == Send ==
        if error_message := self._l10n_jo_validate_config() or self._l10n_jo_validate_fields() or self._submit_to_jofotara():
            self.l10n_jo_edi_error = error_message
            return error_message
        else:
            self.l10n_jo_edi_error = False
            self.l10n_jo_edi_state = 'sent'
            self.with_context(no_new_invoice=True).message_post(
                body=_("E-invoice (JoFotara) submitted successfully."),
                attachment_ids=self.l10n_jo_edi_xml_attachment_id.ids,
            )

```

## File: models\account_tax.py

```python
from odoo import models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    def _l10n_jo_is_exempt_tax(self):
        self.ensure_one()
        exempt_tags = self.env.ref('l10n_jo.tax_report_vat_sale_export_exempt_local_zero_tag')._get_matching_tags()
        exempt_taxes = self.env['account.tax'].search([('repartition_line_ids.tag_ids', 'in', exempt_tags.ids)])
        return self.id in exempt_taxes.ids

```

## File: models\ir_attachment.py

```python
from odoo import _, api, models
from odoo.exceptions import UserError


class IrAttachment(models.Model):
    _inherit = 'ir.attachment'

    @api.ondelete(at_uninstall=True)
    def _except_submitted_invoices_pdfs(self):
        submitted_invoices_pdfs = self.filtered(
            lambda attachment:
            attachment.res_model == 'account.move'
            and attachment.res_id
            and attachment.res_field == 'invoice_pdf_report_file'
        )

        moves = self.env['account.move'].browse(submitted_invoices_pdfs.mapped('res_id')).exists()
        moves_with_jo_qr = moves.filtered('l10n_jo_edi_qr')
        if moves_with_jo_qr:
            raise UserError(_("You cannot delete this Invoice PDF as it has been submitted to JoFotara"))

```

## File: models\res_company.py

```python
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_jo_edi_sequence_income_source = fields.Char(string="Sequence of Income Source")
    l10n_jo_edi_secret_key = fields.Char(string="Jordan EINV Secret Key", groups="base.group_system")
    l10n_jo_edi_client_identifier = fields.Char(string="Jordan EINV Client ID", groups="base.group_system")
    l10n_jo_edi_taxpayer_type = fields.Selection(string="Taxpayer type", selection=[
        ('income', "Unregistered in the sales tax"),
        ('sales', "Registered in the sales tax"),
        ('special', "Registered in the special sales tax"),
    ])

```

## File: models\res_config_settings.py

```python
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_jo_edi_sequence_income_source = fields.Char(string="Sequence of Income Source", related="company_id.l10n_jo_edi_sequence_income_source", readonly=False)
    l10n_jo_edi_secret_key = fields.Char(string="JoFotara Secret Key", related="company_id.l10n_jo_edi_secret_key", readonly=False)
    l10n_jo_edi_client_identifier = fields.Char(string="JoFotara Client ID", related="company_id.l10n_jo_edi_client_identifier", readonly=False)
    l10n_jo_edi_taxpayer_type = fields.Selection(string="Taxpayer type", related="company_id.l10n_jo_edi_taxpayer_type", readonly=False)

```

## File: models\__init__.py

```python
from . import account_edi_xml_ubl_21_jo
from . import account_move
from . import account_tax
from . import ir_attachment
from . import res_company
from . import res_config_settings

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
                <xpath expr="//sheet" position="before">
                    <div class="alert alert-warning" role="alert" invisible="not l10n_jo_edi_error">
                        <div class="p-0 m-0">
                            <span class="mx-1"><b>Warning</b>: this invoice cannot be sent to JoFotara.</span>
                            <a name="download_l10n_jo_edi_computed_xml" type="object" groups="base.group_no_one" class="float-end">Download XML</a>
                        </div>
                        <field name="l10n_jo_edi_error"/>
                    </div>
                </xpath>
                <xpath expr="//group[@name='sale_info_group']" position="inside">
                    <field name="l10n_jo_edi_state" invisible="not l10n_jo_edi_state"/>
                </xpath>
            </field>
        </record>

        <record id="view_out_invoice_tree" model="ir.ui.view">
            <field name="name">account.move.tree</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_out_invoice_tree"/>
            <field name="arch" type="xml">
                <field name="payment_reference" position="before">
                    <field name="l10n_jo_edi_state" optional="hide"/>
                    <field name="l10n_jo_edi_error" optional="hide"/>
                </field>
            </field>
        </record>

        <record id="view_out_credit_note_tree" model="ir.ui.view">
            <field name="name">account.move.tree</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_out_credit_note_tree"/>
            <field name="arch" type="xml">
                <field name="payment_reference" position="before">
                    <field name="l10n_jo_edi_state" optional="hide"/>
                    <field name="l10n_jo_edi_error" optional="hide"/>
                </field>
            </field>
        </record>

        <record id="view_account_invoice_filter" model="ir.ui.view">
            <field name="name">account.invoice.select</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_account_invoice_filter"/>
            <field name="arch" type="xml">
                <xpath expr="//search/group/filter[@name='groupy_by_journal']" position="after">
                    <filter name="l10n_jo_edi_state" context="{'group_by': 'l10n_jo_edi_state'}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
        <xpath expr="//div[@id='qrcode']" position="after">
            <div t-if="o.l10n_jo_edi_qr" name="qr_code">
                <p>
                    <strong class="text-center">JoFotara QR Code</strong>
                    <img style="display:block;"
                        t-att-src="o._l10n_jo_qr_code_src()"/>
                </p>
            </div>
        </xpath>
    </template>

    <!-- Workaround for Studio reports, see odoo/odoo#60660 -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_jo_edi.report_invoice_document'"
                t-call="l10n_jo_edi.report_invoice_document"
                t-lang="lang"/>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="account.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//block[@id='invoicing_settings']" position="after">
                    <block title="Electronic Invoicing (Jordan)" id="l10n_jo_co_settings" invisible="country_code != 'JO'">
                        <setting string="JoFotara Credentials" help="Configure your JoFotara credentials here">
                            <div class="content-group">
                                <div class="row">
                                    <label string="Activity Number" for="l10n_jo_edi_sequence_income_source" class="o_light_label col-lg-5"/>
                                    <field name="l10n_jo_edi_sequence_income_source"/>
                                </div>
                                <div class="row">
                                    <label for="l10n_jo_edi_secret_key" class="col-lg-5 o_light_label"/>
                                    <field name="l10n_jo_edi_secret_key"/>
                                </div>
                                <div class="row">
                                    <label for="l10n_jo_edi_client_identifier" class="col-lg-5 o_light_label"/>
                                    <field name="l10n_jo_edi_client_identifier"/>
                                </div>
                                <div class="row">
                                    <label for="l10n_jo_edi_taxpayer_type" class="col-lg-5 o_light_label"/>
                                    <field name="l10n_jo_edi_taxpayer_type"/>
                                </div>
                            </div>
                        </setting>
                    </block>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\account_move_send.py

```python
from odoo import _, models, fields, api


class AccountMoveSend(models.TransientModel):
    _inherit = "account.move.send"

    l10n_jo_edi_is_visible = fields.Boolean(
        compute="_compute_l10n_jo_edi_is_visible",
        help="Jordan: technical field to determine if the option to submit a Jordanian electronic invoice is visible.",
    )
    l10n_jo_edi_is_enabled = fields.Boolean(
        string="Send JoFotara e-invoice",
        compute="_compute_l10n_jo_edi_is_enabled",
        store=True,
        readonly=False,
        help="Jordan: used to determine whether to submit this e-invoice.",
    )

    @api.depends("move_ids")
    def _compute_l10n_jo_edi_is_visible(self):
        for wizard in self:
            wizard.l10n_jo_edi_is_visible = any(move.l10n_jo_edi_is_needed and move.l10n_jo_edi_state != 'sent' for move in wizard.move_ids)

    @api.depends("l10n_jo_edi_is_visible")
    def _compute_l10n_jo_edi_is_enabled(self):
        for wizard in self:
            wizard.l10n_jo_edi_is_enabled = wizard.l10n_jo_edi_is_visible

    def _get_invoice_extra_attachments(self, move):
        # EXTENDS 'account'
        return super()._get_invoice_extra_attachments(move) + move.l10n_jo_edi_xml_attachment_id

    def _get_placeholder_mail_attachments_data(self, move):
        # EXTENDS 'account'
        res = super()._get_placeholder_mail_attachments_data(move)

        if self.l10n_jo_edi_is_visible and self.l10n_jo_edi_is_enabled:
            attachment_name = move._l10n_jo_edi_get_xml_attachment_name()
            res.append(
                {
                    "id": f"placeholder_{attachment_name}",
                    "name": attachment_name,
                    "mimetype": "application/xml",
                    "placeholder": True,
                }
            )
        return res

    def _get_wizard_values(self):
        # EXTENDS 'account'
        res = super()._get_wizard_values()
        res["l10n_jo_edi_is_enabled"] = self.l10n_jo_edi_is_enabled
        return res

    @api.model
    def _get_wizard_vals_restrict_to(self, only_options):
        # EXTENDS 'account'
        values = super()._get_wizard_vals_restrict_to(only_options)
        return {
            'l10n_jo_edi_is_enabled': False,
            **values,
        }

    def _call_web_service_before_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_before_invoice_pdf_render(invoices_data)
        for invoice, invoice_data in invoices_data.items():
            # Not all invoices may need EDI.
            if not invoice_data.get("l10n_jo_edi_is_enabled") or not (invoice.l10n_jo_edi_is_needed and invoice.l10n_jo_edi_state != 'sent'):
                continue

            if error_message := invoice.with_company(invoice.company_id)._l10n_jo_edi_send():
                invoice_data["error"] = {
                    "error_title": _("Errors when submitting the JoFotara e-invoice:"),
                    "errors": [error_message],
                }

            if self._can_commit():
                self._cr.commit()

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
            <div name="warnings" position="inside">
                <div class="alert alert-warning" role="alert"
                    invisible="not l10n_jo_edi_is_visible or not l10n_jo_edi_is_enabled">
                    Jordanian e-invoicing (JoFotara) was enabled
                </div>
            </div>
            <xpath expr="//div[@name='advanced_options']" position="inside">
                <field name="l10n_jo_edi_is_visible" invisible="1"/>
                <div name="option_l10n_jo_edi" invisible="not l10n_jo_edi_is_visible">
                    <field name="l10n_jo_edi_is_enabled"/>
                    <b>
                        <label for="l10n_jo_edi_is_enabled"/>
                    </b>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
from . import account_move_send

```

