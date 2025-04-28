# Odoo Module: l10n_fr_hr_holidays

Category: Human Resources/Time Off

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
    'name': 'France - Time Off',
    'countries': ['fr'],
    'version': '1.0',
    'category': 'Human Resources/Time Off',
    'summary': 'Management of leaves for part-time workers in France',
    'depends': ['hr_holidays'],
    'auto_install': True,
    'license': 'LGPL-3',
    'data': [
        'views/res_config_settings_views.xml',
    ],
    'demo': [
        'data/l10n_fr_hr_holidays_demo.xml',
    ],
}

```

## File: data\l10n_fr_hr_holidays_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_fr_part_time_calendar" model="resource.calendar">
        <field name="name">Part time</field>
        <field name="company_id" eval="False"/>
        <field name="hours_per_day">9</field>
        <field name="attendance_ids"
            eval="[(5, 0, 0),
                (0, 0, {'name': 'Monday Morning', 'dayofweek': '0', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Monday Afternoon', 'dayofweek': '0', 'hour_from': 13, 'hour_to': 18.0, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Tuesday Morning', 'dayofweek': '1', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Tuesday Afternoon', 'dayofweek': '1', 'hour_from': 13, 'hour_to': 18.0, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Thursday Morning', 'dayofweek': '3', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Thursday Afternoon', 'dayofweek': '3', 'hour_from': 13, 'hour_to': 18.0, 'day_period': 'afternoon'}),
            ]"
        />
    </record>
    <record id="base.partner_demo_company_fr" model="res.partner" forcecreate="1">
        <field name="name">FR Company</field>
        <field name="vat">FR91746948785</field>
        <field name="street">Rue Abbé Huet</field>
        <field name="city">Rennes</field>
        <field name="country_id" ref="base.fr"/>

        <field name="zip">35043</field>
        <field name="phone">+33 6 12 34 56 78</field>
        <field name="email">info@company.frexample.com</field>
        <field name="website">www.frexample.com</field>
    </record>

    <record id="base.demo_company_fr" model="res.company" forcecreate="1">
        <field name="name">FR Company</field>
        <field name="partner_id" ref="base.partner_demo_company_fr"/>
    </record>

    <record id="l10n_fr_part_time_employee" model="hr.employee">
        <field name="company_id" ref="base.demo_company_fr"/>
        <field name="active" eval="1"/>
        <field name="name">Mitchell Admin</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="resource_calendar_id" ref="l10n_fr_part_time_calendar"/>
        <field name="image_1920" eval="obj(ref('base.partner_admin')).image_1920" model="res.partner"/>
    </record>

    <record id="l10n_fr_holiday_status_cl" model="hr.leave.type">
        <field name="name">Paid Time Off</field>
        <field name="company_id" ref="base.demo_company_fr"/>
        <field name="requires_allocation">yes</field>
        <field name="employee_requests">no</field>
        <field name="leave_validation_type">both</field>
        <field name="allocation_validation_type">hr</field>
        <field name="responsible_ids" eval="[(4, ref('base.user_admin'))]"/>
        <field name="icon_id" ref="hr_holidays.icon_14"/>
        <field name="color">2</field>
        <field name="has_valid_allocation">True</field>
    </record>

    <record id="l10n_fr_hr_holidays_allocation" model="hr.leave.allocation">
        <field name="name">Paid Time Off allocation</field>
        <field name="state">confirm</field>
        <field name="holiday_status_id" ref="l10n_fr_holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="date_from" eval="time.strftime('%Y-01-01')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
        <field name="employee_id" ref="l10n_fr_part_time_employee"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="[ref('l10n_fr_hr_holidays_allocation')]"/>
    </function>

    <function model="res.company" name="write">
        <value eval="[ref('base.demo_company_fr')]"/>
        <value eval="{'l10n_fr_reference_leave_type': [(4, ref('l10n_fr_holiday_status_cl'))]}"/>
    </function>

    <function model="res.users" name="write">
        <value eval="[ref('base.user_root'), ref('base.user_admin'), ref('base.user_demo')]"/>
        <value eval="{'company_ids': [(4, ref('base.demo_company_fr'))]}"/>
    </function>
</odoo>

```

## File: models\hr_leave.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import fields, models, api, _
from odoo.exceptions import UserError

class HrLeave(models.Model):
    _inherit = 'hr.leave'

    l10n_fr_date_to_changed = fields.Boolean(export_string_translation=False)

    def _l10n_fr_leave_applies(self):
        # The french l10n is meant to be computed only in very specific cases:
        # - there is only one employee affected by the leave
        # - the company is french
        # - the leave_type is the reference leave_type of that company
        self.ensure_one()
        return self.employee_id and \
               self.company_id.country_id.code == 'FR' and \
               self.resource_calendar_id != self.company_id.resource_calendar_id and \
               self.holiday_status_id == self.company_id._get_fr_reference_leave_type()

    def _get_fr_date_from_to(self, date_from, date_to):
        self.ensure_one()
        # What we need to compute is how much we will need to push date_to in order to account for the lost days
        # This gets even more complicated in two_weeks_calendars

        # The following computation doesn't work for resource calendars in
        # which the employee works zero hours.
        if not (self.resource_calendar_id.attendance_ids):
            raise UserError(_("An employee can't take paid time off in a period without any work hours."))

        if not self.request_unit_hours:
            # Use company's working schedule hours for the leave to avoid duration calculation issues.
            def adjust_date_range(date_from, date_to, period, attendance_ids, employee_id):
                period_ids_from = attendance_ids.filtered(lambda a: a.day_period in period
                                                                    and int(a.dayofweek) == date_from.weekday()
                                                                    and (not a.two_weeks_calendar or int(a.week_type) == a.get_week_type(date_from)))
                period_ids_to = attendance_ids.filtered(lambda a: a.day_period in period
                                                                    and int(a.dayofweek) == date_to.weekday()
                                                                    and (not a.two_weeks_calendar or int(a.week_type) == a.get_week_type(date_to)))
                if period_ids_from:
                    min_hour = min(attendance.hour_from for attendance in period_ids_from)
                    date_from = self._to_utc(date_from, min_hour, employee_id)
                if period_ids_to:
                    max_hour = max(attendance.hour_to for attendance in period_ids_to)
                    date_to = self._to_utc(date_to, max_hour, employee_id)
                return date_from, date_to

            if self.request_unit_half:
                period = ['morning'] if self.request_date_from_period == 'am' else ['afternoon']
            else:
                period = ['morning', 'afternoon']
            attendance_ids = self.company_id.resource_calendar_id.attendance_ids
            date_from, date_to = adjust_date_range(date_from, date_to, period, attendance_ids, self.employee_id)

        if self.request_unit_half and self.request_date_from_period == 'am':
            # In normal workflows request_unit_half implies that date_from and date_to are the same
            # request_unit_half allows us to choose between `am` and `pm`
            # In a case where we work from mon-wed and request a half day in the morning
            # we do not want to push date_to since the next work attendance is actually in the afternoon
            date_from_weektype = str(self.env['resource.calendar.attendance'].get_week_type(date_from))
            date_from_dayofweek = str(date_from.weekday())
            # Fetch the attendances we care about
            attendance_ids = self.resource_calendar_id.attendance_ids.filtered(lambda a:
                a.dayofweek == date_from_dayofweek
                and a.day_period != "lunch"
                and (not self.resource_calendar_id.two_weeks_calendar or a.week_type == date_from_weektype))
            if len(attendance_ids) == 2:
                # The employee took the morning off on a day where he works the afternoon aswell
                return (date_from, date_to)

        # Check calendars for working days until we find the right target, start at date_to + 1 day
        # Postpone date_target until the next working day
        date_start = date_from
        date_target = date_to
        # It is necessary to move the start date up to the first work day of
        # the employee calendar as otherwise days worked on by the company
        # calendar before the actual start of the leave would be taken into
        # account.
        while not self.resource_calendar_id._works_on_date(date_start):
            date_start += relativedelta(days=1)
        while not self.resource_calendar_id._works_on_date(date_target + relativedelta(days=1)):
            date_target += relativedelta(days=1)

        # Undo the last day increment
        return (date_start, date_target)

    @api.depends('request_date_from_period', 'request_hour_from', 'request_hour_to', 'request_date_from', 'request_date_to',
                 'request_unit_half', 'request_unit_hours', 'employee_id')
    def _compute_date_from_to(self):
        super()._compute_date_from_to()
        for leave in self:
            if leave._l10n_fr_leave_applies():
                new_date_from, new_date_to = leave._get_fr_date_from_to(leave.date_from, leave.date_to)
                if new_date_from != leave.date_from:
                    leave.date_from = new_date_from
                if new_date_to != leave.date_to:
                    leave.date_to = new_date_to
                    leave.l10n_fr_date_to_changed = True
                else:
                    leave.l10n_fr_date_to_changed = False

    def _get_durations(self, check_leave_type=True, resource_calendar=None):
        """
        In french time off laws, if an employee has a part time contract, when taking time off
        before one of his off day (compared to the company's calendar) it should also count the time
        between the time off and the next calendar work day/company off day (weekends).

        For example take an employee working mon-wed in a company where the regular calendar is mon-fri.
        If the employee were to take a time off ending on wednesday, the legal duration would count until friday.
        """
        if not resource_calendar:
            fr_leaves = self.filtered(lambda leave: leave._l10n_fr_leave_applies())
            duration_by_leave_id = super(HrLeave, self - fr_leaves)._get_durations(resource_calendar=resource_calendar)
            fr_leaves_by_company = fr_leaves.grouped('company_id')
            for company, leaves in fr_leaves_by_company.items():
                duration_by_leave_id.update(leaves._get_durations(resource_calendar=company.resource_calendar_id))
            return duration_by_leave_id
        return super()._get_durations(resource_calendar=resource_calendar)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import ValidationError


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_fr_reference_leave_type = fields.Many2one(
        'hr.leave.type',
        string='Company Paid Time Off Type')

    def _get_fr_reference_leave_type(self):
        self.ensure_one()
        if not self.l10n_fr_reference_leave_type:
            raise ValidationError(_("You must first define a reference time off type for the company."))
        return self.l10n_fr_reference_leave_type

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_fr_reference_leave_type = fields.Many2one(
        'hr.leave.type',
        related='company_id.l10n_fr_reference_leave_type',
        readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_leave
from . import res_company
from . import res_config_settings

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.hr</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="70"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <block name="work_organization_setting_container" position="after">
                <field name="company_country_code" invisible="1"/>
                <block title="French Time Off Localization" invisible="company_country_code != 'FR'">
                    <setting company_dependent="1" help="Set the time off type used as the company Paid Time Off to compute part-timers leave duration">
                        <field name="l10n_fr_reference_leave_type"
                            class="o_light_label"
                            domain="[('company_id', 'in', [company_id, False])]"
                            context="{'default_company_id': company_id}"/>
                    </setting>
                </block>
            </block>
        </field>
    </record>

    <menuitem id="hr_holidays_menu_configuration"
        name="Settings"
        parent="hr_holidays.menu_hr_holidays_configuration"
        sequence="10"
        action="hr.hr_config_settings_action"
        groups="base.group_system"/>
</odoo>

```

