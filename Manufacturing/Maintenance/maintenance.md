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
Track equipment and maintenance requests""",
    'depends': ['mail'],
    'summary': 'Track equipment and manage maintenance requests',
    'website': 'https://www.odoo.com/app/maintenance',
    'data': [
        'security/maintenance.xml',
        'security/ir.model.access.csv',
        'data/maintenance_data.xml',
        'data/mail_activity_type_data.xml',
        'data/mail_message_subtype_data.xml',
        'views/maintenance_views.xml',
        'views/mail_activity_views.xml',
        'views/res_config_settings_views.xml',
    ],
    'demo': ['data/maintenance_demo.xml'],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'maintenance/static/src/**/*',
        ],
        'web.assets_tests': [
            'maintenance/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\mail_activity_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <!-- Maintenance-specific activities, for automatic generation mainly -->
    <record id="mail_act_maintenance_request" model="mail.activity.type">
        <field name="name">Maintenance Request</field>
        <field name="icon">fa-wrench</field>
        <field name="res_model">maintenance.request</field>
    </record>
</data>
</odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
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

    <!-- Equipment -->
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
from dateutil.relativedelta import relativedelta
from odoo.exceptions import ValidationError
from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.exceptions import UserError
from odoo.osv import expression


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
    note = fields.Html('Comments', translate=True)
    equipment_ids = fields.One2many('maintenance.equipment', 'category_id', string='Equipment', copy=False)
    equipment_count = fields.Integer(string="Equipment Count", compute='_compute_equipment_count')
    maintenance_ids = fields.One2many('maintenance.request', 'category_id', copy=False)
    maintenance_count = fields.Integer(string="Maintenance Count", compute='_compute_maintenance_count')
    maintenance_open_count = fields.Integer(string="Current Maintenance", compute='_compute_maintenance_count')
    alias_id = fields.Many2one(help="Email alias for this equipment category. New emails will automatically "
        "create a new equipment under this category.")
    fold = fields.Boolean(string='Folded in Maintenance Pipe', compute='_compute_fold', store=True)

    def _compute_equipment_count(self):
        equipment_data = self.env['maintenance.equipment']._read_group([('category_id', 'in', self.ids)], ['category_id'], ['__count'])
        mapped_data = {category.id: count for category, count in equipment_data}
        for category in self:
            category.equipment_count = mapped_data.get(category.id, 0)

    def _compute_maintenance_count(self):
        maintenance_data = self.env['maintenance.request']._read_group([('category_id', 'in', self.ids)], ['category_id', 'archive'], ['__count'])
        mapped_data = {(category.id, archive): count for category, archive, count in maintenance_data}
        for category in self:
            category.maintenance_open_count = mapped_data.get((category.id, False), 0)
            category.maintenance_count = category.maintenance_open_count + mapped_data.get((category.id, True), 0)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_contains_maintenance_requests(self):
        for category in self:
            if category.equipment_ids or category.maintenance_ids:
                raise UserError(_("You cannot delete an equipment category containing equipment or maintenance requests."))

    def _alias_get_creation_values(self):
        values = super(MaintenanceEquipmentCategory, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('maintenance.request').id
        if self.id:
            values['alias_defaults'] = defaults = ast.literal_eval(self.alias_defaults or "{}")
            defaults['category_id'] = self.id
        return values


class MaintenanceMixin(models.AbstractModel):
    _name = 'maintenance.mixin'
    _check_company_auto = True
    _description = 'Maintenance Maintained Item'

    company_id = fields.Many2one('res.company', string='Company',
        default=lambda self: self.env.company)
    effective_date = fields.Date('Effective Date', default=fields.Date.context_today, required=True, help="This date will be used to compute the Mean Time Between Failure.")
    maintenance_team_id = fields.Many2one('maintenance.team', string='Maintenance Team', compute='_compute_maintenance_team_id', store=True, readonly=False, check_company=True)
    technician_user_id = fields.Many2one('res.users', string='Technician', tracking=True)
    maintenance_ids = fields.One2many('maintenance.request')  # needs to be extended in order to specify inverse_name !
    maintenance_count = fields.Integer(compute='_compute_maintenance_count', string="Maintenance Count", store=True)
    maintenance_open_count = fields.Integer(compute='_compute_maintenance_count', string="Current Maintenance", store=True)
    expected_mtbf = fields.Integer(string='Expected MTBF', help='Expected Mean Time Between Failure')
    mtbf = fields.Integer(compute='_compute_maintenance_request', string='MTBF', help='Mean Time Between Failure, computed based on done corrective maintenances.')
    mttr = fields.Integer(compute='_compute_maintenance_request', string='MTTR', help='Mean Time To Repair')
    estimated_next_failure = fields.Date(compute='_compute_maintenance_request', string='Estimated time before next failure (in days)', help='Computed as Latest Failure Date + MTBF')
    latest_failure_date = fields.Date(compute='_compute_maintenance_request', string='Latest Failure Date')

    @api.depends('company_id')
    def _compute_maintenance_team_id(self):
        for record in self:
            if record.maintenance_team_id.company_id and record.maintenance_team_id.company_id.id != record.company_id.id:
                record.maintenance_team_id = False

    @api.depends('effective_date', 'maintenance_ids.stage_id', 'maintenance_ids.close_date', 'maintenance_ids.request_date')
    def _compute_maintenance_request(self):
        for record in self:
            maintenance_requests = record.maintenance_ids.filtered(lambda mr: mr.maintenance_type == 'corrective' and mr.stage_id.done)
            record.mttr = len(maintenance_requests) and (sum(int((request.close_date - request.request_date).days) for request in maintenance_requests) / len(maintenance_requests)) or 0
            record.latest_failure_date = max((request.request_date for request in maintenance_requests), default=False)
            record.mtbf = record.latest_failure_date and (record.latest_failure_date - record.effective_date).days / len(maintenance_requests) or 0
            record.estimated_next_failure = record.mtbf and record.latest_failure_date + relativedelta(days=record.mtbf) or False

    @api.depends('maintenance_ids.stage_id.done', 'maintenance_ids.archive')
    def _compute_maintenance_count(self):
        for record in self:
            record.maintenance_count = len(record.maintenance_ids)
            record.maintenance_open_count = len(record.maintenance_ids.filtered(lambda mr: not mr.stage_id.done and not mr.archive))


class MaintenanceEquipment(models.Model):
    _name = 'maintenance.equipment'
    _inherit = ['mail.thread', 'mail.activity.mixin', 'maintenance.mixin']
    _description = 'Maintenance Equipment'
    _check_company_auto = True

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'owner_user_id' in init_values and self.owner_user_id:
            return self.env.ref('maintenance.mt_mat_assign')
        return super(MaintenanceEquipment, self)._track_subtype(init_values)

    @api.depends('serial_no')
    def _compute_display_name(self):
        for record in self:
            if record.serial_no:
                record.display_name = record.name + '/' + record.serial_no
            else:
                record.display_name = record.name

    @api.model
    def _name_search(self, name, domain=None, operator='ilike', limit=None, order=None):
        domain = domain or []
        query = None
        if name and operator not in expression.NEGATIVE_TERM_OPERATORS and operator != '=':
            query = self._search([('name', '=', name)] + domain, limit=limit, order=order)
        return query or super()._name_search(name, domain, operator, limit, order)

    name = fields.Char('Equipment Name', required=True, translate=True)
    active = fields.Boolean(default=True)
    owner_user_id = fields.Many2one('res.users', string='Owner', tracking=True)
    category_id = fields.Many2one('maintenance.equipment.category', string='Equipment Category',
                                  tracking=True, group_expand='_read_group_category_ids')
    partner_id = fields.Many2one('res.partner', string='Vendor', check_company=True)
    partner_ref = fields.Char('Vendor Reference')
    location = fields.Char('Location')
    model = fields.Char('Model')
    serial_no = fields.Char('Serial Number', copy=False)
    assign_date = fields.Date('Assigned Date', tracking=True)
    cost = fields.Float('Cost')
    note = fields.Html('Note')
    warranty_date = fields.Date('Warranty Expiration Date')
    color = fields.Integer('Color Index')
    scrap_date = fields.Date('Scrap Date')
    maintenance_ids = fields.One2many('maintenance.request', 'equipment_id')

    @api.onchange('category_id')
    def _onchange_category_id(self):
        self.technician_user_id = self.category_id.technician_user_id

    _sql_constraints = [
        ('serial_no', 'unique(serial_no)', "Another asset already exists with this serial number!"),
    ]

    @api.model_create_multi
    def create(self, vals_list):
        equipments = super().create(vals_list)
        for equipment in equipments:
            if equipment.owner_user_id:
                equipment.message_subscribe(partner_ids=[equipment.owner_user_id.partner_id.id])
        return equipments

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
    company_id = fields.Many2one('res.company', string='Company', required=True,
        default=lambda self: self.env.company)
    description = fields.Html('Description')
    request_date = fields.Date('Request Date', tracking=True, default=fields.Date.context_today,
                               help="Date requested for the maintenance to happen")
    owner_user_id = fields.Many2one('res.users', string='Created by User', default=lambda s: s.env.uid)
    category_id = fields.Many2one('maintenance.equipment.category', related='equipment_id.category_id', string='Category', store=True, readonly=True)
    equipment_id = fields.Many2one('maintenance.equipment', string='Equipment',
                                   ondelete='restrict', index=True, check_company=True)
    user_id = fields.Many2one('res.users', string='Technician', compute='_compute_user_id', store=True, readonly=False, tracking=True)
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
    maintenance_team_id = fields.Many2one('maintenance.team', string='Team', required=True, default=_get_default_team_id,
                                          compute='_compute_maintenance_team_id', store=True, readonly=False, check_company=True)
    duration = fields.Float(help="Duration in hours.")
    done = fields.Boolean(related='stage_id.done')
    instruction_type = fields.Selection([
        ('pdf', 'PDF'), ('google_slide', 'Google Slide'), ('text', 'Text')],
        string="Instruction", default="text"
    )
    instruction_pdf = fields.Binary('PDF')
    instruction_google_slide = fields.Char('Google Slide', help="Paste the url of your Google Slide. Make sure the access to the document is public.")
    instruction_text = fields.Html('Text')
    recurring_maintenance = fields.Boolean(string="Recurrent", compute='_compute_recurring_maintenance', store=True, readonly=False)
    repeat_interval = fields.Integer(string='Repeat Every', default=1)
    repeat_unit = fields.Selection([
        ('day', 'Days'),
        ('week', 'Weeks'),
        ('month', 'Months'),
        ('year', 'Years'),
    ], default='week')
    repeat_type = fields.Selection([
        ('forever', 'Forever'),
        ('until', 'Until'),
    ], default="forever", string="Until")
    repeat_until = fields.Date(string="End Date")

    def archive_equipment_request(self):
        self.write({'archive': True, 'recurring_maintenance': False})

    def reset_equipment_request(self):
        """ Reinsert the maintenance request into the maintenance pipe in the first stage"""
        first_stage_obj = self.env['maintenance.stage'].search([], order="sequence asc", limit=1)
        # self.write({'active': True, 'stage_id': first_stage_obj.id})
        self.write({'archive': False, 'stage_id': first_stage_obj.id})

    @api.constrains('repeat_interval')
    def _check_repeat_interval(self):
        for record in self:
            if record.repeat_interval < 1:
                raise ValidationError("Repeat Interval cannot be less than 1.")

    @api.depends('company_id', 'equipment_id')
    def _compute_maintenance_team_id(self):
        for request in self:
            if request.equipment_id and request.equipment_id.maintenance_team_id:
                request.maintenance_team_id = request.equipment_id.maintenance_team_id.id
            if request.maintenance_team_id.company_id and request.maintenance_team_id.company_id.id != request.company_id.id:
                request.maintenance_team_id = False

    @api.depends('company_id', 'equipment_id')
    def _compute_user_id(self):
        for request in self:
            if request.equipment_id:
                request.user_id = request.equipment_id.technician_user_id or request.equipment_id.category_id.technician_user_id
            if request.user_id and request.company_id.id not in request.user_id.company_ids.ids:
                request.user_id = False

    @api.depends('maintenance_type')
    def _compute_recurring_maintenance(self):
        for request in self:
            if request.maintenance_type != 'preventive':
                request.recurring_maintenance = False

    @api.model_create_multi
    def create(self, vals_list):
        # context: no_log, because subtype already handle this
        maintenance_requests = super().create(vals_list)
        for request in maintenance_requests:
            if request.owner_user_id or request.user_id:
                request._add_followers()
            if request.equipment_id and not request.maintenance_team_id:
                request.maintenance_team_id = request.equipment_id.maintenance_team_id
            if request.close_date and not request.stage_id.done:
                request.close_date = False
            if not request.close_date and request.stage_id.done:
                request.close_date = fields.Date.today()
        maintenance_requests.activity_update()
        return maintenance_requests

    def write(self, vals):
        # Overridden to reset the kanban_state to normal whenever
        # the stage (stage_id) of the Maintenance Request changes.
        if vals and 'kanban_state' not in vals and 'stage_id' in vals:
            vals['kanban_state'] = 'normal'
        now = fields.Datetime.now()
        if 'stage_id' in vals and self.env['maintenance.stage'].browse(vals['stage_id']).done:
            for request in self:
                if request.maintenance_type != 'preventive' or not request.recurring_maintenance:
                    continue
                schedule_date = request.schedule_date or now
                schedule_date += relativedelta(**{f"{request.repeat_unit}s": request.repeat_interval})
                if request.repeat_type == 'forever' or schedule_date.date() <= request.repeat_until:
                    request.copy({'schedule_date': schedule_date, 'stage_id': request._default_stage().id})
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
        if self._need_new_activity(vals):
            # need to change description of activity also so unlink old and create new activity
            self.activity_unlink(['maintenance.mail_act_maintenance_request'])
            self.activity_update()
        return res

    def _need_new_activity(self, vals):
        return vals.get('equipment_id')

    def _get_activity_note(self):
        self.ensure_one()
        if self.equipment_id:
            return _('Request planned for %s', self.equipment_id._get_html_link())
        return False

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
                note = request._get_activity_note()
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
            team.todo_request_ids = self.env['maintenance.request'].search([('maintenance_team_id', '=', team.id), ('stage_id.done', '=', False), ('archive', '=', False)])
            data = self.env['maintenance.request']._read_group(
                [('maintenance_team_id', '=', team.id), ('stage_id.done', '=', False), ('archive', '=', False)],
                ['schedule_date:year', 'priority', 'kanban_state'],
                ['__count']
            )
            team.todo_request_count = sum(count for (_, _, _, count) in data)
            team.todo_request_count_date = sum(count for (schedule_date, _, _, count) in data if schedule_date)
            team.todo_request_count_high_priority = sum(count for (_, priority, _, count) in data if priority == 3)
            team.todo_request_count_block = sum(count for (_, _, kanban_state, count) in data if kanban_state == 'blocked')
            team.todo_request_count_unscheduled = team.todo_request_count - team.todo_request_count_date

    @api.depends('equipment_ids')
    def _compute_equipment(self):
        for team in self:
            team.equipment_count = len(team.equipment_ids)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_maintenance_worksheet = fields.Boolean(string="Custom Maintenance Worksheets")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import maintenance
from . import res_config_settings

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
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        <field name="comment">The user will be able to manage equipment.</field>
    </record>

    <data noupdate="1">

    <!-- Rules -->
    <record id="equipment_request_rule_user" model="ir.rule">
        <field name="name">Users are allowed to access their own maintenance requests</field>
        <field name="model_id" ref="model_maintenance_request"/>
        <field name="domain_force">['|', '|', ('owner_user_id', '=', user.id), ('message_partner_ids', 'in', [user.partner_id.id]), ('user_id', '=', user.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="equipment_rule_user" model="ir.rule">
        <field name="name">Users are allowed to access equipment they follow</field>
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
        <field name="name">Equipment administrator</field>
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

    </data>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M12.172 24.193s6.121-2.586 7.778-4.243c1.657-1.657 4.243-7.778 4.243-7.778l13.435 13.435s-6.099 2.563-7.779 4.242c-1.68 1.68-4.242 7.779-4.242 7.779L12.172 24.193Z" fill="#2EBCFA"/><path fill-rule="evenodd" clip-rule="evenodd" d="M42.577 22.778c4.296-4.296 4.296-11.26 0-15.556-4.296-4.296-11.26-4.296-15.556 0-4.296 4.296-4.296 11.26 0 15.556 4.295 4.296 11.26 4.296 15.556 0Zm-4.243-4.242a5 5 0 1 0-7.07-7.072 5 5 0 0 0 7.07 7.071ZM22.778 42.577c4.296-4.296 4.296-11.26 0-15.556-4.296-4.296-11.26-4.296-15.556 0-4.296 4.295-4.296 11.26 0 15.556 4.296 4.296 11.26 4.296 15.556 0Zm-4.242-4.243a5 5 0 1 0-7.071-7.07 5 5 0 0 0 7.07 7.07Z" fill="#088BF5"/></svg>

```

## File: static\src\views\calendar_with_recurrence\calendar_with_recurrence_common_popover.js

```javascript
/** @odoo-module **/

import { CalendarCommonPopover } from "@web/views/calendar/calendar_common/calendar_common_popover";

export class CalendarWithRecurrenceCommonPopover extends CalendarCommonPopover {
    onEditEvent() {
        this.props.record.id = this.props.record.rawRecord.id;
        super.onEditEvent();
    }
    onDeleteEvent() {
        this.props.record.id = this.props.record.rawRecord.id;
        super.onDeleteEvent();
    }
}

```

## File: static\src\views\calendar_with_recurrence\calendar_with_recurrence_common_renderer.js

```javascript
/** @odoo-module **/

import { CalendarCommonRenderer } from "@web/views/calendar/calendar_common/calendar_common_renderer";
import { CalendarWithRecurrenceCommonPopover } from "./calendar_with_recurrence_common_popover";

export class CalendarWithRecurrenceCommonRenderer extends CalendarCommonRenderer {
    onDblClick(info) {
        const record = this.props.model.records[info.event.id];
        this.props.editRecord({ ...record, id: record.rawRecord.id });
    }

    fcEventToRecord(event) {
        const record = super.fcEventToRecord(event);
        if (record.id) {
            record.id = this.props.model.records[record.id].rawRecord.id;
        }
        return record;
    }

    convertRecordToEvent(record) {
        const event = super.convertRecordToEvent(record);
        // https://fullcalendar.io/docs/editable
        // this is used to disable the 'drag and drop' and 'resizing' for recurring events
        event.editable = !record.isRecurrent;
        return event;
    }
}

CalendarWithRecurrenceCommonRenderer.components = {
    ...CalendarCommonRenderer.components,
    Popover: CalendarWithRecurrenceCommonPopover,
};

```

## File: static\src\views\calendar_with_recurrence\calendar_with_recurrence_model.js

```javascript
/** @odoo-module */

import { deserializeDateTime, serializeDateTime } from "@web/core/l10n/dates";
import { CalendarModel } from '@web/views/calendar/calendar_model';

export class CalendarWithRecurrenceModel extends CalendarModel {
    async loadRecords(data) {
        const rawRecords = await this.fetchRecords(data);
        const records = {};
        let recordsCounter = 1;
        for (const rawRecord of rawRecords) {
            records[recordsCounter] = {
                ...this.normalizeRecord(rawRecord),
                id: recordsCounter,
            };
            recordsCounter++;
            if (rawRecord.recurring_maintenance && !rawRecord.done && !rawRecord.archive) {
                let { start, end } = data.range;
                if (rawRecord.repeat_type == 'until') {
                    end = luxon.DateTime.min(end, deserializeDateTime(rawRecord.repeat_until)).endOf('day');
                }
                let date = deserializeDateTime(rawRecord.schedule_date);
                date = this._getNextDate(date, rawRecord.repeat_unit + 's', rawRecord.repeat_interval);
                let counter = 1;
                while (date <= end) {
                    if (date > start) {
                        const rawRecordCopy = { ...rawRecord };
                        rawRecordCopy.display_name = rawRecord.display_name + " (+" + counter + ")";
                        rawRecordCopy.schedule_date = serializeDateTime(date);
                        records[recordsCounter] = {
                            ...this.normalizeRecord(rawRecordCopy),
                            id: recordsCounter,
                            isRecurrent: true,
                        };
                        recordsCounter++;
                    }
                    date = this._getNextDate(date, rawRecord.repeat_unit + 's', rawRecord.repeat_interval);
                    counter++;
                }
            }
        }
        return records;
    }
    _getNextDate(date, unit, interval) {
        return date.plus({ [unit]: interval });
    }
}

```

## File: static\src\views\calendar_with_recurrence\calendar_with_recurrence_renderer.js

```javascript
/** @odoo-module **/

import { CalendarRenderer } from "@web/views/calendar/calendar_renderer";
import { CalendarWithRecurrenceCommonRenderer } from './calendar_with_recurrence_common_renderer';
import { CalendarWithRecurrenceYearRenderer } from './calendar_with_recurrence_year_renderer';

export class CalendarWithRecurrenceRenderer extends CalendarRenderer { }

CalendarWithRecurrenceRenderer.components = {
    ...CalendarRenderer.components,
    day: CalendarWithRecurrenceCommonRenderer,
    week: CalendarWithRecurrenceCommonRenderer,
    month: CalendarWithRecurrenceCommonRenderer,
    year: CalendarWithRecurrenceYearRenderer,
};

```

## File: static\src\views\calendar_with_recurrence\calendar_with_recurrence_view.js

```javascript
/** @odoo-module */

import { calendarView } from '@web/views/calendar/calendar_view';
import { CalendarWithRecurrenceModel } from './calendar_with_recurrence_model';
import { CalendarWithRecurrenceRenderer } from './calendar_with_recurrence_renderer';
import { registry } from '@web/core/registry';

const CalendarWithRecurrenceView = {
    ...calendarView,
    Model: CalendarWithRecurrenceModel,
    Renderer: CalendarWithRecurrenceRenderer,
};

registry.category('views').add('calendar_with_recurrence', CalendarWithRecurrenceView);

```

## File: static\src\views\calendar_with_recurrence\calendar_with_recurrence_year_popover.js

```javascript
/** @odoo-module **/

import { CalendarYearPopover } from "@web/views/calendar/calendar_year/calendar_year_popover";

export class CalendarWithRecurrenceYearPopover extends CalendarYearPopover {
    onRecordClick(record) {
        record.id = record.rawRecord.id;
        super.onRecordClick(record);
    }
}

```

## File: static\src\views\calendar_with_recurrence\calendar_with_recurrence_year_renderer.js

```javascript
/** @odoo-module **/

import { CalendarWithRecurrenceYearPopover } from "./calendar_with_recurrence_year_popover";
import { CalendarYearRenderer } from "@web/views/calendar/calendar_year/calendar_year_renderer";

export class CalendarWithRecurrenceYearRenderer extends CalendarYearRenderer { }

CalendarWithRecurrenceYearRenderer.components = {
    ...CalendarYearRenderer.components,
    Popover: CalendarWithRecurrenceYearPopover,
};

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
        <field name="domain">['|', ('res_model', '=', False), ('res_model', '=', 'maintenance.request')]</field>
        <field name="context">{'default_res_model': 'maintenance.request'}</field>
    </record>
    <menuitem id="maintenance_menu_config_activity_type"
        action="mail_activity_type_action_config_maintenance"
        parent="menu_maintenance_configuration"
        sequence="20"
        groups="base.group_no_one"/>
</odoo>
```

## File: views\maintenance_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- equiment.request: views -->
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
                <filter string="Done" name="done" domain="[('stage_id.done', '=', True)]"/>
                <separator/>
                <filter string="Blocked" name="kanban_state_block" domain="[('stage_id.done', '=', False), ('kanban_state', '=', 'blocked')]"/>
                <filter string="Ready" name="done" domain="[('stage_id.done', '=', False), ('kanban_state', '=', 'done')]"/>
                <separator/>
                <filter string="High-priority" name="high_priority" domain="[('stage_id.done', '=', False), ('priority', '=', '3')]"/>
                <separator/>
                <filter string="Unscheduled" name="unscheduled" domain="[('stage_id.done', '=', False), ('schedule_date', '=', False)]"/>
                <separator/>
                <filter name="filter_request_date" date="request_date"/>
                <filter name="filter_schedule_date" date="schedule_date"/>
                <filter name="filter_close_date" date="close_date"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]" groups="mail.group_mail_notification_type_inbox"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <separator/>
                <filter string="Active" name="active" domain="[('archive', '=', False)]"/>
                <filter string="Cancelled" name="inactive" domain="[('archive', '=', True)]"/>
                <group  expand='0' string='Group by...'>
                    <filter string='Assigned to' name="assigned" domain="[]" context="{'group_by': 'user_id'}"/>
                    <filter string='Category' name="category" domain="[]" context="{'group_by' : 'category_id'}"/>
                    <filter string='Stage' name="stages" domain="[]" context="{'group_by' : 'stage_id'}"/>
                    <filter string='Created By' name='created_by' domain="[]" context="{'group_by': 'owner_user_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="maintenance_request_view_activity" model="ir.ui.view">
        <field name="name">maintenance.request.view.activity</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <activity string="Maintenance Request">
                <field name="user_id"/>
                <templates>
                    <div t-name="activity-box">
                        <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                        <div class="flex-grow-1 d-block">
                            <div class="d-flex justify-content-between">
                                <field name="name" display="full" class="o_text_block o_text_bold"/>
                            </div>
                            <field name="equipment_id" muted="1" display="full" class="o_text_block"/>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record id="hr_equipment_request_view_form" model="ir.ui.view">
        <field name="name">equipment.request.form</field>
        <field name="model">maintenance.request</field>
        <field name="arch" type="xml">
            <form string="Maintenance Request">
                <field name="company_id" invisible="1"/>
                <field name="category_id" invisible="1"/>
                <header>
                    <button string="Cancel" name="archive_equipment_request" type="object" invisible="archive"/>
                    <button string="Reopen Request" name="reset_equipment_request" type="object" invisible="not archive"/>
                    <field name="stage_id" widget="statusbar" options="{'clickable': '1'}" invisible="archive"/>
                </header>
                <sheet>
                    <div invisible="not archive">
                        <span class="badge text-bg-warning float-end">Canceled</span>
                    </div>
                    <field name="kanban_state" widget="state_selection"/>
                    <div class="oe_title">
                        <label for="name" string="Request"/>
                        <h1>
                            <field name="name" placeholder="e.g. Screen not working"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="owner_user_id" string="Requested By" invisible="1"/>
                            <field name="equipment_id"  context="{'default_company_id':company_id, 'default_category_id':category_id}"/>
                            <field name="category_id" groups="maintenance.group_equipment_manager" context="{'default_company_id':company_id}" invisible="not equipment_id"/>
                            <field name="request_date" readonly="True"/>
                            <field name="done" invisible="1"/>
                            <field name="close_date" readonly="True" invisible="not done"/>
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
                            <label for="recurring_maintenance" invisible="maintenance_type == 'corrective'"/>
                            <div class="d-inline-flex" invisible="maintenance_type == 'corrective'">
                                <field name="recurring_maintenance" nolabel="1" class="ms-0" style="width: fit-content;"/>
                            </div>
                            <label for="repeat_interval" invisible="not recurring_maintenance"/>
                            <div class="d-flex" invisible="not recurring_maintenance">
                                <field name="repeat_interval" required="recurring_maintenance" class="me-2" style="max-width: 2rem !important;" />
                                <field name="repeat_unit" required="recurring_maintenance" class="me-2" style="max-width: 4rem !important;" />
                                <field name="repeat_type" required="recurring_maintenance" class="me-2" style="max-width: 15rem !important;" />
                                <field name="repeat_until" invisible="repeat_type != 'until'" required="repeat_type == 'until'" class="me-2" />
                            </div>
                            <field name="priority" widget="priority"/>
                            <field name="email_cc" string="Email cc" groups="base.group_no_one"/>
                            <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Notes">
                            <field name='description' placeholder="Internal Notes"/>
                        </page>
                        <page string="Instructions">
                            <group col="1">
                                <field name="instruction_type" widget="radio" nolabel="1"/>
                                <field name="instruction_pdf" help="Upload your PDF file." widget="pdf_viewer" invisible="instruction_type != 'pdf'" required="instruction_type == 'pdf'" nolabel="1"/>
                                <field name="instruction_google_slide" placeholder="Google Slide Link" widget="embed_viewer" invisible="instruction_type != 'google_slide'" required="instruction_type == 'google_slide'" nolabel="1"/>
                                <field name="instruction_text" placeholder="Your instructions" invisible="instruction_type != 'text'" nolabel="1"/>
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
                <field name="archive" />
                <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger"}'/>
                <templates>
                    <t t-name="kanban-tooltip">
                       <ul class="oe_kanban_tooltip">
                          <li t-if="record.category_id.raw_value"><b>Category:</b> <t t-esc="record.category_id.value"/></li>
                          <li t-if="record.user_id.raw_value"><b>Request to:</b> <t t-esc="record.user_id.value"/></li>
                       </ul>
                    </t>
                    <t t-name="kanban-menu">
                        <t t-if="widget.editable"><a role="menuitem" type="edit" class="dropdown-item">Edit...</a></t>
                        <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click oe_semantic_html_override">
                            <div class="oe_kanban_content" tooltip="kanban-tooltip">
                                <div class="o_kanban_record_top">
                                    <b class="o_kanban_record_title"><field name="name"/></b>
                                </div>
                                <div class="o_kanban_record_body">
                                    <span name="owner_user_id" t-if="record.owner_user_id.raw_value">Requested by: <field name="owner_user_id"/><br/></span>
                                    <span class="oe_grey" t-if="record.equipment_id.raw_value"><field name="equipment_id"/><span t-if="record.category_id.raw_value"> (<field name="category_id"/>)</span><br/></span>
                                    <span name="schedule_date" t-if="record.schedule_date.raw_value"><field name="schedule_date"/><br/></span>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <field name="priority" widget="priority"/>
                                        <div class="o_kanban_inline_block ml4 mr4">
                                            <field name="activity_ids" widget="kanban_activity" />
                                        </div>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <div invisible="not archive">
                                            <span class="badge text-bg-warning float-end">Cancelled</span>
                                        </div>
                                        <field name="kanban_state" widget="state_selection"/>
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>
                                </div>
                            </div>
                            <div class="clearfix"></div>
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
                <field name="message_needaction" column_invisible="True"/>
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
            <calendar date_start="schedule_date" date_delay="duration" color="user_id" event_limit="5" js_class="calendar_with_recurrence">
                <field name="user_id" filters="1"/>
                <field name="priority"/>
                <field name="maintenance_type"/>
                <field name="recurring_maintenance" invisible="1"/>
                <field name="repeat_interval" invisible="1"/>
                <field name="repeat_unit" invisible="1"/>
                <field name="repeat_type" invisible="1"/>
                <field name="repeat_until" invisible="1"/>
                <field name="done" invisible="1"/>
                <field name="archive" invisible="1"/>
            </calendar>
        </field>
    </record>

    <!-- equiment.request: actions -->
    <record id="hr_equipment_request_action" model="ir.actions.act_window">
        <field name="name">Maintenance Requests</field>
        <field name="res_model">maintenance.request</field>
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar,activity</field>
        <field name="view_id" ref="hr_equipment_request_view_kanban"/>
        <field name="context">{
            'search_default_active': True,
            'default_user_id': uid
        }</field>
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
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar,activity</field>
        <field name="search_view_id" ref="hr_equipment_request_view_search"/>
        <field name="view_id" ref="hr_equipment_request_view_kanban"/>
        <field name="context">{
            'search_default_category_id': [active_id],
            'search_default_active': True,
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
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar,activity</field>
        <field name="context">{
            'search_default_active': True,
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
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar,activity</field>
        <field name="context">{
            'search_default_active': True,
            'search_default_maintenance_team_id': active_id,
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
        <field name="view_mode">calendar,kanban,tree,form,pivot,graph,activity</field>
        <field name="view_id" ref="hr_equipment_view_calendar"/>
        <field name="context">{
            'search_default_active': True,
            'search_default_todo': True,
        }</field>
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
        <field name="view_mode">graph,pivot,kanban,tree,form,calendar,activity</field>
        <field name="context">{
            'search_default_active': True,
        }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new maintenance request
            </p><p>
                Follow the process of the request and communicate with the collaborator.
            </p>
        </field>
    </record>

    <!-- equiment: views -->
    <record id="hr_equipment_view_form" model="ir.ui.view">
        <field name="name">equipment.form</field>
        <field name="model">maintenance.equipment</field>
        <field name="arch" type="xml">
            <form string="Equipment">
                <sheet>
                    <field name="company_id" invisible="1"/>
                    <div class="oe_button_box" name="button_box">
                        <button name="%(hr_equipment_request_action_from_equipment)d"
                            type="action"
                            class="oe_stat_button"
                            context="{'search_default_active': True, 'default_company_id': company_id, 'default_maintenance_team_id': maintenance_team_id}"
                            icon="fa-wrench">
                            <field string="Maintenance" name="maintenance_open_count" widget="statinfo"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1><field name="name" string="Name" placeholder="e.g. LED Monitor"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="category_id" options="{&quot;no_open&quot;: True}" context="{'default_company_id':company_id}"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                            <field name="owner_user_id" string="Owner"/>
                        </group>
                        <group>
                            <field name="maintenance_team_id" context="{'default_company_id':company_id}"/>
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
                                <group name="statistics">
                                    <label for="expected_mtbf" string="Expected Mean Time Between Failure"/>
                                    <div class="o_row">
                                        <field name="expected_mtbf"/> days
                                    </div>
                                    <label for="mtbf" string="Mean Time Between Failure"/>
                                    <div class="o_row">
                                        <field name="mtbf" /> days
                                    </div>
                                    <label for="estimated_next_failure" string="Estimated Next Failure"/>
                                    <div class="o_row">
                                        <field name="estimated_next_failure" />
                                    </div>
                                    <field name="latest_failure_date" string="Latest Failure" />
                                    <label for="mttr" string="Mean Time To Repair"/>
                                    <div class="o_row">
                                        <field name="mttr" /> days
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
                    <t t-name="kanban-menu">
                        <t t-if="widget.editable"><a role="menuitem" type="edit" class="dropdown-item">Edit...</a></t>
                        <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                        <div role="separator" class="dropdown-divider"></div>
                        <div role="separator" class="dropdown-header">Record Colour</div>
                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click">
                            <div class="oe_kanban_content" tooltip="kanban-tooltip">
                                <div class="o_kanban_record_top">
                                    <b class="o_kanban_record_title"><field name="name"/><small><span t-if="record.model.raw_value"> (<field name="model"/>)</span></small></b>
                                </div>
                                <div class="o_kanban_record_body">
                                    <div t-if="record.serial_no.raw_value"><field name="serial_no"/></div>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <div class="badge text-bg-danger" t-if="!selection_mode and record.maintenance_open_count.raw_value" >
                                            <t t-out="record.maintenance_open_count.raw_value"/> Request
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
                            <div class="clearfix"></div>
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
                <field name="message_needaction" column_invisible="True"/>
                <field name="name"/>
                <!-- <field name="active" invisible="1"/> -->
                <field name="owner_user_id" string="Owner"/>
                <field name="assign_date" groups="base.group_no_one"/>
                <field name="serial_no"/>
                <field name="technician_user_id"/>
                <field name="category_id"/>
                <field name="partner_id" column_invisible="True"/>
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
                <filter string="My Equipment" name="my" domain="[('owner_user_id', '=', uid)]"/>
                <filter string="Assigned" name="assigned" domain="[('owner_user_id', '!=', False)]"/>
                <filter string="Unassigned" name="available" domain="[('owner_user_id', '=', False)]"/>
                <separator/>
                <filter string="Under Maintenance" name="under_maintenance" domain="[('maintenance_open_count', '&gt;', 0)]"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction','=',True)]" groups="mail.group_mail_notification_type_inbox"/>
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
        <field name="name">Equipment</field>
        <field name="res_model">maintenance.equipment</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="view_id" ref="hr_equipment_view_kanban"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new equipment
            </p><p>
                Track equipment and link it to an employee or department.
                You will be able to manage allocations, issues and maintenance of your equipment.
            </p>
        </field>
    </record>

    <!-- equiment: actions -->
    <record id="hr_equipment_action_from_category_form" model="ir.actions.act_window">
        <field name="name">Equipment</field>
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
                Track equipment and link it to an employee or department.
                You will be able to manage allocations, issues and maintenance of your equipment.
            </p>
        </field>
    </record>

    <!-- equipment.category: views -->
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
                        <field string="Maintenance" name="maintenance_open_count" widget="statinfo"/>
                    </button>
                </div>
                <div class="oe_title">
                    <label for="name" string="Category Name"/>
                    <h1>
                        <field name="name" placeholder="e.g. Monitors"/>
                    </h1>
                </div>
                <group>
                    <field name="technician_user_id" class="oe_inline" domain="[('share', '=', False)]"/>
                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" class="oe_inline"/>
                </group>
                <group name="group_alias">
                    <label for="alias_name" string="Email Alias"/>
                    <div name="alias_def">
                        <field name="alias_id" class="oe_read_only oe_inline" string="Email Alias" required="0"/>
                        <div class="oe_edit_only oe_inline" name="edit_alias" style="display: inline;" dir="ltr">
                            <field name="alias_name" class="oe_inline"/>@
                            <field name="alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                                   options="{'no_create': True, 'no_open': True}"/>
                        </div>
                    </div>
                </group>
                <group>
                    <field name="note"/>
                </group>
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
                                    <span class="badge rounded-pill">
                                        <strong>Equipment:</strong> <field name="equipment_count"/>
                                    </span>
                                </div>
                                <div class="col-6 text-end">
                                    <span class="badge rounded-pill">
                                        <strong>Maintenance:</strong> <field name="maintenance_open_count"/>
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

    <!-- equipment.category: actions -->
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

    <!-- equipment.stage: views -->
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

    <!-- equipment.stages: actions -->
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
                <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                <div class="oe_title">
                    <label for="name" string="Team Name"/>
                    <h1>
                        <field name="name" placeholder="e.g. Internal Maintenance"/>
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
            <kanban class="o_kanban_dashboard o_maintenance_team_kanban" create="0">
                <field name="name"/>
                <field name="color"/>
                <field name="todo_request_ids"/>
                <field name="todo_request_count"/>
                <field name="todo_request_count_date"/>
                <field name="todo_request_count_high_priority"/>
                <field name="todo_request_count_block"/>
                <field name="todo_request_count_unscheduled"/>
                <templates>
                    <t t-name="kanban-menu">
                        <div class="container">
                            <div class="row">
                                <div class="col-6 o_kanban_card_manage_section o_kanban_manage_view">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>Requests</span>
                                    </h5>
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
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>Reporting</span>
                                    </h5>
                                    <div role="menuitem">
                                        <a name="%(maintenance_request_action_reports)d" type="action" context="{'search_default_maintenance_team_id': id}">
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
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                            <div t-attf-class="o_kanban_card_header">
                                <div class="o_kanban_card_header_title">
                                    <div class="o_primary">
                                        <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action">
                                            <field name="name"/>
                                        </a></div>
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
                                            <a name="%(hr_equipment_request_action_cal)d" type="action" context="{'search_default_todo': 1, 'search_default_maintenance_team_id': id}">
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
                                            <a name="%(hr_equipment_todo_request_action_from_dashboard)d" type="action" context="{'search_default_todo': 1, 'search_default_unscheduled': 1}">
                                                <t t-esc="record.todo_request_count_unscheduled.value"/>
                                                Unscheduled
                                            </a>
                                        </div>
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

    <!-- equipment.team: actions -->
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
        sequence="160"/>

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
        name="Equipment"
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

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.maintenance</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="35"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form" />
            <field name="arch" type="xml">
                <xpath expr="//form" position="inside">
                    <app data-string="Maintenance" string="Maintenance" name="maintenance" groups="maintenance.group_equipment_manager">
                        <block title="Custom Worksheets">
                            <setting help="Create custom worksheet templates">
                                <field name="module_maintenance_worksheet" widget="upgrade_boolean"/>
                            </setting>
                        </block>
                    </app>
                </xpath>
            </field>
        </record>

        <record id="action_maintenance_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'maintenance', 'bin_size': False}</field>
        </record>

        <menuitem id="menu_maintenance_config" name="Settings" parent="menu_maintenance_configuration"
            sequence="0" action="action_maintenance_configuration" groups="base.group_system"/>

    </data>
</odoo>

```

