# Odoo Module: hr_holidays_attendance

Category: Human Resources

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
    'name': "HR Attendance Holidays",
    'summary': """Attendance Holidays""",
    'category': 'Human Resources',
    'description': """
Convert employee's extra hours to leave allocations.
    """,
    'version': '1.0',
    'depends': ['hr_attendance', 'hr_holidays'],
    'auto_install': True,
    'data': [
        'security/hr_holidays_attendance_security.xml',
        'views/hr_leave_allocation_views.xml',
        'views/hr_leave_type_views.xml',
        'views/hr_leave_views.xml',
        'views/hr_employee_views.xml',
        'views/res_users_views.xml',
        'data/hr_holidays_attendance_data.xml',
    ],
    'demo': [
        'data/hr_holidays_attendance_demo.xml',
    ],
    'assets': {
        'web.assets_qweb': [
            'hr_holidays_attendance/static/src/xml/time_off_calendar.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\hr_holidays_attendance_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="holiday_status_extra_hours" model="hr.leave.type">
            <field name="name">Extra Hours</field>
            <field name="request_unit">hour</field>
            <field name="requires_allocation">no</field>
            <field name="leave_validation_type">manager</field>
            <field name="overtime_deductible" eval="True"/>
            <field name="active" eval="False"/>
            <field name="company_id" eval="False"/>
            <field name="icon_id" ref="hr_holidays.icon_9"/>
        </record>

        <!-- The record above should be archived if no company has overtime counting enabled, otherwise enabled -->
        <function model="res.company" name="_check_extra_hours_time_off"/>
    </data>
</odoo>

```

## File: data\hr_holidays_attendance_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.main_company" model="res.company">
            <field name="hr_attendance_overtime" eval="True" />
            <field name="overtime_start_date" eval="(DateTime.today() + relativedelta(months=-1)).date()" />
        </record>
    </data>
</odoo>

```

## File: models\hr_attendance.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv.expression import AND

class HrAttendance(models.Model):
    _inherit = "hr.attendance"

    def _get_overtime_leave_domain(self):
        domain = super()._get_overtime_leave_domain()
        # resource_id = False => Public holidays
        return AND([domain, ['|', ('holiday_id.holiday_status_id.time_type', '=', 'leave'), ('resource_id', '=', False)]])

```

## File: models\hr_leave.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import timedelta

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.tools import float_round


class HRLeave(models.Model):
    _inherit = 'hr.leave'

    overtime_id = fields.Many2one('hr.attendance.overtime', string='Extra Hours', groups='hr_holidays.group_hr_holidays_user')
    employee_overtime = fields.Float(related='employee_id.total_overtime')
    overtime_deductible = fields.Boolean(compute='_compute_overtime_deductible')

    @api.depends('holiday_status_id')
    def _compute_overtime_deductible(self):
        for leave in self:
            leave.overtime_deductible = leave.holiday_status_id.overtime_deductible and leave.holiday_status_id.requires_allocation == 'no'

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        self._check_overtime_deductible(res)
        return res

    def write(self, vals):
        res = super().write(vals)
        fields_to_check = {'number_of_days', 'date_from', 'date_to', 'state', 'employee_id', 'holiday_status_id'}
        if not any(field for field in fields_to_check if field in vals):
            return res
        if vals.get('holiday_status_id'):
            self._check_overtime_deductible(self)
        #User may not have access to overtime_id field
        for leave in self.sudo().filtered('overtime_id'):
            # It must always be possible to refuse leave based on overtime
            if vals.get('state') in ['refuse']:
                continue
            employee = leave.employee_id
            duration = leave.number_of_hours_display
            overtime_duration = leave.overtime_id.sudo().duration
            if overtime_duration != -1 * duration:
                if duration > employee.total_overtime - overtime_duration:
                    raise ValidationError(_('The employee does not have enough extra hours to extend this leave.'))
                leave.overtime_id.sudo().duration = -1 * duration
        return res

    def _check_overtime_deductible(self, leaves):
        # If the type of leave is overtime deductible, we have to check that the employee has enough extra hours
        for leave in leaves:
            if not leave.overtime_deductible:
                continue
            employee = leave.employee_id.sudo()
            duration = leave.number_of_hours_display
            if duration > employee.total_overtime:
                if employee.user_id == self.env.user:
                    raise ValidationError(_('You do not have enough extra hours to request this leave'))
                raise ValidationError(_('The employee does not have enough extra hours to request this leave.'))
            if not leave.overtime_id:
                leave.sudo().overtime_id = self.env['hr.attendance.overtime'].sudo().create({
                    'employee_id': employee.id,
                    'date': leave.date_from,
                    'adjustment': True,
                    'duration': -1 * duration,
                })

    def action_draft(self):
        overtime_leaves = self.filtered('overtime_deductible')
        if any([l.employee_overtime < float_round(l.number_of_hours_display, 2) for l in overtime_leaves]):
            if self.employee_id.user_id.id == self.env.user.id:
                raise ValidationError(_('You do not have enough extra hours to request this leave'))
            raise ValidationError(_('The employee does not have enough extra hours to request this leave.'))

        res = super().action_draft()
        overtime_leaves.overtime_id.sudo().unlink()
        for leave in overtime_leaves:
            overtime = self.env['hr.attendance.overtime'].sudo().create({
                'employee_id': leave.employee_id.id,
                'date': leave.date_from,
                'adjustment': True,
                'duration': -1 * leave.number_of_hours_display
            })
            leave.sudo().overtime_id = overtime.id
        return res

    def action_refuse(self):
        res = super().action_refuse()
        self.sudo().overtime_id.unlink()
        return res

    def _validate_leave_request(self):
        super()._validate_leave_request()
        self._update_leaves_overtime()

    def _remove_resource_leave(self):
        res = super()._remove_resource_leave()
        self._update_leaves_overtime()
        return res

    def _update_leaves_overtime(self):
        employee_dates = defaultdict(set)
        for leave in self:
            if leave.employee_id and leave.employee_company_id.hr_attendance_overtime:
                for d in range((leave.date_to - leave.date_from).days + 1):
                    employee_dates[leave.employee_id].add(self.env['hr.attendance']._get_day_start_and_day(leave.employee_id, leave.date_from + timedelta(days=d)))
        if employee_dates:
            self.env['hr.attendance']._update_overtime(employee_dates)

    def unlink(self):
        # TODO master change to ondelete
        self.sudo().overtime_id.unlink()
        return super().unlink()

```

## File: models\hr_leave_allocation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.tools import float_round
from odoo.osv import expression


class HolidaysAllocation(models.Model):
    _inherit = 'hr.leave.allocation'

    def default_get(self, fields):
        res = super().default_get(fields)
        if 'holiday_status_id' in fields and self.env.context.get('deduct_extra_hours'):
            domain = [('overtime_deductible', '=', True), ('requires_allocation', '=', 'yes')]
            if self.env.context.get('deduct_extra_hours_employee_request', False):
                # Prevent loading manager allocated time off type in self request contexts
                domain = expression.AND([domain, [('employee_requests', '=', 'yes')]])
            leave_type = self.env['hr.leave.type'].search(domain, limit=1)
            res['holiday_status_id'] = leave_type.id
        return res

    overtime_deductible = fields.Boolean(compute='_compute_overtime_deductible')
    overtime_id = fields.Many2one('hr.attendance.overtime', string='Extra Hours', groups='hr_holidays.group_hr_holidays_user')
    employee_overtime = fields.Float(related='employee_id.total_overtime')
    hr_attendance_overtime = fields.Boolean(related='employee_company_id.hr_attendance_overtime')

    @api.depends('holiday_status_id')
    def _compute_overtime_deductible(self):
        for allocation in self:
            allocation.overtime_deductible = allocation.hr_attendance_overtime and allocation.holiday_status_id.overtime_deductible

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        for allocation in res:
            if allocation.overtime_deductible and allocation.holiday_type == 'employee':
                duration = allocation.number_of_hours_display
                if duration > allocation.employee_id.total_overtime:
                    raise ValidationError(_('The employee does not have enough overtime hours to request this leave.'))
                if not allocation.overtime_id:
                    allocation.sudo().overtime_id = self.env['hr.attendance.overtime'].sudo().create({
                        'employee_id': allocation.employee_id.id,
                        'date': allocation.date_from,
                        'adjustment': True,
                        'duration': -1 * duration,
                    })
        return res

    def write(self, vals):
        res = super().write(vals)
        if 'number_of_days' not in vals:
            return res
        for allocation in self.filtered('overtime_id'):
            employee = allocation.employee_id
            duration = allocation.number_of_hours_display
            overtime_duration = allocation.overtime_id.sudo().duration
            if overtime_duration != -1 * duration:
                if duration > employee.total_overtime - overtime_duration:
                    raise ValidationError(_('The employee does not have enough extra hours to extend this allocation.'))
                allocation.overtime_id.sudo().duration = -1 * duration
        return res

    def action_draft(self):
        overtime_allocations = self.filtered('overtime_deductible')
        if any([a.employee_overtime < float_round(a.number_of_hours_display, 2) for a in overtime_allocations]):
            raise ValidationError(_('The employee does not have enough extra hours to request this allocation.'))
        res = super().action_draft()

        overtime_allocations.overtime_id.sudo().unlink()
        for allocation in overtime_allocations:
            overtime = self.env['hr.attendance.overtime'].sudo().create({
                'employee_id': allocation.employee_id.id,
                'date': allocation.date_from,
                'adjustment': True,
                'duration': -1 * allocation.number_of_hours_display
            })
            allocation.sudo().overtime_id = overtime.id
        return res

    def action_refuse(self):
        res = super().action_refuse()
        self.overtime_id.sudo().unlink()
        return res

```

## File: models\hr_leave_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class HRLeaveType(models.Model):
    _inherit = 'hr.leave.type'

    hr_attendance_overtime = fields.Boolean(compute='_compute_hr_attendance_overtime')
    overtime_deductible = fields.Boolean(
        "Deduct Extra Hours", default=False,
        help="Once a time off of this type is approved, extra hours in attendances will be deducted.")

    def get_employees_days(self, employee_ids, date=None):
        res = super().get_employees_days(employee_ids, date)
        deductible_time_off_type_ids = self.env['hr.leave.type'].search([
            ('overtime_deductible', '=', True),
            ('requires_allocation', '=', 'no')]).ids
        for employee_id, allocations in res.items():
            for allocation_id in allocations:
                if allocation_id in deductible_time_off_type_ids:
                    res[employee_id][allocation_id]['virtual_remaining_leaves'] = self.env['hr.employee'].sudo().browse(employee_id).total_overtime
                    res[employee_id][allocation_id]['overtime_deductible'] = True
                else:
                    res[employee_id][allocation_id]['overtime_deductible'] = False
        return res

    def _get_days_request(self):
        res = super()._get_days_request()
        res[1]['overtime_deductible'] = self.overtime_deductible
        return res

    @api.depends('company_id.hr_attendance_overtime')
    def _compute_hr_attendance_overtime(self):
        # If no company is linked to the time off type, use the current company's setting
        for leave_type in self:
            if leave_type.company_id:
                leave_type.hr_attendance_overtime = leave_type.company_id.hr_attendance_overtime
            else:
                leave_type.hr_attendance_overtime = self.env.company.hr_attendance_overtime

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    @api.model
    def _check_extra_hours_time_off(self):
        extra_hours_time_off_type = self.env.ref('hr_holidays_attendance.holiday_status_extra_hours', raise_if_not_found=False)
        if not extra_hours_time_off_type:
            return
        all_companies = self.env['res.company'].sudo().search([])
        # Unarchive time of type if the feature is enabled
        if any(company.hr_attendance_overtime and not extra_hours_time_off_type.active for company in all_companies):
            extra_hours_time_off_type.toggle_active()
        # Archive time of type if the feature is disabled for all the company
        if all(not company.hr_attendance_overtime and extra_hours_time_off_type.active for company in all_companies):
            extra_hours_time_off_type.toggle_active()

    def write(self, vals):
        res = super().write(vals)
        if 'hr_attendance_overtime' in vals:
            self._check_extra_hours_time_off()
        return res

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResUsers(models.Model):
    _inherit = 'res.users'

    request_overtime = fields.Boolean(compute='_compute_request_overtime')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['request_overtime']

    @api.depends_context('uid')
    @api.depends('total_overtime')
    def _compute_request_overtime(self):
        is_holiday_user = self.env.user.has_group('hr_holidays.group_hr_holidays_user')
        time_off_types = self.env['hr.leave.type'].search_count([
            ('requires_allocation', '=', 'yes'),
            ('employee_requests', '=', 'yes'),
            ('overtime_deductible', '=', True)
        ])
        for user in self:
            if user.total_overtime >= 1:
                if is_holiday_user:
                    user.request_overtime = True
                else:
                    user.request_overtime = time_off_types
            else:
                user.request_overtime = False

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_attendance
from . import hr_leave_allocation
from . import hr_leave_type
from . import hr_leave
from . import res_company
from . import res_users

```

## File: security\hr_holidays_attendance_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_holidays.group_hr_holidays_user" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('hr_attendance.group_hr_attendance_user'))]"/>
    </record>
</odoo>

```

## File: static\src\xml\time_off_calendar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template" xml:space="preserve">
    <t t-inherit="hr_holidays.dashboard_calendar_header" t-inherit-mode="extension">
        <!--inject a first condition-->
        <xpath expr="//t[@t-if='requires_allocation']" position="before">
            <t t-if="timeoff[1]['overtime_deductible'] == true &amp;&amp; !requires_allocation">
                <t t-if="has_icon">
                    <img height="30px" t-attf-src="{{timeoff[1]['icon']}}"/>
                </t>
                <span t-esc="timeoff[1]['usable_remaining_leaves']" class="o_timeoff_huge o_timeoff_purple font-weight-bold align-middle"/><br/>
                <span class="o_timeoff_purple">HOURS AVAILABLE</span>
            </t>
        </xpath>
        <!--change t-if to t-elif-->
        <xpath expr="//t[@t-if='requires_allocation']" position="attributes">
            <attribute name="t-if"/>
            <attribute name="t-elif">requires_allocation</attribute>
        </xpath>
    </t>
    <t t-inherit="hr_holidays.dashboard_calendar_header_mobile" t-inherit-mode="extension">
        <!--inject a first condition-->
        <xpath expr="//t[@t-if='requires_allocation']" position="before">
            <t t-if="timeoff[1]['overtime_deductible'] == true &amp;&amp; !requires_allocation">
                <strong t-esc="timeoff[1]['usable_remaining_leaves']" class="o_timeoff_green"/> Hours <span class="o_timeoff_green ml-2">Available </span>
            </t>
        </xpath>
        <!--change t-if to t-elif-->
        <xpath expr="//t[@t-if='requires_allocation']" position="attributes">
            <attribute name="t-if"/>
            <attribute name="t-elif">requires_allocation</attribute>
        </xpath>
    </t>
</templates>

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="hr_user_view_form" model="ir.ui.view">
        <field name="name">hr.user.preferences.view.form.attendance.inherit</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr_attendance.hr_user_view_form"/>
        <field name="arch" type="xml">
            <!-- Hide Attendance button -->
            <xpath expr="//button[@id='hr_attendance_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '&amp;',
                        ('hr_presence_state', '=', 'present'),
                        ('attendance_state', '=', 'checked_out'),
                        ('is_absent', '=', True),
                        ('id', '=', False),
                    ]}
                </attribute>
            </xpath>
            <!-- Merge invisible attr of both module -->
            <xpath expr="//button[@id='hr_presence_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '|',
                        ('is_absent', '=', True),
                        ('hr_presence_state', '=', 'absent'),
                        ('attendance_state', '=', 'checked_in'),
                        ('id', '=', False),
                    ]}
                </attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_view_form_inherit" model="ir.ui.view">
        <field name="name">hr.holidays.attendance.employee.view.form.inherit</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button
                    name="%(hr_leave_allocation_overtime_manager_action)d"
                    string="Deduct Extra Hours"
                    type="action"
                    groups="hr_holidays.group_hr_holidays_user"
                    context="{'default_employee_id': id, 'deduct_extra_hours': True}"
                    attrs="{'invisible': [('total_overtime', '&lt;=', 1)]}"/>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_view_form" model="ir.ui.view">
        <field name="name">hr.employee.holidays.attendance.inherit</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr_attendance.view_employee_form_inherit_hr_attendance"/>
        <field name="arch" type="xml">
            <!-- Hide Attendance button -->
            <xpath expr="//button[@id='hr_attendance_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '&amp;',
                        ('hr_presence_state', '=', 'present'),
                        ('attendance_state', '=', 'checked_out'),
                        ('is_absent', '=', True),
                        ('id', '=', False),
                    ]}
                </attribute>
            </xpath>
            <!-- Merge invisible attr of both module -->
            <xpath expr="//button[@id='hr_presence_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '|',
                        ('is_absent', '=', True),
                        ('hr_presence_state', '=', 'absent'),
                        ('attendance_state', '=', 'checked_in'),
                        ('id', '=', False),
                    ]}
                </attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\hr_leave_allocation_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <record id="hr_attendance_holidays_hr_leave_allocation_view_form_inherit" model="ir.ui.view">
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_allocation_view_form" />
        <field name="arch" type="xml">
            <xpath expr="(//div[@name='duration_display']/span)[last()]" position="after">
                <field name="hr_attendance_overtime" invisible="1" />
                <field name="overtime_deductible" invisible="1" />
                <field name="employee_overtime" invisible="1" />
                <div class="oe_inline"
                    attrs="{'invisible': ['|', '|', '|',
                    ('hr_attendance_overtime', '=', False), ('employee_id', '=', False),
                    ('overtime_deductible', '=', False), ('employee_overtime', '&lt;=', 0)]}">
                    <field name="employee_overtime" nolabel="1" widget="float_time" class="text-success" /> Extra Hours Available
                </div>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_allocation_overtime_view_form" model="ir.ui.view">
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="hr_attendance_holidays_hr_leave_allocation_view_form_inherit" />
        <field name="mode">primary</field>
        <field name="priority">60</field>
        <field name="arch" type="xml">
            <xpath expr="//header" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <div name="button_box" position="attributes">
                <attribute name="invisible">1</attribute>
            </div>
            <xpath expr="//field[@name='holiday_status_id']" position="attributes">
                <attribute name="domain">[('overtime_deductible', '=', True), ('requires_allocation', '=', 'yes'), ('employee_requests', '=', 'yes')]</attribute>
                <attribute name="options">{'no_create': True, 'no_open': True}</attribute>
            </xpath>
            <xpath expr="//sheet" position="after">
                <footer>
                    <button string="Save" special="save" class="btn btn-primary" close="1" />
                    <button string="Discard" special="cancel" class="btn-secondary" close="1" />
                </footer>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_allocation_overtime_manager_view_form" model="ir.ui.view">
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="hr_leave_allocation_overtime_view_form" />
        <field name="mode">primary</field>
        <field name="priority">70</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='holiday_status_id']" position="attributes">
                <attribute name="domain">[('overtime_deductible', '=', True), ('requires_allocation', '=', 'yes')]</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_allocation_overtime_action" model="ir.actions.act_window">
        <field name="name">New Allocation Request</field>
        <field name="res_model">hr.leave.allocation</field>
        <field name="view_mode">form</field>
        <field name="view_ids" eval="[(5, 0, 0), (0, 0, {'view_mode': 'form', 'view_id': ref('hr_leave_allocation_overtime_view_form')})]"/>
        <field name="target">new</field>
    </record>

    <record id="hr_leave_allocation_overtime_manager_action" model="ir.actions.act_window">
        <field name="name">New Allocation Request</field>
        <field name="res_model">hr.leave.allocation</field>
        <field name="view_mode">form</field>
        <field name="view_ids" eval="[(5, 0, 0), (0, 0, {'view_mode': 'form', 'view_id': ref('hr_leave_allocation_overtime_manager_view_form')})]"/>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: views\hr_leave_type_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="hr_leave_type_view_form" model="ir.ui.view">
        <field name="name">hr.leave.type.view.form.inherit</field>
        <field name="model">hr.leave.type</field>
        <field name="inherit_id" ref="hr_holidays.edit_holiday_status_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='request_unit']" position="after">
                <field name="hr_attendance_overtime" invisible="1" />
                <field name="overtime_deductible" attrs="{'invisible': [('hr_attendance_overtime', '=', False)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\hr_leave_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <record id="hr_leave_view_form" model="ir.ui.view">
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_view_form" />
        <field name="arch" type="xml">
            <xpath expr="(//div[@name='duration_display']/div)[last()]" position="after">
                <field name="overtime_deductible" invisible="1" />
                <field name="employee_overtime" invisible="1" />
                <div attrs="{'invisible': ['|', '|', ('employee_id', '=', False), ('overtime_deductible', '=', False), ('employee_overtime', '&lt;=', 0)]}"
                     groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance">
                    <field name="employee_overtime" nolabel="1" widget="float_time" class="text-success" /> Extra Hours Available
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_users_view_form" model="ir.ui.view">
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <field name="employee_id" invisible="1" />
                <field name="request_overtime" invisible="1" />
                <button
                    name="%(hr_leave_allocation_overtime_action)d"
                    string="Deduct Extra Hours"
                    type="action"
                    context="{'default_employee_id': employee_id, 'deduct_extra_hours': True, 'deduct_extra_hours_employee_request': True}"
                    attrs="{'invisible': [('request_overtime', '=', False)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

