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

from odoo import api, fields, SUPERUSER_ID, _

from odoo.addons.project import _check_exists_collaborators_for_project_sharing


def create_internal_project(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})

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

def _uninstall_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})

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
        'views/project_views.xml',
        'views/project_portal_templates.xml',
        'views/hr_timesheet_portal_templates.xml',
        'report/hr_timesheet_report_view.xml',
        'report/project_report_view.xml',
        'report/report_timesheet_templates.xml',
        'views/hr_views.xml',
        'data/hr_timesheet_data.xml',
        'views/project_sharing_views.xml',
        'views/rating_rating_views.xml',
        'views/project_update_views.xml',
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

    def _task_get_searchbar_sortings(self, milestones_allowed):
        values = super()._task_get_searchbar_sortings(milestones_allowed)
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
                    raw_timesheets_group = Timesheet_sudo.read_group(
                        domain, ["unit_amount:sum", "ids:array_agg(id)"], ["date:day"]
                    )
                    grouped_timesheets = [(Timesheet_sudo.browse(group["ids"]), group["unit_amount"]) for group in raw_timesheets_group]

                else:
                    time_data = Timesheet_sudo.read_group(domain, [field, 'unit_amount:sum'], [field])
                    mapped_time = dict([(m[field][0] if m[field] else False, m['unit_amount']) for m in time_data])
                    grouped_timesheets = [(Timesheet_sudo.concat(*g), mapped_time[k.id]) for k, g in groupbyelem(timesheets, itemgetter(field))]
                return timesheets, grouped_timesheets

            grouped_timesheets = [(
                timesheets,
                sum(Timesheet_sudo.search(domain).mapped('unit_amount'))
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

    def _prepare_tasks_values(self, page, date_begin, date_end, sortby, search, search_in, groupby, url="/my/tasks", domain=None, su=False):
        values = super()._prepare_tasks_values(page, date_begin, date_end, sortby, search, search_in, groupby, url, domain, su)
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

    def _prepare_project_sharing_session_info(self, project, task=None):
        session_info = super()._prepare_project_sharing_session_info(project, task)

        company = project.company_id
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
    <img src="https://download.odoocdn.com/digests/hr_timesheet/static/img/digest_tip_timesheets_hotkeys.gif" class="illustration_border" />
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

    <!-- User Demo -->
    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(4, ref('group_hr_timesheet_user'))]"/>
    </record>

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

## File: models\hr_timesheet.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from lxml import etree
import re

from odoo import api, Command, fields, models, _, _lt
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
    ancestor_task_id = fields.Many2one('project.task', related='task_id.ancestor_id', store=True, index='btree_not_null')
    project_id = fields.Many2one(
        'project.project', 'Project', domain=_domain_project_id, index=True,
        compute='_compute_project_id', store=True, readonly=False)
    user_id = fields.Many2one(compute='_compute_user_id', store=True, readonly=False)
    employee_id = fields.Many2one('hr.employee', "Employee", domain=_domain_employee_id, context={'active_test': False},
        help="Define an 'hourly cost' on the employee to track the cost of their time.")
    job_title = fields.Char(related='employee_id.job_title')
    department_id = fields.Many2one('hr.department', "Department", compute='_compute_department_id', store=True, compute_sudo=True)
    manager_id = fields.Many2one('hr.employee', "Manager", related='employee_id.parent_id', store=True)
    encoding_uom_id = fields.Many2one('uom.uom', compute='_compute_encoding_uom_id')
    partner_id = fields.Many2one(compute='_compute_partner_id', store=True, readonly=False)

    def name_get(self):
        result = super().name_get()
        timesheets_read = self.env[self._name].search_read([('project_id', '!=', False), ('id', 'in', self.ids)], ['id', 'project_id', 'task_id'])
        if not timesheets_read:
            return result
        def _get_display_name(project_id, task_id):
            """ Get the display name of the timesheet based on the project and task
                :param project_id: tuple containing the id and the display name of the project
                :param task_id: tuple containing the id and the display name of the task if a task exists in the timesheet
                              otherwise False.
                :returns: the display name of the timesheet
            """
            if task_id:
                return '%s - %s' % (project_id[1], task_id[1])
            return project_id[1]
        timesheet_dict = {res['id']: _get_display_name(res['project_id'], res['task_id']) for res in timesheets_read}
        return list({**dict(result), **timesheet_dict}.items())

    def _compute_encoding_uom_id(self):
        for analytic_line in self:
            analytic_line.encoding_uom_id = analytic_line.company_id.timesheet_encode_uom_id

    @api.depends('task_id.partner_id', 'project_id.partner_id')
    def _compute_partner_id(self):
        for timesheet in self:
            if timesheet.project_id:
                timesheet.partner_id = timesheet.task_id.partner_id or timesheet.project_id.partner_id

    @api.depends('task_id', 'task_id.project_id')
    def _compute_project_id(self):
        for line in self:
            if not line.task_id.project_id or line.project_id == line.task_id.project_id:
                continue
            line.project_id = line.task_id.project_id

    @api.depends('project_id')
    def _compute_task_id(self):
        for line in self.filtered(lambda line: not line.project_id):
            line.task_id = False

    @api.onchange('project_id')
    def _onchange_project_id(self):
        # TODO KBA in master - check to do it "properly", currently:
        # This onchange is used to reset the task_id when the project changes.
        # Doing it in the compute will remove the task_id when the project of a task changes.
        if self.project_id != self.task_id.project_id:
            self.task_id = False

    @api.depends('employee_id')
    def _compute_user_id(self):
        for line in self:
            line.user_id = line.employee_id.user_id if line.employee_id else self._default_user()

    @api.depends('employee_id')
    def _compute_department_id(self):
        for line in self:
            line.department_id = line.employee_id.department_id

    @api.model_create_multi
    def create(self, vals_list):
        # Before creating a timesheet, we need to put a valid employee_id in the vals
        default_user_id = self._default_user()
        user_ids = []
        employee_ids = []
        # 1/ Collect the user_ids and employee_ids from each timesheet vals
        for vals in vals_list:
            vals.update(self._timesheet_preprocess(vals))
            if not vals.get('project_id'):
                continue
            if not vals.get('name'):
                vals['name'] = '/'
            employee_id = vals.get('employee_id')
            user_id = vals.get('user_id', default_user_id)
            if employee_id and employee_id not in employee_ids:
                employee_ids.append(employee_id)
            elif user_id not in user_ids:
                user_ids.append(user_id)

        # 2/ Search all employees related to user_ids and employee_ids, in the selected companies
        employees = self.env['hr.employee'].sudo().search([
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
            employee_in_id = vals.get('employee_id')
            if employee_in_id:
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
            else:  # ...and raise an error if they fail
                raise ValidationError(error_msg)

        # 5/ Finally, create the timesheets
        lines = super(AccountAnalyticLine, self).create(vals_list)
        for line, values in zip(lines, vals_list):
            if line.project_id:  # applied only for timesheet
                line._timesheet_postprocess(values)
        return lines

    def write(self, values):
        # If it's a basic user then check if the timesheet is his own.
        if not (self.user_has_groups('hr_timesheet.group_hr_timesheet_approver') or self.env.su) and any(self.env.user.id != analytic_line.user_id.id for analytic_line in self):
            raise AccessError(_("You cannot access timesheets that are not yours."))

        values = self._timesheet_preprocess(values)
        if values.get('employee_id'):
            employee = self.env['hr.employee'].browse(values['employee_id'])
            if not employee.active:
                raise UserError(_('You cannot set an archived employee to the existing timesheets.'))
        if 'name' in values and not values.get('name'):
            values['name'] = '/'
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
    def _get_view(self, view_id=None, view_type='form', **options):
        """ Set the correct label for `unit_amount`, depending on company UoM """
        arch, view = super()._get_view(view_id, view_type, **options)
        arch = self.sudo()._apply_timesheet_label(arch, view_type=view_type)
        return arch, view

    @api.model
    def _apply_timesheet_label(self, view_node, view_type='form'):
        doc = view_node
        encoding_uom = self.env.company.timesheet_encode_uom_id
        # Here, we select only the unit_amount field having no string set to give priority to
        # custom inheretied view stored in database. Even if normally, no xpath can be done on
        # 'string' attribute.
        for node in doc.xpath("//field[@name='unit_amount'][@widget='timesheet_uom'][not(@string)]"):
            node.set('string', _('%s Spent') % (re.sub(r'[\(\)]', '', encoding_uom.name or '')))
        return doc

    @api.model
    def _apply_time_label(self, view_node, related_model):
        doc = view_node
        Model = self.env[related_model]
        # Just fetch the name of the uom in `timesheet_encode_uom_id` of the current company
        encoding_uom_name = self.env.company.timesheet_encode_uom_id.with_context(prefetch_fields=False).sudo().name
        for node in doc.xpath("//field[@widget='timesheet_uom'][not(@string)] | //field[@widget='timesheet_uom_no_toggle'][not(@string)]"):
            name_with_uom = re.sub(_('Hours') + "|Hours", encoding_uom_name or '', Model._fields[node.get('name')]._description_string(self.env), flags=re.IGNORECASE)
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

    def _timesheet_preprocess(self, vals):
        """ Deduce other field values from the one given.
            Overrride this to compute on the fly some field that can not be computed fields.
            :param values: dict values for `create`or `write`.
        """
        project = self.env['project.project'].browse(vals.get('project_id', False))
        task = self.env['project.task'].browse(vals.get('task_id', False))
        # task implies project
        if task and not project:
            project = task.project_id
            if not project:
                raise ValidationError(_('You cannot create a timesheet on a private task.'))
            vals['project_id'] = project.id
        # task implies analytic account and tags
        if task and not vals.get('account_id'):
            task_analytic_account_id = task._get_task_analytic_account_id()
            vals['account_id'] = task_analytic_account_id.id
            vals['company_id'] = task_analytic_account_id.company_id.id or task.company_id.id
            if not task_analytic_account_id.active:
                raise UserError(_('You cannot add timesheets to a project or a task linked to an inactive analytic account.'))
        # project implies analytic account
        elif project and not vals.get('account_id'):
            vals['account_id'] = project.analytic_account_id.id
            vals['company_id'] = project.analytic_account_id.company_id.id or project.company_id.id
            if not project.analytic_account_id.active:
                raise UserError(_('You cannot add timesheets to a project linked to an inactive analytic account.'))
        # force customer partner, from the task or the project
        if project and not vals.get('partner_id'):
            partner_id = task.partner_id.id if task else project.partner_id.id
            if partner_id:
                vals['partner_id'] = partner_id
        # set timesheet UoM from the AA company (AA implies uom)
        if not vals.get('product_uom_id') and all(v in vals for v in ['account_id', 'project_id']):  # project_id required to check this is timesheet flow
            analytic_account = self.env['account.analytic.account'].sudo().browse(vals['account_id'])
            uom_id = analytic_account.company_id.project_time_mode_id.id
            if not uom_id:
                company_id = vals.get('company_id', False)
                if not company_id:
                    project = self.env['project.project'].browse(vals.get('project_id'))
                    company_id = project.analytic_account_id.company_id.id or project.company_id.id
                uom_id = self.env['res.company'].browse(company_id).project_time_mode_id.id
            vals['product_uom_id'] = uom_id
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

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import models, fields, api, _
from odoo.tools.float_utils import float_compare
from odoo.exceptions import UserError, ValidationError, RedirectWarning
from odoo.addons.rating.models.rating_data import OPERATOR_MAPPING

PROJECT_TASK_READABLE_FIELDS = {
    'allow_subtasks',
    'allow_timesheets',
    'analytic_account_active',
    'effective_hours',
    'encode_uom_in_days',
    'planned_hours',
    'progress',
    'overtime',
    'remaining_hours',
    'subtask_effective_hours',
    'subtask_planned_hours',
    'timesheet_ids',
    'total_hours_spent',
}

class Project(models.Model):
    _inherit = "project.project"

    allow_timesheets = fields.Boolean(
        "Timesheets", compute='_compute_allow_timesheets', store=True, readonly=False,
        default=True)
    analytic_account_id = fields.Many2one(
        # note: replaces ['|', ('company_id', '=', False), ('company_id', '=', company_id)]
        domain="""[
            '|', ('company_id', '=', False), ('company_id', '=', company_id),
            ('partner_id', '=?', partner_id),
        ]"""
    )

    timesheet_ids = fields.One2many('account.analytic.line', 'project_id', 'Associated Timesheets')
    timesheet_count = fields.Integer(compute="_compute_timesheet_count", groups='hr_timesheet.group_hr_timesheet_user')
    timesheet_encode_uom_id = fields.Many2one('uom.uom', related='company_id.timesheet_encode_uom_id')
    total_timesheet_time = fields.Integer(
        compute='_compute_total_timesheet_time', groups='hr_timesheet.group_hr_timesheet_user',
        help="Total number of time (in the proper UoM) recorded in the project, rounded to the unit.")
    encode_uom_in_days = fields.Boolean(compute='_compute_encode_uom_in_days')
    is_internal_project = fields.Boolean(compute='_compute_is_internal_project', search='_search_is_internal_project')
    remaining_hours = fields.Float(compute='_compute_remaining_hours', string='Remaining Invoiced Time', compute_sudo=True)
    is_project_overtime = fields.Boolean('Project in Overtime', compute='_compute_remaining_hours', search='_search_is_project_overtime', compute_sudo=True)
    allocated_hours = fields.Float(string='Allocated Hours')

    def _compute_encode_uom_in_days(self):
        self.encode_uom_in_days = self.env.company.timesheet_encode_uom_id == self.env.ref('uom.product_uom_day')

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
            ['project_id', 'unit_amount'],
            ['project_id'],
            lazy=False)
        timesheet_time_dict = {res['project_id'][0]: res['unit_amount'] for res in timesheets_read_group}
        for project in self:
            project.remaining_hours = project.allocated_hours - timesheet_time_dict.get(project.id, 0)
            project.is_project_overtime = project.remaining_hours < 0

    @api.model
    def _search_is_project_overtime(self, operator, value):
        if not isinstance(value, bool):
            raise ValueError(_('Invalid value: %s') % value)
        if operator not in ['=', '!=']:
            raise ValueError(_('Invalid operator: %s') % operator)

        query = """
            SELECT Project.id
              FROM project_project AS Project
              JOIN project_task AS Task
                ON Project.id = Task.project_id
             WHERE Project.allocated_hours > 0
               AND Project.allow_timesheets = TRUE
               AND Task.parent_id IS NULL
               AND Task.is_closed IS FALSE
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

    @api.depends('timesheet_ids')
    def _compute_total_timesheet_time(self):
        timesheets_read_group = self.env['account.analytic.line'].read_group(
            [('project_id', 'in', self.ids)],
            ['project_id', 'unit_amount', 'product_uom_id'],
            ['project_id', 'product_uom_id'],
            lazy=False)
        timesheet_time_dict = defaultdict(list)
        uom_ids = set(self.timesheet_encode_uom_id.ids)

        for result in timesheets_read_group:
            uom_id = result['product_uom_id'] and result['product_uom_id'][0]
            if uom_id:
                uom_ids.add(uom_id)
            timesheet_time_dict[result['project_id'][0]].append((uom_id, result['unit_amount']))

        uoms_dict = {uom.id: uom for uom in self.env['uom.uom'].browse(uom_ids)}
        for project in self:
            # Timesheets may be stored in a different unit of measure, so first
            # we convert all of them to the reference unit
            # if the timesheet has no product_uom_id then we take the one of the project
            total_time = 0.0
            for product_uom_id, unit_amount in timesheet_time_dict[project.id]:
                factor = uoms_dict.get(product_uom_id, project.timesheet_encode_uom_id).factor_inv
                total_time += unit_amount * (1.0 if project.encode_uom_in_days else factor)
            # Now convert to the proper unit of measure set in the settings
            total_time *= project.timesheet_encode_uom_id.factor
            project.total_timesheet_time = int(round(total_time))

    @api.depends('timesheet_ids')
    def _compute_timesheet_count(self):
        timesheet_read_group = self.env['account.analytic.line']._read_group(
            [('project_id', 'in', self.ids)],
            ['project_id'],
            ['project_id']
        )
        timesheet_project_map = {project_info['project_id'][0]: project_info['project_id_count'] for project_info in timesheet_read_group}
        for project in self:
            project.timesheet_count = timesheet_project_map.get(project.id, 0)

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

    def name_get(self):
        res = super().name_get()
        if len(self.env.context.get('allowed_company_ids', [])) <= 1:
            return res
        name_mapping = dict(res)
        for project in self:
            if project.is_internal_project:
                name_mapping[project.id] = f'{name_mapping[project.id]} - {project.company_id.name}'
        return list(name_mapping.items())

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

    def _convert_project_uom_to_timesheet_encode_uom(self, time):
        uom_from = self.company_id.project_time_mode_id
        uom_to = self.env.company.timesheet_encode_uom_id
        return round(uom_from._compute_quantity(time, uom_to, raise_if_failure=False), 2)

    def action_project_timesheets(self):
        action = self.env['ir.actions.act_window']._for_xml_id('hr_timesheet.act_hr_timesheet_line_by_project')
        action['display_name'] = _("%(name)s's Timesheets", name=self.name)
        return action


class Task(models.Model):
    _name = "project.task"
    _inherit = "project.task"

    analytic_account_active = fields.Boolean("Active Analytic Account", compute='_compute_analytic_account_active', compute_sudo=True)
    allow_timesheets = fields.Boolean(
        "Allow timesheets",
        compute='_compute_allow_timesheets', compute_sudo=True,
        search='_search_allow_timesheets', readonly=True,
        help="Timesheets can be logged on this task.")
    remaining_hours = fields.Float("Remaining Hours", compute='_compute_remaining_hours', store=True, readonly=True, help="Number of allocated hours minus the number of hours spent.")
    remaining_hours_percentage = fields.Float(compute='_compute_remaining_hours_percentage', search='_search_remaining_hours_percentage')
    effective_hours = fields.Float("Hours Spent", compute='_compute_effective_hours', compute_sudo=True, store=True)
    total_hours_spent = fields.Float("Total Hours", compute='_compute_total_hours_spent', store=True, help="Time spent on this task and its sub-tasks (and their own sub-tasks).")
    progress = fields.Float("Progress", compute='_compute_progress_hours', store=True, group_operator="avg")
    overtime = fields.Float(compute='_compute_progress_hours', store=True)
    subtask_effective_hours = fields.Float("Sub-tasks Hours Spent", compute='_compute_subtask_effective_hours', recursive=True, store=True, help="Time spent on the sub-tasks (and their own sub-tasks) of this task.")
    timesheet_ids = fields.One2many('account.analytic.line', 'task_id', 'Timesheets')
    encode_uom_in_days = fields.Boolean(compute='_compute_encode_uom_in_days', default=lambda self: self._uom_in_days())

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS | PROJECT_TASK_READABLE_FIELDS

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
        timesheet_read_group = self.env['account.analytic.line'].read_group([('task_id', 'in', self.ids)], ['unit_amount', 'task_id'], ['task_id'])
        timesheets_per_task = {res['task_id'][0]: res['unit_amount'] for res in timesheet_read_group}
        for task in self:
            task.effective_hours = timesheets_per_task.get(task.id, 0.0)

    @api.depends('effective_hours', 'subtask_effective_hours', 'planned_hours')
    def _compute_progress_hours(self):
        for task in self:
            if (task.planned_hours > 0.0):
                task_total_hours = task.effective_hours + task.subtask_effective_hours
                task.overtime = max(task_total_hours - task.planned_hours, 0)
                if float_compare(task_total_hours, task.planned_hours, precision_digits=2) >= 0:
                    task.progress = 100
                else:
                    task.progress = round(100.0 * task_total_hours / task.planned_hours, 2)
            else:
                task.progress = 0.0
                task.overtime = 0

    @api.depends('planned_hours', 'remaining_hours')
    def _compute_remaining_hours_percentage(self):
        for task in self:
            if task.planned_hours > 0.0:
                task.remaining_hours_percentage = task.remaining_hours / task.planned_hours
            else:
                task.remaining_hours_percentage = 0.0

    def _search_remaining_hours_percentage(self, operator, value):
        if operator not in OPERATOR_MAPPING:
            raise NotImplementedError(_('This operator %s is not supported in this search method.', operator))
        query = f"""
            SELECT id
              FROM {self._table}
             WHERE remaining_hours > 0
               AND planned_hours > 0
               AND remaining_hours / planned_hours {operator} %s
            """
        return [('id', 'inselect', (query, (value,)))]

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
        for task in self.with_context(active_test=False):
            task.subtask_effective_hours = sum(child_task.effective_hours + child_task.subtask_effective_hours for child_task in task.child_ids)

    def action_view_subtask_timesheet(self):
        self.ensure_one()
        task_ids = self.with_context(active_test=False)._get_subtask_ids_per_task_id().get(self.id, [])
        action = self.env["ir.actions.actions"]._for_xml_id("hr_timesheet.timesheet_action_all")
        graph_view_id = self.env.ref("hr_timesheet.view_hr_timesheet_line_graph_by_employee").id
        new_views = []
        for view in action['views']:
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

    def write(self, values):
        # a timesheet must have an analytic account (and a project)
        if 'project_id' in values and not values.get('project_id') and self._get_timesheet():
            raise UserError(_('This task must be part of a project because there are some timesheets linked to it.'))
        res = super(Task, self).write(values)

        if 'project_id' in values:
            project = self.env['project.project'].browse(values.get('project_id'))
            if project.allow_timesheets:
                # We write on all non yet invoiced timesheet the new project_id (if project allow timesheet)
                self._get_timesheet().write({'project_id': values.get('project_id')})

        return res

    def name_get(self):
        if self.env.context.get('hr_timesheet_display_remaining_hours'):
            name_mapping = dict(super().name_get())
            for task in self:
                if task.allow_timesheets and task.planned_hours > 0 and task.encode_uom_in_days:
                    days_left = _("(%s days remaining)") % task._convert_hours_to_days(task.remaining_hours)
                    name_mapping[task.id] = name_mapping.get(task.id, '') + u"\u00A0" + days_left
                elif task.allow_timesheets and task.planned_hours > 0:
                    hours, mins = (str(int(duration)).rjust(2, '0') for duration in divmod(abs(task.remaining_hours) * 60, 60))
                    hours_left = _(
                        "(%(sign)s%(hours)s:%(minutes)s remaining)",
                        sign='-' if task.remaining_hours < 0 else '',
                        hours=hours,
                        minutes=mins,
                    )
                    name_mapping[task.id] = name_mapping.get(task.id, '') + u"\u00A0" + hours_left
            return list(name_mapping.items())
        return super().name_get()

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
            ['task_id'],
        )
        task_with_timesheets_ids = [res['task_id'][0] for res in timesheet_data]
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

    module_project_timesheet_synchro = fields.Boolean("Awesome Timesheet",
        compute="_compute_timesheet_modules", store=True, readonly=False)
    module_project_timesheet_holidays = fields.Boolean("Time Off",
        compute="_compute_timesheet_modules", store=True, readonly=False)
    reminder_user_allow = fields.Boolean(string="Employee Reminder")
    reminder_manager_allow = fields.Boolean(string="Manager Reminder")
    project_time_mode_id = fields.Many2one(
        'uom.uom', related='company_id.project_time_mode_id', string='Project Time Unit', readonly=False,
        help="This will set the unit of measure used in projects and tasks.\n"
             "If you use the timesheet linked to projects, don't "
             "forget to setup the right unit of measure in your employees.")
    timesheet_encode_uom_id = fields.Many2one('uom.uom', string="Encoding Unit",
        related='company_id.timesheet_encode_uom_id', readonly=False)
    is_encode_uom_days = fields.Boolean(compute='_compute_is_encode_uom_days')

    @api.depends('timesheet_encode_uom_id')
    def _compute_is_encode_uom_days(self):
        product_uom_day = self.env.ref('uom.product_uom_day')
        for settings in self:
            settings.is_encode_uom_days = settings.timesheet_encode_uom_id == product_uom_day

    @api.depends('module_hr_timesheet')
    def _compute_timesheet_modules(self):
        self.filtered(lambda config: not config.module_hr_timesheet).update({
            'module_project_timesheet_synchro': False,
            'module_project_timesheet_holidays': False,
        })

```

## File: models\uom.py

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

from . import hr_timesheet
from . import ir_http
from . import ir_ui_menu
from . import res_company
from . import res_config_settings
from . import project
from . import project_collaborator
from . import uom

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
            <field name="arch" type="xml">
                <search string="Timesheet">
                    <field name="date"/>
                    <field name="employee_id"/>
                    <field name="project_id"/>
                    <field name="ancestor_task_id" groups="project.group_subtask_project"/>
                    <field name="task_id"/>
                    <field name="name"/>
                    <field name="department_id"/>
                    <field name="manager_id"/>
                    <filter name="mine" string="My Timesheets" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter name="month" string="Date" date="date"/>
                    <group expand="0" string="Group By">
                        <filter string="Project" name="groupby_project" domain="[]" context="{'group_by': 'project_id'}"/>
                        <filter string="Ancestor Task" name="groupby_parent_task" domain="[]" context="{'group_by': 'ancestor_task_id'}" groups="project.group_subtask_project"/>
                        <filter string="Task" name="groupby_task" domain="[]" context="{'group_by': 'task_id'}"/>
                        <filter string="Date" name="groupby_date" domain="[]" context="{'group_by': 'date'}" help="Timesheet by Date"/>
                        <filter string="Department" name="groupby_department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter string="Manager" name="groupby_manager" domain="[]" context="{'group_by': 'manager_id'}"/>
                        <filter string="Employee" name="groupby_employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                    </group>
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

        <menuitem id="menu_timesheets_reports"
            name="Reporting"
            parent="timesheet_menu_root"
            sequence="99"/>

        <menuitem id="menu_timesheets_reports_timesheet"
            name="Timesheets"
            parent="menu_timesheets_reports"
            sequence="10"/>

        <menuitem id="menu_hr_activity_analysis"
            parent="menu_timesheets_reports_timesheet"
            action="act_hr_timesheet_report"
            groups="hr_timesheet.group_hr_timesheet_approver"
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

from odoo import fields, models, api


class ReportProjectTaskUser(models.Model):
    _inherit = "report.project.task.user"

    hours_planned = fields.Float('Planned Hours', readonly=True)
    hours_effective = fields.Float('Effective Hours', readonly=True)
    remaining_hours = fields.Float('Remaining Hours', readonly=True)
    progress = fields.Float('Progress', group_operator='avg', readonly=True)
    overtime = fields.Float(readonly=True)

    def _select(self):
        select_to_append = """,
                (t.effective_hours * 100) / NULLIF(t.planned_hours, 0) as progress,
                t.effective_hours as hours_effective,
                t.planned_hours - t.effective_hours - t.subtask_effective_hours as remaining_hours,
                NULLIF(t.planned_hours, 0) as hours_planned,
                t.overtime as overtime
        """
        return super(ReportProjectTaskUser, self)._select() + select_to_append

    def _group_by(self):
        group_by_append = """,
                t.effective_hours,
                t.subtask_effective_hours,
                t.planned_hours,
                t.overtime
        """
        return super(ReportProjectTaskUser, self)._group_by() + group_by_append

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
                    <field name="hours_planned" widget="timesheet_uom" type="measure"/>
                    <field name="hours_effective" widget="timesheet_uom" type="measure"/>
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
                    <field name="hours_planned" widget="timesheet_uom" type="measure"/>
                    <field name="hours_effective" widget="timesheet_uom" type="measure"/>
                    <field name="remaining_hours" widget="timesheet_uom" type="measure"/>
                </xpath>
             </field>
        </record>

        <record id="report_project_task_user_view_search" model="ir.ui.view">
            <field name="name">report.project.task.user.view.search.inherit.hr.timesheet</field>
            <field name="model">report.project.task.user</field>
            <field name="inherit_id" ref="project.view_task_project_user_search" />
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='late']" position="after">
                    <filter string="Tasks in Overtime" name="overtime" domain="[('overtime', '&gt;', 0)]"/>
                </xpath>
                <xpath expr="//filter[@name='my_favorite_projects']" position="after">
                    <filter string="My Team's Projects" name="my_team_projects"  domain="[('project_id.user_id.employee_id.parent_id.user_id', '=', uid), ('project_id.user_id', '!=', uid), ('user_ids', '!=', uid), ('user_ids', '!=', False)]"/>
                    <filter string="My Department's Projects" name="my_department" domain="[('project_id.user_id.employee_id.member_of_department', '=', True)]"/>
                </xpath>
                <xpath expr="//filter[@name='followed_by_me']" position="after">
                    <filter string="My Team's Tasks" name="my_team_tasks" domain="[('user_ids.employee_id.parent_id.user_id', '=', uid)]" />
                    <filter string="My Department's Tasks" name="my_department" domain="[('user_ids.employee_id.member_of_department', '=', True)]"/>
                </xpath>
             </field>
         </record>

        <record id="report_project_task_user_view_tree" model="ir.ui.view">
            <field name="name">report.project.task.user.view.tree.inherit.hr.timesheet</field>
            <field name="model">report.project.task.user</field>
            <field name="inherit_id" ref="project.report_project_task_user_view_tree"/>
            <field name="arch" type="xml">
                <field name="user_ids" position="after">
                    <field name="hours_effective" optional="hide" widget="float_time"/>
                    <field name="progress" optional="hide" widget="progressbar"/>
                </field>
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
                            <th class="text-start align-middle"><span>Date</span></th>
                            <th class="text-start align-middle"><span>Employee</span></th>
                            <th class="text-start align-middle" t-if="show_project"><span>Project</span></th>
                            <th class="text-start align-middle" t-if="show_task"><span>Task</span></th>
                            <th class="text-start align-middle"><span>Description</span></th>
                            <th class="text-end">
                                <span t-if="is_uom_day">Days Spent</span>
                                <span t-else="">Hours Spent</span>
                            </th>
                        </tr>
                   </thead>
                   <tbody>
                        <tr t-foreach="lines" t-as="line" t-att-style="'background-color: #F1F1F1;' if line_index % 2 == 0 else ''">
                            <td class="align-middle">
                               <span t-field="line.date"/>
                            </td>
                            <td class="align-middle">
                               <span t-field="line.user_id.partner_id.name"/>
                               <span t-if="not line.user_id.partner_id.name" t-field="line.employee_id"/>
                            </td>
                            <td t-if="show_project" class="align-middle">
                                <span t-field="line.project_id.sudo().name"/>
                            </td>
                            <td t-if="show_task" class="align-middle">
                                <span t-if="line.task_id" t-field="line.task_id.sudo().name"/>
                            </td>
                            <td class="align-middle">
                                <span t-field="line.name" t-options="{'widget': 'text'}"/>
                            </td>
                            <td class="text-end align-middle">
                                <span t-if="is_uom_day" t-esc="line._get_timesheet_time_day()" t-options="{'widget': 'timesheet_uom'}"/>
                                <span t-else="" t-field="line.unit_amount" t-options="{'widget': 'duration', 'digital': True, 'unit': 'hour', 'round': 'minute'}"/>
                            </td>
                        </tr>
                        <tr>
                            <t t-set="nbCols" t-value="4"/>
                            <t t-if="show_project" t-set="nbCols" t-value="nbCols + 1"/>
                            <t t-if="show_task" t-set="nbCols" t-value="nbCols + 1"/>
                            <td class="text-end" t-attf-colspan="{{nbCols}}">
                                <strong t-if="is_uom_day">
                                    <span style="margin-right: 15px;">Total (Days)</span>
                                    <t t-esc="lines._convert_hours_to_days(sum(lines.mapped('unit_amount')))" t-options="{'widget': 'timesheet_uom'}"/>
                                </strong>
                                <strong t-else="">
                                    <span style="margin-right: 15px;">Total (Hours)</span>
                                    <t t-esc="sum(lines.mapped('unit_amount'))" t-options="{'widget': 'duration', 'digital': True, 'unit': 'hour', 'round': 'minute'}"/>
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
            <t t-call="web.external_layout">
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
            <t t-call="web.external_layout">
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
                                        for the <t t-out="docs.project_id.name"/> Project
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
            <t t-call="web.external_layout">
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
    ancestor_task_id = fields.Many2one("project.task", string="Ancestor Task", readonly=True)
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
                A.ancestor_task_id AS ancestor_task_id,
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
access_project_task,project.task.timesheet.user,model_project_task,hr_timesheet.group_hr_timesheet_user,1,1,0,0
access_timesheets_analysis_report_manager,timesheets.analysis.report,model_timesheets_analysis_report,hr_timesheet.group_timesheet_manager,1,1,1,1
access_timesheets_analysis_report_user,timesheets.analysis.report,model_timesheets_analysis_report,hr_timesheet.group_hr_timesheet_user,1,0,1,0

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

## File: static\src\components\task_with_hours\task_with_hours.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { Many2OneField } from "@web/views/fields/many2one/many2one_field";


class TaskWithHours extends Many2OneField {

    get canCreate() {
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
        return {...super.context, hr_timesheet_display_remaining_hours: true};
    }

    /**
     * @override
     */
    get Many2XAutocompleteProps() {
        const props = super.Many2XAutocompleteProps;
        if (!this.canCreate) {
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
        activeActions.create = activeActions.create && this.canCreate;
        activeActions.createEdit = activeActions.create;
    }

}

registry.category("fields").add("task_with_hours", TaskWithHours);

```

## File: static\src\components\timesheet_uom\timesheet_uom.js

```javascript
/** @odoo-module */

import { useService } from "@web/core/utils/hooks";
import { session } from "@web/session";
import { registry } from "@web/core/registry";
import { FloatFactorField } from "@web/views/fields/float_factor/float_factor_field";
import { FloatToggleField } from "@web/views/fields/float_toggle/float_toggle_field";
import { FloatTimeField } from "@web/views/fields/float_time/float_time_field";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

const { Component } = owl;


export class TimesheetUOM extends Component {

    setup() {
        this.companyService = useService("company");
    }

    get timesheetUOMId() {
        return this.companyService.currentCompany.timesheet_uom_id;
    }

    get timesheetWidget() {
        let timesheet_widget = "float_factor";
        if (this.timesheetUOMId in session.uom_ids) {
            timesheet_widget = session.uom_ids[this.timesheetUOMId].timesheet_widget;
        }
        return timesheet_widget;
    }

    get timesheetComponent() {
        return registry.category("fields").get(this.timesheetWidget, FloatFactorField);
    }

    get timesheetComponentProps() {
        const factorDependantComponents = ["float_toggle", "float_factor"];
        return factorDependantComponents.includes(this.timesheetWidget) ? this.FactorCompanyDependentProps : this.props;
    }

    get FactorCompanyDependentProps() {
        const factor = this.companyService.currentCompany.timesheet_uom_factor || this.props.factor;
        return { ...this.props, factor };
    }

}

TimesheetUOM.props = {
    ...standardFieldProps,
};

TimesheetUOM.template = "hr_timesheet.TimesheetUOM";

TimesheetUOM.components = { FloatFactorField, FloatToggleField, FloatTimeField };

registry.category("fields").add("timesheet_uom", TimesheetUOM);

```

## File: static\src\components\timesheet_uom\timesheet_uom.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>

    <t t-name="hr_timesheet.TimesheetUOM" owl="1">
        <t t-component="timesheetComponent" t-props="timesheetComponentProps"/>
    </t>

</templates>

```

## File: static\src\components\timesheet_uom_no_toggle\timesheet_uom_no_toggle.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";

import { TimesheetUOM } from "../timesheet_uom/timesheet_uom";


export class TimesheetUOMNoToggle extends TimesheetUOM {

    get timesheetWidget() {
        const timesheetWidget = super.timesheetWidget;
        return timesheetWidget !== "float_toggle" ? timesheetWidget : "float_factor";
    }

}

// As FloatToggleField won't be used by TimesheetUOMNoToggle, we remove it from the components that we get from TimesheetUOM.
delete TimesheetUOMNoToggle.components.FloatToggleField;

registry.category("fields").add("timesheet_uom_no_toggle", TimesheetUOMNoToggle);

```

## File: static\src\js\task_with_hours.js

```javascript
/** @odoo-module alias=hr_timesheet.task_with_hours **/

import field_registry from 'web.field_registry';
import TimesheetFieldMany2One from 'hr_timesheet.TimesheetFieldMany2one';

const TaskWithHours = TimesheetFieldMany2One.extend({
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.additionalContext.hr_timesheet_display_remaining_hours = true;
        // By default, we keep the no_quick_create value or we set to false.
        this.nodeOptions.no_quick_create = this.nodeOptions.no_quick_create || false;
    },
    /**
     * @override
     */
    _getDisplayNameWithoutHours: function (value) {
        return value && value.split('\u00A0')[0];
    },
    /**
     * @override
     * @private
     */
    _onInputClick: function () {
        const context = Object.assign(
            this.record.getContext(this.recordParams),
            this.additionalContext
        );
        // We don't want to quick create if no project is set in the timesheet
        const canCreate = 'default_project_id' in context && context.default_project_id;
        this.nodeOptions.no_quick_create =
            this.nodeOptions.no_quick_create || !canCreate;
        this.can_create = this.can_create && canCreate;
        this._super.apply(this, arguments);
    },
    /**
     * @override
     * @private
     */
    _renderEdit: function (){
        this.m2o_value = this._getDisplayNameWithoutHours(this.m2o_value);
        this._super.apply(this, arguments);
    },
});

field_registry.add('task_with_hours', TaskWithHours);

export default TaskWithHours;

```

## File: static\src\js\timesheet_field_many2one.js

```javascript
/** @odoo-module alias=hr_timesheet.TimesheetFieldMany2one **/

import FieldRegistry from 'web.field_registry';
import { FieldMany2One } from 'web.relational_fields';

import { SelectCreateDialog } from "@web/views/view_dialogs/select_create_dialog";
import { Component } from "@odoo/owl";

const TimesheetFieldMany2one = FieldMany2One.extend({
    /**
     * @override
     * @private
     */
    _searchCreatePopup(view, ids, context, dynamicFilters) {
        const options = this._getSearchCreatePopupOptions(view, ids, context, dynamicFilters);
        Component.env.services.dialog.add(SelectCreateDialog, {
            title: options.title,
            resModel: options.res_model,
            multiSelect: false,
            domain: options.domain,
            context: options.context,
            noCreate: options.no_create,
            onSelected: (resId) => {
                return this.reinitialize(resId);
            },
            onClose: () => {
                this.activate();
            }
        });
    },
});

FieldRegistry.add('timesheet_field_many2one', TimesheetFieldMany2one);

export default TimesheetFieldMany2one;

```

## File: static\src\js\timesheet_uom.js

```javascript
odoo.define('hr_timesheet.timesheet_uom', function (require) {
'use strict';

const { registry } = require("@web/core/registry");
const basicFields = require('web.basic_fields');
const fieldUtils = require('web.field_utils');

const fieldRegistry = require('web.field_registry');

// We need the field registry to be populated, as we bind the
// timesheet_uom widget on existing field widgets.
require('web._field_registry');

const session = require('web.session');

const TimesheetUOMMultiCompanyMixin = {
    init: function(parent, name, record, options) {
        this._super(parent, name, record, options);
        const currentCompanyId = session.user_context.allowed_company_ids[0];
        const currentCompany = session.user_companies.allowed_companies[currentCompanyId];
        this.currentCompanyTimesheetUOMFactor = currentCompany.timesheet_uom_factor || 1;
    }
};

/**
 * Extend the float factor widget to set default value for timesheet
 * use case. The 'factor' is forced to be the UoM timesheet
 * conversion from the session info.
 **/
const FieldTimesheetFactor = basicFields.FieldFloatFactor.extend(TimesheetUOMMultiCompanyMixin).extend({
    formatType: 'float_factor',
    /**
     * Override init to tweak options depending on the session info
     *
     * @constructor
     * @override
     */
    init: function(parent, name, record, options) {
        this._super(parent, name, record, options);

        // force factor in format and parse options
        this.nodeOptions.factor = this.currentCompanyTimesheetUOMFactor;
        this.parseOptions.factor = this.currentCompanyTimesheetUOMFactor;
    },
});


/**
 * Extend the float toggle widget to set default value for timesheet
 * use case. The 'range' is different from the default one of the
 * native widget, and the 'factor' is forced to be the UoM timesheet
 * conversion.
 **/
const FieldTimesheetToggle = basicFields.FieldFloatToggle.extend(TimesheetUOMMultiCompanyMixin).extend({
    formatType: 'float_factor',
    /**
     * Override init to tweak options depending on the session info
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
        this.nodeOptions.factor = this.currentCompanyTimesheetUOMFactor;
    },
});


/**
 * Extend float time widget
 */
const FieldTimesheetTime = basicFields.FieldFloatTime.extend(TimesheetUOMMultiCompanyMixin).extend({
    init: function () {
        this._super.apply(this, arguments);
        this.nodeOptions.factor = this.currentCompanyTimesheetUOMFactor;
        this.parseOptions.factor = this.currentCompanyTimesheetUOMFactor;
    }
});

const timesheetUomService = {
    dependencies: ["legacy_session"],
    start() {
        const timesheetUomInfo = {
            widget: null,
            factor: 1,
        };
        if (session.user_context &&
            session.user_context.allowed_company_ids &&
            session.user_context.allowed_company_ids.length) {
            const currentCompanyId = session.user_context.allowed_company_ids[0];
            const currentCompany = session.user_companies.allowed_companies[currentCompanyId];
            const currentCompanyTimesheetUOMId = currentCompany.timesheet_uom_id || false;
            timesheetUomInfo.factor = currentCompany.timesheet_uom_factor || 1;
            if (currentCompanyTimesheetUOMId) {
                timesheetUomInfo.widget = session.uom_ids[currentCompanyTimesheetUOMId].timesheet_widget;
            }
        }

        /**
         * Binding depending on Company Preference
         *
         * determine wich widget will be the timesheet one.
         * Simply match the 'timesheet_uom' widget key with the correct
         * implementation (float_time, float_toggle, ...). The default
         * value will be 'float_factor'.
         **/
        const widgetName = timesheetUomInfo.widget || 'float_factor';

        let FieldTimesheetUom = null;

        if (widgetName === 'float_toggle') {
            FieldTimesheetUom = FieldTimesheetToggle;
        } else if (widgetName === 'float_time') {
            FieldTimesheetUom = FieldTimesheetTime;
        } else {
            FieldTimesheetUom = (
                fieldRegistry.get(widgetName) &&
                fieldRegistry.get(widgetName).extend({ })
            ) || FieldTimesheetFactor;
        }
        fieldRegistry.add('timesheet_uom', FieldTimesheetUom);

        // widget timesheet_uom_no_toggle is the same as timesheet_uom but without toggle.
        // We can modify easly huge amount of days.
        let FieldTimesheetUomWithoutToggle = null;
        if (widgetName === 'float_toggle') {
            FieldTimesheetUomWithoutToggle = FieldTimesheetFactor;
        } else {
            FieldTimesheetUomWithoutToggle = FieldTimesheetTime;
        }
        fieldRegistry.add('timesheet_uom_no_toggle', FieldTimesheetUomWithoutToggle);


        // bind the formatter and parser method, and tweak the options
        const _tweak_options = (options) => {
            if (!_.contains(options, 'factor')) {
                options.factor = timesheetUomInfo.factor;
            }
            return options;
        };

        fieldUtils.format.timesheet_uom = function(value, field, options) {
            options = _tweak_options(options || { });
            const formatter = fieldUtils.format[FieldTimesheetUom.prototype.formatType];
            return formatter(value, field, options);
        };

        fieldUtils.parse.timesheet_uom = function(value, field, options) {
            options = _tweak_options(options || { });
            const parser = fieldUtils.parse[FieldTimesheetUom.prototype.formatType];
            return parser(value, field, options);
        };

        fieldUtils.format.timesheet_uom_no_toggle = function(value, field, options) {
            options = _tweak_options(options || { });
            const formatter = fieldUtils.format[FieldTimesheetUom.prototype.formatType];
            return formatter(value, field, options);
        };

        fieldUtils.parse.timesheet_uom_no_toggle = function(value, field, options) {
            options = _tweak_options(options || { });
            const parser = fieldUtils.parse[FieldTimesheetUom.prototype.formatType];
            return parser(value, field, options);
        };
        if (!registry.category("formatters").contains("timesheet_uom")) {
            registry.category("formatters").add("timesheet_uom", fieldUtils.format.timesheet_uom);
        }
        if (!registry.category("formatters").contains("timesheet_uom_no_toggle")) {
            registry.category("formatters").add("timesheet_uom_no_toggle", fieldUtils.format.timesheet_uom_no_toggle);
        }
        return timesheetUomInfo;
    },
};
registry.category("services").add("timesheet_uom", timesheetUomService);

return {
    FieldTimesheetFactor,
    FieldTimesheetTime,
    FieldTimesheetToggle,
    timesheetUomService,
};

});

```

## File: static\src\views\timesheet_graph\timesheet_graph_model.js

```javascript
/** @odoo-module **/

import { GraphModel } from "@web/views/graph/graph_model";

const FIELDS = [
    'unit_amount', 'effective_hours', 'planned_hours', 'remaining_hours', 'total_hours_spent', 'subtask_effective_hours',
    'overtime', 'number_hours', 'difference', 'hours_effective', 'hours_planned', 'timesheet_unit_amount'
];

export class hrTimesheetGraphModel extends GraphModel {
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
hrTimesheetGraphModel.services = [...GraphModel.services, "company"];

```

## File: static\src\views\timesheet_graph\timesheet_graph_view.js

```javascript
/** @odoo-module **/

import { projectGraphView } from "@project/js/project_graph_view";
import { hrTimesheetGraphModel } from "./timesheet_graph_model";
import { registry } from "@web/core/registry";

const viewRegistry = registry.category("views");

export const hrTimesheetGraphView = {
  ...projectGraphView,
  Model: hrTimesheetGraphModel,
};

viewRegistry.add("hr_timesheet_graphview", hrTimesheetGraphView);

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
        <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="title">Timesheets</t>
                <t t-set="url" t-value="'/my/timesheets'"/>
                <t t-set="placeholder_count" t-value="'timesheet_count'"/>
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
            sequence="75"
            groups="group_hr_timesheet_user"
            web_icon="hr_timesheet,static/description/icon_timesheet.png"/>

        <menuitem id="menu_hr_time_tracking"
            name="Timesheets"
            parent="timesheet_menu_root"
            groups = "group_hr_timesheet_approver"
            sequence="5"/>

        <!--
            Timesheet line Views
        -->
        <record id="hr_timesheet_line_tree" model="ir.ui.view">
            <field name="name">account.analytic.line.tree.hr_timesheet</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <tree editable="bottom" string="Timesheet Activities" sample="1">
                    <field name="date"/>
                    <field name="employee_id" invisible="1"/>
                    <field name="project_id" required="1" options="{'no_create_edit': True, 'no_open': 1}"/>
                    <field name="task_id" optional="show" options="{'no_create_edit': True, 'no_open': True}" widget="task_with_hours" context="{'default_project_id': project_id}"/>
                    <field name="name" optional="show" required="0"/>
                    <field name="unit_amount" optional="show" widget="timesheet_uom" sum="Total" decoration-danger="unit_amount &gt; 24 or unit_amount &lt; 0"/>
                    <field name="company_id" invisible="1"/>
                    <field name="user_id" invisible="1"/>
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
                                <field name="project_id" required="1"/>
                                <field name="task_id" widget="task_with_hours" context="{'default_project_id': project_id}"/>
                                <field name="name"/>
                                <field name="company_id" groups="base.group_multi_company" invisible="1"/>
                            </group>
                            <group>
                                <field name="date"/>
                                <field name="amount" invisible="1"/>
                                <field name="unit_amount" widget="timesheet_uom" decoration-danger="unit_amount &gt; 24"/>
                                <field name="currency_id" invisible="1"/>
                                <field name="company_id" invisible="1"/>
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
                <xpath expr="//field[@name='date']" position="before">
                    <field name="employee_id" required="1" context="{'active_test': True}"/>
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
                    <field name="ancestor_task_id" groups="project.group_subtask_project"/>
                    <field name="task_id"/>
                    <field name="name"/>
                    <field name="department_id"/>
                    <field name="manager_id"/>
                    <filter name="mine" string="My Timesheets" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter name="month" string="Date" date="date"/>
                    <group expand="0" string="Group By">
                        <filter string="Project" name="groupby_project" domain="[]" context="{'group_by': 'project_id'}"/>
                        <filter string="Ancestor Task" name="groupby_parent_task" domain="[]" context="{'group_by': 'ancestor_task_id'}" groups="project.group_subtask_project"/>
                        <filter string="Task" name="groupby_task" domain="[]" context="{'group_by': 'task_id'}"/>
                        <filter string="Date" name="groupby_date" domain="[]" context="{'group_by': 'date'}" help="Timesheet by Date"/>
                        <filter string="Department" name="groupby_department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter string="Manager" name="groupby_manager" domain="[]" context="{'group_by': 'manager_id'}"/>
                        <filter string="Employee" name="groupby_employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                    </group>
                </search>
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
                                            <strong>Duration: </strong><field name="unit_amount" widget="timesheet_uom" decoration-danger="unit_amount &gt; 24"/>
                                        </span>
                                    </div>
                                </div>
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
            <field name="view_mode">tree,form,kanban,pivot,graph</field>
            <field name="domain">[('project_id', '!=', False), ('user_id', '=', uid)]</field>
            <field name="context">{
                "search_default_week":1,
                "is_timesheet": 1,
            }</field>
            <field name="search_view_id" ref="hr_timesheet_line_my_timesheet_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No timesheets found. Let's create one!
              </p>
              <p>
                Keep track of your working hours by project every day and bill your customers for that time.
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
            <field name="sequence">7</field>
            <field name="view_id" ref="hr_timesheet.view_my_timesheet_line_pivot"/>
            <field name="act_window_id" ref="act_hr_timesheet_line"/>
        </record>

        <record id="act_hr_timesheet_line_view_graph" model="ir.actions.act_window.view">
            <field name="view_mode">graph</field>
            <field name="sequence">8</field>
            <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_graph_my"/>
            <field name="act_window_id" ref="act_hr_timesheet_line"/>
        </record>

        <menuitem id="timesheet_menu_activity_mine"
            name="My Timesheets"
            groups="group_hr_timesheet_approver"
            parent="menu_hr_time_tracking"
            action="act_hr_timesheet_line"/>

        <menuitem id="timesheet_menu_activity_user"
            name="My Timesheets"
            groups="group_hr_timesheet_user"
            parent="timesheet_menu_root"
            action="act_hr_timesheet_line"/>

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
                    No timesheets found. Let's create one!
              </p>
              <p>
                Keep track of your working hours by project every day and bill your customers for that time.
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

        <menuitem id="timesheet_menu_activity_all"
            name="All Timesheets"
            parent="menu_hr_time_tracking"
            action="timesheet_action_all"
            groups="hr_timesheet.group_hr_timesheet_approver"/>

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
            'default_employee_id': active_id,
            "is_timesheet": 1,
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
        <field name="inherit_id" ref="hr_hourly_cost.view_employee_form"/>
        <field name="priority" eval="40"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" type="action" name="%(timesheet_action_from_employee)d" icon="fa-calendar" groups="hr_timesheet.group_hr_timesheet_user">
                    <div class="o_stat_info">
                        <span class="o_stat_text">Timesheets</span>
                    </div>
                </button>
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
                        context="{ 'search_default_department_id': [active_id], 'default_department_id': active_id}">
                        Timesheets
                    </a>
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
        <xpath expr="//li[@id='task-nav']" position="before">
            <li t-if="timesheets and allow_timesheets" class="list-group-item d-grid flex-grow-1" id='nav-report'>
                <div class="o_download_pdf d-flex gap-1">
                    <a class="btn btn-secondary flex-fill o_download_btn" t-att-href="task.get_portal_url(report_type='pdf', download=True)" title="Download"><i class="fa fa-download"/> Download</a>
                    <a class="btn btn-secondary flex-fill o_print_btn o_project_timesheet_print" t-att-href="task.get_portal_url(report_type='pdf')" href="#" title="Print" target="_blank"><i class="fa fa-print"/> Print</a>
                </div>
            </li>
        </xpath>
        <xpath expr="//li[@id='nav-header']" position="after">
            <li t-if="timesheets and allow_timesheets" class="nav-item">
                <a class="nav-link ps-3" href="#task_timesheets">
                    Timesheets
                </a>
            </li>
        </xpath>
        <xpath expr="//div[@id='card_body']" position="inside">
            <div class="container" t-if="timesheets and allow_timesheets">
                <hr class="mt-4 mb-1"/>
                <h5 id="task_timesheets" class="mt-2 mb-2" data-anchor="true">Timesheets</h5>
                <t t-call="hr_timesheet.portal_timesheet_table"/>
            </div>
        </xpath>
        <xpath expr="//div[@name='portal_my_task_planned_hours']" position="after">
            <div t-if="task.planned_hours > 0"><strong>Progress:</strong> <span t-field="task.progress"/>%</div>
        </xpath>
        <xpath expr="//div[@name='portal_my_task_planned_hours']/t" position="replace">
            <t t-call="hr_timesheet.portal_my_task_planned_hours_template"></t>
        </xpath>
    </template>

    <template id="portal_my_task_planned_hours_template">
        <t t-if="is_uom_day and timesheets._convert_hours_to_days(task.planned_hours) > 0">
            <strong>Allocated Days:</strong> <span t-esc="timesheets._convert_hours_to_days(task.planned_hours)" t-options='{"widget": "timesheet_uom"}'/>
        </t>
        <t t-if="not is_uom_day and task.planned_hours > 0" t-call="project.portal_my_task_planned_hours_template"></t>
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
            <th t-if="is_uom_day" class="text-end">Days Spent</th>
            <th t-else="" class="text-end">Hours Spent</th>
        </xpath>
        <xpath expr="//tbody/t/tr/td[@name='project_portal_milestones']" position="after">
            <td class="text-end">
                <t t-if="is_uom_day">
                    <t t-out="timesheet_ids._convert_hours_to_days(task.effective_hours)"/>
                    <span t-if="task.planned_hours > 0"> / <t t-out="timesheet_ids._convert_hours_to_days(task.planned_hours)"/></span>
                </t>
                <t t-else="">
                    <span t-field="task.effective_hours" t-options='{"widget": "float_time"}'/>
                    <t t-if="task.planned_hours > 0">
                        /
                        <span t-field="task.planned_hours" t-options='{"widget": "float_time"}'/>
                    </t>
                </t>
            </td>
        </xpath>
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
            <tfoot>
                <tr>
                    <th colspan="3"></th>
                    <th class="text-end">
                        <t t-set="timesheets_amount" t-value="round(sum(timesheets.mapped('unit_amount')), 2) or 0.0"></t>
                        <div t-if="is_uom_day"><strong>Days Spent:</strong> <span t-esc="timesheets._convert_hours_to_days(timesheets_amount)" t-options='{"widget": "timesheet_uom"}'/></div>
                        <div t-else=""><strong>Hours Spent:</strong> <span t-esc="timesheets_amount" t-options='{"widget": "float_time"}'/></div>
                        <t t-set="timesheets_by_subtask_amount" t-value="round(sum(sum(timesheet_by_subtask.mapped('unit_amount') or 0.0) for timesheet_by_subtask in timesheets_by_subtask.values()), 2) or 0.0"></t>
                        <div t-if="timesheets_by_subtask">
                            <div t-if="is_uom_day">Days recorded on sub-tasks: <span t-esc="timesheets._convert_hours_to_days(timesheets_by_subtask_amount)" t-options='{"widget": "timesheet_uom"}'/></div>
                            <div t-else="">Hours recorded on sub-tasks: <span t-esc="timesheets_by_subtask_amount" t-options='{"widget": "float_time"}'/></div>
                        </div>
                        <t t-set="planned_time" t-value="task.planned_hours"></t>
                        <div t-if="planned_time > 0" name="planned_time" t-attf-class="{{task.remaining_hours &lt; 0 and 'text-danger' or ''}}">
                            <div t-if="is_uom_day">Remaining Days: <span t-esc="timesheets._convert_hours_to_days(planned_time - timesheets_amount - timesheets_by_subtask_amount)" t-options='{"widget": "timesheet_uom"}'/></div>
                            <div t-else="">Remaining Hours: <span t-esc="planned_time - timesheets_amount - timesheets_by_subtask_amount" t-options='{"widget": "float_time"}'/></div>
                        </div>
                    </th>
                </tr>
            </tfoot>
        </table>
    </template>

</odoo>

```

## File: views\project_sharing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_sharing_inherit_project_task_view_form" model="ir.ui.view">
        <field name="name">project.sharing.project.task.view.form.inherit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='child_ids']/tree/field[@name='portal_user_names']" position="after">
                <field name="planned_hours" widget="timesheet_uom_no_toggle" sum="Initially Planned Hours" optional="hide"/>
                <field name="effective_hours" widget="timesheet_uom" sum="Effective Hours" optional="hide"/>
                <field name="subtask_effective_hours" string="Sub-tasks Hours Spent" widget="timesheet_uom" sum="Sub-tasks Hours Spent" optional="hide"/>
                <field name="total_hours_spent" string="Total Hours" widget="timesheet_uom" sum="Total Hours" optional="hide"/>
                <field name="remaining_hours" widget="timesheet_uom" sum="Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100"/>
                <field name="progress" widget="progressbar" optional="hide"/>
            </xpath>
            <xpath expr="//notebook/page[@name='description_page']" position="after">
                <field name="analytic_account_active" invisible="1"/>
                <field name="allow_timesheets" invisible="1"/>
                <field name="encode_uom_in_days" invisible="1"/>
                <field name="subtask_count" invisible="1"/>
                <page string="Timesheets" id="timesheets_tab" attrs="{'invisible': [('allow_timesheets', '=', False)]}">
                    <group>
                        <group>
                            <div colspan="2">
                                <label for="planned_hours" string="Allocated Hours" class="me-2" attrs="{'invisible': [('encode_uom_in_days', '=', True)]}"/>
                                <label for="planned_hours" string="Allocated Days" class="me-2" attrs="{'invisible': [('encode_uom_in_days', '=', False)]}"/>
                                <field name="planned_hours" class="o_field_float_time oe_inline ms-2" widget="timesheet_uom_no_toggle"/>
                                <span attrs="{'invisible': ['|', ('allow_subtasks', '=', False), ('subtask_count', '=', 0)]}">
                                    (incl. <field name="subtask_planned_hours" nolabel="1" groups="project.group_subtask_project" widget="timesheet_uom_no_toggle" class="oe_inline"/> on
                                    <span class="fw-bold text-dark"> Sub-tasks</span>)
                                </span>
                            </div>
                        </group>
                        <group>
                            <field name="progress" widget="progressbar"/>
                        </group>
                    </group>
                    <field name="timesheet_ids" mode="tree,kanban"
                          attrs="{'invisible': [('analytic_account_active', '=', False)]}">
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
                    <group attrs="{'invisible': [('analytic_account_active', '=', False)]}">
                        <group class="oe_subtotal_footer oe_right" name="project_hours">
                            <span class="o_td_label float-start">
                                <label class="fw-bold" for="effective_hours" string="Hours Spent" attrs="{'invisible': [('encode_uom_in_days', '=', True)]}"/>
                                <label class="fw-bold" for="effective_hours" string="Days Spent" attrs="{'invisible': [('encode_uom_in_days', '=', False)]}"/>
                            </span>
                            <field name="effective_hours" widget="timesheet_uom" nolabel="1"/>
                            <!-- remove o_form_subtask_button class from master-->
                            <button name="action_view_subtask_timesheet" type="object" class="o_form_subtask_button ps-0 border-0 oe_inline oe_link mb-2 o_td_label float-start" attrs="{'invisible' : ['|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0)]}">
                                <span class="text-nowrap" attrs="{'invisible' : [('encode_uom_in_days', '=', True)]}">Hours Spent on Sub-tasks:</span>
                                <span class="text-nowrap" attrs="{'invisible' : [('encode_uom_in_days', '=', False)]}">Days Spent on Sub-tasks:</span>
                            </button>
                            <field name="subtask_effective_hours" class="mt-2" widget="timesheet_uom"
                                  attrs="{'invisible' : ['|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0)]}" nolabel="1"/>
                            <span id="total_hours_spent_label" attrs="{'invisible': ['|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0)]}" class="o_td_label float-start">
                                <label class="fw-bold" for="total_hours_spent" string="Total Hours"
                                      attrs="{'invisible': [('encode_uom_in_days', '=', True)]}"/>
                                <label class="fw-bold" for="total_hours_spent" string="Total Days"
                                      attrs="{'invisible': [('encode_uom_in_days', '=', False)]}"/>
                            </span>
                            <field name="total_hours_spent" widget="timesheet_uom" class="oe_subtotal_footer_separator" nolabel="1"
                                  attrs="{'invisible' : ['|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0)]}" />
                            <span class="o_td_label float-start">
                                <label class="fw-bold" for="remaining_hours" string="Remaining Hours"
                                       attrs="{'invisible': ['|', '|', ('planned_hours', '=', 0.0), ('encode_uom_in_days', '=', True), ('remaining_hours', '&lt;', 0)]}"/>
                                <label class="fw-bold" for="remaining_hours" string="Remaining Days"
                                       attrs="{'invisible': ['|', '|', ('planned_hours', '=', 0.0), ('encode_uom_in_days', '=', False), ('remaining_hours', '&lt;', 0)]}"/>
                                <label class="fw-bold text-danger" for="remaining_hours" string="Remaining Hours"
                                       attrs="{'invisible': ['|', '|', ('planned_hours', '=', 0.0), ('encode_uom_in_days', '=', True), ('remaining_hours', '&gt;=', 0)]}"/>
                                <label class="fw-bold text-danger" for="remaining_hours" string="Remaining Days"
                                       attrs="{'invisible': ['|', '|', ('planned_hours', '=', 0.0), ('encode_uom_in_days', '=', False), ('remaining_hours', '&gt;=', 0)]}"/>
                            </span>
                            <field name="remaining_hours" widget="timesheet_uom" class="oe_subtotal_footer_separator"
                                  attrs="{'invisible' : [('planned_hours', '=', 0.0)]}" nolabel="1"/>
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
                <field name="planned_hours" />
                <field name="allow_timesheets"/>
                <field name="encode_uom_in_days" invisible="1"/>
            </templates>
            <div class="oe_kanban_bottom_left" position="inside">
                <t name="planned_hours" t-if="record.planned_hours.raw_value &gt; 0 and record.allow_timesheets.raw_value">
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

    <record id="project_sharing_inherit_project_task_view_tree" model="ir.ui.view">
        <field name="name">project.task.tree.inherited</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_tree" />
        <field name="arch" type="xml">
            <field name="portal_user_names" position="after">
                <field name="allow_subtasks" invisible="1"/>
                <field name="allow_timesheets" invisible="1"/>
                <field name="planned_hours" widget="timesheet_uom_no_toggle" sum="Initially Planned Hours" optional="hide" attrs="{'column_invisible': [('allow_timesheets', '=', False)]}"/>
                <field name="effective_hours" widget="timesheet_uom" sum="Effective Hours" optional="hide" attrs="{'column_invisible': [('allow_timesheets', '=', False)]}"/>
                <field name="subtask_effective_hours" widget="timesheet_uom" attrs="{'column_invisible' : ['|', ('allow_subtasks', '=', False), ('allow_timesheets', '=', False)]}" optional="hide"/>
                <field name="total_hours_spent" widget="timesheet_uom" attrs="{'column_invisible' : ['|', ('allow_subtasks', '=', False), ('allow_timesheets', '=', False)]}" optional="hide"/>
                <field name="remaining_hours" widget="timesheet_uom" sum="Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100" attrs="{'column_invisible': [('allow_timesheets', '=', False)]}"/>
                <field name="progress" widget="progressbar" attrs="{'column_invisible': [('allow_timesheets', '=', False)]}" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="project_sharing_project_task_view_search_inherit_timesheet" model="ir.ui.view">
        <field name="name">project.sharing.project.task.view.search.inherit.timesheet</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='late']" position='after'>
                <filter string="Tasks in Overtime" name="overtime" domain="[('overtime', '&gt;', 0)]"/>
                <filter string="Tasks Soon in Overtime" name="remaining_hours_percentage" domain="[('remaining_hours_percentage', '&lt;=', 0.2)]"/>
            </xpath>
        </field>
    </record>

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
    </data>
</odoo>

```

## File: views\project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
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

        <record id="project_project_view_form_simplified_inherit_timesheet" model="ir.ui.view">
            <field name="name">project.project.view.form.simplified.inherit.timesheet</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.project_project_view_form_simplified"/>
            <field name="priority">24</field>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('o_settings_container')]" position="inside">
                    <div class="col-lg-6 o_setting_box">
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
                </xpath>
            </field>
        </record>

        <record id="project_invoice_form" model="ir.ui.view">
            <field name="name">Inherit project form : Invoicing Data</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.edit_project"/>
            <field name="priority">24</field>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='dates']" position="after">
                    <field name="allocated_hours" widget="timesheet_uom_no_toggle" attrs="{'invisible': [('allow_timesheets', '=', False)]}"/>
                </xpath>
                <xpath expr="//group[@name='group_time_managment']" position="attributes">
                    <attribute name="invisible">0</attribute>
                </xpath>
                <xpath expr="//group[@name='group_time_managment']" position="inside">
                    <div class="o_setting_box" id="timesheet_settings" colspan="2">
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
                </xpath>
            </field>
        </record>

        <record model="ir.ui.view" id="view_task_form2_inherited">
            <field name="name">project.task.form.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_form2" />
            <field name="arch" type="xml">
                <xpath expr="//field[@name='child_ids']/tree/field[@name='company_id']" position="after">
                    <field name="planned_hours" widget="timesheet_uom_no_toggle" sum="Initially Planned Hours" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="effective_hours" widget="timesheet_uom" sum="Effective Hours" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="subtask_effective_hours" string="Sub-tasks Hours Spent" widget="timesheet_uom" sum="Sub-tasks Hours Spent" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="total_hours_spent" string="Total Hours" widget="timesheet_uom" sum="Total Hours" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="remaining_hours" widget="timesheet_uom" sum="Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="progress" widget="progressbar" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                </xpath>
                <xpath expr="//notebook/page[@name='description_page']" position="after">
                    <field name="allow_timesheets" invisible="1"/>
                    <field name="allow_subtasks" invisible="1"/>
                    <t groups="hr_timesheet.group_hr_timesheet_user">
                        <field name="analytic_account_active" invisible="1"/>
                        <field name="encode_uom_in_days" invisible="1"/>
                        <field name="subtask_count" invisible="1"/>
                    </t>
                    <page string="Timesheets" id="timesheets_tab" attrs="{'invisible': [('allow_timesheets', '=', False)]}" groups="hr_timesheet.group_hr_timesheet_user">
                        <group>
                            <group>
                                <label for="planned_hours" string="Allocated Hours" attrs="{'invisible': [('encode_uom_in_days', '=', True)]}"/>
                                <label for="planned_hours" string="Allocated Days" attrs="{'invisible': [('encode_uom_in_days', '=', False)]}"/>
                                <div class="o_row">
                                    <field name="planned_hours" class="o_field_float_time oe_inline ms-2" widget="timesheet_uom_no_toggle"/>
                                    <span attrs="{'invisible': ['|', ('allow_subtasks', '=', False), ('subtask_count', '=', 0)]}">
                                        (incl. <field name="subtask_planned_hours" nolabel="1" groups="project.group_subtask_project" widget="timesheet_uom_no_toggle" class="oe_inline"/> on
                                        <span class="fw-bold text-dark"> Sub-tasks</span>)
                                    </span>
                                </div>
                            </group>
                            <group>
                                <field name="progress" widget="progressbar"/>
                            </group>
                        </group>
                        <group name="timesheet_error" attrs="{'invisible': [('analytic_account_active', '!=', False)]}">
                            <div class="alert alert-warning" role="alert" colspan="2">
                                You cannot log timesheets on this project since it is linked to an inactive analytic account. Please change this account, or reactivate the current one to timesheet on the project.
                            </div>
                        </group>
                    <field name="timesheet_ids" mode="tree,kanban" attrs="{'invisible': [('analytic_account_active', '=', False)]}" context="{'default_project_id': project_id, 'default_name':''}">
                        <tree editable="bottom" string="Timesheet Activities" default_order="date">
                            <field name="date"/>
                            <field name="user_id" invisible="1"/>
                            <field name="employee_id" required="1" widget="many2one_avatar_employee" context="{'active_test': True}" options="{'relation': 'hr.employee.public'}" groups="!hr.group_hr_user"/>
                            <field name="employee_id" required="1" widget="many2one_avatar_employee" context="{'active_test': True}" options="{'relation': 'hr.employee'}" groups="hr.group_hr_user"/>
                            <field name="name" required="0"/>
                            <field name="unit_amount" widget="timesheet_uom" decoration-danger="unit_amount &gt; 24 or unit_amount &lt; 0"/>
                            <field name="project_id" invisible="1"/>
                            <field name="task_id" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                        </tree>
                        <kanban class="o_kanban_mobile">
                            <field name="date"/>
                            <field name="user_id"/>
                            <field name="name"/>
                            <field name="unit_amount" decoration-danger="unit_amount &gt; 24"/>
                            <field name="project_id"/>
                            <field name="task_id" invisible="1"/>
                            <templates>
                                <t t-name="kanban-box">
                                    <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                        <div class="row">
                                            <div class="col-6">
                                                <field name="employee_id" widget="many2one_avatar_employee" context="{'active_test': True}" options="{'relation': 'hr.employee.public'}" groups="!hr.group_hr_user"/>
                                                <field name="employee_id" widget="many2one_avatar_employee" context="{'active_test': True}" options="{'relation': 'hr.employee'}" groups="hr.group_hr_user"/>
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
                                    <field name="date"/>
                                    <field name="user_id" invisible="1"/>
                                    <field name="employee_id" required="1" widget="many2one_avatar_employee" context="{'active_test': True}" options="{'relation': 'hr.employee.public'}" groups="!hr.group_hr_user"/>
                                    <field name="employee_id" required="1" widget="many2one_avatar_employee" context="{'active_test': True}" options="{'relation': 'hr.employee'}" groups="hr.group_hr_user"/>
                                    <field name="name" required="0"/>
                                    <field name="unit_amount" string="Duration" widget="float_time" decoration-danger="unit_amount &gt; 24"/>
                                    <field name="project_id" invisible="1"/>
                                    <field name="task_id" invisible="1"/>
                                    <field name="company_id" invisible="1"/>
                                </group>
                            </sheet>
                        </form>
                    </field>
                    <group attrs="{'invisible': [('analytic_account_active', '=', False)]}">
                        <group class="oe_subtotal_footer oe_right" name="project_hours">
                            <span class="o_td_label float-start">
                                <label class="fw-bold" for="effective_hours" string="Hours Spent" attrs="{'invisible': [('encode_uom_in_days', '=', True)]}"/>
                                <label class="fw-bold" for="effective_hours" string="Days Spent" attrs="{'invisible': [('encode_uom_in_days', '=', False)]}"/>
                            </span>
                            <field name="effective_hours" widget="timesheet_uom" nolabel="1"/>
                            <!-- remove o_form_subtask_button class from master-->
                            <button name="action_view_subtask_timesheet" type="object" class="o_form_subtask_button ps-0 border-0 oe_inline oe_link mb-2 o_td_label float-start" attrs="{'invisible' : ['|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0)]}">
                                <span class="text-nowrap" attrs="{'invisible' : [('encode_uom_in_days', '=', True)]}">Hours Spent on Sub-tasks:</span>
                                <span class="text-nowrap" attrs="{'invisible' : [('encode_uom_in_days', '=', False)]}">Days Spent on Sub-tasks:</span>
                            </button>
                            <field name="subtask_effective_hours" class="mt-2" widget="timesheet_uom"
                                   attrs="{'invisible' : ['|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0)]}" nolabel="1"/>
                            <span attrs="{'invisible': ['|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0)]}" class="o_td_label float-start">
                                <label class="fw-bold" for="total_hours_spent" string="Total Hours"
                                       attrs="{'invisible': ['|', '|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0), ('encode_uom_in_days', '=', True)]}"/>
                                <label class="fw-bold" for="total_hours_spent" string="Total Days"
                                       attrs="{'invisible': ['|', '|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0), ('encode_uom_in_days', '=', False)]}"/>
                            </span>
                            <field name="total_hours_spent" widget="timesheet_uom" class="oe_subtotal_footer_separator" nolabel="1"
                                   attrs="{'invisible' : ['|', ('allow_subtasks', '=', False), ('subtask_effective_hours', '=', 0.0)]}" />
                            <span class="o_td_label float-start" attrs="{'invisible': [('planned_hours', '=', 0.0)]}">
                                <label class="fw-bold" for="remaining_hours" string="Remaining Hours"
                                       attrs="{'invisible': ['|', ('encode_uom_in_days', '=', True), ('remaining_hours', '&lt;', 0)]}"/>
                                <label class="fw-bold" for="remaining_hours" string="Remaining Days"
                                       attrs="{'invisible': ['|', ('encode_uom_in_days', '=', False), ('remaining_hours', '&lt;', 0)]}"/>
                                <label class="fw-bold text-danger" for="remaining_hours" string="Remaining Hours"
                                       attrs="{'invisible': ['|', ('encode_uom_in_days', '=', True), ('remaining_hours', '&gt;=', 0)]}"/>
                                <label class="fw-bold text-danger" for="remaining_hours" string="Remaining Days"
                                       attrs="{'invisible': ['|', ('encode_uom_in_days', '=', False), ('remaining_hours', '&gt;=', 0)]}"/>
                            </span>
                            <field name="remaining_hours" widget="timesheet_uom" class="oe_subtotal_footer_separator"
                                   attrs="{'invisible' : [('planned_hours', '=', 0.0)]}" nolabel="1"/>
                        </group>
                    </group>
                </page>
                </xpath>
                <xpath expr="//field[@name='depend_on_ids']/tree//field[@name='company_id']" position="after">
                    <field name="allow_subtasks" invisible="1" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="progress" invisible="1" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="planned_hours" widget="timesheet_uom_no_toggle" sum="Initially Planned Hours" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="effective_hours" widget="timesheet_uom" sum="Effective Hours" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="subtask_effective_hours" widget="timesheet_uom" attrs="{'invisible' : [('allow_subtasks', '=', False)]}" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="total_hours_spent" widget="timesheet_uom" attrs="{'invisible' : [('allow_subtasks', '=', False)]}" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="remaining_hours" widget="timesheet_uom" sum="Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100" groups="hr_timesheet.group_hr_timesheet_user"/>
                    <field name="progress" widget="progressbar" optional="hide" groups="hr_timesheet.group_hr_timesheet_user"/>
                </xpath>
            </field>
        </record>

        <record id="project_project_view_tree_inherit_sale_project" model="ir.ui.view">
            <field name="name">project.project.tree.inherit.sale.timesheet</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='date']" position="after">
                    <field name="allocated_hours" widget="timesheet_uom_no_toggle" optional="show" attrs="{'invisible' : [('allocated_hours', '=', 0)]}"/>
                </xpath>
            </field>
        </record>

        <record id="view_task_tree2_inherited" model="ir.ui.view">
            <field name="name">project.task.tree.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_tree2" />
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <field name="company_id" position="after">
                    <field name="allow_subtasks" invisible="1"/>
                    <field name="progress" invisible="1"/>
                    <field name="planned_hours" widget="timesheet_uom_no_toggle" sum="Initially Planned Hours" optional="hide"/>
                    <field name="effective_hours" widget="timesheet_uom" sum="Effective Hours" optional="hide"/>
                    <field name="subtask_effective_hours" widget="timesheet_uom" attrs="{'invisible' : [('allow_subtasks', '=', False)]}" optional="hide"/>
                    <field name="total_hours_spent" widget="timesheet_uom" attrs="{'invisible' : [('allow_subtasks', '=', False)]}" optional="hide"/>
                    <field name="remaining_hours" widget="timesheet_uom" sum="Remaining Hours" optional="hide" decoration-danger="progress &gt;= 100" decoration-warning="progress &gt;= 80 and progress &lt; 100"/>
                    <field name="progress" widget="progressbar" optional="hide" groups="hr_timesheet.group_hr_timesheet_user" attrs="{'invisible' : [('planned_hours', '=', 0)]}"/>
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
                    <field name="timesheet_count" invisible="1"/>
                    <field name="remaining_hours" invisible="1"/>
                    <field name="encode_uom_in_days" invisible="1"/>
                    <field name="allocated_hours" invisible="1"/>
                </field>
                <xpath expr="//div[hasclass('o_kanban_manage_view')]" position="inside">
                    <div role="menuitem" t-if="record.allow_timesheets.raw_value and record.timesheet_count and record.timesheet_count.raw_value" groups="hr_timesheet.group_hr_timesheet_user">
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

        <record id="view_task_kanban_inherited_progress" model="ir.ui.view">
            <field name="name">project.task.timesheet.kanban.inherited.progress</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_kanban"/>
            <field name="arch" type="xml">
                <templates position="before">
                    <field name="progress" />
                    <field name="remaining_hours" />
                    <field name="planned_hours" />
                    <field name="allow_timesheets"/>
                    <field name="encode_uom_in_days" invisible="1"/>
                </templates>
                <div class="oe_kanban_bottom_left" position="inside">
                   <t name="planned_hours" t-if="record.planned_hours.raw_value &gt; 0 and record.allow_timesheets.raw_value" groups="hr_timesheet.group_hr_timesheet_user">
                        <t t-set="badge" t-value=""/>
                        <t t-set="badge" t-value="'border-warning'" t-if="record.progress.raw_value &gt;= 80 and record.progress.raw_value &lt;= 100"/>
                        <t t-set="badge" t-value="'border-danger'" t-if="record.remaining_hours.raw_value &lt; 0"/>
                        <t t-set="title" t-value="'Remaining days'" t-if="record.encode_uom_in_days.raw_value"/>
                        <t t-set="title" t-value="'Remaining hours'" t-else=""/>
                        <div t-attf-class="oe_kanban_align badge {{ badge }}" t-att-title="title">
                            <field name="remaining_hours" widget="timesheet_uom" />
                        </div>
                   </t>
                </div>
             </field>
         </record>

        <record id="project_task_view_search" model="ir.ui.view">
            <field name="name">project.task.view.search.inherit.sale.timesheet.enterprise</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_search_form"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='followed_by_me']" position='after'>
                    <filter string="My Team's Tasks" name="my_team_tasks" domain="[('user_ids.employee_parent_id.user_id', '=', uid)]"/>
                    <filter string="My Department's Tasks" name="my_department" domain="[('user_ids.employee_id.member_of_department', '=', True)]"/>
                </xpath>
                <xpath expr="//filter[@name='tasks_due_today']" position='after'>
                    <filter string="Tasks in Overtime" name="overtime" domain="[('overtime', '&gt;', 0)]"/>
                    <filter string="Tasks Soon in Overtime" name="remaining_hours_percentage" domain="[('remaining_hours_percentage', '&lt;=', 0.2)]"/>
                </xpath>
            </field>
        </record>

        <record id="view_task_search_form_hr_extended" model="ir.ui.view">
            <field name="name">project.task.view.search.inherit.hr.timesheet</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_search_form_extended"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='my_favorite_projects']" position='after'>
                    <filter string="My Team's Projects" name="my_teams_projects" domain="[('project_id.user_id.employee_parent_id.user_id', '=', uid)]"/>
                    <filter string="My Department's Projects" name="my_department" domain="[('manager_id.employee_id.member_of_department', '=', True)]"/>
                </xpath>
            </field>
        </record>

        <record id="view_project_project_filter_inherit_timesheet" model="ir.ui.view">
            <field name="name">project.project.view.inherit.timesheet</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project_project_filter"/>
            <field name="arch" type="xml">
                <filter name="followed_by_me" position='before'>
                    <filter string="My Team" name="my_team_projects"  domain="[('user_id.employee_parent_id.user_id', '=', uid)]"/>
                    <filter string="My Department" name="my_department" domain="[('user_id.employee_id.member_of_department', '=', True)]"/>
                </filter>
                <filter name="late_milestones" position="before">
                    <filter string="Projects in Overtime" name="projects_in_overtime" domain="[('is_project_overtime', '=', True)]"/>
                </filter>
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
                    <field name="planned_hours" widget="timesheet_uom"/>
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
                <xpath expr="//field[@name='stage_id']" position='after'>
                    <field name="planned_hours" widget="timesheet_uom"/>
                    <field name="remaining_hours" widget="timesheet_uom"/>
                    <field name="effective_hours" widget="timesheet_uom"/>
                    <field name="total_hours_spent" widget="timesheet_uom"/>
                    <field name="overtime" widget="timesheet_uom"/>
                    <field name="subtask_effective_hours" widget="timesheet_uom"/>
                </xpath>
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

## File: views\rating_rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="rating_rating_view_search_project_inherited" model="ir.ui.view">
        <field name="name">rating.rating.search.project.inherited</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="project.rating_rating_view_search_project"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='my_ratings']" position="after">
                <filter string="My Team's Ratings" name="my_team_rating" domain="[('rated_partner_id.user_ids.employee_parent_id.user_id', '=', uid)]"/>
                <filter string="My Department's Ratings" name="my_department_rating" domain="[('rated_partner_id.user_ids.employee_id.member_of_department', '=', True)]"/>
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
        <field name="name">res.config.settings.view.form.inherit.hr.timesheet</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="55"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Timesheets" string="Timesheets" data-key="hr_timesheet" groups="hr_timesheet.group_timesheet_manager" id="timesheets">
                    <h2>Time Encoding</h2>
                    <div class="row mt16 o_settings_container" name="time_encoding_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="time_mode_setting"
                            attrs="{'invisible':[('project_time_mode_id', '!=', False)]}">
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
                        <div class="col-12 col-lg-6 o_setting_box" id="time_unit_timesheets_setting">
                            <div class="o_setting_right_pane">
                                <label for="timesheet_encode_uom_id"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="row">
                                    <div class="text-muted col-md-12">
                                        Time unit used to record your timesheets
                                    </div>
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="timesheet_encode_uom_id" options="{'no_create': True, 'no_open': True}" required="1" class="col-lg-5 ps-0"/>
                                        <field name="is_encode_uom_days" invisible="1"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="synchronize_web_mobile_setting" invisible="1">
                            <div class="o_setting_left_pane">
                                <field name="module_project_timesheet_synchro" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_project_timesheet_synchro"/>
                                <div class="text-muted">
                                    Track your time from anywhere, even offline, with our web/mobile apps
                                </div>
                                <div class="content-group">
                                    <div class="row mt16 oe_center">
                                        <div class="col-lg-3 pe-0 o_chrome_store_link d-none d-sm-inline-block">
                                            <a href="http://www.odoo.com/app/timesheet?platform=chrome" class="align-middle" target="_blank">
                                                <img alt="Google Chrome Store" class="img img-fluid align-middle mt-1" style="height: 85% !important;" src="project/static/src/img/chrome_store.png"/>
                                            </a>
                                        </div>
                                        <div class="col-lg-3 pe-0">
                                            <a href="https://apps.apple.com/be/app/awesome-timesheet/id1078657549" class="align-middle" target="_blank">
                                                <img alt="Apple App Store" class="img img-fluid h-100 o_config_app_store" src="project/static/src/img/app_store.png"/>
                                            </a>
                                        </div>
                                        <div class="col-lg-3 pe-0">
                                            <a href="https://play.google.com/store/apps/details?id=com.odoo.OdooTimesheets" class="align-middle" target="_blank">
                                                <img alt="Google Play Store" class="img img-fluid h-100 o_config_play_store" src="project/static/src/img/play_store.png"/>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Timesheets Control</h2>
                    <div class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="reminder_user_allow" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane" id="reminder_user_allow">
                                <label for="reminder_user_allow"/>
                                <span class="fa fa-lg fa-building-o " title="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="text-muted">
                                    Send a periodical email reminder to timesheets users<br/>
                                    that still have timesheets to encode
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="reminder_manager_allow" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane" id="reminder_manager_allow">
                                <label for="reminder_manager_allow"/>
                                <span class="fa fa-lg fa-building-o " title="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="text-muted">
                                    Send a periodical email reminder to timesheets managers<br/>
                                    that still have timesheets to validate
                                </div>
                            </div>
                        </div>
                    </div>
                    <div name="section_leaves">
                        <h2>Time Off</h2>
                        <div class="row mt16 o_settings_container" name="timesheet_control">
                            <div class="col-12 col-lg-6 o_setting_box" id="timesheet_off_validation_setting">
                                <div class="o_setting_left_pane">
                                    <field name="module_project_timesheet_holidays"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_project_timesheet_holidays"/>
                                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                    <div class="text-muted">
                                        Generate timesheets upon time off validation
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

    <menuitem id="hr_timesheet_menu_configuration" name="Configuration" parent="timesheet_menu_root"
        action="hr_timesheet_config_settings_action" groups="base.group_system" sequence="100"/>
</odoo>

```

