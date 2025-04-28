# Odoo Module: hr_fleet

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Fleet History',
    'version': '1.0',
    'category': 'Human Resources',
    'summary': 'Get history of driven cars by employees',
    'depends': ['hr', 'fleet'],
    'data': [
        'security/ir.model.access.csv',
        'security/hr_fleet_security.xml',
        'views/employee_views.xml',
        'views/fleet_vehicle_views.xml',
        'views/fleet_vehicle_cost_views.xml',
        'wizard/hr_departure_wizard_views.xml',
    ],
    'demo': [
        'data/hr_fleet_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'hr_fleet/static/src/views/**/*',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\hr_fleet_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fleet.vehicle_1" model="fleet.vehicle">
        <field name="driver_employee_id" ref="hr.employee_qdp"/>
    </record>
    <!-- Link addresses linked to cars to employees, ease migration from external to internal -->
    <record id="hr.employee_hne" model="hr.employee">
        <field name="address_home_id" ref="base.res_partner_address_25"/>
    </record>
    <record id="hr.employee_fpi" model="hr.employee">
        <field name="address_home_id" ref="base.res_partner_address_17"/>
    </record>
    <record id="hr.employee_jog" model="hr.employee">
        <field name="address_home_id" ref="base.res_partner_address_16"/>
    </record>
    <record id="hr.employee_jep" model="hr.employee">
        <field name="address_home_id" ref="base.res_partner_address_15"/>
    </record>

    <record id="fleet.vehicle_2" model="fleet.vehicle">
        <field name="driver_employee_id" ref="hr.employee_hne"/>
    </record>
    <record id="fleet.vehicle_3" model="fleet.vehicle">
        <field name="driver_employee_id" ref="hr.employee_fpi"/>
    </record>
    <record id="fleet.vehicle_4" model="fleet.vehicle">
        <field name="driver_employee_id" ref="hr.employee_jog"/>
    </record>
    <record id="fleet.vehicle_5" model="fleet.vehicle">
        <field name="driver_employee_id" ref="hr.employee_jep"/>
    </record>

    <!-- recompute driver_employee_id as we assigned a proper address_home_id to some employees -->
    <function model="fleet.vehicle.assignation.log" name="_compute_driver_employee_id">
        <value model="fleet.vehicle.assignation.log" eval="obj().search([('driver_employee_id', '=', False), ('driver_id', '!=', False)]).ids"/>
    </function>

</odoo>

```

## File: models\employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class Employee(models.Model):
    _inherit = 'hr.employee'

    employee_cars_count = fields.Integer(compute="_compute_employee_cars_count", string="Cars", groups="fleet.fleet_group_manager")
    car_ids = fields.One2many(
        'fleet.vehicle', 'driver_employee_id', string='Vehicles (private)',
        groups="fleet.fleet_group_manager,hr.group_hr_user",
    )
    mobility_card = fields.Char(groups="fleet.fleet_group_user")

    def action_open_employee_cars(self):
        self.ensure_one()

        return {
            "type": "ir.actions.act_window",
            "res_model": "fleet.vehicle.assignation.log",
            "views": [[self.env.ref("hr_fleet.fleet_vehicle_assignation_log_employee_view_list").id, "tree"], [False, "form"]],
            "domain": [("driver_employee_id", "in", self.ids)],
            "context": dict(self._context, default_driver_id=self.user_id.partner_id.id, default_driver_employee_id=self.id),
            "name": "History Employee Cars",
        }

    def _compute_employee_cars_count(self):
        rg = self.env['fleet.vehicle.assignation.log']._read_group([
            ('driver_employee_id', 'in', self.ids),
        ], ['driver_employee_id'], ['driver_employee_id'])
        cars_count = {r['driver_employee_id'][0]: r['driver_employee_id_count'] for r in rg}
        for employee in self:
            employee.employee_cars_count = cars_count.get(employee.id, 0)

    @api.constrains('address_home_id')
    def _check_address_home_id(self):
        no_address = self.filtered(lambda r: not r.address_home_id)
        car_ids = self.env['fleet.vehicle'].sudo().search([
            ('driver_employee_id', 'in', no_address.ids),
        ])
        # Prevent from removing employee address when linked to a car
        if car_ids:
            raise ValidationError(_('Cannot remove address from employees with linked cars.'))


    def write(self, vals):
        res = super().write(vals)
        #Update car partner when it is changed on the employee
        if 'address_home_id' in vals:
            car_ids = self.env['fleet.vehicle'].sudo().search([
                ('driver_employee_id', 'in', self.ids),
                ('driver_id', 'in', self.mapped('address_home_id').ids),
            ])
            if car_ids:
                car_ids.write({'driver_id': vals['address_home_id']})
        if 'mobility_card' in vals:
            #NOTE: keeping it as a search on driver_id but we might be able to use driver_employee_id in the future
            vehicles = self.env['fleet.vehicle'].search([('driver_id', 'in', (self.user_id.partner_id | self.sudo().address_home_id).ids)])
            vehicles._compute_mobility_card()
        return res

class EmployeePublic(models.Model):
    _inherit = 'hr.employee.public'

    mobility_card = fields.Char(readonly=True)

```

## File: models\fleet_vehicle.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

class FleetVehicle(models.Model):
    _inherit = 'fleet.vehicle'

    mobility_card = fields.Char(compute='_compute_mobility_card', store=True)
    driver_employee_id = fields.Many2one(
        'hr.employee', 'Driver (Employee)',
        compute='_compute_driver_employee_id', store=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        tracking=True,
    )
    driver_employee_name = fields.Char(related="driver_employee_id.name")
    future_driver_employee_id = fields.Many2one(
        'hr.employee', 'Future Driver (Employee)',
        compute='_compute_future_driver_employee_id', store=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        tracking=True,
    )

    @api.depends('driver_id')
    def _compute_driver_employee_id(self):
        for vehicle in self:
            if vehicle.driver_id:
                vehicle.driver_employee_id = self.env['hr.employee'].search([
                    ('address_home_id', '=', vehicle.driver_id.id),
                ], limit=1)
            else:
                vehicle.driver_employee_id = False

    @api.depends('future_driver_id')
    def _compute_future_driver_employee_id(self):
        for vehicle in self:
            if vehicle.future_driver_id:
                vehicle.future_driver_employee_id = self.env['hr.employee'].search([
                    ('address_home_id', '=', vehicle.future_driver_id.id),
                ], limit=1)
            else:
                vehicle.future_driver_employee_id = False

    @api.depends('driver_id')
    def _compute_mobility_card(self):
        for vehicle in self:
            employee = self.env['hr.employee']
            if vehicle.driver_id:
                employee = employee.search([('address_home_id', '=', vehicle.driver_id.id)], limit=1)
                if not employee:
                    employee = employee.search([('user_id.partner_id', '=', vehicle.driver_id.id)], limit=1)
            vehicle.mobility_card = employee.mobility_card

    def _update_create_write_vals(self, vals):
        if 'driver_employee_id' in vals:
            partner = False
            if vals['driver_employee_id']:
                employee = self.env['hr.employee'].sudo().browse(vals['driver_employee_id'])
                partner = employee.address_home_id.id
            vals['driver_id'] = partner
        elif 'driver_id' in vals:
            # Reverse the process if we can find a single employee
            employee = False
            if vals['driver_id']:
                # Limit to 2, we only care about the first one if he is the only one
                employee_ids = self.env['hr.employee'].sudo().search([
                    ('address_home_id', '=', vals['driver_id'])
                ], limit=2)
                if len(employee_ids) == 1:
                    employee = employee_ids[0].id
            vals['driver_employee_id'] = employee

        # Same for future driver
        if 'future_driver_employee_id' in vals:
            partner = False
            if vals['future_driver_employee_id']:
                employee = self.env['hr.employee'].sudo().browse(vals['future_driver_employee_id'])
                partner = employee.address_home_id.id
            vals['future_driver_id'] = partner
        elif 'future_driver_id' in vals:
            # Reverse the process if we can find a single employee
            employee = False
            if vals['future_driver_id']:
                # Limit to 2, we only care about the first one if he is the only one
                employee_ids = self.env['hr.employee'].sudo().search([
                    ('address_home_id', '=', vals['future_driver_id'])
                ], limit=2)
                if len(employee_ids) == 1:
                    employee = employee_ids[0].id
            vals['future_driver_employee_id'] = employee

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            self._update_create_write_vals(vals)
        return super().create(vals_list)

    def write(self, vals):
        self._update_create_write_vals(vals)
        if 'driver_employee_id' in vals:
            for vehicle in self:
                if vehicle.driver_employee_id and vehicle.driver_employee_id.id != vals['driver_employee_id']:
                    partners_to_unsubscribe = vehicle.driver_id.ids
                    employee = vehicle.driver_employee_id
                    if employee and employee.user_id.partner_id:
                        partners_to_unsubscribe.append(employee.user_id.partner_id.id)
                    vehicle.message_unsubscribe(partner_ids=partners_to_unsubscribe)
        return super().write(vals)

    def action_open_employee(self):
        self.ensure_one()
        return {
            'name': _('Related Employee'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.employee',
            'view_mode': 'form',
            'res_id': self.driver_employee_id.id,
        }

    def open_assignation_logs(self):
        action = super().open_assignation_logs()
        action['views'] = [[self.env.ref('hr_fleet.fleet_vehicle_assignation_log_view_list').id, 'tree']]
        return action

```

## File: models\fleet_vehicle_assignation_log.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class FleetVehicleAssignationLog(models.Model):
    _inherit = 'fleet.vehicle.assignation.log'

    driver_employee_id = fields.Many2one('hr.employee', string='Driver (Employee)', compute='_compute_driver_employee_id', store=True, readonly=False)
    attachment_number = fields.Integer('Number of Attachments', compute='_compute_attachment_number')

    @api.depends('driver_id')
    def _compute_driver_employee_id(self):
        employees = self.env['hr.employee'].search([('address_home_id', 'in', self.driver_id.ids)])

        for log in self:
            employee = employees.filtered(lambda e: e.address_home_id.id == log.driver_id.id)
            log.driver_employee_id = employee and employee[0] or False

    def _compute_attachment_number(self):
        attachment_data = self.env['ir.attachment']._read_group([
            ('res_model', '=', 'fleet.vehicle.assignation.log'),
            ('res_id', 'in', self.ids)], ['res_id'], ['res_id'])
        attachment = dict((data['res_id'], data['res_id_count']) for data in attachment_data)
        for doc in self:
            doc.attachment_number = attachment.get(doc.id, 0)

    def action_get_attachment_view(self):
        self.ensure_one()
        res = self.env['ir.actions.act_window']._for_xml_id('base.action_attachment')
        res['views'] = [[self.env.ref('hr_fleet.view_attachment_kanban_inherit_hr').id, 'kanban']]
        res['domain'] = [('res_model', '=', 'fleet.vehicle.assignation.log'), ('res_id', 'in', self.ids)]
        res['context'] = {'default_res_model': 'fleet.vehicle.assignation.log', 'default_res_id': self.id}
        return res

```

## File: models\fleet_vehicle_log_contract.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class FleetVehicleLogContract(models.Model):
    _inherit = 'fleet.vehicle.log.contract'

    purchaser_employee_id = fields.Many2one(
        related='vehicle_id.driver_employee_id',
        string='Driver (Employee)',
    )

```

## File: models\fleet_vehicle_log_services.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

class FleetVehicleLogServices(models.Model):
    _inherit = 'fleet.vehicle.log.services'

    purchaser_employee_id = fields.Many2one(
        'hr.employee', string="Driver (Employee)",
        compute='_compute_purchaser_employee_id', readonly=False, store=True,
    )

    @api.depends('vehicle_id', 'purchaser_employee_id')
    def _compute_purchaser_id(self):
        internals = self.filtered(lambda r: r.purchaser_employee_id)
        super(FleetVehicleLogServices, (self - internals))._compute_purchaser_id()
        for service in internals:
            service.purchaser_id = service.purchaser_employee_id.address_home_id

    @api.depends('vehicle_id')
    def _compute_purchaser_employee_id(self):
        for service in self:
            service.purchaser_employee_id = service.vehicle_id.driver_employee_id

```

## File: models\fleet_vehicle_odometer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class FleetVehicleOdometer(models.Model):
    _inherit = 'fleet.vehicle.odometer'

    driver_employee_id = fields.Many2one(
        related='vehicle_id.driver_employee_id', string='Driver (Employee)',
        readonly=True,
    )

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class User(models.Model):
    _inherit = ['res.users']

    employee_cars_count = fields.Integer(related='employee_id.employee_cars_count')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['employee_cars_count']

    def action_open_employee_cars(self):
        return self.employee_id.action_open_employee_cars()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import employee
from . import res_users
from . import fleet_vehicle_assignation_log
from . import fleet_vehicle
from . import fleet_vehicle_log_contract
from . import fleet_vehicle_log_services
from . import fleet_vehicle_odometer

```

## File: security\hr_fleet_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="hr_fleet_rule_vehicle_visibility_hr_officier" model="ir.rule">
            <field name="name">Hr Officer read rights on vehicle with employees assigned</field>
            <field name="model_id" ref="model_fleet_vehicle"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
            <field name="groups" eval="[(4, ref('hr.group_hr_user'))]"/>
            <field name="domain_force">['|', ('driver_employee_id', '!=', False), ('future_driver_employee_id', '!=', False)]</field>
        </record>

        <record id="fleet.fleet_rule_contract_visibility_user" model="ir.rule">
            <field name="domain_force">['|', ('vehicle_id.driver_id','=',user.partner_id.id),
                ('vehicle_id.driver_employee_id.user_id','=',user.id)]</field>
        </record>
        <record id="fleet.fleet_rule_service_visibility_user" model="ir.rule">
            <field name="domain_force">['|', ('vehicle_id.driver_id','=',user.partner_id.id),
                ('vehicle_id.driver_employee_id.user_id','=',user.id)]</field>
        </record>
        <record id="fleet.fleet_rule_odometer_visibility_user" model="ir.rule">
            <field name="domain_force">['|', ('vehicle_id.driver_id','=',user.partner_id.id),
                ('vehicle_id.driver_employee_id.user_id','=',user.id)]</field>
        </record>
        <record id="fleet.fleet_rule_vehicle_visibility_user" model="ir.rule">
            <field name="domain_force">['|', ('driver_id','=',user.partner_id.id),
                ('driver_employee_id.user_id','=',user.id)]</field>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
hr_fleet_vehicle_access_right_hr_officer,hr_fleet_vehicle_access_right_hr_officer,model_fleet_vehicle,hr.group_hr_user,1,0,0,0

```

## File: static\src\views\hr_fleet_kanban\hr_fleet_kanban_controller.js

```javascript
/** @odoo-module **/

import { KanbanController } from "@web/views/kanban/kanban_controller";
import { useBus, useService } from "@web/core/utils/hooks";

const { useRef } = owl;

export class HrFleetKanbanController extends KanbanController {
    setup() {
        super.setup(...arguments);
        this.uploadFileInput = useRef("uploadFileInput");
        this.uploadService = useService("file_upload");
        useBus(
            this.uploadService.bus,
            "FILE_UPLOAD_LOADED",
            () => {
                this.model.load();
            },
        );
    }

    async onInputChange(ev) {
        if (!ev.target.files) {
            return;
        }
        this.uploadService.upload(
            "/web/binary/upload_attachment",
            ev.target.files,
            {
                buildFormData: (formData) => {
                    formData.append("model", "fleet.vehicle.assignation.log");
                    formData.append("id", this.props.context.active_id);
                },
            },
        );
        ev.target.value = "";
    }
}

```

## File: static\src\views\hr_fleet_kanban\hr_fleet_kanban_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="hr_fleet.HrFleetKanbanController.Buttons" t-inherit="web.KanbanView.Buttons" t-inherit-mode="primary" owl="1">
        <xpath expr="//button[hasclass('o-kanban-button-new')]" position="replace">
            <input type="file" multiple="true" t-ref="uploadFileInput" class="o_input_file o_hidden" t-on-change.stop="onInputChange"/>
            <button type="button" t-att-class="'d-none d-md-block btn' + (!env.isSmall ? ' btn-primary' : 'btn-secondary')" t-on-click="() => this.uploadFileInput.el.click()">
                Upload
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\hr_fleet_kanban\hr_fleet_kanban_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { kanbanView } from "@web/views/kanban/kanban_view";
import { HrFleetKanbanController } from "@hr_fleet/views/hr_fleet_kanban/hr_fleet_kanban_controller";

export const hrFleetKanbanView = {
    ...kanbanView,
    Controller: HrFleetKanbanController,
    buttonTemplate: "hr_fleet.HrFleetKanbanController.Buttons",
};
registry.category("views").add("hr_fleet_kanban_view", hrFleetKanbanView);

```

## File: views\employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Employee -->
    <record id="view_employee_form" model="ir.ui.view">
        <field name="name">hr.employee.form.inherit.hr.fleet</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form" />
        <field name="priority" eval="60" />
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button name="action_open_employee_cars" type="object"
                        class="oe_stat_button" icon="fa-car" groups="fleet.fleet_group_manager"
                        attrs="{'invisible': [('employee_cars_count', '=', 0)]}">
                    <field name="employee_cars_count" widget="statinfo" />
                </button>
            </div>
            <group name="application_group" position="attributes">
                <attribute name="invisible">0</attribute>
            </group>
            <group name="application_group" position="inside">
                <field name="mobility_card" string="Fleet Mobility Card"/>
            </group>
        </field>
    </record>

    <record id="view_employee_filter" model="ir.ui.view">
        <field name="name">hr.employee.filter.inherit.hr.fleet</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='job_id']" position="after">
                <field name="car_ids" string="License Plate"
                    filter_domain="[('car_ids.license_plate', 'ilike', self)]"/>
            </xpath>
        </field>
    </record>

    <record id="res_users_view_form_preferences" model="ir.ui.view">
        <field name="name">hr.user.preferences.form.inherit.hr.fleet</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile" />
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_open_employee_cars" type="object"
                        class="oe_stat_button" icon="fa-car" groups="fleet.fleet_group_manager"
                        attrs="{'invisible': [('employee_cars_count','=', 0)]}">
                    <field name="employee_cars_count" widget="statinfo" />
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\fleet_vehicle_cost_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fleet_vehicle_log_contract_view_form_inherit_hr" model="ir.ui.view">
        <field name="name">fleet.vehicle.log.contract.form.inherit.hr</field>
        <field name="model">fleet.vehicle.log.contract</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_log_contract_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='purchaser_id']" position="after">
                <field name="purchaser_employee_id" string="Driver" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="fleet_vehicle_log_contract_view_tree_inherit_hr" model="ir.ui.view">
        <field name="name">fleet.vehicle.log.contract.tree.inherit.hr</field>
        <field name="model">fleet.vehicle.log.contract</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_log_contract_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='purchaser_id']" position="after">
                <field name="purchaser_employee_id" optional="hide" widget="many2one_avatar_employee"/>
            </xpath>
        </field>
    </record>

    <record id="fleet_vehicle_log_contract_view_search_inherit_hr" model="ir.ui.view">
        <field name="name">fleet.vehicle.log.contract.search.inherit.hr</field>
        <field name="model">fleet.vehicle.log.contract</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_log_contract_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='purchaser_id']" position="after">
                <field name="purchaser_employee_id" string="Employee"
                    filter_domain="[('purchaser_employee_id','child_of', self)]"/>
            </xpath>
        </field>
    </record>

    <record id="fleet_vehicle_log_services_view_form_inherit_hr" model="ir.ui.view">
        <field name="name">fleet.vehicle.log.contract.form.inherit.hr</field>
        <field name="model">fleet.vehicle.log.services</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_log_services_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='purchaser_id']" position="after">
                <field name="purchaser_employee_id" string="Driver (Employee)" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="fleet_vehicle_log_services_view_tree_inherit_hr" model="ir.ui.view">
        <field name="name">fleet.vehicle.log.services.tree.inherit.hr</field>
        <field name="model">fleet.vehicle.log.services</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_log_services_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='purchaser_id']" position="after">
                <field name="purchaser_employee_id" readonly="1" widget="many2one_avatar" optional="hide"/>
            </xpath>
        </field>
    </record>

    <record id="fleet_vehicle_log_services_view_kanban_inherit_hr" model="ir.ui.view">
        <field name="name">fleet.vehicle.log.services.kanban.inherit.hr</field>
        <field name="model">fleet.vehicle.log.services</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_log_services_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div/field[@name='purchaser_id']" position="after">
                <field name="purchaser_employee_id"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\fleet_vehicle_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fleet_vehicle_odometer_view_tree" model="ir.ui.view">
        <field name="name">fleet.vehicle.odometer.view.tree.inherit.hr.fleet</field>
        <field name="model">fleet.vehicle.odometer</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_odometer_view_tree" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='driver_id']" position="after">
                <field name="driver_employee_id" widget="many2one_avatar" optional="hide"/>
            </xpath>
        </field>
    </record>

    <!-- main view for fleet-->
    <record id="fleet_vehicle_assignation_log_view_list" model="ir.ui.view">
        <field name="name">fleet.vehicle.assignation.log.view.tree.inherit.hr.fleet</field>
        <field name="model">fleet.vehicle.assignation.log</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_assignation_log_view_list" />
        <field name="arch" type="xml">
            <field name="vehicle_id" position="attributes">
                <attribute name="optional">hide</attribute>
            </field>
            <field name="driver_id" position="after">
                <field name="driver_employee_id" widget="many2one_avatar"/>
            </field>
            <field name="date_end" position="after">
                <field name="attachment_number" optional="show" />
                <button name="action_get_attachment_view" string="Attachments" type="object" icon="fa-paperclip"/>
            </field>
        </field>
    </record>

    <!-- for employee cars -->
    <record id="fleet_vehicle_assignation_log_employee_view_list" model="ir.ui.view">
        <field name="name">fleet.vehicle.assignation.log.view.tree.inherit.hr.fleet</field>
        <field name="model">fleet.vehicle.assignation.log</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_assignation_log_view_list" />
        <field name="arch" type="xml">
            <field name="driver_id" position="replace" />
            <field name="date_end" position="after">
                <field name="driver_id" string="Current Driver" optional="hide"/>
                <field name="attachment_number" optional="show" />
                <button name="action_get_attachment_view" string="Attachments" type="object" icon="fa-paperclip" />
            </field>
        </field>
    </record>

    <record id="fleet_vehicle_view_form_inherit_hr" model="ir.ui.view">
        <field name="name">fleet.vehicle.form.inherit.hr</field>
        <field name="model">fleet.vehicle</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='driver_id']" position="after">
                <field name="driver_employee_id" invisible="1"/>
                <field name="mobility_card" readonly="1"/>
            </xpath>
            <xpath expr="//field[@name='future_driver_id']" position="after">
                <field name="future_driver_employee_id" invisible="1"/>
            </xpath>
            <button name="open_assignation_logs" position="before">
                <button name="action_open_employee" type="object" class="oe_stat_button" icon="fa-id-card-o" groups="hr.group_hr_user" attrs="{'invisible': [('driver_employee_id', '=', False)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">1</span>
                        <span class="o_stat_text">Employee</span>
                    </div>
                </button>
            </button>
        </field>
    </record>

    <record id="fleet_vehicle_view_search_inherit_hr" model="ir.ui.view">
        <field name="name">fleet.vehicle.search.inherit.hr</field>
        <field name="model">fleet.vehicle</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='license_plate']" position="after">
                <field name="mobility_card"/>
            </xpath>
            <xpath expr="//field[@name='driver_id']" position="after">
                <field name="driver_employee_id" string="Current Driver (Employee)"/>
            </xpath>
            <xpath expr="//field[@name='log_drivers']" position="attributes">
                <attribute name="filter_domain">[
                    '|', '|', '|', '|',
                    ('log_drivers.driver_id.name', 'ilike', self),
                    ('driver_id.name', 'ilike', self),
                    ('future_driver_id.name', 'ilike', self),
                    ('driver_employee_id.name', 'ilike', self),
                    ('future_driver_employee_id.name', 'ilike', self),
                ]</attribute>
            </xpath>
        </field>
    </record>

    <record id="fleet_vehicle_view_tree_inherit_hr" model="ir.ui.view">
        <field name="model">fleet.vehicle</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_view_tree"/>
        <field name="arch" type="xml">
            <field name="driver_id" position="after">
                <field name="driver_employee_id" optional="hide"/>
            </field>
            <field name="future_driver_id" position="after">
                <field name="future_driver_employee_id" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="view_attachment_kanban_inherit_hr" model="ir.ui.view">
       <field name="name">ir.attachment.kanban.inherit.hr</field>
       <field name="model">ir.attachment</field>
       <field name="inherit_id" ref="mail.view_document_file_kanban"/>
       <field name="mode">primary</field>
       <field name="arch" type="xml">
           <xpath expr="//kanban" position="attributes">
                <attribute name="js_class">hr_fleet_kanban_view</attribute>
            </xpath>
       </field>
   </record>
</odoo>

```

## File: wizard\hr_departure_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class HrDepartureWizard(models.TransientModel):
    _inherit = 'hr.departure.wizard'

    release_campany_car = fields.Boolean("Release Company Car", default=lambda self: self.env.user.user_has_groups('fleet.fleet_group_user'))

    def action_register_departure(self):
        super(HrDepartureWizard, self).action_register_departure()
        if self.release_campany_car:
            self._free_campany_car()

    def _free_campany_car(self):
        """Find all fleet.vehichle.assignation.log records that link to the employee, if there is no 
        end date or end date > departure date, update the date. Also check fleet.vehicle to see if 
        there is any record with its dirver_id to be the employee, set them to False."""
        drivers = self.employee_id.user_id.partner_id | self.employee_id.sudo().address_home_id
        assignations = self.env['fleet.vehicle.assignation.log'].search([('driver_id', 'in', drivers.ids)])
        for assignation in assignations:
            if self.departure_date and (not assignation.date_end or assignation.date_end > self.departure_date):
                assignation.write({'date_end': self.departure_date})
        cars = self.env['fleet.vehicle'].search([('driver_id', 'in', drivers.ids)])
        cars.write({'driver_id': False, 'driver_employee_id': False})

```

## File: wizard\hr_departure_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_departure_wizard_view_form" model="ir.ui.view">
        <field name="name">hr.departure.wizard.view.form.extend2</field>
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
                <div><field name="release_campany_car"/><label for="release_campany_car" string="Company Car"/></div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_departure_wizard

```

