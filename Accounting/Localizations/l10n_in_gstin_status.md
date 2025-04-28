# Odoo Module: l10n_in_gstin_status

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "Indian - Check GST Number Status",
    'countries': ['in'],
    "category": "Accounting/Localizations",
    "depends": [
        "l10n_in",
    ],
    "data": [
        "views/account_move_views.xml",
        "views/res_partner_views.xml",
    ],
    "installable": True,
    "license": "LGPL-3",
}

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_in_partner_gstin_status = fields.Boolean(
        string="GST Status",
        compute="_compute_l10n_in_partner_gstin_status_and_date",
    )
    l10n_in_show_gstin_status = fields.Boolean(compute="_compute_l10n_in_show_gstin_status")
    l10n_in_gstin_verified_date = fields.Date(compute="_compute_l10n_in_partner_gstin_status_and_date")

    @api.depends('partner_id', 'state', 'payment_state', 'l10n_in_gst_treatment')
    def _compute_l10n_in_show_gstin_status(self):
        indian_moves = self.filtered(lambda m: m.country_code == 'IN')
        (self - indian_moves).l10n_in_show_gstin_status = False
        for move in indian_moves:
            move.l10n_in_show_gstin_status = (
                move.partner_id
                and move.state == 'posted'
                and move.move_type != 'entry'
                and move.payment_state not in ['paid', 'reversed']
                and move.l10n_in_gst_treatment in ['regular', 'composition', 'special_economic_zone', 'deemed_export', 'uin_holders']
            )

    @api.depends('partner_id')
    def _compute_l10n_in_partner_gstin_status_and_date(self):
        for move in self:
            if move.country_code == 'IN' and move.payment_state not in ['paid', 'reversed'] and move.state != 'cancel':
                move.l10n_in_partner_gstin_status = move.partner_id.l10n_in_gstin_verified_status
                move.l10n_in_gstin_verified_date = move.partner_id.l10n_in_gstin_verified_date
            else:
                move.l10n_in_partner_gstin_status = False
                move.l10n_in_gstin_verified_date = False

    def l10n_in_verify_partner_gstin_status(self):
        self.ensure_one()
        return self.with_company(self.company_id).partner_id.action_l10n_in_verify_gstin_status()

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError, AccessError, ValidationError
from odoo.addons.l10n_in.models.iap_account import IAP_SERVICE_NAME

_logger = logging.getLogger(__name__)


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_in_gstin_verified_status = fields.Boolean(
        string="GST Status",
        tracking=True,
    )
    l10n_in_gstin_verified_date = fields.Date(
        string="GSTIN Verified Date",
        tracking=True,
    )

    @api.onchange('vat')
    def _onchange_l10n_in_gst_status(self):
        """
        Reset GST Status Whenever the `vat` of partner changes
        """
        for partner in self:
            if partner.country_code == 'IN':
                partner.l10n_in_gstin_verified_status = False
                partner.l10n_in_gstin_verified_date = False

    def action_l10n_in_verify_gstin_status(self):
        self.ensure_one()
        self.check_access('write')
        if self.env.company.sudo().account_fiscal_country_id.code != 'IN':
            raise UserError(_('You must be logged in an Indian company to use this feature'))
        if not self.vat:
            raise ValidationError(_("Please enter the GSTIN"))
        is_production = self.env.company.sudo().l10n_in_edi_production_env
        params = {
            "gstin_to_search": self.vat,
        }
        try:
            response = self.env['iap.account']._l10n_in_connect_to_server(
                is_production,
                params,
                '/iap/l10n_in_reports/1/public/search',
                "l10n_in_gstin_status.endpoint"
            )
        except AccessError:
            raise UserError(_("Unable to connect with GST network"))
        if response.get('error') and any(e.get('code') == 'no-credit' for e in response['error']):
            return self.env["bus.bus"]._sendone(self.env.user.partner_id, "iap_notification",
                {
                    "type": "no_credit",
                    "title": _("Not enough credits to check GSTIN status"),
                    "get_credits_url": self.env["iap.account"].get_credits_url(service_name=IAP_SERVICE_NAME),
                },
            )
        gst_status = response.get('data', {}).get('sts', "")
        if gst_status.casefold() == 'active':
            l10n_in_gstin_verified_status = True
        elif gst_status:
            l10n_in_gstin_verified_status = False
            date_from = response.get("data", {}).get("cxdt", '')
            if date_from and re.search(r'\d', date_from):
                message = _(
                    "GSTIN %(vat)s is %(status)s and Effective from %(date_from)s.",
                    vat=self.vat,
                    status=gst_status,
                    date_from=date_from,
                )
            else:
                message = _(
                    "GSTIN %(vat)s is %(status)s, effective date is not available.",
                    vat=self.vat,
                    status=gst_status
                )
            if not is_production:
                message += _(" Warning: You are currently in a test environment. The result is a dummy.")
            self.message_post(body=message)
        else:
            _logger.info("GST status check error %s", response)
            if response.get('error') and any(e.get('code') == 'SWEB_9035' for e in response['error']):
                raise UserError(
                    _("The provided GSTIN is invalid. Please check the GSTIN and try again.")
                )
            default_error_message = _(
                "Something went wrong while fetching the GST status."
                "Please Contact Support if the error persists with"
                "Response: %(response)s",
                response=response
            )
            error_messages = [
                f"[{error.get('code') or _('Unknown')}] {error.get('message') or default_error_message}"
                for error in response.get('error')
            ]
            raise UserError(
                error_messages
                and '\n'.join(error_messages)
                or default_error_message
            )
        self.write({
            "l10n_in_gstin_verified_status": l10n_in_gstin_verified_status,
            "l10n_in_gstin_verified_date": fields.Date.today(),
        })
        return {
            "type": "ir.actions.client",
            "tag": "display_notification",
            "params": {
                "type": "info",
                "message": _("GSTIN Status Updated Successfully"),
                "next": {"type": "ir.actions.act_window_close"},
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import res_partner

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="move_form_inherit_l10n_in_gst_verification" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n.in.gst.verification</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='ref']" position="after">
                <label for="l10n_in_partner_gstin_status"
                    invisible="not l10n_in_show_gstin_status"/>
                <div name="status_date_container"
                    invisible="not l10n_in_show_gstin_status">
                    <field name="l10n_in_partner_gstin_status" class="d-none"/>
                    <span class="text-nowrap" readonly="1">
                        <span invisible="not l10n_in_partner_gstin_status"
                            class="oe_inline text-success">Active</span>
                        <span invisible="not l10n_in_gstin_verified_date or l10n_in_partner_gstin_status"
                            class="oe_inline text-danger">Inactive</span>
                        <span class="text-muted oe_inline">
                            <span invisible="l10n_in_gstin_verified_date">Not Checked</span>
                            <span invisible="not l10n_in_gstin_verified_date" class="ps-3">Checked: </span>
                            <field name="l10n_in_gstin_verified_date" class="oe_inline" widget="remaining_days"/>
                            <button name="l10n_in_verify_partner_gstin_status"
                                type="object" icon="fa-refresh"
                                class="oe_link p-0 ps-3" title="Verify GSTIN status" />
                        </span>
                    </span>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="l10n_in_view_partner_base_vat_form" model="ir.ui.view">
            <field name="name">l10n.in.gstin.status.view.partner.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base_vat.view_partner_base_vat_form" />
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vies_valid']" position="after">
                    <span invisible="country_code != 'IN' or not vat or 'IN' not in fiscal_country_codes">
                        <span invisible="not l10n_in_gstin_verified_date or not l10n_in_gstin_verified_status"
                            class="oe_inline text-success">Active</span>
                        <span invisible="not l10n_in_gstin_verified_date or l10n_in_gstin_verified_status"
                            class="oe_inline text-danger">Inactive</span>
                        <span invisible="not l10n_in_gstin_verified_date and not l10n_in_gstin_verified_status" class="text-muted">
                            (
                            <field name="l10n_in_gstin_verified_date" widget="remaining_days" class="oe_inline" readonly="1" />
                            <button name="action_l10n_in_verify_gstin_status" type="object" icon="fa-refresh"
                                class="oe_link p-0 ps-2" title="Reverify GSTIN status" />
                            )
                        </span>
                        <button string="Check Status" name="action_l10n_in_verify_gstin_status" type="object"
                            icon="fa-check" class="oe_link p-0" title="Check GSTIN status"
                            invisible="l10n_in_gstin_verified_date" />
                    </span>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

