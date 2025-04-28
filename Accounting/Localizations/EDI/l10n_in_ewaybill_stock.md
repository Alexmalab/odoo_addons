# Odoo Module: l10n_in_ewaybill_stock

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": """Indian - E-waybill Stock""",
    "version": "1.0",
    'countries': ['in'],
    "category": "Accounting/Localizations/EDI",
    "depends": [
        "l10n_in_stock",
        "l10n_in_edi_ewaybill",
    ],
    "description": """
Indian E-waybill for Stock
==========================

This module enables users to create E-waybill from Inventory App without generating an invoice
    """,
    "data": [
        'security/ir_rules.xml',
        "security/ir.model.access.csv",
        "data/ewaybill_type_data.xml",
        "views/l10n_in_ewaybill_views.xml",
        "views/stock_picking_views.xml",
        "report/ewaybill_report_views.xml",
        "report/ewaybill_report.xml",
        "wizard/l10n_in_ewaybill_cancel_views.xml",
    ],
    'installable': True,
    'auto_install': True,
    "license": "LGPL-3",
}

```

## File: data\ewaybill_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="type_delivery_challan_sub_job_work" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">Job Work</field>
        <field name="sub_type_code">4</field>
        <field name="allowed_supply_type">out</field>
    </record>

    <record id="type_delivery_challan_sub_skd_ckd_work" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">SKD/CKD/Lots</field>
        <field name="sub_type_code">9</field>
        <field name="allowed_supply_type">both</field>
    </record>

    <record id="type_delivery_challan_sub_recipient_not_known" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">Recipient not known</field>
        <field name="sub_type_code">11</field>
        <field name="allowed_supply_type">out</field>
    </record>

    <record id="type_delivery_challan_sub_for_own_use" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">For Own Use</field>
        <field name="sub_type_code">5</field>
        <field name="allowed_supply_type">both</field>
    </record>

    <record id="type_delivery_challan_sub_exhibition_or_fairs" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">Exhibition or fairs</field>
        <field name="sub_type_code">12</field>
        <field name="allowed_supply_type">both</field>
    </record>

    <record id="type_delivery_challan_sub_line_sales" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">Line Sales</field>
        <field name="sub_type_code">10</field>
        <field name="allowed_supply_type">out</field>
    </record>

    <record id="type_delivery_challan_sub_others" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">Others</field>
        <field name="sub_type_code">8</field>
        <field name="allowed_supply_type">both</field>
    </record>

    <record id="type_delivery_challan_sub_job_work_returns" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">Job Work Returns</field>
        <field name="sub_type_code">6</field>
        <field name="allowed_supply_type">in</field>
    </record>

    <record id="type_delivery_challan_sub_sales_return" model="l10n.in.ewaybill.type">
        <field name="name">Delivery Challan</field>
        <field name="code">CHL</field>
        <field name="sub_type">Sales Return</field>
        <field name="sub_type_code">7</field>
        <field name="allowed_supply_type">in</field>
    </record>
</odoo>

```

## File: models\l10n_in_ewaybill.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import logging
import pytz
import re
from datetime import datetime
from collections import defaultdict
from psycopg2 import OperationalError

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.addons.l10n_in_ewaybill_stock.tools.ewaybill_api import EWayBillApi, EWayBillError


_logger = logging.getLogger(__name__)


class Ewaybill(models.Model):
    _name = "l10n.in.ewaybill"
    _description = "e-Waybill"
    _inherit = ['portal.mixin', 'mail.thread', 'mail.activity.mixin']
    _check_company_auto = True

    # Ewaybill details generated from the API
    name = fields.Char("e-Waybill Number", copy=False, readonly=True, tracking=True)
    ewaybill_date = fields.Date("e-Waybill Date", copy=False, readonly=True, tracking=True)
    ewaybill_expiry_date = fields.Date("e-Waybill Valid Upto", copy=False, readonly=True, tracking=True)

    state = fields.Selection(string='Status', selection=[
        ('pending', 'Pending'),
        ('challan', 'Challan'),
        ('generated', 'Generated'),
        ('cancel', 'Cancelled'),
    ], required=True, readonly=True, copy=False, tracking=True, default='pending')

    # Stock picking details
    picking_id = fields.Many2one("stock.picking", "Stock Transfer", copy=False)
    move_ids = fields.One2many(related="picking_id.move_ids")
    picking_type_code = fields.Selection(related='picking_id.picking_type_id.code')

    # Document details
    document_date = fields.Datetime("Document Date", related="picking_id.date_done")
    document_number = fields.Char("Document", related="picking_id.name")
    company_id = fields.Many2one("res.company", related="picking_id.company_id")
    company_currency_id = fields.Many2one(related="company_id.currency_id")
    supply_type = fields.Selection(string="Supply Type", selection=[
        ("O", "Outward"),
        ("I", "Inward")
    ], compute="_compute_supply_type")
    partner_bill_from_id = fields.Many2one(
        "res.partner",
        string='Bill From',
        compute="_compute_document_partners_details",
        check_company=True,
        store=True,
        readonly=False
    )
    partner_bill_to_id = fields.Many2one(
        "res.partner",
        string='Bill To',
        compute="_compute_document_partners_details",
        check_company=True,
        store=True,
        readonly=False
    )
    partner_ship_from_id = fields.Many2one(
        "res.partner",
        string='Dispatch From',
        compute="_compute_document_partners_details",
        check_company=True,
        store=True,
        readonly=False
    )
    partner_ship_to_id = fields.Many2one(
        'res.partner',
        string='Ship To',
        compute='_compute_document_partners_details',
        check_company=True,
        store=True,
        readonly=False
    )

    # Fields to determine which partner details are editable
    is_bill_to_editable = fields.Boolean(compute="_compute_is_editable")
    is_bill_from_editable = fields.Boolean(compute="_compute_is_editable")
    is_ship_to_editable = fields.Boolean(compute="_compute_is_editable")
    is_ship_from_editable = fields.Boolean(compute="_compute_is_editable")

    fiscal_position_id = fields.Many2one(
        comodel_name="account.fiscal.position",
        string="Fiscal Position",
        compute="_compute_fiscal_position",
        check_company=True,
        store=True,
        readonly=False
    )

    # E-waybill Document Type
    type_id = fields.Many2one("l10n.in.ewaybill.type", "Document Type", tracking=True, required=True)
    sub_type_code = fields.Char(related="type_id.sub_type_code")
    type_description = fields.Char(string="Description")

    # Transportation details
    distance = fields.Integer("Distance", tracking=True)
    mode = fields.Selection([
        ("1", "By Road"),
        ("2", "Rail"),
        ("3", "Air"),
        ("4", "Ship or Ship Cum Road/Rail")
    ], string="Transportation Mode", copy=False, tracking=True, default="1")

    # Vehicle Number and Type required when transportation mode is By Road.
    vehicle_no = fields.Char("Vehicle Number", copy=False, tracking=True)
    vehicle_type = fields.Selection([
        ("R", "Regular"),
        ("O", "Over Dimensional Cargo")],
        string="Vehicle Type",
        compute="_compute_vehicle_type",
        store=True,
        copy=False,
        tracking=True,
        readonly=False
    )

    # Document number and date required in case of transportation mode is Rail, Air or Ship.
    transportation_doc_no = fields.Char(
        string="Transporter Doc No",
        copy=False, tracking=True)
    transportation_doc_date = fields.Date(
        string="Transporter Doc Date",
        copy=False,
        tracking=True)

    transporter_id = fields.Many2one("res.partner", "Transporter", copy=False, tracking=True)

    error_message = fields.Html(readonly=True)
    blocking_level = fields.Selection([
        ("warning", "Warning"),
        ("error", "Error")],
        string="Blocking Level", readonly=True)

    content = fields.Binary(compute='_compute_content', compute_sudo=True)
    cancel_reason = fields.Selection(selection=[
        ("1", "Duplicate"),
        ("2", "Data Entry Mistake"),
        ("3", "Order Cancelled"),
        ("4", "Others"),
    ], string="Cancel reason", copy=False, tracking=True)
    cancel_remarks = fields.Char("Cancel remarks", copy=False, tracking=True)

    def _compute_supply_type(self):
        for ewaybill in self:
            ewaybill.supply_type = ewaybill.picking_type_code == 'incoming' and 'I' or 'O'

    @api.depends('picking_id')
    def _compute_document_partners_details(self):
        for ewaybill in self.filtered(lambda ewb: ewb.state == 'pending'):
            picking_id = ewaybill.picking_id
            if ewaybill.picking_type_code == 'incoming':
                ewaybill.partner_bill_to_id = picking_id.company_id.partner_id
                ewaybill.partner_bill_from_id = picking_id.partner_id
                ewaybill.partner_ship_to_id = picking_id.picking_type_id.warehouse_id.partner_id
                ewaybill.partner_ship_from_id = picking_id.partner_id
            else:
                ewaybill.partner_bill_to_id = picking_id.partner_id
                ewaybill.partner_bill_from_id = picking_id.company_id.partner_id
                ewaybill.partner_ship_to_id = picking_id.partner_id
                ewaybill.partner_ship_from_id = picking_id.picking_type_id.warehouse_id.partner_id
                if partner_invoice_id := ewaybill.picking_id._l10n_in_get_invoice_partner():
                    ewaybill.partner_bill_to_id = partner_invoice_id

            if (
                ewaybill.picking_type_code == 'dropship' and
                (dest_partner := ewaybill.picking_id._get_l10n_in_dropship_dest_partner())
            ):
                ewaybill.partner_ship_to_id = dest_partner
                ewaybill.partner_ship_from_id = ewaybill.picking_id.partner_id

    @api.depends('partner_bill_from_id', 'partner_bill_to_id')
    def _compute_fiscal_position(self):
        for ewaybill in self.filtered(lambda ewb: ewb.state == 'pending'):
            ewaybill.fiscal_position_id = (
                self.env['account.fiscal.position'].with_company(ewaybill.company_id)._get_fiscal_position(
                    ewaybill.picking_type_code == 'incoming'
                    and ewaybill.partner_bill_from_id
                    or ewaybill.partner_bill_to_id
                )
                or ewaybill.picking_id._l10n_in_get_fiscal_position()
            )

    @api.depends('partner_ship_from_id', 'partner_ship_to_id', 'partner_bill_from_id', 'partner_bill_to_id')
    def _compute_is_editable(self):
        for ewaybill in self:
            is_incoming = ewaybill.picking_type_code == "incoming"
            ewaybill.is_bill_to_editable = not is_incoming
            ewaybill.is_bill_from_editable = is_incoming
            ewaybill.is_ship_from_editable = is_incoming and ewaybill._is_overseas()
            ewaybill.is_ship_to_editable = not is_incoming and not ewaybill._is_overseas()

    def _compute_content(self):
        for ewaybill in self:
            ewaybill.content = base64.b64encode(json.dumps(ewaybill._ewaybill_generate_direct_json()).encode())

    @api.depends('name', 'state')
    def _compute_display_name(self):
        for ewaybill in self:
            ewaybill.display_name = (
                (ewaybill.state == 'pending' and _('Pending'))
                or (ewaybill.state == 'challan' and _('Challan'))
                or ewaybill.name
            )

    @api.depends('mode')
    def _compute_vehicle_type(self):
        """when transportation mode is ship then vehicle type should be Over Dimensional Cargo (ODC)"""
        for ewaybill in self.filtered(lambda ewb: ewb.state == 'pending' and ewb.mode == "4"):
            ewaybill.vehicle_type = 'O'

    def action_export_json(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'url': '/web/content/l10n.in.ewaybill/%s/content' % self.id
        }

    def generate_ewaybill(self):
        for ewaybill in self:
            if errors := ewaybill._check_configuration():
                raise UserError('\n'.join(errors))
            ewaybill._generate_ewaybill_direct()

    def cancel_ewaybill(self):
        self.ensure_one()
        return {
            'name': _('Cancel Ewaybill'),
            'res_model': 'l10n.in.ewaybill.cancel',
            'view_mode': 'form',
            'context': {
                'default_l10n_in_ewaybill_id': self.id,
            },
            'target': 'new',
            'type': 'ir.actions.act_window',
        }

    def reset_to_pending(self):
        self.ensure_one()
        if self.state not in ('cancel', 'challan'):
            raise UserError(_("Only Delivery Challan and Cancelled E-waybill can be reset to pending."))
        self.write({
            'name': False,
            'state': 'pending',
            'cancel_reason': False,
            'cancel_remarks': False,
        })

    def action_set_to_challan(self):
        self.ensure_one()
        if self.state != 'pending':
            raise UserError(_("The challan can only be generated in the Pending state."))
        self.write({
            'state': 'challan',
        })

    def _is_overseas(self):
        self.ensure_one()
        return self._get_gst_treatment()[1] in ('overseas', 'special_economic_zone')

    def _check_configuration(self):
        error_message = []
        methods_to_check = [
            self._check_partners,
            self._check_document_number,
            self._check_lines,
            self._check_gst_treatment,
            self._check_transporter,
        ]
        for get_error_message in methods_to_check:
            error_message.extend(get_error_message())
        return error_message

    def _check_transporter(self):
        error_message = []
        if self.transporter_id and not self.transporter_id.vat:
            error_message.append(_("- Transporter %s does not have a GST Number", self.transporter_id.name))
        if self.mode == "4" and self.vehicle_no and self.vehicle_type == "R":
            error_message.append(_("- Vehicle type can not be regular when the transportation mode is ship"))
        return error_message

    def _check_partners(self):
        error_message = []
        partners = {
            self.partner_bill_to_id, self.partner_bill_from_id, self.partner_ship_to_id, self.partner_ship_from_id
        }
        for partner in partners:
            error_message += self._l10n_in_validate_partner(partner)
        return error_message

    @api.model
    def _l10n_in_validate_partner(self, partner):
        """
        Validation method for Stock Ewaybill (different from the one in EDI Ewaybill)
        """
        message = []
        if partner.country_id.code == "IN":
            if partner.state_id and not partner.state_id.l10n_in_tin:
                message.append(_("- TIN number not set in state %s", partner.state_id.name))
            if not partner.state_id:
                message.append(_("- State is required"))
            if not partner.zip or not re.match("^[0-9]{6}$", partner.zip):
                message.append(_("- Zip code required and should be 6 digits"))
        elif not partner.country_id:
            message.append(_("- Country is required"))
        if message:
            message.insert(0, "%s" % partner.display_name)
        return message

    def _check_document_number(self):
        if not re.match("^.{1,16}$", self.document_number):
            return [_("Document number should be set and not more than 16 characters")]
        return []

    def _check_lines(self):
        error_message = []
        AccountEDI = self.env['account.edi.format']
        for line in self.move_ids:
            if not (hsn_code := AccountEDI._l10n_in_edi_extract_digits(line.product_id.l10n_in_hsn_code)):
                error_message.append(_("HSN code is not set in product %s", line.product_id.name))
            elif not re.match("^[0-9]+$", hsn_code):
                error_message.append(_(
                    "Invalid HSN Code (%s) in product %s", hsn_code, line.product_id.name
                ))
        return error_message

    def _check_gst_treatment(self):
        partner, gst_treatment = self._get_gst_treatment()
        if not gst_treatment:
            return [_("Set GST Treatment for in %s", partner.display_name)]
        return []

    def _get_gst_treatment(self):
        if self.picking_type_code == 'incoming':
            partner = self.partner_bill_from_id
        else:
            partner = self.partner_bill_to_id
        return partner, partner.l10n_in_gst_treatment

    def _write_error(self, error_message, blocking_level='error'):
        self.write({
            'error_message': error_message,
            'blocking_level': blocking_level,
        })

    def _write_successfully_response(self, response_vals):
        response_vals.update({
            'error_message': False,
            'blocking_level': False,
        })
        self.write(response_vals)

    def _lock_ewaybill(self):
        try:
            # Lock e-Waybill
            with self.env.cr.savepoint(flush=False):
                self._cr.execute('SELECT * FROM l10n_in_ewaybill WHERE id IN %s FOR UPDATE NOWAIT', [tuple(self.ids)])
        except OperationalError as e:
            if e.pgcode == '55P03':
                raise UserError(_('This document is being sent by another process already.'))
            else:
                raise

    def _handle_internal_warning_if_present(self, response):
        if warnings := response.get('odoo_warning'):
            for warning in warnings:
                if warning.get('message_post'):
                    odoobot = self.env.ref("base.partner_root")
                    self.message_post(
                        author_id=odoobot.id,
                        body=warning.get('message')
                    )
                else:
                    self._write_error(warning.get('message'))

    def _handle_error(self, ewaybill_error):
        self._handle_internal_warning_if_present(ewaybill_error.error_json)
        error_message = ewaybill_error.get_all_error_message()
        blocking_level = "error"
        if "404" in ewaybill_error.error_codes:
            blocking_level = "warning"
        self._write_error(error_message, blocking_level)

    def _ewaybill_cancel(self):
        cancel_json = {
            "ewbNo": int(self.name),
            "cancelRsnCode": int(self.cancel_reason),
            "CnlRem": self.cancel_remarks,
        }
        ewb_api = EWayBillApi(self.company_id)
        self._lock_ewaybill()
        try:
            ewb_api._ewaybill_cancel(cancel_json)
        except EWayBillError as error:
            self._handle_error(error)
            return False
        self._write_successfully_response({'state': 'cancel'})
        self._cr.commit()

    def _l10n_in_ewaybill_stock_handle_zero_distance_alert_if_present(self, response):
        if self.distance == 0 and (alert := response.get('data').get('alert')):
            pattern = r"Distance between these two pincodes is (\d+)"
            if (match := re.search(pattern, alert)) and (dist := int(match.group(1))) > 0:
                self.distance = dist

    def _generate_ewaybill_direct(self):
        ewb_api = EWayBillApi(self.company_id)
        generate_json = self._ewaybill_generate_direct_json()
        self._lock_ewaybill()
        try:
            response = ewb_api._ewaybill_generate(generate_json)
        except EWayBillError as error:
            self._handle_error(error)
            return False
        self._handle_internal_warning_if_present(response)  # In case of error 604
        response_data = response.get("data")
        response_values = {
            'name': response_data.get("ewayBillNo"),
            'state': 'generated',
            'ewaybill_date': self._indian_timezone_to_odoo_utc(
                response_data['ewayBillDate']
            ),
            'ewaybill_expiry_date': self._indian_timezone_to_odoo_utc(
                response_data.get('validUpto')
            ),
        }
        self._l10n_in_ewaybill_stock_handle_zero_distance_alert_if_present(response)
        self._write_successfully_response(response_values)
        self._cr.commit()

    @api.model
    def _indian_timezone_to_odoo_utc(self, str_date, time_format='%d/%m/%Y %I:%M:%S %p'):
        """
            This method is used to convert date from Indian timezone to UTC
        """
        if not str_date:
            return False
        try:
            local_time = datetime.strptime(str_date, time_format)
        except ValueError:
            try:
                # Misc. due to a bug in eWaybill sometimes there are chances of getting below format in response
                local_time = datetime.strptime(str_date, "%d/%m/%Y %H:%M:%S ")
            except ValueError:
                # Worst senario no date format matched
                _logger.warning("Something went wrong while L10nInEwaybill date conversion")
                return fields.Datetime.to_string(fields.Datetime.now())
        utc_time = local_time.astimezone(pytz.utc)
        return fields.Datetime.to_string(utc_time)

    @api.model
    def _get_partner_state_code(self, partner):
        return int(partner.state_id.l10n_in_tin) if partner.country_id.code == "IN" else 99

    def _l10n_in_tax_details(self):
        tax_details = {
            'line_tax_details': defaultdict(dict),
            'tax_details': defaultdict(float)
        }
        for move in self.move_ids:
            line_tax_vals = self._l10n_in_tax_details_by_line(move)
            tax_details['line_tax_details'][move.id] = line_tax_vals
            for val_field in ['total_excluded', 'total_included', 'total_void']:
                tax_details['tax_details'][val_field] += line_tax_vals[val_field]
            for tax in ['igst', 'cgst', 'sgst', 'cess_non_advol', 'cess', 'other']:
                for taxes in line_tax_vals['taxes']:
                    for field_key in ["rate", "amount"]:
                        if (key := f"{tax}_{field_key}") in taxes:
                            tax_details['tax_details'][key] += taxes[key]
        return tax_details

    def _l10n_in_tax_details_by_line(self, move):
        taxes = move.ewaybill_tax_ids.compute_all(price_unit=move.ewaybill_price_unit, quantity=move.quantity)
        for tax in taxes['taxes']:
            tax_id = self.env['account.tax'].browse(tax['id'])
            tax_name = "other"
            for gst_tax_name in ['igst', 'sgst', 'cgst']:
                if self.env.ref("l10n_in.tax_tag_%s" % (gst_tax_name)).id in tax['tag_ids']:
                    tax_name = gst_tax_name
            if self.env.ref("l10n_in.tax_tag_cess").id in tax['tag_ids']:
                tax_name = tax_id.amount_type != "percent" and "cess_non_advol" or "cess"
            rate_key = "%s_rate" % tax_name
            amount_key = "%s_amount" % tax_name
            tax.setdefault(rate_key, 0)
            tax.setdefault(amount_key, 0)
            tax[rate_key] += tax_id.amount
            tax[amount_key] += tax['amount']
        return taxes

    def _get_l10n_in_ewaybill_line_details(self, line, tax_details):
        AccountEDI = self.env['account.edi.format']
        product = line.product_id
        line_details = {
            "productName": product.name,
            "hsnCode": AccountEDI._l10n_in_edi_extract_digits(product.l10n_in_hsn_code),
            "productDesc": product.name,
            "quantity": line.quantity,
            "qtyUnit": line.product_uom.l10n_in_code and line.product_uom.l10n_in_code.split("-")[
                0] or "OTH",
            "taxableAmount": AccountEDI._l10n_in_round_value(tax_details['total_excluded']),
        }
        gst_types = ('sgst', 'cgst', 'igst')
        gst_tax_rates = {}
        for tax in tax_details.get('taxes'):
            for gst_type in gst_types:
                if tax_rate := tax.get(f'{gst_type}_rate'):
                    gst_tax_rates.update({
                        f"{gst_type}Rate": AccountEDI._l10n_in_round_value(tax_rate)
                    })
            if cess_rate := tax.get("cess_rate"):
                line_details.update({"cessRate": AccountEDI._l10n_in_round_value(cess_rate)})
            if cess_non_advol := tax.get("cess_non_advol_amount"):
                line_details.update({
                    "cessNonadvol": AccountEDI._l10n_in_round_value(cess_non_advol)
                })
        line_details.update(
            gst_tax_rates
            or dict.fromkeys(
                [f"{gst_type}Rate" for gst_type in gst_types],
                0
            )
        )
        return line_details

    def _prepare_ewaybill_base_json_payload(self):

        def get_transaction_type(seller_details, dispatch_details, buyer_details, ship_to_details):
            """
                1 - Regular
                2 - Bill To - Ship To
                3 - Bill From - Dispatch From
                4 - Combination of 2 and 3
            """
            if seller_details != dispatch_details and buyer_details != ship_to_details:
                return 4
            elif seller_details != dispatch_details:
                return 3
            elif buyer_details != ship_to_details:
                return 2
            else:
                return 1

        def prepare_details(key_paired_function, partner_detail):
            return {
                f"{place}{key}": fun(partner)
                for key, fun in key_paired_function
                for place, partner in partner_detail
            }
        ewaybill_json = {
                # document details
                "supplyType": self.supply_type,
                "subSupplyType": self.type_id.sub_type_code,
                "docType": self.type_id.code,
                "transactionType": get_transaction_type(
                    self.partner_bill_from_id,
                    self.partner_ship_from_id,
                    self.partner_bill_to_id,
                    self.partner_ship_to_id
                ),
                "transDistance": str(self.distance),
                "docNo": self.document_number,
                "docDate": (self.document_date or fields.Datetime.now()).strftime("%d/%m/%Y"),
                # bill details
                **prepare_details(
                    key_paired_function={
                        'Gstin': lambda p: p.commercial_partner_id.vat or "URP",
                        'TrdName': lambda p: p.commercial_partner_id.name,
                        'StateCode': self._get_partner_state_code,
                    }.items(),
                    partner_detail={'from': self.partner_bill_from_id, 'to': self.partner_bill_to_id}.items()
                ),
                # shipping details
                **prepare_details(
                    key_paired_function={
                        "Addr1": lambda p: p.street and p.street[:120] or "",
                        "Addr2": lambda p: p.street2 and p.street2[:120] or "",
                        "Place": lambda p: p.city and p.city[:50] or "",
                        "Pincode": lambda p: int(p.zip) if p.country_id.code == "IN" else 999999,
                    }.items(),
                    partner_detail={'from': self.partner_ship_from_id, 'to': self.partner_ship_to_id}.items()
                ),
                "actToStateCode": self._get_partner_state_code(self.partner_ship_to_id),
                "actFromStateCode": self._get_partner_state_code(self.partner_ship_from_id),
        }
        if self.type_id.sub_type_code == '8':
            ewaybill_json["subSupplyDesc"] = self.type_description
        return ewaybill_json

    def _prepare_ewaybill_transportation_json_payload(self):
        # only pass transporter details when value is exist
        return dict(
            filter(lambda kv: kv[1], {
                "transporterId": self.transporter_id.vat,
                "transporterName": self.transporter_id.name,
                "transMode": self.mode,
                "transDocNo": self.transportation_doc_no,
                "transDocDate": self.transportation_doc_date and self.transportation_doc_date.strftime("%d/%m/%Y"),
                "vehicleNo": self.vehicle_no,
                "vehicleType": self.vehicle_type,
            }.items())
        )

    def _prepare_ewaybill_tax_details_json_payload(self):
        tax_details = self._l10n_in_tax_details()
        round_value = self.env['account.edi.format']._l10n_in_round_value
        return {
            "itemList": [
                self._get_l10n_in_ewaybill_line_details(line, tax_details['line_tax_details'][line.id])
                for line in self.move_ids
            ],
            "totalValue": round_value(tax_details['tax_details'].get('total_excluded', 0.00)),
            **{
                f'{tax_type}Value': round_value(tax_details.get('tax_details').get(f'{tax_type}_amount', 0.00))
                for tax_type in ['cgst', 'sgst', 'igst', 'cess']
            },
            "cessNonAdvolValue": round_value(tax_details.get('cess_non_advol_amount', 0.00)),
            "otherValue": round_value(tax_details.get('other_amount', 0.00)),
            "totInvValue": round_value(tax_details['tax_details'].get('total_included', 0.00)),
        }

    def _ewaybill_generate_direct_json(self):
        return {
            **self._prepare_ewaybill_base_json_payload(),
            **self._prepare_ewaybill_transportation_json_payload(),
            **self._prepare_ewaybill_tax_details_json_payload(),
        }

    @api.ondelete(at_uninstall=False)
    def _unlink_l10n_in_ewaybill_prevent(self):
        if self.filtered(lambda ewaybill: ewaybill.state != 'pending'):
            raise UserError(_("You cannot delete a generated E-waybill. Instead, you should cancel it."))

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
import logging

_logger = logging.getLogger(__name__)


class StockMove(models.Model):
    _inherit = "stock.move"
    _description = "Stock Move Ewaybill"

    l10n_in_ewaybill_id = fields.One2many(related="picking_id.l10n_in_ewaybill_id")
    company_currency_id = fields.Many2one(related='company_id.currency_id')

    # Need to store values because we send it to the ewaybill and we need to keep the same value
    ewaybill_price_unit = fields.Monetary(
        compute='_compute_l10n_in_ewaybill_price_unit',
        currency_field='company_currency_id',
        store=True,
        readonly=False
    )
    ewaybill_tax_ids = fields.Many2many(
        comodel_name='account.tax',
        string="Taxes",
        compute='_compute_l10n_in_tax_ids',
        store=True,
        readonly=False
    )

    @api.depends('l10n_in_ewaybill_id')
    def _compute_l10n_in_ewaybill_price_unit(self):
        for line in self:
            if line.l10n_in_ewaybill_id.state == 'pending' and line.picking_id.country_code == 'IN':
                line.ewaybill_price_unit = line._l10n_in_get_product_price_unit()

    @api.depends('l10n_in_ewaybill_id.fiscal_position_id')
    def _compute_l10n_in_tax_ids(self):
        for line in self:
            if line.l10n_in_ewaybill_id.state == 'pending' and line.picking_id.country_code == 'IN':
                taxes_details = line._l10n_in_get_product_tax()
                taxes = taxes_details['taxes']
                if taxes_details['is_from_order']:
                    # Don't map taxes if they are from sale/purchase order
                    line.ewaybill_tax_ids = taxes
                else:
                    if fiscal_position := line.l10n_in_ewaybill_id.fiscal_position_id:
                        taxes = fiscal_position.map_tax(taxes)
                    line.ewaybill_tax_ids = taxes.filtered_domain(
                        self.env['account.tax']._check_company_domain(self.company_id)
                    )

```

## File: models\stock_picking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.exceptions import UserError


class StockPicking(models.Model):
    _inherit = "stock.picking"

    l10n_in_ewaybill_id = fields.One2many('l10n.in.ewaybill', 'picking_id', string='Ewaybill')

    def _get_l10n_in_ewaybill_form_action(self):
        return self.env.ref('l10n_in_ewaybill_stock.l10n_in_ewaybill_form_action')._get_action_dict()

    def action_l10n_in_ewaybill_create(self):
        self.ensure_one()
        if (
            product_with_no_hsn := self.move_ids.mapped('product_id').filtered(
                lambda product: not product.l10n_in_hsn_code
            )
        ):
            raise UserError(_("Please set HSN code in below products: \n%s", '\n'.join(
                [product.name for product in product_with_no_hsn]
            )))
        if self.l10n_in_ewaybill_id:
            raise UserError(_("Ewaybill already created for this picking."))
        action = self._get_l10n_in_ewaybill_form_action()
        type_xml_trailing_id = (
            'type_delivery_challan_sub_sales_return'
            if self.picking_type_code == 'incoming'
            else 'type_delivery_challan_sub_others'
        )
        ewaybill = self.env['l10n.in.ewaybill'].create({
            'picking_id': self.id,
            'type_id': self.env.ref(f'l10n_in_ewaybill_stock.{type_xml_trailing_id}').id,
        })
        action['res_id'] = ewaybill.id
        return action

    def action_open_l10n_in_ewaybill(self):
        self.ensure_one()
        action = self._get_l10n_in_ewaybill_form_action()
        action['res_id'] = self.l10n_in_ewaybill_id.id
        return action

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import l10n_in_ewaybill
from . import stock_picking
from . import stock_move

```

## File: report\ewaybill_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="report_ewaybill">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="doc">
                    <t t-call="web.internal_layout">
                        <div class="page">
                            <style>
                                table {
                                    width: 100%;
                                }
                                th, td {
                                  padding: 5px;
                                  font-size: 14px;
                                }
                            </style>
                            <t t-set="challan_states" t-value="('challan', 'pending')"/>
                            <table class="border-bottom border-dark">
                                <tr class="d-flex justify-content-between align-items-center px-4 py-3">
                                    <td style="width: 8rem;">
                                        <img t-if="doc.company_id.logo" t-att-src="image_data_uri(doc.company_id.logo)"
                                             style="max-width: 8rem; height: auto;" alt="Company Logo"/>
                                    </td>
                                    <td>
                                        <h4 t-if="doc.state in challan_states and doc.picking_id" style="margin-top: 1rem;">Delivery Challan</h4>
                                        <h4 t-else="" style="margin-top: 1rem;">e-Way Bill</h4>
                                    </td>
                                    <t t-if="doc.state not in challan_states">
                                        <td>
                                            <img t-if="doc.name != False"
                                                 t-att-src="'/report/barcode/QR/' + str(doc.name) + '/' + str(doc.company_id.vat) + '/' + str(doc.ewaybill_date)"
                                                 style="width: 6.25rem; height: 6.25rem;" alt="Barcode"/>
                                        </td>
                                    </t>
                                </tr>
                            </table>
                            <br/>

                            <t t-set="generate_json" t-value="doc._ewaybill_generate_direct_json()"/>
                            <t t-if="doc.state not in challan_states">
                                <h5>E-WAY BILL Details</h5>
                                <table class="text-nowrap">
                                    <tr>
                                        <td>eway Bill No: <t t-out="doc.name"/></td>
                                        <td>Generated Date: <t t-out="doc.ewaybill_date"/></td>
                                        <td>Generated By: <t t-out="doc.company_id.vat"/></td>
                                        <td><t t-if="doc.name != False"></t>
                                            Valid Upto: <t t-out="doc.ewaybill_expiry_date"/>
                                            <t t-if="doc.mode == '1'">
                                                (Vehicle Type is <t t-out="dict(doc._fields['vehicle_type'].selection).get(doc.vehicle_type)"/>)
                                            </t>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td>
                                            Mode:<t t-out="dict(doc._fields['mode']._description_selection(doc.env)).get(doc.mode, '')"/>
                                        </td>
                                        <td colspan="3">
                                            Approx Distance: <t t-out="doc.distance"/> km
                                        </td>
                                    </tr>
                                    <tr>
                                        <td style="white-space: normal; word-break: break-all;">
                                            Type: <t t-out="doc.supply_type"/> - <t t-out="doc.type_id.sub_type"/>-<t t-out="doc.type_description"/>
                                        </td>
                                        <td colspan="2">
                                            Document Details:<t t-out="doc.type_id.name"/> - <t t-out="doc.picking_id.name"/> - <t t-out="doc.ewaybill_date"/>
                                        </td>
                                        <td>Transaction Type:
                                            <t t-if="generate_json['transactionType'] == 2">
                                                Bill To - Ship To
                                            </t>
                                            <t t-elif="generate_json['transactionType'] == 3">
                                                Bill From - Dispatch From
                                            </t>
                                            <t t-elif="generate_json['transactionType'] == 4">
                                                Combination of 2 and 3
                                            </t>
                                            <t t-else="">
                                                Regular
                                            </t>
                                        </td>
                                    </tr>
                                </table>
                                <div class="border-bottom border-dark mt-3"/>
                                <br/>
                            </t>
                            <t t-else="">
                                <h5>Document Details</h5>
                                <table>
                                    <tr>
                                        <td>Document No.: <b><t t-out="doc.picking_id.name"/></b></td>
                                        <td>Generated Date: <t t-out="doc.picking_id.scheduled_date"/></td>
                                    </tr>
                                </table>
                                <div class="border-bottom border-dark mt-3"/>
                                <br/>
                            </t>

                            <h5>Address Details</h5>
                            <div>
                                <table>
                                    <tr>
                                        <td>From</td>
                                        <td/>
                                        <td>To</td>
                                    </tr>
                                    <tr>
                                        <td class="border border-dark">
                                            GSTIN: <t t-out="generate_json['fromGstin']"/><br/>
                                            <t t-out="doc.partner_bill_from_id.name"/><br/>
                                            <t t-out="doc.partner_bill_from_id.state_id.name"/><br/>
                                            <br/>
                                            ::Dispatch From:: <br/><br/>
                                            <span t-field="doc.partner_ship_from_id" t-options='{"widget": "contact", "fields": ["address"], "no_marker": True}'/>
                                        </td>
                                        <td/>
                                        <td class="border border-dark">
                                            GSTIN: <t t-out="generate_json['toGstin']"/><br/>
                                            <t t-out="doc.partner_bill_to_id.name"/><br/>
                                            <t t-out="doc.partner_bill_to_id.state_id.name"/><br/>
                                            <br/>
                                            ::Ship To:: <br/><br/>
                                            <span t-field="doc.partner_ship_to_id" t-options='{"widget": "contact", "fields": ["address"], "no_marker": True}'/>
                                        </td>
                                    </tr>
                                </table>
                            </div>
                            <div class="border-bottom border-dark mt-3"/>
                            <br/>
                            <h5>Goods Details</h5>
                            <table class="border border-dark">
                                <thead>
                                    <tr class="border-bottom border-dark">
                                        <th>HSN Code</th>
                                        <th>Product Description</th>
                                        <th>Quantity</th>
                                        <th>Taxable Amount Rs.</th>
                                        <th>Tax Rate (C+S+I+Cess+Cess Non Advol)</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <t t-set="items" t-value="generate_json['itemList']"/>
                                    <tr t-foreach="items" t-as="item" class="border-bottom border-dark">
                                        <td>
                                            <t t-out="item['hsnCode']"/>
                                        </td>
                                        <td>
                                            <t t-out="item['productDesc']"/>
                                        </td>
                                        <td>
                                            <t t-out="item['quantity']"/> <t t-out="item['qtyUnit']"/>
                                        </td>
                                        <td>
                                            <t t-out="item['taxableAmount']"/>
                                        </td>
                                        <td>
                                            <t t-out="
                                                ('%.2f' % item['cgstRate'] if 'cgstRate' in item else 'NE') + ' + ' +
                                                ('%.2f' % item['sgstRate'] if 'sgstRate' in item else 'NE') + ' + ' +
                                                ('%.2f' % item['igstRate'] if 'igstRate' in item else 'NE') + ' + ' +
                                                ('%.2f' % item['cessRate'] if 'cessRate' in item else 'NE') + ' + ' +
                                                (str(round(item['cessNonadvol'], 2)) if 'cessNonadvol' in item else 'NE')
                                            "/>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                            <br/>

                            <div class="border-bottom border-dark">
                                <table>
                                    <tr>
                                        <td>
                                            Total Taxable Amount <strong><t t-out="generate_json['totalValue']" t-options='{"widget": "monetary", "display_currency": doc.company_currency_id}'/></strong>
                                        </td>
                                        <td>
                                            CGST Amount <strong><t t-out="generate_json['cgstValue']" t-options='{"widget": "monetary", "display_currency": doc.company_currency_id}'/></strong>
                                        </td>
                                        <td>
                                            SGST Amount <strong><t t-out="generate_json['sgstValue']" t-options='{"widget": "monetary", "display_currency": doc.company_currency_id}'/></strong>
                                        </td>
                                        <td>
                                            IGST Amount <strong><t t-out="generate_json['igstValue']" t-options='{"widget": "monetary", "display_currency": doc.company_currency_id}'/></strong>
                                        </td>
                                        <td>
                                            CESS Amount <strong><t t-out="generate_json['cessValue']" t-options='{"widget": "monetary", "display_currency": doc.company_currency_id}'/></strong>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td>
                                            CESS Non.Advol Amt <strong><t t-out="generate_json['cessNonAdvolValue']" t-options='{"widget": "monetary", "display_currency": doc.company_currency_id}'/></strong>
                                        </td>
                                        <td>
                                            Other Amt <strong><t t-out="generate_json['otherValue']" t-options='{"widget": "monetary", "display_currency": doc.company_currency_id}'/></strong>
                                        </td>
                                        <td>
                                            Total Inv. Amt <strong><t t-out="generate_json['totInvValue']" t-options='{"widget": "monetary", "display_currency": doc.company_currency_id}'/></strong>
                                        </td>
                                    </tr>
                                </table>
                            </div>
                            <br/>

                            <h5>Transportation Details</h5>
                            <table class="border-bottom border-dark">
                                <tr>
                                    <td>Transporter ID and Name: <t t-out="doc.transporter_id.vat"/> &amp;
                                        <t t-out="doc.transporter_id.name"/>
                                    </td>
                                    <td>Transporter Doc and Date: <t t-out="doc.transportation_doc_no"/> &amp;
                                        <t t-out="doc.transportation_doc_date"/>
                                    </td>
                                </tr>
                            </table>
                            <br/>

                            <h5>Vehicle Details</h5>
                            <div class="border border-dark">
                                <table>
                                    <thead>
                                        <tr class="border-bottom border-dark">
                                            <th>Mode</th>
                                            <th>Vehicle / Trans <br/>Doc No &amp; Dt.</th>
                                            <th>From</th>
                                            <t t-if="doc.state not in challan_states">
                                                <th>Entered Date</th>
                                                <th>Entered By</th>
                                            </t>
                                            <t t-else=""><th>Approx Distance</th></t>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <tr>
                                            <td>
                                                <t t-out="dict(doc._fields['mode']._description_selection
                                                (doc.env)).get(doc.mode, '')"/>
                                            </td>
                                            <td>
                                                <t t-if="doc.vehicle_no" t-out="doc.vehicle_no"/>
                                                <t t-else="" t-out="doc.transportation_doc_no"/><t t-out="doc.transportation_doc_date"/>
                                            </td>
                                            <td>
                                                <t t-out="generate_json['fromPlace']"/>
                                            </td>
                                            <t t-if="doc.state not in challan_states">
                                                <td>
                                                    <t t-out="doc.ewaybill_date"/>
                                                </td>
                                                <td>
                                                    <t t-out="doc.company_id.vat"/>
                                                </td>
                                            </t>
                                            <t t-else="">
                                                <td>
                                                    <t t-out="doc.distance"/>Km
                                                </td>
                                            </t>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                            <br/>

                            <div class="border-bottom border-dark text-center">
                                <t t-if="doc.state not in challan_states" >
                                    <img t-if="doc.name != False" t-att-src="'/report/barcode/Code128/' + doc.name" style="width:250px;height:35px;" alt="Barcode"/>
                                    <br/>
                                    <t t-out="doc.name"/>
                                </t>
                            </div>
                        </div>
                    </t>
                </t>
            </t>
        </template>
    </data>
</odoo>

```

## File: report\ewaybill_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="paperformat_ewaybill" model="report.paperformat">
            <field name="name">A4 - ewaybill</field>
            <field name="default" eval="True"/>
            <field name="format">A4</field>
            <field name="page_height">0</field>
            <field name="page_width">0</field>
            <field name="orientation">Portrait</field>
            <field name="margin_top">5</field>
            <field name="margin_bottom">5</field>
            <field name="margin_left">7</field>
            <field name="margin_right">7</field>
            <field name="header_line" eval="False"/>
            <field name="header_spacing">15</field>
            <field name="dpi">90</field>
        </record>

        <record id="action_report_ewaybill" model="ir.actions.report">
            <field name="name">Ewaybill / Delivery Challan</field>
            <field name="model">l10n.in.ewaybill</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">l10n_in_ewaybill_stock.report_ewaybill</field>
            <field name="report_file">l10n_in_ewaybill_stock.report_ewaybill</field>
            <field name="print_report_name">'Ewaybill - %s' % (object.document_number)</field>
            <field name="paperformat_id" ref="l10n_in_ewaybill_stock.paperformat_ewaybill"/>
            <field name="binding_model_id" ref="model_l10n_in_ewaybill"/>
            <field name="binding_type">report</field>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_in_ewaybill_stock.access_l10n_in_ewaybill,access_l10n_in_ewaybill,l10n_in_ewaybill_stock.model_l10n_in_ewaybill,stock.group_stock_manager,1,1,1,1
l10n_in_ewaybill_stock.access_l10n_in_ewaybill_cancel,access_l10n_in_ewaybill_cancel,l10n_in_ewaybill_stock.model_l10n_in_ewaybill_cancel,stock.group_stock_manager,1,1,1,1

```

## File: security\ir_rules.xml

```xml
<?xml version='1.0' encoding='utf-8'?>

<odoo>
    <data noupdate="1">
        <record model="ir.rule" id="l10n_in_ewaybill_comp_rule">
            <field name="name">L10nIn Ewaybill multi-company</field>
            <field name="model_id" ref="model_l10n_in_ewaybill"/>
            <field name="domain_force">[('company_id', 'in', company_ids)]</field>
        </record>
    </data>
</odoo>

```

## File: tools\ewaybill_api.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import contextlib

from datetime import timedelta
from markupsafe import Markup

from odoo import fields, _
from odoo.addons.iap import jsonrpc
from odoo.exceptions import AccessError
from odoo.addons.l10n_in_edi.models.account_edi_format import DEFAULT_IAP_ENDPOINT, DEFAULT_IAP_TEST_ENDPOINT
from odoo.addons.l10n_in_edi_ewaybill.models.error_codes import ERROR_CODES


_logger = logging.getLogger(__name__)


class EWayBillError(Exception):

    def __init__(self, response):
        self.error_json = self._set_missing_error_message(response)
        self.error_json.setdefault('odoo_warning', [])
        self.error_codes = self.get_error_codes()
        super().__init__(response)

    def _set_missing_error_message(self, response):
        for error in response.get('error', []):
            if error.get('code') and not error.get('message'):
                error['message'] = self._find_missing_error_message(error.get('code'))
        return response

    @staticmethod
    def _find_missing_error_message(code):
        error_message = ERROR_CODES.get(code)
        return error_message or _("We don't know the error message for this error code. Please contact support.")

    def get_all_error_message(self):
        return Markup("<br/>").join(
            ["[%s] %s" % (e.get("code"), e.get("message")) for e in self.error_json.get('error')]
        )

    def get_error_codes(self):
        return [e.get("code") for e in self.error_json['error']]


class EWayBillApi:

    DEFAULT_HELP_MESSAGE = _(
        "Somehow this E-waybill has been %s in the government portal before. "
        "You can verify by checking the details into the government "
        "(https://ewaybillgst.gov.in/Others/EBPrintnew.asp)"
    )

    def __init__(self, company):
        company.ensure_one()
        self.company = company
        self.env = self.company.env

    def _ewaybill_jsonrpc_to_server(self, url_path, params):
        user_token = self.env["iap.account"].get("l10n_in_edi")
        params.update({
            "account_token": user_token.account_token,
            "dbuuid": self.env["ir.config_parameter"].sudo().get_param("database.uuid"),
            "username": self.company.sudo().l10n_in_edi_ewaybill_username,
            "gstin": self.company.vat,
        })
        if self.company.sudo().l10n_in_edi_production_env:
            default_endpoint = DEFAULT_IAP_ENDPOINT
        else:
            default_endpoint = DEFAULT_IAP_TEST_ENDPOINT
        endpoint = self.env["ir.config_parameter"].sudo().get_param("l10n_in_edi_ewaybill.endpoint", default_endpoint)
        url = f"{endpoint}{url_path}"
        try:
            response = jsonrpc(url, params=params, timeout=10)
            if response.get('error'):
                raise EWayBillError(response)
        except AccessError as e:
            _logger.warning("Connection error: %s", e.args[0])
            raise EWayBillError({
                "error": [{
                    "code": "access_error",
                    "message": _("Unable to connect to the E-WayBill service."
                                 "The web service may be temporary down. Please try again in a moment.")
                }]
            })
        return response

    def _ewaybill_check_authentication(self):
        sudo_company = self.company.sudo()
        if sudo_company.l10n_in_edi_ewaybill_username and sudo_company._l10n_in_edi_ewaybill_token_is_valid():
            return True
        elif sudo_company.l10n_in_edi_ewaybill_username and sudo_company.l10n_in_edi_ewaybill_password:
            try:
                self._ewaybill_authenticate()
                return True
            except EWayBillError:
                return False
        return False

    def _ewaybill_authenticate(self):
        params = {"password": self.company.sudo().l10n_in_edi_ewaybill_password}
        response = self._ewaybill_jsonrpc_to_server(url_path="/iap/l10n_in_edi_ewaybill/1/authenticate", params=params)
        if response and response.get("status_cd") == "1":
            self.company.sudo().l10n_in_edi_ewaybill_auth_validity = fields.Datetime.now() + timedelta(
                hours=6, minutes=00, seconds=00)

    def _ewaybill_make_transaction(self, operation_type, json_payload):
        """
        :params operation_type: operation_type must be strictly `generate` or `cancel`
        :params json_payload: to be sent as params
        This method handles the common errors in generating and canceling the ewaybill
        """
        try:
            if not self._ewaybill_check_authentication():
                self._raise_ewaybill_no_config_error()
            params = {"json_payload": json_payload}
            url_path = f"/iap/l10n_in_edi_ewaybill/1/{operation_type}"
            response = self._ewaybill_jsonrpc_to_server(
                url_path=url_path,
                params=params
            )
            return response
        except EWayBillError as e:
            if "no-credit" in e.error_codes:
                e.error_json['odoo_warning'].append({
                    'message': self.env['account.edi.format']._l10n_in_edi_get_iap_buy_credits_message(self.company)
                })
                raise

            if '238' in e.error_codes:
                # Invalid token eror then create new token and send generate request again.
                # This happens when authenticate called from another odoo instance with same credentials
                # (like. Demo/Test)
                with contextlib.suppress(EWayBillError):
                    self._ewaybill_authenticate()
                return self._ewaybill_jsonrpc_to_server(
                    url_path=url_path,
                    params=params,
                )

            if operation_type == "cancel" and "312" in e.error_codes:
                # E-waybill is already canceled
                # this happens when timeout from the Government portal but IRN is generated
                e.error_json['odoo_warning'].append({
                    'message': Markup("%s<br/>%s:<br/>%s") % (
                        self.DEFAULT_HELP_MESSAGE % 'cancelled',
                        _("Error"),
                        e.get_all_error_message()
                    ),
                    'message_post': True
                })
                raise

            if operation_type == "generate" and "604" in e.error_codes:
                # Get E-waybill by details in case of E-waybill is already generated
                # this happens when timeout from the Government portal but E-waybill is generated
                response = self._ewaybill_get_by_consigner(
                    document_type=json_payload.get("docType"),
                    document_number=json_payload.get("docNo")
                )
                return response
            raise

    def _ewaybill_generate(self, json_payload):
        return self._ewaybill_make_transaction("generate", json_payload)

    def _ewaybill_cancel(self, json_payload):
        return self._ewaybill_make_transaction("cancel", json_payload)

    def _ewaybill_get_by_consigner(self, document_type, document_number):
        if not self._ewaybill_check_authentication():
            self._raise_ewaybill_no_config_error()
        params = {"document_type": document_type, "document_number": document_number}
        response = self._ewaybill_jsonrpc_to_server(
            url_path="/iap/l10n_in_edi_ewaybill/1/getewaybillgeneratedbyconsigner",
            params=params
        )
        # Add warning that ewaybill was already generated
        response.update({
            'odoo_warning': [{
                'message': self.DEFAULT_HELP_MESSAGE % 'generated',
                'message_post': True
            }]
        })
        return response

    @staticmethod
    def _raise_ewaybill_no_config_error():
        raise EWayBillError({
            "error": [{
                "code": "0",
                "message": _(
                    "Unable to send E-waybill."
                    "Create an API user in NIC portal, and set it using the top menu: Configuration > Settings."
                )
            }]
        })

```

## File: tools\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ewaybill_api

```

## File: views\l10n_in_ewaybill_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_in_ewaybill_form_view" model="ir.ui.view">
        <field name="name">l10n.in.ewaybill.form.view</field>
        <field name="model">l10n.in.ewaybill</field>
        <field name="arch" type="xml">
            <form string="Ewaybill" create="false">
                <header>
                    <button name="generate_ewaybill" string="Generate e-Waybill" class="oe_highlight" type="object" invisible="state != 'pending'" data-hotkey="g"/>
                    <button name="cancel_ewaybill" string="Cancel e-Waybill" type="object" invisible="state != 'generated'" data-hotkey="c"/>
                    <button name="reset_to_pending" string="Reset to Pending" type="object" invisible="not state in ['cancel', 'challan']" data-hotkey="r"/>
                    <button name="action_set_to_challan" string="Use as Challan" type="object" invisible="state != 'pending'" data-hotkey="d"/>
                    <button name="action_export_json" string="Download JSON" class="oe_highlight" type="object" invisible="not error_message" groups="base.group_no_one"/>
                    <field name="state" widget="statusbar" statusbar_visible="pending,generated" invisible="state == 'challan'"/>
                </header>
                <field name="blocking_level" invisible="1"/>
                <div class="alert alert-danger" role="alert" style="margin-bottom:0px;" invisible="not error_message or blocking_level == 'error'">
                    <div class="o_row">
                        <field name="error_message"/>
                    </div>
                </div>
                <div class="alert alert-warning" role="alert" style="margin-bottom:0px;" invisible="not error_message or blocking_level == 'warning'">
                    <div class="o_row">
                        <field name="error_message"/>
                    </div>
                </div>
                <sheet>
                    <div class="oe_title">
                        <h1 invisible="not name">
                            <field name="name" readonly="1"/>
                        </h1>
                    </div>
                    <group name="document_details" string="Document Details">
                        <group>
                            <field name="picking_id" invisible="1"/>
                            <field name="picking_type_code" invisible="1"/>
                            <field name="sub_type_code" invisible="1"/>
                            <field name="type_id" widget="selection" readonly="state != 'pending'" domain="[
                                        ('allowed_supply_type', 'in', (picking_type_code == 'incoming' and 'in' or 'out', 'both')), ('code','=','CHL')]"/>
                            <field name="type_description" invisible="sub_type_code != '8'" required="sub_type_code == '8'"/>
                        </group>
                        <group>
                            <field name="ewaybill_date" invisible="not ewaybill_date"/>
                            <field name="document_number"/>
                            <field name="document_date" invisible="picking_type_code == 'incoming'" readonly="1"/>
                        </group>
                    </group>
                    <group name="partners" string="Address Details">
                        <field name="company_id" invisible="1"/>
                        <field name="is_bill_to_editable" invisible="1"/>
                        <field name="is_bill_from_editable" invisible="1"/>
                        <field name="is_ship_to_editable" invisible="1"/>
                        <field name="is_ship_from_editable" invisible="1"/>
                        <group>
                            <field name="fiscal_position_id" readonly="state != 'pending'"/>
                        </group>
                        <group>
                        </group>
                        <group>
                            <field name="partner_bill_from_id"
                                   force_save="1"
                                   readonly="state == 'pending' and not is_bill_from_editable or state != 'pending'"/>
                            <field name="partner_bill_to_id"
                                   force_save="1"
                                   readonly="state == 'pending' and not is_bill_to_editable or state != 'pending'"/>
                        </group>
                        <group>
                            <field name="partner_ship_from_id" force_save="1" readonly="state == 'pending' and not is_ship_from_editable or state != 'pending'"/>
                            <field name="partner_ship_to_id" force_save="1" readonly="state == 'pending' and not is_ship_to_editable or state != 'pending'"/>
                        </group>
                    </group>
                    <group name="transporter" string="Transporter Details">
                        <group>
                            <field name="transporter_id" readonly="state != 'pending'" required="not mode"/>
                        </group>
                        <group/>
                    </group>
                    <group name="part_b" string="Transportation Details (Part B)">
                        <group>
                            <field name="mode" readonly="state != 'pending'"/>
                            <field name="vehicle_type"
                                   invisible="mode not in ('1','4')"
                                   required="mode == '1'"
                                   readonly="state != 'pending'"/>
                            <field name="vehicle_no"
                                   invisible="mode not in ('1','4')"
                                   required="mode == '1'"
                                   readonly="state != 'pending'"/>
                            <label for="distance" readonly="state != 'pending'"/>
                            <div class="o_row" name="distance">
                                <field name="distance" readonly="state != 'pending'"/>
                                <span>km</span>
                            </div>
                        </group>
                        <group>
                            <label for="transportation_doc_no" invisible="mode != '1'"/>
                            <label for="transportation_doc_no" invisible="mode != '2'" string="RR No"/>
                            <label for="transportation_doc_no" invisible="mode != '3'" string="Airway Bill"/>
                            <label for="transportation_doc_no" invisible="mode != '4'" string="Bill of lading No"/>
                            <div class="o_row">
                                <field name="transportation_doc_no"
                                       readonly="state != 'pending'"
                                       required="mode in ('2', '3', '4')"
                                       invisible="not mode"/>
                            </div>


                            <label for="transportation_doc_date" invisible="mode != '1'"/>
                            <label for="transportation_doc_date" invisible="mode != '2'" string="RR Date"/>
                            <label for="transportation_doc_date" invisible="mode != '3'" string="Airway Bill Date"/>
                            <label for="transportation_doc_date" invisible="mode != '4'" string="Bill of lading Date"/>
                            <div class="o_row">
                                <field name="transportation_doc_date"
                                       readonly="state != 'pending'"
                                       required="mode in ('2', '3', '4')"
                                       invisible="not mode"/>
                            </div>
                        </group>
                    </group>
                    <group name="cancel_ewaybill" string="Cancel details" invisible="state != 'cancel'">
                        <group>
                            <field name="cancel_reason" readonly="state != 'generated'"/>
                            <field name="cancel_remarks" readonly="state != 'generated'"/>
                        </group>
                        <group>
                        </group>
                    </group>
                    <notebook>
                        <page string="Item Details">
                            <field name="move_ids" mode="tree,kanban" force_save="1" readonly="state != 'pending'">
                                <tree editable="bottom" create="0" delete="0">
                                    <field name="company_currency_id" column_invisible="1"/>
                                    <field name="company_id" column_invisible="1"/>
                                    <field name="product_id" readonly="1"/>
                                    <field name="quantity" string="Quantity" readonly="1"/>
                                    <field name="ewaybill_price_unit" string="Unit Price"/>
                                    <field name="ewaybill_tax_ids" widget="many2many_tags"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="l10n_in_ewaybill_form_action" model="ir.actions.act_window">
        <field name="name">e-Waybill</field>
        <field name="res_model">l10n.in.ewaybill</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="l10n_in_ewaybill_form_view"/>
    </record>
</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_form_inherit_ewaybill" model="ir.ui.view">
        <field name="name">view.picking.form.inherit.ewaybill</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header/field[@name='state']" position="before">
                <button string="Create e-Waybill / Challan"
                        type="object"
                        name="action_l10n_in_ewaybill_create"
                        invisible="
                            country_code != 'IN'
                            or l10n_in_ewaybill_id
                            or (picking_type_code == 'incoming' and state not in ('done', 'assigned'))
                            or (picking_type_code != 'incoming' and state != 'done')"
                        data-hotkey="e"
                        groups="stock.group_stock_manager"/>
            </xpath>
            <xpath expr="//sheet/div[hasclass('oe_button_box')]" position="inside">
                <field name="l10n_in_ewaybill_id" invisible="1" groups="stock.group_stock_manager"/>
                <button name="action_open_l10n_in_ewaybill"
                        class="oe_stat_button"
                        icon="fa-truck"
                        type="object"
                        invisible="not l10n_in_ewaybill_id"
                        groups="stock.group_stock_manager">
                        <div class="o_stat_info">
                            <span class="o_stat_text">e-Waybill / Challan</span>
                        </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\l10n_in_ewaybill_cancel.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class EwaybillCancel(models.TransientModel):

    _name = 'l10n.in.ewaybill.cancel'
    _description = 'Cancel Ewaybill'

    l10n_in_ewaybill_id = fields.Many2one('l10n.in.ewaybill', string='Ewaybill', required=True)
    cancel_reason = fields.Selection(selection=[
        ("1", "Duplicate"),
        ("2", "Data Entry Mistake"),
        ("3", "Order Cancelled"),
        ("4", "Others"),
        ], string="Cancel Reason", required=True
    )
    cancel_remarks = fields.Char("Cancel Remarks")

    def cancel_ewaybill(self):
        self.l10n_in_ewaybill_id.write({
            'cancel_reason': self.cancel_reason,
            'cancel_remarks': self.cancel_remarks,
        })
        self.l10n_in_ewaybill_id._ewaybill_cancel()

```

## File: wizard\l10n_in_ewaybill_cancel_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_ewaybill_cancel_form" model="ir.ui.view">
            <field name="name">l10n.in.ewaybill.cancel.form</field>
            <field name="model">l10n.in.ewaybill.cancel</field>
            <field name="arch" type="xml">
                <form>
                    <group>
                        <group>
                            <field name="cancel_reason"/>
                            <field name="cancel_remarks"/>
                        </group>
                        <group></group>
                    </group>
                    <footer>
                        <button string='Cancel Ewaybill' name="cancel_ewaybill" type="object" class="btn-primary" data-hotkey="c"/>
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
               </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import l10n_in_ewaybill_cancel

```

