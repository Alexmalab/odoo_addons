# Odoo Module: snailmail_account

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import controllers
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
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
from odoo import _
from odoo.addons.portal.controllers.portal import CustomerPortal


class PortalAccount(CustomerPortal):

    def _prepare_portal_layout_values(self):
        # EXTENDS 'portal'
        portal_layout_values = super()._prepare_portal_layout_values()
        portal_layout_values['invoice_sending_methods'].update({'snailmail': _('by Post')})
        return portal_layout_values

```

## File: controllers\__init__.py

```python
from . import portal

```

## File: models\account_move_send.py

```python
from odoo import api, models, _


class AccountMoveSend(models.AbstractModel):
    _inherit = 'account.move.send'

    # -------------------------------------------------------------------------
    # ALERTS
    # -------------------------------------------------------------------------
    def _get_alerts(self, moves, moves_data):
        # EXTENDS 'account'
        alerts = super()._get_alerts(moves, moves_data)
        if snailmail_moves_without_valid_address := moves.filtered(
            lambda m: 'snailmail' in moves_data[m]['sending_methods'] and not self.env['snailmail.letter']._is_valid_address(m.partner_id)
        ):
            alerts['snailmail_account_partner_invalid_address'] = {
                'level': 'danger' if len(snailmail_moves_without_valid_address) == 1 else 'warning',
                'message': _(
                    "The partners on the following invoices have no valid address, "
                    "so those invoices will not be sent: %s",
                    ", ".join(snailmail_moves_without_valid_address.mapped('name'))
                ),
                'action_text': _("View Invoice(s)"),
                'action': snailmail_moves_without_valid_address._get_records_action(name=_("Check Invoice(s)")),
            }
        return alerts

    # -------------------------------------------------------------------------
    # HELPERS
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

    # -------------------------------------------------------------------------
    # SENDING METHODS
    # -------------------------------------------------------------------------
    def _is_applicable_to_move(self, method, move, **move_data):
        # EXTENDS 'account'
        if method == 'snailmail':
            return self.env['snailmail.letter']._is_valid_address(move.partner_id)
        else:
            return super()._is_applicable_to_move(method, move, **move_data)

    def _hook_if_success(self, moves_data):
        # EXTENDS 'account'
        super()._hook_if_success(moves_data)

        to_send = {
            move: move_data
            for move, move_data in moves_data.items()
            if 'snailmail' in move_data['sending_methods'] and self._is_applicable_to_move('snailmail', move, **move_data)
        }
        if to_send:
            self.env['snailmail.letter'].create([
                {
                    'user_id': move_data.get('author_user_id') or self.env.user.id,
                    **self._prepare_snailmail_letter_values(move),
                }
                for move, move_data in to_send.items()
            ])\
            ._snailmail_print(immediate=False)

```

## File: models\res_partner.py

```python
from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    invoice_sending_method = fields.Selection(
        selection_add=[('snailmail', 'by Post')],
    )

```

## File: models\__init__.py

```python
from . import account_move_send
from . import res_partner

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
                        <field name="snailmail_cover" readonly="snailmail_cover_readonly"/>
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

## File: wizard\account_move_send_batch_wizard.py

```python
from odoo import _, fields, models


class AccountMoveSendBatchWizard(models.TransientModel):
    _inherit = 'account.move.send.batch.wizard'

    send_by_post_stamps = fields.Integer(compute='_compute_send_by_post_stamps')

    def _compute_send_by_post_stamps(self):
        for wizard in self:
            partner_with_valid_address = wizard.move_ids.partner_id.filtered(
                self.env['snailmail.letter']._is_valid_address
            )
            wizard.send_by_post_stamps = len(partner_with_valid_address)

    def _compute_summary_data(self):
        # EXTENDS 'account'
        super()._compute_summary_data()
        for wizard in self:
            if wizard.summary_data and 'snailmail' in wizard.summary_data:
                wizard.summary_data['snailmail'].update({'extra': _('(Stamps: %s)', wizard.send_by_post_stamps)})

```

## File: wizard\__init__.py

```python
from . import account_move_send_batch_wizard

```

