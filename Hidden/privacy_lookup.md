# Odoo Module: privacy_lookup

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Privacy',
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['mail'],
    'data': [
        'wizard/privacy_lookup_wizard_views.xml',
        'views/privacy_log_views.xml',
        'security/ir.model.access.csv',
        'data/ir_actions_server_data.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\ir_actions_server_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_actions_server_archive_all" model="ir.actions.server">
        <field name="name">Archive Selection</field>
        <field name="model_id" ref="privacy_lookup.model_privacy_lookup_wizard_line"/>
        <field name="binding_model_id" ref="privacy_lookup.model_privacy_lookup_wizard_line"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
records.action_archive_all()
        </field>
    </record>

    <record id="ir_actions_server_unlink_all" model="ir.actions.server">
        <field name="name">Delete Selection</field>
        <field name="model_id" ref="privacy_lookup.model_privacy_lookup_wizard_line"/>
        <field name="binding_model_id" ref="privacy_lookup.model_privacy_lookup_wizard_line"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
records.action_unlink_all()
        </field>
    </record>
</odoo>

```

## File: models\privacy_log.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class PrivacyLog(models.Model):
    _name = 'privacy.log'
    _description = 'Privacy Log'
    _rec_name = 'user_id'

    date = fields.Datetime(default=fields.Datetime.now, required=True)
    anonymized_name = fields.Char(required=True)
    anonymized_email = fields.Char(required=True)
    user_id = fields.Many2one(
        'res.users', string="Handled By", required=True,
        default=lambda self: self.env.user)
    execution_details = fields.Text()
    records_description = fields.Text(string="Found Records")
    additional_note = fields.Text()

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            vals['anonymized_name'] = self._anonymize_name(vals['anonymized_name'])
            vals['anonymized_email'] = self._anonymize_email(vals['anonymized_email'])
        return super().create(vals_list)

    def _anonymize_name(self, label):
        if not label:
            return ''
        if '@' in label:
            return self._anonymize_email(label)
        return ' '.join(e[0] + '*' * (len(e) - 1) for e in label.split(' ') if e)

    def _anonymize_email(self, label):
        def _anonymize_user(label):
            return '.'.join(e[0] + '*' * (len(e) - 1) for e in label.split('.') if e)
        def _anonymize_domain(label):
            if label in ['gmail.com', 'hotmail.com', 'yahoo.com']:  # More than half of addresses domains
                return label
            split_domain = label.split('.')
            return '.'.join([e[0] + '*' * (len(e) - 1) for e in split_domain[:-1] if e] + [split_domain[-1]])
        if not label or '@' not in label:
            return UserError(_('This email address is not valid (%s)', label))
        user, domain = label.split('@')
        return '{}@{}'.format(_anonymize_user(user), _anonymize_domain(domain))

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    def action_privacy_lookup(self):
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id('privacy_lookup.action_privacy_lookup_wizard')
        action['context'] = {
            'default_email': self.email,
            'default_name': self.name,
        }
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import privacy_log

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
privacy_lookup.access_privacy_lookup_wizard,access_privacy_lookup_wizard,privacy_lookup.model_privacy_lookup_wizard,base.group_system,1,1,1,1
privacy_lookup.access_privacy_lookup_wizard_line,access_privacy_lookup_wizard_line,privacy_lookup.model_privacy_lookup_wizard_line,base.group_system,1,1,1,0
privacy_lookup.access_privacy_log,access_privacy_log,privacy_lookup.model_privacy_log,base.group_system,1,1,1,1
```

## File: views\privacy_log_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="privacy_log_view_list" model="ir.ui.view">
        <field name="name">privacy.log.view.list</field>
        <field name="model">privacy.log</field>
        <field name="arch" type="xml">
            <list string="Privacy Logs">
                <field name="date" />
                <field name="anonymized_name"/>
                <field name="anonymized_email"/>
                <field name="user_id" widget="many2one_avatar_user"/>
                <field name="execution_details"/>
            </list>
        </field>
    </record>

    <record id="privacy_log_view_form" model="ir.ui.view">
        <field name="name">privacy.log.view.form</field>
        <field name="model">privacy.log</field>
        <field name="arch" type="xml">
            <form string="Privacy Log">
                <sheet>
                    <div class="oe_title">
                        <h1>
                            <field name="anonymized_name"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="anonymized_email"/>
                            <field name="date"/>
                            <field name="user_id" widget="many2one_avatar_user"/>
                        </group>
                        <group>
                            <field name="records_description"/>
                            <field name="execution_details"/>
                        </group>
                    </group>
                    <group>
                        <field name="additional_note"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="privacy_log_action" model="ir.actions.act_window">
        <field name="name">Privacy Logs</field>
        <field name="res_model">privacy.log</field>
        <field name="view_mode">list,form</field>
        <field name="context">{'create': False}</field>
    </record>

    <record id="privacy_log_form_action" model="ir.actions.act_window">
        <field name="name">Privacy Logs</field>
        <field name="res_model">privacy.log</field>
        <field name="view_mode">form</field>
        <field name="context">{'create': False}</field>
    </record>

    <menuitem
        id="privacy_menu"
        name="Privacy"
        parent="base.menu_custom"
        sequence="26"/>

    <menuitem
        name="Privacy Logs"
        action="privacy_log_action"
        id="pricacy_log_menu"
        parent="privacy_menu"
        sequence="1"/>
</odoo>

```

## File: wizard\privacy_lookup_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError
from odoo.tools import SQL


class PrivacyLookupWizard(models.TransientModel):
    _name = 'privacy.lookup.wizard'
    _description = 'Privacy Lookup Wizard'
    _transient_max_count = 0
    _transient_max_hours = 24

    name = fields.Char(required=True)
    email = fields.Char(required=True)
    line_ids = fields.One2many('privacy.lookup.wizard.line', 'wizard_id')
    execution_details = fields.Text(compute='_compute_execution_details', store=True)
    log_id = fields.Many2one('privacy.log')
    records_description = fields.Text(compute='_compute_records_description')
    line_count = fields.Integer(compute='_compute_line_count')

    @api.depends('line_ids')
    def _compute_line_count(self):
        for wizard in self:
            wizard.line_count = len(wizard.line_ids)

    def _compute_display_name(self):
        self.display_name = _('Privacy Lookup')

    def _get_query_models_blacklist(self):
        return [
            # Already Managed
            'res.partner',
            'res.users',
            # Ondelete Cascade
            'mail.notification',
            'mail.followers',
            'discuss.channel.member',
            # Special case for direct messages
            'mail.message',
        ]

    def _get_query(self):
        name = self.name.strip()
        email = f"%{self.email.strip()}%"
        email_normalized = tools.email_normalize(self.email.strip())

        # Step 1: Retrieve users/partners liked to email address or name
        query = SQL("""
            WITH indirect_references AS (
                SELECT id
                FROM res_partner
                WHERE email_normalized = %s
                OR name ilike %s)
            SELECT
                %s AS res_model_id,
                id AS res_id,
                active AS is_active
            FROM res_partner
            WHERE id IN (SELECT id FROM indirect_references)
            UNION ALL
            SELECT
                %s AS res_model_id,
                id AS res_id,
                active AS is_active
            FROM res_users
            WHERE (
                (login ilike %s)
                OR
                (partner_id IN (
                    SELECT id
                    FROM res_partner
                    WHERE email ilike %s or name ilike %s)))
            -- Step 2: Special case for direct messages
            UNION ALL
            SELECT
                %s AS res_model_id,
                id AS res_id,
                True AS is_active
            FROM mail_message
            WHERE author_id IN (SELECT id FROM indirect_references)
        """,
            # Indirect references CTE
            email_normalized, name,
            # Search on res.partner
            self.env['ir.model.data']._xmlid_to_res_id('base.model_res_partner'),
            # Search on res.users
            self.env['ir.model.data']._xmlid_to_res_id('base.model_res_users'), email, email, name,
            # Direct messages
            self.env['ir.model.data']._xmlid_to_res_id('mail.model_mail_message'),
        )

        # Step 3: Retrieve info on other models
        blacklisted_models = self._get_query_models_blacklist()
        for model_name in self.env:
            if model_name in blacklisted_models:
                continue

            model = self.env[model_name]
            if model._transient or not model._auto:
                continue

            table_name = model._table

            conditions = []
            # 3.1 Search Basic Personal Data Records (aka email/name usage)
            for field_name in ['email_normalized', 'email', 'email_from', 'company_email']:
                if field_name in model and model._fields[field_name].store:
                    rec_name = model._rec_name or 'name'
                    is_normalized = field_name == 'email_normalized' or (model_name == 'mailing.trace' and field_name == 'email')

                    conditions.append(SQL(
                        "%s %s %s",
                        SQL.identifier(field_name),
                        SQL('=') if is_normalized else SQL('ilike'),  # Manage Foo Bar <foo@bar.com>
                        email_normalized if is_normalized else email
                    ))
                    if rec_name in model and model._fields[model._rec_name].store and model._fields[model._rec_name].type == 'char' and not model._fields[model._rec_name].translate:
                        conditions.append(SQL(
                            "%s ilike %s",
                            SQL.identifier(rec_name),
                            name,
                        ))
                    if is_normalized:
                        break

            # 3.2 Search Indirect Personal Data References (aka partner_id)
            conditions.extend(
                SQL(
                    "%s in (SELECT id FROM indirect_references)",
                    model._field_to_sql(table_name, field_name),
                )
                for field_name, field in model._fields.items()
                if field.comodel_name == 'res.partner'
                if field.store
                if field.type == 'many2one'
                if field.ondelete != 'cascade'
            )

            if conditions:
                query = SQL("""
                    %s
                    UNION ALL
                    SELECT
                        %s AS res_model_id,
                        id AS res_id,
                        %s AS is_active
                    FROM %s
                    WHERE %s
                """,
                    query,
                    self.env['ir.model'].search([('model', '=', model_name)]).id,
                    SQL.identifier('active') if 'active' in model else True,
                    SQL.identifier(table_name),
                    SQL(" OR ").join(conditions),
                )
        return query

    def action_lookup(self):
        self.ensure_one()
        query = self._get_query()
        self.env.flush_all()
        self.env.cr.execute(query)
        results = self.env.cr.dictfetchall()
        self.line_ids = [(5, 0, 0)] + [(0, 0, reference) for reference in results]
        return self.action_open_lines()

    def _post_log(self):
        self.ensure_one()
        if not self.log_id and self.execution_details:
            self.log_id = self.env['privacy.log'].create({
                'anonymized_name': self.name,
                'anonymized_email': self.email,
                'execution_details': self.execution_details,
                'records_description': self.records_description,
            })
        else:
            self.log_id.execution_details = self.execution_details
            self.log_id.records_description = self.records_description

    @api.depends('line_ids.execution_details')
    def _compute_execution_details(self):
        for wizard in self:
            wizard.execution_details = '\n'.join(line.execution_details for line in wizard.line_ids if line.execution_details)
            wizard._post_log()

    @api.depends('line_ids')
    def _compute_records_description(self):
        for wizard in self:
            if not wizard.line_ids:
                wizard.records_description = ''
                continue
            records_by_model = defaultdict(list)
            for line in wizard.line_ids:
                records_by_model[line.res_model_id].append(line.res_id)
            wizard.records_description = '\n'.join('{model_name} ({count}): {ids_str}'.format(
                model_name=(f'{model.name} - {model.model}' if self.env.user.has_group('base.group_no_one') else model.name),
                count=len(ids),
                ids_str=', '.join('#%s' % (rec_id) for rec_id in ids),
            ) for model, ids in records_by_model.items())

    def action_open_lines(self):
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id('privacy_lookup.action_privacy_lookup_wizard_line')
        action['domain'] = [('wizard_id', '=', self.id)]
        return action


class PrivacyLookupWizardLine(models.TransientModel):
    _name = 'privacy.lookup.wizard.line'
    _description = 'Privacy Lookup Wizard Line'
    _transient_max_count = 0
    _transient_max_hours = 24

    @api.model
    def _selection_target_model(self):
        return [(model.model, model.name) for model in self.env['ir.model'].sudo().search([])]

    wizard_id = fields.Many2one('privacy.lookup.wizard')
    res_id = fields.Integer(
        string="Resource ID",
        required=True)
    res_name = fields.Char(
        string='Resource name',
        compute='_compute_res_name',
        store=True)
    res_model_id = fields.Many2one(
        'ir.model',
        'Related Document Model',
        ondelete='cascade')
    res_model = fields.Char(
        string='Document Model',
        related='res_model_id.model',
        store=True,
        readonly=True)
    resource_ref = fields.Reference(
        string='Record',
        selection='_selection_target_model',
        compute='_compute_resource_ref',
        inverse='_set_resource_ref')
    has_active = fields.Boolean(compute='_compute_has_active', store=True)
    is_active = fields.Boolean()
    is_unlinked = fields.Boolean()
    execution_details = fields.Char(default='')

    @api.depends('res_model', 'res_id', 'is_unlinked')
    def _compute_resource_ref(self):
        for line in self:
            if line.res_model and line.res_model in self.env and not line.is_unlinked:
                # Exclude records that can't be read (eg: multi-company ir.rule)
                try:
                    self.env[line.res_model].browse(line.res_id).check_access('read')
                    line.resource_ref = '%s,%s' % (line.res_model, line.res_id or 0)
                except Exception:
                    line.resource_ref = None
            else:
                line.resource_ref = None

    def _set_resource_ref(self):
        for line in self:
            if line.resource_ref:
                line.res_id = line.resource_ref.id

    @api.depends('res_model_id')
    def _compute_has_active(self):
        for line in self:
            if not line.res_model_id:
                line.has_active = False
                continue
            line.has_active = 'active' in self.env[line.res_model]

    @api.depends('res_model', 'res_id')
    def _compute_res_name(self):
        for line in self:
            if not line.res_id or not line.res_model:
                continue
            record = self.env[line.res_model].sudo().browse(line.res_id)
            if not record.exists():
                continue
            name = record.display_name
            line.res_name = name if name else f'{line.res_model_id.name}/{line.res_id}'

    @api.onchange('is_active')
    def _onchange_is_active(self):
        for line in self:
            if not line.res_model_id or not line.res_id:
                continue
            action = _('Unarchived') if line.is_active else _('Archived')
            line.execution_details = '%s %s #%s' % (action, line.res_model_id.name, line.res_id)
            self.env[line.res_model].sudo().browse(line.res_id).write({'active': line.is_active})

    def action_unlink(self):
        self.ensure_one()
        if self.is_unlinked:
            raise UserError(_('The record is already unlinked.'))
        self.env[self.res_model].sudo().browse(self.res_id).unlink()
        self.execution_details = '%s %s #%s' % (_('Deleted'), self.res_model_id.name, self.res_id)
        self.is_unlinked = True

    def action_archive_all(self):
        for line in self:
            if not line.has_active or not line.is_active:
                continue
            line.is_active = False
            line._onchange_is_active()

    def action_unlink_all(self):
        for line in self:
            if line.is_unlinked:
                continue
            line.action_unlink()

    def action_open_record(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_id': self.res_id,
            'res_model': self.res_model,
        }

```

## File: wizard\privacy_lookup_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="privacy_lookup_wizard_view_form" model="ir.ui.view">
        <field name="name">privacy.lookup.wizard.view.form</field>
        <field name="model">privacy.lookup.wizard</field>
        <field name="arch" type="xml">
            <form string="Privacy Lookup">
                <header>
                    <button string="Lookup" name="action_lookup" type="object" class="oe_highlight" data-hotkey="q" invisible="line_ids"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button
                            name="action_open_lines"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-file-text-o">
                            <field name="line_count" string="References" widget="statinfo"/>
                        </button>
                    </div>
                    <group>
                        <group>
                            <field name="email" readonly="line_ids"/>
                            <field name="name" readonly="line_ids"/>
                            <field name="records_description" invisible="1"/>
                            <field name="execution_details" invisible="execution_details == ''"/>
                            <field name="log_id" invisible="1"/>
                        </group>
                        <field name="line_ids" options="{'no_open': True, 'no_create': True}" invisible="1"/>
                    </group>
                </sheet>
           </form>
        </field>
    </record>

    <record id="action_privacy_lookup_wizard" model="ir.actions.act_window">
        <field name="name">Privacy Lookup</field>
        <field name="res_model">privacy.lookup.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="privacy_lookup_wizard_view_form"/>
        <field name="target">current</field>
    </record>

    <record id="privacy_lookup_wizard_line_view_tree" model="ir.ui.view">
        <field name="name">privacy.lookup.wizard.line.view.list</field>
        <field name="model">privacy.lookup.wizard.line</field>
        <field name="arch" type="xml">
            <list create="0">
                <field name="res_model_id"/>
                <field name="res_name" column_invisible="True"/>
                <field name="res_model" column_invisible="True"/>
                <field name="resource_ref" options="{'model_field': 'res_model_id', 'no_create': True}" invisible="not resource_ref"/>
                <button name="action_open_record" type="object" icon="fa-external-link" invisible="not resource_ref" title="Open Record"/>
                <field name="res_id" string="ID"/>
                <field name="has_active" column_invisible="True"/>
                <field name="execution_details" column_invisible="True"/>
                <field name="is_active" widget="boolean_toggle" invisible="is_unlinked or not has_active"/>
                <field name="is_unlinked" column_invisible="True"/>
                <button name="action_unlink" string="Delete" type="object" icon="fa-trash" invisible="is_unlinked" confirm="This operation is irreversible. Do you wish to proceed to the record deletion?"/>
            </list>
        </field>
    </record>

    <record id="privacy_lookup_wizard_line_view_search" model="ir.ui.view">
        <field name="name">privacy.lookup.wizard.line.view.search</field>
        <field name="model">privacy.lookup.wizard.line</field>
        <field name="arch" type="xml">
            <search string="Search References">
                <field name="res_model_id"/>
                <field name="has_active" invisible="1"/>
                <field name="is_active" invisible="1"/>
                <filter string="Can be archived" name="can_be_archived" domain="[('has_active', '=', True)]"/>
                <filter string="Archived" name="archived" domain="[('is_active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Model" name="group_by_res_model_id" domain="[]" context="{'group_by': 'res_model_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_privacy_lookup_wizard_line" model="ir.actions.act_window">
        <field name="name">Privacy Lookup Line</field>
        <field name="res_model">privacy.lookup.wizard.line</field>
        <field name="view_mode">list</field>
        <field name="view_id" ref="privacy_lookup_wizard_line_view_tree"/>
        <field name="context">{'search_default_group_by_res_model_id': 1, 'no_create_edit': True}</field>
        <field name="target">current</field>
    </record>

    <record id="ir_action_server_action_privacy_lookup_partner" model="ir.actions.server">
        <field name="name">Privacy Lookup</field>
        <field name="model_id" ref="base.model_res_partner"/>
        <field name="binding_model_id" ref="base.model_res_partner"/>
        <field name="binding_view_types">form</field>
        <field name="groups_id" eval="[Command.link(ref('base.group_system'))]" />
        <field name="state">code</field>
        <field name="code">
action = record.action_privacy_lookup()
        </field>
    </record>

    <record id="ir_action_server_action_privacy_lookup_user" model="ir.actions.server">
        <field name="name">Privacy Lookup</field>
        <field name="model_id" ref="base.model_res_users"/>
        <field name="binding_model_id" ref="base.model_res_users"/>
        <field name="binding_view_types">form</field>
        <field name="groups_id" eval="[Command.link(ref('base.group_system'))]" />
        <field name="state">code</field>
        <field name="code">
action = record.partner_id.action_privacy_lookup()
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import privacy_lookup_wizard

```

