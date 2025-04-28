# Odoo Module: account_audit_trail

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from odoo.exceptions import UserError


def uninstall_hook(env):
    if not env.ref('base.module_base').demo:
        raise UserError('The module "Account Audit Trail" (account_audit_trail) cannot be uninstalled.')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Account Audit Trail',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'summary': 'Account Audit Trail',
    'depends': ['account'],
    'data': [
        'report/audit_trail_report_views.xml',
        'views/res_config_settings_views.xml',
    ],
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: models\account_bank_statement_line.py

```python
from odoo import models


class AccountBankStatementLine(models.Model):
    _inherit = "account.bank.statement.line"

    def unlink(self):
        tracked_lines = self.filtered(lambda stl: stl.company_id.check_account_audit_trail)
        super(AccountBankStatementLine, tracked_lines.with_context(soft_delete=True)).unlink()
        return super(AccountBankStatementLine, self - tracked_lines).unlink()

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
import logging

from odoo import api, models, _
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class AccountMove(models.Model):
    _inherit = "account.move"

    @api.ondelete(at_uninstall=False)
    def _unlink_account_audit_trail_except_once_post(self):
        if not self.env.context.get('force_delete') and self._is_protected_by_audit_trail():
            raise UserError(_("To keep the audit trail, you can not delete journal entries once they have been posted.\nInstead, you can cancel the journal entry."))

    def unlink(self):
        if self.env.context.get('soft_delete'):
            self.button_cancel()
            return True
        # Add logger here because in api ondelete account.move.line is deleted and we can't get total amount
        logger_msg = False
        if self.env.context.get('force_delete') and self._is_protected_by_audit_trail():
            moves_details = []
            for move in self:
                entry_details = f"{move.name} ({move.id}) amount {move.amount_total} {move.currency_id.name} and partner {move.partner_id.display_name}"
                account_balances_per_account = defaultdict(float)
                for line in move.line_ids:
                    account_balances_per_account[line.account_id] += line.balance
                account_details = "\n".join(
                    f"- {account.name} ({account.id}) with balance {balance} {move.currency_id.name}"
                    for account, balance in account_balances_per_account.items()
                )
                moves_details.append(f"{entry_details}\n{account_details}")
            moves_details = "\n".join(moves_details)
            logger_msg = f"\nForce deleted Journal Entries by {self.env.user.name} ({self.env.user.id})\nEntries\n{moves_details}"
        res = super().unlink()
        if logger_msg:
            _logger.info(logger_msg)
        return res

    def _is_protected_by_audit_trail(self):
        return any(move.posted_before and move.company_id.check_account_audit_trail for move in self)

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv.expression import OR

bypass_token = object()


class Message(models.Model):
    _inherit = 'mail.message'

    account_audit_log_preview = fields.Html(string="Description", compute="_compute_account_audit_log_preview")
    account_audit_log_move_id = fields.Many2one(
        comodel_name='account.move',
        string="Journal Entry",
        compute="_compute_account_audit_log_move_id",
        search="_search_account_audit_log_move_id",
    )
    account_audit_log_partner_id = fields.Many2one(
        comodel_name='res.partner',
        string="Partner",
        compute="_compute_account_audit_log_partner_id",
        search="_search_account_audit_log_partner_id",
    )
    account_audit_log_account_id = fields.Many2one(
        comodel_name='account.account',
        string="Account",
        compute="_compute_account_audit_log_account_id",
        search="_search_account_audit_log_account_id",
    )
    account_audit_log_tax_id = fields.Many2one(
        comodel_name='account.tax',
        string="Tax",
        compute="_compute_account_audit_log_tax_id",
        search="_search_account_audit_log_tax_id",
    )
    account_audit_log_company_id = fields.Many2one(
        comodel_name='res.company',
        string="Company ",
        compute="_compute_account_audit_log_company_id",
        search="_search_account_audit_log_company_id",
    )
    account_audit_log_display_name = fields.Char(compute='_compute_account_audit_log_display_name')
    show_audit_log = fields.Boolean(compute="_compute_show_audit_log", search="_search_show_audit_log")

    def _compute_account_audit_log_preview(self):
        for message in self:
            title = message.subject or message.preview
            tracking_value_ids = message.sudo().tracking_value_ids.filtered(lambda tracking: not tracking.field_groups or self.env.is_superuser() or self.user_has_groups(tracking.field_groups))
            if not title and tracking_value_ids:
                title = _("Updated")
            elif not title and message.subtype_id and not message.subtype_id.internal:
                title = message.subtype_id.display_name
            audit_log_preview = Markup("<div>%s</div>") % title
            for fmt_vals in tracking_value_ids._tracking_value_format():
                field_desc = fmt_vals['changedField']
                old_value = fmt_vals['oldValue']['value']
                new_value = fmt_vals['newValue']['value']
                audit_log_preview += Markup(
                    "<li>%(old_value)s <i class='o_TrackingValue_separator fa fa-long-arrow-right mx-1 text-600' title='%(title)s' role='img' aria-label='%(title)s'></i>%(new_value)s (%(field)s)</li>"
                ) % {
                    'old_value': old_value,
                    'new_value': new_value,
                    'title': _("Changed"),
                    'field': field_desc,
                }
            message.account_audit_log_preview = audit_log_preview

    def _compute_account_audit_log_move_id(self):
        self._compute_audit_log_related_record_id('account.move', 'account_audit_log_move_id', [
            ('company_id.check_account_audit_trail', '=', True),
        ])

    def _search_account_audit_log_move_id(self, operator, value):
        return self._search_audit_log_related_record_id('account.move', operator, value)

    def _compute_account_audit_log_account_id(self):
        self._compute_audit_log_related_record_id('account.account', 'account_audit_log_account_id', [
            ('company_id.check_account_audit_trail', '=', True),
        ])

    def _search_account_audit_log_account_id(self, operator, value):
        return self._search_audit_log_related_record_id('account.account', operator, value)

    def _compute_account_audit_log_tax_id(self):
        self._compute_audit_log_related_record_id('account.tax', 'account_audit_log_tax_id', [
            ('company_id.check_account_audit_trail', '=', True),
        ])

    def _search_account_audit_log_tax_id(self, operator, value):
        return self._search_audit_log_related_record_id('account.tax', operator, value)

    def _compute_account_audit_log_company_id(self):
        self._compute_audit_log_related_record_id('res.company', 'account_audit_log_company_id', [
            ('check_account_audit_trail', '=', True),
        ])

    def _search_account_audit_log_company_id(self, operator, value):
        return self._search_audit_log_related_record_id('res.company', operator, value)

    def _compute_account_audit_log_partner_id(self):
        self._compute_audit_log_related_record_id('res.partner', 'account_audit_log_partner_id', [
            '|', ('company_id', '=', False), ('company_id.check_account_audit_trail', '=', True),
            '|', ('customer_rank', '>', 0), ('supplier_rank', '>', 0),
        ])

    def _search_account_audit_log_partner_id(self, operator, value):
        return self._search_audit_log_related_record_id('res.partner', operator, value)

    def _compute_account_audit_log_display_name(self):
        for message in self:
            message.account_audit_log_display_name = (
                message.account_audit_log_move_id
                or message.account_audit_log_account_id
                or message.account_audit_log_tax_id
                or message.account_audit_log_partner_id
                or message.account_audit_log_company_id
            ).display_name

    def _compute_show_audit_log(self):
        for message in self:
            message.show_audit_log = message.message_type == 'notification' and (
                message.account_audit_log_move_id
                or message.account_audit_log_account_id
                or message.account_audit_log_tax_id
                or message.account_audit_log_partner_id
                or message.account_audit_log_company_id
            )

    def _search_show_audit_log(self, operator, value):
        if operator not in ['=', '!='] or not isinstance(value, bool):
            raise UserError(_('Operation not supported'))
        return [('message_type', '=', 'notification')] + OR([
            [('model', '=', 'account.move'), ('res_id', 'in', self.env['account.move']._search([
                ('company_id.check_account_audit_trail', operator, value),
            ]))],
            [('model', '=', 'account.account'), ('res_id', 'in', self.env['account.account']._search([
                ('company_id.check_account_audit_trail', operator, value),
            ]))],
            [('model', '=', 'account.tax'), ('res_id', 'in', self.env['account.tax']._search([
                ('company_id.check_account_audit_trail', operator, value),
            ]))],
            [('model', '=', 'res.partner'), ('res_id', 'in', self.env['res.partner']._search([
                '|', ('company_id', '=', False), ('company_id.check_account_audit_trail', operator, value),
                '|', ('customer_rank', '>', 0), ('supplier_rank', '>', 0),
            ]))],
            [('model', '=', 'res.company'), ('res_id', 'in', self.env['res.company']._search([
                ('check_account_audit_trail', operator, value),
            ]))],
        ])

    def _compute_audit_log_related_record_id(self, model, fname, domain):
        messages_of_related = self.filtered(lambda m: m.model == model and m.res_id)
        (self - messages_of_related)[fname] = False
        if messages_of_related:
            related_recs = self.env[model].sudo().search([('id', 'in', messages_of_related.mapped('res_id'))] + domain)
            recs_by_id = {record.id: record for record in related_recs}
            for message in messages_of_related:
                message[fname] = recs_by_id.get(message.res_id, False)

    def _search_audit_log_related_record_id(self, model, operator, value):
        if operator in ['=', 'like', 'ilike', '!=', 'not ilike', 'not like'] and isinstance(value, str):
            res_id_domain = [('res_id', 'in', self.env[model]._name_search(value, operator=operator))]
        elif operator in ['=', 'in', '!=', 'not in']:
            res_id_domain = [('res_id', operator, value)]
        else:
            raise UserError(_('Operation not supported'))
        return [('model', '=', model)] + res_id_domain

    @api.ondelete(at_uninstall=True)
    def _except_audit_log(self):
        if self.env.context.get('bypass_audit') is bypass_token:
            return
        to_check = self
        partner_message = self.filtered(lambda m: m.account_audit_log_partner_id)
        if partner_message:
            # The audit trail uses the cheaper check on `customer_rank`, but that field could be set
            # without actually having an invoice linked (i.e. creation of the contact through the
            # Invoicing/Customers menu)
            has_related_move = self.env['account.move'].sudo().search_count([
                ('partner_id', 'in', partner_message.account_audit_log_partner_id.ids),
                ('company_id.check_account_audit_trail', '=', True),
            ], limit=1)
            if not has_related_move:
                to_check -= partner_message
        for message in to_check:
            if message.show_audit_log and not (
                message.account_audit_log_move_id
                and not message.account_audit_log_move_id.posted_before
            ):
                raise UserError(_("You cannot remove parts of the audit trail."))

    def write(self, vals):
        # We allow any whitespace modifications in the subject
        normalized_subject = ' '.join(vals['subject'].split()) if vals.get('subject') else None
        if (
            vals.keys() & {'res_id', 'res_model', 'message_type', 'subtype_id'}
            or ('subject' in vals and any(' '.join(s.subject.split()) != normalized_subject for s in self if s.subject))
            or ('body' in vals and any(self.mapped('body')))
        ):
            self._except_audit_log()
        return super().write(vals)

```

## File: models\mail_tracking_value.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class MailTrackingValues(models.Model):
    _inherit = 'mail.tracking.value'

    @api.ondelete(at_uninstall=True)
    def _except_audit_log(self):
        self.mail_message_id._except_audit_log()

    def write(self, vals):
        self._except_audit_log()
        return super().write(vals)

```

## File: models\merge_partner_automatic.py

```python
from odoo import models
from .mail_message import bypass_token


class MergePartnerAutomatic(models.TransientModel):
    _inherit = 'base.partner.merge.automatic.wizard'

    def _update_reference_fields(self, src_partners, dst_partner):
        return super(MergePartnerAutomatic, self.with_context(bypass_audit=bypass_token))._update_reference_fields(src_partners, dst_partner)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Company(models.Model):
    _inherit = "res.company"

    check_account_audit_trail = fields.Boolean(string='Audit Trail')

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    check_account_audit_trail = fields.Boolean(string='Audit Trail', related='company_id.check_account_audit_trail', readonly=False)

    @api.constrains('check_account_audit_trail')
    def _check_audit_trail_records(self):
        if not self.check_account_audit_trail:
            move_count = self.env['account.move'].search_count([('company_id', '=', self.company_id.id)], limit=1)
            if move_count > 0:
                raise UserError(_("Can't disable audit trail when there are existing records."))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_bank_statement_line
from . import account_move
from . import mail_message
from . import mail_tracking_value
from . import merge_partner_automatic
from . import res_company
from . import res_config_settings

```

## File: report\audit_trail_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_message_tree_audit_log" model="ir.ui.view">
        <field name="name">mail.message.tree.inherit.audit.log</field>
        <field name="model">mail.message</field>
        <field name="priority">99</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <tree edit="0" delete="0" create="0" action="action_open_document" type="object">
                <field name="res_id" column_invisible="True"/>
                <field name="date"/>
                <field name="author_id" widget="many2one_avatar"/>
                <field name="account_audit_log_display_name" string="Name"/>
                <field name="account_audit_log_preview"/>
            </tree>
        </field>
    </record>

    <record id="view_message_tree_audit_log_search" model="ir.ui.view">
        <field name="name">mail.message.search</field>
        <field name="model">mail.message</field>
        <field name="priority">99</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <search string="Messages Search">
                <field name="account_audit_log_move_id"/>
                <field name="account_audit_log_account_id"/>
                <field name="account_audit_log_tax_id"/>
                <field name="account_audit_log_partner_id"/>
                <field name="account_audit_log_company_id"/>
                <field name="author_id"/>
                <field name="date"/>
                <filter string="Journal Entry" name="account_move" domain="[('model', '=', 'account.move')]"/>
                <filter string="Account" name="account_account" domain="[('model', '=', 'account.account')]"/>
                <filter string="Taxes" name="account_tax" domain="[('model', '=', 'account.tax')]"/>
                <filter string="Partners" name="res_partner" domain="[('model', '=', 'res.partner')]"/>
                <filter string="Company" name="res_company" domain="[('model', '=', 'res.company')]"/>
                <separator/>
                <filter string="Update Only" name="update_only" domain="[('tracking_value_ids', '!=', False)]" groups="base.group_system"/>
                <filter string="Create Only" name="create_only" domain="[('tracking_value_ids', '=', False)]" groups="base.group_system"/>
                <separator/>
                <filter name="date" string="Date" date="date"/>
                <group expand="0" string="Group By">
                    <filter string="Date" name="group_by_date" domain="[]" context="{'group_by': 'date'}"/>
                    <filter string="Record" name="group_by_log_move_id" domain="[]" context="{'group_by': 'res_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_account_audit_trail_report" model="ir.actions.act_window">
        <field name="name">Audit Trail</field>
        <field name="res_model">mail.message</field>
        <field name="view_id" ref="view_message_tree_audit_log"/>
        <field name="view_mode">tree</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            To see the audit log you have to enable the audit trail option from settings
            </p>
        </field>
        <field name="domain">[
            ('message_type', '=', 'notification'),
            ('show_audit_log', '=', True),
        ]</field>
        <field name="search_view_id" ref="view_message_tree_audit_log_search"/>
    </record>

    <menuitem id="account_audit_trail_menu" name="Audit Trail" action="action_account_audit_trail_report" parent="account.account_reports_management_menu"/>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form_inherit_account_audit_trail" model="ir.ui.view">
        <field name="name">res.config.settings.form.inherit.account.audit.trail</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <app name="account" position="inside">
                <block title="Audit Trail" name="audit_trail_setting_container">
                    <setting string="Audit Trail" company_dependent="1" help="Activate Audit Trail">
                        <field name="check_account_audit_trail"/>
                        <button name="%(account_audit_trail.action_account_audit_trail_report)d"
                                type="action"
                                string="Go to Audit Trail"
                                class="oe_highlight"
                                invisible="check_account_audit_trail == False"/>
                    </setting>
                </block>
            </app>
        </field>
    </record>
</odoo>

```

