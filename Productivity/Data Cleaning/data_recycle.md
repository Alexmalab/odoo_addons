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
    'auto_install': False,
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
            ['recycle_model_id'])
        counts = {cd['recycle_model_id'][0]: cd['recycle_model_id_count'] for cd in count_data}
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

        if records_count:
            partner_ids = self.notify_user_ids.partner_id.ids
            menu_id = self.env.ref('data_recycle.menu_data_cleaning_root').id
            kwargs = {
                'body': self.env['ir.qweb']._render('data_recycle.notification', {
                    'records_count': records_count,
                    'res_model_label': self.res_model_id.name,
                    'recycle_model_id': self.id,
                    'menu_id': menu_id
                }),
                'partner_ids': partner_ids,
            }
            self.env['mail.thread'].with_context(mail_notify_author=True).message_notify(**kwargs)

    def write(self, vals):
        if 'active' in vals and not vals['active']:
            self.env['data_recycle.record'].search([('recycle_model_id', 'in', self.ids)]).unlink()
        super().write(vals)

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
    name = fields.Char('Record Name', compute='_compute_values', compute_sudo=True)
    recycle_model_id = fields.Many2one('data_recycle.model', string='Recycle Model', ondelete='cascade')

    res_id = fields.Integer('Record ID', index=True)
    res_model_id = fields.Many2one(related='recycle_model_id.res_model_id', store=True, readonly=True)
    res_model_name = fields.Char(related='recycle_model_id.res_model_name', store=True, readonly=True)

    company_id = fields.Many2one('res.company', compute='_compute_values', store=True)

    @api.model
    def _get_company_id(self, record):
        company_id = self.env['res.company']
        if 'company_id' in self.env[record._name]:
            company_id = record.company_id
        return company_id

    @api.depends('res_id')
    def _compute_values(self):
        original_records = {'%s_%s' % (r._name, r.id): r for r in self._original_records()}
        for record in self:
            original_record = original_records.get('%s_%s' % (record.res_model_name, record.res_id))
            if original_record:
                record.company_id = self._get_company_id(original_record)
                record.name = original_record.display_name or _('Undefined Name')
            else:
                record.company_id = self.env['res.company']
                record.name = _('**Record Deleted**')

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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 140 140"><defs><style>.cls-1,.cls-4,.cls-8{fill:#fff;}.cls-1,.cls-3,.cls-4,.cls-6{fill-rule:evenodd;}.cls-2{mask:url(#mask);}.cls-3{fill:url(#linear-gradient);}.cls-4,.cls-6{fill-opacity:0.38;}.cls-5{fill:#393939;opacity:0.32;}.cls-5,.cls-7{isolation:isolate;}.cls-7{opacity:0.3;}</style><mask id="mask" x="0" y="0" width="140" height="140" maskUnits="userSpaceOnUse"><g id="b"><path id="a" class="cls-1" d="M8,0H130c8,0,10,2,10,10V130c0,8-2,10-10,10H8c-6,0-8-2-8-10V10C0,2,2,0,8,0Z"/></g></mask><linearGradient id="linear-gradient" x1="-1641.82" y1="-64.72" x2="-1643.82" y2="-66.72" gradientTransform="matrix(70, 0, 0, -70, 115067.27, -4530.13)" gradientUnits="userSpaceOnUse"><stop offset="0" stop-color="#94b6c8"/><stop offset="1" stop-color="#6a9eba"/></linearGradient></defs><title>Asset 41</title><g id="Layer_2" data-name="Layer 2"><g id="Layer_1-2" data-name="Layer 1"><g class="cls-2"><path class="cls-3" d="M0,0H140V140H0Z"/><path class="cls-4" d="M8,2H130q8,0,10,4V0H0V6Q2,2,8,2Z"/><path class="cls-5" d="M118.55,91.67,116,92.75l-4.57,5.06v-1l-1.09-1.37-.95-1.93L121.6,79.93l.09-4.15-.92-4.35-2.21-1.18L116.23,70l-8,8.83V74.31l14.11-15.73h-3.82l1-11.88-4.85,1.39-6.4,7v-5.2l12.86-14.26.43-7.92-3-2.95-1.31.37-1.73.73-4.28,4.53-2.65-5.08-.74.25-.53-1.38L89.25,42.11,68.49,69.19l2.16,8.23L51.77,67,25.18,90.67l-9.62-4.54L32.38,67.26V61.63l9.37-10.44h-1l5.65-6.25V43.49l4.69-4.56H41.75L33,48.29l-.62,2.9H23L0,76.63V130c0,6,4,8,8,8H89.57l31.52-34.93,1.56-6.22Z"/><polygon class="cls-5" points="32.38 24.64 0 60.7 0 67.55 38.62 24.64 32.38 24.64"/><path class="cls-6" d="M8,138H130q8,0,10-6v8H0v-8Q2,138,8,138Z"/><g class="cls-7"><path d="M116.66,38.37a3.69,3.69,0,0,0,4.94-1.17,6.53,6.53,0,0,0,1.05-3.51V31.63a6.07,6.07,0,0,0-.7-2.85,4.74,4.74,0,0,0-1.5-1.83,3.47,3.47,0,0,0-1.9-.49,3.36,3.36,0,0,0-3,1.67,6.45,6.45,0,0,0-1.06,3.5v2.06a6.19,6.19,0,0,0,.7,2.85A4.64,4.64,0,0,0,116.66,38.37Zm-.31-6.74a4.14,4.14,0,0,1,.81-2.6,1.68,1.68,0,0,1,1.39-.7,1.64,1.64,0,0,1,1.37.63,4.36,4.36,0,0,1,.85,2.67v2.06A4.09,4.09,0,0,1,120,36.3a1.66,1.66,0,0,1-1.38.7,1.61,1.61,0,0,1-1.36-.63,4.34,4.34,0,0,1-.86-2.68Z"/><path d="M98.92,59.21H97V48.94L93,50a1.68,1.68,0,0,0-.81.38A.84.84,0,0,0,92,51a1,1,0,0,0,.26.69.83.83,0,0,0,.62.29,2.76,2.76,0,0,0,.6-.11l1.68-.44v7.82h-1.9a1.43,1.43,0,0,0-1,.26.88.88,0,0,0-.29.67.9.9,0,0,0,.29.69,1.43,1.43,0,0,0,1,.26h5.67a1.43,1.43,0,0,0,1-.26.94.94,0,0,0,0-1.36A1.43,1.43,0,0,0,98.92,59.21Z"/><path d="M110.16,59.21h-1.89V48.94l-4,1.05a1.69,1.69,0,0,0-.8.38.84.84,0,0,0-.19.59,1,1,0,0,0,.26.69.81.81,0,0,0,.62.29,2.76,2.76,0,0,0,.6-.11l1.67-.44v7.82H104.5a1.48,1.48,0,0,0-1,.26.88.88,0,0,0-.28.67.89.89,0,0,0,.28.69,1.48,1.48,0,0,0,1,.26h5.66a1.44,1.44,0,0,0,1-.26.94.94,0,0,0,0-1.36A1.44,1.44,0,0,0,110.16,59.21Z"/><path d="M115.75,59.21a1.44,1.44,0,0,0-1,.26.88.88,0,0,0-.29.67.9.9,0,0,0,.29.69,1.44,1.44,0,0,0,1,.26h5.66a1.48,1.48,0,0,0,1-.26,1,1,0,0,0,0-1.36,1.48,1.48,0,0,0-1-.26h-1.89V48.94l-4,1.05a1.68,1.68,0,0,0-.81.38.89.89,0,0,0-.19.59,1,1,0,0,0,.26.69.83.83,0,0,0,.62.29,2.72,2.72,0,0,0,.61-.11l1.67-.44v7.82Z"/><path d="M82.92,83.35a3.52,3.52,0,0,0,1.92.5,3.37,3.37,0,0,0,3-1.68,6.48,6.48,0,0,0,1.06-3.5V76.61a6,6,0,0,0-.71-2.85,4.82,4.82,0,0,0-1.49-1.83,3.53,3.53,0,0,0-1.91-.5,3.36,3.36,0,0,0-3,1.68,6.52,6.52,0,0,0-1.05,3.5v2.06a6,6,0,0,0,.7,2.84A4.6,4.6,0,0,0,82.92,83.35Zm-.31-6.74A4.11,4.11,0,0,1,83.43,74a1.68,1.68,0,0,1,1.39-.69,1.62,1.62,0,0,1,1.36.63A4.32,4.32,0,0,1,87,76.61v2.06a4.15,4.15,0,0,1-.81,2.61,1.68,1.68,0,0,1-1.38.69,1.61,1.61,0,0,1-1.37-.63,4.29,4.29,0,0,1-.86-2.67Z"/><path d="M98.92,81.7H97V71.43l-4,1a1.68,1.68,0,0,0-.81.38.84.84,0,0,0-.19.59,1,1,0,0,0,.26.69.82.82,0,0,0,.62.28,2.25,2.25,0,0,0,.6-.11l1.68-.43V81.7h-1.9a1.43,1.43,0,0,0-1,.26.93.93,0,0,0,0,1.35,1.38,1.38,0,0,0,1,.26h5.67a1.38,1.38,0,0,0,1-.26.82.82,0,0,0,.29-.67.84.84,0,0,0-.29-.68A1.43,1.43,0,0,0,98.92,81.7Z"/><path d="M110.16,81.7h-1.89V71.43l-4,1a1.69,1.69,0,0,0-.8.38.84.84,0,0,0-.19.59,1,1,0,0,0,.26.69.8.8,0,0,0,.62.28,2.25,2.25,0,0,0,.6-.11l1.67-.43V81.7H104.5a1.48,1.48,0,0,0-1,.26,1,1,0,0,0,0,1.35,1.43,1.43,0,0,0,1,.26h5.66a1.39,1.39,0,0,0,1-.26.86.86,0,0,0,.29-.67.88.88,0,0,0-.29-.68A1.44,1.44,0,0,0,110.16,81.7Z"/><path d="M120.45,71.93a3.47,3.47,0,0,0-1.9-.5,3.35,3.35,0,0,0-3,1.68,6.45,6.45,0,0,0-1.06,3.5v2.06a6.12,6.12,0,0,0,.7,2.84,4.61,4.61,0,0,0,1.49,1.84,3.5,3.5,0,0,0,1.91.5,3.39,3.39,0,0,0,3-1.68,6.47,6.47,0,0,0,1.05-3.5V76.61a6,6,0,0,0-.7-2.85A4.74,4.74,0,0,0,120.45,71.93Zm.32,6.74a4.09,4.09,0,0,1-.82,2.61,1.68,1.68,0,0,1-1.38.69,1.61,1.61,0,0,1-1.36-.63,4.29,4.29,0,0,1-.86-2.67V76.61a4.17,4.17,0,0,1,.81-2.61,1.7,1.7,0,0,1,1.39-.69,1.64,1.64,0,0,1,1.37.63,4.32,4.32,0,0,1,.85,2.67Z"/><path d="M87.67,104.19H85.78V93.92l-4,1a1.68,1.68,0,0,0-.8.38.86.86,0,0,0-.2.59,1,1,0,0,0,.27.7.8.8,0,0,0,.61.28,2.24,2.24,0,0,0,.61-.11l1.67-.44v7.83H82a1.41,1.41,0,0,0-1,.26.93.93,0,0,0,0,1.35,1.41,1.41,0,0,0,1,.26h5.66a1.43,1.43,0,0,0,1-.26.86.86,0,0,0,.29-.67.88.88,0,0,0-.29-.68A1.43,1.43,0,0,0,87.67,104.19Z"/><path d="M98.92,104.19H97V93.92l-4,1a1.78,1.78,0,0,0-.81.38.85.85,0,0,0-.19.59,1,1,0,0,0,.26.7.82.82,0,0,0,.62.28,2.25,2.25,0,0,0,.6-.11l1.68-.44v7.83h-1.9a1.38,1.38,0,0,0-1,.26.93.93,0,0,0,0,1.35,1.38,1.38,0,0,0,1,.26h5.67a1.38,1.38,0,0,0,1-.26.82.82,0,0,0,.29-.67.84.84,0,0,0-.29-.68A1.38,1.38,0,0,0,98.92,104.19Z"/><path d="M109.21,94.42a3.5,3.5,0,0,0-1.91-.5,3.36,3.36,0,0,0-3,1.68,6.52,6.52,0,0,0-1,3.5v2.06a6,6,0,0,0,.7,2.84,4.68,4.68,0,0,0,1.48,1.84,3.52,3.52,0,0,0,1.92.5,3.36,3.36,0,0,0,3-1.68,6.4,6.4,0,0,0,1.06-3.5V99.1a6,6,0,0,0-.71-2.85A4.65,4.65,0,0,0,109.21,94.42Zm.31,6.74a4.15,4.15,0,0,1-.81,2.61,1.68,1.68,0,0,1-1.38.69,1.62,1.62,0,0,1-1.37-.63,4.33,4.33,0,0,1-.86-2.67V99.1a4.09,4.09,0,0,1,.82-2.61,1.68,1.68,0,0,1,1.39-.69,1.62,1.62,0,0,1,1.36.63,4.32,4.32,0,0,1,.85,2.67Z"/><path d="M120.45,94.42a3.47,3.47,0,0,0-1.9-.5,3.35,3.35,0,0,0-3,1.68,6.45,6.45,0,0,0-1.06,3.5v2.06a6.12,6.12,0,0,0,.7,2.84,4.69,4.69,0,0,0,1.49,1.84,3.5,3.5,0,0,0,1.91.5,3.39,3.39,0,0,0,3-1.68,6.47,6.47,0,0,0,1.05-3.5V99.1a6,6,0,0,0-.7-2.85A4.66,4.66,0,0,0,120.45,94.42Zm.32,6.74a4.09,4.09,0,0,1-.82,2.61,1.68,1.68,0,0,1-1.38.69,1.61,1.61,0,0,1-1.36-.63,4.33,4.33,0,0,1-.86-2.67V99.1a4.15,4.15,0,0,1,.81-2.61,1.7,1.7,0,0,1,1.39-.69,1.64,1.64,0,0,1,1.37.63,4.32,4.32,0,0,1,.85,2.67Z"/><path d="M76.43,104.19h-1.9V99.57a39.23,39.23,0,0,1-1.87,3.85v.77h-.42c-.38.65-.78,1.26-1.18,1.87h5.37a1.38,1.38,0,0,0,1-.26.86.86,0,0,0,.29-.67.88.88,0,0,0-.29-.68A1.38,1.38,0,0,0,76.43,104.19Z"/><path d="M110.7,28.78A4.73,4.73,0,0,0,109.21,27a3.6,3.6,0,0,0-1.88-.49l-4.1,4.62v2.61a6.07,6.07,0,0,0,.7,2.85,4.63,4.63,0,0,0,1.48,1.83,3.43,3.43,0,0,0,1.92.5,3.37,3.37,0,0,0,3-1.67,6.45,6.45,0,0,0,1.06-3.51V31.63A6.08,6.08,0,0,0,110.7,28.78Zm-1.18,4.91a4.15,4.15,0,0,1-.81,2.61,1.66,1.66,0,0,1-1.38.7,1.62,1.62,0,0,1-1.37-.63,4.34,4.34,0,0,1-.86-2.68V31.63a4.08,4.08,0,0,1,.82-2.6,1.67,1.67,0,0,1,1.39-.7,1.62,1.62,0,0,1,1.36.63,4.36,4.36,0,0,1,.85,2.67Z"/><path d="M104,18.13a4,4,0,0,0-2-.52A3.17,3.17,0,0,0,99.36,19L66.93,67.53l-.11-.07a14.4,14.4,0,0,0-8-2.67,12,12,0,0,0-2.46.26c-4.76,1-8,4.78-9.16,6.35l-1.1,1.47,27.75,20,.86-2.17c2.43-6.15,4-12.25-2.4-18.83l33-49.51A3.07,3.07,0,0,0,104,18.13Z"/><path d="M54.66,82.31l-10-7.21L43.62,76C36.39,81.82,29,87.83,20,87.83a21.89,21.89,0,0,1-8.84-1.94l-.11,0a.34.34,0,0,0-.18.06.3.3,0,0,0-.1.29,53.8,53.8,0,0,0,1.53,5.87,2,2,0,0,0,1.29,1.32,24,24,0,0,0,7.77,1.28h.53c5.19-.14,13.24-2,22.12-10.07a.71.71,0,0,1,.46-.19.6.6,0,0,1,.45.19c.5.53,0,1.7-1.27,3-9,9-16,9.71-22.44,9.71a26.45,26.45,0,0,1-6.8-.86h-.09a.26.26,0,0,0-.21.1.32.32,0,0,0-.05.34A45.7,45.7,0,0,0,19.92,107a2,2,0,0,0,1.56.75h.18a32.61,32.61,0,0,0,6.65-1.3,22,22,0,0,0,7.41-3.76,14.12,14.12,0,0,0,1.51-1.38A6.53,6.53,0,0,1,38.65,100a.3.3,0,0,1,.15,0h0a.23.23,0,0,1,.17.11,1.76,1.76,0,0,1-.13,2.1c-3.75,4.69-8.8,7.32-15.45,8a.3.3,0,0,0-.26.21.31.31,0,0,0,.08.33A31.82,31.82,0,0,0,30,115.85a1.4,1.4,0,0,0,.7.18,1.52,1.52,0,0,0,.42-.06c7.66-2.24,20.81-7.64,22.9-18.57a.61.61,0,0,1,.39-.51l.28-.1.25.21a2.23,2.23,0,0,1,.29,1.68c-1.71,8.7-8.76,15.2-20.38,18.8a.27.27,0,0,0-.21.28.29.29,0,0,0,.19.29,35.88,35.88,0,0,0,12.1,2c.65,0,1.3,0,2-.05,1.83-.8,17.58-8.22,23-24.25a.31.31,0,0,0-.1-.35Z"/><path d="M32.38,69.51c-.68-10.05-1.8-13.4-9.37-16.07h0c7.57-2.92,8.69-5.15,9.37-16.07h0C33,47.9,34.38,51,41.75,53.44h0c-6.7,2.67-7.82,3.78-9.37,16.07"/><path d="M46.43,47.19c-.34-3.76-.9-5-4.68-6h0c3.78-1.09,4.34-1.93,4.68-6h0c.31,4,1,5.12,4.69,6h0c-3.35,1-3.91,1.42-4.69,6"/><path d="M35.5,30.47c-.23-2.24-.6-3-3.12-3.58h0c2.52-.65,2.89-1.15,3.12-3.58h0c.21,2.35.67,3.05,3.12,3.58h0c-2.23.59-2.6.84-3.12,3.58"/></g><path class="cls-8" d="M116.66,36.13a3.5,3.5,0,0,0,1.91.5,3.39,3.39,0,0,0,3-1.68,6.47,6.47,0,0,0,1.05-3.5V29.39a6,6,0,0,0-.7-2.85,4.71,4.71,0,0,0-1.5-1.84,3.47,3.47,0,0,0-1.9-.49,3.35,3.35,0,0,0-3,1.68,6.42,6.42,0,0,0-1.06,3.5v2.06a6.12,6.12,0,0,0,.7,2.84A4.69,4.69,0,0,0,116.66,36.13Zm-.31-6.74a4.15,4.15,0,0,1,.81-2.61,1.7,1.7,0,0,1,1.39-.69,1.64,1.64,0,0,1,1.37.63,4.32,4.32,0,0,1,.85,2.67v2.06a4.13,4.13,0,0,1-.82,2.61,1.68,1.68,0,0,1-1.38.69,1.61,1.61,0,0,1-1.36-.63,4.33,4.33,0,0,1-.86-2.67Z"/><path class="cls-8" d="M98.92,57H97V46.7l-4,1a1.78,1.78,0,0,0-.81.38.84.84,0,0,0-.19.59,1,1,0,0,0,.26.7.82.82,0,0,0,.62.28,2.76,2.76,0,0,0,.6-.11l1.68-.44V57h-1.9a1.38,1.38,0,0,0-1,.26.86.86,0,0,0-.29.67.88.88,0,0,0,.29.68,1.43,1.43,0,0,0,1,.26h5.67a1.43,1.43,0,0,0,1-.26.93.93,0,0,0,0-1.35A1.38,1.38,0,0,0,98.92,57Z"/><path class="cls-8" d="M110.16,57h-1.89V46.7l-4,1a1.79,1.79,0,0,0-.8.38.84.84,0,0,0-.19.59,1,1,0,0,0,.26.7.8.8,0,0,0,.62.28,2.76,2.76,0,0,0,.6-.11l1.67-.44V57H104.5a1.43,1.43,0,0,0-1,.26.85.85,0,0,0-.28.67.87.87,0,0,0,.28.68,1.48,1.48,0,0,0,1,.26h5.66a1.44,1.44,0,0,0,1-.26.93.93,0,0,0,0-1.35A1.39,1.39,0,0,0,110.16,57Z"/><path class="cls-8" d="M115.75,57a1.39,1.39,0,0,0-1,.26.86.86,0,0,0-.29.67.88.88,0,0,0,.29.68,1.44,1.44,0,0,0,1,.26h5.66a1.48,1.48,0,0,0,1-.26,1,1,0,0,0,0-1.35,1.43,1.43,0,0,0-1-.26h-1.89V46.7l-4,1a1.78,1.78,0,0,0-.81.38.89.89,0,0,0-.19.59,1,1,0,0,0,.26.7.82.82,0,0,0,.62.28,2.72,2.72,0,0,0,.61-.11l1.67-.44V57Z"/><path class="cls-8" d="M82.92,81.1a3.43,3.43,0,0,0,1.92.5,3.38,3.38,0,0,0,3-1.67,6.54,6.54,0,0,0,1.06-3.51v-2a6.09,6.09,0,0,0-.71-2.86,4.82,4.82,0,0,0-1.49-1.83,3.52,3.52,0,0,0-1.91-.49,3.37,3.37,0,0,0-3,1.67,6.53,6.53,0,0,0-1.05,3.51v2a6.07,6.07,0,0,0,.7,2.85A4.63,4.63,0,0,0,82.92,81.1Zm-.31-6.73a4.09,4.09,0,0,1,.82-2.61,1.67,1.67,0,0,1,1.39-.7,1.62,1.62,0,0,1,1.36.63A4.37,4.37,0,0,1,87,74.37v2A4.15,4.15,0,0,1,86.22,79a1.66,1.66,0,0,1-1.38.7,1.61,1.61,0,0,1-1.37-.63,4.34,4.34,0,0,1-.86-2.68Z"/><path class="cls-8" d="M98.92,79.45H97V69.19l-4,1a1.78,1.78,0,0,0-.81.38.84.84,0,0,0-.19.59,1,1,0,0,0,.26.69.83.83,0,0,0,.62.29,2.76,2.76,0,0,0,.6-.11l1.68-.44v7.82h-1.9a1.43,1.43,0,0,0-1,.26.94.94,0,0,0,0,1.36,1.43,1.43,0,0,0,1,.26h5.67a1.43,1.43,0,0,0,1-.26.85.85,0,0,0,.29-.67.86.86,0,0,0-.29-.69A1.43,1.43,0,0,0,98.92,79.45Z"/><path class="cls-8" d="M110.16,79.45h-1.89V69.19l-4,1a1.79,1.79,0,0,0-.8.38.84.84,0,0,0-.19.59.94.94,0,0,0,.26.69.81.81,0,0,0,.62.29,2.76,2.76,0,0,0,.6-.11l1.67-.44v7.82H104.5a1.48,1.48,0,0,0-1,.26,1,1,0,0,0,0,1.36,1.48,1.48,0,0,0,1,.26h5.66a1.44,1.44,0,0,0,1-.26.88.88,0,0,0,.29-.67.9.9,0,0,0-.29-.69A1.44,1.44,0,0,0,110.16,79.45Z"/><path class="cls-8" d="M120.45,69.68a3.47,3.47,0,0,0-1.9-.49,3.36,3.36,0,0,0-3,1.67,6.45,6.45,0,0,0-1.06,3.51v2a6.19,6.19,0,0,0,.7,2.85,4.64,4.64,0,0,0,1.49,1.83,3.69,3.69,0,0,0,4.94-1.17,6.53,6.53,0,0,0,1.05-3.51v-2a6.08,6.08,0,0,0-.7-2.86A4.74,4.74,0,0,0,120.45,69.68Zm.32,6.74A4.09,4.09,0,0,1,120,79a1.66,1.66,0,0,1-1.38.7,1.61,1.61,0,0,1-1.36-.63,4.34,4.34,0,0,1-.86-2.68v-2a4.15,4.15,0,0,1,.81-2.61,1.68,1.68,0,0,1,1.39-.7,1.64,1.64,0,0,1,1.37.63,4.37,4.37,0,0,1,.85,2.68Z"/><path class="cls-8" d="M87.67,101.94H85.78V91.67l-4,1a1.59,1.59,0,0,0-.8.38.84.84,0,0,0-.2.59,1,1,0,0,0,.27.69.81.81,0,0,0,.61.29,2.72,2.72,0,0,0,.61-.11l1.67-.44v7.82H82a1.46,1.46,0,0,0-1,.26.94.94,0,0,0,0,1.36,1.46,1.46,0,0,0,1,.26h5.66a1.48,1.48,0,0,0,1-.26.94.94,0,0,0,0-1.36A1.48,1.48,0,0,0,87.67,101.94Z"/><path class="cls-8" d="M98.92,101.94H97V91.67l-4,1a1.68,1.68,0,0,0-.81.38.84.84,0,0,0-.19.59,1,1,0,0,0,.26.69.83.83,0,0,0,.62.29,2.76,2.76,0,0,0,.6-.11l1.68-.44v7.82h-1.9a1.43,1.43,0,0,0-1,.26.94.94,0,0,0,0,1.36,1.43,1.43,0,0,0,1,.26h5.67a1.43,1.43,0,0,0,1-.26.94.94,0,0,0,0-1.36A1.43,1.43,0,0,0,98.92,101.94Z"/><path class="cls-8" d="M109.21,92.17a3.5,3.5,0,0,0-1.91-.5,3.36,3.36,0,0,0-3,1.68,6.52,6.52,0,0,0-1,3.5v2.06a6,6,0,0,0,.7,2.84,4.6,4.6,0,0,0,1.48,1.84,3.43,3.43,0,0,0,1.92.5,3.37,3.37,0,0,0,3-1.67,6.45,6.45,0,0,0,1.06-3.51V96.85A6.08,6.08,0,0,0,110.7,94,4.73,4.73,0,0,0,109.21,92.17Zm.31,6.74a4.15,4.15,0,0,1-.81,2.61,1.69,1.69,0,0,1-1.38.7,1.61,1.61,0,0,1-1.37-.64,4.29,4.29,0,0,1-.86-2.67V96.85a4.08,4.08,0,0,1,.82-2.6,1.67,1.67,0,0,1,1.39-.7,1.62,1.62,0,0,1,1.36.63,4.34,4.34,0,0,1,.85,2.67Z"/><path class="cls-8" d="M120.45,92.17a3.47,3.47,0,0,0-1.9-.5,3.35,3.35,0,0,0-3,1.68,6.45,6.45,0,0,0-1.06,3.5v2.06a6.12,6.12,0,0,0,.7,2.84,4.61,4.61,0,0,0,1.49,1.84,3.69,3.69,0,0,0,4.94-1.17,6.53,6.53,0,0,0,1.05-3.51V96.85A6.07,6.07,0,0,0,122,94,4.74,4.74,0,0,0,120.45,92.17Zm.32,6.74a4.09,4.09,0,0,1-.82,2.61,1.69,1.69,0,0,1-1.38.7,1.59,1.59,0,0,1-1.36-.64,4.29,4.29,0,0,1-.86-2.67V96.85a4.14,4.14,0,0,1,.81-2.6,1.68,1.68,0,0,1,1.39-.7,1.64,1.64,0,0,1,1.37.63,4.34,4.34,0,0,1,.85,2.67Z"/><path class="cls-8" d="M76.43,101.94h-1.9V97.33a41.56,41.56,0,0,1-1.87,3.85v.76h-.42c-.38.65-.78,1.26-1.18,1.88h5.37a1.43,1.43,0,0,0,1-.26.94.94,0,0,0,0-1.36A1.43,1.43,0,0,0,76.43,101.94Z"/><path class="cls-8" d="M104,15.89a3.9,3.9,0,0,0-2-.53,3.15,3.15,0,0,0-2.69,1.35L66.93,65.29l-.11-.08a14.4,14.4,0,0,0-8-2.67,12,12,0,0,0-2.46.26c-4.76,1-8,4.79-9.16,6.35l-1.1,1.48,27.75,20,.86-2.17c2.43-6.15,4-12.24-2.4-18.83l33-49.51A3.06,3.06,0,0,0,104,15.89Z"/><path class="cls-8" d="M54.66,80.06l-10-7.2-1.07.86C36.39,79.58,29,85.59,20,85.59a22.06,22.06,0,0,1-8.84-1.94l-.11,0a.34.34,0,0,0-.18.07.29.29,0,0,0-.1.28,53.85,53.85,0,0,0,1.53,5.88,2,2,0,0,0,1.29,1.32,24.33,24.33,0,0,0,7.77,1.27h.53c5.19-.13,13.24-2,22.12-10.06a.67.67,0,0,1,.46-.2.61.61,0,0,1,.45.2c.5.52,0,1.69-1.27,3-9,9-16,9.71-22.44,9.71a26.45,26.45,0,0,1-6.8-.86l-.09,0a.26.26,0,0,0-.21.1.32.32,0,0,0-.05.34,45.47,45.47,0,0,0,5.82,10.14,2,2,0,0,0,1.56.75h.18a31.93,31.93,0,0,0,6.65-1.29,22.38,22.38,0,0,0,7.41-3.76,15.35,15.35,0,0,0,1.51-1.39,6.79,6.79,0,0,1,1.42-1.31.44.44,0,0,1,.15,0h0a.32.32,0,0,1,.17.11,1.78,1.78,0,0,1-.13,2.11c-3.75,4.68-8.8,7.32-15.45,8a.3.3,0,0,0-.26.2.32.32,0,0,0,.08.34A32.11,32.11,0,0,0,30,113.6a1.41,1.41,0,0,0,.7.19,1.23,1.23,0,0,0,.42-.07c7.66-2.24,20.81-7.64,22.9-18.56a.6.6,0,0,1,.39-.51l.28-.1.25.2a2.24,2.24,0,0,1,.29,1.69c-1.71,8.7-8.76,15.19-20.38,18.79a.28.28,0,0,0-.21.28.28.28,0,0,0,.19.29,35.88,35.88,0,0,0,12.1,2c.65,0,1.3,0,2,0,1.83-.8,17.58-8.23,23-24.26a.31.31,0,0,0-.1-.35Z"/><path class="cls-8" d="M32.38,67.26c-.68-10.05-1.8-13.4-9.37-16.07h0c7.57-2.91,8.69-5.15,9.37-16.07h0c.62,10.53,2,13.68,9.37,16.07h0c-6.7,2.67-7.82,3.79-9.37,16.07"/><path class="cls-8" d="M46.43,45c-.34-3.77-.9-5-4.68-6h0c3.78-1.1,4.34-1.93,4.68-6h0c.31,3.94,1,5.12,4.69,6h0c-3.35,1-3.91,1.41-4.69,6"/><path class="cls-8" d="M35.5,28.22c-.23-2.24-.6-3-3.12-3.58h0c2.52-.65,2.89-1.14,3.12-3.57h0c.21,2.34.67,3,3.12,3.57h0c-2.23.6-2.6.85-3.12,3.58"/></g><path class="cls-8" d="M110.7,26.54a4.7,4.7,0,0,0-1.49-1.84,3.6,3.6,0,0,0-1.88-.49l-4,3.9a5.92,5.92,0,0,0-.14,1.32v.93h0v1.08a6,6,0,0,0,.18,1.47,5.9,5.9,0,0,0,.53,1.41v0a4.72,4.72,0,0,0,1.48,1.83,3.52,3.52,0,0,0,1.92.5,3.36,3.36,0,0,0,3-1.68,6.4,6.4,0,0,0,1.06-3.5V29.39A6,6,0,0,0,110.7,26.54Zm-1.18,4.91a4.19,4.19,0,0,1-.81,2.61,1.68,1.68,0,0,1-1.38.69,1.62,1.62,0,0,1-1.37-.63,4.33,4.33,0,0,1-.86-2.67V29.39a4.09,4.09,0,0,1,.82-2.61,1.68,1.68,0,0,1,1.39-.69,1.62,1.62,0,0,1,1.36.63,4.32,4.32,0,0,1,.85,2.67Z"/></g></g></svg>
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
import { session } from "@web/session";

export class DataRecycleListController extends DataCleaningCommonListController {
    /**
     * Validate all the records selected
     */
    async onValidateClick() {
        let record_ids;
        if (this.isDomainSelected) {
            const domain = this.props.domain;
            record_ids = await this._domainToResIds(domain, session.active_ids_limit);
        } else {
            record_ids = await this.getSelectedResIds();
        }

        await this.orm.call('data_recycle.record', 'action_validate', [record_ids]);
        await this.model.load();
        this.model.notify();
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
    <t t-name="DataRecycle.buttons" t-inherit="web.ListView.Buttons" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-if='props.showButtons']">
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
        web_icon="data_recycle,static/description/icon.svg"
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
                                <field name="include_archived" attrs="{'invisible': [('recycle_action', '!=', 'unlink')]}"/>

                                <!-- Manual cleaning -->
                                <label for="notify_user_ids" attrs="{'invisible': [('recycle_mode', '=', 'automatic')]}" />
                                <div attrs="{'invisible': [('recycle_mode', '=', 'automatic')]}">
                                    <field name="notify_user_ids" widget="many2many_tags"  options="{'no_create': True, 'no_edit': True}" domain="[('share', '=', False)]" nolabel="1"/>
                                    <div class="d-flex w-50" attrs="{'invisible': [('notify_user_ids', '=', [])]}">
                                        <span class="me-1">Every</span>
                                        <field name="notify_frequency" attrs="{'required': [('notify_user_ids', '!=', [])]}" />
                                        <field name="notify_frequency_period" attrs="{'required': [('notify_user_ids', '!=', [])]}" />
                                    </div>
                                </div>
                            </group>
                        </group>
                        <group attrs="{'invisible': [('res_model_id', '!=', False)]}">
                            <group>
                                <div class="alert alert-info" role="alert" colspan="2">
                                    Select a model to configure recycling actions
                                </div>
                            </group>
                        </group>
                        <group attrs="{'invisible': [('res_model_id', '=', False)]}">
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
                    <field name="active" invisible="1" />
                    <field name="res_model_name" invisible="1" />
                    <field name="res_id" />
                    <field name="recycle_model_id" string="Recycle Rule" optional="hide" />
                    <field name="name" />
                    <button icon="fa-check" string="Validate" type="object" name="action_validate" />
                    <button icon="fa-times" string="Discard" type="object" name="action_discard" attrs="{'invisible': [('active', '=', False)]}" />
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

