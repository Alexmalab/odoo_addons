# Odoo Module: project_timesheet_holidays

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from odoo import _


def post_init(env):
    """ Set the timesheet project and task on existing leave type. Do it in post_init to
        be sure the internal project/task of res.company are set. (Since timesheet_generate field
        is true by default, those 2 fields are required on the leave type).
    """
    type_ids_ref = env.ref('hr_timesheet.internal_project_default_stage', raise_if_not_found=False)
    type_ids = [(4, type_ids_ref.id)] if type_ids_ref else []
    companies = env['res.company'].search(['|', ('internal_project_id', '=', False), ('leave_timesheet_task_id', '=', False)])
    internal_projects_by_company_dict = None
    project = env['project.project']
    for company in companies:
        company = company.with_company(company)
        if not company.internal_project_id:
            if not internal_projects_by_company_dict:
                internal_projects_by_company_read = project.search_read([
                    ('name', '=', _('Internal')),
                    ('allow_timesheets', '=', True),
                    ('company_id', 'in', companies.ids),
                ], ['company_id', 'id'])
                internal_projects_by_company_dict = {res['company_id'][0]: res['id'] for res in internal_projects_by_company_read}
            project_id = internal_projects_by_company_dict.get(company.id, False)
            if not project_id:
                project_id = project.create({
                    'name': _('Internal'),
                    'allow_timesheets': True,
                    'company_id': company.id,
                    'type_ids': type_ids,
                }).id
            company.write({'internal_project_id': project_id})
        if not company.leave_timesheet_task_id:
            task = company.env['project.task'].create({
                'name': _('Time Off'),
                'project_id': company.internal_project_id.id,
                'active': True,
                'company_id': company.id,
            })
            company.write({
                'leave_timesheet_task_id': task.id,
            })

    for hr_leave_type in env['hr.leave.type'].search([('timesheet_generate', '=', True), ('company_id', '!=', False), ('timesheet_project_id', '=', False)]):
        project_id = hr_leave_type.company_id.internal_project_id
        default_task_id = hr_leave_type.company_id.leave_timesheet_task_id
        hr_leave_type.write({
            'timesheet_project_id': project_id.id,
            'timesheet_task_id': default_task_id.id if default_task_id and default_task_id.project_id == project_id else False,
        })

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Timesheet when on Time Off',
    'version': '1.0',
    'category': 'Human Resources',
    'summary': 'Schedule timesheet when on time off',
    'description': """
Bridge module to integrate leaves in timesheet
================================================

This module allows to automatically log timesheets when employees are
on leaves. Project and task can be configured company-wide.
    """,
    'depends': ['hr_timesheet', 'hr_holidays'],
    'data': [
        'views/res_config_settings_views.xml',
        'views/hr_holidays_views.xml',
        'views/project_task_views.xml',
        'security/ir.model.access.csv',

    ],
    'demo': [
        'data/holiday_timesheets_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'post_init_hook': 'post_init',
    'license': 'LGPL-3',
}

```

## File: data\holiday_timesheets_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <function model="hr.leave" name="_remove_resource_leave">
            <value eval="[ref('hr_holidays.hr_holidays_sl'), ref('hr_holidays.hr_holidays_cl_qdp'), ref('hr_holidays.hr_holidays_sl_qdp')]"/>
        </function>
        <function model="hr.leave" name="_validate_leave_request">
            <value eval="[ref('hr_holidays.hr_holidays_sl'), ref('hr_holidays.hr_holidays_cl_qdp'), ref('hr_holidays.hr_holidays_sl_qdp')]"/>
        </function>
        <function model="hr.leave" name="_create_resource_leave">
            <value eval="[ref('hr_holidays.hr_holidays_sl'), ref('hr_holidays.hr_holidays_cl_qdp'), ref('hr_holidays.hr_holidays_sl_qdp')]"/>
        </function>
    </data>
</odoo>

```

## File: models\account_analytic.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import RedirectWarning, UserError
from odoo.osv import expression


class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    holiday_id = fields.Many2one("hr.leave", string='Time Off Request', copy=False, index='btree_not_null')
    global_leave_id = fields.Many2one("resource.calendar.leaves", string="Global Time Off", index='btree_not_null', ondelete='cascade')
    task_id = fields.Many2one(domain="[('allow_timesheets', '=', True), ('project_id', '=?', project_id), ('is_timeoff_task', '=', False)]")

    def _get_redirect_action(self):
        leave_form_view_id = self.env.ref('hr_holidays.hr_leave_view_form').id
        action_data = {
           'name': _('Time Off'),
           'type': 'ir.actions.act_window',
           'res_model': 'hr.leave',
           'views': [(self.env.ref('hr_holidays.hr_leave_view_tree_my').id, 'list'), (leave_form_view_id, 'form')],
           'domain': [('id', 'in', self.holiday_id.ids)],
        }
        if len(self.holiday_id) == 1:
            action_data['views'] = [(leave_form_view_id, 'form')]
            action_data['res_id'] = self.holiday_id.id
        return action_data

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_leave(self):
        if any(line.global_leave_id for line in self):
            raise UserError(_('You cannot delete timesheets that are linked to global time off.'))
        elif any(line.holiday_id for line in self):
            if not self.env.user.has_group('hr_holidays.group_hr_holidays_user') and self.env.user not in self.holiday_id.sudo().user_id:
                raise UserError(_('You cannot delete timesheets that are linked to time off requests. Please cancel your time off request from the Time Off application instead.'))
            warning_msg = _('You cannot delete timesheets linked to time off. Please, cancel the time off instead.')
            action = self._get_redirect_action()
            raise RedirectWarning(warning_msg, action, _('View Time Off'))

    def _check_can_write(self, values):
        if not self.env.su and self.holiday_id:
            raise UserError(_('You cannot modify timesheets that are linked to time off requests. Please use the Time Off application to modify your time off requests instead.'))
        return super()._check_can_write(values)

    def _check_can_create(self):
        if not self.env.su and any(task.is_timeoff_task for task in self.task_id):
            raise UserError(_('You cannot create timesheets for a task that is linked to a time off type. Please use the Time Off application to request new time off instead.'))
        return  super()._check_can_create()

    def _get_favorite_project_id_domain(self, employee_id=False):
        return expression.AND([
            super()._get_favorite_project_id_domain(employee_id),
            [('holiday_id', '=', False), ('global_leave_id', '=', False)],
        ])

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from collections import defaultdict


class Employee(models.Model):
    _inherit = 'hr.employee'

    @api.model_create_multi
    def create(self, vals_list):
        employees = super().create(vals_list)
        if self.env.context.get('salary_simulation'):
            return employees

        # We need to create timesheet entries for the global time off that are already created
        # and are planned for after this employee creation date
        self.with_context(allowed_company_ids=employees.company_id.ids) \
            ._create_future_public_holidays_timesheets(employees)
        return employees

    def write(self, vals):
        if vals.get('active'):
            inactive_emp = self.filtered(lambda e: not e.active)
        result = super(Employee, self).write(vals)
        self_company = self.with_context(allowed_company_ids=self.company_id.ids)
        if 'active' in vals:
            if vals.get('active'):
                # Create future holiday timesheets
                inactive_emp = inactive_emp.with_env(self_company.env)
                inactive_emp._create_future_public_holidays_timesheets(inactive_emp)
            else:
                # Delete future holiday timesheets
                self_company._delete_future_public_holidays_timesheets()
        elif 'resource_calendar_id' in vals:
            # Update future holiday timesheets
            self_company._delete_future_public_holidays_timesheets()
            self_company._create_future_public_holidays_timesheets(self_company)
        return result

    def _delete_future_public_holidays_timesheets(self):
        future_timesheets = self.env['account.analytic.line'].sudo().search([('global_leave_id', '!=', False), ('date', '>=', fields.date.today()), ('employee_id', 'in', self.ids)])
        future_timesheets.write({'global_leave_id': False})
        future_timesheets.unlink()

    def _create_future_public_holidays_timesheets(self, employees):
        lines_vals = []
        today = fields.Datetime.today()
        global_leaves_wo_calendar = defaultdict(lambda: self.env["resource.calendar.leaves"])
        global_leaves_wo_calendar.update(dict(self.env['resource.calendar.leaves']._read_group(
            [('calendar_id', '=', False), ('date_from', '>=', today)],
            groupby=['company_id'],
            aggregates=['id:recordset'],
        )))
        for employee in employees:
            if not employee.active:
                continue
            # First we look for the global time off that are already planned after today
            global_leaves = employee.resource_calendar_id.global_leave_ids.filtered(lambda l: l.date_from >= today) + global_leaves_wo_calendar[employee.company_id]
            work_hours_data = global_leaves._work_time_per_day()
            for global_time_off in global_leaves:
                for index, (day_date, work_hours_count) in enumerate(work_hours_data[employee.resource_calendar_id.id][global_time_off.id]):
                    lines_vals.append(
                        global_time_off._timesheet_prepare_line_values(
                            index,
                            employee,
                            work_hours_data[global_time_off.id],
                            day_date,
                            work_hours_count
                        )
                    )
        return self.env['account.analytic.line'].sudo().create(lines_vals)

```

## File: models\hr_holidays.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class HolidaysType(models.Model):
    _inherit = "hr.leave.type"

    timesheet_generate = fields.Boolean(
        'Generate Timesheets', compute='_compute_timesheet_generate', store=True, readonly=False,
        help="If checked, when validating a time off, timesheet will be generated in the Vacation Project of the company.")
    timesheet_project_id = fields.Many2one('project.project', string="Project", domain="[('company_id', 'in', [False, company_id])]",
        compute="_compute_timesheet_project_id", store=True, readonly=False)
    timesheet_task_id = fields.Many2one(
        'project.task', string="Task", compute='_compute_timesheet_task_id',
        store=True, readonly=False,
        domain="[('project_id', '=', timesheet_project_id),"
                "('project_id', '!=', False),"
                "('company_id', 'in', [False, company_id])]")

    @api.depends('timesheet_task_id', 'timesheet_project_id')
    def _compute_timesheet_generate(self):
        for leave_type in self:
            leave_type.timesheet_generate = not leave_type.company_id or (leave_type.timesheet_task_id and leave_type.timesheet_project_id)

    @api.depends('company_id')
    def _compute_timesheet_project_id(self):
        for leave in self:
            leave.timesheet_project_id = leave.company_id.internal_project_id

    @api.depends('timesheet_project_id')
    def _compute_timesheet_task_id(self):
        for leave_type in self:
            default_task_id = leave_type.company_id.leave_timesheet_task_id

            if default_task_id and default_task_id.project_id == leave_type.timesheet_project_id:
                leave_type.timesheet_task_id = default_task_id
            else:
                leave_type.timesheet_task_id = False

    @api.constrains('timesheet_generate', 'timesheet_project_id', 'timesheet_task_id')
    def _check_timesheet_generate(self):
        for holiday_status in self:
            if holiday_status.timesheet_generate and holiday_status.company_id:
                if not holiday_status.timesheet_project_id or not holiday_status.timesheet_task_id:
                    raise ValidationError(_("Both the internal project and task are required to "
                    "generate a timesheet for the time off %s. If you don't want a timesheet, you should "
                    "leave the internal project and task empty.", holiday_status.name))


class Holidays(models.Model):
    _inherit = "hr.leave"

    timesheet_ids = fields.One2many('account.analytic.line', 'holiday_id', string="Analytic Lines")

    def _validate_leave_request(self):
        """ Timesheet will be generated on leave validation only if timesheet_generate is True
            If company is set, timesheet_project_id and timesheet_task_id from leave type are
            used as project_id and task_id.
            Else, internal_project_id and leave_timesheet_task_id are used.
            The generated timesheet will be attached to this project/task.
        """
        vals_list = []
        leave_ids = []
        for leave in self:
            if leave.holiday_type != 'employee' or not leave.holiday_status_id.timesheet_generate:
                continue

            if leave.holiday_status_id.company_id:
                project, task = leave.holiday_status_id.timesheet_project_id, leave.holiday_status_id.timesheet_task_id
            else:
                project, task = leave.employee_id.company_id.internal_project_id, leave.employee_id.company_id.leave_timesheet_task_id

            if not project or not task:
                continue

            leave_ids.append(leave.id)
            if not leave.employee_id:
                continue

            work_hours_data = leave.employee_id.list_work_time_per_day(
                leave.date_from,
                leave.date_to)

            for index, (day_date, work_hours_count) in enumerate(work_hours_data):
                vals_list.append(leave._timesheet_prepare_line_values(index, work_hours_data, day_date, work_hours_count, project, task))

        # Unlink previous timesheets to avoid doublon (shouldn't happen on the interface but meh)
        old_timesheets = self.env["account.analytic.line"].sudo().search([('project_id', '!=', False), ('holiday_id', 'in', leave_ids)])
        if old_timesheets:
            old_timesheets.holiday_id = False
            old_timesheets.unlink()

        self.env['account.analytic.line'].sudo().create(vals_list)

        return super()._validate_leave_request()

    def _timesheet_prepare_line_values(self, index, work_hours_data, day_date, work_hours_count, project, task):
        self.ensure_one()
        return {
            'name': _("Time Off (%s/%s)", index + 1, len(work_hours_data)),
            'project_id': project.id,
            'task_id': task.id,
            'account_id': project.sudo().analytic_account_id.id,
            'unit_amount': work_hours_count,
            'user_id': self.employee_id.user_id.id,
            'date': day_date,
            'holiday_id': self.id,
            'employee_id': self.employee_id.id,
            'company_id': task.sudo().company_id.id or project.sudo().company_id.id,
        }

    def _check_missing_global_leave_timesheets(self):
        if not self:
            return
        min_date = min([leave.date_from for leave in self])
        max_date = max([leave.date_to for leave in self])

        global_leaves = self.env['resource.calendar.leaves'].search([
            ("resource_id", "=", False),
            ("date_to", ">=", min_date),
            ("date_from", "<=", max_date),
            ("company_id.internal_project_id", "!=", False),
            ("company_id.leave_timesheet_task_id", "!=", False),
        ])
        if global_leaves:
            global_leaves._generate_public_time_off_timesheets(self.sudo().employee_ids)

    def action_refuse(self):
        """ Remove the timesheets linked to the refused holidays """
        result = super(Holidays, self).action_refuse()
        timesheets = self.sudo().mapped('timesheet_ids')
        timesheets.write({'holiday_id': False})
        timesheets.unlink()
        self._check_missing_global_leave_timesheets()
        return result

    def _action_user_cancel(self, reason):
        res = super()._action_user_cancel(reason)
        timesheets = self.sudo().timesheet_ids
        timesheets.write({'holiday_id': False})
        timesheets.unlink()
        self._check_missing_global_leave_timesheets()
        return res

    def _force_cancel(self, *args, **kwargs):
        super()._force_cancel(*args, **kwargs)
        # override this method to reevaluate timesheets after the leaves are updated via force cancel
        timesheets = self.sudo().timesheet_ids
        timesheets.holiday_id = False
        timesheets.unlink()

    def write(self, vals):
        res = super().write(vals)
        # reevaluate timesheets after the leaves are wrote in order to remove empty timesheets
        timesheet_ids_to_remove = []
        for leave in self:
            if leave.number_of_days == 0 and leave.sudo().timesheet_ids:
                leave.sudo().timesheet_ids.holiday_id = False
                timesheet_ids_to_remove.extend(leave.timesheet_ids)
        self.env['account.analytic.line'].browse(set(timesheet_ids_to_remove)).sudo().unlink()
        return res

```

## File: models\project_task.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _

class Task(models.Model):
    _inherit = 'project.task'

    leave_types_count = fields.Integer(compute='_compute_leave_types_count', string="Time Off Types Count")
    is_timeoff_task = fields.Boolean("Is Time off Task", compute="_compute_is_timeoff_task", search="_search_is_timeoff_task")

    def _compute_leave_types_count(self):
        time_off_type_read_group = self.env['hr.leave.type']._read_group(
            [('timesheet_task_id', 'in', self.ids)],
            ['timesheet_task_id'],
            ['__count'],
        )
        time_off_type_count_per_task = {timesheet_task.id: count for timesheet_task, count in time_off_type_read_group}
        for task in self:
            task.leave_types_count = time_off_type_count_per_task.get(task.id, 0)

    def _compute_is_timeoff_task(self):
        timeoff_tasks = self.filtered(lambda task: task.leave_types_count or task.company_id.leave_timesheet_task_id == task)
        timeoff_tasks.is_timeoff_task = True
        (self - timeoff_tasks).is_timeoff_task = False

    def _search_is_timeoff_task(self, operator, value):
        if operator not in ['=', '!='] or not isinstance(value, bool):
            raise NotImplementedError(_('Operation not supported'))
        leave_type_read_group = self.env['hr.leave.type']._read_group(
            [('timesheet_task_id', '!=', False)],
            [],
            ['timesheet_task_id:recordset'],
        )
        [timeoff_tasks] = leave_type_read_group[0]
        if self.env.company.leave_timesheet_task_id:
            timeoff_tasks |= self.env.company.leave_timesheet_task_id
        if operator == '!=':
            value = not value
        return [('id', 'in' if value else 'not in', timeoff_tasks.ids)]

```

## File: models\resource_calendar_leaves.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from pytz import timezone, utc

from odoo import api, fields, models, _


class ResourceCalendarLeaves(models.Model):
    _inherit = "resource.calendar.leaves"

    timesheet_ids = fields.One2many('account.analytic.line', 'global_leave_id', string="Analytic Lines")

    def _get_resource_calendars(self):
        leaves_with_calendar = self.filtered('calendar_id')
        calendars = leaves_with_calendar.calendar_id
        leaves_wo_calendar = self - leaves_with_calendar
        if leaves_wo_calendar:
            calendars += self.env['resource.calendar'].search([('company_id', 'in', leaves_wo_calendar.company_id.ids)])
        return calendars

    def _work_time_per_day(self, resource_calendars=False):
        """ Get work time per day based on the calendar and its attendances

            1) Gets all calendars with their characteristics (i.e.
                (a) the leaves in it,
                (b) the resources which have a leave,
                (c) the oldest and
                (d) the latest leave dates
               ) for leaves in self (first for calendar's leaves, then for company's global leaves)
            2) Search the attendances based on the characteristics retrieved for each calendar.
                The attendances found are the ones between the date_from of the oldest leave
                and the date_to of the most recent leave.
            3) Create a dict as result of this method containing:
                {
                    leave: {
                            max(date_start of work hours, date_start of the leave):
                                the duration in days of the work including the leave
                    }
                }
        """
        resource_calendars = resource_calendars or self._get_resource_calendars()
        # to easily find the calendar with its id.
        calendars_dict = {calendar.id: calendar for calendar in resource_calendars}

        leaves_read_group = self.env['resource.calendar.leaves']._read_group(
            [('id', 'in', self.ids), ('calendar_id', '!=', False)],
            ['calendar_id'],
            ['id:recordset', 'resource_id:recordset', 'date_from:min', 'date_to:max'],
        )
        # dict of keys: calendar_id
        #   and values : { 'date_from': datetime, 'date_to': datetime, resources: self.env['resource.resource'] }
        cal_attendance_intervals_dict = {}
        for calendar, leaves, resources, date_from_min, date_to_max in leaves_read_group:
            calendar_data = {
                'date_from': utc.localize(date_from_min),
                'date_to': utc.localize(date_to_max),
                'resources': resources,
                'leaves': leaves,
            }
            cal_attendance_intervals_dict[calendar.id] = calendar_data

        comp_leaves_read_group = self.env['resource.calendar.leaves']._read_group(
            [('id', 'in', self.ids), ('calendar_id', '=', False)],
            ['company_id'],
            ['id:recordset', 'resource_id:recordset', 'date_from:min', 'date_to:max'],
        )
        for company, leaves, resources, date_from_min, date_to_max in comp_leaves_read_group:
            for calendar_id in resource_calendars.ids:
                if calendars_dict[calendar_id].company_id != company:
                    continue  # only consider global leaves of the same company as the calendar
                calendar_data = cal_attendance_intervals_dict.get(calendar_id)
                if calendar_data is None:
                    calendar_data = {
                        'date_from': utc.localize(date_from_min),
                        'date_to': utc.localize(date_to_max),
                        'resources': resources,
                        'leaves': leaves,
                    }
                    cal_attendance_intervals_dict[calendar_id] = calendar_data
                else:
                    calendar_data.update(
                        date_from=min(utc.localize(date_from_min), calendar_data['date_from']),
                        date_to=max(utc.localize(date_to_max), calendar_data['date_to']),
                        resources=resources | calendar_data['resources'],
                        leaves=leaves | calendar_data['leaves'],
                    )

        # dict of keys: calendar_id
        #   and values: a dict of keys: leave.id
        #         and values: a dict of keys: date
        #              and values: number of days
        results = defaultdict(lambda: defaultdict(lambda: defaultdict(float)))
        for calendar_id, cal_attendance_intervals_params_entry in cal_attendance_intervals_dict.items():
            calendar = calendars_dict[calendar_id]
            work_hours_intervals = calendar._attendance_intervals_batch(
                cal_attendance_intervals_params_entry['date_from'],
                cal_attendance_intervals_params_entry['date_to'],
                cal_attendance_intervals_params_entry['resources'],
                tz=timezone(calendar.tz)
            )
            for leave in cal_attendance_intervals_params_entry['leaves']:
                work_hours_data = work_hours_intervals[leave.resource_id.id]

                for date_from, date_to, dummy in work_hours_data:
                    if date_to > utc.localize(leave.date_from) and date_from < utc.localize(leave.date_to):
                        tmp_start = max(date_from, utc.localize(leave.date_from))
                        tmp_end = min(date_to, utc.localize(leave.date_to))
                        results[calendar_id][leave.id][tmp_start.date()] += (tmp_end - tmp_start).total_seconds() / 3600
                results[calendar_id][leave.id] = sorted(results[calendar_id][leave.id].items())
        return results

    def _timesheet_create_lines(self):
        """ Create timesheet leaves for each employee using the same calendar containing in self.calendar_id

            If the employee has already a time off in the same day then no timesheet should be created.
        """
        resource_calendars = self._get_resource_calendars()
        work_hours_data = self._work_time_per_day(resource_calendars)
        employees_groups = self.env['hr.employee']._read_group(
            [('resource_calendar_id', 'in', resource_calendars.ids), ('company_id', 'in', self.env.companies.ids)],
            ['resource_calendar_id'],
            ['id:recordset'])
        mapped_employee = {
            resource_calendar.id: employees
            for resource_calendar, employees in employees_groups
        }
        employee_ids_all = [_id for __, employees in employees_groups for _id in employees._ids]
        min_date = max_date = None
        for values in work_hours_data.values():
            for vals in values.values():
                for d, dummy in vals:
                    if not min_date and not max_date:
                        min_date = max_date = d
                    elif d < min_date:
                        min_date = d
                    elif d > max_date:
                        max_date = d

        holidays_read_group = self.env['hr.leave']._read_group([
            ('employee_id', 'in', employee_ids_all),
            ('date_from', '<=', max_date),
            ('date_to', '>=', min_date),
            ('state', '=', 'validate'),
        ], ['employee_id'], ['date_from:array_agg', 'date_to:array_agg'])
        holidays_by_employee = {
            employee.id: [
                (date_from.date(), date_to.date()) for date_from, date_to in zip(date_from_list, date_to_list)
            ] for employee, date_from_list, date_to_list in holidays_read_group
        }
        vals_list = []

        def get_timesheets_data(employees, work_hours_list, vals_list):
            for employee in employees:
                holidays = holidays_by_employee.get(employee.id)
                for index, (day_date, work_hours_count) in enumerate(work_hours_list):
                    if not holidays or all(not (date_from <= day_date and date_to >= day_date) for date_from, date_to in holidays):
                        vals_list.append(
                            leave._timesheet_prepare_line_values(
                                index,
                                employee,
                                work_hours_list,
                                day_date,
                                work_hours_count
                            )
                        )
            return vals_list

        for leave in self:
            if not leave.calendar_id:
                for calendar_id, calendar_employees in mapped_employee.items():
                    work_hours_list = work_hours_data[calendar_id][leave.id]
                    vals_list = get_timesheets_data(calendar_employees, work_hours_list, vals_list)
            else:
                employees = mapped_employee.get(leave.calendar_id.id, self.env['hr.employee'])
                work_hours_list = work_hours_data[leave.calendar_id.id][leave.id]
                vals_list = get_timesheets_data(employees, work_hours_list, vals_list)

        return self.env['account.analytic.line'].sudo().create(vals_list)

    def _timesheet_prepare_line_values(self, index, employee_id, work_hours_data, day_date, work_hours_count):
        self.ensure_one()
        return {
            'name': _("Time Off (%s/%s)", index + 1, len(work_hours_data)),
            'project_id': employee_id.company_id.internal_project_id.id,
            'task_id': employee_id.company_id.leave_timesheet_task_id.id,
            'account_id': employee_id.company_id.internal_project_id.analytic_account_id.id,
            'unit_amount': work_hours_count,
            'user_id': employee_id.user_id.id,
            'date': day_date,
            'global_leave_id': self.id,
            'employee_id': employee_id.id,
            'company_id': employee_id.company_id.id,
        }

    def _generate_timesheeets(self):
        results_with_leave_timesheet = self.filtered(lambda r: not r.resource_id and r.company_id.internal_project_id and r.company_id.leave_timesheet_task_id)
        if results_with_leave_timesheet:
            results_with_leave_timesheet._timesheet_create_lines()

    def _generate_public_time_off_timesheets(self, employees):
        timesheet_vals_list = []
        resource_calendars = self._get_resource_calendars()
        work_hours_data = self._work_time_per_day(resource_calendars)
        timesheet_read_group = self.env['account.analytic.line']._read_group(
            [('global_leave_id', 'in', self.ids), ('employee_id', 'in', employees.ids)],
            ['employee_id'],
            ['date:array_agg']
        )
        timesheet_dates_per_employee_id = {
            employee.id: date
            for employee, date in timesheet_read_group
        }
        for leave in self:
            for employee in employees:
                if leave.calendar_id and employee.resource_calendar_id != leave.calendar_id:
                    continue
                calendar = leave.calendar_id or employee.resource_calendar_id
                work_hours_list = work_hours_data[calendar.id][leave.id]
                timesheet_dates = timesheet_dates_per_employee_id.get(employee.id, [])
                for index, (day_date, work_hours_count) in enumerate(work_hours_list):
                    generate_timesheet = day_date not in timesheet_dates
                    if not generate_timesheet:
                        continue
                    timesheet_vals = leave._timesheet_prepare_line_values(
                        index,
                        employee,
                        work_hours_list,
                        day_date,
                        work_hours_count
                    )
                    timesheet_vals_list.append(timesheet_vals)
        return self.env['account.analytic.line'].sudo().create(timesheet_vals_list)

    @api.model_create_multi
    def create(self, vals_list):
        results = super(ResourceCalendarLeaves, self).create(vals_list)
        results._generate_timesheeets()
        return results

    def write(self, vals):
        date_from, date_to, calendar_id = vals.get('date_from'), vals.get('date_to'), vals.get('calendar_id')
        global_time_off_updated = self.env['resource.calendar.leaves']
        if date_from or date_to or 'calendar_id' in vals:
            global_time_off_updated = self.filtered(lambda r: (date_from is not None and r.date_from != date_from) or (date_to is not None and r.date_to != date_to) or (calendar_id is None or r.calendar_id.id != calendar_id))
            timesheets = global_time_off_updated.sudo().timesheet_ids
            if timesheets:
                timesheets.write({'global_leave_id': False})
                timesheets.unlink()
        result = super(ResourceCalendarLeaves, self).write(vals)
        global_time_off_updated and global_time_off_updated.sudo()._generate_timesheeets()
        return result

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class Company(models.Model):
    _inherit = 'res.company'

    leave_timesheet_task_id = fields.Many2one(
        'project.task', string="Time Off Task",
        domain="[('project_id', '=', internal_project_id)]")

    def _create_internal_project_task(self):
        projects = super()._create_internal_project_task()
        for project in projects:
            company = project.company_id
            company = company.with_company(company)
            if not company.leave_timesheet_task_id:
                task = company.env['project.task'].sudo().create({
                    'name': _('Time Off'),
                    'project_id': company.internal_project_id.id,
                    'active': True,
                    'company_id': company.id,
                })
                company.write({
                    'leave_timesheet_task_id': task.id,
                })
        return projects

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    internal_project_id = fields.Many2one(
        related='company_id.internal_project_id', required=True, string="Internal Project",
        domain="[('company_id', '=', company_id)]", readonly=False,
        help="The default project used when automatically generating timesheets via time off requests."
             " You can specify another project on each time off type individually.")
    leave_timesheet_task_id = fields.Many2one(
        related='company_id.leave_timesheet_task_id', string="Time Off Task", readonly=False,
        domain="[('company_id', '=', company_id), ('project_id', '=?', internal_project_id)]",
        help="The default task used when automatically generating timesheets via time off requests."
             " You can specify another task on each time off type individually.")

    @api.onchange('internal_project_id')
    def _onchange_timesheet_project_id(self):
        if self.internal_project_id != self.leave_timesheet_task_id.project_id:
            self.leave_timesheet_task_id = False

    @api.onchange('leave_timesheet_task_id')
    def _onchange_timesheet_task_id(self):
        if self.leave_timesheet_task_id:
            self.internal_project_id = self.leave_timesheet_task_id.project_id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company # has to be before hr_holidays to create needed columns on res.company
from . import account_analytic
from . import hr_holidays
from . import project_task
from . import res_config_settings
from . import resource_calendar_leaves
from . import hr_employee

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_account_analytic_account_leaves_manager,account.analytic.account,analytic.model_account_analytic_account,hr_holidays.group_hr_holidays_manager,1,0,0,0

```

## File: views\hr_holidays_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="hr_holiday_status_view_form_inherit" model="ir.ui.view">
        <field name="name">hr.leave.type.form</field>
        <field name="model">hr.leave.type</field>
        <field name="inherit_id" ref="hr_holidays.edit_holiday_status_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='visual']" position="before">
                <group name="timesheet" groups="base.group_no_one" string="Timesheets">
                    <div class="text-muted mb-4" colspan="2">
                        Generate timesheets when validating time off requests of this type
                    </div>
                    <group>
                        <field name="timesheet_project_id" context="{'active_test': False}" invisible="not company_id"/>
                        <field name="company_id" invisible="1"/>
                        <field name="timesheet_task_id" context="{'active_test': False, 'default_project_id': timesheet_project_id}" invisible="not timesheet_project_id" required="timesheet_project_id"/>
                        <field name="timesheet_generate" invisible="company_id"/>
                    </group>
                </group>

            </xpath>
        </field>
    </record>

</odoo>
```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="leave_task_form_view">
        <field name="name">project.task.leave.form.view</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="hr_timesheet.view_task_form2_inherited"/>
        <field name="arch" type="xml">
            <xpath expr="/form" position="inside">
                <field name="is_timeoff_task" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='timesheet_ids']" position="attributes">
                <attribute name="invisible">not analytic_account_active</attribute>
                <attribute name="readonly">is_timeoff_task</attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.project.timesheet.holidays</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="hr_timesheet.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='timesheet_off_validation_setting']" position="inside">
                <div invisible="not module_project_timesheet_holidays">
                    <div class="row mt16">
                        <div class="w-100">
                            <label string="Project" for="internal_project_id" class="col-2 col-lg-3"/>
                            <field name="internal_project_id" context="{'active_test': False, 'default_company_id': company_id}" class="oe_inline ml16"/>
                        </div>
                        <div class="w-100">
                            <label string="Task" for="leave_timesheet_task_id" class="col-2 col-lg-3"/>
                            <field name="leave_timesheet_task_id" context="{'active_test': False, 'default_project_id': internal_project_id}" class="oe_inline ml16"/>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

