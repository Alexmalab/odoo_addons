# Odoo Module: data_recycle

Category: Productivity/Data Cleaning

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Data Recycle',
    'version': '1.3',
    'category': 'Productivity/Data Cleaning',
    'summary': 'Find old records and archive/delete them',
    'description': """Find old records and archive/delete them""",
    'depends': ['mail'],
    'data': [
        'data/ir_cron_data.xml',
        'views/data_recycle_model_views.xml',
        'views/data_recycle_record_views.xml',
        'views/data_cleaning_menu.xml',
        'views/data_recycle_templates.xml',
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'data_recycle/static/src/views/*.js',
            'data_recycle/static/src/views/*.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_clean_records" model="ir.cron">
        <field name="name">Data Recycle: Clean Records</field>
        <field name="model_id" ref="model_data_recycle_model"/>
        <field name="state">code</field>
        <field name="code">model._cron_recycle_records()</field>
        <field name="active" eval="True"/>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
        <field name="nextcall" eval="(DateTime.now().replace(hour=3, minute=0) + timedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')" />
    </record>
</odoo>

```

## File: models\data_recycle_model.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast

from collections import defaultdict
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import config, split_every
from odoo.osv import expression

# When recycle_mode = automatic, _recycle_records calls action_validate.
# This is quite slow so requires smaller batch size.
DR_CREATE_STEP_AUTO = 5000
DR_CREATE_STEP_MANUAL = 50000


class DataRecycleModel(models.Model):
    _name = 'data_recycle.model'
    _description = 'Recycling Model'
    _order = 'name'

    active = fields.Boolean(default=True)
    name = fields.Char(
        compute='_compute_name', string='Name', readonly=False, store=True, required=True, copy=True)

    res_model_id = fields.Many2one('ir.model', string='Model', required=True, ondelete='cascade')
    res_model_name = fields.Char(
        related='res_model_id.model', string='Model Name', readonly=True, store=True)
    recycle_record_ids = fields.One2many('data_recycle.record', 'recycle_model_id')

    recycle_mode = fields.Selection([
        ('manual', 'Manual'),
        ('automatic', 'Automatic'),
    ], string='Recycle Mode', default='manual', required=True)
    recycle_action = fields.Selection([
        ('archive', 'Archive'),
        ('unlink', 'Delete'),
    ], string="Recycle Action", default='unlink', required=True)

    # Rule
    domain = fields.Char(string="Filter", compute='_compute_domain', readonly=False, store=True)
    time_field_id = fields.Many2one(
        'ir.model.fields', string='Time Field',
        domain="[('model_id', '=', res_model_id), ('ttype', 'in', ('date', 'datetime')), ('store', '=', True)]",
        ondelete='cascade')
    time_field_delta = fields.Integer(string='Delta', default=1)
    time_field_delta_unit = fields.Selection([
        ('days', 'Days'),
        ('weeks', 'Weeks'),
        ('months', 'Months'),
        ('years', 'Years')], string='Delta Unit', default='months')
    include_archived = fields.Boolean()

    records_to_recycle_count = fields.Integer(
        'Records To Recycle', compute='_compute_records_to_recycle_count')

    # User Notifications for Manual clean
    notify_user_ids = fields.Many2many(
        'res.users', string='Notify Users',
        domain=lambda self: [('groups_id', 'in', self.env.ref('base.group_system').id)],
        default=lambda self: self.env.user,
        help='List of users to notify when there are new records to recycle')
    notify_frequency = fields.Integer(string='Notify', default=1)
    notify_frequency_period = fields.Selection([
        ('days', 'Days'),
        ('weeks', 'Weeks'),
        ('months', 'Months')], string='Notify Frequency Period', default='weeks')
    last_notification = fields.Datetime(readonly=True)

    _sql_constraints = [
        ('check_notif_freq', 'CHECK(notify_frequency > 0)', 'The notification frequency should be greater than 0'),
    ]

    @api.constrains('recycle_action')
    def _check_recycle_action(self):
        for model in self:
            if model.recycle_action == 'archive' and 'active' not in self.env[model.res_model_name]:
                raise UserError(_("This model doesn't manage archived records. Only deletion is possible."))

    @api.depends('res_model_id')
    def _compute_domain(self):
        self.domain = '[]'

    @api.depends('res_model_id')
    def _compute_name(self):
        for model in self:
            if model.name:
                continue
            model.name = model.res_model_id.name if model.res_model_id else ''

    def _compute_records_to_recycle_count(self):
        count_data = self.env['data_recycle.record']._read_group(
            [('recycle_model_id', 'in', self.ids)],
            ['recycle_model_id'],
            ['__count'])
        counts = {recycle_model.id: count for recycle_model, count in count_data}
        for model in self:
            model.records_to_recycle_count = counts[model.id] if model.id in counts else 0

    def _cron_recycle_records(self):
        self.sudo().search([])._recycle_records(batch_commits=True)
        self.sudo()._notify_records_to_recycle()

    def _recycle_records(self, batch_commits=False):
        self.env.flush_all()
        records_to_clean = []
        is_test = bool(config['test_enable'] or config['test_file'])

        existing_recycle_records = self.env['data_recycle.record'].with_context(
            active_test=False).search([('recycle_model_id', 'in', self.ids)])
        mapped_existing_records = defaultdict(list)
        for recycle_record in existing_recycle_records:
            mapped_existing_records[recycle_record.recycle_model_id].append(recycle_record.res_id)

        for recycle_model in self:
            rule_domain = ast.literal_eval(recycle_model.domain) if recycle_model.domain and recycle_model.domain != '[]' else []
            if recycle_model.time_field_id and recycle_model.time_field_delta and recycle_model.time_field_delta_unit:
                if recycle_model.time_field_id.ttype == 'date':
                    now = fields.Date.today()
                else:
                    now = fields.Datetime.now()
                delta = relativedelta(**{recycle_model.time_field_delta_unit: recycle_model.time_field_delta})
                rule_domain = expression.AND([rule_domain, [(recycle_model.time_field_id.name, '<=', now - delta)]])
            model = self.env[recycle_model.res_model_name]
            if recycle_model.include_archived:
                model = model.with_context(active_test=False)
            records_to_recycle = model.search(rule_domain)
            records_to_create = [{
                'res_id': record.id,
                'recycle_model_id': recycle_model.id,
            } for record in records_to_recycle if record.id not in mapped_existing_records[recycle_model]]

            if recycle_model.recycle_mode == 'automatic':
                for records_to_create_batch in split_every(DR_CREATE_STEP_AUTO, records_to_create):
                    self.env['data_recycle.record'].create(records_to_create_batch).action_validate()
                    if batch_commits and not is_test:
                        # Commit after each batch iteration to avoid complete rollback on timeout as
                        # this can create lots of new records.
                        self.env.cr.commit()
            else:
                records_to_clean = records_to_clean + records_to_create
        for records_to_clean_batch in split_every(DR_CREATE_STEP_MANUAL, records_to_clean):
            self.env['data_recycle.record'].create(records_to_clean_batch)
            if batch_commits and not is_test:
                self.env.cr.commit()

    @api.model
    def _notify_records_to_recycle(self):
        for recycle in self.search([('recycle_mode', '=', 'manual')]):
            if not recycle.notify_user_ids or not recycle.notify_frequency:
                continue

            if recycle.notify_frequency_period == 'days':
                delta = relativedelta(days=recycle.notify_frequency)
            elif recycle.notify_frequency_period == 'weeks':
                delta = relativedelta(weeks=recycle.notify_frequency)
            else:
                delta = relativedelta(months=recycle.notify_frequency)

            if not recycle.last_notification or\
                    (recycle.last_notification + delta) < fields.Datetime.now():
                recycle.last_notification = fields.Datetime.now()
                recycle._send_notification(delta)

    def _send_notification(self, delta):
        self.ensure_one()
        last_date = fields.Date.today() - delta
        records_count = self.env['data_recycle.record'].search_count([
            ('recycle_model_id', '=', self.id),
            ('create_date', '>=', last_date)
        ])
        partner_ids = self.notify_user_ids.partner_id.ids if records_count else []
        if partner_ids:
            menu_id = self.env.ref('data_recycle.menu_data_cleaning_root').id
            self.env['mail.thread'].message_notify(
                body=self.env['ir.qweb']._render(
                    'data_recycle.notification',
                    {
                        'records_count': records_count,
                        'res_model_label': self.res_model_id.name,
                        'recycle_model_id': self.id,
                        'menu_id': menu_id
                    }
                ),
                model=self._name,
                notify_author=True,
                partner_ids=partner_ids,
                res_id=self.id,
                subject=_('Data to Recycle'),
            )

    def write(self, vals):
        if 'active' in vals and not vals['active']:
            self.env['data_recycle.record'].search([('recycle_model_id', 'in', self.ids)]).unlink()
        return super().write(vals)

    def open_records(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("data_recycle.action_data_recycle_record")
        action['context'] = dict(ast.literal_eval(action.get('context')), searchpanel_default_recycle_model_id=self.id)
        return action

    def action_recycle_records(self):
        self.sudo()._recycle_records()
        if self.recycle_mode == 'manual':
            return self.open_records()
        return

```

## File: models\data_recycle_record.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import models, api, fields, _


class DataRecycleRecord(models.Model):
    _name = 'data_recycle.record'
    _description = 'Recycling Record'

    active = fields.Boolean('Active', default=True)
    name = fields.Char('Record Name', compute='_compute_name', compute_sudo=True)
    recycle_model_id = fields.Many2one('data_recycle.model', string='Recycle Model', ondelete='cascade')

    res_id = fields.Integer('Record ID', index=True)
    res_model_id = fields.Many2one(related='recycle_model_id.res_model_id', store=True, readonly=True)
    res_model_name = fields.Char(related='recycle_model_id.res_model_name', store=True, readonly=True)

    company_id = fields.Many2one('res.company', compute='_compute_company_id', store=True)

    @api.model
    def _get_company_id(self, record):
        company_id = self.env['res.company']
        if 'company_id' in self.env[record._name]:
            company_id = record.company_id
        return company_id

    @api.depends('res_id')
    def _compute_name(self):
        original_records = {(r._name, r.id): r for r in self._original_records()}
        for record in self:
            original_record = original_records.get((record.res_model_name, record.res_id))
            if original_record:
                record.name = original_record.display_name or _('Undefined Name')
            else:
                record.name = _('**Record Deleted**')

    @api.depends('res_id')
    def _compute_company_id(self):
        original_records = {(r._name, r.id): r for r in self._original_records()}
        for record in self:
            original_record = original_records.get((record.res_model_name, record.res_id))
            if original_record:
                record.company_id = self._get_company_id(original_record)
            else:
                record.company_id = self.env['res.company']

    def _original_records(self):
        if not self:
            return []

        records = []
        records_per_model = {}
        for record in self.filtered(lambda r: r.res_model_name):
            ids = records_per_model.get(record.res_model_name, [])
            ids.append(record.res_id)
            records_per_model[record.res_model_name] = ids

        for model, record_ids in records_per_model.items():
            recs = self.env[model].with_context(active_test=False).sudo().browse(record_ids).exists()
            records += [r for r in recs]
        return records

    def action_validate(self):
        records_done = self.env['data_recycle.record']
        record_ids_to_archive = defaultdict(list)
        record_ids_to_unlink = defaultdict(list)
        original_records = {'%s_%s' % (r._name, r.id): r for r in self._original_records()}
        for record in self:
            original_record = original_records.get('%s_%s' % (record.res_model_name, record.res_id))
            records_done |= record
            if not original_record:
                continue
            if record.recycle_model_id.recycle_action == "archive":
                record_ids_to_archive[original_record._name].append(original_record.id)
            elif record.recycle_model_id.recycle_action == "unlink":
                record_ids_to_unlink[original_record._name].append(original_record.id)
        for model_name, ids in record_ids_to_archive.items():
            self.env[model_name].sudo().browse(ids).toggle_active()
        for model_name, ids in record_ids_to_unlink.items():
            self.env[model_name].sudo().browse(ids).unlink()
        records_done.unlink()

    def action_discard(self):
        self.write({'active': False})

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import data_recycle_model
from . import data_recycle_record

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_data_recycle_model_group_system,access_data_recycle_model_group_system,model_data_recycle_model,base.group_system,1,1,1,1
access_data_recycle_record_group_system,access_data_recycle_record_group_system,model_data_recycle_record,base.group_system,1,1,1,1
```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="m15.17 41-2.293-8h24.246l-2.292 8H15.169Z" fill="#2EBCFA"/><path d="m11.723 28.974-.004-.015L13.847 27h8.618l14.714 5.805-.056.195h-12.63l-12.77-4.026ZM8.296 17.01l-.005-.017L16.889 14l19.265 7H18.917L8.296 17.01Z" fill="#005E7A"/><path d="M8.292 17 6 9h38l-2.292 8H8.292Z" fill="#144496"/><path d="m11.724 28.977-2.292-8h31.136l-2.292 8H11.724Z" fill="#088BF5"/></svg>

```

## File: static\src\views\data_cleaning_common_list.js

```javascript
/** @odoo-module **/

import { ListController } from "@web/views/list/list_controller";
import { useService } from "@web/core/utils/hooks";


export class DataCleaningCommonListController extends ListController {

    setup() {
        super.setup();
        this.orm = useService("orm");
        this.actionService = useService("action");
        this.notificationService = useService("notification");
    }

    /**
     * Open the form view of the original record, and not the data_merge.record view
     * @override
     */
    openRecord(record) {
        this.actionService.doAction({
            type: 'ir.actions.act_window',
            views: [[false, 'form']],
            res_model: record.data.res_model_name,
            res_id: record.data.res_id,
            context: {
                create: false,
                edit: false
            }
        });
    }

    /**
     * Unselect all the records
     */
    onUnselectClick() {
        this.discardSelection();
    }
};

```

## File: static\src\views\data_recycle_list_view.js

```javascript
/** @odoo-module **/

import { DataCleaningCommonListController } from "@data_recycle/views/data_cleaning_common_list";
import { registry } from '@web/core/registry';
import { listView } from '@web/views/list/list_view';

export class DataRecycleListController extends DataCleaningCommonListController {
    /**
     * Validate all the records selected
     */
    async onValidateClick() {
        const record_ids = await this.getSelectedResIds();

        await this.orm.call('data_recycle.record', 'action_validate', [record_ids]);
        await this.model.load();
    }
};

registry.category('views').add('data_recycle_list', {
    ...listView,
    Controller: DataRecycleListController,
    buttonTemplate: 'DataRecycle.buttons',
});


```

## File: static\src\views\data_recycle_list_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="DataRecycle.buttons" t-inherit="web.ListView.Buttons" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_list_buttons')]" position="inside">
            <t t-if="nbSelected">
                <button type="button" class="btn btn-primary o_data_recycle_validate_button" t-on-click="onValidateClick">
                    Validate
                </button>
                <button type="button" class="btn btn-secondary o_data_recycle_unselect_button" t-on-click="onUnselectClick">
                    Unselect
                </button>
            </t>
        </xpath>
    </t>
</templates>

```

## File: views\data_cleaning_menu.xml

```xml
<?xml version="1.0"?>
<odoo>
    <menuitem
        id="menu_data_cleaning_root"
        name="Data Cleaning"
        web_icon="data_recycle,static/description/icon.png"
        sequence="250"/>

        <menuitem
            id="menu_data_recycle_record"
            name="Recycle Records"
            parent="menu_data_cleaning_root"
            action="action_data_recycle_record"
            sequence="10"/>

        <menuitem
            id="menu_data_cleaning_config"
            name="Configuration"
            parent="menu_data_cleaning_root"
            sequence="100"/>

            <menuitem
                id="menu_data_cleaning_config_rules"
                name="Rules"
                parent="menu_data_cleaning_config"
                sequence="1"/>

                <menuitem
                    id="menu_data_cleaning_config_rules_recycle"
                    name="Recycle Records"
                    parent="menu_data_cleaning_config_rules"
                    action="action_data_recycle_config"
                    sequence="10"/>
</odoo>

```

## File: views\data_recycle_model_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="view_data_recycle_model_list">
            <field name="name">Field Recyle Model List</field>
            <field name="model">data_recycle.model</field>
            <field name="arch" type="xml">
                <tree decoration-muted="not active">
                    <field name="name" />
                    <field name="res_model_id" />
                    <field name="recycle_mode" groups="base.group_no_one" />
                    <field name="recycle_action" groups="base.group_no_one" />
                    <field name="active" widget="boolean_toggle" />
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="view_data_merge_model_form">
            <field name="name">Field Recyle Model Form</field>
            <field name="model">data_recycle.model</field>
            <field name="arch" type="xml">
                <form>
                    <header>
                        <button name="action_recycle_records" type="object" string="Run Now" class="oe_highlight" />
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" name="open_records"
                                    type="object" icon="fa-bars">
                                <field name="records_to_recycle_count" string="Records" widget="statinfo"/>
                            </button>
                        </div>
                        <div class="oe_title">
                            <h1>
                                <field name="name" />
                            </h1>
                        </div>
                        <group>
                            <group>
                                <field name="res_model_id" options="{'no_create': True, 'no_open': True}" />
                                <field name="res_model_name" invisible="1" />
                                <field name="active" widget="boolean_toggle" />
                            </group>
                            <group>
                                <field name="recycle_mode" widget="radio" options="{'horizontal': true}" />
                                <field name="recycle_action" widget="radio" options="{'horizontal': true}" />
                                <field name="include_archived" invisible="recycle_action != 'unlink'"/>

                                <!-- Manual cleaning -->
                                <label for="notify_user_ids" invisible="recycle_mode == 'automatic'" />
                                <div invisible="recycle_mode == 'automatic'">
                                    <field name="notify_user_ids" widget="many2many_tags"  options="{'no_create': True, 'no_edit': True}" domain="[('share', '=', False)]" nolabel="1"/>
                                    <div class="d-flex w-50" invisible="not notify_user_ids">
                                        <span class="me-1">Every</span>
                                        <field name="notify_frequency" required="notify_user_ids" />
                                        <field name="notify_frequency_period" required="notify_user_ids" />
                                    </div>
                                </div>
                            </group>
                        </group>
                        <group invisible="res_model_id">
                            <group>
                                <div class="alert alert-info" role="alert" colspan="2">
                                    Select a model to configure recycling actions
                                </div>
                            </group>
                        </group>
                        <group invisible="not res_model_id">
                            <field name="domain" widget="domain" options="{'model': 'res_model_name'}"/>
                            <field name="time_field_id"/>
                            <field name="time_field_delta"/>
                            <field name="time_field_delta_unit"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_data_recycle_config">
            <field name="name">Recyle Records Rules</field>
            <field name="res_model">data_recycle.model</field>
            <field name="view_mode">tree,form</field>
            <field name="domain">['|', ('active', '=', False), ('active', '=', True)]</field>
        </record>
    </data>
</odoo>

```

## File: views\data_recycle_record_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="view_data_recycle_record_list">
            <field name="name">Field Recycle Record List</field>
            <field name="model">data_recycle.record</field>
            <field name="arch" type="xml">
                <tree js_class="data_recycle_list" sample="1" create="0" export_xlsx="0">
                    <field name="active" column_invisible="True" />
                    <field name="res_model_name" column_invisible="True" />
                    <field name="res_id" />
                    <field name="recycle_model_id" string="Recycle Rule" optional="hide" />
                    <field name="name" />
                    <button icon="fa-check" string="Validate" type="object" name="action_validate" />
                    <button icon="fa-times" string="Discard" type="object" name="action_discard" invisible="not active" />
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="view_data_recycle_record_search">
            <field name="name">Field Recycle Record Search</field>
            <field name="model">data_recycle.record</field>
            <field name="arch" type="xml">
                <search string="Records">
                    <filter name="active" string="Discarded" domain="[('active', '=', False)]" />
                    <searchpanel>
                        <field name="recycle_model_id" icon="fa-bars" string="Recycle Rules" />
                    </searchpanel>
                </search>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_data_recycle_record">
            <field name="name">Field Recycle Records</field>
            <field name="res_model">data_recycle.record</field>
            <field name="view_mode">tree,form</field>
            <field name="domain"></field>
            <field name="search_view_id" ref="view_data_recycle_record_search" />
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No cleaning suggestions
              </p>
              <p>
              Configure rules to identify records to clean
              </p>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_data_recycle_record_notification">
            <field name="name">Field Recycle Records</field>
            <field name="res_model">data_recycle.record</field>
            <field name="view_mode">tree,form</field>
            <field name="context">{ 'searchpanel_default_recycle_model_id': active_id }</field>
            <field name="search_view_id" ref="view_data_recycle_record_search" />
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No cleaning suggestions
              </p>
              <p>
              Configure rules to identify records to clean
              </p>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\data_recycle_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="notification">
We've identified <t t-esc="records_count" /> records to clean with the '<t t-esc="res_model_label" />' recycling rule.<br/>
You can validate those changes <a t-attf-href="/web?#action=data_recycle.action_data_recycle_record_notification&amp;active_id={{recycle_model_id}}&amp;menu_id={{menu_id}}">here</a>.
    </template>
</odoo>

```

