# Odoo Module: maintenance

Category: Manufacturing/Maintenance

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

{
    'name': 'Maintenance',
    'version': '1.0',
    'sequence': 100,
    'category': 'Manufacturing/Maintenance',
    'description': """
        Track equipments and maintenance requests""",
    'depends': ['mail'],
    'summary': 'Track equipment and manage maintenance requests',
    'website': 'https://www.odoo.com/page/tpm-maintenance-software',
    'data': [
        'security/maintenance.xml',
        'security/ir.model.access.csv',
        'data/maintenance_data.xml',
        'data/mail_data.xml',
        'views/maintenance_views.xml',
        'views/maintenance_templates.xml',
        'views/mail_activity_views.xml',
        'data/maintenance_cron.xml',
    ],
    'demo': ['data/maintenance_demo.xml'],
    'installable': True,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <!-- Maintenance-specific activities, for automatic generation mainly -->
    <record id="mail_act_maintenance_request" model="mail.activity.type">
        <field name="name">Maintenance Request</field>
        <field name="icon">fa-wrench</field>
        <field name="res_model_id" ref="maintenance.model_maintenance_request"/>
    </record>

    <!-- email alias for maintenance requests -->
    <record id="mail_alias_equipment" model="mail.alias">
        <field name="alias_name">helpdesk</field>
        <field name="alias_model_id" ref="model_maintenance_request"/>
        <field name="alias_user_id" ref="base.user_admin"/>
    </record>

    <!-- Maintenance Request-related subtypes for messaging / Chatter -->
    <record id="mt_req_created" model="mail.message.subtype">
        <field name="name">Request Created</field>
        <field name="res_model">maintenance.request</field>
        <field name="default" eval="False"/>
        <field name="hidden" eval="True"/>
        <field name="description">Maintenance Request created</field>
    </record>
    <record id="mt_req_status" model="mail.message.subtype">
        <field name="name">Status Changed</field>
        <field name="res_model">maintenance.request</field>
        <field name="default" eval="True"/>
        <field name="description">Status changed</field>
    </record>

    <!-- Equipment-related subtypes for messaging / Chatter -->
    <record id="mt_mat_assign" model="mail.message.subtype">
        <field name="name">Equipment Assigned</field>
        <field name="res_model">maintenance.equipment</field>
        <field name="description">Equipment Assigned</field>
    </record>

    <!-- Equipment Category-related subtypes for messaging / Chatter -->
    <record id="mt_cat_req_created" model="mail.message.subtype">
        <field name="name">Maintenance Request Created</field>
        <field name="res_model">maintenance.equipment.category</field>
        <field name="default" eval="True"/>
        <field name="parent_id" ref="mt_req_created"/>
        <field name="relation_field">category_id</field>
    </record>
    <record id="mt_cat_mat_assign" model="mail.message.subtype">
        <field name="name">Equipment Assigned</field>
        <field name="res_model">maintenance.equipment.category</field>
        <field name="default" eval="True"/>
        <field name="parent_id" ref="mt_mat_assign"/>
        <field name="relation_field">category_id</field>
    </record>
    <record id="equipment_team_maintenance" model="maintenance.team">
        <field name="name">Internal Maintenance</field>
    </record>
</data>
</odoo>

```

## File: data\maintenance_cron.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
	<record model="ir.cron" id="maintenance_requests_cron">
        <field name="name">Maintenance: generate preventive maintenance requests</field>
        <field name="model_id" ref="model_maintenance_equipment"/>
        <field name="state">code</field>
        <field name="code">model._cron_generate_requests()</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
    </record>
</odoo>
```

## File: data\maintenance_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

    <!-- Standard stages for Maintenance Request -->
    <record id="stage_0" model="maintenance.stage">
        <field name="name">New Request</field>
        <field name="sequence" eval="1" />
        <field name="fold" eval="False" />
    </record>
    <record id="stage_1" model="maintenance.stage">
        <field name="name">In Progress</field>
        <field name="sequence" eval="2" />
        <field name="fold" eval="False" />
    </record>
    <record id="stage_3" model="maintenance.stage">
        <field name="name">Repaired</field>
        <field name="sequence" eval="3" />
        <field name="fold" eval="True" />
        <field name="done" eval="True" />
    </record>
    <record id="stage_4" model="maintenance.stage">
        <field name="name">Scrap</field>
        <field name="sequence" eval="4" />
        <field name="fold" eval="True" />
        <field name="done" eval="True" />
    </record>

</data>
</odoo>

```

## File: data\maintenance_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">


    <!--  Maintenance teams -->
    <record id="equipment_team_metrology" model="maintenance.team">
        <field name="name">Metrology</field>
    </record>
    <record id="equipment_team_subcontractor" model="maintenance.team">
        <field name="name">Subcontractor</field>
    </record>

    <!-- Equipment categories -->
    <record id="equipment_computer" model="maintenance.equipment.category">
        <field name="name">Computers</field>
    </record>
    <record id="equipment_software" model="maintenance.equipment.category">
        <field name="name">Software</field>
    </record>
    <record id="equipment_printer" model="maintenance.equipment.category">
        <field name="name">Printers</field>
    </record>
    <record id="equipment_monitor" model="maintenance.equipment.category">
        <field name="name">Monitors</field>
        <field name="technician_user_id" ref="base.user_admin"/>
        <field name="color">3</field>
    </record>
    <record id="equipment_phone" model="maintenance.equipment.category">
        <field name="name">Phones</field>
        <field name="technician_user_id" ref="base.user_admin"/>
    </record>

    <!-- Equipments -->
    <record id="equipment_monitor1" model="maintenance.equipment">
        <field name="name">Samsung Monitor 15"</field>
        <field name="category_id" ref="equipment_monitor"/>
        <field name="owner_user_id" ref="base.user_admin"/>
        <field name="technician_user_id" ref="base.user_admin"/>
        <field name="assign_date" eval="time.strftime('%Y-%m-10')"/>
        <field name="serial_no">MT/122/11112222</field>
        <field name="model">NP300E5X</field>
    </record>
    <record id="equipment_monitor4" model="maintenance.equipment">
        <field name="name">Samsung Monitor 15"</field>
        <field name="category_id" ref="equipment_monitor"/>
        <field name="owner_user_id" ref="base.user_admin"/>
        <field name="technician_user_id" ref="base.user_admin"/>
        <field name="assign_date" eval="time.strftime('%Y-01-01')"/>
        <field name="serial_no">MT/125/22778837</field>
        <field name="model">NP355E5X</field>
    </record>
    <record id="equipment_monitor6" model="maintenance.equipment">
        <field name="name">Samsung Monitor 15"</field>
        <field name="category_id" ref="equipment_monitor"/>
        <field name="owner_user_id" ref="base.user_demo"/>
        <field name="technician_user_id" ref="base.user_demo"/>
        <field name="assign_date" eval="time.strftime('%Y-02-01')"/>
        <field name="serial_no">MT/127/18291018</field>
        <field name="model">NP355E5X</field>
        <field name="color">3</field>
    </record>
     <record id="equipment_computer3" model="maintenance.equipment">
        <field name="name">Acer Laptop</field>
        <field name="category_id" ref="equipment_computer"/>
        <field name="owner_user_id" ref="base.user_demo"/>
        <field name="technician_user_id" ref="base.user_admin"/>
        <field name="assign_date" eval="time.strftime('%Y-03-08')"/>
        <field name="serial_no">LP/203/19281928</field>
        <field name="model">NE56R</field>
     </record>
     <record id="equipment_computer5" model="maintenance.equipment">
        <field name="name">Acer Laptop</field>
        <field name="category_id" ref="equipment_computer"/>
        <field name="owner_user_id" ref="base.user_admin"/>
        <field name="technician_user_id" ref="base.user_demo"/>
        <field name="assign_date" eval="time.strftime('%Y-04-08')"/>
        <field name="serial_no">LP/205/12928291</field>
        <field name="model">V5131</field>
     </record>
     <record id="equipment_computer9" model="maintenance.equipment">
        <field name="name">HP Laptop</field>
        <field name="category_id" ref="equipment_computer"/>
        <field name="owner_user_id" ref="base.user_admin"/>
        <field name="technician_user_id" ref="base.user_demo"/>
        <field name="assign_date" eval="time.strftime('%Y-%m-11')"/>
        <field name="serial_no">LP/303/28292090</field>
        <field name="model">17-j059nr</field>
     </record>
     <record id="equipment_computer11" model="maintenance.equipment">
        <field name="name">HP Laptop</field>
        <field name="category_id" ref="equipment_computer"/>
        <field name="owner_user_id" ref="base.user_demo"/>
        <field name="technician_user_id" ref="base.user_demo"/>
        <field name="assign_date" eval="time.strftime('%Y-05-01')"/>
        <field name="serial_no">LP/305/17281718</field>
     </record>
     <record id="equipment_printer1" model="maintenance.equipment">
        <field name="name">HP Inkjet printer</field>
        <field name="category_id" ref="equipment_printer"/>
        <field name="technician_user_id" ref="base.user_demo"/>
        <field name="serial_no">PR/011/2928191889</field>
     </record>

    <!--Maintenance Request-->

    <record id="m_request_3" model="maintenance.request">
        <field name="name">Resolution is bad</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="owner_user_id" ref="base.user_admin"/>
        <field name="equipment_id" ref="equipment_monitor6"/>
        <field name="color">7</field>
        <field name="stage_id" ref="stage_3"/>
        <field name="maintenance_team_id" ref="equipment_team_maintenance"/>
    </record>
    <record id="m_request_4" model="maintenance.request">
        <field name="name">Some keys are not working</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="owner_user_id" ref="base.user_admin"/>
        <field name="equipment_id" ref="equipment_computer3"/>
        <field name="stage_id" ref="stage_0"/>
        <field name="maintenance_team_id" ref="equipment_team_maintenance"/>
    </record>
    <record id="m_request_6" model="maintenance.request">
        <field name="name">Motherboard failed</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="owner_user_id" ref="base.user_admin"/>
        <field name="equipment_id" ref="equipment_computer5"/>
         <field name="stage_id" ref="stage_4"/>
         <field name="maintenance_team_id" ref="equipment_team_maintenance"/>
    </record>
    <record id="m_request_7" model="maintenance.request">
        <field name="name">Battery drains fast</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="owner_user_id" ref="base.user_demo"/>
        <field name="equipment_id" ref="equipment_computer9"/>
        <field name="stage_id" ref="stage_1"/>
        <field name="maintenance_team_id" ref="equipment_team_maintenance"/>
    </record>
    <record id="m_request_8" model="maintenance.request">
        <field name="name">Touchpad not working</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="owner_user_id" ref="base.user_demo"/>
        <field name="equipment_id" ref="equipment_computer11"/>
        <field name="stage_id" ref="stage_1"/>
        <field name="maintenance_team_id" ref="equipment_team_maintenance"/>
    </record>
</data>
</odoo>

```

## File: models\maintenance.py

```python
# -*- coding: utf-8 -*-

import ast

from datetime import date, datetime, timedelta

from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.exceptions import UserError
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT, DEFAULT_SERVER_DATETIME_FORMAT


class MaintenanceStage(models.Model):
    """ Model for case stages. This models the main stages of a Maintenance Request management flow. """

    _name = 'maintenance.stage'
    _description = 'Maintenance Stage'
    _order = 'sequence, id'

    name = fields.Char('Name', required=True, translate=True)
    sequence = fields.Integer('Sequence', default=20)
    fold = fields.Boolean('Folded in Maintenance Pipe')
    done = fields.Boolean('Request Done')


class MaintenanceEquipmentCategory(models.Model):
    _name = 'maintenance.equipment.category'
    _inherit = ['mail.alias.mixin', 'mail.thread']
    _description = 'Maintenance Equipment Category'

    @api.depends('equipment_ids')
    def _compute_fold(self):
        # fix mutual dependency: 'fold' depends on 'equipment_count', which is
        # computed with a read_group(), which retrieves 'fold'!
        self.fold = False
        for category in self:
            category.fold = False if category.equipment_count else True

    name = fields.Char('Category Name', required=True, translate=True)
    company_id = fields.Many2one('res.company', string='Company',
        default=lambda self: self.env.company)
    technician_user_id = fields.Many2one('res.users', 'Responsible', tracking=True, default=lambda self: self.env.uid)
    color = fields.Integer('Color Index')
    note = fields.Text('Comments', translate=True)
    equipment_ids = fields.One2many('maintenance.equipment', 'category_id', string='Equipments', copy=False)
    equipment_count = fields.Integer(string="Equipment", compute='_compute_equipment_count')
    maintenance_ids = fields.One2many('maintenance.request', 'category_id', copy=False)
    maintenance_count = fields.Integer(string="Maintenance Count", compute='_compute_maintenance_count')
    alias_id = fields.Many2one(
        'mail.alias', 'Alias', ondelete='restrict', required=True,
        help="Email alias for this equipment category. New emails will automatically "
        "create a new equipment under this category.")
    fold = fields.Boolean(string='Folded in Maintenance Pipe', compute='_compute_fold', store=True)

    def _compute_equipment_count(self):
        equipment_data = self.env['maintenance.equipment'].read_group([('category_id', 'in', self.ids)], ['category_id'], ['category_id'])
        mapped_data = dict([(m['category_id'][0], m['category_id_count']) for m in equipment_data])
        for category in self:
            category.equipment_count = mapped_data.get(category.id, 0)

    def _compute_maintenance_count(self):
        maintenance_data = self.env['maintenance.request'].read_group([('category_id', 'in', self.ids)], ['category_id'], ['category_id'])
        mapped_data = dict([(m['category_id'][0], m['category_id_count']) for m in maintenance_data])
        for category in self:
            category.maintenance_count = mapped_data.get(category.id, 0)

    def unlink(self):
        for category in self:
            if category.equipment_ids or category.maintenance_ids:
                raise UserError(_("You cannot delete an equipment category containing equipments or maintenance requests."))
        return super(MaintenanceEquipmentCategory, self).unlink()

    def _alias_get_creation_values(self):
        values = super(MaintenanceEquipmentCategory, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('maintenance.request').id
        if self.id:
            values['alias_defaults'] = defaults = ast.literal_eval(self.alias_defaults or "{}")
            defaults['category_id'] = self.id
        return values


class MaintenanceEquipment(models.Model):
    _name = 'maintenance.equipment'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = 'Maintenance Equipment'
    _check_company_auto = True

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'owner_user_id' in init_values and self.owner_user_id:
            return self.env.ref('maintenance.mt_mat_assign')
        return super(MaintenanceEquipment, self)._track_subtype(init_values)

    def name_get(self):
        result = []
        for record in self:
            if record.name and record.serial_no:
                result.append((record.id, record.name + '/' + record.serial_no))
            if record.name and not record.serial_no:
                result.append((record.id, record.name))
        return result

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        args = args or []
        equipment_ids = []
        if name:
            equipment_ids = self._search([('name', '=', name)] + args, limit=limit, access_rights_uid=name_get_uid)
        if not equipment_ids:
            equipment_ids = self._search([('name', operator, name)] + args, limit=limit, access_rights_uid=name_get_uid)
        return equipment_ids

    name = fields.Char('Equipment Name', required=True, translate=True)
    company_id = fields.Many2one('res.company', string='Company',
        default=lambda self: self.env.company)
    active = fields.Boolean(default=True)
    technician_user_id = fields.Many2one('res.users', string='Technician', tracking=True)
    owner_user_id = fields.Many2one('res.users', string='Owner', tracking=True)
    category_id = fields.Many2one('maintenance.equipment.category', string='Equipment Category',
                                  tracking=True, group_expand='_read_group_category_ids')
    partner_id = fields.Many2one('res.partner', string='Vendor', check_company=True)
    partner_ref = fields.Char('Vendor Reference')
    location = fields.Char('Location')
    model = fields.Char('Model')
    serial_no = fields.Char('Serial Number', copy=False)
    assign_date = fields.Date('Assigned Date', tracking=True)
    effective_date = fields.Date('Effective Date', default=fields.Date.context_today, required=True, help="Date at which the equipment became effective. This date will be used to compute the Mean Time Between Failure.")
    cost = fields.Float('Cost')
    note = fields.Text('Note')
    warranty_date = fields.Date('Warranty Expiration Date')
    color = fields.Integer('Color Index')
    scrap_date = fields.Date('Scrap Date')
    maintenance_ids = fields.One2many('maintenance.request', 'equipment_id')
    maintenance_count = fields.Integer(compute='_compute_maintenance_count', string="Maintenance Count", store=True)
    maintenance_open_count = fields.Integer(compute='_compute_maintenance_count', string="Current Maintenance", store=True)
    period = fields.Integer('Days between each preventive maintenance')
    next_action_date = fields.Date(compute='_compute_next_maintenance', string='Date of the next preventive maintenance', store=True)
    maintenance_team_id = fields.Many2one('maintenance.team', string='Maintenance Team', check_company=True, ondelete="restrict")
    maintenance_duration = fields.Float(help="Maintenance Duration in hours.")

    @api.depends('effective_date', 'period', 'maintenance_ids.request_date', 'maintenance_ids.close_date')
    def _compute_next_maintenance(self):
        date_now = fields.Date.context_today(self)
        equipments = self.filtered(lambda x: x.period > 0)
        for equipment in equipments:
            next_maintenance_todo = self.env['maintenance.request'].search([
                ('equipment_id', '=', equipment.id),
                ('maintenance_type', '=', 'preventive'),
                ('stage_id.done', '!=', True),
                ('close_date', '=', False)], order="request_date asc", limit=1)
            last_maintenance_done = self.env['maintenance.request'].search([
                ('equipment_id', '=', equipment.id),
                ('maintenance_type', '=', 'preventive'),
                ('stage_id.done', '=', True),
                ('close_date', '!=', False)], order="close_date desc", limit=1)
            if next_maintenance_todo and last_maintenance_done:
                next_date = next_maintenance_todo.request_date
                date_gap = next_maintenance_todo.request_date - last_maintenance_done.close_date
                # If the gap between the last_maintenance_done and the next_maintenance_todo one is bigger than 2 times the period and next request is in the future
                # We use 2 times the period to avoid creation too closed request from a manually one created
                if date_gap > timedelta(0) and date_gap > timedelta(days=equipment.period) * 2 and next_maintenance_todo.request_date > date_now:
                    # If the new date still in the past, we set it for today
                    if last_maintenance_done.close_date + timedelta(days=equipment.period) < date_now:
                        next_date = date_now
                    else:
                        next_date = last_maintenance_done.close_date + timedelta(days=equipment.period)
            elif next_maintenance_todo:
                next_date = next_maintenance_todo.request_date
                date_gap = next_maintenance_todo.request_date - date_now
                # If next maintenance to do is in the future, and in more than 2 times the period, we insert an new request
                # We use 2 times the period to avoid creation too closed request from a manually one created
                if date_gap > timedelta(0) and date_gap > timedelta(days=equipment.period) * 2:
                    next_date = date_now + timedelta(days=equipment.period)
            elif last_maintenance_done:
                next_date = last_maintenance_done.close_date + timedelta(days=equipment.period)
                # If when we add the period to the last maintenance done and we still in past, we plan it for today
                if next_date < date_now:
                    next_date = date_now
            else:
                next_date = equipment.effective_date + timedelta(days=equipment.period)
            equipment.next_action_date = next_date
        (self - equipments).next_action_date = False

    @api.depends('maintenance_ids.stage_id.done')
    def _compute_maintenance_count(self):
        for equipment in self:
            equipment.maintenance_count = len(equipment.maintenance_ids)
            equipment.maintenance_open_count = len(equipment.maintenance_ids.filtered(lambda x: not x.stage_id.done))

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id and self.maintenance_team_id:
            if self.maintenance_team_id.company_id and not self.maintenance_team_id.company_id.id == self.company_id.id:
                self.maintenance_team_id = False

    @api.onchange('category_id')
    def _onchange_category_id(self):
        self.technician_user_id = self.category_id.technician_user_id

    _sql_constraints = [
        ('serial_no', 'unique(serial_no)', "Another asset already exists with this serial number!"),
    ]

    @api.model
    def create(self, vals):
        equipment = super(MaintenanceEquipment, self).create(vals)
        if equipment.owner_user_id:
            equipment.message_subscribe(partner_ids=[equipment.owner_user_id.partner_id.id])
        return equipment

    def write(self, vals):
        if vals.get('owner_user_id'):
            self.message_subscribe(partner_ids=self.env['res.users'].browse(vals['owner_user_id']).partner_id.ids)
        return super(MaintenanceEquipment, self).write(vals)

    @api.model
    def _read_group_category_ids(self, categories, domain, order):
        """ Read group customization in order to display all the categories in
            the kanban view, even if they are empty.
        """
        category_ids = categories._search([], order=order, access_rights_uid=SUPERUSER_ID)
        return categories.browse(category_ids)

    def _prepare_maintenance_request_vals(self, date):
        self.ensure_one()
        return {
            'name': _('Preventive Maintenance - %s', self.name),
            'request_date': date,
            'schedule_date': date,
            'category_id': self.category_id.id,
            'equipment_id': self.id,
            'maintenance_type': 'preventive',
            'owner_user_id': self.owner_user_id.id,
            'user_id': self.technician_user_id.id,
            'maintenance_team_id': self.maintenance_team_id.id,
            'duration': self.maintenance_duration,
            'company_id': self.company_id.id or self.env.company.id
        }

    def _create_new_request(self, date):
        self.ensure_one()
        vals = self._prepare_maintenance_request_vals(date)
        maintenance_requests = self.env['maintenance.request'].create(vals)
        return maintenance_requests

    @api.model
    def _cron_generate_requests(self):
        """
            Generates maintenance request on the next_action_date or today if none exists
        """
        for equipment in self.search([('period', '>', 0)]):
            next_requests = self.env['maintenance.request'].search([('stage_id.done', '=', False),
                                                    ('equipment_id', '=', equipment.id),
                                                    ('maintenance_type', '=', 'preventive'),
                                                    ('request_date', '=', equipment.next_action_date)])
            if not next_requests:
                equipment._create_new_request(equipment.next_action_date)


class MaintenanceRequest(models.Model):
    _name = 'maintenance.request'
    _inherit = ['mail.thread.cc', 'mail.activity.mixin']
    _description = 'Maintenance Request'
    _order = "id desc"
    _check_company_auto = True

    @api.returns('self')
    def _default_stage(self):
        return self.env['maintenance.stage'].search([], limit=1)

    def _creation_subtype(self):
        return self.env.ref('maintenance.mt_req_created')

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'stage_id' in init_values:
            return self.env.ref('maintenance.mt_req_status')
        return super(MaintenanceRequest, self)._track_subtype(init_values)

    def _get_default_team_id(self):
        MT = self.env['maintenance.team']
        team = MT.search([('company_id', '=', self.env.company.id)], limit=1)
        if not team:
            team = MT.search([], limit=1)
        return team.id

    name = fields.Char('Subjects', required=True)
    company_id = fields.Many2one('res.company', string='Company',
        default=lambda self: self.env.company)
    description = fields.Text('Description')
    request_date = fields.Date('Request Date', tracking=True, default=fields.Date.context_today,
                               help="Date requested for the maintenance to happen")
    owner_user_id = fields.Many2one('res.users', string='Created by User', default=lambda s: s.env.uid)
    category_id = fields.Many2one('maintenance.equipment.category', related='equipment_id.category_id', string='Category', store=True, readonly=True)
    equipment_id = fields.Many2one('maintenance.equipment', string='Equipment',
                                   ondelete='restrict', index=True, check_company=True)
    user_id = fields.Many2one('res.users', string='Technician', tracking=True)
    stage_id = fields.Many2one('maintenance.stage', string='Stage', ondelete='restrict', tracking=True,
                               group_expand='_read_group_stage_ids', default=_default_stage, copy=False)
    priority = fields.Selection([('0', 'Very Low'), ('1', 'Low'), ('2', 'Normal'), ('3', 'High')], string='Priority')
    color = fields.Integer('Color Index')
    close_date = fields.Date('Close Date', help="Date the maintenance was finished. ")
    kanban_state = fields.Selection([('normal', 'In Progress'), ('blocked', 'Blocked'), ('done', 'Ready for next stage')],
                                    string='Kanban State', required=True, default='normal', tracking=True)
    # active = fields.Boolean(default=True, help="Set active to false to hide the maintenance request without deleting it.")
    archive = fields.Boolean(default=False, help="Set archive to true to hide the maintenance request without deleting it.")
    maintenance_type = fields.Selection([('corrective', 'Corrective'), ('preventive', 'Preventive')], string='Maintenance Type', default="corrective")
    schedule_date = fields.Datetime('Scheduled Date', help="Date the maintenance team plans the maintenance.  It should not differ much from the Request Date. ")
    maintenance_team_id = fields.Many2one('maintenance.team', string='Team', required=True, default=_get_default_team_id, check_company=True)
    duration = fields.Float(help="Duration in hours.")
    done = fields.Boolean(related='stage_id.done')

    def archive_equipment_request(self):
        self.write({'archive': True})

    def reset_equipment_request(self):
        """ Reinsert the maintenance request into the maintenance pipe in the first stage"""
        first_stage_obj = self.env['maintenance.stage'].search([], order="sequence asc", limit=1)
        # self.write({'active': True, 'stage_id': first_stage_obj.id})
        self.write({'archive': False, 'stage_id': first_stage_obj.id})

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id and self.maintenance_team_id:
            if self.maintenance_team_id.company_id and not self.maintenance_team_id.company_id.id == self.company_id.id:
                self.maintenance_team_id = False

    @api.onchange('equipment_id')
    def onchange_equipment_id(self):
        if self.equipment_id:
            self.user_id = self.equipment_id.technician_user_id if self.equipment_id.technician_user_id else self.equipment_id.category_id.technician_user_id
            self.category_id = self.equipment_id.category_id
            if self.equipment_id.maintenance_team_id:
                self.maintenance_team_id = self.equipment_id.maintenance_team_id.id

    @api.onchange('category_id')
    def onchange_category_id(self):
        if not self.user_id or not self.equipment_id or (self.user_id and not self.equipment_id.technician_user_id):
            self.user_id = self.category_id.technician_user_id

    @api.model
    def create(self, vals):
        # context: no_log, because subtype already handle this
        request = super(MaintenanceRequest, self).create(vals)
        if request.owner_user_id or request.user_id:
            request._add_followers()
        if request.equipment_id and not request.maintenance_team_id:
            request.maintenance_team_id = request.equipment_id.maintenance_team_id
        if not request.stage_id.done:
            request.close_date = False
        elif request.stage_id.done and not request.close_date:
            request.close_date = fields.Date.today()
        request.activity_update()
        return request

    def write(self, vals):
        # Overridden to reset the kanban_state to normal whenever
        # the stage (stage_id) of the Maintenance Request changes.
        if vals and 'kanban_state' not in vals and 'stage_id' in vals:
            vals['kanban_state'] = 'normal'
        res = super(MaintenanceRequest, self).write(vals)
        if vals.get('owner_user_id') or vals.get('user_id'):
            self._add_followers()
        if 'stage_id' in vals:
            self.filtered(lambda m: m.stage_id.done).write({'close_date': fields.Date.today()})
            self.filtered(lambda m: not m.stage_id.done).write({'close_date': False})
            self.activity_feedback(['maintenance.mail_act_maintenance_request'])
            self.activity_update()
        if vals.get('user_id') or vals.get('schedule_date'):
            self.activity_update()
        if vals.get('equipment_id'):
            # need to change description of activity also so unlink old and create new activity
            self.activity_unlink(['maintenance.mail_act_maintenance_request'])
            self.activity_update()
        return res

    def activity_update(self):
        """ Update maintenance activities based on current record set state.
        It reschedule, unlink or create maintenance request activities. """
        self.filtered(lambda request: not request.schedule_date).activity_unlink(['maintenance.mail_act_maintenance_request'])
        for request in self.filtered(lambda request: request.schedule_date):
            date_dl = fields.Datetime.from_string(request.schedule_date).date()
            updated = request.activity_reschedule(
                ['maintenance.mail_act_maintenance_request'],
                date_deadline=date_dl,
                new_user_id=request.user_id.id or request.owner_user_id.id or self.env.uid)
            if not updated:
                if request.equipment_id:
                    note = _('Request planned for <a href="#" data-oe-model="%s" data-oe-id="%s">%s</a>') % (
                        request.equipment_id._name, request.equipment_id.id, request.equipment_id.display_name)
                else:
                    note = False
                request.activity_schedule(
                    'maintenance.mail_act_maintenance_request',
                    fields.Datetime.from_string(request.schedule_date).date(),
                    note=note, user_id=request.user_id.id or request.owner_user_id.id or self.env.uid)

    def _add_followers(self):
        for request in self:
            partner_ids = (request.owner_user_id.partner_id + request.user_id.partner_id).ids
            request.message_subscribe(partner_ids=partner_ids)

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        """ Read group customization in order to display all the stages in the
            kanban view, even if they are empty
        """
        stage_ids = stages._search([], order=order, access_rights_uid=SUPERUSER_ID)
        return stages.browse(stage_ids)


class MaintenanceTeam(models.Model):
    _name = 'maintenance.team'
    _description = 'Maintenance Teams'

    name = fields.Char('Team Name', required=True, translate=True)
    active = fields.Boolean(default=True)
    company_id = fields.Many2one('res.company', string='Company',
        default=lambda self: self.env.company)
    member_ids = fields.Many2many(
        'res.users', 'maintenance_team_users_rel', string="Team Members",
        domain="[('company_ids', 'in', company_id)]")
    color = fields.Integer("Color Index", default=0)
    request_ids = fields.One2many('maintenance.request', 'maintenance_team_id', copy=False)
    equipment_ids = fields.One2many('maintenance.equipment', 'maintenance_team_id', copy=False)

    # For the dashboard only
    todo_request_ids = fields.One2many('maintenance.request', string="Requests", copy=False, compute='_compute_todo_requests')
    todo_request_count = fields.Integer(string="Number of Requests", compute='_compute_todo_requests')
    todo_request_count_date = fields.Integer(string="Number of Requests Scheduled", compute='_compute_todo_requests')
    todo_request_count_high_priority = fields.Integer(string="Number of Requests in High Priority", compute='_compute_todo_requests')
    todo_request_count_block = fields.Integer(string="Number of Requests Blocked", compute='_compute_todo_requests')
    todo_request_count_unscheduled = fields.Integer(string="Number of Requests Unscheduled", compute='_compute_todo_requests')

    @api.depends('request_ids.stage_id.done')
    def _compute_todo_requests(self):
        for team in self:
            team.todo_request_ids = self.env['maintenance.request'].search([('maintenance_team_id', '=', team.id), ('stage_id.done', '=', False)])
            team.todo_request_count = len(team.todo_request_ids)
            team.todo_request_count_date = self.env['maintenance.request'].search_count([('maintenance_team_id', '=', team.id), ('schedule_date', '!=', False)])
            team.todo_request_count_high_priority = self.env['maintenance.request'].search_count([('maintenance_team_id', '=', team.id), ('priority', '=', '3')])
            team.todo_request_count_block = self.env['maintenance.request'].search_count([('maintenance_team_id', '=', team.id), ('kanban_state', '=', 'blocked')])
            team.todo_request_count_unscheduled = self.env['maintenance.request'].search_count([('maintenance_team_id', '=', team.id), ('schedule_date', '=', False)])

    @api.depends('equipment_ids')
    def _compute_equipment(self):
        for team in self:
            team.equipment_count = len(team.equipment_ids)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import maintenance

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_equipment_user,equipment.user,model_maintenance_equipment,base.group_user,1,0,0,0
access_equipment_admin_user,equipment.admin.user,model_maintenance_equipment,group_equipment_manager,1,1,1,1
access_maintenance_system_user,equipment.request system user,model_maintenance_request,base.group_user,1,1,1,1
access_equipment_category_user,equipment.category.user,model_maintenance_equipment_category,base.group_user,1,0,0,0
access_equipment_category_admin_user,equipment.category system user,model_maintenance_equipment_category,group_equipment_manager,1,1,1,1
access_maintenance_stage_user,maintenance.stage.user,model_maintenance_stage,base.group_user,1,0,0,0
access_maintenance_stage_admin_user,equipment.request.stage system user,model_maintenance_stage,group_equipment_manager,1,1,1,1
access_maintenance_team_user,maintenance.team.user,model_maintenance_team,base.group_user,1,0,0,0
access_maintenance_team_admin_user,maintenance.team.admin.user,model_maintenance_team,group_equipment_manager,1,1,1,1
access_mail_activity_type_equipment_manager,mail.activity.type.equipment.manager,mail.model_mail_activity_type,maintenance.group_equipment_manager,1,1,1,1
```

## File: security\maintenance.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- This group is only allowed to deal with equipment registration and maintenance -->
    <record id="group_equipment_manager" model="res.groups">
        <field name="name">Equipment Manager</field>
        <field name="category_id" ref="base.module_category_manufacturing_maintenance"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
        <field name="comment">The user will be able to manage equipments.</field>
    </record>

    <data noupdate="1">

    <!-- Rules -->
    <record id="equipment_request_rule_user" model="ir.rule">
        <field name="name">Users are allowed to access their own maintenance requests</field>
        <field name="model_id" ref="model_maintenance_request"/>
        <field name="domain_force">['|', '|', ('owner_user_id', '=', user.id), ('message_partner_ids', 'in', [user.partner_id.id]), ('user_id.id', '=', user.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="equipment_rule_user" model="ir.rule">
        <field name="name">Users are allowed to access equipments they follow</field>
        <field name="model_id" ref="model_maintenance_equipment"/>
        <field name="domain_force">[('message_partner_ids', 'in', [user.partner_id.id])]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="equipment_request_rule_admin_user" model="ir.rule">
        <field name="name">Administrator of maintenance requests</field>
        <field name="model_id" ref="model_maintenance_request"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('group_equipment_manager'))]"/>
    </record>

    <record id="equipment_rule_admin_user" model="ir.rule">
        <field name="name">Equipments administrator</field>
        <field name="model_id" ref="model_maintenance_equipment"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('group_equipment_manager'))]"/>
    </record>

    <record id="maintenance_request_comp_rule" model="ir.rule">
        <field name="name">Maintenance Request Multi-company rule</field>
        <field name="model_id" ref="model_maintenance_request"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="maintenance_equipment_comp_rule" model="ir.rule">
        <field name="name">Maintenance Equipment Multi-company rule</field>
        <field name="model_id" ref="model_maintenance_equipment"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="maintenance_team_comp_rule" model="ir.rule">
        <field name="name">Maintenance Team Multi-company rule</field>
        <field name="model_id" ref="model_maintenance_team"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="maintenance_equipment_category_comp_rule" model="ir.rule">
        <field name="name">Maintenance Equipment Category Multi-company rule</field>
        <field name="model_id" ref="model_maintenance_equipment_category"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="base.user_admin" model="res.users">
        <field name="groups_id" eval="[(4, ref('group_equipment_manager'))]"/>
    </record>

    </data>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="70" height="70" viewBox="0 0 70 70" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><title>maintenance/static/description/icon</title><defs><path d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z" id="a"/><linearGradient x1="100%" y1="0%" x2="0%" y2="98.616%" id="c"><stop stop-color="#797C79" offset="0%"/><stop stop-color="#545554" offset="100%"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z" fill="#FFF" fill-opacity=".383"/><path d="M42.863 70H4c-2 0-4-.149-4-4.17V42.7L22 21l35 33.362L42.863 70z" fill="#393939" opacity=".324"/><path d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z" fill="#000" fill-opacity=".383"/><path d="M34.821 40.185l-9.488-9.488-3.163 3.163h-3.162c-1.487 1.054-3.02 2.372-4.602 3.953-1.02 1.53-1.81 2.88-2.372 4.05-.622-2.173-.173-4.424 1.346-6.755 1.116-1.36 2.465-3.093 4.046-5.202v-2.372l7.907-7.906c2.635.527 3.69-.528 3.163-3.163l3.162-3.163 6.326 5.535A38.251 38.251 0 0 1 34.82 22c-2.636-.528-3.953.527-3.953 3.162l-2.372 2.372 8.697 9.488 3.953-.79L56.96 52.045v3.163l-4.744 4.744h-3.163L34.03 44.138v-2.372l.791-1.58z" fill="#000" opacity=".3"/><path d="M34.821 38.185l-9.488-9.488-3.163 3.163h-3.162c-1.487 1.054-3.02 2.372-4.602 3.953-1.02 1.53-1.81 2.88-2.372 4.05-.622-2.173-.173-4.424 1.346-6.755 1.116-1.36 2.465-3.093 4.046-5.202v-2.372l7.907-7.906c2.635.527 3.69-.528 3.163-3.163l3.162-3.163 6.326 5.535A38.251 38.251 0 0 1 34.82 20c-2.636-.528-3.953.527-3.953 3.162l-2.372 2.372 8.697 9.488 3.953-.79L56.96 50.045v3.163l-4.744 4.744h-3.163L34.03 42.138v-2.372l.791-1.58z" fill="#FFF"/></g></g></svg>
```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Activity types config -->
    <record id="mail_activity_type_action_config_maintenance" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model_id', '=', False), ('res_model_id.model', '=', 'maintenance.request')]</field>
        <field name="context">{'default_res_model': 'maintenance.request'}</field>
    </record>
    <menuitem id="maintenance_menu_config_activity_type"
        action="mail_activity_type_action_config_maintenance"
        parent="menu_maintenance_configuration"
        sequence="20"
        groups="base.group_no_one"/>
</odoo>
```

## File: views\maintenance_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="maintenance assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/maintenance/static/src/scss/maintenance_team_dashboard.scss"/>
        </xpath>
    </template>
</odoo>

```

## File: views\maintenance_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- equiment.request : views -->
    <record id="hr_equipment_request_view_search" model="ir.ui.view">
        <field name="name">equipment.request.search</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <search string="Maintenance Request Search">
                <field name="name" string="Request"/>
                <field name="category_id"/>
                <field name="user_id"/>
                <field name="equipment_id"/>
                <field name="owner_user_id"/>
                <field name="stage_id"/>
                <field name="maintenance_team_id"/>
                <filter string="My Maintenances" name="my_maintenances" domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter string="To Do" name="todo" domain="[('stage_id.done', '=', False)]"/>
                <separator/>
                <filter string="Blocked" name="kanban_state_block" domain="[('kanban_state', '=', 'blocked')]"/>
                <filter string="Ready" name="done" domain="[('kanban_state', '=', 'done')]"/>
                <separator/>
                <filter string="High-priority" name="high_priority" domain="[('priority', '=', '3')]"/>
                <separator/>
                <filter string="Unscheduled" name="unscheduled" domain="[('schedule_date', '=', False)]"/>
                <separator/>
                <filter name="filter_request_date" date="request_date"/>
                <filter name="filter_schedule_date" date="schedule_date"/>
                <filter name="filter_close_date" date="close_date"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('archive', '=', True)]"/>
                <group  expand='0' string='Group by...'>
                    <filter string='Assigned to' name="assigned" domain="[]" context="{'group_by': 'user_id'}"/>
                    <filter string='Category' name="category" domain="[]" context="{'group_by' : 'category_id'}"/>
                    <filter string='Stage' name="stages" domain="[]" context="{'group_by' : 'stage_id'}"/>
                    <filter string='Created By' name='created_by' domain="[]" context="{'group_by': 'owner_user_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="hr_equipment_request_view_form" model="ir.ui.view">
        <field name="name">equipment.request.form</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <form string="Maintenance Request">
                <header>
                    <button string="Cancel" name="archive_equipment_request" type="object" attrs="{'invisible': [('archive', '=', True)]}"/>
                    <button string="Reopen Request" name="reset_equipment_request" type="object" attrs="{'invisible': [('archive', '=', False)]}"/>
                    <field name="stage_id" widget="statusbar" options="{'clickable': '1'}" attrs="{'invisible': [('archive', '=', True)]}"/>
                </header>
                <sheet>
                    <div attrs="{'invisible': [('archive', '=', False)]}">
                        <span class="badge badge-warning float-right">Canceled</span>
                    </div>
                    <div class="oe_right">
                        <field name="kanban_state" class="oe_inline" widget="state_selection"/>
                    </div>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only" string="Title"/>
                        <h1>
                            <field name="name" placeholder="Maintenance Request"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="owner_user_id" string="Requested By" invisible="1"/>
                            <field name="equipment_id"  context="{'default_company_id':company_id, 'default_category_id':category_id}"/>
                            <field name="category_id" groups="maintenance.group_equipment_manager" context="{'default_company_id':company_id}" attrs="{'invisible': [('equipment_id', '=', False)]}"/>
                            <field name="request_date" readonly="True"/>
                            <field name="done" invisible="1"/>
                            <field name="close_date" attrs="{'invisible': [('done', '!=', True)]}" readonly="True"/>
                            <field name="archive" invisible="1"/>
                            <field name="maintenance_type" widget="radio"/>
                        </group>
                        <group>
                            <field name="maintenance_team_id" options="{'no_create': True, 'no_open': True}"/>
                            <field name="user_id" string="Responsible"/>
                            <field name="schedule_date"/>
                            <label for="duration"/>
                            <div>
                                <field name="duration"
                                       widget="float_time"
                                       class="oe_inline"/> <span class="ml8">hours</span>
                            </div>
                            <field name="priority" widget="priority"/>
                            <field name="email_cc" string="Email cc" groups="base.group_no_one"/>
                            <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                        </group>
                    </group>
                    <field name='description' placeholder="Internal Notes"/>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="hr_equipment_request_view_kanban" model="ir.ui.view">
        <field name="name">equipment.request.kanban</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <kanban default_group_by="stage_id" sample="1">
                <field name="stage_id"/>
                <field name="color"/>
                <field name="priority"/>
                <field name="equipment_id"/>
                <field name="user_id"/>
                <field name="owner_user_id"/>
                <field name="category_id"/>
                <field name="kanban_state"/>
                <field name="activity_ids" />
                <field name="activity_state" />
                <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger"}'/>
                <templates>
                    <t t-name="kanban-tooltip">
                       <ul class="oe_kanban_tooltip">
                          <li t-if="record.category_id.raw_value"><b>Category:</b> <t t-esc="record.category_id.value"/></li>
                          <li t-if="record.user_id.raw_value"><b>Request to:</b> <t t-esc="record.user_id.value"/></li>
                       </ul>
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click oe_semantic_html_override">
                            <div class="o_dropdown_kanban dropdown">

                                <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                    <span class="fa fa-ellipsis-v"/>
                                </a>
                                <div class="dropdown-menu" role="menu">
                                    <t t-if="widget.editable"><a role="menuitem" type="edit" class="dropdown-item">Edit...</a></t>
                                    <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                                    <ul class="oe_kanban_colorpicker" data-field="color"/>
                                </div>
                            </div>
                            <div class="oe_kanban_content" tooltip="kanban-tooltip">
                                <div class="o_kanban_record_top">
                                    <b class="o_kanban_record_title"><field name="name"/></b>
                                </div>
                                <div class="o_kanban_record_body">
                                    <span name="owner_user_id" t-if="record.owner_user_id.raw_value">Requested by : <field name="owner_user_id"/><br/></span>
                                    <span class="oe_grey" t-if="record.equipment_id.raw_value"><field name="equipment_id"/><br/></span>
                                    <span t-if="record.category_id.raw_value"><field name="category_id"/></span>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <field name="priority" widget="priority"/>
                                        <div class="o_kanban_inline_block ml4 mr4">
                                            <field name="activity_ids" widget="kanban_activity" />
                                        </div>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="kanban_state" widget="state_selection"/>
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>
                                </div>
                            </div>
                            <div class="oe_clear"></div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_equipment_request_view_tree" model="ir.ui.view">
        <field name="name">equipment.request.tree</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <tree string="maintenance Request" multi_edit="1" sample="1">
                <field name="message_needaction" invisible="1"/>
                <field name="name"/>
                <field name="request_date" groups="base.group_no_one"/>
                <field name="owner_user_id"/>
                <field name="user_id"/>
                <field name="category_id" readonly="1" groups="maintenance.group_equipment_manager"/>
                <field name="stage_id"/>
                <field name="company_id" readonly="1" groups="base.group_multi_company"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record id="hr_equipment_request_view_graph" model="ir.ui.view">
        <field name="name">equipment.request.graph</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <graph string="maintenance Request" sample="1">
                <field name="user_id"/>
                <field name="stage_id"/>
            </graph>
        </field>
    </record>

    <record id="hr_equipment_request_view_pivot" model="ir.ui.view">
        <field name="name">equipment.request.pivot</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <pivot string="maintenance Request" sample="1">
                <field name="user_id"/>
                <field name="stage_id"/>
                <field name="color" invisible="1"/>
            </pivot>
        </field>
    </record>


    <record id="hr_equipment_view_calendar" model="ir.ui.view">
        <field name="name">equipment.request.calendar</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <calendar date_start="schedule_date" date_delay="duration" color="user_id" event_limit="5">
                <field name="user_id" filters="1"/>
                <field name="priority"/>
                <field name="maintenance_type"/>
            </calendar>
        </field>
    </record>

    <!-- equiment.request : actions -->
    <record id="hr_equipment_request_action" model="ir.actions.act_window">
        <field name="name">Maintenance Requests</field>
        <field name="res_model">maintenance.request</field>
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar</field>
        <field name="view_id" ref="hr_equipment_request_view_kanban"/>
        <field name="context">{'default_user_id': uid}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new maintenance request
            </p><p>
                Follow the process of the request and communicate with the collaborator.
            </p>
        </field>
    </record>

    <record id="hr_equipment_request_action_link" model="ir.actions.act_window">
        <field name="name">Maintenance Requests</field>
        <field name="res_model">maintenance.request</field>
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar</field>
        <field name="search_view_id" ref="hr_equipment_request_view_search"/>
        <field name="view_id" ref="hr_equipment_request_view_kanban"/>
        <field name="context">{
            'search_default_category_id': [active_id],
            'default_category_id': active_id,
        }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new maintenance request
            </p><p>
                Follow the process of the request and communicate with the collaborator.
            </p>
        </field>
    </record>

    <record id="hr_equipment_request_action_from_equipment" model="ir.actions.act_window">
        <field name="name">Maintenance Requests</field>
        <field name="res_model">maintenance.request</field>
        <field name="binding_model_id" ref="maintenance.model_maintenance_equipment"/>
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar</field>
        <field name="context">{
            'default_equipment_id': active_id,
        }</field>
        <field name="domain">[('equipment_id', '=', active_id)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new maintenance request
            </p><p>
                Follow the process of the request and communicate with the collaborator.
            </p>
        </field>
    </record>

    <record id="hr_equipment_todo_request_action_from_dashboard" model="ir.actions.act_window">
        <field name="name">Maintenance Requests</field>
        <field name="res_model">maintenance.request</field>
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar</field>
        <field name="context">{
            'default_maintenance_team_id': active_id,
        }</field>
        <field name="domain">[('maintenance_team_id', '=', active_id), ('maintenance_type', 'in', context.get('maintenance_type', ['preventive', 'corrective']))]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new maintenance request
            </p><p>
                Follow the process of the request and communicate with the collaborator.
            </p>
        </field>
    </record>

    <record id="hr_equipment_request_action_cal" model="ir.actions.act_window">
        <field name="name">Maintenance Requests</field>
        <field name="res_model">maintenance.request</field>
        <field name="view_mode">calendar,kanban,tree,form,pivot,graph</field>
        <field name="view_id" ref="hr_equipment_view_calendar"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new maintenance request
            </p><p>
                Follow the process of the request and communicate with the collaborator.
            </p>
        </field>
    </record>

    <record id="maintenance_request_action_reports" model="ir.actions.act_window">
        <field name="name">Maintenance Requests</field>
        <field name="res_model">maintenance.request</field>
        <field name="view_mode">graph,pivot,kanban,tree,form,calendar</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new maintenance request
            </p><p>
                Follow the process of the request and communicate with the collaborator.
            </p>
        </field>
    </record>

    <!-- equiment : views -->
    <record id="hr_equipment_view_form" model="ir.ui.view">
        <field name="name">equipment.form</field>
        <field name="model">maintenance.equipment</field>
        <field name="arch" type="xml">
            <form string="Equipments">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="%(hr_equipment_request_action_from_equipment)d"
                            type="action"
                            class="oe_stat_button"
                            context="{'default_company_id': company_id}"
                            icon="fa-wrench">
                            <field string="Maintenance" name="maintenance_count" widget="statinfo"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1><field name="name" string="Name" placeholder="Equipment Name"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="category_id" options="{&quot;no_open&quot;: True}" context="{'default_company_id':company_id}"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                            <field name="owner_user_id" string="Owner"/>
                        </group>
                        <group>
                            <field name="maintenance_team_id" attrs="{'required': [('period', '!=', 0)]}" context="{'default_company_id':company_id}"/>
                            <field name="technician_user_id" domain="[('share', '=', False)]"/>
                            <field name="assign_date" groups="base.group_no_one"/>
                            <field name="scrap_date" groups="base.group_no_one"/>
                            <field name="location" string="Used in location"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Description" name="description">
                            <field name="note"/>
                        </page>
                        <page string="Product Information" name="product_information">
                            <group>
                                <group>
                                    <field name="partner_id"/>
                                    <field name="partner_ref"/>
                                    <field name="model"/>
                                    <field name="serial_no"/>
                                </group><group>
                                    <field name="effective_date"/>
                                    <field name="cost" groups="maintenance.group_equipment_manager"/>
                                    <field name="warranty_date"/>
                                </group>
                            </group>
                        </page>
                        <page string="Maintenance" name="maintenance">
                            <group>
                                <group name="maintenance">
                                    <field name="next_action_date" class="oe_read_only" string="Next Preventive Maintenance"/>
                                    <label for="period" string="Preventive Maintenance Frequency"/>
                                    <div class="o_row">
                                        <field name="period"/> days
                                    </div>
                                    <label for="maintenance_duration" string="Maintenance Duration"/>
                                    <div class="o_row">
                                        <field name="maintenance_duration"/> hours
                                    </div>
                                </group>
                            </group>
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

    <record id="hr_equipment_view_kanban" model="ir.ui.view">
        <field name="name">equipment.kanban</field>
        <field name="model">maintenance.equipment</field>
        <field name="arch" type="xml">
            <kanban sample="1">
                <field name="name"/>
                <field name="color"/>
                <field name="technician_user_id"/>
                <field name="owner_user_id"/>
                <field name="category_id"/>
                <field name="serial_no"/>
                <field name="model"/>
                <field name="maintenance_ids"/>
                <field name="maintenance_open_count"/>
                <field name="next_action_date"/>
                <field name="activity_ids" />
                <field name="activity_state" />
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-tooltip">
                        <ul class="oe_kanban_tooltip">
                            <li t-if="record.serial_no.raw_value"><b>Serial Number:</b> <t t-esc="record.serial_no.value"/></li>
                            <li t-if="record.model.raw_value"><b>Model Number:</b> <t t-esc="record.model.value"/></li>
                        </ul>
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click">
                            <div class="o_dropdown_kanban dropdown" t-if="!selection_mode">

                                <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                    <span class="fa fa-ellipsis-v"/>
                                </a>
                                <div class="dropdown-menu" role="menu">
                                    <t t-if="widget.editable"><a role="menuitem" type="edit" class="dropdown-item">Edit...</a></t>
                                    <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                                    <div role="separator" class="dropdown-divider"></div>
                                    <div role="separator" class="dropdown-header">Record Colour</div>
                                    <ul class="oe_kanban_colorpicker" data-field="color"/>
                                </div>
                            </div>
                            <div class="oe_kanban_content" tooltip="kanban-tooltip">
                                <div class="o_kanban_record_top">
                                    <b class="o_kanban_record_title"><field name="name"/><small><span t-if="record.model.raw_value"> (<field name="model"/>)</span></small></b>
                                </div>
                                <div class="o_kanban_record_body">
                                    <div t-if="record.serial_no.raw_value"><field name="serial_no"/></div>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <div class="badge badge-danger" t-if="!selection_mode and record.maintenance_open_count.raw_value" >
                                            <t t-raw="record.maintenance_open_count.raw_value"/> Request
                                        </div>
                                        <div class="badge badge-secondary" t-if="!selection_mode and record.next_action_date.raw_value" >
                                            <t t-raw="moment(record.next_action_date.raw_value).format('MMMM Do')"/>
                                        </div>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <div class="o_kanban_inline_block" t-if="!selection_mode">
                                            <field name="activity_ids" widget="kanban_activity" />
                                        </div>
                                        <field name="owner_user_id" widget="many2one_avatar_user"/>
                                    </div>
                                </div>
                            </div>
                            <div class="oe_clear"></div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_equipment_view_tree" model="ir.ui.view">
        <field name="name">equipment.tree</field>
        <field name="model">maintenance.equipment</field>
        <field name="arch" type="xml">
            <tree string="Assign To User" sample="1">
                <field name="message_needaction" invisible="1"/>
                <field name="name"/>
                <!-- <field name="active" invisible="1"/> -->
                <field name="owner_user_id" string="Owner"/>
                <field name="assign_date" groups="base.group_no_one"/>
                <field name="serial_no"/>
                <field name="technician_user_id"/>
                <field name="category_id"/>
                <field name="partner_id" invisible="1"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record id="hr_equipment_view_search" model="ir.ui.view">
        <field name="name">equipment.search</field>
        <field name="model">maintenance.equipment</field>
        <field name="arch" type="xml">
            <search string="Search">
                <field string="Equipment" name="name" filter_domain="[
                    '|', '|', '|',
                    ('name', 'ilike', self), ('model', 'ilike', self), ('serial_no', 'ilike', self), ('partner_ref', 'ilike', self)]"/>
                <field string="Category" name="category_id"/>
                <field name="owner_user_id"/>
                <filter string="My Equipments" name="my" domain="[('owner_user_id', '=', uid)]"/>
                <filter string="Assigned" name="assigned" domain="[('owner_user_id', '!=', False)]"/>
                <filter string="Unassigned" name="available" domain="[('owner_user_id', '=', False)]"/>
                <separator/>
                <filter string="Under Maintenance" name="under_maintenance" domain="[('maintenance_open_count', '&gt;', 0)]"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction','=',True)]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                <group  expand='0' string='Group by...'>
                    <filter string='Technician' name="technicians" domain="[]" context="{'group_by': 'technician_user_id'}"/>
                    <filter string='Category' name="category" domain="[]" context="{'group_by': 'category_id'}"/>
                    <filter string='Owner' name="owner" domain="[]" context="{'group_by': 'owner_user_id'}"/>
                    <filter string='Vendor' name="vendor" domain="[]" context="{'group_by': 'partner_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="hr_equipment_action" model="ir.actions.act_window">
        <field name="name">Equipments</field>
        <field name="res_model">maintenance.equipment</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="view_id" ref="hr_equipment_view_kanban"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new equipment
            </p><p>
                Track equipments and link it to an employee or department.
                You will be able to manage allocations, issues and maintenance of your equipment.
            </p>
        </field>
    </record>

    <!-- equiment : actions -->
    <record id="hr_equipment_action_from_category_form" model="ir.actions.act_window">
        <field name="name">Equipments</field>
        <field name="res_model">maintenance.equipment</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="search_view_id" ref="hr_equipment_view_search"/>
        <field name="view_id" ref="hr_equipment_view_kanban"/>
        <field name="context">{
            'search_default_category_id': [active_id],
            'default_category_id': active_id,
        }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new equipment
            </p><p>
                Track equipments and link it to an employee or department.
                You will be able to manage allocations, issues and maintenance of your equipment.
            </p>
        </field>
    </record>

    <!-- equipment.category : views -->
    <record id="hr_equipment_category_view_form" model="ir.ui.view">
        <field name="name">equipment.category.form</field>
        <field name="model">maintenance.equipment.category</field>
        <field name="arch" type="xml">
            <form string="Equipment Categories">
                <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="%(hr_equipment_action_from_category_form)d"
                        class="oe_stat_button"
                        icon="fa-cubes"
                        type="action">
                        <field string="Equipment" name="equipment_count" widget="statinfo"/>
                    </button>
                    <button name="%(hr_equipment_request_action_link)d"
                        type="action"
                        class="oe_stat_button"
                        icon="fa-wrench">
                        <field string="Maintenance" name="maintenance_count" widget="statinfo"/>
                    </button>
                </div>
                <div class="oe_title">
                    <label for="name" class="oe_edit_only" string="Category Name"/>
                    <h1>
                        <field name="name"/>
                    </h1>
                </div>
                <group>
                    <field name="technician_user_id" class="oe_inline" domain="[('share', '=', False)]"/>
                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" class="oe_inline"/>
                </group>
                <group name="group_alias" attrs="{'invisible': [('alias_domain', '=', False)]}" groups="base.group_no_one">
                    <label for="alias_name" string="Email Alias"/>
                    <div name="alias_def">
                        <field name="alias_id" class="oe_read_only oe_inline" string="Email Alias" required="0"/>
                        <div class="oe_edit_only oe_inline" name="edit_alias" style="display: inline;">
                            <field name="alias_name" class="oe_inline"/>@<field name="alias_domain" class="oe_inline" readonly="1"/>
                        </div>
                    </div>
                </group>
                <field name="note" nolabel="1"/>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="hr_equipment_category_view_tree" model="ir.ui.view">
        <field name="name">equipment.category.tree</field>
        <field name="model">maintenance.equipment.category</field>
        <field name="arch" type="xml">
            <tree string="Assign To User">
                <field name="name" string="Name"/>
                <field name="technician_user_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="hr_equipment_category_view_search" model="ir.ui.view">
        <field name="name">equipment.category.search</field>
        <field name="model">maintenance.equipment.category</field>
        <field name="arch" type="xml">
            <search string="Search">
                <field name="name" string="Category Name" filter_domain="[('name','ilike',self)]"/>
                <group  expand='0' string='Group by...'>
                    <filter string='Responsible' name="responsible" domain="[]" context="{'group_by' : 'technician_user_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="view_maintenance_equipment_category_kanban" model="ir.ui.view">
        <field name="name">maintenance.equipment.category.kanban</field>
        <field name="model">maintenance.equipment.category</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="name"/>
                <field name="technician_user_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="mb4">
                                <strong><field name="name"/></strong>
                            </div>
                            <div class="row mt4">
                                <div class="col-6">
                                    <span class="badge badge-pill">
                                        <strong>Equipments:</strong> <field name="equipment_count"/>
                                    </span>
                                </div>
                                <div class="col-6 text-right">
                                    <span class="badge badge-pill">
                                        <strong>Maintenance:</strong> <field name="maintenance_count"/>
                                    </span>
                                    <field name="technician_user_id" widget="many2one_avatar_user"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- equipment.category : actions -->
    <record id="hr_equipment_category_action" model="ir.actions.act_window">
        <field name="name">Equipment Categories</field>
        <field name="res_model">maintenance.equipment.category</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="view_id" ref="hr_equipment_category_view_tree"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new equipment category
            </p>
        </field>
    </record>

    <!-- equipment.stage : views -->
    <record id="hr_equipment_stage_view_search" model="ir.ui.view">
        <field name="name">equipment.stage.search</field>
        <field name="model">maintenance.stage</field>
        <field name="arch" type="xml">
            <search string="Maintenance Request Stages">
               <field name="name" string="Maintenance Request Stages"/>
            </search>
        </field>
    </record>

    <record id="hr_equipment_stage_view_tree" model="ir.ui.view">
        <field name="name">equipment.stage.tree</field>
        <field name="model">maintenance.stage</field>
        <field name="arch" type="xml">
            <tree string="Maintenance Request Stage" editable="top">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="fold"/>
                <field name="done"/>
            </tree>
        </field>
    </record>
    <record id="hr_equipment_stage_view_kanban" model="ir.ui.view">
        <field name="name">equipment.stage.kanban</field>
        <field name="model">maintenance.stage</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div>
                                <strong><field name="name"/></strong>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- equipment.stages : actions -->
    <record id="hr_equipment_stage_action" model="ir.actions.act_window">
        <field name="name">Stages</field>
        <field name="res_model">maintenance.stage</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a new stage in the maintenance request
          </p>
        </field>
    </record>

    <!-- maintenance.team: views -->
    <record id="maintenance_team_view_form" model="ir.ui.view">
        <field name="name">maintenance.team.form</field>
        <field name="model">maintenance.team</field>
        <field name="arch" type="xml">
            <form string="Maintenance Team">
                <sheet>
                <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                <div class="oe_title">
                    <label for="name" class="oe_edit_only" string="Team Name"/>
                    <h1>
                        <field name="name"/>
                    </h1>
                </div>
                <group>
                    <group>
                        <field name="active" invisible="1"/>
                        <field name="member_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create': True}" domain="[('share', '=', False)]"/>
                    </group>
                    <group>
                        <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                    </group>
                </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="maintenance_team_view_tree" model="ir.ui.view">
        <field name="name">maintenance.team.tree</field>
        <field name="model">maintenance.team</field>
        <field name="arch" type="xml">
            <tree string="Maintenance Team" editable="bottom">
                <field name="name"/>
                <field name="member_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create': True}" domain="[('share', '=', False)]"/>
                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
            </tree>
        </field>
    </record>

    <record id="maintenance_team_view_kanban" model="ir.ui.view">
        <field name="name">maintenance.team.kanban</field>
        <field name="model">maintenance.team</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div>
                                <strong><field name="name"/></strong>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="maintenance_team_kanban" model="ir.ui.view">
        <field name="name">maintenance.team.kanban</field>
        <field name="model">maintenance.team</field>
        <field name="arch" type="xml">
            <kanban class="oe_background_grey o_kanban_dashboard o_maintenance_team_kanban" create="0">
                <field name="name"/>
                <field name="color"/>
                <field name="todo_request_ids"/>
                <field name="todo_request_count"/>
                <field name="todo_request_count_date"/>
                <field name="todo_request_count_high_priority"/>
                <field name="todo_request_count_block"/>
                <field name="todo_request_count_unscheduled"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                            <div t-attf-class="o_kanban_card_header">
                                <div class="o_kanban_card_header_title">
                                    <div class="o_primary">
                                        <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action">
                                            <field name="name"/>
                                        </a></div>
                                </div>
                                <div class="o_kanban_manage_button_section">
                                    <a class="o_kanban_manage_toggle_button" href="#"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
                                </div>
                            </div>
                            <div class="container o_kanban_card_content">
                                <div class="row">
                                    <div class="col-6 o_kanban_primary_left">
                                        <button class="btn btn-primary" name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action" context="{'search_default_todo': 1}">
                                            <t t-esc="record.todo_request_count.value"/> To Do
                                        </button>
                                    </div>
                                    <div class="col-6 o_kanban_primary_right">
                                        <div t-if="record.todo_request_count_date.raw_value > 0">
                                            <a name="%(hr_equipment_request_action_cal)d" type="action">
                                                <t t-esc="record.todo_request_count_date.value"/>
                                                Scheduled
                                            </a>
                                        </div>
                                        <div t-if="record.todo_request_count_high_priority.raw_value > 0">
                                            <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action" context="{'search_default_high_priority': 1}">
                                                <t t-esc="record.todo_request_count_high_priority.value"/>
                                                Top Priorities
                                            </a>
                                        </div>
                                        <div t-if="record.todo_request_count_block.raw_value > 0">
                                            <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action" context="{'search_default_kanban_state_block': 1}">
                                                <t t-esc="record.todo_request_count_block.value"/>
                                                Blocked
                                            </a>
                                        </div>
                                        <div t-if="record.todo_request_count_unscheduled.raw_value > 0">
                                            <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action" context="{'search_default_unscheduled': 1}">
                                                <t t-esc="record.todo_request_count_unscheduled.value"/>
                                                Unscheduled
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div><div class="container o_kanban_card_manage_pane dropdown-menu" role="menu">
                                <div class="row">
                                    <div class="col-6 o_kanban_card_manage_section o_kanban_manage_view">
                                        <div role="menuitem" class="o_kanban_card_manage_title">
                                            <span>Requests</span>
                                        </div>
                                        <div role="menuitem">
                                            <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action">
                                                All
                                            </a>
                                        </div>
                                        <div role="menuitem">
                                            <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action" context="{'search_default_todo': 1}">
                                                To Do
                                            </a>
                                        </div>
                                        <div role="menuitem">
                                            <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action" context="{'search_default_progress': 1}">
                                                In Progress
                                            </a>
                                        </div>
                                        <div role="menuitem">
                                            <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action" context="{'search_default_done': 1}">
                                                Done
                                            </a>
                                        </div>
                                    </div>
                                    <div class="col-6 o_kanban_card_manage_section o_kanban_manage_new">
                                        <div role="menuitem" class="o_kanban_card_manage_title">
                                            <span>Reporting</span>
                                        </div>
                                        <div role="menuitem">
                                            <a name="%(maintenance_request_action_reports)d" type="action" context="{'search_default_maintenance_team_id': active_id}">
                                            Maintenance Requests
                                            </a>
                                        </div>
                                    </div>
                                </div>
                                <div t-if="widget.editable" class="o_kanban_card_manage_settings row">
                                    <div class="col-8" role="menuitem" aria-haspopup="true">
                                        <ul role="menu" class="oe_kanban_colorpicker" data-field="color"/>
                                    </div>
                                    <div role="menuitem" class="col-4">
                                        <a type="edit" class="dropdown-item">Configuration</a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="maintenance_team_view_search" model="ir.ui.view">
        <field name="name">maintenance.team.search</field>
        <field name="model">maintenance.team</field>
        <field name="arch" type="xml">
            <search string="Search">
                <field string="Team" name="name"/>
                <filter string="Archived" domain="[('active', '=', False)]" name="inactive"/>
            </search>
        </field>
    </record>

    <!-- equipment.team : actions -->
    <record id="maintenance_team_action_settings" model="ir.actions.act_window">
        <field name="name">Teams</field>
        <field name="res_model">maintenance.team</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="search_view_id" ref="maintenance_team_view_search"/>
        <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('maintenance_team_view_tree')}),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('maintenance_team_view_kanban')})]"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a team in the maintenance request
          </p>
        </field>
    </record>

    <record id="maintenance_dashboard_action" model="ir.actions.act_window">
        <field name="name">Maintenance Teams</field>
        <field name="res_model">maintenance.team</field>
        <field name="view_mode">kanban,form</field>
        <field name="view_id" ref="maintenance_team_kanban"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a new stage in the maintenance request
          </p>
        </field>
    </record>


    <!-- Menu items hierachy -->
    <menuitem
        id="menu_maintenance_title"
        name="Maintenance"
        web_icon="maintenance,static/description/icon.png"
        sequence="110"/>

    <menuitem
        id="menu_m_dashboard"
        name="Dashboard"
        parent="menu_maintenance_title"
        groups="group_equipment_manager,base.group_user"
        action="maintenance_dashboard_action"
        sequence="0"/>

    <menuitem
        id="menu_m_request"
        name="Maintenance"
        parent="menu_maintenance_title"
        groups="group_equipment_manager,base.group_user"
        sequence="1"/>

    <menuitem
        id="menu_m_request_form"
        name="Maintenance Requests"
        parent="menu_m_request"
        action="hr_equipment_request_action"
        groups="group_equipment_manager,base.group_user"
        sequence="1"/>

    <menuitem
        id="menu_m_request_calendar"
        name="Maintenance Calendar"
        parent="menu_m_request"
        action="hr_equipment_request_action_cal"
        groups="group_equipment_manager,base.group_user"
        sequence="2"/>

    <menuitem
        id="menu_equipment_form"
        name="Equipments"
        parent="menu_maintenance_title"
        action="hr_equipment_action"
        groups="group_equipment_manager,base.group_user"
        sequence="2"/>

    <menuitem
        id="menu_m_reports"
        name="Reporting"
        parent="menu_maintenance_title"
        groups="group_equipment_manager,base.group_user"
        sequence="3"/>

    <menuitem
        id="menu_m_reports_oee"
        name="Overall Equipment Effectiveness (OEE)"
        parent="menu_m_reports"
        groups="group_equipment_manager,base.group_user"
        sequence="1"/>

    <menuitem
        id="menu_m_reports_losses"
        name="Losses Analysis"
        parent="menu_m_reports"
        groups="group_equipment_manager,base.group_user"
        sequence="2"/>

    <menuitem
        id="maintenance_reporting"
        name="Reporting"
        parent="menu_maintenance_title"
        sequence="20"/>
    <menuitem
        id="maintenance_request_reporting"
        action="maintenance_request_action_reports"
        parent="maintenance_reporting"/>

    <menuitem
        id="menu_maintenance_configuration"
        name="Configuration"
        parent="menu_maintenance_title"
        groups="group_equipment_manager"
        sequence="100"/>

    <menuitem
        id="menu_maintenance_teams"
        name="Maintenance Teams"
        parent="menu_maintenance_configuration"
        action="maintenance_team_action_settings"
        groups="group_equipment_manager"
        sequence="1"/>

    <menuitem
        id="menu_maintenance_cat"
        name="Equipment Categories"
        parent="menu_maintenance_configuration"
        action="hr_equipment_category_action"
        sequence="2"/>

    <menuitem
        id="menu_maintenance_stage_configuration"
        name="Maintenance Stages"
        parent="menu_maintenance_configuration"
        action="hr_equipment_stage_action"
        groups="base.group_no_one"
        sequence="3" />
</odoo>

```

