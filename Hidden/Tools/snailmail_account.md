# Odoo Module: snailmail_account

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Snail Mail - Account",
    'description': """
Allows users to send invoices by post
=====================================================
        """,
    'category': 'Hidden/Tools',
    'version': '0.1',
    'depends': ['account', 'snailmail'],
    'data': [
        'views/res_config_settings_views.xml',
        'wizard/account_move_send_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountMove(models.Model):
    _inherit = "account.move"

    def _get_pdf_and_send_invoice_vals(self, template, **kwargs):
        # EXTENDS account
        vals = super()._get_pdf_and_send_invoice_vals(template, **kwargs)
        vals['checkbox_send_by_post'] = False
        return vals

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Company(models.Model):
    _inherit = "res.company"

    invoice_is_snailmail = fields.Boolean(string='Send by Post', default=False)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-	
# Part of Odoo. See LICENSE file for full copyright and licensing details.	

from odoo import fields, models	


class ResConfigSettings(models.TransientModel):	
    _inherit = 'res.config.settings'	

    invoice_is_snailmail = fields.Boolean(string='Send by Post', related='company_id.invoice_is_snailmail', readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_move
from . import res_company
from . import res_config_settings

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.snailmail.account</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="100"/>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='send_default']" position="inside">
                <div invisible="not module_snailmail_account">
                    <field name="invoice_is_snailmail"/>
                    <label for="invoice_is_snailmail"/>
                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                </div>
            </xpath>

            <setting id="send_invoices_followups" position="inside">
                <div class="mt16" invisible="not module_snailmail_account">
                    <div>
                        <field name="snailmail_color"/>
                        <label for="snailmail_color"/>
                        <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                    </div>
                    <div>
                        <field name="snailmail_duplex"/>
                        <label for="snailmail_duplex"/>
                        <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                    </div>
                    <div>
                        <field name="snailmail_cover"/>
                        <label for="snailmail_cover"/>
                        <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                    </div>
                </div>
                <widget name="iap_buy_more_credits" service_name="snailmail"/>
            </setting>
        </field>
    </record>
</odoo>

```

## File: wizard\account_move_send.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, _


class AccountMoveSend(models.TransientModel):
    _inherit = 'account.move.send'

    enable_send_by_post = fields.Boolean(compute='_compute_enable_send_by_post')
    checkbox_send_by_post = fields.Boolean(
        string="By Post",
        compute='_compute_checkbox_send_by_post',
        store=True,
        readonly=False,
    )
    send_by_post_cost = fields.Integer(string='Stamps', compute='_compute_send_by_post_extra_fields')
    send_by_post_warning_message = fields.Text(compute='_compute_send_by_post_extra_fields')
    send_by_post_readonly = fields.Boolean(compute='_compute_send_by_post_extra_fields')

    def _get_wizard_values(self):
        # EXTENDS 'account'
        values = super()._get_wizard_values()
        values['send_by_post'] = self.checkbox_send_by_post
        return values

    @api.model
    def _get_wizard_vals_restrict_to(self, only_options):
        # EXTENDS 'account'
        values = super()._get_wizard_vals_restrict_to(only_options)
        return {
            'checkbox_send_by_post': False,
            **values,
        }

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('mode')
    def _compute_enable_send_by_post(self):
        for wizard in self:
            wizard.enable_send_by_post = wizard.mode in ('invoice_single', 'invoice_multi') \
                and all(x.state == 'posted' for x in wizard.move_ids)

    @api.depends('company_id')
    def _compute_checkbox_send_by_post(self):
        for wizard in self:
            wizard.checkbox_send_by_post = wizard.company_id.invoice_is_snailmail

    @api.depends('mode', 'checkbox_send_by_post')
    def _compute_send_by_post_extra_fields(self):
        for wizard in self:
            partner_with_valid_address = wizard.move_ids.partner_id \
                .filtered(self.env['snailmail.letter']._is_valid_address)
            wizard.send_by_post_cost = len(partner_with_valid_address)
            wizard.send_by_post_readonly = not partner_with_valid_address
            wizard.send_by_post_warning_message = False

            if wizard.enable_send_by_post and wizard.checkbox_send_by_post:
                invoice_without_valid_address = wizard.move_ids.filtered(
                    lambda move: not self.env['snailmail.letter']._is_valid_address(move.partner_id))
                if invoice_without_valid_address:
                    wizard.send_by_post_warning_message = _(
                        "The partners on the following invoices have no valid address, "
                        "so those invoices will not be sent: %s",
                        ", ".join(invoice_without_valid_address.mapped('name'))
                    )

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    @api.model
    def _prepare_snailmail_letter_values(self, move):
        return {
            'partner_id': move.partner_id.id,
            'model': 'account.move',
            'res_id': move.id,
            'company_id': move.company_id.id,
            'report_template': self.env['ir.actions.report']._get_report('account.account_invoices').id
        }

    @api.model
    def _hook_if_success(self, moves_data, from_cron=False, allow_fallback_pdf=False):
        # EXTENDS 'account'
        super()._hook_if_success(moves_data, from_cron=from_cron, allow_fallback_pdf=allow_fallback_pdf)

        to_send = {
            move: move_data
            for move, move_data in moves_data.items()
            if move_data.get('send_by_post') and move.invoice_pdf_report_id
        }
        if to_send:
            self.env['snailmail.letter'].create([
                {
                    'user_id': move_data.get('sp_user_id', self.env.user.id),
                    **self._prepare_snailmail_letter_values(move),
                }
                for move, move_data in to_send.items()
            ])\
            ._snailmail_print(immediate=False)

```

## File: wizard\account_move_send_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_move_send_inherit_snailmail" model="ir.ui.view">
        <field name="name">account.move.send.form.inherit.snailmail</field>
        <field name="model">account.move.send</field>
        <field name="inherit_id" ref="account.account_move_send_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='standard_options']" position="inside">
                <field name="enable_send_by_post" invisible="1"/>
                <field name="send_by_post_readonly" invisible="1"/>

                <div name="option_send_by_post"
                     invisible="not enable_send_by_post">
                    <field name="checkbox_send_by_post" readonly="send_by_post_readonly"/>
                    <b><label for="checkbox_send_by_post" class="mr4"/></b>
                    <i class="fa fa-question-circle"
                        role="img"
                        aria-label="Warning"
                        title="The address is unknown on the partner"
                        invisible="not send_by_post_readonly"/>

                    <span invisible="send_by_post_cost in (0, 1) or not checkbox_send_by_post">
                        <b>(
                            <field name="send_by_post_cost" options="{'digits':[0,0]}" class="mr4"/>
                            <label for="send_by_post_cost"/>
                        )</b>
                    </span>
                </div>
            </xpath>
            <xpath expr="//div[@name='warnings']" position="inside">
                <div class="alert alert-warning"
                     role="alert"
                     invisible="not send_by_post_warning_message">
                    <field name="send_by_post_warning_message"/>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_move_send

```

