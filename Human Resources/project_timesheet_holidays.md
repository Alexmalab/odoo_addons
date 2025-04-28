# Odoo Module: project_timesheet_holidays

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from odoo import _


def post_init(cr, registry):
    """ Set the timesheet project and task on existing leave type. Do it in post_init to
        be sure the internal project/task of res.company are set. (Since timesheet_generate field
        is true by default, those 2 fields are required on the leave type).
    """
    from odoo import api, SUPERUSER_ID

    env = api.Environment(cr, SUPERUSER_ID, {})

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

    for hr_leave_type in env['hr.leave.type'].search(['|', ('timesheet_project_id', '=', False), ('timesheet_task_id', '=', False)]):
        company = hr_leave_type.company_id or env.company
        hr_leave_type.write({
            'timesheet_project_id': company.internal_project_id.id,
            'timesheet_task_id': company.leave_timesheet_task_id.id,
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
        'security/ir.model.access.csv',

    ],
    'installable': True,
    'auto_install': True,
    'post_init_hook': 'post_init',
    'license': 'LGPL-3',
}

```

## File: models\account_analytic.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression


class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    holiday_id = fields.Many2one("hr.leave", string='Leave Request', copy=False)
    global_leave_id = fields.Many2one("resource.calendar.leaves", string="Global Time Off", ondelete='cascade')
    task_id = fields.Many2one(domain="[('allow_timesheets', '=', True),"
        "('project_id', '=?', project_id), ('is_timeoff_task', '=', False)]")

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_leave(self):
        if any(line.holiday_id for line in self):
            raise UserError(_('You cannot delete timesheets that are linked to time off requests. Please cancel your time off request from the Time Off application instead.'))

    @api.model_create_multi
    def create(self, vals_list):
        if not self.env.su:
            task_ids = [vals['task_id'] for vals in vals_list if vals.get('task_id')]
            has_timeoff_task = self.env['project.task'].search_count([('id', 'in', task_ids), ('is_timeoff_task', '=', True)], limit=1) > 0
            if has_timeoff_task:
                raise UserError(_('You cannot create timesheets for a task that is linked to a time off type. Please use the Time Off application to request new time off instead.'))
        return super().create(vals_list)

    def _check_can_update_timesheet(self):
        return self.env.su or not self.filtered('holiday_id')

    def write(self, vals):
        if not self._check_can_update_timesheet():
            raise UserError(_('You cannot modify timesheets that are linked to time off requests. Please use the Time Off application to modify your time off requests instead.'))
        return super().write(vals)

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
        result = super(Employee, self).write(vals)
        self_company = self.with_context(allowed_company_ids=self.company_id.ids)
        if 'active' in vals:
            if vals.get('active'):
                # Create future holiday timesheets
                self_company._create_future_public_holidays_timesheets(self)
            else:
                # Delete future holiday timesheets
                self_company._delete_future_public_holidays_timesheets()
        elif 'resource_calendar_id' in vals:
            # Update future holiday timesheets
            self_company._delete_future_public_holidays_timesheets()
            self_company._create_future_public_holidays_timesheets(self)
        return result

    def _delete_future_public_holidays_timesheets(self):
        future_timesheets = self.env['account.analytic.line'].sudo().search([('global_leave_id', '!=', False), ('date', '>=', fields.date.today()), ('employee_id', 'in', self.ids)])
        future_timesheets.write({'global_leave_id': False})
        future_timesheets.unlink()

    def _create_future_public_holidays_timesheets(self, employees):
        lines_vals = []
        for employee in employees:
            if not employee.active:
                continue
            # First we look for the global time off that are already planned after today
            global_leaves = employee.resource_calendar_id.global_leave_ids.filtered(lambda l: l.date_from >= fields.Datetime.today())
            work_hours_data = global_leaves._work_time_per_day()
            for global_time_off in global_leaves:
                for index, (day_date, work_hours_count) in enumerate(work_hours_data[global_time_off.id]):
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

    def _default_project_id(self):
        company = self.company_id if self.company_id else self.env.company
        return company.internal_project_id.id

    def _default_task_id(self):
        company = self.company_id if self.company_id else self.env.company
        return company.leave_timesheet_task_id.id

    timesheet_generate = fields.Boolean(
        'Generate Timesheet', compute='_compute_timesheet_generate', store=True, readonly=False,
        help="If checked, when validating a time off, timesheet will be generated in the Vacation Project of the company.")
    timesheet_project_id = fields.Many2one('project.project', string="Project", default=_default_project_id, domain="[('company_id', '=', company_id)]")
    timesheet_task_id = fields.Many2one(
        'project.task', string="Task", compute='_compute_timesheet_task_id',
        store=True, readonly=False, default=_default_task_id,
        domain="[('project_id', '=', timesheet_project_id),"
                "('project_id', '!=', False),"
                "('company_id', '=', company_id)]")

    @api.depends('timesheet_task_id', 'timesheet_project_id')
    def _compute_timesheet_generate(self):
        for leave_type in self:
            leave_type.timesheet_generate = leave_type.timesheet_task_id and leave_type.timesheet_project_id

    @api.depends('timesheet_project_id')
    def _compute_timesheet_task_id(self):
        for leave_type in self:
            company = leave_type.company_id if leave_type.company_id else self.env.company
            default_task_id = company.leave_timesheet_task_id

            if default_task_id and default_task_id.project_id == leave_type.timesheet_project_id:
                leave_type.timesheet_task_id = default_task_id
            else:
                leave_type.timesheet_task_id = False

    @api.constrains('timesheet_generate', 'timesheet_project_id', 'timesheet_task_id')
    def _check_timesheet_generate(self):
        for holiday_status in self:
            if holiday_status.timesheet_generate:
                if not holiday_status.timesheet_project_id or not holiday_status.timesheet_task_id:
                    raise ValidationError(_("Both the internal project and task are required to "
                    "generate a timesheet for the time off %s. If you don't want a timesheet, you should "
                    "leave the internal project and task empty.") % (holiday_status.name))


class Holidays(models.Model):
    _inherit = "hr.leave"

    timesheet_ids = fields.One2many('account.analytic.line', 'holiday_id', string="Analytic Lines")

    def _validate_leave_request(self):
        """ Timesheet will be generated on leave validation only if a timesheet_project_id and a
            timesheet_task_id are set on the corresponding leave type. The generated timesheet will
            be attached to this project/task.
        """
        holidays = self.filtered(
            lambda l: l.holiday_type == 'employee' and
            l.holiday_status_id.timesheet_project_id and
            l.holiday_status_id.timesheet_task_id and
            l.holiday_status_id.timesheet_project_id.sudo().company_id == (l.holiday_status_id.company_id or self.env.company))

        # Unlink previous timesheets do avoid doublon (shouldn't happen on the interface but meh)
        old_timesheets = holidays.sudo().timesheet_ids
        if old_timesheets:
            old_timesheets.holiday_id = False
            old_timesheets.unlink()

        # create the timesheet on the vacation project
        holidays._timesheet_create_lines()

        return super()._validate_leave_request()

    def _timesheet_create_lines(self):
        vals_list = []
        for leave in self:
            if not leave.employee_id:
                continue
            work_hours_data = leave.employee_id.list_work_time_per_day(
                leave.date_from,
                leave.date_to)
            for index, (day_date, work_hours_count) in enumerate(work_hours_data):
                vals_list.append(leave._timesheet_prepare_line_values(index, work_hours_data, day_date, work_hours_count))
        return self.env['account.analytic.line'].sudo().create(vals_list)

    def _timesheet_prepare_line_values(self, index, work_hours_data, day_date, work_hours_count):
        self.ensure_one()
        return {
            'name': _("Time Off (%s/%s)", index + 1, len(work_hours_data)),
            'project_id': self.holiday_status_id.timesheet_project_id.id,
            'task_id': self.holiday_status_id.timesheet_task_id.id,
            'account_id': self.holiday_status_id.timesheet_project_id.analytic_account_id.id,
            'unit_amount': work_hours_count,
            'user_id': self.employee_id.user_id.id,
            'date': day_date,
            'holiday_id': self.id,
            'employee_id': self.employee_id.id,
            'company_id': self.holiday_status_id.timesheet_task_id.company_id.id or self.holiday_status_id.timesheet_project_id.company_id.id,
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
            ("calendar_id", "!=", False),
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

```

## File: models\project_task.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _

class Task(models.Model):
    _inherit = 'project.task'

    leave_types_count = fields.Integer(compute='_compute_leave_types_count')
    is_timeoff_task = fields.Boolean("Is Time off Task", compute="_compute_is_timeoff_task", search="_search_is_timeoff_task")

    def _compute_leave_types_count(self):
        time_off_type_read_group = self.env['hr.leave.type']._read_group(
            [('timesheet_task_id', 'in', self.ids)],
            ['timesheet_task_id'],
            ['timesheet_task_id'],
        )
        time_off_type_count_per_task = {res['timesheet_task_id'][0]: res['timesheet_task_id_count'] for res in time_off_type_read_group}
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
            ['timesheet_task_ids:array_agg(timesheet_task_id)'],
            [],
        )
        timeoff_task_ids = leave_type_read_group[0]['timesheet_task_ids'] if leave_type_read_group[0]['timesheet_task_ids'] else []
        if self.env.company.leave_timesheet_task_id:
            timeoff_task_ids.append(self.env.company.leave_timesheet_task_id.id)
        if operator == '!=':
            value = not value
        return [('id', 'in' if value else 'not in', timeoff_task_ids)]

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

    def _work_time_per_day(self):
        """ Get work time per day based on the calendar and its attendances

            1) Gets all calendars with their characteristics (i.e.
                (a) the leaves in it,
                (b) the resources which have a leave,
                (c) the oldest and
                (d) the latest leave dates
               ) for leaves in self.
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
        leaves_read_group = self.env['resource.calendar.leaves']._read_group(
            [('id', 'in', self.ids)],
            ['calendar_id', 'ids:array_agg(id)', 'resource_ids:array_agg(resource_id)', 'min_date_from:min(date_from)', 'max_date_to:max(date_to)'],
            ['calendar_id'],
        )
        # dict of keys: calendar_id
        #   and values : { 'date_from': datetime, 'date_to': datetime, resources: self.env['resource.resource'] }
        cal_attendance_intervals_dict = {
            res['calendar_id'][0]: {
                'date_from': utc.localize(res['min_date_from']),
                'date_to': utc.localize(res['max_date_to']),
                'resources': self.env['resource.resource'].browse(res['resource_ids'] if res['resource_ids'] and res['resource_ids'][0] else []),
                'leaves': self.env['resource.calendar.leaves'].browse(res['ids']),
            } for res in leaves_read_group
        }
        # to easily find the calendar with its id.
        calendars_dict = {calendar.id: calendar for calendar in self.calendar_id}

        # dict of keys: leave.id
        #   and values: a dict of keys: date
        #                   and values: number of days
        results = defaultdict(lambda: defaultdict(float))
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
                        results[leave.id][tmp_start.date()] += (tmp_end - tmp_start).total_seconds() / 3600
                results[leave.id] = sorted(results[leave.id].items())
        return results

    def _timesheet_create_lines(self):
        """ Create timesheet leaves for each employee using the same calendar containing in self.calendar_id

            If the employee has already a time off in the same day then no timesheet should be created.
        """
        work_hours_data = self._work_time_per_day()
        employees_groups = self.env['hr.employee']._read_group(
            [('resource_calendar_id', 'in', self.calendar_id.ids), ('company_id', 'in', self.env.companies.ids)],
            ['resource_calendar_id', 'ids:array_agg(id)'],
            ['resource_calendar_id'])
        mapped_employee = {
            employee['resource_calendar_id'][0]: self.env['hr.employee'].browse(employee['ids'])
            for employee in employees_groups
        }
        employee_ids_set = set()
        employee_ids_set.update(*[line['ids'] for line in employees_groups])
        min_date = max_date = None
        for values in work_hours_data.values():
            for d, dummy in values:
                if not min_date and not max_date:
                    min_date = max_date = d
                elif d < min_date:
                    min_date = d
                elif d > max_date:
                    max_date = d

        holidays_read_group = self.env['hr.leave']._read_group([
            ('employee_id', 'in', list(employee_ids_set)),
            ('date_from', '<=', max_date),
            ('date_to', '>=', min_date),
            ('state', '=', 'validate'),
        ], ['date_from_list:array_agg(date_from)', 'date_to_list:array_agg(date_to)', 'employee_id'], ['employee_id'])
        holidays_by_employee = {
            line['employee_id'][0]: [
                (date_from.date(), date_to.date()) for date_from, date_to in zip(line['date_from_list'], line['date_to_list'])
            ] for line in holidays_read_group
        }
        vals_list = []
        for leave in self:
            for employee in mapped_employee.get(leave.calendar_id.id, self.env['hr.employee']):
                holidays = holidays_by_employee.get(employee.id)
                work_hours_list = work_hours_data[leave.id]
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

    def _generate_public_time_off_timesheets(self, employees):
        timesheet_vals_list = []
        work_hours_data = self._work_time_per_day()
        timesheet_read_group = self.env['account.analytic.line']._read_group(
            [('global_leave_id', 'in', self.ids), ('employee_id', 'in', employees.ids)],
            ['date:array_agg'],
            ['employee_id']
        )
        timesheet_dates_per_employee_id = {
            res['employee_id'][0]: res['date']
            for res in timesheet_read_group
        }
        for leave in self:
            for employee in employees:
                if employee.resource_calendar_id != leave.calendar_id:
                    continue
                work_hours_list = work_hours_data[leave.id]
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
        results_with_leave_timesheet = results.filtered(lambda r: not r.resource_id.id and r.calendar_id.company_id.internal_project_id and r.calendar_id.company_id.leave_timesheet_task_id)
        results_with_leave_timesheet and results_with_leave_timesheet._timesheet_create_lines()
        return results

    def write(self, vals):
        date_from, date_to, calendar_id = vals.get('date_from'), vals.get('date_to'), vals.get('calendar_id')
        global_time_off_updated = self.env['resource.calendar.leaves']
        if date_from or date_to or 'calendar_id' in vals:
            global_time_off_updated = self.filtered(lambda r: (date_from is not None and r.date_from != date_from) or (date_to is not None and r.date_to != date_to) or (calendar_id is not None and r.calendar_id.id != calendar_id))
            timesheets = global_time_off_updated.sudo().timesheet_ids
            if timesheets:
                timesheets.write({'global_leave_id': False})
                timesheets.unlink()
        result = super(ResourceCalendarLeaves, self).write(vals)
        if global_time_off_updated:
            global_time_offs_with_leave_timesheet = global_time_off_updated.filtered(lambda r: not r.resource_id and r.calendar_id.company_id.internal_project_id and r.calendar_id.company_id.leave_timesheet_task_id)
            global_time_offs_with_leave_timesheet.sudo()._timesheet_create_lines()
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
                <group name="timesheet" groups="base.group_no_one" string="Timesheet">
                    <group>
                        <field name="timesheet_project_id" context="{'active_test': False}"/>
                        <field name="company_id" invisible="1"/>
                        <field name="timesheet_task_id" context="{'active_test': False, 'default_project_id': timesheet_project_id}" attrs="
                            {'invisible': [('timesheet_project_id', '=', False)], 'required': [('timesheet_project_id', '!=', False)]}"/>
                        <field name="timesheet_generate" invisible="1"/>
                    </group>
                </group>

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
            <xpath expr="//div[@id='module_project_timesheet_holidays']" position="replace">
                <div attrs="{'invisible': [('module_project_timesheet_holidays','=',False)]}">
                    <div class="row mt16">
                        <div class="w-100">
                            <label string="Project" for="internal_project_id" class="col-2 col-lg-3"/>
                            <field name="internal_project_id" context="{'active_test': False}" class="oe_inline ml16"/>
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

