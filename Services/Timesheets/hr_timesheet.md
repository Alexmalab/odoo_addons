# Odoo Module: hr_timesheet

Category: Services/Timesheets

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import wizard
from . import populate

from odoo import fields, _

from odoo.addons.project import _check_exists_collaborators_for_project_sharing


def create_internal_project(env):
    # allow_timesheets is set by default, but erased for existing projects at
    # installation, as there is no analytic account for them.
    env['project.project'].search([]).write({'allow_timesheets': True})

    admin = env.ref('base.user_admin', raise_if_not_found=False)
    if not admin:
        return
    project_ids = env['res.company'].search([])._create_internal_project_task()
    env['account.analytic.line'].create([{
        'name': _("Analysis"),
        'user_id': admin.id,
        'date': fields.datetime.today(),
        'unit_amount': 0,
        'project_id': task.project_id.id,
        'task_id': task.id,
    } for task in project_ids.task_ids.filtered(lambda t: t.company_id in admin.employee_ids.company_id)])

    _check_exists_collaborators_for_project_sharing(env)

def _uninstall_hook(env):

    def update_action_window(xmlid):
        act_window = env.ref(xmlid, raise_if_not_found=False)
        if act_window and act_window.domain and 'is_internal_project' in act_window.domain:
            act_window.domain = []

    update_action_window('project.open_view_project_all')
    update_action_window('project.open_view_project_all_group_stage')

    # archive the internal projects
    project_ids = env['res.company'].search([('internal_project_id', '!=', False)]).mapped('internal_project_id')
    if project_ids:
        project_ids.write({'active': False})

    env['ir.model.data'].search([('name', 'ilike', 'internal_project_default_stage')]).unlink()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Task Logs',
    'version': '1.0',
    'category': 'Services/Timesheets',
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
    'website': 'https://www.odoo.com/app/timesheet',
    'depends': ['hr', 'hr_hourly_cost', 'analytic', 'project', 'uom'],
    'data': [
        'security/hr_timesheet_security.xml',
        'security/ir.model.access.csv',
        'security/ir.model.access.xml',
        'data/digest_data.xml',
        'views/hr_timesheet_views.xml',
        'views/res_config_settings_views.xml',
        'views/project_project_views.xml',
        'views/project_task_views.xml',
        'views/project_task_portal_templates.xml',
        'views/hr_timesheet_portal_templates.xml',
        'report/hr_timesheet_report_view.xml',
        'report/project_report_view.xml',
        'report/report_timesheet_templates.xml',
        'views/hr_department_views.xml',
        'views/hr_employee_views.xml',
        'data/hr_timesheet_data.xml',
        'views/project_task_sharing_views.xml',
        'views/project_update_views.xml',
        'wizard/hr_employee_delete_wizard_views.xml',
        'views/hr_timesheet_menus.xml',
    ],
    'demo': [
        'data/hr_timesheet_demo.xml',
    ],
    'installable': True,
    'post_init_hook': 'create_internal_project',
    'uninstall_hook': '_uninstall_hook',
    'assets': {
        'web.assets_backend': [
            'hr_timesheet/static/src/**/*',
        ],
        'web.qunit_suite_tests': [
            'hr_timesheet/static/tests/**/*',
        ],
        'project.webclient': [
            'hr_timesheet/static/src/services/**/*',
            'hr_timesheet/static/src/components/**/*',
            'hr_timesheet/static/src/scss/timesheets_task_form.scss'
        ],
    },
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
from odoo.osv.expression import AND, OR

from odoo.addons.portal.controllers.portal import CustomerPortal, pager as portal_pager
from odoo.addons.project.controllers.portal import ProjectCustomerPortal


class TimesheetCustomerPortal(CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        if 'timesheet_count' in counters:
            Timesheet = request.env['account.analytic.line']
            domain = Timesheet._timesheet_get_portal_domain()
            values['timesheet_count'] = Timesheet.sudo().search_count(domain)
        return values

    def _get_searchbar_inputs(self):
        return {
            'all': {'input': 'all', 'label': _('Search in All')},
            'employee': {'input': 'employee', 'label': _('Search in Employee')},
            'project': {'input': 'project', 'label': _('Search in Project')},
            'task': {'input': 'task', 'label': _('Search in Task')},
            'name': {'input': 'name', 'label': _('Search in Description')},
        }

    def _task_get_searchbar_sortings(self, milestones_allowed, project=False):
        values = super()._task_get_searchbar_sortings(milestones_allowed, project)
        values['progress'] = {'label': _('Progress'), 'order': 'progress asc', 'sequence': 10}
        return values

    def _get_searchbar_groupby(self):
        return {
            'none': {'input': 'none', 'label': _('None')},
            'project': {'input': 'project', 'label': _('Project')},
            'task': {'input': 'task', 'label': _('Task')},
            'date': {'input': 'date', 'label': _('Date')},
            'employee': {'input': 'employee', 'label': _('Employee')}
        }

    def _get_search_domain(self, search_in, search):
        search_domain = []
        if search_in in ('project', 'all'):
            search_domain = OR([search_domain, [('project_id', 'ilike', search)]])
        if search_in in ('name', 'all'):
            search_domain = OR([search_domain, [('name', 'ilike', search)]])
        if search_in in ('employee', 'all'):
            search_domain = OR([search_domain, [('employee_id', 'ilike', search)]])
        if search_in in ('task', 'all'):
            search_domain = OR([search_domain, [('task_id', 'ilike', search)]])
        return search_domain

    def _get_groupby_mapping(self):
        return {
            'project': 'project_id',
            'task': 'task_id',
            'employee': 'employee_id',
            'date': 'date'
        }

    def _get_searchbar_sortings(self):
        return {
            'date': {'label': _('Newest'), 'order': 'date desc'},
            'employee': {'label': _('Employee'), 'order': 'employee_id'},
            'project': {'label': _('Project'), 'order': 'project_id'},
            'task': {'label': _('Task'), 'order': 'task_id'},
            'name': {'label': _('Description'), 'order': 'name'},
        }

    @http.route(['/my/timesheets', '/my/timesheets/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_timesheets(self, page=1, sortby=None, filterby=None, search=None, search_in='all', groupby='none', **kw):
        Timesheet = request.env['account.analytic.line']
        domain = Timesheet._timesheet_get_portal_domain()
        Timesheet_sudo = Timesheet.sudo()

        values = self._prepare_portal_layout_values()
        _items_per_page = 100

        searchbar_sortings = self._get_searchbar_sortings()

        searchbar_inputs = self._get_searchbar_inputs()

        searchbar_groupby = self._get_searchbar_groupby()

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
            domain += self._get_search_domain(search_in, search)

        timesheet_count = Timesheet_sudo.search_count(domain)
        # pager
        pager = portal_pager(
            url="/my/timesheets",
            url_args={'sortby': sortby, 'search_in': search_in, 'search': search, 'filterby': filterby, 'groupby': groupby},
            total=timesheet_count,
            page=page,
            step=_items_per_page
        )

        def get_timesheets():
            groupby_mapping = self._get_groupby_mapping()
            field = groupby_mapping.get(groupby, None)
            orderby = '%s, %s' % (field, order) if field else order
            timesheets = Timesheet_sudo.search(domain, order=orderby, limit=_items_per_page, offset=pager['offset'])
            if field:
                if groupby == 'date':
                    raw_timesheets_group = Timesheet_sudo._read_group(
                        domain, ['date:day'], ['unit_amount:sum', 'id:recordset']
                    )
                    grouped_timesheets = [(records, unit_amount) for __, unit_amount, records in raw_timesheets_group]

                else:
                    time_data = Timesheet_sudo._read_group(domain, [field], ['unit_amount:sum'])
                    mapped_time = {field.id: unit_amount for field, unit_amount in time_data}
                    grouped_timesheets = [(Timesheet_sudo.concat(*g), mapped_time[k.id]) for k, g in groupbyelem(timesheets, itemgetter(field))]
                return timesheets, grouped_timesheets

            grouped_timesheets = [(
                timesheets,
                Timesheet_sudo._read_group(domain, aggregates=['unit_amount:sum'])[0][0]
            )] if timesheets else []
            return timesheets, grouped_timesheets

        timesheets, grouped_timesheets = get_timesheets()

        values.update({
            'timesheets': timesheets,
            'grouped_timesheets': grouped_timesheets,
            'page_name': 'timesheet',
            'default_url': '/my/timesheets',
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'search_in': search_in,
            'search': search,
            'sortby': sortby,
            'groupby': groupby,
            'searchbar_inputs': searchbar_inputs,
            'searchbar_groupby': searchbar_groupby,
            'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
            'filterby': filterby,
            'is_uom_day': request.env['account.analytic.line']._is_timesheet_encode_uom_day(),
        })
        return request.render("hr_timesheet.portal_my_timesheets", values)

class TimesheetProjectCustomerPortal(ProjectCustomerPortal):

    def _show_task_report(self, task_sudo, report_type, download):
        domain = request.env['account.analytic.line']._timesheet_get_portal_domain()
        task_domain = AND([domain, [('task_id', '=', task_sudo.id)]])
        timesheets = request.env['account.analytic.line'].sudo().search(task_domain)
        return self._show_report(model=timesheets,
            report_type=report_type, report_ref='hr_timesheet.timesheet_report_task_timesheets', download=download)

    def _prepare_tasks_values(self, page, date_begin, date_end, sortby, search, search_in, groupby, url="/my/tasks", domain=None, su=False, project=False):
        values = super()._prepare_tasks_values(page, date_begin, date_end, sortby, search, search_in, groupby, url, domain, su, project)
        values.update(
            is_uom_day=request.env['account.analytic.line']._is_timesheet_encode_uom_day(),
        )

        return values

```

## File: controllers\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from odoo.http import request
from odoo.osv import expression

from odoo.addons.project.controllers.portal import CustomerPortal


class ProjectCustomerPortal(CustomerPortal):

    def _get_project_sharing_company(self, project):
        company = project.company_id
        if not company:
            timesheet = request.env['account.analytic.line'].sudo().search([('project_id', '=', project.id)], limit=1)
            company = timesheet.company_id or request.env.user.company_id
        return company

    def _prepare_project_sharing_session_info(self, project, task=None):
        session_info = super()._prepare_project_sharing_session_info(project, task)
        company = request.env['res.company'].sudo().browse(session_info['user_companies']['current_company'])
        timesheet_encode_uom = company.timesheet_encode_uom_id
        project_time_mode_uom = company.project_time_mode_id

        session_info['user_companies']['allowed_companies'][company.id].update(
            timesheet_uom_id=timesheet_encode_uom.id,
            timesheet_uom_factor=project_time_mode_uom._compute_quantity(
                1.0,
                timesheet_encode_uom,
                round=False
            ),
        )
        session_info['uom_ids'] = {
            uom.id:
                {
                    'id': uom.id,
                    'name': uom.name,
                    'rounding': uom.rounding,
                    'timesheet_widget': uom.timesheet_widget,
                } for uom in [timesheet_encode_uom, project_time_mode_uom]
        }
        return session_info

    def _task_get_page_view_values(self, task, access_token, **kwargs):
        values = super(ProjectCustomerPortal, self)._task_get_page_view_values(task, access_token, **kwargs)
        domain = request.env['account.analytic.line']._timesheet_get_portal_domain()
        task_domain = expression.AND([domain, [('task_id', '=', task.id)]])
        subtask_domain = expression.AND([domain, [('task_id', 'in', task.child_ids.ids)]])
        timesheets = request.env['account.analytic.line'].sudo().search(task_domain)
        subtasks_timesheets = request.env['account.analytic.line'].sudo().search(subtask_domain)
        timesheets_by_subtask = defaultdict(lambda: request.env['account.analytic.line'].sudo())
        for timesheet in subtasks_timesheets:
            timesheets_by_subtask[timesheet.task_id] |= timesheet
        values['allow_timesheets'] = task.allow_timesheets
        values['timesheets'] = timesheets
        values['timesheets_by_subtask'] = timesheets_by_subtask
        values['is_uom_day'] = request.env['account.analytic.line']._is_timesheet_encode_uom_day()
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal
from . import project

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <record id="digest_tip_hr_timesheet_0" model="digest.tip">
            <field name="name">Tip: Record your Timesheets faster</field>
            <field name="sequence">2200</field>
            <field name="group_id" ref="hr_timesheet.group_hr_timesheet_user" />
            <field name="tip_description" type="html">
<div>
    <b class="tip_title">Tip: Record your Timesheets faster</b>
    <p class="tip_content">Record your timesheets in an instant by pressing Shift + the corresponding hotkey to add 15min to your projects.</p>
    <img src="https://download.odoocdn.com/digests/hr_timesheet/static/img/digest_tip_timesheets_hotkeys.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\hr_timesheet_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Set the JS widget -->
    <record id="uom.product_uom_day" model="uom.uom">
        <field name="timesheet_widget">float_toggle</field>
    </record>

    <function model="account.analytic.line" name="_ensure_uom_hours"/>

    <record id="uom.product_uom_hour" model="uom.uom">
        <field name="timesheet_widget">float_time</field>
    </record>

    <!-- Force Analytic account creation for projects allowing timesheet (default is True) -->
    <function
        model="project.project"
        name="_init_data_analytic_account"
        eval="[]"/>

    <record id="internal_project_default_stage" model="project.task.type">
        <field name="sequence">1</field>
        <field name="name">Internal</field>
    </record>

</odoo>

```

## File: data\hr_timesheet_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.user_demo" model="res.users">
            <field name="groups_id" eval="[
                (3, ref('project.group_project_manager')),
                (3, ref('hr_timesheet.group_timesheet_manager'))]"/>
        </record>
    </data>

    <!-- Projects -->
    <record id="project.project_project_1" model="project.project">
        <field name="allow_timesheets" eval="True"/>
    </record>

    <record id="project.project_project_2" model="project.project">
        <field name="allow_timesheets" eval="True"/>
    </record>

    <!-- Timesheet Lines -->
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

    <record id="project_1_task_1_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_5" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-12)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-13)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_7" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-15)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-17)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-19)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-20)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-24)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-26)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-27)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-28)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-28)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-32)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-34)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-35)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-37)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-38)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-48)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_22" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-49)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_23" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-50)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_24" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-51)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_1_account_analytic_line_25" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-55)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-12)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-12)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_6" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-19)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-20)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-23)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-25)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-28)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_12" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-30)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-31)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-34)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-35)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-36)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-38)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-39)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-42)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_20" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-46)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-49)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_22" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-53)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_23" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-53)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_2_account_analytic_line_24" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-60)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_7" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-11)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-15)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-17)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-18)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-20)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-25)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-27)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-27)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-31)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-37)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-37)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-38)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-39)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-40)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-40)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_22" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-41)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_23" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-44)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_24" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-44)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_25" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-45)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_26" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-45)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_27" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-46)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_28" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-47)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_29" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-48)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_30" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-50)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_5_account_analytic_line_31" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-58)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_5"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-11)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-13)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_9" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-13)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-16)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-16)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-21)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-22)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-23)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-26)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-29)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-36)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-39)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-40)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-41)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-43)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_22" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-48)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_23" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-54)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_24" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-58)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_25" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-59)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_6_account_analytic_line_26" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-60)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_1" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-17)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-19)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_8" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-21)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-22)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-23)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-25)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-29)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-30)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-30)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-31)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-35)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-41)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-42)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-44)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-45)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-47)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_22" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-51)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_7_account_analytic_line_23" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-57)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_2" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_6" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-11)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-15)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-16)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-18)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_10" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-22)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-24)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-26)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-33)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-33)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-34)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-36)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-42)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-43)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-43)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-46)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-47)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_22" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-50)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_23" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-51)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_24" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-52)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_25" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-52)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_26" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-52)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_27" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-55)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_28" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-56)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_29" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-56)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_30" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-57)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_31" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-58)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_8_account_analytic_line_32" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-59)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_8"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-18)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_5" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-21)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-24)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-29)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-32)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-32)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-33)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-49)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-53)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-54)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-54)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-55)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-56)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_17" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-57)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-59)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_9_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-60)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_9"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_10_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_10"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_10_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_10"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_1_task_12_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Meeting</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() - relativedelta(weeks=2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_12"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_13_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Summary Writing</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() - relativedelta(weeks=2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_13"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_1_task_14_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Preparation</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() - relativedelta(weeks=2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_1"/>
        <field name="task_id" ref="project.project_1_task_14"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-15)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-15)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-15)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_7" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_8" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_14" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-14)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-13)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_21" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-13)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_22" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-13)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_1_account_analytic_line_23" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-13)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_1"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-12)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-12)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-12)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-11)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_13" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_2_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_2"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_9" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_14" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_22" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_23" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_24" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_25" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_26" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_3_account_analytic_line_27" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_3"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_3" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_7" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_14" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_15" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_18" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_4_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_4"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_6" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_8" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_10" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_18" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_19" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_5_account_analytic_line_22" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_5"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_1" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_3" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_5" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_jod"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_lur"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() - relativedelta(days=1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_22" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_23" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_24" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_6_account_analytic_line_25" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_jog"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_6"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_1" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_2" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_3" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_4" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_5" model="account.analytic.line">
        <field name="name">On Site Visit</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_6" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_7" model="account.analytic.line">
        <field name="name">Quality analysis</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_8" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_jgo"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_9" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_10" model="account.analytic.line">
        <field name="name">Design</field>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_11" model="account.analytic.line">
        <field name="name">Training</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_12" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_13" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_14" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_ngh"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_15" model="account.analytic.line">
        <field name="name">Presentation</field>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">3</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-90.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_16" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_17" model="account.analytic.line">
        <field name="name">Call</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_18" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_19" model="account.analytic.line">
        <field name="name">Sprint</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_20" model="account.analytic.line">
        <field name="name">Requirements analysis</field>
        <field name="employee_id" ref="hr.employee_jep"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">1</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-30.0</field>
    </record>

    <record id="project_2_task_7_account_analytic_line_21" model="account.analytic.line">
        <field name="name">Delivery</field>
        <field name="employee_id" ref="hr.employee_hne"/>
        <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
        <field name="unit_amount">2</field>
        <field name="project_id" ref="project.project_project_2"/>
        <field name="task_id" ref="project.project_2_task_7"/>
        <field name="amount">-60.0</field>
    </record>

    <!-- Projects -->
    <record id="project.project_1_milestone_2" model="project.milestone">
        <field name="deadline" eval="(DateTime.now() + relativedelta(years=1, months=3)).strftime('%Y-%m-15')"/>
    </record>

    <record id="project_update_1" model="project.update" context="{'default_project_id': ref('project.project_project_1')}">
        <field name="name">Weekly review</field>
        <field name="user_id" eval="ref('base.user_demo')"/>
        <field name="progress" eval="30"/>
        <field name="status">on_hold</field>
    </record>
    <record id="project_update_2" model="project.update" context="{'default_project_id': ref('project.project_project_2')}">
        <field name="name">Weekly review</field>
        <field name="user_id" eval="ref('base.user_admin')"/>
        <field name="progress" eval="30"/>
        <field name="status">off_track</field>
    </record>

</odoo>

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import api, models, fields, _
from odoo.exceptions import UserError


class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    has_timesheet = fields.Boolean(compute='_compute_has_timesheet')

    def _compute_has_timesheet(self):
        self.env.cr.execute("""
        SELECT id, EXISTS(SELECT 1 FROM account_analytic_line WHERE project_id IS NOT NULL AND employee_id = e.id limit 1)
          FROM hr_employee e
         WHERE id in %s
        """, (tuple(self.ids), ))

        result = {eid[0]: eid[1] for eid in self.env.cr.fetchall()}

        for employee in self:
            employee.has_timesheet = result.get(employee.id, False)

    @api.depends('company_id', 'user_id')
    @api.depends_context('allowed_company_ids')
    def _compute_display_name(self):
        super()._compute_display_name()
        allowed_company_ids = self.env.context.get('allowed_company_ids', [])
        if len(allowed_company_ids) <= 1:
            return

        employees_count_per_user = {
            user.id: count
            for user, count in self.env['hr.employee'].sudo()._read_group(
                [('user_id', 'in', self.user_id.ids), ('company_id', 'in', allowed_company_ids)],
                ['user_id'],
                ['__count'],
            )
        }
        for employee in self:
            if employees_count_per_user.get(employee.user_id.id, 0) > 1:
                employee.display_name = f'{employee.display_name} - {employee.company_id.name}'

    def action_unlink_wizard(self):
        wizard = self.env['hr.employee.delete.wizard'].create({
            'employee_ids': self.ids,
        })
        if not self.user_has_groups('hr_timesheet.group_hr_timesheet_approver') and wizard.has_timesheet and not wizard.has_active_employee:
            raise UserError(_('You cannot delete employees who have timesheets.'))

        return {
            'name': _('Confirmation'),
            'view_mode': 'form',
            'res_model': 'hr.employee.delete.wizard',
            'views': [(self.env.ref('hr_timesheet.hr_employee_delete_wizard_form').id, 'form')],
            'type': 'ir.actions.act_window',
            'res_id': wizard.id,
            'target': 'new',
            'context': self.env.context,
        }

    def action_timesheet_from_employee(self):
        action = self.env["ir.actions.act_window"]._for_xml_id("hr_timesheet.timesheet_action_from_employee")
        context = literal_eval(action['context'].replace('active_id', str(self.id)))
        context['create'] = context.get('create', True) and self.active
        context['grid_range'] = "week"
        action['context'] = context
        return action

```

## File: models\hr_timesheet.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
import re

from odoo import api, fields, models, _, _lt
from odoo.exceptions import UserError, AccessError, ValidationError
from odoo.osv import expression

class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    def _get_favorite_project_id_domain(self, employee_id=False):
        employee_id = employee_id or self.env.user.employee_id.id
        return [
            ('employee_id', '=', employee_id),
            ('project_id', '!=', False),
        ]

    @api.model
    def _get_favorite_project_id(self, employee_id=False):
        last_timesheet_ids = self.search(self._get_favorite_project_id_domain(employee_id), limit=5)
        if len(last_timesheet_ids.project_id) == 1:
            return last_timesheet_ids.project_id.id
        return False

    @api.model
    def default_get(self, field_list):
        result = super(AccountAnalyticLine, self).default_get(field_list)
        if not self.env.context.get('default_employee_id') and 'employee_id' in field_list and result.get('user_id'):
            result['employee_id'] = self.env['hr.employee'].search([('user_id', '=', result['user_id']), ('company_id', '=', result.get('company_id', self.env.company.id))], limit=1).id
        if not self._context.get('default_project_id') and self._context.get('is_timesheet'):
            employee_id = result.get('employee_id', self.env.context.get('default_employee_id', False))
            favorite_project_id = self._get_favorite_project_id(employee_id)
            if favorite_project_id:
                result['project_id'] = favorite_project_id
        return result

    def _domain_project_id(self):
        domain = [('allow_timesheets', '=', True)]
        if not self.user_has_groups('hr_timesheet.group_timesheet_manager'):
            return expression.AND([domain,
                ['|', ('privacy_visibility', '!=', 'followers'), ('message_partner_ids', 'in', [self.env.user.partner_id.id])]
            ])
        return domain

    def _domain_employee_id(self):
        if not self.user_has_groups('hr_timesheet.group_hr_timesheet_approver'):
            return [('user_id', '=', self.env.user.id)]
        return []

    task_id = fields.Many2one(
        'project.task', 'Task', index='btree_not_null',
        compute='_compute_task_id', store=True, readonly=False,
        domain="[('allow_timesheets', '=', True), ('project_id', '=?', project_id)]")
    parent_task_id = fields.Many2one('project.task', related='task_id.parent_id', store=True)
    project_id = fields.Many2one(
        'project.project', 'Project', domain=_domain_project_id, index=True,
        compute='_compute_project_id', store=True, readonly=False)
    user_id = fields.Many2one(compute='_compute_user_id', store=True, readonly=False)
    employee_id = fields.Many2one('hr.employee', "Employee", domain=_domain_employee_id, context={'active_test': False},
        index=True, help="Define an 'hourly cost' on the employee to track the cost of their time.")
    job_title = fields.Char(related='employee_id.job_title')
    department_id = fields.Many2one('hr.department', "Department", compute='_compute_department_id', store=True, compute_sudo=True)
    manager_id = fields.Many2one('hr.employee', "Manager", related='employee_id.parent_id', store=True)
    encoding_uom_id = fields.Many2one('uom.uom', compute='_compute_encoding_uom_id')
    partner_id = fields.Many2one(compute='_compute_partner_id', store=True, readonly=False)
    readonly_timesheet = fields.Boolean(string="Readonly Timesheet", compute="_compute_readonly_timesheet", compute_sudo=True)

    @api.depends('project_id', 'task_id')
    def _compute_display_name(self):
        analytic_line_with_project = self.filtered('project_id')
        super(AccountAnalyticLine, self - analytic_line_with_project)._compute_display_name()
        for analytic_line in analytic_line_with_project:
            if analytic_line.task_id:
                analytic_line.display_name = f"{analytic_line.project_id.display_name} - {analytic_line.task_id.display_name}"
            else:
                analytic_line.display_name = analytic_line.project_id.display_name

    def _is_readonly(self):
        self.ensure_one()
        # is overridden in other timesheet related modules
        return False

    def _compute_readonly_timesheet(self):
        # Since the mrp_module gives write access to portal user on timesheet, we check that the user is an internal one before giving the write access.
        # It is not supposed to be needed, since portal user are not supposed to have access to the views using this field, but better be safe than sorry
        if not self.env.user.has_group('base.group_user'):
            self.readonly_timesheet = True
        else:
            readonly_timesheets = self.filtered(lambda timesheet: timesheet._is_readonly())
            readonly_timesheets.readonly_timesheet = True
            (self - readonly_timesheets).readonly_timesheet = False

    def _compute_encoding_uom_id(self):
        for analytic_line in self:
            analytic_line.encoding_uom_id = analytic_line.company_id.timesheet_encode_uom_id

    @api.depends('task_id.partner_id', 'project_id.partner_id')
    def _compute_partner_id(self):
        super()._compute_partner_id()
        for timesheet in self:
            if timesheet.project_id:
                timesheet.partner_id = timesheet.task_id.partner_id or timesheet.project_id.partner_id

    @api.depends('task_id')
    def _compute_project_id(self):
        for line in self:
            if not line.task_id.project_id or line.project_id == line.task_id.project_id:
                continue
            line.project_id = line.task_id.project_id

    @api.depends('project_id')
    def _compute_task_id(self):
        for line in self:
            if line.project_id and line.project_id == line.task_id.project_id:
                continue
            line.task_id = False

    @api.onchange('project_id')
    def _onchange_project_id(self):
        # TODO KBA in master - check to do it "properly", currently:
        # This onchange is used to reset the task_id when the project changes.
        # Doing it in the compute will remove the task_id when the project of a task changes.
        if self.project_id != self.task_id.project_id:
            self.task_id = False

    @api.depends('employee_id.user_id')
    def _compute_user_id(self):
        for line in self:
            line.user_id = line.employee_id.user_id if line.employee_id else self._default_user()

    @api.depends('employee_id')
    def _compute_department_id(self):
        for line in self:
            line.department_id = line.employee_id.department_id

    def _check_can_write(self, values):
        # If it's a basic user then check if the timesheet is his own.
        if not (self.user_has_groups('hr_timesheet.group_hr_timesheet_approver') or self.env.su) and any(self.env.user.id != analytic_line.user_id.id for analytic_line in self):
            raise AccessError(_("You cannot access timesheets that are not yours."))

    def _check_can_create(self):
        # override in other modules to check current user has create access
        pass

    @api.model_create_multi
    def create(self, vals_list):
        # Before creating a timesheet, we need to put a valid employee_id in the vals
        default_user_id = self._default_user()
        user_ids = []
        employee_ids = []
        # 1/ Collect the user_ids and employee_ids from each timesheet vals
        vals_list = self._timesheet_preprocess(vals_list)
        for vals in vals_list:
            if not vals.get('project_id'):
                continue
            if not vals.get('name'):
                vals['name'] = '/'
            employee_id = vals.get('employee_id', self._context.get('default_employee_id', False))
            if employee_id and employee_id not in employee_ids:
                employee_ids.append(employee_id)
            else:
                user_id = vals.get('user_id', default_user_id)
                if user_id not in user_ids:
                    user_ids.append(user_id)

        # 2/ Search all employees related to user_ids and employee_ids, in the selected companies
        HrEmployee_sudo = self.env['hr.employee'].sudo()
        employees = HrEmployee_sudo.search([
            '&', '|', ('user_id', 'in', user_ids), ('id', 'in', employee_ids), ('company_id', 'in', self.env.companies.ids)
        ])

        #                 ┌───── in search results = active/in companies ────────> was found with... ─── employee_id ───> (A) There is nothing to do, we will use this employee_id
        # 3/ Each employee                                                                          └──── user_id ──────> (B)** We'll need to select the right employee for this user
        #                 └─ not in search results = archived/not in companies ──> (C) We raise an error as we can't create a timesheet for an archived employee
        # ** We can rely on the user to get the employee_id if
        #    he has an active employee in the company of the timesheet
        #    or he has only one active employee for all selected companies
        valid_employee_per_id = {}
        employee_id_per_company_per_user = defaultdict(dict)
        for employee in employees:
            if employee.id in employee_ids:
                valid_employee_per_id[employee.id] = employee
            else:
                employee_id_per_company_per_user[employee.user_id.id][employee.company_id.id] = employee.id

        # 4/ Put valid employee_id in each vals
        error_msg = _lt('Timesheets must be created with an active employee in the selected companies.')
        for vals in vals_list:
            if not vals.get('project_id'):
                continue
            employee_in_id = vals.get('employee_id', self._context.get('default_employee_id', False))
            if employee_in_id:
                company = False
                if not vals.get('company_id'):
                    company = HrEmployee_sudo.browse(employee_in_id).company_id
                    vals['company_id'] = company.id
                if not vals.get('product_uom_id'):
                    vals['product_uom_id'] = company.project_time_mode_id.id if company else self.env['res.company'].browse(vals.get('company_id', self.env.company.id)).project_time_mode_id.id
                if employee_in_id in valid_employee_per_id:
                    vals['user_id'] = valid_employee_per_id[employee_in_id].sudo().user_id.id   # (A) OK
                    continue
                else:
                    raise ValidationError(error_msg)                                      # (C) KO
            else:
                user_id = vals.get('user_id', default_user_id)                                  # (B)...

            # ...Look for an employee, with ** conditions
            employee_per_company = employee_id_per_company_per_user.get(user_id)
            employee_out_id = False
            if employee_per_company:
                company_id = list(employee_per_company)[0] if len(employee_per_company) == 1\
                        else vals.get('company_id', self.env.company.id)
                employee_out_id = employee_per_company.get(company_id, False)

            if employee_out_id:
                vals['employee_id'] = employee_out_id
                vals['user_id'] = user_id
                company = False
                if not vals.get('company_id'):
                    company = HrEmployee_sudo.browse(employee_out_id).company_id
                    vals['company_id'] = company.id
                if not vals.get('product_uom_id'):
                    vals['product_uom_id'] = company.project_time_mode_id.id if company else self.env['res.company'].browse(vals.get('company_id', self.env.company.id)).project_time_mode_id.id
            else:  # ...and raise an error if they fail
                raise ValidationError(error_msg)

        # 5/ Finally, create the timesheets
        lines = super(AccountAnalyticLine, self).create(vals_list)
        lines._check_can_create()
        for line, values in zip(lines, vals_list):
            if line.project_id:  # applied only for timesheet
                line._timesheet_postprocess(values)
        return lines

    def write(self, values):
        self._check_can_write(values)

        values = self._timesheet_preprocess([values])[0]
        if values.get('employee_id'):
            employee = self.env['hr.employee'].browse(values['employee_id'])
            if not employee.active:
                raise UserError(_('You cannot set an archived employee to the existing timesheets.'))
        if 'name' in values and not values.get('name'):
            values['name'] = '/'
        if 'company_id' in values and not values.get('company_id'):
            del values['company_id']
        result = super(AccountAnalyticLine, self).write(values)
        # applied only for timesheet
        self.filtered(lambda t: t.project_id)._timesheet_postprocess(values)
        return result

    @api.model
    def _get_view_cache_key(self, view_id=None, view_type='form', **options):
        """The override of _get_view changing the time field labels according to the company timesheet encoding UOM
        makes the view cache dependent on the company timesheet encoding uom"""
        key = super()._get_view_cache_key(view_id, view_type, **options)
        return key + (self.env.company.timesheet_encode_uom_id,)

    @api.model
    def get_views(self, views, options=None):
        res = super().get_views(views, options)
        if options and options.get('toolbar'):
            wip_report_id = None

            def get_wip_report_id():
                return self.env['ir.model.data']._xmlid_to_res_id("mrp_account.wip_report", raise_if_not_found=False)

            for view_data in res['views'].values():
                print_data_list = view_data.get('toolbar', {}).get('print')
                if print_data_list:
                    if wip_report_id is None and re.search(r'widget="timesheet_uom(\w)*"', view_data['arch']):
                        wip_report_id = get_wip_report_id()
                    if wip_report_id:
                        view_data['toolbar']['print'] = [print_data for print_data in print_data_list if print_data['id'] != wip_report_id]
        return res

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        """ Set the correct label for `unit_amount`, depending on company UoM """
        arch, view = super()._get_view(view_id, view_type, **options)
        # Use of sudo as the portal user doesn't have access to uom
        arch = self.sudo()._apply_timesheet_label(arch, view_type=view_type)
        arch = self._apply_time_label(arch, related_model=self._name)
        return arch, view

    @api.model
    def _apply_timesheet_label(self, view_node, view_type='form'):
        doc = view_node
        encoding_uom = self.env.company.timesheet_encode_uom_id
        # Here, we select only the unit_amount field having no string set to give priority to
        # custom inheretied view stored in database. Even if normally, no xpath can be done on
        # 'string' attribute.
        for node in doc.xpath("//field[@name='unit_amount'][@widget='timesheet_uom'][not(@string)]"):
            node.set('string', _('%s Spent', re.sub(r'[\(\)]', '', encoding_uom.name or '')))
        return doc

    @api.model
    def _apply_time_label(self, view_node, related_model):
        doc = view_node
        Model = self.env[related_model]
        # Just fetch the name of the uom in `timesheet_encode_uom_id` of the current company
        encoding_uom_name = self.env.company.timesheet_encode_uom_id.with_context(prefetch_fields=False).sudo().name
        for node in doc.xpath("//field[@widget='timesheet_uom'][not(@string)] | //field[@widget='timesheet_uom_no_toggle'][not(@string)]"):
            name_with_uom = re.sub(re.escape(_('Hours')) + "|Hours", encoding_uom_name or '', Model._fields[node.get('name')]._description_string(self.env), flags=re.IGNORECASE)
            node.set('string', name_with_uom)

        return doc

    def _timesheet_get_portal_domain(self):
        if self.env.user.has_group('hr_timesheet.group_hr_timesheet_user'):
            # Then, he is internal user, and we take the domain for this current user
            return self.env['ir.rule']._compute_domain(self._name)
        return [
            '|',
                '&',
                    '|',
                        ('task_id.project_id.message_partner_ids', 'child_of', [self.env.user.partner_id.commercial_partner_id.id]),
                        ('task_id.message_partner_ids', 'child_of', [self.env.user.partner_id.commercial_partner_id.id]),
                    ('task_id.project_id.privacy_visibility', '=', 'portal'),
                '&',
                    ('task_id', '=', False),
                    '&',
                        ('project_id.message_partner_ids', 'child_of', [self.env.user.partner_id.commercial_partner_id.id]),
                        ('project_id.privacy_visibility', '=', 'portal')
        ]

    def _timesheet_preprocess(self, vals_list):
        """ Deduce other field values from the one given.
            Overrride this to compute on the fly some field that can not be computed fields.
            :param vals_list: list of dict from `create`or `write`.
        """
        timesheet_indices = set()
        task_ids, project_ids, account_ids = set(), set(), set()
        for index, vals in enumerate(vals_list):
            if not vals.get('project_id') and not vals.get('task_id'):
                continue
            timesheet_indices.add(index)
            if vals.get('task_id'):
                task_ids.add(vals['task_id'])
            elif vals.get('project_id'):
                project_ids.add(vals['project_id'])
            if vals.get('account_id'):
                account_ids.add(vals['account_id'])

        task_per_id = {}
        if task_ids:
            tasks = self.env['project.task'].sudo().browse(task_ids)
            for task in tasks:
                task_per_id[task.id] = task
                if not task.project_id:
                    raise ValidationError(_('Timesheets cannot be created on a private task.'))
            account_ids = account_ids.union(tasks.analytic_account_id.ids, tasks.project_id.analytic_account_id.ids)

        project_per_id = {}
        if project_ids:
            projects = self.env['project.project'].sudo().browse(project_ids)
            account_ids = account_ids.union(projects.analytic_account_id.ids)
            project_per_id = {p.id: p for p in projects}

        accounts = self.env['account.analytic.account'].sudo().browse(account_ids)
        account_per_id = {account.id: account for account in accounts}

        uom_id_per_company = {
            company: company.project_time_mode_id.id
            for company in accounts.company_id
        }

        for index in timesheet_indices:
            vals = vals_list[index]
            data = task_per_id[vals['task_id']] if vals.get('task_id') else project_per_id[vals['project_id']]
            if not vals.get('project_id'):
                vals['project_id'] = data.project_id.id
            if not vals.get('account_id'):
                account = data._get_task_analytic_account_id() if vals.get('task_id') else data.analytic_account_id
                if not account or not account.active:
                    raise ValidationError(_('Timesheets must be created on a project or a task with an active analytic account.'))
                vals['account_id'] = account.id
                vals['company_id'] = account.company_id.id or data.company_id.id
            if not vals.get('product_uom_id'):
                company = account_per_id[vals['account_id']].company_id or data.company_id
                vals['product_uom_id'] = uom_id_per_company.get(company.id, company.project_time_mode_id.id) or self.env.company.project_time_mode_id.id
            # Set the top-level analytic plan according to the given analytic account
            plan = self.env['account.analytic.account'].browse(vals.get('account_id')).root_plan_id
            if plan:
                vals[plan._column_name()] = vals.get('account_id')
        return vals_list

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
                dictionary values to write (may be empty).
        """
        result = {id_: {} for id_ in self.ids}
        sudo_self = self.sudo()  # this creates only one env for all operation that required sudo()
        # (re)compute the amount (depending on unit_amount, employee_id for the cost, and account_id for currency)
        if any(field_name in values for field_name in ['unit_amount', 'employee_id', 'account_id']):
            for timesheet in sudo_self:
                cost = timesheet._hourly_cost()
                amount = -timesheet.unit_amount * cost
                amount_converted = timesheet.employee_id.currency_id._convert(
                    amount, timesheet.account_id.currency_id or timesheet.currency_id, self.env.company, timesheet.date)
                result[timesheet.id].update({
                    'amount': amount_converted,
                })
        return result

    def _is_timesheet_encode_uom_day(self):
        company_uom = self.env.company.timesheet_encode_uom_id
        return company_uom == self.env.ref('uom.product_uom_day')

    def _is_updatable_timesheet(self):
        return True

    @api.model
    def _convert_hours_to_days(self, time):
        uom_hour = self.env.ref('uom.product_uom_hour')
        uom_day = self.env.ref('uom.product_uom_day')
        return round(uom_hour._compute_quantity(time, uom_day, raise_if_failure=False), 2)

    def _get_timesheet_time_day(self):
        return self._convert_hours_to_days(self.unit_amount)

    def _hourly_cost(self):
        self.ensure_one()
        return self.employee_id.hourly_cost or 0.0

    def _get_report_base_filename(self):
        task_ids = self.task_id
        if len(task_ids) == 1:
            return _('Timesheets - %s', task_ids.name)
        return _('Timesheets')

    def _default_user(self):
        return self.env.context.get('user_id', self.env.user.id)

    @api.model
    def _ensure_uom_hours(self):
        uom_hours = self.env.ref('uom.product_uom_hour', raise_if_not_found=False)
        if not uom_hours:
            uom_hours = self.env['uom.uom'].create({
                'name': "Hours",
                'category_id': self.env.ref('uom.uom_categ_wtime').id,
                'factor': 8,
                'uom_type': "smaller",
            })
            self.env['ir.model.data'].create({
                'name': 'product_uom_hour',
                'model': 'uom.uom',
                'module': 'uom',
                'res_id': uom_hours.id,
                'noupdate': True,
            })

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        """ The widget 'timesheet_uom' needs to know which UoM conversion factor and which javascript
            widget to apply, depending on the current company.
        """
        result = super(Http, self).session_info()
        if self.env.user._is_internal():
            company_ids = self.env.user.company_ids

            for company in company_ids:
                result["user_companies"]["allowed_companies"][company.id].update({
                    "timesheet_uom_id": company.timesheet_encode_uom_id.id,
                    "timesheet_uom_factor": company.project_time_mode_id._compute_quantity(
                        1.0,
                        company.timesheet_encode_uom_id,
                        round=False
                    ),
                })
            result["uom_ids"] = self.get_timesheet_uoms()
        return result

    @api.model
    def get_timesheet_uoms(self):
        company_ids = self.env.user.company_ids
        uom_ids = company_ids.mapped('timesheet_encode_uom_id') | \
                  company_ids.mapped('project_time_mode_id')
        return {
            uom.id:
                {
                    'id': uom.id,
                    'name': uom.name,
                    'rounding': uom.rounding,
                    'timesheet_widget': uom.timesheet_widget,
                } for uom in uom_ids
        }

```

## File: models\ir_ui_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrUiMenu(models.Model):
    _inherit = 'ir.ui.menu'

    def _load_menus_blacklist(self):
        res = super()._load_menus_blacklist()
        if self.env.user.has_group('hr_timesheet.group_hr_timesheet_approver'):
            res.append(self.env.ref('hr_timesheet.timesheet_menu_activity_user').id)
        return res

```

## File: models\project_collaborator.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProjectCollaborator(models.Model):
    _inherit = 'project.collaborator'

    @api.model
    def _toggle_project_sharing_portal_rules(self, active):
        super()._toggle_project_sharing_portal_rules(active)
        # ir.model.access
        access_timesheet_portal = self.env.ref('hr_timesheet.access_account_analytic_line_portal_user').sudo()
        if access_timesheet_portal.active != active:
            access_timesheet_portal.write({'active': active})

        # ir.rule
        timesheet_portal_ir_rule = self.env.ref('hr_timesheet.timesheet_line_rule_portal_user').sudo()
        if timesheet_portal_ir_rule.active != active:
            timesheet_portal_ir_rule.write({'active': active})

```

## File: models\project_project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from collections import defaultdict

from odoo import models, fields, api, _, _lt
from odoo.exceptions import ValidationError, RedirectWarning

class Project(models.Model):
    _inherit = "project.project"

    allow_timesheets = fields.Boolean(
        "Timesheets", compute='_compute_allow_timesheets', store=True, readonly=False,
        default=True)
    analytic_account_id = fields.Many2one(
        # note: replaces ['|', ('company_id', '=', False), ('company_id', '=', company_id)]
        domain="""[
            '|', ('company_id', '=', False), ('company_id', '=?', company_id),
            ('partner_id', '=?', partner_id),
        ]"""
    )

    timesheet_ids = fields.One2many('account.analytic.line', 'project_id', 'Associated Timesheets')
    timesheet_encode_uom_id = fields.Many2one('uom.uom', compute='_compute_timesheet_encode_uom_id')
    total_timesheet_time = fields.Integer(
        compute='_compute_total_timesheet_time', groups='hr_timesheet.group_hr_timesheet_user',
        help="Total number of time (in the proper UoM) recorded in the project, rounded to the unit.", compute_sudo=True)
    encode_uom_in_days = fields.Boolean(compute='_compute_encode_uom_in_days')
    is_internal_project = fields.Boolean(compute='_compute_is_internal_project', search='_search_is_internal_project')
    remaining_hours = fields.Float(compute='_compute_remaining_hours', string='Remaining Invoiced Time', compute_sudo=True)
    is_project_overtime = fields.Boolean('Project in Overtime', compute='_compute_remaining_hours', search='_search_is_project_overtime', compute_sudo=True)
    allocated_hours = fields.Float(string='Allocated Hours')

    def _compute_encode_uom_in_days(self):
        self.encode_uom_in_days = self.env.company.timesheet_encode_uom_id == self.env.ref('uom.product_uom_day')

    @api.depends('company_id', 'company_id.timesheet_encode_uom_id')
    @api.depends_context('company')
    def _compute_timesheet_encode_uom_id(self):
        for project in self:
            project.timesheet_encode_uom_id = project.company_id.timesheet_encode_uom_id or self.env.company.timesheet_encode_uom_id

    @api.depends('analytic_account_id')
    def _compute_allow_timesheets(self):
        without_account = self.filtered(lambda t: not t.analytic_account_id and t._origin)
        without_account.update({'allow_timesheets': False})

    @api.depends('company_id')
    def _compute_is_internal_project(self):
        for project in self:
            project.is_internal_project = project == project.company_id.internal_project_id

    @api.model
    def _search_is_internal_project(self, operator, value):
        if not isinstance(value, bool):
            raise ValueError(_('Invalid value: %s', value))
        if operator not in ['=', '!=']:
            raise ValueError(_('Invalid operator: %s', operator))

        query = """
            SELECT C.internal_project_id
            FROM res_company C
            WHERE C.internal_project_id IS NOT NULL
        """
        if (operator == '=' and value is True) or (operator == '!=' and value is False):
            operator_new = 'inselect'
        else:
            operator_new = 'not inselect'
        return [('id', operator_new, (query, ()))]

    @api.model
    def _get_view_cache_key(self, view_id=None, view_type='form', **options):
        """The override of _get_view changing the time field labels according to the company timesheet encoding UOM
        makes the view cache dependent on the company timesheet encoding uom"""
        key = super()._get_view_cache_key(view_id, view_type, **options)
        return key + (self.env.company.timesheet_encode_uom_id,)

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        if view_type in ['tree', 'form'] and self.env.company.timesheet_encode_uom_id == self.env.ref('uom.product_uom_day'):
            arch = self.env['account.analytic.line']._apply_time_label(arch, related_model=self._name)
        return arch, view

    @api.depends('allow_timesheets', 'timesheet_ids')
    def _compute_remaining_hours(self):
        timesheets_read_group = self.env['account.analytic.line']._read_group(
            [('project_id', 'in', self.ids)],
            ['project_id'],
            ['unit_amount:sum'],
        )
        timesheet_time_dict = {project.id: unit_amount_sum for project, unit_amount_sum in timesheets_read_group}
        for project in self:
            project.remaining_hours = project.allocated_hours - timesheet_time_dict.get(project.id, 0)
            project.is_project_overtime = project.remaining_hours < 0

    @api.model
    def _search_is_project_overtime(self, operator, value):
        if not isinstance(value, bool):
            raise ValueError(_('Invalid value: %s', value))
        if operator not in ['=', '!=']:
            raise ValueError(_('Invalid operator: %s', operator))

        query = """
            SELECT Project.id
              FROM project_project AS Project
              JOIN project_task AS Task
                ON Project.id = Task.project_id
             WHERE Project.allocated_hours > 0
               AND Project.allow_timesheets = TRUE
               AND Task.parent_id IS NULL
               AND Task.state IN ('01_in_progress', '02_changes_requested', '03_approved', '04_waiting_normal')
          GROUP BY Project.id
            HAVING Project.allocated_hours - SUM(Task.effective_hours) < 0
        """
        if (operator == '=' and value is True) or (operator == '!=' and value is False):
            operator_new = 'inselect'
        else:
            operator_new = 'not inselect'
        return [('id', operator_new, (query, ()))]

    @api.constrains('allow_timesheets', 'analytic_account_id')
    def _check_allow_timesheet(self):
        for project in self:
            if project.allow_timesheets and not project.analytic_account_id:
                raise ValidationError(_('You cannot use timesheets without an analytic account.'))

    @api.depends('timesheet_ids', 'timesheet_encode_uom_id')
    def _compute_total_timesheet_time(self):
        timesheets_read_group = self.env['account.analytic.line']._read_group(
            [('project_id', 'in', self.ids)],
            ['project_id', 'product_uom_id'],
            ['unit_amount:sum'],
        )
        timesheet_time_dict = defaultdict(list)
        for project, product_uom, unit_amount_sum in timesheets_read_group:
            timesheet_time_dict[project.id].append((product_uom, unit_amount_sum))

        for project in self:
            # Timesheets may be stored in a different unit of measure, so first
            # we convert all of them to the reference unit
            # if the timesheet has no product_uom_id then we take the one of the project
            total_time = 0.0
            for product_uom, unit_amount in timesheet_time_dict[project.id]:
                factor = (product_uom or project.timesheet_encode_uom_id).factor_inv
                total_time += unit_amount * (1.0 if project.encode_uom_in_days else factor)
            # Now convert to the proper unit of measure set in the settings
            total_time *= project.timesheet_encode_uom_id.factor
            project.total_timesheet_time = int(round(total_time))

    @api.model_create_multi
    def create(self, vals_list):
        """ Create an analytic account if project allow timesheet and don't provide one
            Note: create it before calling super() to avoid raising the ValidationError from _check_allow_timesheet
        """
        defaults = self.default_get(['allow_timesheets', 'analytic_account_id'])
        for vals in vals_list:
            allow_timesheets = vals.get('allow_timesheets', defaults.get('allow_timesheets'))
            analytic_account_id = vals.get('analytic_account_id', defaults.get('analytic_account_id'))
            if allow_timesheets and not analytic_account_id:
                analytic_account = self._create_analytic_account_from_values(vals)
                vals['analytic_account_id'] = analytic_account.id
        return super().create(vals_list)

    def write(self, values):
        # create the AA for project still allowing timesheet
        if values.get('allow_timesheets') and not values.get('analytic_account_id'):
            for project in self:
                if not project.analytic_account_id:
                    project._create_analytic_account()
        return super(Project, self).write(values)

    @api.depends('is_internal_project', 'company_id')
    @api.depends_context('allowed_company_ids')
    def _compute_display_name(self):
        super()._compute_display_name()
        if len(self.env.context.get('allowed_company_ids') or []) <= 1:
            return

        for project in self:
            if project.is_internal_project:
                project.display_name = f'{project.display_name} - {project.company_id.name}'

    @api.model
    def _init_data_analytic_account(self):
        self.search([('analytic_account_id', '=', False), ('allow_timesheets', '=', True)])._create_analytic_account()

    @api.ondelete(at_uninstall=False)
    def _unlink_except_contains_entries(self):
        """
        If some projects to unlink have some timesheets entries, these
        timesheets entries must be unlinked first.
        In this case, a warning message is displayed through a RedirectWarning
        and allows the user to see timesheets entries to unlink.
        """
        projects_with_timesheets = self.filtered(lambda p: p.timesheet_ids)
        if projects_with_timesheets:
            if len(projects_with_timesheets) > 1:
                warning_msg = _("These projects have some timesheet entries referencing them. Before removing these projects, you have to remove these timesheet entries.")
            else:
                warning_msg = _("This project has some timesheet entries referencing it. Before removing this project, you have to remove these timesheet entries.")
            raise RedirectWarning(
                warning_msg, self.env.ref('hr_timesheet.timesheet_action_project').id,
                _('See timesheet entries'), {'active_ids': projects_with_timesheets.ids})

    @api.model
    def get_create_edit_project_ids(self):
        return []

    def _convert_project_uom_to_timesheet_encode_uom(self, time):
        uom_from = self.company_id.project_time_mode_id
        uom_to = self.env.company.timesheet_encode_uom_id
        return round(uom_from._compute_quantity(time, uom_to, raise_if_failure=False), 2)

    def action_project_timesheets(self):
        action = self.env['ir.actions.act_window']._for_xml_id('hr_timesheet.act_hr_timesheet_line_by_project')
        action['display_name'] = _("%(name)s's Timesheets", name=self.name)
        return action

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if not self.allow_timesheets or not self.env.user.has_group("hr_timesheet.group_hr_timesheet_user"):
            return buttons

        encode_uom = self.env.company.timesheet_encode_uom_id
        uom_ratio = self.env.ref('uom.product_uom_hour').factor / encode_uom.factor

        allocated = self.allocated_hours / uom_ratio
        effective = self.total_timesheet_time / uom_ratio
        color = ""
        if allocated:
            number = f"{round(effective)} / {round(allocated)} {encode_uom.name}"
            success_rate = round(100 * effective / allocated)
            if success_rate > 100:
                number = _lt(
                    "%(effective)s / %(allocated)s %(uom_name)s",
                    effective=round(effective),
                    allocated=round(allocated),
                    uom_name=encode_uom.name,
                )
                color = "text-danger"
            else:
                number = _lt(
                    "%(effective)s / %(allocated)s %(uom_name)s (%(success_rate)s%%)",
                    effective=round(effective),
                    allocated=round(allocated),
                    uom_name=encode_uom.name,
                    success_rate=success_rate,
                )
                if success_rate >= 80:
                    color = "text-warning"
                else:
                    color = "text-success"
        else:
            number = _lt(
                    "%(effective)s %(uom_name)s",
                    effective=round(effective),
                    uom_name=encode_uom.name,
                )

        buttons.append({
            "icon": f"clock-o {color}",
            "text": _lt("Timesheets"),
            "number": number,
            "action_type": "object",
            "action": "action_project_timesheets",
            "show": True,
            "sequence": 2,
        })
        if allocated and success_rate > 100:
            buttons.append({
                "icon": f"warning {color}",
                "text": _lt("Extra Time"),
                "number": _lt(
                    "%(exceeding_hours)s %(uom_name)s (+%(exceeding_rate)s%%)",
                    exceeding_hours=round(effective - allocated),
                    uom_name=encode_uom.name,
                    exceeding_rate=round(100 * (effective - allocated) / allocated),
                ),
                "action_type": "object",
                "action": "action_project_timesheets",
                "show": True,
                "sequence": 3,
            })

        return buttons

    def action_view_tasks(self):
        # Using the timesheet filter hide context
        action = super().action_view_tasks()
        action['context']['allow_timesheets'] = self.allow_timesheets
        return action

    def action_project_sharing(self):
        # Using the timesheet filter hide context
        action = super().action_project_sharing()
        action['context']['allow_timesheets'] = self.allow_timesheets
        return action

```

## File: models\project_task.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re

from odoo import models, fields, api, _
from odoo.exceptions import UserError, RedirectWarning
from odoo.addons.rating.models.rating_data import OPERATOR_MAPPING

PROJECT_TASK_READABLE_FIELDS = {
    'allow_timesheets',
    'analytic_account_active',
    'effective_hours',
    'encode_uom_in_days',
    'allocated_hours',
    'progress',
    'overtime',
    'remaining_hours',
    'subtask_effective_hours',
    'subtask_allocated_hours',
    'timesheet_ids',
    'total_hours_spent',
}

class Task(models.Model):
    _name = "project.task"
    _inherit = "project.task"

    project_id = fields.Many2one(domain="['|', ('company_id', '=', False), ('company_id', '=?',  company_id), ('is_internal_project', '=', False)]")
    analytic_account_active = fields.Boolean("Active Analytic Account", compute='_compute_analytic_account_active', compute_sudo=True, recursive=True)
    allow_timesheets = fields.Boolean(
        "Allow timesheets",
        compute='_compute_allow_timesheets', search='_search_allow_timesheets',
        compute_sudo=True, readonly=True,
        help="Timesheets can be logged on this task.")
    remaining_hours = fields.Float("Remaining Hours", compute='_compute_remaining_hours', store=True, readonly=True, help="Number of allocated hours minus the number of hours spent.")
    remaining_hours_percentage = fields.Float(compute='_compute_remaining_hours_percentage', search='_search_remaining_hours_percentage')
    effective_hours = fields.Float("Hours Spent", compute='_compute_effective_hours', compute_sudo=True, store=True)
    total_hours_spent = fields.Float("Total Hours", compute='_compute_total_hours_spent', store=True, help="Time spent on this task and its sub-tasks (and their own sub-tasks).")
    progress = fields.Float("Progress", compute='_compute_progress_hours', store=True, group_operator="avg")
    overtime = fields.Float(compute='_compute_progress_hours', store=True)
    subtask_effective_hours = fields.Float("Hours Spent on Sub-Tasks", compute='_compute_subtask_effective_hours', recursive=True, store=True, help="Time spent on the sub-tasks (and their own sub-tasks) of this task.")
    timesheet_ids = fields.One2many('account.analytic.line', 'task_id', 'Timesheets')
    encode_uom_in_days = fields.Boolean(compute='_compute_encode_uom_in_days', default=lambda self: self._uom_in_days())
    display_name = fields.Char(help="""Use these keywords in the title to set new tasks:\n
        30h Allocate 30 hours to the task
        #tags Set tags on the task
        @user Assign the task to a user
        ! Set the task a high priority\n
        Make sure to use the right format and order e.g. Improve the configuration screen 5h #feature #v16 @Mitchell !""",
    )
    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS | PROJECT_TASK_READABLE_FIELDS

    @api.constrains('project_id')
    def _check_project_root(self):
        private_tasks = self.filtered(lambda t: not t.project_id)
        if private_tasks and self.env['account.analytic.line'].sudo().search_count([('task_id', 'in', private_tasks.ids)], limit=1):
            raise UserError(_("This task cannot be private because there are some timesheets linked to it."))

    def _uom_in_days(self):
        return self.env.company.timesheet_encode_uom_id == self.env.ref('uom.product_uom_day')

    def _compute_encode_uom_in_days(self):
        self.encode_uom_in_days = self._uom_in_days()

    @api.depends('project_id.allow_timesheets')
    def _compute_allow_timesheets(self):
        for task in self:
            task.allow_timesheets = task.project_id.allow_timesheets

    def _search_allow_timesheets(self, operator, value):
        query = self.env['project.project'].sudo()._search([
            ('allow_timesheets', operator, value),
        ])
        return [('project_id', 'in', query)]

    @api.depends('analytic_account_id.active', 'project_id.analytic_account_id.active')
    def _compute_analytic_account_active(self):
        """ Overridden in sale_timesheet """
        for task in self:
            task.analytic_account_active = task._get_task_analytic_account_id().active

    @api.depends('timesheet_ids.unit_amount')
    def _compute_effective_hours(self):
        if not any(self._ids):
            for task in self:
                task.effective_hours = sum(task.timesheet_ids.mapped('unit_amount'))
            return
        timesheet_read_group = self.env['account.analytic.line']._read_group([('task_id', 'in', self.ids)], ['task_id'], ['unit_amount:sum'])
        timesheets_per_task = {task.id: amount for task, amount in timesheet_read_group}
        for task in self:
            task.effective_hours = timesheets_per_task.get(task.id, 0.0)

    @api.depends('effective_hours', 'subtask_effective_hours', 'allocated_hours')
    def _compute_progress_hours(self):
        for task in self:
            if (task.allocated_hours > 0.0):
                task_total_hours = task.effective_hours + task.subtask_effective_hours
                task.overtime = max(task_total_hours - task.allocated_hours, 0)
                task.progress = round(100.0 * task_total_hours / task.allocated_hours, 2)
            else:
                task.progress = 0.0
                task.overtime = 0

    @api.depends('allocated_hours', 'remaining_hours')
    def _compute_remaining_hours_percentage(self):
        for task in self:
            if task.allocated_hours > 0.0:
                task.remaining_hours_percentage = task.remaining_hours / task.allocated_hours
            else:
                task.remaining_hours_percentage = 0.0

    def _search_remaining_hours_percentage(self, operator, value):
        if operator not in OPERATOR_MAPPING:
            raise NotImplementedError(_('This operator %s is not supported in this search method.', operator))
        query = f"""
            SELECT id
              FROM {self._table}
             WHERE remaining_hours > 0
               AND allocated_hours > 0
               AND remaining_hours / allocated_hours {operator} %s
            """
        return [('id', 'inselect', (query, (value,)))]

    @api.depends('effective_hours', 'subtask_effective_hours', 'allocated_hours')
    def _compute_remaining_hours(self):
        for task in self:
            if not task.allocated_hours:
                task.remaining_hours = 0.0
            else:
                task.remaining_hours = task.allocated_hours - task.effective_hours - task.subtask_effective_hours

    @api.depends('effective_hours', 'subtask_effective_hours')
    def _compute_total_hours_spent(self):
        for task in self:
            task.total_hours_spent = task.effective_hours + task.subtask_effective_hours

    @api.depends('child_ids.effective_hours', 'child_ids.subtask_effective_hours')
    def _compute_subtask_effective_hours(self):
        for task in self.with_context(active_test=False):
            task.subtask_effective_hours = sum(child_task.effective_hours + child_task.subtask_effective_hours for child_task in task.child_ids)

    def _get_group_pattern(self):
        return {
            **super()._get_group_pattern(),
            'allocated_hours': r'\s(\d+(?:\.\d+)?)[hH]',
        }

    def _prepare_pattern_groups(self):
        return [self._get_group_pattern()['allocated_hours']] + super()._prepare_pattern_groups()

    def _get_cannot_start_with_patterns(self):
        return super()._get_cannot_start_with_patterns() + [r'(?!\d+(?:\.\d+)?(?:h|H))']

    def _extract_allocated_hours(self):
        allocated_hours_group = self._get_group_pattern()['allocated_hours']
        if self.allow_timesheets:
            self.allocated_hours = sum(float(num) for num in re.findall(allocated_hours_group, self.display_name))
            self.display_name, dummy = re.subn(allocated_hours_group, '', self.display_name)

    def _get_groups(self):
        return [lambda task: task._extract_allocated_hours()] + super()._get_groups()

    def action_view_subtask_timesheet(self):
        self.ensure_one()
        is_internal_user = self.env.user.has_group('base.group_user')
        task_ids = self.with_context(active_test=False)._get_subtask_ids_per_task_id().get(self.id, [])
        action = self.env["ir.actions.actions"]._for_xml_id("hr_timesheet.timesheet_action_all")
        graph_view_id = self.env.ref("hr_timesheet.view_hr_timesheet_line_graph_by_employee").id
        new_views = []
        for view in action['views']:
            if not is_internal_user:
                if view[1] == 'tree':
                    tree_view_id = self.env['ir.model.data']._xmlid_to_res_id('hr_timesheet.hr_timesheet_line_portal_tree')
                    if tree_view_id:
                        new_views.insert(0, (tree_view_id, 'tree'))
                        continue
                elif view[1] == 'form':
                    form_view_id = self.env['ir.model.data']._xmlid_to_res_id('hr_timesheet.timesheet_view_form_portal_user')
                    if form_view_id:
                        new_views.append((form_view_id, 'form'))
                        continue
                elif view[1] == 'kanban':
                    kanban_view_id = self.env['ir.model.data']._xmlid_to_res_id('hr_timesheet.view_kanban_account_analytic_line_portal_user')
                    if kanban_view_id:
                        new_views.append((kanban_view_id, 'kanban'))
                        continue
            if view[1] == 'graph':
                view = (graph_view_id, 'graph')
            new_views.insert(0, view) if view[1] == 'tree' else new_views.append(view)

        action.update({
            'display_name': _('Timesheets'),
            'context': {'default_project_id': self.project_id.id, 'grid_range': 'week'},
            'domain': [('project_id', '!=', False), ('task_id', 'in', task_ids)],
            'views': new_views,
        })
        return action

    def _get_timesheet(self):
        # Is override in sale_timesheet
        return self.timesheet_ids

    @api.depends_context('hr_timesheet_display_remaining_hours')
    def _compute_display_name(self):
        super()._compute_display_name()
        if self.env.context.get('hr_timesheet_display_remaining_hours'):
            for task in self:
                if task.allow_timesheets and task.allocated_hours > 0 and task.encode_uom_in_days:
                    days_left = _("(%s days remaining)", task._convert_hours_to_days(task.remaining_hours))
                    task.display_name = task.display_name + "\u00A0" + days_left
                elif task.allow_timesheets and task.allocated_hours > 0:
                    hours, mins = (str(int(duration)).rjust(2, '0') for duration in divmod(abs(task.remaining_hours) * 60, 60))
                    hours_left = _(
                        "(%(sign)s%(hours)s:%(minutes)s remaining)",
                        sign='-' if task.remaining_hours < 0 else '',
                        hours=hours,
                        minutes=mins,
                    )
                    task.display_name = task.display_name + "\u00A0" + hours_left

    @api.model
    def _get_view_cache_key(self, view_id=None, view_type='form', **options):
        """The override of _get_view changing the time field labels according to the company timesheet encoding UOM
        makes the view cache dependent on the company timesheet encoding uom"""
        key = super()._get_view_cache_key(view_id, view_type, **options)
        return key + (self.env.company.timesheet_encode_uom_id,)

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        """ Set the correct label for `unit_amount`, depending on company UoM """
        arch, view = super()._get_view(view_id, view_type, **options)
        # Use of sudo as the portal user doesn't have access to uom
        arch = self.env['account.analytic.line'].sudo()._apply_timesheet_label(arch)

        if view_type in ['tree', 'pivot', 'graph', 'form'] and self.env.company.timesheet_encode_uom_id == self.env.ref('uom.product_uom_day'):
            arch = self.env['account.analytic.line']._apply_time_label(arch, related_model=self._name)

        return arch, view

    @api.ondelete(at_uninstall=False)
    def _unlink_except_contains_entries(self):
        """
        If some tasks to unlink have some timesheets entries, these
        timesheets entries must be unlinked first.
        In this case, a warning message is displayed through a RedirectWarning
        and allows the user to see timesheets entries to unlink.
        """
        timesheet_data = self.env['account.analytic.line'].sudo()._read_group(
            [('task_id', 'in', self.ids)],
            ['task_id'],
        )
        task_with_timesheets_ids = [task.id for task, in timesheet_data]
        if task_with_timesheets_ids:
            if len(task_with_timesheets_ids) > 1:
                warning_msg = _("These tasks have some timesheet entries referencing them. Before removing these tasks, you have to remove these timesheet entries.")
            else:
                warning_msg = _("This task has some timesheet entries referencing it. Before removing this task, you have to remove these timesheet entries.")
            raise RedirectWarning(
                warning_msg, self.env.ref('hr_timesheet.timesheet_action_task').id,
                _('See timesheet entries'), {'active_ids': task_with_timesheets_ids})

    @api.model
    def _convert_hours_to_days(self, time):
        uom_hour = self.env.ref('uom.product_uom_hour')
        uom_day = self.env.ref('uom.product_uom_day')
        return round(uom_hour._compute_quantity(time, uom_day, raise_if_failure=False), 2)

```

## File: models\project_update.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ProjectUpdate(models.Model):
    _inherit = "project.update"

    display_timesheet_stats = fields.Boolean(compute="_compute_display_timesheet_stats")
    allocated_time = fields.Integer("Allocated Time", readonly=True)
    timesheet_time = fields.Integer("Timesheet Time", readonly=True)
    timesheet_percentage = fields.Integer(compute="_compute_timesheet_percentage")
    uom_id = fields.Many2one("uom.uom", "Unit Of Measure", readonly=True)

    def _compute_timesheet_percentage(self):
        for update in self:
            update.timesheet_percentage = update.allocated_time and round(update.timesheet_time * 100 / update.allocated_time)

    def _compute_display_timesheet_stats(self):
        for update in self:
            update.display_timesheet_stats = update.project_id.allow_timesheets

    # ---------------------------------
    # ORM Override
    # ---------------------------------
    @api.model_create_multi
    def create(self, vals_list):
        updates = super().create(vals_list)
        encode_uom = self.env.company.timesheet_encode_uom_id
        ratio = self.env.ref("uom.product_uom_hour").ratio / encode_uom.ratio
        for update in updates:
            project = update.project_id
            project.sudo().last_update_id = update
            update.write({
                "uom_id": encode_uom,
                "allocated_time": round(project.allocated_hours / ratio),
                "timesheet_time": round(project.total_timesheet_time / ratio),
            })
        return updates

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class ResCompany(models.Model):
    _inherit = 'res.company'

    @api.model
    def _default_project_time_mode_id(self):
        uom = self.env.ref('uom.product_uom_hour', raise_if_not_found=False)
        wtime = self.env.ref('uom.uom_categ_wtime')
        if not uom:
            uom = self.env['uom.uom'].search([('category_id', '=', wtime.id), ('uom_type', '=', 'reference')], limit=1)
        if not uom:
            uom = self.env['uom.uom'].search([('category_id', '=', wtime.id)], limit=1)
        return uom

    @api.model
    def _default_timesheet_encode_uom_id(self):
        uom = self.env.ref('uom.product_uom_hour', raise_if_not_found=False)
        wtime = self.env.ref('uom.uom_categ_wtime')
        if not uom:
            uom = self.env['uom.uom'].search([('category_id', '=', wtime.id), ('uom_type', '=', 'reference')], limit=1)
        if not uom:
            uom = self.env['uom.uom'].search([('category_id', '=', wtime.id)], limit=1)
        return uom
    
    project_time_mode_id = fields.Many2one('uom.uom', string='Project Time Unit',
        default=_default_project_time_mode_id,
        help="This will set the unit of measure used in projects and tasks.\n"
             "If you use the timesheet linked to projects, don't "
             "forget to setup the right unit of measure in your employees.")
    timesheet_encode_uom_id = fields.Many2one('uom.uom', string="Timesheet Encoding Unit",
        default=_default_timesheet_encode_uom_id, domain=lambda self: [('category_id', '=', self.env.ref('uom.uom_categ_wtime').id)])
    internal_project_id = fields.Many2one(
        'project.project', string="Internal Project",
        help="Default project value for timesheet generated from time off type.")

    @api.constrains('internal_project_id')
    def _check_internal_project_id_company(self):
        if self.filtered(lambda company: company.internal_project_id and company.internal_project_id.sudo().company_id != company):
            raise ValidationError(_('The Internal Project of a company should be in that company.'))

    @api.model_create_multi
    def create(self, values):
        company = super(ResCompany, self).create(values)
        # use sudo as the user could have the right to create a company
        # but not to create a project. On the other hand, when the company
        # is created, it is not in the allowed_company_ids on the env
        company.sudo()._create_internal_project_task()
        return company

    def _create_internal_project_task(self):
        results = []
        type_ids_ref = self.env.ref('hr_timesheet.internal_project_default_stage', raise_if_not_found=False)
        type_ids = [(4, type_ids_ref.id)] if type_ids_ref else []
        for company in self:
            company = company.with_company(company)
            results += [{
                'name': _('Internal'),
                'allow_timesheets': True,
                'company_id': company.id,
                'type_ids': type_ids,
                'task_ids': [(0, 0, {
                    'name': name,
                    'company_id': company.id,
                }) for name in [_('Training'), _('Meeting')]]
            }]
        project_ids = self.env['project.project'].create(results)
        projects_by_company = {project.company_id.id: project for project in project_ids}
        for company in self:
            company.internal_project_id = projects_by_company.get(company.id, False)
        return project_ids

    def _is_timesheet_hour_uom(self):
        return self.timesheet_encode_uom_id and self.timesheet_encode_uom_id == self.env.ref('uom.product_uom_hour')

    def _timesheet_uom_text(self):
        return self._is_timesheet_hour_uom() and _("hours") or _("days")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_project_timesheet_holidays = fields.Boolean("Time Off",
        compute="_compute_timesheet_modules", store=True, readonly=False)
    reminder_user_allow = fields.Boolean(string="Employee Reminder")
    reminder_allow = fields.Boolean(string="Approver Reminder")
    project_time_mode_id = fields.Many2one(
        'uom.uom', related='company_id.project_time_mode_id', string='Project Time Unit', readonly=False,
        help="This will set the unit of measure used in projects and tasks.\n"
             "If you use the timesheet linked to projects, don't "
             "forget to setup the right unit of measure in your employees.")
    is_encode_uom_days = fields.Boolean(compute='_compute_is_encode_uom_days')
    timesheet_encode_method = fields.Selection([
        ('hours', 'Hours / Minutes'),
        ('days', 'Days / Half-Days'),
    ], string='Encoding Method', compute="_compute_timesheet_encode_method", inverse="_inverse_timesheet_encode_method", required=True)

    @api.depends('company_id')
    def _compute_timesheet_encode_method(self):
        uom_day = self.env.ref('uom.product_uom_day', raise_if_not_found=False)
        for settings in self:
            settings.timesheet_encode_method = 'days' if settings.company_id.timesheet_encode_uom_id == uom_day else 'hours'

    def _inverse_timesheet_encode_method(self):
        uom_day = self.env.ref('uom.product_uom_day', raise_if_not_found=False)
        uom_hour = self.env.ref('uom.product_uom_hour', raise_if_not_found=False)
        for settings in self:
            settings.company_id.timesheet_encode_uom_id = uom_day if settings.timesheet_encode_method == 'days' else uom_hour

    @api.depends('timesheet_encode_method')
    def _compute_is_encode_uom_days(self):
        for settings in self:
            settings.is_encode_uom_days = settings.timesheet_encode_method == 'days'

    @api.depends('module_hr_timesheet')
    def _compute_timesheet_modules(self):
        self.filtered(lambda config: not config.module_hr_timesheet).update({
            'module_project_timesheet_holidays': False,
        })

```

## File: models\uom_uom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Uom(models.Model):
    _inherit = 'uom.uom'

    def _unprotected_uom_xml_ids(self):
        # Override
        # When timesheet App is installed, we also need to protect the hour UoM
        # from deletion (and warn in case of modification)
        return [
            "product_uom_dozen",
        ]

    # widget used in the webclient when this unit is the one used to encode timesheets.
    timesheet_widget = fields.Char("Widget")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee
from . import hr_timesheet
from . import ir_http
from . import ir_ui_menu
from . import res_company
from . import res_config_settings
from . import project_project
from . import project_task
from . import project_update
from . import project_collaborator
from . import uom_uom

```

## File: populate\hr_timesheet.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from collections import defaultdict
from dateutil.relativedelta import relativedelta

from odoo import models
from odoo.tools import populate

class AccountAnalyticLine(models.Model):
    _inherit = "account.analytic.line"
    _populate_sizes = {"small": 500, "medium": 5000, "large": 50000}
    _populate_dependencies = ["project.project", "project.task", "hr.employee"]


    def _populate_factories(self):
        projects_groups = self.env['project.project']._read_group(
            domain=[('id', 'in', self.env.registry.populated_models["project.project"])],
            groupby=['company_id'],
            aggregates=['id:array_agg'],
        )
        project_ids = []
        projects_per_company = defaultdict(list)
        for company, ids in projects_groups:
            project_ids += ids
            projects_per_company[company.id] = ids

        tasks_per_project = {
            project.id: ids
            for project, ids in self.env['project.task']._read_group(
                domain=[
                    ('id', 'in', self.env.registry.populated_models["project.task"]),
                    ('project_id', 'in', project_ids),
                ],
                groupby=['project_id'],
                aggregates=['id:array_agg'],
            )
        }
        employees_per_company = {
            company.id: ids
            for company, ids in self.env['hr.employee']._read_group(
                domain=[('id', 'in', self.env.registry.populated_models["hr.employee"])],
                groupby=['company_id'],
                aggregates=['id:array_agg'],
            )
        }
        # Companies with projects and employees only
        company_ids = list(
            set(self.env.registry.populated_models["res.company"])\
          & set(employees_per_company.keys())\
          & set(projects_per_company.keys())
        )

        def get_company_id(random, **kwargs):
            return random.choice(company_ids)

        def get_project_id(random, **kwargs):
            return random.choice(projects_per_company[kwargs['values']['company_id']])

        def get_task_id(random, **kwargs):
            task_ids = tasks_per_project[kwargs['values']['project_id']]
            return random.choice(task_ids + [False] * (len(task_ids) // 3))

        def get_employee_id(random, **kwargs):
            return random.choice(employees_per_company[kwargs['values']['company_id']])

        return [
            ("date", populate.randdatetime(relative_before=relativedelta(months=-3), relative_after=relativedelta(months=3))),
            ('unit_amount', populate.randfloat(0.0, 8.0)),
            ("company_id", populate.compute(get_company_id)),
            ("project_id", populate.compute(get_project_id)),
            ("task_id", populate.compute(get_task_id)),
            ("employee_id", populate.compute(get_employee_id)),
        ]

```

## File: populate\__init__.py

```python
from . import hr_timesheet

```

## File: report\hr_timesheet_report_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="timesheets_analysis_report_pivot_employee" model="ir.ui.view">
            <field name="name">timesheets.analysis.report.pivot</field>
            <field name="model">timesheets.analysis.report</field>
            <field name="arch" type="xml">
                <pivot string="Timesheets Analysis" sample="1" disable_linking="True">
                    <field name="employee_id" type="row"/>
                    <field name="date" interval="month" type="col"/>
                    <field name="amount" string="Timesheet Costs"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </pivot>
            </field>
        </record>

        <record id="timesheets_analysis_report_graph_employee" model="ir.ui.view">
            <field name="name">timesheets.analysis.report.graph</field>
            <field name="model">timesheets.analysis.report</field>
            <field name="arch" type="xml">
                <graph string="Timesheets" sample="1" js_class="hr_timesheet_graphview" disable_linking="True">
                    <field name="employee_id" type="row"/>
                    <field name="amount" string="Timesheet Costs"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </graph>
            </field>
        </record>

        <record id="timesheets_analysis_report_pivot_project" model="ir.ui.view">
            <field name="name">timesheets.analysis.report.pivot</field>
            <field name="model">timesheets.analysis.report</field>
            <field name="arch" type="xml">
                <pivot string="Timesheets Analysis" sample="1" disable_linking="True">
                    <field name="project_id" type="row"/>
                    <field name="date" interval="month" type="col"/>
                    <field name="amount" string="Timesheet Costs"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </pivot>
            </field>
        </record>

        <record id="timesheets_analysis_report_graph_project" model="ir.ui.view">
            <field name="name">timesheets.analysis.report.graph</field>
            <field name="model">timesheets.analysis.report</field>
            <field name="arch" type="xml">
                <graph string="Timesheets" sample="1" js_class="hr_timesheet_graphview" disable_linking="True">
                    <field name="project_id" type="row"/>
                    <field name="amount" string="Timesheet Costs"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </graph>
            </field>
        </record>

        <record id="timesheets_analysis_report_pivot_task" model="ir.ui.view">
            <field name="name">timesheets.analysis.report.pivot</field>
            <field name="model">timesheets.analysis.report</field>
            <field name="arch" type="xml">
                <pivot string="Timesheets Analysis" sample="1" disable_linking="True">
                    <field name="amount" string="Timesheet Costs"/>
                    <field name="project_id" type="row"/>
                    <field name="task_id" type="row"/>
                    <field name="date" interval="month" type="col"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </pivot>
            </field>
        </record>

        <record id="timesheets_analysis_report_graph_task" model="ir.ui.view">
            <field name="name">timesheets.analysis.report.graph</field>
            <field name="model">timesheets.analysis.report</field>
            <field name="arch" type="xml">
                <graph string="Timesheets" sample="1" js_class="hr_timesheet_graphview" disable_linking="True">
                    <field name="project_id" type="row"/>
                    <field name="task_id" type="row"/>
                    <field name="amount" string="Timesheet Costs"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </graph>
            </field>
        </record>

        <record id="hr_timesheet_report_search" model="ir.ui.view">
            <field name="name">timesheets.analysis.report.search</field>
            <field name="model">timesheets.analysis.report</field>
            <field name="inherit_id" ref="hr_timesheet.hr_timesheet_line_search"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <search position="attributes">
                    <attribute name="string">Timesheet Report</attribute>
                </search>
            </field>
        </record>

        <!-- Group by employee -->
        <record id="act_hr_timesheet_report" model="ir.actions.act_window">
            <field name="name">Timesheets by Employee</field>
            <field name="res_model">timesheets.analysis.report</field>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{}</field>
            <field name="search_view_id" ref="hr_timesheet_report_search"/>
            <field name="view_mode">pivot,graph</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_empty_folder">
                No data yet!
              </p><p>
                Analyze the projects and tasks on which your employees spend their time.<br/>
                Evaluate which part is billable and what costs it represents.
              </p>
            </field>
        </record>

        <record model="ir.actions.act_window.view" id="act_hr_timesheet_report_pivot">
            <field name="sequence" eval="5"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="hr_timesheet.timesheets_analysis_report_pivot_employee"/>
            <field name="act_window_id" ref="act_hr_timesheet_report"/>
        </record>

        <record id="timesheet_action_view_report_by_employee_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="6"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="hr_timesheet.timesheets_analysis_report_graph_employee"/>
            <field name="act_window_id" ref="act_hr_timesheet_report"/>
        </record>

        <!-- Group by project-->
        <record id="timesheet_action_report_by_project" model="ir.actions.act_window">
            <field name="name">Timesheets by Project</field>
            <field name="res_model">timesheets.analysis.report</field>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{}</field>
            <field name="search_view_id" ref="hr_timesheet_report_search"/>
            <field name="view_mode">pivot,graph</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_empty_folder">
                No data yet!
              </p><p>
                Analyze the projects and tasks on which your employees spend their time.<br/>
                Evaluate which part is billable and what costs it represents.
              </p>
            </field>
        </record>

        <record id="timesheet_action_view_report_by_project_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="hr_timesheet.timesheets_analysis_report_pivot_project"/>
            <field name="act_window_id" ref="timesheet_action_report_by_project"/>
        </record>

        <record id="timesheet_action_view_report_by_project_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="6"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="hr_timesheet.timesheets_analysis_report_graph_project"/>
            <field name="act_window_id" ref="timesheet_action_report_by_project"/>
        </record>

        <!-- Group by task -->
        <record id="timesheet_action_report_by_task" model="ir.actions.act_window">
            <field name="name">Timesheets by Task</field>
            <field name="res_model">timesheets.analysis.report</field>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{}</field>
            <field name="search_view_id" ref="hr_timesheet_report_search"/>
            <field name="view_mode">pivot,graph</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_empty_folder">
                No data yet!
              </p><p>
                Analyze the projects and tasks on which your employees spend their time.<br/>
                Evaluate which part is billable and what costs it represents.
              </p>
            </field>
        </record>

        <record id="timesheet_action_view_report_by_task_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="hr_timesheet.timesheets_analysis_report_pivot_task"/>
            <field name="act_window_id" ref="timesheet_action_report_by_task"/>
        </record>

        <record id="timesheet_action_view_report_by_task_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="6"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="hr_timesheet.timesheets_analysis_report_graph_task"/>
            <field name="act_window_id" ref="timesheet_action_report_by_task"/>
        </record>
    </data>
</odoo>

```

## File: report\project_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ReportProjectTaskUser(models.Model):
    _inherit = "report.project.task.user"

    allocated_hours = fields.Float('Allocated Time', readonly=True)
    effective_hours = fields.Float('Hours Spent', readonly=True)
    remaining_hours = fields.Float('Remaining Hours', readonly=True)
    remaining_hours_percentage = fields.Float('Remaining Hours Percentage', readonly=True)
    progress = fields.Float('Progress', group_operator='avg', readonly=True)
    overtime = fields.Float(readonly=True)
    total_hours_spent = fields.Float('Hours By Task (Including Subtasks)', help="Time spent on this task, including its sub-tasks.")

    def _select(self):
        return super()._select() +  """,
                CASE WHEN COALESCE(t.allocated_hours, 0) = 0 THEN 0.0 ELSE LEAST((t.effective_hours * 100) / t.allocated_hours, 100) END as progress,
                t.effective_hours,
                CASE WHEN COALESCE(t.allocated_hours, 0) = 0 THEN 0.0 ELSE t.allocated_hours - t.effective_hours END as remaining_hours,
                CASE WHEN t.allocated_hours > 0 THEN t.remaining_hours / t.allocated_hours ELSE 0 END as remaining_hours_percentage,
                COALESCE(t.allocated_hours, 0) as allocated_hours,
                t.overtime,
                t.total_hours_spent
        """

    def _group_by(self):
        return super()._group_by() + """,
                t.effective_hours,
                t.subtask_effective_hours,
                t.allocated_hours,
                t.overtime,
                t.total_hours_spent
        """

    @api.model
    def _get_view_cache_key(self, view_id=None, view_type='form', **options):
        """The override of _get_view changing the time field labels according to the company timesheet encoding UOM
        makes the view cache dependent on the company timesheet encoding uom"""
        key = super()._get_view_cache_key(view_id, view_type, **options)
        return key + (self.env.company.timesheet_encode_uom_id,)

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        if view_type in ['pivot', 'graph'] and self.env.company.timesheet_encode_uom_id == self.env.ref('uom.product_uom_day'):
            arch = self.env['account.analytic.line']._apply_time_label(arch, related_model=self._name)
        return arch, view

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
                <xpath expr="//graph" position="attributes">
                    <attribute name="js_class">hr_timesheet_graphview</attribute>
                </xpath>
                <xpath expr="//field[@name='project_id']" position='after'>
                    <field name="allocated_hours" widget="timesheet_uom" type="measure"/>
                    <field name="effective_hours" widget="timesheet_uom" type="measure"/>
                    <field name="overtime" widget="timesheet_uom"/>
                    <field name="total_hours_spent" widget="timesheet_uom"/>
                    <field name="remaining_hours" widget="timesheet_uom" type="measure"/>
                </xpath>
             </field>
        </record>

        <record id="view_task_project_user_pivot_inherited" model="ir.ui.view">
            <field name="name">report.project.task.user.pivot.inherited</field>
            <field name="model">report.project.task.user</field>
            <field name="inherit_id" ref="project.view_task_project_user_pivot"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='project_id']" position='after'>
                    <field name="allocated_hours" widget="timesheet_uom" type="measure"/>
                    <field name="effective_hours" widget="timesheet_uom" type="measure"/>
                    <field name="remaining_hours" widget="timesheet_uom" type="measure"/>
                    <field name="total_hours_spent" widget="timesheet_uom" type="measure"/>
                    <field name="overtime" widget="timesheet_uom" type="measure"/>
                </xpath>
             </field>
        </record>
    </data>
</odoo>

```

## File: report\report_timesheet_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="hr_timesheet.timesheet_table">
        <t t-set='is_uom_day' t-value='lines._is_timesheet_encode_uom_day()'/>
        <div class="row mt8">
            <div class="col-12">
                <table class="table table-sm">
                    <thead style="display: table-row-group">
                        <tr>
                            <t t-set="timesheet_record_label" t-value="False"/>
                            <t t-if="show_task">
                                <t t-set="timesheet_record_label">Task</t>
                            </t>
                            <t t-elif="show_project">
                                <t t-set="timesheet_record_label">Project</t>
                            </t>
                            <th class="text-start align-middle"><span>Date</span></th>
                            <th class="text-start align-middle"><span>Employee</span></th>
                            <th class="text-start align-middle" t-if="timesheet_record_label"><span t-out="timesheet_record_label"/></th>
                            <th class="text-start align-middle"><span>Description</span></th>
                            <th class="text-end">
                                <span t-if="not is_uom_day">Hours</span>
                                <span t-else="">Days</span>
                            </th>
                        </tr>
                   </thead>
                   <tbody>
                        <tr t-foreach="lines" t-as="line" t-att-style="'background-color: #F1F1F1;' if line_index % 2 == 0 else ''">
                            <td class="align-middle">
                               <span t-field="line.date">2021-09-01</span>
                            </td>
                            <td class="align-middle">
                               <span t-if="line.user_id.partner_id.name" t-field="line.user_id.partner_id.name">Audrey Peterson</span>
                               <span t-else="" t-field="line.employee_id">Audrey Peterson</span>
                            </td>
                            <t t-set="timesheet_record_info" t-value="False"/>
                            <t t-if="show_project">
                                <t t-set="timesheet_record_info" t-value="line.project_id.sudo().name"/>
                                <t t-if="show_task and line.task_id" t-set="timesheet_record_info" t-value="'%s / %s' % (timesheet_record_info, line.task_id.sudo().name)"/>
                            </t>
                            <t t-elif="show_task" t-set="timesheet_record_info" t-value="line.task_id.sudo().name"/>
                            <td t-if="show_task or show_project" class="align-middle">
                                <span t-if="timesheet_record_info" t-out="timesheet_record_info">Research and Development/New Portal System</span>
                            </td>
                            <td class="align-middle">
                                <span t-field="line.name" t-options="{'widget': 'text'}">Call client and discuss project</span>
                            </td>
                            <td class="text-end align-middle">
                                <span t-if="not is_uom_day" t-field="line.unit_amount" t-options="{'widget': 'duration', 'digital': True, 'unit': 'hour', 'round': 'minute'}">2 hours</span>
                                <span t-else="" t-esc="line._get_timesheet_time_day()" t-options="{'widget': 'timesheet_uom'}">1 day</span>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end" colspan="100">
                                <strong t-if="not is_uom_day">
                                    <span style="margin-right: 15px;">Total (Hours)</span>
                                    <t t-esc="sum(lines.mapped('unit_amount'))" t-options="{'widget': 'duration', 'digital': True, 'unit': 'hour', 'round': 'minute'}">2 hours</t>
                                </strong>
                                <strong t-else="">
                                    <span style="margin-right: 15px;">Total (Days)</span>
                                    <t t-esc="lines._convert_hours_to_days(sum(lines.mapped('unit_amount')))" t-options="{'widget': 'timesheet_uom'}">1 day</t>
                                </strong>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </template>

    <template id="hr_timesheet.timesheet_project_task_page">
        <t t-set="show_record" t-value="len(docs.ids) == 1"/>
        <t t-set="title" t-value="docs._description"/>
        <t t-set="company" t-value="docs.company_id if len(docs) == 1 else docs.env.company"/>
        <t t-call="web.html_container">
            <t t-call="web.internal_layout">
                <div class="page">
                    <t t-foreach="docs" t-as="doc">
                        <t t-if="from_project" t-set="show_task"
                            t-value="bool(doc.timesheet_ids.task_id)"/>
                        <div class="oe_structure"/>
                        <div class="row mt8">
                            <div class="col-12">
                                <t t-if="doc.allow_timesheets and doc.timesheet_ids">
                                    <h1 class="my-4">
                                        <t t-if="not show_record">
                                            <t t-out="title"/>: <span t-field="doc.name"/>
                                        </t>
                                    </h1>
                                    <h2>
                                        <span>Timesheets
                                            <t t-if="show_record">
                                                for the <t t-out="doc.name"/> <t t-out="title"/>
                                            </t>
                                        </span>
                                    </h2>
                                    <t t-set='lines' t-value='doc.timesheet_ids'/>
                                    <t t-call="hr_timesheet.timesheet_table"/>
                                </t>
                            </div>
                        </div>
                    </t>
                </div>
            </t>
        </t>
    </template>

    <template id="report_timesheet">
        <t t-call="web.html_container">
            <t t-call="web.internal_layout">
                <t t-set="company" t-value="docs.project_id.company_id if len(docs.project_id) == 1 else docs.env.company"/>
                <t t-set="show_task" t-value="bool(docs.task_id)"/>
                <t t-set="show_project" t-value="len(docs.project_id) > 1"/>
                <div class="page">
                    <div class="oe_structure"/>
                    <div class="row mt8">
                        <div class="col-lg-12">
                            <h2>
                                <span>Timesheets
                                    <t t-if="len(docs.project_id) == 1">
                                        for the <t t-out="docs.project_id.name">Research and Development</t> Project
                                    </t>
                                </span>
                            </h2>
                        </div>
                    </div>
                    <t t-set='lines' t-value='docs'/>
                    <t t-call="hr_timesheet.timesheet_table"/>
                    <div class="oe_structure"/>
                </div>
            </t>
        </t>
    </template>

    <!-- Project Task Timesheet Report for given timesheets -->
    <template id="report_timesheet_task">
        <t t-call="web.html_container">
            <t t-call="web.internal_layout">
                <t t-set="company" t-value="docs.project_id.company_id if len(docs.project_id) == 1 else docs.env.company"/>
                <t t-set="show_task" t-value="len(docs.task_id) > 1"/>
                <t t-set="show_project" t-value="False"/>
                <div class="page">
                    <div class="oe_structure"/>
                    <div class="row mt8">
                        <div class="col-12">
                            <h2>
                                <span>Timesheets
                                    <t t-if="len(docs.task_id) == 1">
                                        for <t t-out="docs.task_id.name"/>
                                    </t>
                                </span>
                            </h2>
                        </div>
                    </div>
                    <t t-set='lines' t-value='docs'/>
                    <t t-call="hr_timesheet.timesheet_table"/>
                    <div class="oe_structure"/>
                </div>
            </t>
        </t>
    </template>

    <record id="timesheet_report" model="ir.actions.report">
        <field name="name">Timesheets</field>
        <field name="model">account.analytic.line</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr_timesheet.report_timesheet</field>
        <field name="report_file">report_timesheet</field>
        <field name="binding_model_id" ref="model_account_analytic_line"/>
        <field name="binding_type">report</field>
    </record>

    <!-- Project Task Timesheet Report -->
    <template id="report_project_task_timesheet">
        <t t-call="hr_timesheet.timesheet_project_task_page"/>
    </template>

    <record id="timesheet_report_task" model="ir.actions.report">
        <field name="name">Timesheets</field>
        <field name="model">project.task</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr_timesheet.report_project_task_timesheet</field>
        <field name="report_file">report_timesheet_task</field>
        <field name="binding_model_id" ref="model_project_task"/>
        <field name="binding_type">report</field>
    </record>

    <!-- Project Timesheet Report -->
    <template id="report_timesheet_project">
        <t t-set="from_project" t-value="True"/>
        <t t-call="hr_timesheet.timesheet_project_task_page"/>
    </template>

    <record id="timesheet_report_project" model="ir.actions.report">
        <field name="name">Timesheets</field>
        <field name="model">project.project</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr_timesheet.report_timesheet_project</field>
        <field name="report_file">report_timesheet_project</field>
        <field name="binding_model_id" ref="model_project_project"/>
    </record>

    <record id="timesheet_report_task_timesheets" model="ir.actions.report">
        <field name="name">Timesheets</field>
        <field name="model">account.analytic.line</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr_timesheet.report_timesheet_task</field>
        <field name="report_file">report_timesheet</field>
        <field name="binding_type">report</field>
    </record>
</odoo>

```

## File: report\timesheets_analysis_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from psycopg2 import sql

from odoo import api, tools, fields, models


class TimesheetsAnalysisReport(models.Model):
    _name = "timesheets.analysis.report"
    _description = "Timesheets Analysis Report"
    _auto = False

    name = fields.Char("Description", readonly=True)
    user_id = fields.Many2one("res.users", string="User", readonly=True)
    project_id = fields.Many2one("project.project", string="Project", readonly=True)
    task_id = fields.Many2one("project.task", string="Task", readonly=True)
    parent_task_id = fields.Many2one("project.task", string="Parent Task", readonly=True)
    employee_id = fields.Many2one("hr.employee", string="Employee", readonly=True)
    manager_id = fields.Many2one("hr.employee", "Manager", readonly=True)
    company_id = fields.Many2one("res.company", string="Company", readonly=True)
    department_id = fields.Many2one("hr.department", string="Department", readonly=True)
    currency_id = fields.Many2one('res.currency', string="Currency", readonly=True)
    date = fields.Date("Date", readonly=True)
    amount = fields.Monetary("Amount", readonly=True)
    unit_amount = fields.Float("Hours Spent", readonly=True)

    @property
    def _table_query(self):
        return "%s %s %s" % (self._select(), self._from(), self._where())

    @api.model
    def _select(self):
        return """
            SELECT
                A.id AS id,
                A.name AS name,
                A.user_id AS user_id,
                A.project_id AS project_id,
                A.task_id AS task_id,
                A.parent_task_id AS parent_task_id,
                A.employee_id AS employee_id,
                A.manager_id AS manager_id,
                A.company_id AS company_id,
                A.department_id AS department_id,
                A.currency_id AS currency_id,
                A.date AS date,
                A.amount AS amount,
                A.unit_amount AS unit_amount
        """

    @api.model
    def _from(self):
        return "FROM account_analytic_line A"

    @api.model
    def _where(self):
        return "WHERE A.project_id IS NOT NULL"

    @api.model
    def _get_view_cache_key(self, view_id=None, view_type='form', **options):
        """The override of _get_view changing the time field labels according to the company timesheet encoding UOM
        makes the view cache dependent on the company timesheet encoding uom"""
        key = super()._get_view_cache_key(view_id, view_type, **options)
        return key + (self.env.company.timesheet_encode_uom_id,)

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        if view_type in ["pivot", "graph"] and self.env.company.timesheet_encode_uom_id == self.env.ref("uom.product_uom_day"):
            arch = self.env["account.analytic.line"]._apply_time_label(arch, related_model=self._name)
        return arch, view

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute(
            sql.SQL("CREATE or REPLACE VIEW {} as ({})").format(
                sql.Identifier(self._table),
                sql.SQL(self._table_query)
            )
        )

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_report
from . import timesheets_analysis_report

```

## File: security\hr_timesheet_security.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.module.category" id="base.module_category_services_timesheets">
            <field name="description">Helps you manage the timesheets.</field>
            <field name="sequence">13</field>
        </record>

        <record id="group_hr_timesheet_user" model="res.groups">
            <field name="name">User: own timesheets only</field>
            <field name="category_id" ref="base.module_category_services_timesheets"/>
            <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

        <record id="group_hr_timesheet_approver" model="res.groups">
            <field name="name">User: all timesheets</field>
            <field name="category_id" ref="base.module_category_services_timesheets"/>
            <field name="implied_ids" eval="[(4, ref('hr_timesheet.group_hr_timesheet_user'))]"/>
        </record>

        <record id="group_timesheet_manager" model="res.groups">
            <field name="name">Administrator</field>
            <field name="category_id" ref="base.module_category_services_timesheets"/>
            <field name="implied_ids" eval="[(4, ref('hr_timesheet.group_hr_timesheet_approver')), (4, ref('hr.group_hr_user'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('group_timesheet_manager'))]"/>
        </record>

        <record id="timesheet_line_rule_portal_user" model="ir.rule">
            <field name="name">account.analytic.line.timesheet.portal.user</field>
            <field name="model_id" ref="analytic.model_account_analytic_line"/>
            <field name="active">0</field>
            <field name="domain_force">[
                ('project_id', '!=', False),
                '|',
                    ('project_id.message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
                    ('task_id.message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
                ('project_id.privacy_visibility', '=', 'portal'),
                ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),
            ]</field>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="True"/>
            <field name="perm_unlink" eval="True"/>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        </record>

        <record id="timesheet_line_rule_user" model="ir.rule">
            <field name="name">account.analytic.line.timesheet.user</field>
            <field name="model_id" ref="analytic.model_account_analytic_line"/>
            <field name="domain_force">[
                ('user_id', '=', user.id),
                ('project_id', '!=', False),
                '|', '|',
                    ('project_id.privacy_visibility', '!=', 'followers'),
                    ('project_id.message_partner_ids', 'in', [user.partner_id.id]),
                    ('task_id.message_partner_ids', 'in', [user.partner_id.id])
            ]</field>
            <field name="groups" eval="[(4, ref('group_hr_timesheet_user'))]"/>
        </record>

        <record id="timesheet_line_rule_approver" model="ir.rule">
            <field name="name">account.analytic.line.timesheet.approver</field>
            <field name="model_id" ref="analytic.model_account_analytic_line" />
            <field name="domain_force">[
                ('project_id', '!=', False),
                '|',
                    ('project_id.privacy_visibility', '!=', 'followers'),
                    ('project_id.message_partner_ids', 'in', [user.partner_id.id])
            ]</field>
            <field name="groups" eval="[(4, ref('hr_timesheet.group_hr_timesheet_approver'))]" />
        </record>

        <record id="timesheet_line_rule_manager" model="ir.rule">
            <field name="name">account.analytic.line.timesheet.manager</field>
            <field name="model_id" ref="analytic.model_account_analytic_line"/>
            <field name="domain_force">[('project_id', '!=', False)]</field>
            <field name="groups" eval="[(4, ref('group_timesheet_manager')), (4, ref('project.group_project_manager'))]"/>
        </record>

        <record id="project.group_project_manager" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('hr_timesheet.group_hr_timesheet_approver'))]"/>
        </record>

        <record model="ir.rule" id="timesheets_analysis_report_comp_rule">
            <field name="name">Timesheets Analysis Report multi-company</field>
            <field name="model_id" ref="model_timesheets_analysis_report"/>
            <field name="domain_force">[('company_id', 'in', company_ids)]</field>
        </record>

        <record id="timesheet_analysis_report_user" model="ir.rule">
            <field name="name">Timesheets Analysis Report user</field>
            <field name="model_id" ref="model_timesheets_analysis_report"/>
            <field name="domain_force">[
                ('user_id', '=', user.id),
                '|', '|',
                    ('project_id.privacy_visibility', '!=', 'followers'),
                    ('project_id.message_partner_ids', 'in', [user.partner_id.id]),
                    ('task_id.message_partner_ids', 'in', [user.partner_id.id])
            ]</field>
            <field name="groups" eval="[(4, ref('group_hr_timesheet_user'))]"/>
        </record>

        <record id="timesheet_analysis_report_approver" model="ir.rule">
            <field name="name">Timesheets Analysis Report approver</field>
            <field name="model_id" ref="model_timesheets_analysis_report"/>
            <field name="domain_force">[
                '|',
                    ('project_id.privacy_visibility', '!=', 'followers'),
                    ('project_id.message_partner_ids', 'in', [user.partner_id.id])
            ]</field>
            <field name="groups" eval="[(4, ref('group_hr_timesheet_approver'))]"/>
        </record>

        <record id="timesheet_analysis_report_manager" model="ir.rule">
            <field name="name">Timesheets Analysis Report manager</field>
            <field name="model_id" ref="model_timesheets_analysis_report"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_timesheet_manager')), (4, ref('project.group_project_manager'))]"/>
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
access_timesheets_analysis_report_manager,timesheets.analysis.report,model_timesheets_analysis_report,hr_timesheet.group_timesheet_manager,1,0,0,0
access_timesheets_analysis_report_user,timesheets.analysis.report,model_timesheets_analysis_report,hr_timesheet.group_hr_timesheet_user,1,0,0,0
access_hr_employee_delete_wizard,hr.employee.delete.wizard,model_hr_employee_delete_wizard,hr.group_hr_user,1,1,1,0

```

## File: security\ir.model.access.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <data noupdate="1">

        <record id="access_account_analytic_line_portal_user" model="ir.model.access">
            <field name="name">analytic.account.analytic.line.timesheet.portal.user</field>
            <field name="model_id" ref="analytic.model_account_analytic_line"/>
            <field name="group_id" ref="base.group_portal"/>
            <field name="active">0</field>
            <field name="perm_read">1</field>
            <field name="perm_write">0</field>
            <field name="perm_create">0</field>
            <field name="perm_unlink">0</field>
        </record>

    </data>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M45.445 23.222A22.222 22.222 0 1 0 12.11 42.467l11.111-19.245h22.223Z" fill="#FC868B"/><path d="M5.313 32.53A22.222 22.222 0 1 0 37.889 7.533L26.778 26.778 5.313 32.53Z" fill="#2EBCFA"/><path d="M23.221 45.444c12.274 0 22.223-9.95 22.223-22.223 0-5.23-1.807-10.039-4.832-13.835a22.128 22.128 0 0 0-13.835-4.831c-12.273 0-22.222 9.949-22.222 22.222 0 5.23 1.807 10.04 4.831 13.835a22.128 22.128 0 0 0 13.835 4.832Z" fill="#2D6388"/><path d="M7.719 7.303c.646-.63 1.33-1.22 2.05-1.768.227.056.445.161.639.316L26.58 18.796c1.82 1.456 1.946 4.192.297 5.84-1.649 1.647-4.387 1.521-5.845-.297L8.075 8.181a1.651 1.651 0 0 1-.356-.878Z" fill="#fff"/></svg>

```

## File: static\img\timesheet.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M16.376 45.5317C17.1026 33.5076 23.9747 21.4504 33.1645 13.7722C38.4136 9.4823 45.6679 6.72472 52.2221 10.3616C51.9341 10.1907 43.9663 5.15163 43.9745 5.2212C42.6941 4.60548 41.2935 4.26875 39.8761 4.17736C31.2311 3.84811 24.2539 10.041 19.251 16.5016C15.3733 21.5634 12.3934 27.347 10.6655 33.4905C8.28743 41.465 8.5365 52.7762 15.5077 56.6752C15.5075 56.6753 24.089 61.0976 24.089 61.0976C19.9257 58.925 15.8798 52.1542 16.376 45.5317Z" fill="#FBDBD0"/>
<path d="M36.8403 11.2289C25.5026 17.8293 16.3392 33.833 16.3762 46.9724C16.4131 60.1091 25.6363 65.4088 36.974 58.8084C48.3144 52.2066 57.4776 36.2028 57.4407 23.0662C57.4037 9.9268 48.1806 4.62689 36.8403 11.2289ZM36.809 55.0952C27.2055 60.6649 19.3665 56.1774 19.3352 45.0923C19.3039 34.0038 27.092 20.4531 36.6956 14.8834C46.3021 9.31192 54.1411 13.7993 54.1724 24.888C54.2037 35.9732 46.4155 49.5239 36.809 55.0952Z" fill="white"/>
<path d="M36.809 55.0952C27.2055 60.6649 19.3665 56.1774 19.3352 45.0923C19.3039 34.0038 27.092 20.4531 36.6956 14.8834C46.3021 9.31192 54.1411 13.7993 54.1724 24.888C54.2037 35.9732 46.4155 49.5239 36.809 55.0952Z" fill="white"/>
<path d="M48.2282 13.2289C50.3568 15.6328 51.9008 19.8191 51.9135 24.0517C51.9476 35.3917 43.4906 49.2021 33.026 54.8997C29.727 56.6958 26.2628 56.199 23.4779 55.7818C27.781 58.7178 31.8169 57.9904 36.809 55.0952C46.4156 49.5239 54.2037 35.9732 54.1724 24.888C54.1565 19.253 52.658 15.3425 48.2282 13.2289Z" fill="#C1DBF6"/>
<path d="M40.7406 35.8751L46.1874 38.6438L45.0732 40.4964L39.8762 37.7142L40.7406 35.8751Z" fill="#374874"/>
<path d="M35.1716 32.7948L31.8221 23.0592L32.8727 22.3906L36.2633 31.6678L35.1716 32.7948Z" fill="#374874"/>
<path d="M34.8618 37.2808C34.972 35.4566 36.0146 33.5669 37.4087 32.4021C38.205 31.7514 39.3785 31.1622 40.3728 31.7139C40.3291 31.688 39.0694 30.9524 39.0706 30.9629C38.8764 30.8695 38.6639 30.8184 38.4489 30.8046C37.1374 30.7546 35.9306 31.8146 35.1716 32.7948C34.5833 33.5626 34.1313 34.4401 33.8691 35.3721C33.5084 36.5818 33.5462 38.2978 34.6037 38.8893C34.6037 38.8893 35.799 39.5839 35.799 39.5839C35.1674 39.2543 34.7865 38.2854 34.8618 37.2808Z" fill="#374874"/>
<path d="M39.522 31.9525C40.3559 31.9525 40.8557 32.6377 40.8589 33.7854C40.8642 35.6538 39.4945 38.0257 37.8678 38.9648C37.4252 39.2204 36.9888 39.3555 36.6058 39.3555C35.7496 39.3555 35.2364 38.6846 35.2333 37.5609C35.228 35.6912 36.6342 33.2972 38.3039 32.3331C38.7352 32.0841 39.1564 31.9525 39.522 31.9525ZM39.522 31.4954C39.0861 31.4954 38.5947 31.6375 38.0754 31.9373C36.2942 32.9656 34.7703 35.5155 34.7761 37.5623C34.7802 38.9998 35.538 39.8126 36.6058 39.8126C37.0583 39.8126 37.5663 39.6667 38.0964 39.3607C39.8776 38.3324 41.3219 35.8308 41.3161 33.784C41.312 32.3339 40.5811 31.4954 39.522 31.4954Z" fill="#374874"/>
<path d="M38.0805 33.7538C37.1711 34.2788 36.4336 35.5561 36.4366 36.6012C36.4396 37.6461 37.1818 38.0692 38.0913 37.5441C39.0007 37.019 39.7382 35.7418 39.7352 34.6967C39.7322 33.6516 38.99 33.2287 38.0805 33.7538Z" fill="#374874"/>
<path d="M39.2165 4.16477C39.4349 4.16477 39.6556 4.16903 39.8762 4.17739C41.2935 4.2688 42.6941 4.60551 43.9745 5.22137C43.9745 5.22091 43.9748 5.22071 43.9755 5.22071C44.0484 5.22071 48.222 7.84772 50.6054 9.34727C54.7836 11.2764 57.421 16.0536 57.4407 23.0662C57.4776 36.2029 48.3144 52.2066 36.974 58.8084C33.6668 60.7338 30.5403 61.6451 27.7689 61.6451C26.3434 61.6451 25.0119 61.4039 23.7981 60.9357C23.895 60.9913 23.9919 61.0471 24.089 61.0977C24.0877 61.0971 23.9331 61.0175 23.6716 60.8826C23.1017 60.6536 22.5592 60.3733 22.0449 60.0443C19.5068 58.7363 15.5076 56.6753 15.5077 56.6753C8.53651 52.7763 8.28744 41.4651 10.6655 33.4906C12.3934 27.3472 15.3733 21.5634 19.251 16.5017C24.1262 10.2059 30.8751 4.1642 39.2165 4.16477ZM39.2152 3.25049C32.0094 3.25049 25.0493 7.52057 18.5281 15.9419C14.5215 21.1719 11.5 27.1508 9.78746 33.2357C8.46776 37.6646 8.06266 42.5881 8.67615 46.7447C9.42938 51.8483 11.6374 55.5582 15.0614 57.4733L15.0654 57.4661C15.1426 57.5157 15.214 57.5525 15.2656 57.5791L15.7534 57.8305L17.4057 58.6821L21.589 60.8379C22.129 61.1803 22.7008 61.4749 23.29 61.7145L23.6583 61.9043L23.6695 61.9101C23.8036 61.9792 23.9468 62.012 24.088 62.012C24.0938 62.012 24.0996 62.012 24.1053 62.0119C25.2531 62.3754 26.4823 62.5593 27.7689 62.5593C30.8074 62.5593 34.0592 61.5631 37.4339 59.5986C43.055 56.3263 48.3229 50.7305 52.2672 43.8423C56.2112 36.9543 58.3732 29.5751 58.3549 23.0637C58.335 15.9992 55.6712 10.711 51.0434 8.54272L50.9002 8.45266C49.6953 7.69451 48.1026 6.6925 46.8023 5.88C45.2748 4.92553 44.6738 4.54999 44.3685 4.40221L44.3708 4.39753C43.0224 3.74887 41.53 3.36795 39.935 3.26505C39.9269 3.2645 39.9189 3.26416 39.9109 3.26381C39.6806 3.25499 39.447 3.25053 39.2165 3.25053L39.2152 3.25049Z" fill="#374874"/>
<path d="M44.4976 12.4896C50.21 12.489 54.1502 17.0316 54.1723 24.888C54.2036 35.9732 46.4155 49.5239 36.809 55.0952C34.0095 56.7188 31.3597 57.4879 29.0098 57.4879C23.2984 57.4879 19.3573 52.946 19.3351 45.0923C19.3039 34.0038 27.0919 20.453 36.6955 14.8834C39.4959 13.2593 42.1472 12.4899 44.4976 12.4896ZM44.4988 12.0325V12.4896L44.4985 12.0325C41.9771 12.0327 39.275 12.8589 36.4662 14.4879C26.7364 20.1308 18.8463 33.8604 18.878 45.0936C18.889 48.9926 19.8589 52.2528 21.6827 54.5217C23.4831 56.7612 26.0167 57.945 29.0098 57.945C31.5291 57.945 34.2303 57.1192 37.0383 55.4906C46.7698 49.8469 54.6612 36.118 54.6295 24.8866C54.6185 20.9867 53.6488 17.7259 51.8254 15.4567C50.0252 13.2165 47.4917 12.0325 44.4988 12.0325Z" fill="#374874"/>
</svg>

```

## File: static\src\components\progress_bar\project_task_progress_bar_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { progressBarField, ProgressBarField } from "@web/views/fields/progress_bar/progress_bar_field";

export class ProjectTaskProgressBarField extends ProgressBarField {
    get progressBarColorClass() {
        if (this.currentValue > this.maxValue) {
            return super.progressBarColorClass;
        }

        return this.currentValue < 80 ? "bg-success" : "bg-warning";
    }
}

export const projectTaskProgressBarField = {
    ...progressBarField,
    component: ProjectTaskProgressBarField,
};

registry.category("fields").add("project_task_progressbar", projectTaskProgressBarField);

```

## File: static\src\components\task_with_hours\task_with_hours.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { Many2OneField, many2OneField } from "@web/views/fields/many2one/many2one_field";
import { onWillStart } from "@odoo/owl";

export class TaskWithHours extends Many2OneField {
    setup() {
        super.setup();
        onWillStart(this.onWillStart);
    }

    async onWillStart() { }

    canCreate() {
        return Boolean(this.context.default_project_id);
    }

    /**
     * @override
     */
    get displayName() {
        const displayName = super.displayName;
        return displayName ? displayName.split('\u00A0')[0] : displayName;
    }

    /**
     * @override
     */
    get context() {
        return { ...super.context, hr_timesheet_display_remaining_hours: true };
    }

    /**
     * @override
     */
    get Many2XAutocompleteProps() {
        const props = super.Many2XAutocompleteProps;
        if (!this.canCreate()) {
            props.quickCreate = null;
        }
        return props;
    }

    /**
     * @override
     */
    computeActiveActions(props) {
        super.computeActiveActions(props);
        const activeActions = this.state.activeActions;
        activeActions.create = activeActions.create && this.canCreate(props);
        activeActions.createEdit = activeActions.createEdit && this.canCreate(props);
    }

}

export const taskWithHours = {
    ...many2OneField,
    component: TaskWithHours,
};

registry.category("fields").add("task_with_hours", taskWithHours);

```

## File: static\src\components\timesheet_uom\timesheet_uom.js

```javascript
/** @odoo-module */

import { useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";
import { FloatFactorField } from "@web/views/fields/float_factor/float_factor_field";
import { FloatToggleField } from "@web/views/fields/float_toggle/float_toggle_field";
import { FloatTimeField } from "@web/views/fields/float_time/float_time_field";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

import { Component } from "@odoo/owl";

export class TimesheetUOM extends Component {
    static props = {
        ...standardFieldProps,
    };

    static template = "hr_timesheet.TimesheetUOM";

    static components = { FloatFactorField, FloatToggleField, FloatTimeField };

    setup() {
        this.timesheetUOMService = useService("timesheet_uom");
    }

    get timesheetComponent() {
        return this.timesheetUOMService.getTimesheetComponent();
    }

    get timesheetComponentProps() {
        return this.timesheetUOMService.getTimesheetComponentProps(this.props);
    }
}

export const timesheetUOM = {
    component: TimesheetUOM,
};

registry.category("fields").add("timesheet_uom", timesheetUOM);

```

## File: static\src\components\timesheet_uom\timesheet_uom.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>

    <t t-name="hr_timesheet.TimesheetUOM">
        <t t-component="timesheetComponent" t-props="timesheetComponentProps"/>
    </t>

</templates>

```

## File: static\src\components\timesheet_uom_no_toggle\timesheet_uom_no_toggle.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";

import { TimesheetUOM, timesheetUOM } from "../timesheet_uom/timesheet_uom";

export class TimesheetUOMNoToggle extends TimesheetUOM {
    get timesheetComponent() {
        if (this.timesheetUOMService.timesheetWidget === "float_toggle") {
            return this.timesheetUOMService.getTimesheetComponent("float_factor");
        }
        return super.timesheetComponent;
    }
}

// As FloatToggleField won't be used by TimesheetUOMNoToggle, we remove it from the components that we get from TimesheetUOM.
delete TimesheetUOMNoToggle.components.FloatToggleField;

export const timesheetUOMNoToggle = {
    ...timesheetUOM,
    component: TimesheetUOMNoToggle,
};

registry.category("fields").add("timesheet_uom_no_toggle", timesheetUOMNoToggle);

```

## File: static\src\services\timesheet_uom_service.js

```javascript
/** @odoo-module */

import { session } from "@web/session";
import { registry } from "@web/core/registry";
import { formatFloatTime, formatFloatFactor } from "@web/views/fields/formatters";
import { formatFloat } from "@web/core/utils/numbers";
import { FloatFactorField } from "@web/views/fields/float_factor/float_factor_field";

export const timesheetUOMService = {
    dependencies: ["company"],
    start(env, { company }) {
        const service = {
            get timesheetUOMId() {
                return company.currentCompany.timesheet_uom_id;
            },
            get timesheetWidget() {
                let timesheet_widget = "float_factor";
                if (session.uom_ids && this.timesheetUOMId in session.uom_ids) {
                    timesheet_widget = session.uom_ids[this.timesheetUOMId].timesheet_widget;
                }
                return timesheet_widget;
            },
            getTimesheetComponent(widgetName = this.timesheetWidget) {
                return registry.category("fields").get(widgetName, { component: FloatFactorField })
                    .component;
            },
            getTimesheetComponentProps(props) {
                const factorDependantComponents = ["float_toggle", "float_factor"];
                return factorDependantComponents.includes(this.timesheetWidget)
                    ? this._getFactorCompanyDependentProps(props)
                    : props;
            },
            _getFactorCompanyDependentProps(props) {
                const factor = company.currentCompany.timesheet_uom_factor || props.factor;
                return { ...props, factor };
            },
            get formatter() {
                if (this.timesheetWidget === "float_time") {
                    return formatFloatTime;
                }
                const factor = company.currentCompany.timesheet_uom_factor || 1;
                if (this.timesheetWidget === "float_toggle") {
                    return (value, options = {}) => formatFloat(value * factor, options);
                }
                return (value, options = {}) =>
                    formatFloatFactor(value, Object.assign({ factor }, options));
            },
        };
        if (!registry.category("formatters").contains("timesheet_uom")) {
            registry.category("formatters").add("timesheet_uom", service.formatter);
        }
        if (!registry.category("formatters").contains("timesheet_uom_no_toggle")) {
            registry.category("formatters").add("timesheet_uom_no_toggle", service.formatter);
        }
        return service;
    },
};

registry.category("services").add("timesheet_uom", timesheetUOMService);

```

## File: static\src\views\timesheet_graph\timesheet_graph_model.js

```javascript
/** @odoo-module **/

import { ProjectTaskGraphModel } from "@project/views/project_task_graph/project_task_graph_model";

const FIELDS = [
    'unit_amount', 'effective_hours', 'allocated_hours', 'remaining_hours', 'total_hours_spent', 'subtask_effective_hours',
    'overtime', 'number_hours', 'difference', 'timesheet_unit_amount'
];

export class hrTimesheetGraphModel extends ProjectTaskGraphModel {
    /**
     * @override
     */
    setup(params, services) {
        super.setup(...arguments);
        this.companyService = services.company;
    }

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Override processDataPoints to take into account the analytic line uom.
     * @override
     */
    _getProcessedDataPoints() {
        const currentCompany = this.companyService.currentCompany;
        const factor = currentCompany.timesheet_uom_factor || 1;
        if (factor !== 1 && FIELDS.includes(this.metaData.measure)) {
            // recalculate the Duration values according to the timesheet_uom_factor
            for (const dataPt of this.dataPoints) {
                dataPt.value *= factor;
            }
        }
        return super._getProcessedDataPoints(...arguments);
    }
}
hrTimesheetGraphModel.services = [...ProjectTaskGraphModel.services, "company"];

```

## File: static\src\views\timesheet_graph\timesheet_graph_view.js

```javascript
/** @odoo-module **/

import { projectTaskGraphView } from "@project/views/project_task_graph/project_task_graph_view";
import { hrTimesheetGraphModel } from "./timesheet_graph_model";
import { registry } from "@web/core/registry";

const viewRegistry = registry.category("views");

export const hrTimesheetGraphView = {
  ...projectTaskGraphView,
  Model: hrTimesheetGraphModel,
};

viewRegistry.add("hr_timesheet_graphview", hrTimesheetGraphView);

```

## File: views\hr_department_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_department_view_kanban" model="ir.ui.view">
        <field name="name">hr.department.kanban.inherit</field>
        <field name="model">hr.department</field>
        <field name="inherit_id" ref="hr.hr_department_view_kanban"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <a name="%(act_hr_timesheet_report)d" type="action"
                        groups="hr_timesheet.group_timesheet_manager" class="dropdown-item"
                        context="{ 'search_default_department_id': [id], 'default_department_id': id}">
                        Timesheets
                    </a>
                </xpath>
            </data>
        </field>
    </record>
</odoo>

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="timesheet_action_view_from_employee_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_timesheet_line_form"/>
        <field name="act_window_id" ref="timesheet_action_from_employee"/>
    </record>

    <record id="hr_employee_view_form_inherit_timesheet" model="ir.ui.view">
        <field name="name">hr.employee.form.timesheet</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr_hourly_cost.view_employee_form"/>
        <field name="priority" eval="40"/>
        <field name="arch" type="xml">
            <xpath expr="/form" position="attributes">
                <attribute name="delete">0</attribute>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="has_timesheet" groups="hr_timesheet.group_hr_timesheet_user" invisible="1"/>
                <button invisible="not has_timesheet" class="oe_stat_button" type="object" name="action_timesheet_from_employee" icon="fa-calendar" groups="hr_timesheet.group_hr_timesheet_user">
                    <div class="o_stat_info">
                        <span class="o_stat_text">Timesheets</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

     <record id="view_employee_tree_inherit_timesheet" model="ir.ui.view">
        <field name="name">hr.employee.tree.timesheet</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"/>
        <field name="arch" type="xml">
            <xpath expr="/tree" position="attributes">
                <attribute name="delete">0</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_department_view_kanban" model="ir.ui.view">
        <field name="name">hr.department.kanban.inherit</field>
        <field name="model">hr.department</field>
        <field name="inherit_id" ref="hr.hr_department_view_kanban"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <a name="%(act_hr_timesheet_report)d" type="action"
                        groups="hr_timesheet.group_timesheet_manager" class="dropdown-item"
                        context="{ 'search_default_department_id': [id], 'default_department_id': id}">
                        Timesheets
                    </a>
                </xpath>
            </data>
        </field>
    </record>

    <record id="unlink_employee_action" model="ir.actions.server">
        <field name="name">Delete</field>
        <field name="model_id" ref="hr.model_hr_employee"/>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="binding_view_types">form,list</field>
        <field name="state">code</field>
        <field name="code">action = records.action_unlink_wizard()</field>
    </record>
</odoo>

```

## File: views\hr_timesheet_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="timesheet_menu_root"
              name="Timesheets"
              sequence="75"
              groups="group_hr_timesheet_user"
              web_icon="hr_timesheet,static/description/icon_timesheet.png"
    >
        <menuitem id="timesheet_menu_activity_user"
                  name="My Timesheets"
                  groups="group_hr_timesheet_user"
                  action="act_hr_timesheet_line"
        />
        <menuitem id="menu_hr_time_tracking"
                  name="Timesheets"
                  groups="group_hr_timesheet_approver"
                  sequence="5"
        >
            <menuitem id="timesheet_menu_activity_mine"
                      name="My Timesheets"
                      groups="group_hr_timesheet_approver"
                      action="act_hr_timesheet_line"
            />
            <menuitem id="timesheet_menu_activity_all"
                      name="All Timesheets"
                      action="timesheet_action_all"
                      groups="hr_timesheet.group_hr_timesheet_approver"
            />
        </menuitem>
        <menuitem id="menu_timesheets_reports"
                  name="Reporting"
                  groups="group_hr_timesheet_approver"
                  sequence="99"
        >
            <menuitem id="menu_timesheets_reports_timesheet"
                      name="Timesheets"
                      sequence="10"
            >
                <menuitem id="menu_hr_activity_analysis"
                          action="act_hr_timesheet_report"
                          groups="hr_timesheet.group_hr_timesheet_approver"
                          name="By Employee"
                          sequence="10"
                />
                <menuitem id="timesheet_menu_report_timesheet_by_project"
                          action="timesheet_action_report_by_project"
                          name="By Project"
                          sequence="15"
                />
                <menuitem id="timesheet_menu_report_timesheet_by_task"
                          action="timesheet_action_report_by_task"
                          name="By Task"
                          sequence="20"
                />
            </menuitem>
        </menuitem>
        <menuitem id="hr_timesheet_menu_configuration"
                  name="Configuration"
                  action="hr_timesheet_config_settings_action"
                  groups="base.group_system"
                  sequence="100"
        />
    </menuitem>
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

    <template id="portal_my_home_timesheet" name="Show Timesheets" customize_show="True"  inherit_id="portal.portal_my_home" priority="45">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="before">
            <t t-set="portal_service_category_enable" t-value="True"/>
        </xpath>
        <div id="portal_service_category" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="icon" t-value="'/hr_timesheet/static/img/timesheet.svg'"/>
                <t t-set="title">Timesheets</t>
                <t t-set="url" t-value="'/my/timesheets'"/>
                <t t-set="text">Review all timesheets related to your projects</t>
                <t t-set="placeholder_count" t-value="'timesheet_count'"/>
            </t>
        </div>
    </template>

    <template id="portal_my_timesheets" name="My Timesheets">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Timesheets</t>
            </t>
            <t t-if="not grouped_timesheets">
                <div class="alert alert-warning" role="alert">
                    There are no timesheets.
                </div>
            </t>
            <t t-if="grouped_timesheets">
                <t t-call="portal.portal_table">
                    <thead>
                        <tr>
                            <th t-if="not groupby == 'date'">Date</th>
                            <th t-if="not groupby == 'employee'">Employee</th>
                            <th t-if="not groupby == 'project'">Project</th>
                            <th t-if="not groupby == 'task'">Task</th>
                            <th>Description</th>
                            <th t-if="is_uom_day" class="text-end" t-att-colspan="2 if groupby != 'none' else 0">Days Spent</th>
                            <th t-else="" class="text-end" t-att-colspan="2 if groupby != 'none' else 0">Hours Spent</th>
                        </tr>
                    </thead>
                    <t t-foreach="grouped_timesheets" t-as="timesheets_with_hours">
                        <t t-set="timesheets" t-value="timesheets_with_hours[0]"/>
                        <t t-set="hours_spent" t-value="timesheets_with_hours[1]"/>
                        <tbody style="font-size: 0.8rem">
                            <tr t-if="not groupby =='none'" class="table-light">
                                <t t-if="groupby == 'project'">
                                    <th t-if="groupby == 'project'" colspan="5">
                                        <span t-field="timesheets[0].project_id.name"/>
                                    </th>
                                    <th colspan="1" class="text-end text-muted">
                                        <t t-if="is_uom_day">
                                            Total: <span class="text-muted" t-esc="timesheets._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                                        </t>
                                        <t t-else="">
                                            Total: <span class="text-muted" t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                                        </t>
                                    </th>
                                </t>
                                <t t-elif="groupby == 'task'">
                                    <th colspan="5">
                                        <span t-field="timesheets[0].task_id.name"/>
                                    </th>
                                    <th colspan="1" class="text-end text-muted">
                                        <t t-if="is_uom_day">
                                            Total: <span t-esc="timesheets._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                                        </t>
                                        <t t-else="">
                                            Total: <span t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                                        </t>
                                    </th>
                                </t>
                                <t t-elif="groupby == 'date'">
                                    <th colspan="5">
                                        <span t-field="timesheets[0].date"/>
                                    </th>
                                    <th colspan="1" class="text-end text-muted">
                                        <t t-if="is_uom_day">
                                            Total: <span t-esc="timesheets._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                                        </t>
                                        <t t-else="">
                                            Total: <span t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                                        </t>
                                    </th>
                                </t>
                                <t t-elif="groupby == 'employee'">
                                    <th colspan="5">
                                        <span t-field="timesheets[0].employee_id.name"/>
                                    </th>
                                    <th colspan="1" class="text-end text-muted">
                                        <t t-if="is_uom_day">
                                            Total: <span t-esc="timesheets._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                                        </t>
                                        <t t-else="">
                                            Total: <span t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                                        </t>
                                    </th>
                                </t>
                            </tr>
                            <tr t-else="">
                                <div style="text-align: right;" class="me-2 mb-1 text-muted">
                                    <t t-if="is_uom_day">
                                        Total: <span t-esc="timesheets._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                                    </t>
                                    <t t-else="">
                                        Total: <span t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                                    </t>
                                </div>
                            </tr>
                        </tbody>
                        <tbody style="font-size: 0.8rem">
                            <t t-foreach="timesheets" t-as="timesheet">
                                <tr>
                                    <td t-if="not groupby == 'date'"><span t-field="timesheet.date" t-options='{"widget": "date"}'/></td>
                                    <td t-if="not groupby == 'employee'"><span t-field="timesheet.employee_id" t-att-title="timesheet.employee_id.display_name" /></td>
                                    <td t-if="not groupby == 'project'"><span t-field="timesheet.project_id" t-att-title="timesheet.project_id.display_name"/></td>
                                    <td t-if="not groupby == 'task'"><span t-field="timesheet.task_id" t-att-title="timesheet.task_id.display_name"/></td>
                                    <td><span t-esc="timesheet.name" t-att-title="timesheet.name"/></td>
                                    <td class="text-end" t-att-colspan="2 if groupby != 'none' else 0">
                                        <span t-if="is_uom_day" t-esc="timesheet._get_timesheet_time_day()" t-options='{"widget": "timesheet_uom"}'/>
                                        <span t-else="" t-field="timesheet.unit_amount" t-options='{"widget": "float_time"}'/>
                                    </td>
                                </tr>
                            </t>
                        </tbody>
                    </t>
                </t>
            </t>
        </t>
    </template>

    <template id="portal_timesheet_table" name="Portal Timesheet Table">
        <table class="o_portal_my_doc_table table table-sm">
            <thead>
              <tr>
                <th>Date</th>
                <th>Employee</th>
                <th>Description</th>
                <th t-if="is_uom_day" class="text-end">Days Spent</th>
                <th t-else="" class="text-end">Hours Spent</th>
              </tr>
            </thead>
            <tr t-foreach="timesheets" t-as="timesheet">
                <td><t t-esc="timesheet.date" t-options='{"widget": "date"}'/></td>
                <td t-attf-title="#{timesheet.employee_id.name}"><t t-esc="timesheet.employee_id.name"/></td>
                <td><t t-esc="timesheet.name"/></td>
                <td class="text-end">
                    <span t-if="is_uom_day" t-esc="timesheet._get_timesheet_time_day()" t-options='{"widget": "timesheet_uom"}'/>
                    <span t-else="" t-field="timesheet.unit_amount" t-options='{"widget": "float_time"}'/>
                </td>
            </tr>
        </table>
        <div class="container_subtotal">
            <div class="row justify-content-end">
                <div t-attf-class="{{'col-auto' if report_type != 'html' else 'col-sm-2'}}">
                    <table class="table table-sm">
                        <tr>
                            <t t-set="timesheets_amount" t-value="round(sum(timesheets.mapped('unit_amount')), 2)"></t>
                            <td><strong>Total: </strong></td>
                            <td class="text-end">
                                <t t-if="is_uom_day">
                                    <span t-esc="timesheets._convert_hours_to_days(timesheets_amount)" t-options="{'widget': 'timesheet_uom'}"/>
                                </t>
                                <t t-else="">
                                    <span t-esc="timesheets_amount" t-options='{"widget": "float_time"}'/>
                                </t>
                            </td>
                        </tr>
                        <t t-set="timesheets_by_subtask_amount" t-value="round(sum(sum(timesheet_by_subtask.mapped('unit_amount') or 0.0) for timesheet_by_subtask in timesheets_by_subtask.values()), 2) or 0.0"></t>
                        <tr>
                            <div t-if="timesheets_by_subtask">
                                <t t-if="is_uom_day">
                                    <td><strong>Days recorded on sub-tasks: </strong></td>
                                    <td class="text-end">
                                        <span t-esc="timesheets._convert_hours_to_days(timesheets_by_subtask_amount)" t-options='{"widget": "timesheet_uom"}'/>
                                    </td>
                                </t>
                                <t t-else="">
                                    <td><strong>Hours recorded on sub-tasks: </strong></td>
                                    <td class="text-end">
                                        <span t-esc="timesheets_by_subtask_amount" t-options='{"widget": "float_time"}'/>
                                    </td>
                                </t>
                            </div>
                        </tr>
                        <t t-set="allocated_time" t-value="task.allocated_hours"></t>
                        <tr t-attf-class="{{task.remaining_hours &lt; 0 and 'text-danger' or ''}}">
                            <div t-if="allocated_time > 0" name="allocated_time">
                                <t t-if="is_uom_day">
                                    <td><strong>Remaining Days: </strong></td>
                                    <td class="text-end">
                                        <span t-esc="timesheets._convert_hours_to_days(allocated_time - timesheets_amount - timesheets_by_subtask_amount)" t-options='{"widget": "timesheet_uom"}'/>
                                    </td>
                                </t>
                                <t t-else="">
                                    <td><strong>Remaining Hours: </strong></td>
                                    <td class="text-end">
                                        <span t-esc="allocated_time - timesheets_amount - timesheets_by_subtask_amount" t-options='{"widget": "float_time"}'/>
                                    </td>
                                </t>
                            </div>
                        </tr>
                    </table>
                </div>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\hr_timesheet_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="hr_timesheet_line_tree" model="ir.ui.view">
            <field name="name">account.analytic.line.tree.hr_timesheet</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <tree editable="bottom" string="Timesheet Activities" sample="1" decoration-muted="readonly_timesheet == True">
                    <field name="readonly_timesheet" column_invisible="True"/>
                    <field name="date" readonly="readonly_timesheet"/>
                    <field name="employee_id" column_invisible="True" readonly="readonly_timesheet"/>
                    <field name="project_id" options="{'no_create_edit': True}" required="1" readonly="readonly_timesheet"
                        context="{'search_default_my_projects': True}"/>
                    <field name="task_id" optional="show" options="{'no_create_edit': True}" widget="task_with_hours"
                        context="{'default_project_id': project_id, 'search_default_my_tasks': True, 'search_default_open_tasks': True}"
                        readonly="readonly_timesheet"/>
                    <field name="name" optional="show" required="0" readonly="readonly_timesheet"/>
                    <field name="unit_amount" optional="show" widget="timesheet_uom" sum="Total" readonly="readonly_timesheet"
                        decoration-danger="unit_amount &gt; 24 or unit_amount &lt; 0" decoration-muted="unit_amount == 0"/>
                    <field name="company_id" column_invisible="True"/>
                    <field name="user_id" column_invisible="True"/>
                </tree>
            </field>
        </record>

        <record id="hr_timesheet_line_portal_tree" model="ir.ui.view">
            <field name="name">portal.hr_timesheet.account.analytic.line.tree</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="hr_timesheet_line_tree"/>
            <field name="mode">primary</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="edit">0</attribute>
                    <attribute name="create">0</attribute>
                    <attribute name="delete">0</attribute>
                </xpath>
                <xpath expr="//field[@name='task_id']" position="attributes">
                    <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
                </xpath>
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
                    <attribute name="column_invisible">0</attribute>
                    <attribute name="required">1</attribute>
                    <attribute name="widget">many2one_avatar_employee</attribute>
                    <attribute name="context">{'active_test': True}</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_hr_timesheet_line_pivot" model="ir.ui.view">
            <field name="name">account.analytic.line.pivot</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <pivot string="Timesheets" sample="1">
                    <field name="employee_id" type="row"/>
                    <field name="date" interval="month" type="col"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                    <field name="amount" string="Timesheet Costs"/>
                </pivot>
            </field>
        </record>

        <record id="view_my_timesheet_line_pivot" model="ir.ui.view">
            <field name="name">account.analytic.line.pivot</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <pivot string="Timesheet" sample="1">
                    <field name="date" interval="week" type="row"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                    <field name="amount" string="Timesheet Costs"/>
                </pivot>
            </field>
        </record>

        <record id="view_hr_timesheet_line_graph" model="ir.ui.view">
            <field name="name">account.analytic.line.graph</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <graph string="Timesheets" sample="1" js_class="hr_timesheet_graphview">
                    <field name="task_id"/>
                    <field name="project_id"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                    <field name="amount" string="Timesheet Costs"/>
                </graph>
            </field>
        </record>

        <!-- For My Timesheet view, groups by week then project -->
        <record id="view_hr_timesheet_line_graph_my" model="ir.ui.view">
            <field name="name">account.analytic.line.graph</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <graph string="Timesheet" sample="1" js_class="hr_timesheet_graphview">
                    <field name="date" interval="week"/>
                    <field name="project_id"/>
                    <field name="amount" type="measure" string="Timesheet Costs"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </graph>
            </field>
        </record>

        <!-- For Other Timesheet view, groups by employee then project -->
        <record id="view_hr_timesheet_line_graph_all" model="ir.ui.view">
            <field name="name">account.analytic.line.graph</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <graph string="Timesheet" sample="1" js_class="hr_timesheet_graphview">
                    <field name="employee_id"/>
                    <field name="project_id"/>
                    <field name="amount" type="measure" string="Timesheet Costs"/>
                    <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                </graph>
            </field>
        </record>

        <record id="view_hr_timesheet_line_graph_by_employee" model="ir.ui.view">
            <field name="name">account.analytic.line.graph.by.employee</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="hr_timesheet.view_hr_timesheet_line_graph_all"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <field name="project_id" position="replace"/>
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
                                <field name="readonly_timesheet" invisible="1"/>
                                <field name="project_id" options="{'no_create_edit': True}"
                                    context="{'search_default_my_projects': True}"
                                    required="1"
                                    readonly="readonly_timesheet"/>
                                <field name="task_id" widget="task_with_hours" options="{'no_create_edit': True}"
                                    context="{'default_project_id': project_id, 'search_default_my_tasks': True, 'search_default_open_tasks': True}"
                                    readonly="readonly_timesheet"/>
                                <field name="company_id" groups="base.group_multi_company" invisible="1"/>
                            </group>
                            <group>
                                <field name="date" readonly="readonly_timesheet"/>
                                <field name="amount" invisible="1"/>
                                <field name="unit_amount" widget="timesheet_uom" decoration-danger="unit_amount &gt; 24"
                                    readonly="readonly_timesheet" decoration-muted="unit_amount == 0"/>
                                <field name="currency_id" invisible="1"/>
                                <field name="company_id" invisible="1"/>
                            </group>
                        </group>
                        <field name="name" placeholder="Describe your activity" widget="text" nolabel="1" required="0" readonly="readonly_timesheet"/>
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
                <xpath expr="//field[@name='date']" position="before">
                    <field name="employee_id" groups="hr_timesheet.group_hr_timesheet_approver" widget="many2one_avatar_employee"
                        required="1"
                        readonly="readonly_timesheet" context="{'active_test': True}"/>
                    <field name="user_id" invisible="1" groups="hr_timesheet.group_hr_timesheet_approver"/>
                </xpath>
            </field>
        </record>

        <record id="hr_timesheet_line_search" model="ir.ui.view">
            <field name="name">account.analytic.line.search</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="analytic.view_account_analytic_line_filter"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='month']" position="before">
                    <filter name="mine" string="My Timesheets" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                </xpath>
                <xpath expr="//group[@name='groupby']" position="before">
                    <field name="employee_id"/>
                    <field name="project_id"/>
                    <field name="task_id"/>
                    <field name="parent_task_id"/>
                    <field name="department_id"/>
                    <field name="manager_id"/>
                </xpath>
                <xpath expr="//group[@name='groupby']" position="inside">
                    <filter string="Project" name="groupby_project" domain="[]" context="{'group_by': 'project_id'}"/>
                    <filter string="Parent Task" name="groupby_parent_task" domain="[]" context="{'group_by': 'parent_task_id'}"/>
                    <filter string="Task" name="groupby_task" domain="[]" context="{'group_by': 'task_id'}"/>
                    <filter string="Department" name="groupby_department" domain="[]" context="{'group_by': 'department_id'}"/>
                    <filter string="Manager" name="groupby_manager" domain="[]" context="{'group_by': 'manager_id'}"/>
                    <filter string="Employee" name="groupby_employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                </xpath>
            </field>
        </record>

        <record id="timesheet_view_form_portal_user" model="ir.ui.view">
            <field name="name">account.analytic.line.form</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="hr_timesheet.timesheet_view_form_user"/>
            <field name="mode">primary</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='employee_id']" position="attributes">
                    <attribute name="required">1</attribute>
                    <attribute name="widget">many2one_avatar_employee</attribute>
                    <attribute name="context">{'active_test': True}</attribute>
                    <attribute name="options">{'no_open': True}</attribute>
                    <attribute name="readonly">1</attribute>
                </xpath>
                <xpath expr="//field[@name='project_id']" position="attributes">
                    <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
                    <attribute name="readonly">1</attribute>
                </xpath>
                <xpath expr="//field[@name='task_id']" position="attributes">
                    <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
                    <attribute name="readonly">1</attribute>
                </xpath>
            </field>
        </record>

        <record id="hr_timesheet_line_my_timesheet_search" model="ir.ui.view">
            <field name="name">view.search.my.timesheet.menu</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="hr_timesheet_line_search"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <field name="employee_id" position="replace"/>
                <field name="department_id" position="replace"/>
                <field name="manager_id" position="replace"/>
                <filter name="mine" position="replace"/>
                <filter name="groupby_department" position="replace"/>
                <filter name="groupby_manager" position="replace"/>
                <filter name="groupby_employee" position="replace"/>
            </field>
        </record>

        <record id="view_kanban_account_analytic_line" model="ir.ui.view">
            <field name="name">account.analytic.line.kanban</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1">
                    <field name="date"/>
                    <field name="employee_id"/>
                    <field name="user_id"/>
                    <field name="company_id"/>
                    <field name="name"/>
                    <field name="project_id"/>
                    <field name="task_id" context="{'default_project_id': project_id}" domain="[('project_id', '=', project_id)]"/>
                    <field name="unit_amount" widget="timesheet_uom"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click o_kanban_record_has_image_fill px-0 pb-0">
                                <div class="oe_kanban_details d-flex flex-column">
                                    <div class="o_kanban_record_top px-2 h-50">
                                        <img t-att-src="kanban_image('hr.employee', 'avatar_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value" class="o_image_40_cover float-start"/>
                                        <div class="o_kanban_record_headings ps-4 pe-2">
                                            <div class="text-truncate">
                                                <strong class="o_kanban_record_title">
                                                    <span t-esc="record.project_id.value" t-att-title="record.project_id.value"/>
                                                </strong>
                                            </div>
                                            <div class="text-truncate">
                                                <i><span t-esc="record.task_id.value" t-att-title="record.task_id.value"/></i>
                                            </div>
                                            <div class="text-truncate">
                                                <span t-esc="record.name.value" t-att-title="record.name.value"/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="o_kanban_record_bottom d-block mx-2">
                                        <hr class="mt4 mb4"/>
                                        <span>
                                            <i class="fa fa-calendar me-1" role="img" aria-label="Date" title="Date"></i>
                                            <t t-esc="record.date.value"/>
                                        </span>
                                        <span class="float-end" name="duration">
                                            <strong>Duration: </strong><field name="unit_amount" widget="timesheet_uom" decoration-danger="unit_amount &gt; 24" decoration-muted="unit_amount == 0"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_kanban_account_analytic_line_portal_user" model="ir.ui.view">
            <field name="name">portal.account.analytic.line.kanban</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="hr_timesheet.view_kanban_account_analytic_line"/>
            <field name="mode">primary</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <xpath expr="//img[hasclass('o_image_40_cover')]" position="replace"/>
                <xpath expr="//div[hasclass('o_kanban_record_headings')]" position="attributes">
                    <attribute name="class">o_kanban_record_headings pe-2</attribute>
                </xpath>
            </field>
        </record>
        <!--
            Actions
        -->
        <record id="act_hr_timesheet_line" model="ir.actions.act_window">
            <field name="name">My Timesheets</field>
            <field name="res_model">account.analytic.line</field>
            <field name="view_mode">tree,form,kanban,pivot,graph</field>
            <field name="domain">[('project_id', '!=', False), ('user_id', '=', uid)]</field>
            <field name="context">{
                "search_default_week":1,
                "is_timesheet": 1,
            }</field>
            <field name="search_view_id" ref="hr_timesheet_line_my_timesheet_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No activities found. Let's start a new one!
              </p>
              <p>
                Track your working hours by projects every day and invoice this time to your customers.
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

        <record id="act_hr_timesheet_line_view_pivot" model="ir.actions.act_window.view">
            <field name="view_mode">pivot</field>
            <field name="sequence" eval="7"/>
            <field name="view_id" ref="view_my_timesheet_line_pivot"/>
            <field name="act_window_id" ref="act_hr_timesheet_line"/>
        </record>

        <record id="act_hr_timesheet_line_view_graph" model="ir.actions.act_window.view">
            <field name="view_mode">graph</field>
            <field name="sequence" eval="8"/>
            <field name="view_id" ref="view_hr_timesheet_line_graph_my"/>
            <field name="act_window_id" ref="act_hr_timesheet_line"/>
        </record>

        <record id="timesheet_action_task" model="ir.actions.act_window">
            <field name="name">Task's Timesheets</field>
            <field name="res_model">account.analytic.line</field>
            <field name="context">{
                'is_timesheet': 1,
            }</field>
            <field name="domain">[('task_id', 'in', active_ids)]</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="timesheet_view_tree_user"/>
        </record>

        <record id="timesheet_action_project" model="ir.actions.act_window">
            <field name="name">Project's Timesheets</field>
            <field name="res_model">account.analytic.line</field>
            <field name="context">{
                'is_timesheet': 1,
            }</field>
            <field name="domain">[('project_id', 'in', active_ids)]</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="timesheet_view_tree_user"/>
        </record>

        <record id="timesheet_action_all" model="ir.actions.act_window">
            <field name="name">All Timesheets</field>
            <field name="res_model">account.analytic.line</field>
            <field name="view_mode">tree,form,kanban,pivot,graph</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="domain">[('project_id', '!=', False)]</field>
            <field name="context">{
                'search_default_week':1,
                'is_timesheet': 1,
            }</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No activities found. Let's start a new one!
              </p>
              <p>
                Track your working hours by projects every day and invoice this time to your customers.
              </p>
            </field>
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

        <record id="timesheet_action_view_all_kanban" model="ir.actions.act_window.view">
            <field name="view_mode">kanban</field>
            <field name="sequence">6</field>
            <field name="view_id" ref="hr_timesheet.view_kanban_account_analytic_line"/>
            <field name="act_window_id" ref="timesheet_action_all"/>
        </record>

        <record id="timesheet_action_view_all_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="7"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="view_hr_timesheet_line_pivot"/>
            <field name="act_window_id" ref="timesheet_action_all"/>
        </record>

        <record id="timesheet_action_view_all_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="8"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="view_hr_timesheet_line_graph_all"/>
            <field name="act_window_id" ref="timesheet_action_all"/>
        </record>

        <record id="timesheet_action_from_employee" model="ir.actions.act_window">
            <field name="name">Timesheets</field>
            <field name="res_model">account.analytic.line</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="domain">[('project_id', '!=', False), ('employee_id', '=', active_id)]</field>
            <field name="context">{
                'default_employee_id': active_id,
                "is_timesheet": 1,
            }</field>
        </record>

        <record id="act_hr_timesheet_line_by_project" model="ir.actions.act_window">
            <field name="name">Timesheets</field>
            <field name="res_model">account.analytic.line</field>
            <field name="view_mode">tree,kanban,pivot,graph,form</field>
            <field name="view_id" ref="timesheet_view_tree_user"/>
            <field name="domain">[('project_id', '=', active_id)]</field>
            <field name="context">{
                "default_project_id": active_id,
                "is_timesheet": 1,
            }</field>
            <field name="search_view_id" ref="hr_timesheet_line_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Record a new activity
              </p><p>
                Track your working hours by projects every day and invoice this time to your customers.
              </p>
            </field>
        </record>

        <record id="act_hr_timesheet_line_by_project_view_tree" model="ir.actions.act_window.view">
            <field name="view_mode">tree</field>
            <field name="sequence" eval="1"/>
            <field name="view_id" ref="timesheet_view_tree_user"/>
            <field name="act_window_id" ref="act_hr_timesheet_line_by_project"/>
        </record>

        <record id="act_hr_timesheet_line_by_project_view_kanban" model="ir.actions.act_window.view">
            <field name="view_mode">kanban</field>
            <field name="sequence" eval="2"/>
            <field name="view_id" ref="view_kanban_account_analytic_line"/>
            <field name="act_window_id" ref="act_hr_timesheet_line_by_project"/>
        </record>

        <record id="act_hr_timesheet_line_by_project_view_pivot" model="ir.actions.act_window.view">
            <field name="view_mode">pivot</field>
            <field name="sequence" eval="3"/>
            <field name="view_id" ref="view_hr_timesheet_line_pivot"/>
            <field name="act_window_id" ref="act_hr_timesheet_line_by_project"/>
        </record>

        <record id="act_hr_timesheet_line_by_project_view_graph" model="ir.actions.act_window.view">
            <field name="view_mode">graph</field>
            <field name="sequence" eval="4"/>
            <field name="view_id" ref="view_hr_timesheet_line_graph_all"/>
            <field name="act_window_id" ref="act_hr_timesheet_line_by_project"/>
        </record>

        <record id="act_hr_timesheet_line_by_project_view_form" model="ir.actions.act_window.view">
            <field name="view_mode">form</field>
            <field name="sequence" eval="10"/>
            <field name="view_id" ref="hr_timesheet_line_form"/>
            <field name="act_window_id" ref="act_hr_timesheet_line_by_project"/>
        </record>
    </data>
</odoo>

```

## File: views\project_project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="project_project_view_form_simplified_inherit_timesheet" model="ir.ui.view">
            <field name="name">project.project.view.form.simplified.inherit.timesheet</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.project_project_view_form_simplified"/>
            <field name="priority">24</field>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('o_settings_container')]" position="inside">
                    <setting string="Timesheets" help="Log time on tasks">
                        <field name="allow_timesheets"/>
                    </setting>
                </xpath>
            </field>
        </record>

        <record id="project_invoice_form" model="ir.ui.view">
            <field name="name">Inherit project form : Invoicing Data</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.edit_project"/>
            <field name="priority">24</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='date']" position="after">
                    <field name="allocated_hours" widget="timesheet_uom_no_toggle" invisible="not allow_timesheets"/>
                </xpath>
                <xpath expr="//group[@name='group_time_managment']" position="attributes">
                    <attribute name="invisible">0</attribute>
                </xpath>
                <xpath expr="//group[@name='group_time_managment']" position="inside">
                    <setting id="timesheet_settings" string="Timesheets" help="Log time on tasks">
                        <field name="allow_timesheets"/>
                    </setting>
                </xpath>
            </field>
        </record>

        <record id="project_project_view_tree_inherit_sale_project" model="ir.ui.view">
            <field name="name">project.project.tree.inherit.sale.timesheet</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='date']" position="after">
                    <field name="allocated_hours" widget="timesheet_uom_no_toggle" optional="hide" invisible="allocated_hours == 0"/>
                </xpath>
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
                    <field name="remaining_hours" invisible="1"/>
                    <field name="encode_uom_in_days" invisible="1"/>
                    <field name="allocated_hours" invisible="1"/>
                </field>
                <xpath expr="//div[hasclass('o_kanban_manage_view')]" position="inside">
                    <div role="menuitem" t-if="record.allow_timesheets.raw_value" groups="hr_timesheet.group_hr_timesheet_user">
                        <a name="action_project_timesheets" type="object">Timesheets</a>
                    </div>
                </xpath>
                <xpath expr="//div[hasclass('o_project_kanban_boxes')]" position="after">
                    <t t-set="badgeColor" t-value="'border-success'"/>
                    <t t-set="badgeColor" t-value="'border-danger'" t-if="record.remaining_hours.raw_value &lt; 0"/>
                    <t t-set="title" t-value="'Remaining days'" t-if="record.encode_uom_in_days.raw_value"/>
                    <t t-set="title" t-value="'Remaining hours'" t-else=""/>
                    <div t-if="record.allow_timesheets.raw_value and record.allocated_hours.raw_value &gt; 0"
                        t-attf-class="oe_kanban_align badge border {{ badgeColor }}" t-att-title="title" groups="hr_timesheet.group_hr_timesheet_user">
                        <field name="remaining_hours" widget="timesheet_uom"/>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="view_project_project_filter_inherit_timesheet" model="ir.ui.view">
            <field name="name">project.project.view.inherit.timesheet</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project_project_filter"/>
            <field name="arch" type="xml">
                <filter name="late_milestones" position="before">
                    <filter string="Timesheets &gt;100%" name="projects_in_overtime" domain="[('is_project_overtime', '=', True)]"/>
                </filter>
            </field>
        </record>

        <record id="project.open_view_project_all" model="ir.actions.act_window">
            <field name="domain">[('is_internal_project', '=', False)]</field>
        </record>

        <record id="project.open_view_project_all_group_stage" model="ir.actions.act_window">
            <field name="domain">[('is_internal_project', '=', False)]</field>
        </record>
    </data>
</odoo>

```

## File: views\project_task_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="portal_my_task" inherit_id="project.portal_my_task" name="Portal: My Task with Timesheets">
        <xpath expr="//div[@id='task-nav']" position="before">
            <div t-if="timesheets and allow_timesheets" class="d-grid flex-grow-1" id='nav-report'>
                <div class="o_download_pdf">
                    <a class="btn btn-secondary o_print_btn o_project_timesheet_print d-block" t-att-href="task.get_portal_url(report_type='pdf')" href="#" title="View Details" target="_blank"><i class="fa fa-print"/> View Details</a>
                </div>
            </div>
        </xpath>
        <xpath expr="//li[@id='nav-header']" position="after">
            <div t-if="timesheets and allow_timesheets" class="nav-item">
                <a class="nav-link p-0" href="#task_timesheets">
                    Timesheets
                </a>
            </div>
        </xpath>
        <xpath expr="//div[@id='card_body']" position="inside">
            <div class="container" t-if="timesheets and allow_timesheets">
                <h5 id="task_timesheets" class="mt-2 mb-2" data-anchor="true">Timesheets</h5>
                <t t-call="hr_timesheet.portal_timesheet_table"/>
            </div>
        </xpath>
        <xpath expr="//div[@name='portal_my_task_allocated_hours']" position="after">
            <div t-if="task.allocated_hours > 0 and allow_timesheets"><strong>Progress:</strong> <span t-field="task.progress"/>%</div>
        </xpath>
        <xpath expr="//div[@name='portal_my_task_allocated_hours']/t" position="replace">
            <t t-call="hr_timesheet.portal_my_task_allocated_hours_template"></t>
        </xpath>
    </template>

    <template id="portal_my_task_allocated_hours_template">
        <t t-if="is_uom_day and timesheets._convert_hours_to_days(task.allocated_hours) > 0">
            <span t-out="timesheets._convert_hours_to_days(task.allocated_hours)" t-options='{"widget": "timesheet_uom"}'/>
        </t>
        <t t-if="not is_uom_day and task.allocated_hours > 0 and allow_timesheets" t-call="project.portal_my_task_allocated_hours_template"></t>
    </template>

    <template id="portal_tasks_list_inherit" inherit_id="project.portal_tasks_list" name="Portal: My Tasks with Timesheets">
        <xpath expr="//t[@t-foreach='tasks']/tr" position="before">
            <t t-set="timesheet_ids" t-value="task.sudo().timesheet_ids"/>
            <t t-set="is_uom_day" t-value="timesheet_ids._is_timesheet_encode_uom_day()"/>
        </xpath>
        <xpath expr="//thead/tr/t[@t-set='number_of_header']" position="attributes">
            <attribute name="t-value">9</attribute>
        </xpath>
        <xpath expr="//thead/tr/th[@name='project_portal_milestones']" position="after">
            <t t-if="not project or project.allow_timesheets">
                <th t-if="is_uom_day" class="text-end">Days Spent</th>
                <th t-else="" class="text-end">Hours Spent</th>
            </t>
        </xpath>
        <xpath expr="//tbody/t/tr/td[@name='project_portal_milestones']" position="after">
            <td t-if="not project or project.allow_timesheets" class="text-end">
                <t t-if="task.allow_timesheets">
                    <t t-if="is_uom_day">
                        <t t-out="timesheet_ids._convert_hours_to_days(task.effective_hours)"/>
                        <span t-if="task.allocated_hours > 0"> / <t t-out="timesheet_ids._convert_hours_to_days(task.allocated_hours)"/></span>
                    </t>
                    <t t-else="">
                        <span t-field="task.effective_hours" t-options='{"widget": "float_time"}'/>
                        <t t-if="task.allocated_hours > 0">
                            /
                            <span t-field="task.allocated_hours" t-options='{"widget": "float_time"}'/>
                        </t>
                    </t>
                </t>
            </td>
        </xpath>
    </template>

</odoo>

```

## File: views\project_task_sharing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_sharing_inherit_project_task_view_form" model="ir.ui.view">
        <field name="name">project.sharing.project.task.view.form.inherit</field>
        <field name="model">project.task</field>
        <field name="priority">500</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='child_ids']/tree/field[@name='portal_user_names']" position="after">
                <field name="allocated_hours" widget="timesheet_uom_no_toggle" sum="Total Allocated Time" optional="hide"/>
                <field name="effective_hours" widget="timesheet_uom" sum="Effective Hours" optional="hide"/>
                <field name="subtask_effective_hours" string="Sub-tasks Hours Spent" widget="timesheet_uom" sum="Sub-tasks Hours Spent" optional="hide"/>
                <field name="total_hours_spent" string="Total Hours" widget="timesheet_uom" sum="Total Hours" optional="hide"/>
                <field name="remaining_hours" widget="timesheet_uom" sum="Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100"/>
                <field name="progress" widget="project_task_progressbar" optional="hide" options="{'overflow_class': 'bg-danger'}"/>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="encode_uom_in_days" invisible="1"/>
                <field name="subtask_count" invisible="1"/>
                <label for="allocated_hours" invisible="not allow_timesheets"/>
                <div class="text-nowrap" invisible="not allow_timesheets">
                    <field name="allocated_hours" class="oe_inline" widget="float_time"/>
                    <span invisible="subtask_count == 0">
                        (incl. <field name="subtask_allocated_hours" nolabel="1" widget="timesheet_uom_no_toggle" class="oe_inline"/> on
                        <span class="fw-bold text-dark"> Sub-tasks</span>)
                    </span>
                    <span class="ps-1">(<field name="progress" class="oe_inline" nolabel="1" widget="integer"/> %)</span>
                </div>
            </xpath>
            <xpath expr="//notebook/page[@name='description_page']" position="after">
                <field name="analytic_account_active" invisible="1"/>
                <field name="allow_timesheets" invisible="1"/>
                <page string="Timesheets" name="page_timesheets" id="timesheets_tab" invisible="not allow_timesheets">
                    <field name="timesheet_ids" mode="tree,kanban"
                          invisible="not analytic_account_active">
                        <tree string="Timesheet Activities" default_order="date" no_open="1" create="false" delete="0">
                            <field name="date"/>
                            <field name="employee_id"/>
                            <field name="name"/>
                            <field name="unit_amount" widget="timesheet_uom" decoration-danger="unit_amount &gt; 24"/>
                        </tree>
                        <kanban class="o_kanban_mobile">
                            <field name="date"/>
                            <field name="employee_id"/>
                            <field name="name"/>
                            <field name="unit_amount" decoration-danger="unit_amount &gt; 24"/>
                            <field name="project_id"/>
                            <templates>
                                <t t-name="kanban-box">
                                    <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                        <div class="row">
                                            <div class="col-6">
                                                <field name="employee_id" invisible="1"/>
                                                <strong><span><t t-esc="record.employee_id.value"/></span></strong>
                                            </div>
                                            <div class="col-6 float-end text-end">
                                                <strong><t t-esc="record.date.value"/></strong>
                                            </div>
                                        </div>
                                        <div class="row">
                                            <div class="col-6 text-muted">
                                                <span><t t-esc="record.name.value"/></span>
                                            </div>
                                            <div class="col-6">
                                                <span class="float-end text-end">
                                                    <field name="unit_amount" widget="float_time"/>
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </t>
                            </templates>
                        </kanban>
                    </field>
                    <group invisible="not analytic_account_active">
                        <group class="oe_subtotal_footer" name="project_hours">
                            <span class="o_td_label float-start">
                                <label class="fw-bold" for="effective_hours" string="Hours Spent" invisible="encode_uom_in_days"/>
                                <label class="fw-bold" for="effective_hours" string="Days Spent" invisible="not encode_uom_in_days"/>
                            </span>
                            <field name="effective_hours" widget="timesheet_uom" nolabel="1"/>
                            <button name="action_view_subtask_timesheet" type="object" class="ps-0 border-0 oe_inline oe_link mb-2 o_td_label float-start" invisible="subtask_effective_hours == 0.0">
                                <span class="text-nowrap" invisible="encode_uom_in_days">Hours Spent on Sub-tasks:</span>
                                <span class="text-nowrap" invisible="not encode_uom_in_days">Days Spent on Sub-tasks:</span>
                            </button>
                            <field name="subtask_effective_hours" class="mt-2" widget="timesheet_uom"
                                  invisible="subtask_effective_hours == 0.0" nolabel="1"/>
                            <span id="total_hours_spent_label" invisible="subtask_effective_hours == 0.0" class="o_td_label float-start">
                                <label class="fw-bold" for="total_hours_spent" string="Total Hours"
                                      invisible="encode_uom_in_days"/>
                                <label class="fw-bold" for="total_hours_spent" string="Total Days"
                                      invisible="not encode_uom_in_days"/>
                            </span>
                            <field name="total_hours_spent" widget="timesheet_uom" class="oe_subtotal_footer_separator" nolabel="1"
                                  invisible="subtask_effective_hours == 0.0" />
                            <span class="o_td_label float-start">
                                <label class="fw-bold" for="remaining_hours" string="Remaining Hours"
                                       invisible="allocated_hours == 0.0 or encode_uom_in_days or remaining_hours &lt; 0"/>
                                <label class="fw-bold" for="remaining_hours" string="Remaining Days"
                                       invisible="allocated_hours == 0.0 or not encode_uom_in_days or remaining_hours &lt; 0"/>
                                <label class="fw-bold text-danger" for="remaining_hours" string="Remaining Hours"
                                       invisible="allocated_hours == 0.0 or encode_uom_in_days or remaining_hours &gt;= 0"/>
                                <label class="fw-bold text-danger" for="remaining_hours" string="Remaining Days"
                                       invisible="allocated_hours == 0.0 or not encode_uom_in_days or remaining_hours &gt;= 0"/>
                            </span>
                            <field name="remaining_hours" widget="timesheet_uom" class="oe_subtotal_footer_separator"
                                  invisible="allocated_hours == 0.0" nolabel="1"/>
                        </group>
                    </group>
                </page>
            </xpath>
        </field>
    </record>

    <record id="project_sharing_kanban_inherit_project_task_view_kanban" model="ir.ui.view">
        <field name="name">project.sharing.project.task.timesheet.kanban.inherited</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_kanban"/>
        <field name="arch" type="xml">
            <templates position="before">
                <field name="progress" />
                <field name="remaining_hours" />
                <field name="allocated_hours" />
                <field name="allow_timesheets"/>
                <field name="encode_uom_in_days" invisible="1"/>
            </templates>
            <div class="oe_kanban_bottom_left" position="inside">
                <t name="allocated_hours" t-if="record.allocated_hours.raw_value &gt; 0 and record.allow_timesheets.raw_value">
                    <t t-set="badge" t-value="'border border-success'"/>
                    <t t-set="badge" t-value="'border border-warning'" t-if="record.progress.raw_value &gt;= 80 and record.progress.raw_value &lt;= 100"/>
                    <t t-set="badge" t-value="'border border-danger'" t-if="record.remaining_hours.raw_value &lt; 0"/>
                    <t t-set="title" t-value="'Remaining days'" t-if="record.encode_uom_in_days.raw_value"/>
                    <t t-set="title" t-value="'Remaining hours'" t-else=""/>
                    <div t-attf-class="oe_kanban_align badge {{ badge }}" t-att-title="title">
                        <field name="remaining_hours" widget="timesheet_uom" />
                    </div>
                </t>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="view_task_form2_inherited">
            <field name="name">project.task.form.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_form2" />
            <field name="arch" type="xml">
                <xpath expr="//field[@name='child_ids']/tree/field[@name='company_id']" position="after">
                    <field name="allocated_hours" widget="timesheet_uom_no_toggle" sum="Total Allocated Time" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="effective_hours" widget="timesheet_uom" sum="Hours Spent" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="subtask_effective_hours" widget="timesheet_uom" sum="Sub-tasks Hours Spent" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="total_hours_spent" widget="timesheet_uom" sum="Total Hours" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="remaining_hours" widget="timesheet_uom" sum="Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="progress" widget="project_task_progressbar" optional="hide" options="{'overflow_class': 'bg-danger'}" groups="hr_timesheet.group_hr_timesheet_user"/>
                </xpath>
                <xpath expr="//label[@for='date_deadline']" position="before">
                    <field name="encode_uom_in_days" invisible="1"/>
                    <field name="subtask_count" invisible="1"/>
                    <label for="allocated_hours" invisible="not allow_timesheets" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <div class="text-nowrap" invisible="not allow_timesheets" groups="hr_timesheet.group_hr_timesheet_user">
                        <field name="allocated_hours" class="oe_inline o_field_float_time o_task_planned_hours" widget="timesheet_uom_no_toggle"/>
                        <span invisible="subtask_count == 0">
                            (incl. <field name="subtask_allocated_hours" nolabel="1" widget="timesheet_uom_no_toggle" class="oe_inline"/> on
                            <span class="fw-bold text-dark"> Sub-tasks</span>)
                        </span>
                        <span invisible="not project_id">(<field name="progress" class="oe_inline" nolabel="1" widget="integer"/> %)</span>
                    </div>
                </xpath>
                <xpath expr="//notebook/page[@name='description_page']" position="after">
                    <field name="allow_timesheets" invisible="1"/>
                    <t groups="hr_timesheet.group_hr_timesheet_user">
                        <field name="analytic_account_active" invisible="1"/>
                    </t>
                    <page string="Timesheets" name="page_timesheets" id="timesheets_tab" invisible="not allow_timesheets" groups="hr_timesheet.group_hr_timesheet_user">
                        <group name="timesheet_error" invisible="analytic_account_active">
                            <div class="alert alert-warning mb-n4 mt-3" role="alert" colspan="2">
                                You cannot log timesheets on this project since it is linked to an inactive analytic account. Please change this account, or reactivate the current one to timesheet on the project.
                            </div>
                        </group>
                    <field name="timesheet_ids" mode="tree,kanban" invisible="not analytic_account_active" context="{'default_project_id': project_id, 'default_name':''}">
                        <tree editable="bottom" string="Timesheet Activities" default_order="date" decoration-muted="readonly_timesheet == True">
                            <field name="readonly_timesheet" column_invisible="True"/>
                            <field name="date" readonly="readonly_timesheet"/>
                            <field name="user_id" column_invisible="True"/>
                            <field name="employee_id" widget="many2one_avatar_employee" context="{'active_test': True}"
                                required="1"
                                readonly="readonly_timesheet"/>
                            <field name="name" required="0" readonly="readonly_timesheet"/>
                            <field name="unit_amount" widget="timesheet_uom" decoration-danger="unit_amount &gt; 24 or unit_amount &lt; 0"
                                readonly="readonly_timesheet"/>
                            <field name="project_id" column_invisible="True"/>
                            <field name="task_id" column_invisible="True"/>
                            <field name="company_id" column_invisible="True"/>
                        </tree>
                        <kanban class="o_kanban_mobile">
                            <field name="readonly_timesheet"/>
                            <field name="date"/>
                            <field name="user_id"/>
                            <field name="name"/>
                            <field name="unit_amount" decoration-danger="unit_amount &gt; 24"/>
                            <field name="project_id"/>
                            <field name="task_id"/>
                            <templates>
                                <t t-name="kanban-box">
                                    <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                        <div class="row">
                                            <div class="col-6 d-flex">
                                                <field name="employee_id" class="me-1" widget="many2one_avatar_employee" context="{'active_test': True}" readonly="readonly_timesheet"/>
                                                <strong><span><t t-esc="record.employee_id.value"/></span></strong>
                                            </div>
                                            <div class="col-6 float-end text-end">
                                                <strong><t t-esc="record.date.value"/></strong>
                                            </div>
                                        </div>
                                        <div class="row">
                                            <div class="col-6 text-muted">
                                                <span><t t-esc="record.name.value"/></span>
                                            </div>
                                            <div class="col-6">
                                                <span class="float-end text-end">
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
                                    <field name="readonly_timesheet" invisible="1"/>
                                    <field name="date" readonly="readonly_timesheet"/>
                                    <field name="user_id" invisible="1" readonly="readonly_timesheet"/>
                                    <field name="employee_id" widget="many2one_avatar_employee" context="{'active_test': True}" required="1" readonly="readonly_timesheet"/>
                                    <field name="name" required="0" readonly="readonly_timesheet"/>
                                    <field name="unit_amount" string="Duration" widget="float_time" decoration-danger="unit_amount &gt; 24"
                                        readonly="readonly_timesheet"/>
                                    <field name="project_id" invisible="1"/>
                                    <field name="task_id" invisible="1"/>
                                    <field name="company_id" invisible="1"/>
                                </group>
                            </sheet>
                        </form>
                    </field>
                    <group invisible="not analytic_account_active">
                        <group class="oe_subtotal_footer" name="project_hours">
                            <span class="o_td_label float-start">
                                <label class="fw-bold" for="effective_hours" string="Hours Spent" invisible="encode_uom_in_days"/>
                                <label class="fw-bold" for="effective_hours" string="Days Spent" invisible="not encode_uom_in_days"/>
                            </span>
                            <field name="effective_hours" widget="timesheet_uom" nolabel="1"/>
                            <button name="action_view_subtask_timesheet" type="object" class="ps-0 border-0 oe_inline oe_link mb-2 o_td_label float-start" invisible="subtask_effective_hours == 0.0">
                                <span class="text-nowrap" invisible="encode_uom_in_days">Hours Spent on Sub-tasks:</span>
                                <span class="text-nowrap" invisible="not encode_uom_in_days">Days Spent on Sub-tasks:</span>
                            </button>
                            <field name="subtask_effective_hours" class="mt-2" widget="timesheet_uom"
                                   invisible="subtask_effective_hours == 0.0" nolabel="1"/>
                            <span invisible="subtask_effective_hours == 0.0" class="o_td_label float-start my-1">
                                <label class="fw-bold" for="total_hours_spent" string="Total Hours"
                                       invisible="subtask_effective_hours == 0.0 or encode_uom_in_days"/>
                                <label class="fw-bold" for="total_hours_spent" string="Total Days"
                                       invisible="subtask_effective_hours == 0.0 or not encode_uom_in_days"/>
                            </span>
                            <field name="total_hours_spent" widget="timesheet_uom" class="oe_subtotal_footer_separator" nolabel="1"
                                   invisible="subtask_effective_hours == 0.0" />
                            <span class="o_td_label float-start my-1" invisible="allocated_hours == 0.0">
                                <label class="fw-bold" for="remaining_hours" string="Remaining Hours"
                                       invisible="encode_uom_in_days or remaining_hours &lt; 0"/>
                                <label class="fw-bold" for="remaining_hours" string="Remaining Days"
                                       invisible="not encode_uom_in_days or remaining_hours &lt; 0"/>
                                <label class="fw-bold text-danger" for="remaining_hours" string="Remaining Hours"
                                       invisible="encode_uom_in_days or remaining_hours &gt;= 0"/>
                                <label class="fw-bold text-danger" for="remaining_hours" string="Remaining Days"
                                       invisible="not encode_uom_in_days or remaining_hours &gt;= 0"/>
                            </span>
                            <field name="remaining_hours" widget="timesheet_uom" class="oe_subtotal_footer_separator"
                                   invisible="allocated_hours == 0.0" nolabel="1" decoration-danger="remaining_hours &lt; 0"/>
                        </group>
                    </group>
                </page>
                </xpath>
                <xpath expr="//field[@name='depend_on_ids']/tree//field[@name='company_id']" position="after">
                    <field name="progress" column_invisible="True" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="allocated_hours" widget="timesheet_uom_no_toggle" sum="Total Allocated Time" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="effective_hours" widget="timesheet_uom" sum="Effective Hours" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="subtask_effective_hours" widget="timesheet_uom" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="total_hours_spent" widget="timesheet_uom" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="remaining_hours" widget="timesheet_uom" sum="Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="progress" widget="project_task_progressbar" optional="hide" options="{'overflow_class': 'bg-danger'}" groups="hr_timesheet.group_hr_timesheet_user"/>
                </xpath>
            </field>
        </record>

        <record id="view_task_tree2_inherited" model="ir.ui.view">
            <field name="name">project.task.tree.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.project_task_view_tree_main_base" />
            <field name="arch" type="xml">
                <field name="date_deadline" position="before">
                    <field name="progress" column_invisible="True"/>
                    <field name="effective_hours" column_invisible="True"/>
                    <field name="allocated_hours" widget="timesheet_uom_no_toggle" sum="Total Allocated Time" invisible="allocated_hours == 0" optional="hide" column_invisible="not context.get('allow_timesheets', True)"/>
                    <field name="effective_hours" widget="timesheet_uom" sum="Effective Hours" optional="show" invisible="effective_hours == 0" column_invisible="not context.get('allow_timesheets', True)"/>
                    <field name="subtask_effective_hours" widget="timesheet_uom" sum="Sub-Tasks Total Effective Hours" optional="hide" column_invisible="not context.get('allow_timesheets', True)"/>
                    <field name="total_hours_spent" widget="timesheet_uom" sum="Total Hours" optional="hide" column_invisible="not context.get('allow_timesheets', True)"/>
                    <field name="remaining_hours" widget="timesheet_uom" sum="Total Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100" invisible="allocated_hours == 0" column_invisible="not context.get('allow_timesheets', True)"/>
                    <field name="progress" widget="project_task_progressbar" avg="Average of Progress" optional="show" groups="hr_timesheet.group_hr_timesheet_user" invisible="allocated_hours == 0" options="{'overflow_class': 'bg-danger'}" column_invisible="not context.get('allow_timesheets', True)"/>
                </field>
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
                    <field name="allocated_hours" />
                    <field name="allow_timesheets"/>
                    <field name="encode_uom_in_days" invisible="1"/>
                </templates>
                <div class="oe_kanban_bottom_left" position="inside">
                   <t name="allocated_hours" t-if="record.allocated_hours.raw_value &gt; 0 and record.allow_timesheets.raw_value" groups="hr_timesheet.group_hr_timesheet_user">
                        <t t-set="badge" t-value=""/>
                        <t t-set="badge" t-value="'border border-warning'" t-if="record.progress.raw_value &gt;= 80 and record.progress.raw_value &lt;= 100"/>
                        <t t-set="badge" t-value="'border border-danger'" t-if="record.remaining_hours.raw_value &lt; 0"/>
                        <t t-set="title" t-value="'Remaining days'" t-if="record.encode_uom_in_days.raw_value"/>
                        <t t-set="title" t-value="'Remaining hours'" t-else=""/>
                        <div t-attf-class="oe_kanban_align badge {{ badge }} me-0" t-att-title="title">
                            <field name="remaining_hours" widget="timesheet_uom" />
                        </div>
                   </t>
                </div>
             </field>
         </record>

        <record id="project_task_view_search" model="ir.ui.view">
            <field name="name">project.task.view.search.inherit.sale.timesheet.enterprise</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_search_form_project_fsm_base"/>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='blocking']/following-sibling::separator[1]" position="after">
                    <filter string="Timesheets 80%" name="timesheet_80" invisible="not context.get('allow_timesheets')"
                        domain="[('remaining_hours_percentage', '&gt;', 0.0), ('remaining_hours_percentage', '&lt;=', 0.2)]"/>
                    <filter string="Timesheets &gt;100%" name="timesheet_exceeded" domain="[('overtime', '&gt;', 0)]"
                        invisible="not context.get('allow_timesheets')"/>
                    <separator/>
                </xpath>
            </field>
        </record>

        <record id="project_task_view_graph" model="ir.ui.view">
            <field name="name">project.task.view.graph.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_project_task_graph"/>
            <field name="arch" type="xml">
                <xpath expr="//graph" position="attributes">
                    <attribute name="js_class">hr_timesheet_graphview</attribute>
                </xpath>
                <xpath expr="//field[@name='stage_id']" position='after'>
                    <field name="allocated_hours" widget="timesheet_uom"/>
                    <field name="remaining_hours" widget="timesheet_uom"/>
                    <field name="effective_hours" widget="timesheet_uom"/>
                    <field name="total_hours_spent" widget="timesheet_uom"/>
                    <field name="overtime" widget="timesheet_uom"/>
                    <field name="subtask_effective_hours" widget="timesheet_uom"/>
                </xpath>
            </field>
        </record>

        <record id="project_task_view_pivot" model="ir.ui.view">
            <field name="name">project.task.view.pivot.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_project_task_pivot"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='allocated_hours']" position='attributes'>
                    <attribute name="widget">timesheet_uom</attribute>
                </xpath>
                <xpath expr="//field[@name='allocated_hours']" position='after'>
                    <field name="remaining_hours" widget="timesheet_uom"/>
                    <field name="effective_hours" widget="timesheet_uom"/>
                    <field name="total_hours_spent" widget="timesheet_uom"/>
                    <field name="overtime" widget="timesheet_uom"/>
                    <field name="subtask_effective_hours" widget="timesheet_uom"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\project_update_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="project_update_view_search_inherit" model="ir.ui.view">
            <field name="name">project.update.view.search.inherit</field>
            <field name="model">project.update</field>
            <field name="inherit_id" ref="project.project_update_view_search"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='my_updates']" position='after'>
                    <filter string="My Team's Updates" name="my_team_updates" domain="[('user_id.employee_parent_id.user_id', '=', uid)]"/>
                    <filter string="My Department's Updates" name="my_department_updates" domain="[('user_id.employee_id.member_of_department', '=', True)]"/>
                </xpath>
            </field>
        </record>
        <record id="project_update_view_kanban_inherit" model="ir.ui.view">
            <field name="name">project.update.view.kanban.inherit</field>
            <field name="model">project.update</field>
            <field name="inherit_id" ref="project.project_update_view_kanban"/>
            <field name="arch" type="xml">
                <b id="tasks_stats" position='after'>
                    <field name="display_timesheet_stats" invisible="1"/>
                    <div invisible="not display_timesheet_stats">
                        <field name="timesheet_time"/><span invisible="not allocated_time"> / <field name="allocated_time"/></span> <field name="uom_id" no_open="1"/><span invisible="not allocated_time"> (<field name="timesheet_percentage"/>%)</span>
                    </div>
                </b>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.hr.timesheet</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="55"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside">
                <app data-string="Timesheets" string="Timesheets" name="hr_timesheet" groups="hr_timesheet.group_timesheet_manager" id="timesheets">
                    <block title="Time Encoding" name="time_encoding_setting_container">
                        <setting id="time_mode_setting" company_dependent="1" invisible="project_time_mode_id">
                            <field name="project_time_mode_id" options="{'no_create': True, 'no_open': True}"/>
                        </setting>
                        <setting company_dependent="1" help="Time unit used to record your timesheets" id="time_unit_timesheets_setting">
                            <field name="timesheet_encode_method" class="col-lg-5 ps-0" widget="radio"/>
                            <field name="is_encode_uom_days" invisible="1"/>
                        </setting>
                    </block>
                    <block title="Timesheets Control">
                        <setting id="reminder_user_allow" company_dependent="1" help="Send a periodical email reminder to timesheets users that still have timesheets to encode">
                            <field name="reminder_user_allow" widget="upgrade_boolean"/>
                        </setting>
                        <setting id="reminder_allow" company_dependent="1" help="Send a periodical email reminder to timesheets approvers that still have timesheets to validate">
                            <field name="reminder_allow" widget="upgrade_boolean"/>
                        </setting>
                    </block>
                    <div name="section_leaves">
                        <block title="Time Off" name="timesheet_control">
                            <setting company_dependent="1" documentation="/applications/services/timesheets/overview/time_off.html" help="Generate timesheets for validated time off requests and public holidays" id="timesheet_off_validation_setting">
                                <field name="module_project_timesheet_holidays"/>
                            </setting>
                        </block>
                    </div>
                </app>
            </xpath>
        </field>
    </record>

    <record id="hr_timesheet_config_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'hr_timesheet', 'bin_size': False}</field>
    </record>
</odoo>

```

## File: wizard\hr_employee_delete_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class HrEmployeDeleteWizard(models.TransientModel):
    _name = 'hr.employee.delete.wizard'
    _description = 'Employee Delete Wizard'

    employee_ids = fields.Many2many('hr.employee', string='Employees', context={'active_test': False})
    has_active_employee = fields.Boolean(string='Has Active Employee', compute='_compute_has_active_employee')
    has_timesheet = fields.Boolean(string='Has Timesheet', compute='_compute_has_timesheet', compute_sudo=True)

    @api.depends('employee_ids')
    def _compute_has_timesheet(self):
        timesheet_read_group = self.env['account.analytic.line']._read_group([
            ('employee_id', 'in', self.employee_ids.ids)],
            ['employee_id'],
        )
        timesheet_employee_map = {employee.id for [employee] in timesheet_read_group}
        for wizard in self:
            wizard.has_timesheet = timesheet_employee_map & set(wizard.employee_ids.ids)

    @api.depends('employee_ids')
    def _compute_has_active_employee(self):
        unarchived_employees = self.env['hr.employee'].search([('id', '=', self.employee_ids.ids)])
        for wizard in self:
            wizard.has_active_employee = any(emp in wizard.employee_ids for emp in unarchived_employees)

    def action_archive(self):
        self.ensure_one()
        if len(self.employee_ids) != 1:
            return self.employee_ids.toggle_active()
        return {
            'name': _('Employee Termination'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.departure.wizard',
            'views': [[False, 'form']],
            'view_mode': 'form',
            'target': 'new',
            'context': {
                'active_id': self.employee_ids.id,
                'toggle_active': True,
            }
        }

    def action_confirm_delete(self):
        self.ensure_one()
        self.employee_ids.unlink()
        return self.env['ir.actions.act_window']._for_xml_id('hr.open_view_employee_list_my')

    def action_open_timesheets(self):
        self.ensure_one()
        employees = self.with_context(active_test=False).employee_ids
        action = {
           'name': _('Employees\' Timesheets'),
           'type': 'ir.actions.act_window',
           'res_model': 'account.analytic.line',
           'view_mode': 'tree,form',
           'views': [(False, 'tree'), (False, 'form')],
           'domain': [('employee_id', 'in', employees.ids), ('project_id', '!=', False)],
        }
        if len(employees) == 1:
            action['name'] = _('Timesheets of %(name)s', name=employees.name)
        return action

```

## File: wizard\hr_employee_delete_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="hr_employee_delete_wizard_form" model="ir.ui.view">
            <field name="name">Delete Employee</field>
            <field name="model">hr.employee.delete.wizard</field>
            <field name="arch" type="xml">
                <form string="Delete Employee">
                    <field name="has_timesheet" invisible="1"/>
                    <field name="has_active_employee" invisible="1"/>
                    <span invisible="not has_timesheet">
                        You cannot delete employees who have timesheets.
                        <span invisible="not has_active_employee">
                            You can either archive these employees or first delete all of their timesheets.
                        </span>
                        <span invisible="has_active_employee" groups="hr_timesheet.group_hr_timesheet_approver">
                            Please first delete all of their timesheets.
                        </span>
                    </span>
                    <span invisible="has_timesheet">
                        Are you sure you want to delete these employees?
                    </span>
                    <footer invisible="not has_timesheet">
                        <button string="Archive Employees" type="object" name="action_archive" class="btn btn-primary"
                            invisible="not has_active_employee" data-hotkey="q"/>
                        <button string="See Timesheets" type="object" name="action_open_timesheets" class="btn btn-primary" groups="hr_timesheet.group_hr_timesheet_approver"
                            invisible="has_active_employee" data-hotkey="w"/>
                        <button string="See Timesheets" type="object" name="action_open_timesheets" class="btn btn-secondary" groups="hr_timesheet.group_hr_timesheet_approver"
                            invisible="not has_active_employee" data-hotkey="w"/>
                        <button string="Discard" special="cancel" data-hotkey="x"/>
                    </footer>
                    <footer invisible="has_timesheet">
                        <button string="Ok" type="object" name="action_confirm_delete" class="btn btn-primary" data-hotkey="q"/>
                        <button string="Discard" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee_delete_wizard

```

