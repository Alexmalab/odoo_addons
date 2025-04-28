# Odoo Module: hr_timesheet

Category: Operations/Timesheets

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Task Logs',
    'version': '1.0',
    'category': 'Operations/Timesheets',
    'sequence': 23,
    'summary': 'Track employee time on tasks',
    'description': """
This module implements a timesheet system.
==========================================

Each employee can encode and track their time spent on the different projects.

Lots of reporting on time and employee tracking are provided.

It is completely integrated with the cost accounting module. It allows you to set
up a management by affair.
    """,
    'website': 'https://www.odoo.com/page/timesheet-mobile-app',
    'depends': ['hr', 'analytic', 'project', 'uom'],
    'data': [
        'security/hr_timesheet_security.xml',
        'security/ir.model.access.csv',
        'views/assets.xml',
        'views/hr_timesheet_views.xml',
        'views/res_config_settings_views.xml',
        'views/project_views.xml',
        'views/project_portal_templates.xml',
        'views/hr_timesheet_portal_templates.xml',
        'report/hr_timesheet_report_view.xml',
        'report/report_timesheet_templates.xml',
        'views/hr_views.xml',
        'data/hr_timesheet_data.xml',
    ],
    'demo': [
        'data/hr_timesheet_demo.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import OrderedDict
from dateutil.relativedelta import relativedelta
from operator import itemgetter

from odoo import fields, http, _
from odoo.http import request
from odoo.tools import date_utils, groupby as groupbyelem
from odoo.osv.expression import AND

from odoo.addons.portal.controllers.portal import CustomerPortal, pager as portal_pager


class TimesheetCustomerPortal(CustomerPortal):

    def _prepare_home_portal_values(self):
        values = super(TimesheetCustomerPortal, self)._prepare_home_portal_values()
        domain = request.env['account.analytic.line']._timesheet_get_portal_domain()
        values['timesheet_count'] = request.env['account.analytic.line'].sudo().search_count(domain)
        return values

    @http.route(['/my/timesheets', '/my/timesheets/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_timesheets(self, page=1, sortby=None, filterby=None, search=None, search_in='all', groupby='project', **kw):
        Timesheet_sudo = request.env['account.analytic.line'].sudo()
        values = self._prepare_portal_layout_values()
        domain = request.env['account.analytic.line']._timesheet_get_portal_domain()

        searchbar_sortings = {
            'date': {'label': _('Newest'), 'order': 'date desc'},
            'name': {'label': _('Name'), 'order': 'name'},
        }

        searchbar_inputs = {
            'all': {'input': 'all', 'label': _('Search in All')},
        }

        searchbar_groupby = {
            'none': {'input': 'none', 'label': _('None')},
            'project': {'input': 'project', 'label': _('Project')},
        }

        today = fields.Date.today()
        quarter_start, quarter_end = date_utils.get_quarter(today)
        last_week = today + relativedelta(weeks=-1)
        last_month = today + relativedelta(months=-1)
        last_year = today + relativedelta(years=-1)

        searchbar_filters = {
            'all': {'label': _('All'), 'domain': []},
            'today': {'label': _('Today'), 'domain': [("date", "=", today)]},
            'week': {'label': _('This week'), 'domain': [('date', '>=', date_utils.start_of(today, "week")), ('date', '<=', date_utils.end_of(today, 'week'))]},
            'month': {'label': _('This month'), 'domain': [('date', '>=', date_utils.start_of(today, 'month')), ('date', '<=', date_utils.end_of(today, 'month'))]},
            'year': {'label': _('This year'), 'domain': [('date', '>=', date_utils.start_of(today, 'year')), ('date', '<=', date_utils.end_of(today, 'year'))]},
            'quarter': {'label': _('This Quarter'), 'domain': [('date', '>=', quarter_start), ('date', '<=', quarter_end)]},
            'last_week': {'label': _('Last week'), 'domain': [('date', '>=', date_utils.start_of(last_week, "week")), ('date', '<=', date_utils.end_of(last_week, 'week'))]},
            'last_month': {'label': _('Last month'), 'domain': [('date', '>=', date_utils.start_of(last_month, 'month')), ('date', '<=', date_utils.end_of(last_month, 'month'))]},
            'last_year': {'label': _('Last year'), 'domain': [('date', '>=', date_utils.start_of(last_year, 'year')), ('date', '<=', date_utils.end_of(last_year, 'year'))]},
        }
        # default sort by value
        if not sortby:
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']
        # default filter by value
        if not filterby:
            filterby = 'all'
        domain = AND([domain, searchbar_filters[filterby]['domain']])

        if search and search_in:
            domain = AND([domain, [('name', 'ilike', search)]])

        timesheet_count = Timesheet_sudo.search_count(domain)
        # pager
        pager = portal_pager(
            url="/my/timesheets",
            url_args={'sortby': sortby, 'search_in': search_in, 'search': search, 'filterby': filterby},
            total=timesheet_count,
            page=page,
            step=self._items_per_page
        )

        if groupby == 'project':
            order = "project_id, %s" % order
        timesheets = Timesheet_sudo.search(domain, order=order, limit=self._items_per_page, offset=pager['offset'])
        if groupby == 'project':
            grouped_timesheets = [Timesheet_sudo.concat(*g) for k, g in groupbyelem(timesheets, itemgetter('project_id'))]
        else:
            grouped_timesheets = [timesheets]

        values.update({
            'timesheets': timesheets,
            'grouped_timesheets': grouped_timesheets,
            'page_name': 'timesheet',
            'default_url': '/my/timesheets',
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'search_in': search_in,
            'sortby': sortby,
            'groupby': groupby,
            'searchbar_inputs': searchbar_inputs,
            'searchbar_groupby': searchbar_groupby,
            'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
            'filterby': filterby,
        })
        return request.render("hr_timesheet.portal_my_timesheets", values)

```

## File: controllers\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request
from odoo.osv import expression

from odoo.addons.project.controllers.portal import CustomerPortal


class ProjectCustomerPortal(CustomerPortal):

    def _task_get_page_view_values(self, task, access_token, **kwargs):
        values = super(ProjectCustomerPortal, self)._task_get_page_view_values(task, access_token, **kwargs)
        domain = request.env['account.analytic.line']._timesheet_get_portal_domain()
        domain = expression.AND([domain, [('task_id', '=', task.id)]])
        timesheets = request.env['account.analytic.line'].sudo().search(domain)
        values['timesheets'] = timesheets
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal
from . import project

```

## File: data\hr_timesheet_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

	<!-- For a perfect onboarding, we want the tour of project to be consumed
	only by admin when installing hr_timesheet, to avoid confusing -->
    <record id="web_tour_project_consumed_by_admin" model="web_tour.tour">
        <field name="name">project_tour</field>
        <field name="user_id" ref='base.user_admin'/>
    </record>

    <!-- Set the JS widget -->
    <record id="uom.product_uom_day" model="uom.uom">
        <field name="timesheet_widget">float_toggle</field>
    </record>

    <record id="uom.product_uom_hour" model="uom.uom">
        <field name="timesheet_widget">float_time</field>
    </record>

    <!-- Force Analytic account creation for projects allowing timesheet (default is True) -->
    <function
        model="project.project"
        name="_init_data_analytic_account"
        eval="[]"/>

</odoo>

```

## File: data\hr_timesheet_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Employee -->
    <record id="hr.employee_admin" model="hr.employee">
        <field name="timesheet_cost">100</field>
    </record>

    <record id="hr.employee_qdp" model="hr.employee">
        <field name="timesheet_cost">75</field>
        <field name="parent_id" ref="hr.employee_admin"/>
    </record>

    <!-- Projects -->
    <record id="account_analytic_account_project_2" model="account.analytic.account">
        <field name="name">Research &amp; Development Project</field>
        <field name="code">RD</field>
        <field name="active" eval="True"/>
    </record>

    <record id="project.project_project_2" model="project.project">
        <field name="analytic_account_id" ref="account_analytic_account_project_2"/>
    </record>

    <record id="working_hours_requirements" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="user_id" ref='base.user_admin'/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2.00</field>
        <field name="project_id" ref='project.project_project_2'/>
        <field name="amount">-60.00</field>
    </record>

    <record id="working_hours_design" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="user_id" ref='base.user_admin'/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1.00</field>
        <field name="project_id" ref='project.project_project_2'/>
        <field name="amount">-30.00</field>
    </record>

    <record id="working_hours_coding" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="user_id" ref='base.user_admin'/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3.00</field>
        <field name="project_id" ref='project.project_project_2'/>
        <field name="amount">-90.00</field>
    </record>

    <record id="working_hours_testing" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="user_id" ref='base.user_admin'/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1.00</field>
        <field name="project_id" ref='project.project_project_2'/>
        <field name="amount">-30.00</field>
    </record>

    <record id="working_hours_maintenance" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="user_id" ref='base.user_admin'/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1.00</field>
        <field name="project_id" ref='project.project_project_2'/>
        <field name="amount">-30.00</field>
    </record>

</odoo>

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    timesheet_cost = fields.Monetary('Timesheet Cost', currency_field='currency_id',
    	groups="hr.group_hr_user", default=0.0)
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id', readonly=True)

```

## File: models\hr_timesheet.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from lxml import etree
import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError


class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    @api.model
    def default_get(self, field_list):
        result = super(AccountAnalyticLine, self).default_get(field_list)
        if 'encoding_uom_id' in field_list:
            result['encoding_uom_id'] = self.env.company.timesheet_encode_uom_id.id
        if not self.env.context.get('default_employee_id') and 'employee_id' in field_list and result.get('user_id'):
            result['employee_id'] = self.env['hr.employee'].search([('user_id', '=', result['user_id'])], limit=1).id
        return result

    task_id = fields.Many2one('project.task', 'Task', index=True, domain="[('company_id', '=', company_id)]")
    project_id = fields.Many2one('project.project', 'Project', domain=[('allow_timesheets', '=', True)])

    employee_id = fields.Many2one('hr.employee', "Employee")
    department_id = fields.Many2one('hr.department', "Department", compute='_compute_department_id', store=True, compute_sudo=True)
    encoding_uom_id = fields.Many2one('uom.uom', compute='_compute_encoding_uom_id')

    def _compute_encoding_uom_id(self):
        for analytic_line in self:
            analytic_line.encoding_uom_id = self.env.company.timesheet_encode_uom_id

    @api.constrains('task_id', 'project_id')
    def _check_task_project(self):
        for line in self:
            if line.task_id and line.project_id and line.task_id.project_id != line.project_id:
                raise ValidationError(_(
                    "The project and the task's project are inconsistent. " +
                    "The selected task must be in the selected project."
                ))

    @api.onchange('project_id')
    def onchange_project_id(self):
        # force domain on task when project is set
        if self.project_id:
            if self.project_id != self.task_id.project_id:
                # reset task when changing project
                self.task_id = False
            return {'domain': {
                'task_id': [('project_id', '=', self.project_id.id)]
            }}
        return {'domain': {
            'task_id': [('project_id.allow_timesheets', '=', True)]
        }}


    @api.onchange('task_id')
    def _onchange_task_id(self):
        if not self.project_id:
            self.project_id = self.task_id.project_id

    @api.onchange('employee_id')
    def _onchange_employee_id(self):
        if self.employee_id:
            self.user_id = self.employee_id.user_id
        else:
            self.user_id = self._default_user()

    @api.depends('employee_id')
    def _compute_department_id(self):
        for line in self:
            line.department_id = line.employee_id.department_id

    # ----------------------------------------------------
    # ORM overrides
    # ----------------------------------------------------

    @api.model
    def create(self, values):
        # compute employee only for timesheet lines, makes no sense for other lines
        if not values.get('employee_id') and values.get('project_id'):
            if values.get('user_id'):
                ts_user_id = values['user_id']
            else:
                ts_user_id = self._default_user()
            values['employee_id'] = self.env['hr.employee'].search([('user_id', '=', ts_user_id)], limit=1).id

        values = self._timesheet_preprocess(values)
        result = super(AccountAnalyticLine, self).create(values)
        if result.project_id:  # applied only for timesheet
            result._timesheet_postprocess(values)
        return result

    def write(self, values):
        values = self._timesheet_preprocess(values)
        result = super(AccountAnalyticLine, self).write(values)
        # applied only for timesheet
        self.filtered(lambda t: t.project_id)._timesheet_postprocess(values)
        return result

    @api.model
    def fields_view_get(self, view_id=None, view_type='form', toolbar=False, submenu=False):
        """ Set the correct label for `unit_amount`, depending on company UoM """
        result = super(AccountAnalyticLine, self).fields_view_get(view_id=view_id, view_type=view_type, toolbar=toolbar, submenu=submenu)
        result['arch'] = self._apply_timesheet_label(result['arch'])
        return result

    @api.model
    def _apply_timesheet_label(self, view_arch):
        doc = etree.XML(view_arch)
        encoding_uom = self.env.company.timesheet_encode_uom_id
        # Here, we select only the unit_amount field having no string set to give priority to
        # custom inheretied view stored in database. Even if normally, no xpath can be done on
        # 'string' attribute.
        for node in doc.xpath("//field[@name='unit_amount'][@widget='timesheet_uom'][not(@string)]"):
            node.set('string', _('Duration (%s)') % (re.sub(r'[\(\)]', '', encoding_uom.name or '')))
        return etree.tostring(doc, encoding='unicode')

    # ----------------------------------------------------
    # Business Methods
    # ----------------------------------------------------

    def _timesheet_get_portal_domain(self):
        return ['&',
                ('task_id.project_id.privacy_visibility', '=', 'portal'),
                '|',
                ('task_id.project_id.message_partner_ids', 'child_of', [self.env.user.partner_id.commercial_partner_id.id]),
                ('task_id.message_partner_ids', 'child_of', [self.env.user.partner_id.commercial_partner_id.id])]

    def _timesheet_preprocess(self, vals):
        """ Deduce other field values from the one given.
            Overrride this to compute on the fly some field that can not be computed fields.
            :param values: dict values for `create`or `write`.
        """
        # project implies analytic account
        if vals.get('project_id') and not vals.get('account_id'):
            project = self.env['project.project'].browse(vals.get('project_id'))
            vals['account_id'] = project.analytic_account_id.id
            vals['company_id'] = project.analytic_account_id.company_id.id or project.company_id.id
            if not project.analytic_account_id.active:
                raise UserError(_('The project you are timesheeting on is not linked to an active analytic account. Set one on the project configuration.'))
        # employee implies user
        if vals.get('employee_id') and not vals.get('user_id'):
            employee = self.env['hr.employee'].browse(vals['employee_id'])
            vals['user_id'] = employee.user_id.id
        # force customer partner, from the task or the project
        if (vals.get('project_id') or vals.get('task_id')) and not vals.get('partner_id'):
            partner_id = False
            if vals.get('task_id'):
                partner_id = self.env['project.task'].browse(vals['task_id']).partner_id.id
            else:
                partner_id = self.env['project.project'].browse(vals['project_id']).partner_id.id
            if partner_id:
                vals['partner_id'] = partner_id
        # set timesheet UoM from the AA company (AA implies uom)
        if 'product_uom_id' not in vals and all([v in vals for v in ['account_id', 'project_id']]):  # project_id required to check this is timesheet flow
            analytic_account = self.env['account.analytic.account'].sudo().browse(vals['account_id'])
            vals['product_uom_id'] = analytic_account.company_id.project_time_mode_id.id
        return vals

    def _timesheet_postprocess(self, values):
        """ Hook to update record one by one according to the values of a `write` or a `create`. """
        sudo_self = self.sudo()  # this creates only one env for all operation that required sudo() in `_timesheet_postprocess_values`override
        values_to_write = self._timesheet_postprocess_values(values)
        for timesheet in sudo_self:
            if values_to_write[timesheet.id]:
                timesheet.write(values_to_write[timesheet.id])
        return values

    def _timesheet_postprocess_values(self, values):
        """ Get the addionnal values to write on record
            :param dict values: values for the model's fields, as a dictionary::
                {'field_name': field_value, ...}
            :return: a dictionary mapping each record id to its corresponding
                dictionnary values to write (may be empty).
        """
        result = {id_: {} for id_ in self.ids}
        sudo_self = self.sudo()  # this creates only one env for all operation that required sudo()
        # (re)compute the amount (depending on unit_amount, employee_id for the cost, and account_id for currency)
        if any([field_name in values for field_name in ['unit_amount', 'employee_id', 'account_id']]):
            for timesheet in sudo_self:
                cost = timesheet.employee_id.timesheet_cost or 0.0
                amount = -timesheet.unit_amount * cost
                amount_converted = timesheet.employee_id.currency_id._convert(
                    amount, timesheet.account_id.currency_id or timesheet.currency_id, self.env.company, timesheet.date)
                result[timesheet.id].update({
                    'amount': amount_converted,
                })
        return result

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        """ The widget 'timesheet_uom' needs to know which UoM conversion factor and which javascript
            widget to apply, depending on th ecurrent company.
        """
        result = super(Http, self).session_info()
        if self.env.user.has_group('base.group_user'):
            company = self.env.company
            encoding_uom = company.timesheet_encode_uom_id

            result['timesheet_uom'] = encoding_uom.read(['name', 'rounding', 'timesheet_widget'])[0]
            result['timesheet_uom_factor'] = company.project_time_mode_id._compute_quantity(1.0, encoding_uom, round=False)  # convert encoding uom into stored uom to get conversion factor
        return result

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import UserError, ValidationError


class Project(models.Model):
    _inherit = "project.project"

    allow_timesheets = fields.Boolean("Timesheets", default=True, help="Enable timesheeting on the project.")

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        domain = []
        if self.partner_id:
            domain = [('partner_id', '=', self.partner_id.id)]
        return {'domain': {'analytic_account_id': domain}}

    @api.onchange('analytic_account_id')
    def _onchange_analytic_account(self):
        if not self.analytic_account_id and self._origin:
            self.allow_timesheets = False

    @api.constrains('allow_timesheets', 'analytic_account_id')
    def _check_allow_timesheet(self):
        for project in self:
            if project.allow_timesheets and not project.analytic_account_id:
                raise ValidationError(_('To allow timesheet, your project %s should have an analytic account set.' % (project.name,)))

    @api.model
    def name_create(self, name):
        """ Create a project with name_create should generate analytic account creation """
        values = {
            'name': name,
            'allow_timesheets': True,
        }
        return self.create(values).name_get()[0]

    @api.model
    def create(self, values):
        """ Create an analytic account if project allow timesheet and don't provide one
            Note: create it before calling super() to avoid raising the ValidationError from _check_allow_timesheet
        """
        allow_timesheets = values['allow_timesheets'] if 'allow_timesheets' in values else self.default_get(['allow_timesheets'])['allow_timesheets']
        if allow_timesheets and not values.get('analytic_account_id'):
            analytic_account = self._create_analytic_account_from_values(values)
            values['analytic_account_id'] = analytic_account.id
        return super(Project, self).create(values)

    def write(self, values):
        # create the AA for project still allowing timesheet
        if values.get('allow_timesheets'):
            for project in self:
                if not project.analytic_account_id and not values.get('analytic_account_id'):
                    project._create_analytic_account()
        result = super(Project, self).write(values)
        return result

    def unlink(self):
        for project in self.with_context(active_test=False):
            timesheets = self.env['account.analytic.line'].search([('project_id', '=', project.id)])
            if timesheets:
                raise UserError(_('You cannot delete a project containing timesheets. You can either archive it or first delete all of its timesheets.'))
        return super(Project, self).unlink()

    # ---------------------------------------------------
    #  Business Methods
    # ---------------------------------------------------

    @api.model
    def _init_data_analytic_account(self):
        self.search([('analytic_account_id', '=', False), ('allow_timesheets', '=', True)])._create_analytic_account()


class Task(models.Model):
    _inherit = "project.task"

    analytic_account_active = fields.Boolean("Analytic Account", related='project_id.analytic_account_id.active', readonly=True)
    allow_timesheets = fields.Boolean("Allow timesheets", related='project_id.allow_timesheets', help="Timesheets can be logged on this task.", readonly=True)
    remaining_hours = fields.Float("Remaining Hours", compute='_compute_remaining_hours', store=True, readonly=True, help="Total remaining time, can be re-estimated periodically by the assignee of the task.")
    effective_hours = fields.Float("Hours Spent", compute='_compute_effective_hours', compute_sudo=True, store=True, help="Computed using the sum of the task work done.")
    total_hours_spent = fields.Float("Total Hours", compute='_compute_total_hours_spent', store=True, help="Computed as: Time Spent + Sub-tasks Hours.")
    progress = fields.Float("Progress", compute='_compute_progress_hours', store=True, group_operator="avg", help="Display progress of current task.")
    subtask_effective_hours = fields.Float("Sub-tasks Hours Spent", compute='_compute_subtask_effective_hours', store=True, help="Sum of actually spent hours on the subtask(s)")
    timesheet_ids = fields.One2many('account.analytic.line', 'task_id', 'Timesheets')

    @api.depends('timesheet_ids.unit_amount')
    def _compute_effective_hours(self):
        for task in self:
            task.effective_hours = round(sum(task.timesheet_ids.mapped('unit_amount')), 2)

    @api.depends('effective_hours', 'subtask_effective_hours', 'planned_hours')
    def _compute_progress_hours(self):
        for task in self:
            if (task.planned_hours > 0.0):
                task_total_hours = task.effective_hours + task.subtask_effective_hours
                if task_total_hours > task.planned_hours:
                    task.progress = 100
                else:
                    task.progress = round(100.0 * task_total_hours / task.planned_hours, 2)
            else:
                task.progress = 0.0

    @api.depends('effective_hours', 'subtask_effective_hours', 'planned_hours')
    def _compute_remaining_hours(self):
        for task in self:
            task.remaining_hours = task.planned_hours - task.effective_hours - task.subtask_effective_hours

    @api.depends('effective_hours', 'subtask_effective_hours')
    def _compute_total_hours_spent(self):
        for task in self:
            task.total_hours_spent = task.effective_hours + task.subtask_effective_hours

    @api.depends('child_ids.effective_hours', 'child_ids.subtask_effective_hours')
    def _compute_subtask_effective_hours(self):
        for task in self:
            task.subtask_effective_hours = sum(child_task.effective_hours + child_task.subtask_effective_hours for child_task in task.child_ids)

    # ---------------------------------------------------------
    # ORM
    # ---------------------------------------------------------

    def write(self, values):
        # a timesheet must have an analytic account (and a project)
        if 'project_id' in values and self and not values.get('project_id'):
                raise UserError(_('This task must be part of a project because there are some timesheets linked to it.'))
        return super(Task, self).write(values)

    @api.model
    def _fields_view_get(self, view_id=None, view_type='form', toolbar=False, submenu=False):
        """ Set the correct label for `unit_amount`, depending on company UoM """
        result = super(Task, self)._fields_view_get(view_id=view_id, view_type=view_type, toolbar=toolbar, submenu=submenu)
        result['arch'] = self.env['account.analytic.line']._apply_timesheet_label(result['arch'])
        return result

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    @api.model
    def _default_project_time_mode_id(self):
        uom = self.env.ref('uom.product_uom_hour', raise_if_not_found=False)
        if not uom:
            uom = self.env['uom.uom'].search([('measure_type', '=', 'working_time'), ('uom_type', '=', 'reference')], limit=1)
        if not uom:
            uom = self.env['uom.uom'].search([('measure_type', '=', 'working_time')], limit=1)
        return uom

    @api.model
    def _default_timesheet_encode_uom_id(self):
        uom = self.env.ref('uom.product_uom_hour', raise_if_not_found=False)
        if not uom:
            uom = self.env['uom.uom'].search([('measure_type', '=', 'working_time'), ('uom_type', '=', 'reference')], limit=1)
        if not uom:
            uom = self.env['uom.uom'].search([('measure_type', '=', 'working_time')], limit=1)
        return uom

    project_time_mode_id = fields.Many2one('uom.uom', string='Project Time Unit',
        default=_default_project_time_mode_id, domain=[('measure_type', '=', 'working_time')],
        help="This will set the unit of measure used in projects and tasks.\n"
             "If you use the timesheet linked to projects, don't "
             "forget to setup the right unit of measure in your employees.")
    timesheet_encode_uom_id = fields.Many2one('uom.uom', string="Timesheet Encoding Unit",
        default=_default_timesheet_encode_uom_id, domain=[('measure_type', '=', 'working_time')], required=True,
        help="""This will set the unit of measure used to encode timesheet. This will simply provide tools
        and widgets to help the encoding. All reporting will still be expressed in hours (default value).""")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_project_timesheet_synchro = fields.Boolean("Awesome Timesheet")
    module_project_timesheet_holidays = fields.Boolean("Record Time Off")
    project_time_mode_id = fields.Many2one(
        'uom.uom', related='company_id.project_time_mode_id', string='Project Time Unit', readonly=False,
        help="This will set the unit of measure used in projects and tasks.\n"
             "If you use the timesheet linked to projects, don't "
             "forget to setup the right unit of measure in your employees.")
    timesheet_encode_uom_id = fields.Many2one('uom.uom', string="Encoding Unit",
        related='company_id.timesheet_encode_uom_id', readonly=False,
        help="""This will set the unit of measure used to encode timesheet. This will simply provide tools
        and widgets to help the encoding. All reporting will still be expressed in hours (default value).""")

```

## File: models\uom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Uom(models.Model):
    _inherit = 'uom.uom'

    timesheet_widget = fields.Char("Widget", help="Widget used in the webclient when this unit is the one used to encode timesheets.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee
from . import hr_timesheet
from . import ir_http
from . import res_company
from . import res_config_settings
from . import project
from . import uom

```

## File: report\hr_timesheet_report_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="act_hr_timesheet_report" model="ir.actions.act_window">
            <field name="name">Timesheets By Employee</field>
            <field name="res_model">account.analytic.line</field>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{'search_default_groupby_employee':1,}</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Record a new activity
              </p><p>
                You can register and track your workings hours by project every
                day. Every time spent on a project will become a cost and can be re-invoiced to
                customers if required.
              </p>
            </field>
        </record>

        <record model="ir.actions.act_window.view" id="act_hr_timesheet_report_pivot">
            <field name="sequence" eval="5"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_pivot"/>
            <field name="act_window_id" ref="act_hr_timesheet_report"/>
        </record>

        <record id="timesheet_action_view_report_by_employee_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="6"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_graph"/>
            <field name="act_window_id" ref="act_hr_timesheet_report"/>
        </record>

        <record model="ir.actions.act_window.view" id="act_hr_timesheet_report_tree">
            <field name="sequence" eval="10"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="hr_timesheet.hr_timesheet_line_tree"/>
            <field name="act_window_id" ref="act_hr_timesheet_report"/>
        </record>

        <record model="ir.actions.act_window.view" id="act_hr_timesheet_report_form">
            <field name="sequence" eval="15"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="hr_timesheet.hr_timesheet_line_form"/>
            <field name="act_window_id" ref="act_hr_timesheet_report"/>
        </record>

        <record id="timesheet_action_report_by_project" model="ir.actions.act_window">
            <field name="name">Timesheets By Project</field>
            <field name="res_model">account.analytic.line</field>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{'search_default_groupby_project':1,}</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Record a new activity
              </p><p>
                You can register and track your workings hours by project every
                day. Every time spent on a project will become a cost and can be re-invoiced to
                customers if required.
              </p>
            </field>
        </record>

        <record id="timesheet_action_view_report_by_project_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_pivot"/>
            <field name="act_window_id" ref="timesheet_action_report_by_project"/>
        </record>

        <record id="timesheet_action_view_report_by_project_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="6"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_graph"/>
            <field name="act_window_id" ref="timesheet_action_report_by_project"/>
        </record>

        <record id="timesheet_action_view_report_by_project_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="10"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="hr_timesheet.hr_timesheet_line_tree"/>
            <field name="act_window_id" ref="timesheet_action_report_by_project"/>
        </record>

        <record id="timesheet_action_view_report_by_project_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="15"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="hr_timesheet.hr_timesheet_line_form"/>
            <field name="act_window_id" ref="timesheet_action_report_by_project"/>
        </record>

        <record id="timesheet_action_report_by_task" model="ir.actions.act_window">
            <field name="name">Timesheets By Task</field>
            <field name="res_model">account.analytic.line</field>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{'search_default_groupby_project':1,'search_default_groupby_task':1,}</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Record a new activity
              </p><p>
                You can register and track your workings hours by project every
                day. Every time spent on a project will become a cost and can be re-invoiced to
                customers if required.
              </p>
            </field>
        </record>

        <record id="timesheet_action_view_report_by_task_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_pivot"/>
            <field name="act_window_id" ref="timesheet_action_report_by_task"/>
        </record>

        <record id="timesheet_action_view_report_by_task_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="6"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_graph"/>
            <field name="act_window_id" ref="timesheet_action_report_by_task"/>
        </record>

        <record id="timesheet_action_view_report_by_task_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="10"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="hr_timesheet.hr_timesheet_line_tree"/>
            <field name="act_window_id" ref="timesheet_action_report_by_task"/>
        </record>

        <record id="timesheet_action_view_report_by_task_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="15"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="hr_timesheet.hr_timesheet_line_form"/>
            <field name="act_window_id" ref="timesheet_action_report_by_task"/>
        </record>

        <menuitem id="menu_timesheets_reports"
            name="Reporting"
            parent="timesheet_menu_root"
            groups="hr_timesheet.group_timesheet_manager"
            sequence="99"/>

        <menuitem id="menu_timesheets_reports_timesheet"
            name="Timesheets"
            parent="menu_timesheets_reports"
            groups="hr_timesheet.group_timesheet_manager"
            sequence="10"/>

        <menuitem id="menu_hr_activity_analysis"
            parent="menu_timesheets_reports_timesheet"
            action="act_hr_timesheet_report"
            name="By Employee"
            sequence="10"/>

        <menuitem id="timesheet_menu_report_timesheet_by_project"
            parent="menu_timesheets_reports_timesheet"
            action="timesheet_action_report_by_project"
            name="By Project"
            sequence="15"/>

        <menuitem id="timesheet_menu_report_timesheet_by_task"
            parent="menu_timesheets_reports_timesheet"
            action="timesheet_action_report_by_task"
            name="By Task"
            sequence="20"/>

    </data>
</odoo>

```

## File: report\project_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ReportProjectTaskUser(models.Model):
    _inherit = "report.project.task.user"

    hours_planned = fields.Float('Planned Hours', readonly=True)
    hours_effective = fields.Float('Effective Hours', readonly=True)
    remaining_hours = fields.Float('Remaining Hours', readonly=True)
    progress = fields.Float('Progress', group_operator='avg', readonly=True)

    def _select(self):
        return super(ReportProjectTaskUser, self)._select() + """,
            progress as progress,
            t.effective_hours as hours_effective,
            t.planned_hours - t.effective_hours - t.subtask_effective_hours as remaining_hours,
            planned_hours as hours_planned"""

    def _group_by(self):
        return super(ReportProjectTaskUser, self)._group_by() + """,
            remaining_hours,
            t.effective_hours,
            progress,
            planned_hours
            """

```

## File: report\project_report_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_task_project_user_graph_inherited" model="ir.ui.view">
            <field name="name">report.project.task.user.graph.inherited</field>
            <field name="model">report.project.task.user</field>
            <field name="inherit_id" ref="project.view_task_project_user_graph" />
            <field name="arch" type="xml">
                <graph string="Tasks Analysis" type="bar">
                    <field name="project_id" position="after">
                        <field name="hours_planned" type="measure"/>
                        <field name="remaining_hours" type="measure"/>
                    </field>
                 </graph>
             </field>
        </record>
    </data>
</odoo>

```

## File: report\report_timesheet_templates.xml

```xml
<odoo>
    <template id="report_timesheet">
        <t t-call="web.html_container">
            <t t-call="web.external_layout">
                <t t-set="company" t-value="docs.mapped('project_id')[0].company_id if len(docs.mapped('project_id')) == 1 else docs.env.company"/>
                <t t-set="show_task" t-value="bool(docs.mapped('task_id'))"/>
                <t t-set="show_project" t-value="len(docs.mapped('project_id')) > 1"/>
                <div class="page">
                    <div class="oe_structure"/>
                    <div class="row" style="margin-top:10px;">
                        <div class="col-lg-12">
                            <h2>
                                <span>Timesheet Entries
                                    <t t-if="len(docs.mapped('project_id')) == 1">
                                        for the <t t-esc="docs.mapped('project_id')[0].name"/> Project
                                    </t>
                                </span>
                            </h2>
                        </div>
                    </div>

                    <div class="row" style="margin-top:10px;">
                        <div class="col-lg-12">
                            <table class="table table-sm">
                                <thead>
                                    <tr>
                                        <th><span>Date</span></th>
                                        <th><span>Responsible</span></th>
                                        <th><span>Description</span></th>
                                        <th t-if="show_project">Project</th>
                                        <th t-if="show_task">Task</th>
                                        <th class="text-right"><span>Time</span></th>
                                    </tr>
                               </thead>
                               <tbody>
                                    <tr t-foreach="docs" t-as="l">
                                        <td>
                                           <span t-field="l.date"/>
                                        </td>
                                        <td>
                                           <span t-field="l.user_id.partner_id.name"/>
                                           <span t-if="not l.user_id.partner_id.name" t-field="l.employee_id"/>
                                        </td>
                                        <td >
                                            <span t-field="l.name" t-options="{'widget': 'text'}"/>
                                        </td>
                                        <td t-if="show_project">
                                            <span t-field="l.project_id.sudo().name"/>
                                        </td>
                                        <td t-if="show_task">
                                            <t t-if="l.task_id"><span t-field="l.task_id.sudo().name"/></t>
                                        </td>
                                        <td class="text-right">
                                            <span t-field="l.unit_amount" t-options="{'widget': 'duration', 'digital': True, 'unit': 'hour', 'round': 'minute'}"/>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td />
                                        <td />
                                        <td t-if="show_project"/>
                                        <td t-if="show_task"/>
                                        <td class="text-right"><strong>Total</strong></td>
                                        <td class="text-right"><strong t-esc="sum(docs.mapped('unit_amount'))" t-options="{'widget': 'duration', 'digital': True, 'unit': 'hour', 'round': 'minute'}"/></td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                    <div class="oe_structure"/>
                </div>
            </t>
        </t>
    </template>

    <report id="timesheet_report"
            model="account.analytic.line"
            string="Timesheet Entries"
            report_type="qweb-pdf"
            name="hr_timesheet.report_timesheet"
            file="report_timesheet"
    />
</odoo>

```

## File: report\__init__.py

```python
from . import project_report

```

## File: security\hr_timesheet_security.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.module.category" id="base.module_category_operations_timesheets">
            <field name="description">Helps you manage the timesheets.</field>
            <field name="sequence">13</field>
        </record>

        <record id="group_hr_timesheet_user" model="res.groups">
            <field name="name">See all Timesheets</field>
            <field name="category_id" ref="base.module_category_operations_timesheets"/>
            <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('group_hr_timesheet_user'))]"/>
        </record>
    </data>

    <data noupdate="0">
        <!-- This rule is in noupdate mode because it can be modify by other advanced modules -->
        <record id="timesheet_line_rule_user" model="ir.rule">
            <field name="name">account.analytic.line.timesheet.user</field>
            <field name="model_id" ref="analytic.model_account_analytic_line"/>
            <field name="domain_force">[('user_id', '=', user.id), ('project_id', '!=', False)]</field>
            <field name="groups" eval="[(4, ref('group_hr_timesheet_user'))]"/>
            <field name="perm_create" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_read" eval="0"/>
        </record>

        <record id="hr_timesheet.group_hr_timesheet_approver" model="res.groups">
            <field name="name">Team Approver</field>
            <field name="category_id" ref="base.module_category_operations_timesheets"/>
            <field name="implied_ids" eval="[(4, ref('hr_timesheet.group_hr_timesheet_user'))]"/>
        </record>

        <record id="group_timesheet_manager" model="res.groups">
            <field name="name">Administrator</field>
            <field name="category_id" ref="base.module_category_operations_timesheets"/>
            <field name="implied_ids" eval="[(4, ref('group_hr_timesheet_approver')), (4, ref('hr.group_hr_user'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>
    </data>

    <data noupdate="1">

        <record id="timesheet_line_rule_manager" model="ir.rule">
            <field name="name">account.analytic.line.timesheet.manager</field>
            <field name="model_id" ref="analytic.model_account_analytic_line"/>
            <field name="domain_force">[('project_id', '!=', False)]</field>
            <field name="groups" eval="[(4, ref('group_timesheet_manager'))]"/>
            <field name="perm_create" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_read" eval="1"/>
        </record>

    </data>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_account_analytic_line_user,analytic.account.analytic.line.timesheet.user,analytic.model_account_analytic_line,hr_timesheet.group_hr_timesheet_user,1,1,1,1
access_account_analytic_user,analytic.account.analytic.timesheet.user,analytic.model_account_analytic_account,hr_timesheet.group_hr_timesheet_user,1,1,0,0
access_uom_uom_hr_timesheet,uom.uom.timesheet.user,uom.model_uom_uom,hr_timesheet.group_hr_timesheet_user,1,0,0,0
access_project_project,project.project.timesheet.user,model_project_project,hr_timesheet.group_hr_timesheet_user,1,0,0,0
access_project_task,project.task.timesheet.user,model_project_task,hr_timesheet.group_hr_timesheet_user,1,1,0,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#B06161"/>
            <stop offset="45.785%" stop-color="#984E4E"/>
            <stop offset="100%" stop-color="#7C3838"/>
        </linearGradient>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M4,59 C2,59 -7.10542736e-15,58.8521303 0,54.8596491 L0,35.1929825 L29.9218256,0 L35,0 L35,11.3859649 L49,13.4561404 L54,35.1929825 L37.7206839,59 L4,59 Z" opacity=".324" transform="translate(0 11)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <path fill="#000" d="M37.9166667,30.1041667 L37.9166667,42.0625 C37.9166667,42.3116313 37.8365885,42.516276 37.6764323,42.6764323 C37.516276,42.8365885 37.3116313,42.9166667 37.0625,42.9166667 L28.5208333,42.9166667 C28.2717019,42.9166667 28.0670573,42.8365885 27.906901,42.6764323 C27.7467448,42.516276 27.6666667,42.3116313 27.6666667,42.0625 L27.6666667,40.3541667 C27.6666667,40.1050353 27.7467448,39.9003906 27.906901,39.7402344 C28.0670573,39.5800781 28.2717019,39.5 28.5208333,39.5 L34.5,39.5 L34.5,30.1041667 C34.5,29.8550353 34.5800781,29.6503906 34.7402344,29.4902344 C34.9003906,29.3300781 35.1050353,29.25 35.3541667,29.25 L37.0625,29.25 C37.3116313,29.25 37.516276,29.3300781 37.6764323,29.4902344 C37.8365885,29.6503906 37.9166667,29.8550353 37.9166667,30.1041667 Z M49.0208333,39.5 C49.0208333,36.8663189 48.3713095,34.4372824 47.0722656,32.2128906 C45.7732215,29.9884988 44.0115027,28.2267792 41.7871094,26.9277344 C39.5627158,25.6286895 37.1336811,24.9791667 34.5,24.9791667 C31.8663189,24.9791667 29.4372824,25.6286895 27.2128906,26.9277344 C24.9884988,28.2267792 23.2267792,29.9884988 21.9277344,32.2128906 C20.6286895,34.4372824 19.9791667,36.8663189 19.9791667,39.5 C19.9791667,42.1336811 20.6286895,44.5627158 21.9277344,46.7871094 C23.2267792,49.0115027 24.9884988,50.7732215 27.2128906,52.0722656 C29.4372824,53.3713095 31.8663189,54.0208333 34.5,54.0208333 C37.1336811,54.0208333 39.5627158,53.3713095 41.7871094,52.0722656 C44.0115027,50.7732215 45.7732215,49.0115027 47.0722656,46.7871094 C48.3713095,44.5627158 49.0208333,42.1336811 49.0208333,39.5 Z M50.0369141,26.0953981 L52.6900045,24.0961511 C53.1310787,23.7637779 53.7580818,23.8518974 54.090455,24.2929716 L55.8959001,26.6888781 C56.2282733,27.1299524 56.1401538,27.7569554 55.6990796,28.0893287 L52.8223311,30.2571141 C54.2741095,33.1090334 55,36.1899942 55,39.5 C55,43.219185 54.0835491,46.649198 52.250651,49.7900391 C50.4177527,52.9308798 47.9308798,55.4177527 44.7900391,57.250651 C41.649198,59.0835491 38.219185,60 34.5,60 C30.7808149,60 27.3508035,59.0835491 24.2099609,57.250651 C21.0691184,55.4177527 18.5822488,52.9308798 16.749349,49.7900391 C14.9164495,46.649198 14,43.219185 14,39.5 C14,35.7808149 14.9164495,32.3508035 16.749349,29.2099609 C18.5822488,26.0691184 21.0691184,23.5822488 24.2099609,21.749349 C26.6343486,20.3345506 29.2310275,19.4657874 32,19.143059 L32,17 L30,17 C29.4477153,17 29,16.5522847 29,16 L29,14 C29,13.4477153 29.4477153,13 30,13 L40,13 C40.5522847,13 41,13.4477153 41,14 L41,16 C41,16.5522847 40.5522847,17 40,17 L38,17 L38,19.2845984 C40.3963771,19.684578 42.6597235,20.5061615 44.7900391,21.749349 C46.8093002,22.9277288 48.5582591,24.3764115 50.0369141,26.0953981 Z" opacity=".3"/>
            <path fill="#FFF" d="M37.9166667,28.1041667 L37.9166667,40.0625 C37.9166667,40.3116313 37.8365885,40.516276 37.6764323,40.6764323 C37.516276,40.8365885 37.3116313,40.9166667 37.0625,40.9166667 L28.5208333,40.9166667 C28.2717019,40.9166667 28.0670573,40.8365885 27.906901,40.6764323 C27.7467448,40.516276 27.6666667,40.3116313 27.6666667,40.0625 L27.6666667,38.3541667 C27.6666667,38.1050353 27.7467448,37.9003906 27.906901,37.7402344 C28.0670573,37.5800781 28.2717019,37.5 28.5208333,37.5 L34.5,37.5 L34.5,28.1041667 C34.5,27.8550353 34.5800781,27.6503906 34.7402344,27.4902344 C34.9003906,27.3300781 35.1050353,27.25 35.3541667,27.25 L37.0625,27.25 C37.3116313,27.25 37.516276,27.3300781 37.6764323,27.4902344 C37.8365885,27.6503906 37.9166667,27.8550353 37.9166667,28.1041667 Z M49.0208333,37.5 C49.0208333,34.8663189 48.3713095,32.4372824 47.0722656,30.2128906 C45.7732215,27.9884988 44.0115027,26.2267792 41.7871094,24.9277344 C39.5627158,23.6286895 37.1336811,22.9791667 34.5,22.9791667 C31.8663189,22.9791667 29.4372824,23.6286895 27.2128906,24.9277344 C24.9884988,26.2267792 23.2267792,27.9884988 21.9277344,30.2128906 C20.6286895,32.4372824 19.9791667,34.8663189 19.9791667,37.5 C19.9791667,40.1336811 20.6286895,42.5627158 21.9277344,44.7871094 C23.2267792,47.0115027 24.9884988,48.7732215 27.2128906,50.0722656 C29.4372824,51.3713095 31.8663189,52.0208333 34.5,52.0208333 C37.1336811,52.0208333 39.5627158,51.3713095 41.7871094,50.0722656 C44.0115027,48.7732215 45.7732215,47.0115027 47.0722656,44.7871094 C48.3713095,42.5627158 49.0208333,40.1336811 49.0208333,37.5 Z M50.0369141,24.0953981 L52.6900045,22.0961511 C53.1310787,21.7637779 53.7580818,21.8518974 54.090455,22.2929716 L55.8959001,24.6888781 C56.2282733,25.1299524 56.1401538,25.7569554 55.6990796,26.0893287 L52.8223311,28.2571141 C54.2741095,31.1090334 55,34.1899942 55,37.5 C55,41.219185 54.0835491,44.649198 52.250651,47.7900391 C50.4177527,50.9308798 47.9308798,53.4177527 44.7900391,55.250651 C41.649198,57.0835491 38.219185,58 34.5,58 C30.7808149,58 27.3508035,57.0835491 24.2099609,55.250651 C21.0691184,53.4177527 18.5822488,50.9308798 16.749349,47.7900391 C14.9164495,44.649198 14,41.219185 14,37.5 C14,33.7808149 14.9164495,30.3508035 16.749349,27.2099609 C18.5822488,24.0691184 21.0691184,21.5822488 24.2099609,19.749349 C26.6343486,18.3345506 29.2310275,17.4657874 32,17.143059 L32,15 L30,15 C29.4477153,15 29,14.5522847 29,14 L29,12 C29,11.4477153 29.4477153,11 30,11 L40,11 C40.5522847,11 41,11.4477153 41,12 L41,14 C41,14.5522847 40.5522847,15 40,15 L38,15 L38,17.2845984 C40.3963771,17.684578 42.6597235,18.5061615 44.7900391,19.749349 C46.8093002,20.9277288 48.5582591,22.3764115 50.0369141,24.0953981 Z"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\timesheet_uom.js

```javascript
odoo.define('hr_timesheet.timesheet_uom', function (require) {
'use strict';

var AbstractField = require('web.AbstractField');
var basicFields = require('web.basic_fields');
var fieldUtils = require('web.field_utils');

var fieldRegistry = require('web.field_registry');

// We need the field registry to be populated, as we bind the
// timesheet_uom widget on existing field widgets.
require('web._field_registry');

var session = require('web.session');

/**
 * Extend the float factor widget to set default value for timesheet
 * use case. The 'factor' is forced to be the UoM timesheet
 * conversion from the session info.
 **/
var FieldTimesheetFactor = basicFields.FieldFloatFactor.extend({
    formatType: 'float_factor',
    /**
     * Override init to tweak options depending on the session_info
     *
     * @constructor
     * @override
     */
    init: function(parent, name, record, options) {
        this._super(parent, name, record, options);

        // force factor in format and parse options
        if (session.timesheet_uom_factor) {
            this.nodeOptions.factor = session.timesheet_uom_factor;
            this.parseOptions.factor = session.timesheet_uom_factor;
        }
    },
});


/**
 * Extend the float toggle widget to set default value for timesheet
 * use case. The 'range' is different from the default one of the
 * native widget, and the 'factor' is forced to be the UoM timesheet
 * conversion.
 **/
var FieldTimesheetToggle = basicFields.FieldFloatToggle.extend({
    formatType: 'float_factor',
    /**
     * Override init to tweak options depending on the session_info
     *
     * @constructor
     * @override
     */
    init: function(parent, name, record, options) {
        options = options || {};
        var fieldsInfo = record.fieldsInfo[options.viewType || 'default'];
        var attrs = options.attrs || (fieldsInfo && fieldsInfo[name]) || {};

        var hasRange = _.contains(_.keys(attrs.options || {}), 'range');

        this._super(parent, name, record, options);

        // Set the timesheet widget options: the range can be customized
        // by setting the option on the field in the view. The factor
        // is forced to be the UoM conversion factor.
        if (!hasRange) {
            this.nodeOptions.range = [0.00, 1.00, 0.50];
        }
        this.nodeOptions.factor = session.timesheet_uom_factor;
    },
});


/**
 * Binding depending on Company Preference
 *
 * determine wich widget will be the timesheet one.
 * Simply match the 'timesheet_uom' widget key with the correct
 * implementation (float_time, float_toggle, ...). The default
 * value will be 'float_factor'.
**/
var widgetName = 'timesheet_uom' in session ?
         session.timesheet_uom.timesheet_widget : 'float_factor';
var FieldTimesheetUom = widgetName === 'float_toggle' ? FieldTimesheetToggle
        : (
            (
                fieldRegistry.get(widgetName) &&
                fieldRegistry.get(widgetName).extend({})
            ) ||
            FieldTimesheetFactor
        );

fieldRegistry.add('timesheet_uom', FieldTimesheetUom);


// bind the formatter and parser method, and tweak the options
var _tweak_options = function(options) {
    if (!_.contains(options, 'factor')) {
        options.factor = session.timesheet_uom_factor;
    }
    return options;
};

fieldUtils.format.timesheet_uom = function(value, field, options) {
    options = _tweak_options(options || {});
    var formatter = fieldUtils.format[FieldTimesheetUom.prototype.formatType];
    return formatter(value, field, options);
};

fieldUtils.parse.timesheet_uom = function(value, field, options) {
    options = _tweak_options(options || {});
    var parser = fieldUtils.parse[FieldTimesheetUom.prototype.formatType];
    return parser(value, field, options);
};

return FieldTimesheetUom;
});

```

## File: views\assets.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="assets_backend" name="timesheet assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/hr_timesheet/static/src/js/timesheet_uom.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\hr_timesheet_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="portal_layout" name="Portal layout: timesheet menu entry" inherit_id="portal.portal_breadcrumbs" priority="35">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <li t-if="page_name == 'timesheet' or timesheet" t-attf-class="breadcrumb-item #{'active' if not timesheet else ''}">
                <t>Timesheets</t>
            </li>
        </xpath>
    </template>

    <template id="portal_my_home_timesheet" name="Portal My Home : timesheets entries" inherit_id="portal.portal_my_home" priority="45">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
            <t t-if="timesheet_count" t-call="portal.portal_docs_entry">
                <t t-set="title">Timesheets</t>
                <t t-set="url" t-value="'/my/timesheets'"/>
                <t t-set="count" t-value="timesheet_count"/>
            </t>
        </xpath>
    </template>

    <template id="portal_my_timesheets" name="My Timesheets">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Timesheets</t>
            </t>
            <t t-if="not grouped_timesheets">
                <div class="alert alert-warning mt8" role="alert">
                    There are no timesheets.
                </div>
            </t>
            <t t-if="grouped_timesheets">
                <t t-call="portal.portal_table">
                    <t t-foreach="grouped_timesheets" t-as="timesheets">
                        <thead>
                            <tr t-attf-class="{{'thead-light' if not groupby == 'none' else ''}}">
                                <th t-if="groupby == 'none'">Description</th>
                                <th t-else="">
                                    <em class="font-weight-normal text-muted">Timesheets for project:</em>
                                    <span t-field="timesheets[0].project_id.name"/></th>
                                <th>Date</th>
                                <th>Employee</th>
                                <th class="text-right">Duration</th>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="timesheets" t-as="timesheet">
                                <tr>
                                    <td><span t-esc="timesheet.name"/></td>
                                    <td><span t-field="timesheet.date" t-options='{"widget": "date"}'/></td>
                                    <td><span t-field="timesheet.employee_id"/></td>
                                    <td class="text-right"><span t-field="timesheet.unit_amount" t-options='{"widget": "duration", "unit": "hour", "round": "minute"}'/></td>
                                </tr>
                            </t>
                        </tbody>
                    </t>
                </t>
            </t>
        </t>
    </template>

</odoo>

```

## File: views\hr_timesheet_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- Timesheet root menus -->
        <menuitem id="timesheet_menu_root"
            name="Timesheets"
            sequence="55"
            groups="group_hr_timesheet_user"
            web_icon="hr_timesheet,static/description/icon_timesheet.png"/>

        <menuitem id="menu_hr_time_tracking"
            name="Timesheet"
            parent="timesheet_menu_root"
            sequence="5"/>

        <!--
            Timesheet line Views
        -->
        <record id="hr_timesheet_line_tree" model="ir.ui.view">
            <field name="name">account.analytic.line.tree.hr_timesheet</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <tree editable="top" string="Timesheet Activities">
                    <field name="date"/>
                    <field name="employee_id" invisible="1"/>
                    <field name="name"/>
                    <field name="project_id" required="1" context="{'form_view_ref': 'project.project_project_view_form_simplified',}"/>
                    <field name="task_id" context="{'default_project_id': project_id}" domain="[('project_id', '=', project_id)]"/>
                    <field name="unit_amount" widget="timesheet_uom" sum="Total"/>
                    <field name="company_id" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="timesheet_view_tree_user" model="ir.ui.view">
            <field name="name">account.analytic.line.view.tree.with.user</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="hr_timesheet_line_tree"/>
            <field name="mode">primary</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='employee_id']" position="attributes">
                    <attribute name="invisible">0</attribute>
                    <attribute name="required">1</attribute>
                    <attribute name="options">{"no_open": True}</attribute>
                </xpath>
                <xpath expr="//field[@name='date']" position="after">
                    <field name="user_id" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                </xpath>
            </field>
        </record>

        <record id="view_hr_timesheet_line_pivot" model="ir.ui.view">
            <field name="name">account.analytic.line.pivot</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <pivot string="Timesheet">
                    <field name="employee_id" type="row"/>
                    <field name="date" interval="month" type="col"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </pivot>
            </field>
        </record>

        <record id="view_hr_timesheet_line_graph" model="ir.ui.view">
            <field name="name">account.analytic.line.graph</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <graph string="Timesheet">
                    <field name="task_id" type="row"/>
                    <field name="project_id" type="row"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </graph>
            </field>
        </record>

        <record id="hr_timesheet_line_form" model="ir.ui.view">
            <field name="name">account.analytic.line.form</field>
            <field name="model">account.analytic.line</field>
            <field name="priority">1</field>
            <field name="inherit_id" eval="False"/>
            <field name="arch" type="xml">
                <form string="Analytic Entry">
                    <sheet string="Analytic Entry">
                        <group>
                            <group>
                                <field name="date"/>
                                <field name="project_id" required="1" context="{'form_view_ref': 'project.project_project_view_form_simplified',}"/>
                                <field name="task_id" context="{'default_project_id': project_id}" domain="[('project_id', '=', project_id)]"/>
                                <field name="name"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                            </group>
                            <group>
                                <field name="amount"/>
                                <field name="unit_amount" widget="timesheet_uom"/>
                                <field name="currency_id" invisible="1"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="timesheet_view_form_user" model="ir.ui.view">
            <field name="name">account.analytic.line.tree.with.user</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="hr_timesheet.hr_timesheet_line_form"/>
            <field name="mode">primary</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='company_id']" position="before">
                    <field name="employee_id" required="1" options='{"no_open": True}'/>
                    <field name="user_id" invisible="1"/>
                </xpath>
            </field>
        </record>

        <record id="hr_timesheet_line_search" model="ir.ui.view">
            <field name="name">account.analytic.line.search</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <search string="Timesheet">
                    <field name="date"/>
                    <field name="employee_id"/>
                    <field name="project_id"/>
                    <field name="task_id"/>
                    <field name="name"/>
                    <filter name="mine" string="My Timesheets" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter name="month" string="Date" date="date"/>
                    <group expand="0" string="Group By">
                        <filter string="Employee" name="groupby_employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                        <filter string="Project" name="groupby_project" domain="[]" context="{'group_by': 'project_id'}"/>
                        <filter string="Task" name="groupby_task" domain="[]" context="{'group_by': 'task_id'}"/>
                        <filter string="Date" name="groupby_date" domain="[]" context="{'group_by': 'date'}" help="Timesheet by Date"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="view_kanban_account_analytic_line" model="ir.ui.view">
            <field name="name">account.analytic.line.kanban</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="date"/>
                    <field name="user_id"/>
                    <field name="name"/>
                    <field name="project_id"/>
                    <field name="task_id" context="{'default_project_id': project_id}" domain="[('project_id', '=', project_id)]"/>
                    <field name="unit_amount"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-2">
                                        <img t-att-src="kanban_image('res.users', 'image_128', record.user_id.raw_value)" t-att-title="record.user_id.value" t-att-alt="record.user_id.value" class="oe_kanban_avatar o_image_40_cover float-left"/>
                                    </div>
                                    <div class="col-10">
                                        <div>
                                            <strong><t t-esc="record.project_id.value"/></strong>
                                        </div>
                                        <div class="text-muted">
                                            <span>
                                                <t t-esc="record.name.value"/>
                                            </span>
                                        </div>
                                    </div>
                                </div>
                                <hr class="mt4 mb4"/>
                                <span>
                                    <i class="fa fa-calendar" role="img" aria-label="Date" title="Date"></i>
                                    <t t-esc="record.date.value"/>
                                </span>
                                <span class="float-right">
                                    <strong>Duration: </strong><field name="unit_amount" widget="timesheet_uom"/>
                                </span>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <!--
            Menus and Actions
        -->
        <record id="act_hr_timesheet_line" model="ir.actions.act_window">
            <field name="name">My Timesheets</field>
            <field name="res_model">account.analytic.line</field>
            <field name="view_mode">tree,form</field>
            <field name="domain">[('project_id', '!=', False), ('user_id', '=', uid)]</field>
            <field name="context">{
                "search_default_week":1,
            }</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Record a new activity
              </p><p>
                You can register and track your workings hours by project every
                day. Every time spent on a project will become a cost and can be re-invoiced to
                customers if required.
              </p>
            </field>
        </record>

        <record id="act_hr_timesheet_line_view_tree" model="ir.actions.act_window.view">
            <field name="view_mode">tree</field>
            <field name="sequence" eval="4"/>
            <field name="view_id" ref="hr_timesheet_line_tree"/>
            <field name="act_window_id" ref="act_hr_timesheet_line"/>
        </record>

        <record id="act_hr_timesheet_line_view_form" model="ir.actions.act_window.view">
            <field name="view_mode">form</field>
            <field name="sequence" eval="5"/>
            <field name="view_id" ref="hr_timesheet_line_form"/>
            <field name="act_window_id" ref="act_hr_timesheet_line"/>
        </record>

        <record id="act_hr_timesheet_line_view_kanban" model="ir.actions.act_window.view">
            <field name="view_mode">kanban</field>
            <field name="sequence">6</field>
            <field name="view_id" ref="hr_timesheet.view_kanban_account_analytic_line"/>
            <field name="act_window_id" ref="act_hr_timesheet_line"/>
        </record>

        <menuitem id="timesheet_menu_activity_mine"
            name="My Timesheets"
            parent="menu_hr_time_tracking"
            action="act_hr_timesheet_line"/>

        <record id="timesheet_action_all" model="ir.actions.act_window">
            <field name="name">All Timesheets</field>
            <field name="res_model">account.analytic.line</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{
                'search_default_week':1,
                'search_default_groupby_employee':1,
                'search_default_groupby_project':1,
                'search_default_groupby_task':1,
            }</field>
        </record>

        <record id="timesheet_action_view_all_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="4"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="timesheet_view_tree_user"/>
            <field name="act_window_id" ref="timesheet_action_all"/>
        </record>

        <record id="timesheet_action_view_all_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="timesheet_view_form_user"/>
            <field name="act_window_id" ref="timesheet_action_all"/>
        </record>

        <menuitem id="timesheet_menu_activity_all"
            name="All Timesheets"
            parent="menu_hr_time_tracking"
            action="timesheet_action_all"/>

    </data>
</odoo>

```

## File: views\hr_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="timesheet_action_from_employee" model="ir.actions.act_window">
        <field name="name">Timesheets</field>
        <field name="res_model">account.analytic.line</field>
        <field name="search_view_id" ref="hr_timesheet_line_search"/>
        <field name="domain">[('project_id', '!=', False)]</field>
        <field name="context">{
            'search_default_month':1,
            'search_default_employee_id': [active_id],
            'default_employee_id': active_id
        }</field>
    </record>

    <record id="timesheet_action_view_from_employee_list" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="hr_timesheet_line_tree"/>
        <field name="act_window_id" ref="timesheet_action_from_employee"/>
    </record>

    <record id="timesheet_action_view_from_employee_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_timesheet_line_form"/>
        <field name="act_window_id" ref="timesheet_action_from_employee"/>
    </record>

    <record id="hr_employee_view_form_inherit_timesheet" model="ir.ui.view">
        <field name="name">hr.employee.form.timesheet</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='hr_settings']" position="inside">
                <group>
                    <group string="Timesheets" name="timesheet" groups="hr_timesheet.group_timesheet_manager">
                        <label for="timesheet_cost"/>
                        <div>
                            <field name="timesheet_cost" class="oe_inline"/> per hour
                            <field name="currency_id" invisible="1"/>
                        </div>
                    </group>
                </group>
            </xpath>
            <div name="button_box" position="inside">
                <button class="oe_stat_button" type="action" name="%(timesheet_action_from_employee)d" icon="fa-calendar" groups="hr_timesheet.group_hr_timesheet_user">
                    <div class="o_stat_info">
                        <span class="o_stat_text">Timesheets</span>
                    </div>
                </button>
            </div>
        </field>
    </record>

    <record id="hr_department_view_kanban" model="ir.ui.view">
        <field name="name">hr.department.kanban.inherit</field>
        <field name="model">hr.department</field>
        <field name="inherit_id" ref="hr.hr_department_view_kanban"/>
        <field name="groups_id" eval="[(4,ref('hr_timesheet.group_timesheet_manager'))]"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <div class="row">
                        <div class="col-12 text-left">
                            <a name="%(act_hr_timesheet_report)d" type="action"
                               context="{ 'search_default_department_id': [active_id], 'default_department_id': active_id}">
                            Timesheets
                        </a>
                        </div>
                    </div>
                </xpath>

            </data>
        </field>
    </record>
</odoo>

```

## File: views\project_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="portal_my_task" inherit_id="project.portal_my_task" name="Portal: My Task with Timesheets">
        <xpath expr="//t[@t-set='card_body']" position="inside">
            <div class="container" t-if="timesheets">
                <hr class="mt-4 mb-1"/>
                <h5 class="mt-2 mb-2">Timesheets</h5>
                <t t-call="hr_timesheet.portal_timesheet_table"/>
            </div>
        </xpath>
    </template>

    <template id="portal_timesheet_table" name="Portal Timesheet Table">
        <table class="table table-sm">
            <thead>
              <tr>
                <th>Date</th>
                <th>Description</th>
                <th>Employee</th>
                <th class="text-right">Duration</th>
              </tr>
            </thead>
            <tr t-foreach="timesheets" t-as="timesheet">
                <td><t t-esc="timesheet.date" t-options='{"widget": "date"}'/></td>
                <td><t t-esc="timesheet.name"/></td>
                <td><t t-esc="timesheet.employee_id.name"/></td>
                <td class="text-right"><span t-field="timesheet.unit_amount" t-options='{"widget": "duration", "unit": "hour", "round": "minute"}'/></td>
            </tr>
        </table>
    </template>

</odoo>

```

## File: views\project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="act_hr_timesheet_line_by_project" model="ir.actions.act_window">
            <field name="name">Activities</field>
            <field name="res_model">account.analytic.line</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="timesheet_view_tree_user"/>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{"default_project_id": active_id, "search_default_project_id": [active_id]}</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Record a new activity
              </p><p>
                You can register and track your workings hours by project every
                day. Every time spent on a project will become a cost and can be re-invoiced to
                customers if required.
              </p>
            </field>
        </record>

        <record id="project_project_view_form_simplified_inherit_timesheet" model="ir.ui.view">
            <field name="name">project.project.view.form.simplified.inherit.timesheet</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.project_project_view_form_simplified"/>
            <field name="priority">24</field>
            <field name="arch" type="xml">
                <field name="user_id" position="after">
                    <field name="allow_timesheets"/>
                </field>
            </field>
        </record>

        <record id="project_invoice_form" model="ir.ui.view">
            <field name="name">Inherit project form : Invoicing Data</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.edit_project"/>
            <field name="priority">24</field>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="%(act_hr_timesheet_line_by_project)d" type="action" icon="fa-calendar" string="Timesheets" attrs="{'invisible': [('allow_timesheets', '=', False)]}" groups="hr_timesheet.group_hr_timesheet_user"/>
                </div>
                <xpath expr="//div[@id='rating_settings']/.." position="before">
                    <div class="row mt16 o_settings_container">
                        <div class="col-lg-6 o_setting_box"  id="timesheet_settings">
                            <div class="o_setting_left_pane">
                                <field name="allow_timesheets"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="allow_timesheets" string="Timesheets"/>
                                <div class="text-muted">
                                    Log time on tasks
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record model="ir.ui.view" id="view_task_form2_inherited">
            <field name="name">project.task.form.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_form2" />
            <field name="groups_id" eval="[(6,0, (ref('hr_timesheet.group_hr_timesheet_user'),))]"/>
            <field name="arch" type="xml">
                <xpath expr="//notebook/page[@name='description_page']" position="after">
                    <field name="analytic_account_active" invisible="1"/>
                    <field name="allow_timesheets" invisible="1"/>
                    <page string="Timesheets" id="timesheets_tab" attrs="{'invisible': [('allow_timesheets', '=', False)]}">
                        <group>
                            <group>
                                <field name="planned_hours" widget="float_time"/>
                                    <label for="subtask_planned_hours" groups="project.group_subtask_project" attrs="{'invisible': [('subtask_count', '=', 0)]}"/>
                                    <div class="o_row" groups="project.group_subtask_project" attrs="{'invisible': [('subtask_count', '=', 0)]}">
                                        <field name="subtask_planned_hours" widget="float_time"/><span> planned hours</span>
                                    </div>
                            </group>
                            <group>
                                <field name="progress" widget="progressbar"/>
                            </group>
                        </group>
                        <group name="timesheet_error" attrs="{'invisible': [('analytic_account_active', '!=', False)]}">
                            <div class="alert alert-warning" role="alert">
                                You can not log timesheets on this project since is linked to an inactive analytic account. Please change it, or reactivate the current one to timesheet on the project.
                            </div>
                        </group>
                    <field name="timesheet_ids" mode="tree,kanban" attrs="{'invisible': [('analytic_account_active', '=', False)]}" context="{'default_project_id': project_id, 'default_name':''}">
                        <tree editable="bottom" string="Timesheet Activities" default_order="date">
                            <field name="date"/>
                            <field name="user_id" invisible="1"/>
                            <field name="employee_id" required="1"/>
                            <field name="name"/>
                            <field name="unit_amount" widget="timesheet_uom"/>
                            <field name="project_id" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                        </tree>
                        <kanban class="o_kanban_mobile">
                            <field name="date"/>
                            <field name="user_id"/>
                            <field name="employee_id"/>
                            <field name="name"/>
                            <field name="unit_amount"/>
                            <field name="project_id"/>
                            <templates>
                                <t t-name="kanban-box">
                                    <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                        <div class="row">
                                            <div class="col-6">
                                                <strong><span><t t-esc="record.employee_id.value"/></span></strong>
                                            </div>
                                            <div class="col-6 pull-right text-right">
                                                <strong><t t-esc="record.date.value"/></strong>
                                            </div>
                                        </div>
                                        <div class="row">
                                            <div class="col-6 text-muted">
                                                <span><t t-esc="record.name.value"/></span>
                                            </div>
                                            <div class="col-6">
                                                <span class="pull-right text-right">
                                                    <field name="unit_amount" widget="float_time"/>
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </t>
                            </templates>
                        </kanban>
                        <form  string="Timesheet Activities">
                            <sheet>
                                 <group>
                                    <field name="date"/>
                                    <field name="user_id" invisible="1"/>
                                    <field name="employee_id" required="1"/>
                                    <field name="name"/>
                                    <field name="unit_amount" string="Duration" widget="float_time"/>
                                    <field name="project_id" invisible="1"/>
                                    <field name="company_id" invisible="1"/>
                                </group>
                            </sheet>
                        </form>
                    </field>
                    <group attrs="{'invisible': [('analytic_account_active', '=', False)]}">
                        <group class="oe_subtotal_footer oe_right" name="project_hours">
                            <field name="effective_hours" widget="float_time" />
                            <field name="subtask_effective_hours" widget="float_time" attrs="{'invisible' : [('subtask_effective_hours', '=', 0.0)]}" />
                            <field name="total_hours_spent" widget="float_time" class="oe_subtotal_footer_separator" attrs="{'invisible' : [('subtask_effective_hours', '=', 0.0)]}" />
                            <field name="remaining_hours" widget="float_time" class="oe_subtotal_footer_separator" attrs="{'invisible' : [('planned_hours', '=', 0.0)]}"/>
                        </group>
                    </group>
                </page>
                </xpath>
            </field>
        </record>

        <record id="view_task_tree2_inherited" model="ir.ui.view">
            <field name="name">project.task.tree.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_tree2" />
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <field name="user_id" position="after">
                    <field name="planned_hours" widget="float_time" sum="Initially Planned Hours" optional="show"/>
                    <field name="remaining_hours" widget="float_time" sum="Remaining Hours" readonly="1" optional="show"/>
                    <field name="progress" widget="progressbar" optional="show" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="effective_hours" widget="float_time" sum="Spent Hours" invisible="1"/>
                </field>
            </field>
        </record>

        <record id="view_project_kanban_inherited" model="ir.ui.view">
            <field name="name">project.project.timesheet.kanban.inherited</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project_kanban"/>
            <field name="priority">24</field>
            <field name="arch" type="xml">
                <field name="partner_id" position="after">
                    <field name="allow_timesheets" invisible="1"/>
                </field>
                <xpath expr="//div[hasclass('o_project_kanban_boxes')]" position="inside">
                    <a t-if="record.allow_timesheets.raw_value" class="o_project_kanban_box o_project_timesheet_box" name="%(act_hr_timesheet_line_by_project)d" type="action" groups="hr_timesheet.group_hr_timesheet_user">
                        <div>
                            <span class="o_label">Timesheets</span>
                        </div>
                    </a>
                </xpath>
                <xpath expr="//a[@name='action_view_account_analytic_line']" position="attributes">
                    <attribute name="t-if">record.analytic_account_id.raw_value and !record.allow_timesheets.raw_value</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_task_kanban_inherited_progress" model="ir.ui.view">
            <field name="name">project.task.timesheet.kanban.inherited.progress</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_kanban"/>
            <field name="arch" type="xml">
                <templates position="before">
                    <field name="progress" />
                    <field name="remaining_hours" />
                </templates>
                <div class="oe_kanban_bottom_left" position="inside">
                   <t t-if="record.progress.raw_value &gt; 80 and record.progress.raw_value &lt; 100">
                       <div t-att-class="'oe_kanban_align badge badge-' + (record.progress.raw_value &gt;= 99 ? 'danger': 'warning')" title="Remaining hours">
                            <field name="remaining_hours" widget="float_time" />
                       </div>
                   </t>
                </div>
             </field>
         </record>

    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <menuitem id="hr_timesheet_menu_configuration" name="Configuration" parent="timesheet_menu_root"
        groups="group_timesheet_manager" sequence="100"/>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.hr.timesheet</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="55"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Timesheets" string="Timesheets" data-key="hr_timesheet" groups="hr_timesheet.group_timesheet_manager" id="timesheets">
                    <h2>Time Encoding</h2>
                    <div class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box" attrs="{'invisible':[('project_time_mode_id', '!=', False)]}">
                            <div class="o_setting_right_pane">
                                <label for="project_time_mode_id"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="project_time_mode_id" options="{'no_create': True, 'no_open': True}"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_right_pane">
                                <label for="timesheet_encode_uom_id"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="row">
                                    <div class="text-muted col-md-12">
                                        Set the time unit used to record your timesheets
                                    </div>
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="timesheet_encode_uom_id" options="{'no_create': True, 'no_open': True}" required="1" class="col-lg-5"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="module_project_timesheet_synchro" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_project_timesheet_synchro"/>
                                <div class="text-muted">
                                    Synchronize time spent with our web/mobile apps
                                </div>
                                <div class="content-group">
                                    <div class="row mt16 oe_center">
                                        <div class="col-lg-4">
                                            <a href="http://www.odoo.com/page/timesheet?platform=chrome">
                                                <img alt="Google Chrome Store" class="img img-fluid h-100" src="project/static/src/img/chrome_store.png"/>
                                            </a>
                                        </div>
                                        <div class="col-lg-4">
                                            <a href="http://www.odoo.com/page/timesheet?platform=ios">
                                                <img alt="Apple App Store" class="img img-fluid h-100" src="project/static/src/img/app_store.png"/>
                                            </a>
                                        </div>
                                        <div class="col-lg-4">
                                            <a href="http://www.odoo.com/page/timesheet?platform=android">
                                                <img alt="Google Play Store" class="img img-fluid h-100" src="project/static/src/img/play_store.png"/>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div name="section_leaves" groups="base.group_no_one">
                        <h2>Time Off</h2>
                        <div class="row mt16 o_settings_container" name="timesheet_control">
                            <div class="col-12 col-lg-6 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="module_project_timesheet_holidays"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_project_timesheet_holidays"/>
                                    <div class="text-muted">
                                        Create timesheets upon time off validation
                                    </div>
                                    <div class="content-group">
                                        <div id="module_project_timesheet_holidays"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="hr_timesheet_config_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'hr_timesheet', 'bin_size': False}</field>
    </record>

    <menuitem id="hr_timesheet_config_settings_menu_action" name="Settings" parent="hr_timesheet_menu_configuration"
        action="hr_timesheet_config_settings_action" sequence="0" groups="base.group_system"/>
</odoo>

```

