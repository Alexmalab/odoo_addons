# Odoo Module: project_timesheet_holidays

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


def post_init(cr, registry):
    """ Set the timesheet project and task on existing leave type. Do it in post_init to
        be sure the internal project/task of res.company are set. (Since timesheet_generate field
        is true by default, those 2 fields are required on the leave type).
    """
    from odoo import api, SUPERUSER_ID

    env = api.Environment(cr, SUPERUSER_ID, {})
    for hr_leave_type in env['hr.leave.type'].search([('timesheet_generate', '=', True), ('timesheet_project_id', '=', False)]):
        company = hr_leave_type.company_id or env.company
        hr_leave_type.write({
            'timesheet_project_id': company.leave_timesheet_project_id.id,
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
    'demo': [],
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


class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    holiday_id = fields.Many2one("hr.leave", string='Leave Request')

    def unlink(self):
        if any(line.holiday_id for line in self):
            raise UserError(_('You cannot delete timesheet lines attached to a leaves. Please cancel the leaves instead.'))
        return super(AccountAnalyticLine, self).unlink()

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
        return company.leave_timesheet_project_id.id

    def _default_task_id(self):
        company = self.company_id if self.company_id else self.env.company
        return company.leave_timesheet_task_id.id

    timesheet_generate = fields.Boolean(
        'Generate Timesheet', compute='_compute_timesheet_generate', store=True, readonly=False,
        help="If checked, when validating a time off, timesheet will be generated in the Vacation Project of the company.")
    timesheet_project_id = fields.Many2one('project.project', string="Project", default=_default_project_id, domain="[('company_id', '=', company_id)]", help="The project will contain the timesheet generated when a time off is validated.")
    timesheet_task_id = fields.Many2one(
        'project.task', string="Task for timesheet", compute='_compute_timesheet_task_id',
        store=True, readonly=False, default=_default_task_id,
        domain="[('project_id', '=', timesheet_project_id),"
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
        # create the timesheet on the vacation project
        for holiday in self.filtered(
                lambda request: request.holiday_type == 'employee' and
                                request.holiday_status_id.timesheet_project_id and
                                request.holiday_status_id.timesheet_task_id):
            holiday._timesheet_create_lines()

        return super(Holidays, self)._validate_leave_request()

    def _timesheet_create_lines(self):
        self.ensure_one()
        vals_list = []
        work_hours_data = self.employee_id.list_work_time_per_day(
            self.date_from,
            self.date_to,
        )
        for index, (day_date, work_hours_count) in enumerate(work_hours_data):
            vals_list.append(self._timesheet_prepare_line_values(index, work_hours_data, day_date, work_hours_count))
        timesheets = self.env['account.analytic.line'].sudo().create(vals_list)
        return timesheets

    def _timesheet_prepare_line_values(self, index, work_hours_data, day_date, work_hours_count):
        self.ensure_one()
        return {
            'name': "%s (%s/%s)" % (self.holiday_status_id.name or '', index + 1, len(work_hours_data)),
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

    def action_refuse(self):
        """ Remove the timesheets linked to the refused holidays """
        result = super(Holidays, self).action_refuse()
        timesheets = self.sudo().mapped('timesheet_ids')
        timesheets.write({'holiday_id': False})
        timesheets.unlink()
        return result

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class Company(models.Model):
    _inherit = 'res.company'

    leave_timesheet_project_id = fields.Many2one(
        'project.project', string="Internal Project",
        help="Default project value for timesheet generated from time off type.")
    leave_timesheet_task_id = fields.Many2one(
        'project.task', string="Time Off Task",
        domain="[('project_id', '=', leave_timesheet_project_id)]")

    @api.constrains('leave_timesheet_project_id')
    def _check_leave_timesheet_project_id_company(self):
        for company in self:
            if company.leave_timesheet_project_id:
                if company.leave_timesheet_project_id.sudo().company_id != company:
                    raise ValidationError(_('The Internal Project of a company should be in that company.'))

    def init(self):
        for company in self.search([('leave_timesheet_project_id', '=', False)]):
            company = company.with_company(company)
            project = company.env['project.project'].search([
                ('name', '=', _('Internal')),
                ('allow_timesheets', '=', True),
                ('company_id', '=', company.id),
            ], limit=1)
            if not project:
                project = company.env['project.project'].create({
                    'name': _('Internal'),
                    'allow_timesheets': True,
                    'company_id': company.id,
                })
            company.write({
                'leave_timesheet_project_id': project.id,
            })
            if not company.leave_timesheet_task_id:
                task = company.env['project.task'].create({
                    'name': _('Time Off'),
                    'project_id': company.leave_timesheet_project_id.id,
                    'active': False,
                    'company_id': company.id,
                })
                company.write({
                    'leave_timesheet_task_id': task.id,
                })

    def _create_internal_project_task(self):
        projects = super()._create_internal_project_task()
        for project in projects:
            company = project.company_id
            company = company.with_company(company)
            if not company.leave_timesheet_project_id:
                company.write({
                    'leave_timesheet_project_id': project.id,
                })
            if not company.leave_timesheet_task_id:
                task = company.env['project.task'].sudo().create({
                    'name': _('Time Off'),
                    'project_id': company.leave_timesheet_project_id.id,
                    'active': False,
                    'company_id': company.id,
                })
                company.write({
                    'leave_timesheet_task_id': task.id,
                })

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    leave_timesheet_project_id = fields.Many2one(
        related='company_id.leave_timesheet_project_id', required=True, string="Internal Project",
        domain="[('company_id', '=', company_id)]", readonly=False)
    leave_timesheet_task_id = fields.Many2one(
        related='company_id.leave_timesheet_task_id', string="Time Off Task", readonly=False,
        domain="[('company_id', '=', company_id), ('project_id', '=?', leave_timesheet_project_id)]")

    @api.onchange('leave_timesheet_project_id')
    def _onchange_timesheet_project_id(self):
        if self.leave_timesheet_project_id != self.leave_timesheet_task_id.project_id:
            self.leave_timesheet_task_id = False

    @api.onchange('leave_timesheet_task_id')
    def _onchange_timesheet_task_id(self):
        if self.leave_timesheet_task_id:
            self.leave_timesheet_project_id = self.leave_timesheet_task_id.project_id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company # has to be before hr_holidays to create needed columns on res.company
from . import account_analytic
from . import hr_holidays
from . import res_config_settings

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
            <xpath expr="//group[@name='calendar']" position="after">
                <group name="timesheet" string="Timesheet" groups="base.group_no_one">
                    <field name="timesheet_project_id" context="{'active_test': False}"/>
                    <field name="timesheet_task_id" context="{'active_test': False}"/>
                    <field name="timesheet_generate" invisible="1"/>
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
                            <label string="Project" for="leave_timesheet_project_id" class="col-3 col-lg-3 o_light_label"/>
                            <field name="leave_timesheet_project_id" context="{'active_test': False}" class="oe_inline"/>
                        </div>
                        <div class="w-100">
                            <label string="Task" for="leave_timesheet_task_id" class="col-3 col-lg-3 o_light_label"/>
                            <field name="leave_timesheet_task_id" context="{'active_test': False, 'default_project_id': leave_timesheet_project_id}" class="oe_inline"/>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

