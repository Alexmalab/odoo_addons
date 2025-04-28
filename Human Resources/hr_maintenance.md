# Odoo Module: hr_maintenance

Category: Human Resources

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

{
    'name': 'Maintenance - HR',
    'version': '1.0',
    'sequence': 125,
    'category': 'Human Resources',
    'description': """
Bridge between HR and Maintenance.""",
    'depends': ['hr', 'maintenance'],
    'summary': 'Equipment, Assets, Internal Hardware, Allocation Tracking',
    'data': [
        'security/equipment.xml',
        'views/maintenance_views.xml',
        'views/hr_views.xml',
        'wizard/hr_departure_wizard_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\equipment.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, tools


class MaintenanceEquipment(models.Model):
    _inherit = 'maintenance.equipment'

    employee_id = fields.Many2one('hr.employee', compute='_compute_equipment_assign',
        store=True, readonly=False, string='Assigned Employee', tracking=True)
    department_id = fields.Many2one('hr.department', compute='_compute_equipment_assign',
        store=True, readonly=False, string='Assigned Department', tracking=True)
    equipment_assign_to = fields.Selection(
        [('department', 'Department'), ('employee', 'Employee'), ('other', 'Other')],
        string='Used By',
        required=True,
        default='employee')
    owner_user_id = fields.Many2one(compute='_compute_owner', store=True)
    assign_date = fields.Date(compute='_compute_equipment_assign', store=True, readonly=False, copy=True)

    @api.depends('employee_id', 'department_id', 'equipment_assign_to')
    def _compute_owner(self):
        for equipment in self:
            equipment.owner_user_id = self.env.user.id
            if equipment.equipment_assign_to == 'employee':
                equipment.owner_user_id = equipment.employee_id.user_id.id
            elif equipment.equipment_assign_to == 'department':
                equipment.owner_user_id = equipment.department_id.manager_id.user_id.id

    @api.depends('equipment_assign_to')
    def _compute_equipment_assign(self):
        for equipment in self:
            if equipment.equipment_assign_to == 'employee':
                equipment.department_id = False
                equipment.employee_id = equipment.employee_id
            elif equipment.equipment_assign_to == 'department':
                equipment.employee_id = False
                equipment.department_id = equipment.department_id
            else:
                equipment.department_id = equipment.department_id
                equipment.employee_id = equipment.employee_id
            equipment.assign_date = fields.Date.context_today(self)

    @api.model_create_multi
    def create(self, vals_list):
        equipments = super().create(vals_list)
        for equipment in equipments:
            # subscribe employee or department manager when equipment assign to him.
            partner_ids = []
            if equipment.employee_id and equipment.employee_id.user_id:
                partner_ids.append(equipment.employee_id.user_id.partner_id.id)
            if equipment.department_id and equipment.department_id.manager_id and equipment.department_id.manager_id.user_id:
                partner_ids.append(equipment.department_id.manager_id.user_id.partner_id.id)
            if partner_ids:
                equipment.message_subscribe(partner_ids=partner_ids)
        return equipments

    def write(self, vals):
        partner_ids = []
        # subscribe employee or department manager when equipment assign to employee or department.
        if vals.get('employee_id'):
            user_id = self.env['hr.employee'].browse(vals['employee_id'])['user_id']
            if user_id:
                partner_ids.append(user_id.partner_id.id)
        if vals.get('department_id'):
            department = self.env['hr.department'].browse(vals['department_id'])
            if department and department.manager_id and department.manager_id.user_id:
                partner_ids.append(department.manager_id.user_id.partner_id.id)
        if partner_ids:
            self.message_subscribe(partner_ids=partner_ids)
        return super(MaintenanceEquipment, self).write(vals)

    def _track_subtype(self, init_values):
        self.ensure_one()
        if ('employee_id' in init_values and self.employee_id) or ('department_id' in init_values and self.department_id):
            return self.env.ref('maintenance.mt_mat_assign')
        return super(MaintenanceEquipment, self)._track_subtype(init_values)


class MaintenanceRequest(models.Model):
    _inherit = 'maintenance.request'

    @api.returns('self')
    def _default_employee_get(self):
        return self.env.user.employee_id

    employee_id = fields.Many2one('hr.employee', string='Employee', default=_default_employee_get)
    owner_user_id = fields.Many2one(compute='_compute_owner', store=True)
    equipment_id = fields.Many2one(domain="['|', ('employee_id', '=', employee_id), ('employee_id', '=', False)]")

    @api.depends('employee_id')
    def _compute_owner(self):
        for r in self:
            if r.equipment_id.equipment_assign_to == 'employee':
                r.owner_user_id = r.employee_id.user_id.id
            else:
                r.owner_user_id = False

    @api.model_create_multi
    def create(self, vals_list):
        requests = super().create(vals_list)
        for request in requests:
            if request.employee_id.user_id:
                request.message_subscribe(partner_ids=[request.employee_id.user_id.partner_id.id])
        return requests

    def write(self, vals):
        if vals.get('employee_id'):
            employee = self.env['hr.employee'].browse(vals['employee_id'])
            if employee and employee.user_id:
                self.message_subscribe(partner_ids=[employee.user_id.partner_id.id])
        return super(MaintenanceRequest, self).write(vals)

    @api.model
    def message_new(self, msg, custom_values=None):
        """ Overrides mail_thread message_new that is called by the mailgateway
            through message_process.
            This override updates the document according to the email.
        """
        if custom_values is None:
            custom_values = {}
        email = tools.email_split(msg.get('from')) and tools.email_split(msg.get('from'))[0] or False
        user = self.env['res.users'].search([('login', '=', email)], limit=1)
        if user:
            employee = self.env.user.employee_id
            if employee:
                custom_values['employee_id'] = employee and employee[0].id
        return super(MaintenanceRequest, self).message_new(msg, custom_values=custom_values)

```

## File: models\res_users.py

```python
from odoo import api, models, fields


class Users(models.Model):
    _inherit = 'res.users'

    equipment_ids = fields.One2many('maintenance.equipment', 'owner_user_id', string="Managed Equipment")
    equipment_count = fields.Integer(related='employee_id.equipment_count', string="Assigned Equipment")

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['equipment_count']


class Employee(models.Model):
    _inherit = 'hr.employee'

    equipment_ids = fields.One2many('maintenance.equipment', 'employee_id', groups="hr.group_hr_user")
    equipment_count = fields.Integer('Equipment Count', compute='_compute_equipment_count', groups="hr.group_hr_user")

    @api.depends('equipment_ids')
    def _compute_equipment_count(self):
        for employee in self:
            employee.equipment_count = len(employee.equipment_ids)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import equipment
from . import res_users

```

## File: security\equipment.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- HR officers and HR managers are allowed to manage equipment -->
    <record id="hr.group_hr_user" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('maintenance.group_equipment_manager'))]"/>
    </record>

</odoo>

```

## File: views\hr_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_view_form" model="ir.ui.view">
        <field name="name">hr.employee.view.form.inherit.maintenance</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="priority" eval="50"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button name="%(maintenance.hr_equipment_action)d"
                    context="{'search_default_employee_id': id, 'default_employee_id': id}"
                    groups="maintenance.group_equipment_manager"
                    class="o_stat_button"
                    icon="fa-cubes"
                    type="action">
                    <field name="equipment_count" widget="statinfo"/>
                </button>
            </div>
        </field>
    </record>

    <!-- user preferences -->
    <record id="res_users_view_form_preference" model="ir.ui.view">
        <field name="name">res.users.view.form.inherit.maintenance</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="employee_id" invisible="1"/>
                <button name="%(maintenance.hr_equipment_action)d"
                    context="{'search_default_employee_id': employee_id, 'default_employee_id': employee_id}"
                    class="o_stat_button"
                    icon="fa-cubes"
                    type="action">
                    <field name="equipment_count" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\maintenance_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- equiment.request : views -->
    <record id="maintenance_request_view_search_inherit_hr" model="ir.ui.view">
        <field name="name">maintenance.request.view.search.inherit.hr</field>
        <field name="model">maintenance.request</field>
        <field name="inherit_id" ref="maintenance.hr_equipment_request_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="field[@name='user_id']" position="after">
                <field name="employee_id"/>
            </xpath>
            <xpath expr="//filter[@name='my_maintenances']" position="replace">
                <filter string="My Maintenances" name="my_maintenances" domain="[('employee_id.user_id', '=', uid)]"/>
            </xpath>
            <xpath expr="//filter[@name='created_by']" position="replace">
                <filter string='Created By' name='created_by' domain="[]" context="{'group_by': 'employee_id'}"/>
            </xpath>
        </field>
    </record>

    <record id="maintenance_request_view_form_inherit_hr" model="ir.ui.view">
        <field name="name">maintenance.request.view.form.inherit.hr</field>
        <field name="model">maintenance.request</field>
        <field name="inherit_id" ref="maintenance.hr_equipment_request_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='owner_user_id']" position="replace">
                <field name="employee_id" string="Created By"
                    widget="many2one_avatar_employee"
                    options="{'no_create_edit': True, 'no_open': True}"/>
            </xpath>
        </field>
    </record>

    <record id="maintenance_request_view_kanban_inherit_hr" model="ir.ui.view">
        <field name="name">maintenance.request.view.kanban.inherit.hr</field>
        <field name="model">maintenance.request</field>
        <field name="inherit_id" ref="maintenance.hr_equipment_request_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//span[@name='owner_user_id']" position="replace">
                <field name="employee_id"/>
            </xpath>
        </field>
    </record>

    <record id="maintenance_request_view_tree_inherit_hr" model="ir.ui.view">
        <field name="name">maintenance.request.view.list.inherit.hr</field>
        <field name="model">maintenance.request</field>
        <field name="inherit_id" ref="maintenance.hr_equipment_request_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='owner_user_id']" position="replace">
                <field name="employee_id"/>
            </xpath>
        </field>
    </record>

    <!-- equiment : views -->
    <record id="maintenance_equipment_view_search_inherit_hr" model='ir.ui.view'>
      <field name="name">maintenance.equipment.view.search.inherit.hr</field>
      <field name="model">maintenance.equipment</field>
      <field name="inherit_id" ref="maintenance.hr_equipment_view_search"/>
      <field name="arch" type="xml">
        <filter name="assigned" position="attributes">
          <attribute name="domain">['|', ('employee_id', '!=', False), ('department_id', '!=', False)]</attribute>
        </filter>
        <filter name="available" position="attributes">
          <attribute name="domain">[('employee_id', '=', False), ('department_id', '=', False)]</attribute>
        </filter>
        <field name="owner_user_id" position="after">
          <field name="employee_id"/>
          <field name="department_id"/>
        </field>
        <group position="inside">
          <filter string="Employee" name="employee" domain="[]" context="{'group_by': 'employee_id'}"/>
          <filter string="Department" name="department" domain="[]" context="{'group_by': 'department_id'}"/>
        </group>
      </field>
    </record>

    <record id="maintenance_equipment_view_form_inherit_hr" model="ir.ui.view">
        <field name="name">maintenance.equipment.view.form.inherit.hr</field>
        <field name="model">maintenance.equipment</field>
        <field name="inherit_id" ref="maintenance.hr_equipment_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='owner_user_id']" position="replace">
                <field name="equipment_assign_to" widget="radio"/>
                <field name="employee_id" string="Employee" invisible="equipment_assign_to == 'department' or not equipment_assign_to" widget="many2one_avatar_employee"/>
                <field name="department_id" string="Department" invisible="equipment_assign_to == 'employee' or not equipment_assign_to"/>
            </xpath>
        </field>
    </record>

    <record id="maintenance_equipment_view_kanban_inherit_hr" model="ir.ui.view">
        <field name="name">maintenance.equipment.view.kanban.inherit.hr</field>
        <field name="model">maintenance.equipment</field>
        <field name="inherit_id" ref="maintenance.hr_equipment_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='owner_user_id']" position='replace'>
                <field name="employee_id" widget="many2one_avatar_employee"/>
            </xpath>
            <xpath expr="//field[@name='serial_no']" position='after'>
                <div t-if="!record.employee_id.raw_value">Unassigned</div>
                <field name="employee_id"/>
                <field name="department_id"/>
            </xpath>
        </field>
    </record>

    <record id="maintenance_equipment_view_tree_inherit_hr" model="ir.ui.view">
        <field name="name">maintenance.equipment.view.list.inherit.hr</field>
        <field name="model">maintenance.equipment</field>
        <field name="inherit_id" ref="maintenance.hr_equipment_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='owner_user_id']" position="replace">
                <field name="employee_id" string="Employee"/>
                <field name="department_id" string="Department"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\hr_departure_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import Command, fields, models


class HrDepartureWizard(models.TransientModel):
    _inherit = 'hr.departure.wizard'

    unassign_equipment = fields.Boolean("Free Equiments", default=True, help="Unassign Employee from Equipments")

    def action_register_departure(self):
        super().action_register_departure()
        if self.unassign_equipment:
            self.employee_id.update({'equipment_ids': [Command.unlink(equipment.id) for equipment in self.employee_id.equipment_ids]})

```

## File: wizard\hr_departure_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_departure_wizard_view_form" model="ir.ui.view">
        <field name="name">hr.departure.wizard.view.form.extend</field>
        <field name="model">hr.departure.wizard</field>
        <field name="inherit_id" ref="hr.hr_departure_wizard_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@id='activities_label']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='activities']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='activities']" position="inside">
                <div><field name="unassign_equipment"/><label for="unassign_equipment" string="Equipment"/></div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_departure_wizard

```

