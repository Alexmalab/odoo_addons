# Odoo Module: project

Category: Services/Project

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

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Project',
    'version': '1.1',
    'website': 'https://www.odoo.com/page/project-management',
    'category': 'Services/Project',
    'sequence': 45,
    'summary': 'Organize and plan your projects',
    'depends': [
        'analytic',
        'base_setup',
        'mail',
        'portal',
        'rating',
        'resource',
        'web',
        'web_tour',
        'digest',
    ],
    'description': "",
    'data': [
        'security/project_security.xml',
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'report/project_report_views.xml',
        'views/analytic_views.xml',
        'views/digest_views.xml',
        'views/rating_views.xml',
        'views/project_views.xml',
        'views/res_partner_views.xml',
        'views/res_config_settings_views.xml',
        'views/mail_activity_views.xml',
        'views/project_assets.xml',
        'views/project_portal_templates.xml',
        'data/digest_data.xml',
        'data/project_mail_template_data.xml',
        'data/project_data.xml',
        'wizard/project_delete_wizard_views.xml',
        'wizard/project_task_type_delete_views.xml',
    ],
    'demo': ['data/project_demo.xml'],
    'qweb': ['static/src/xml/project_templates.xml'],
    'test': [
    ],
    'installable': True,
    'auto_install': False,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import OrderedDict
from operator import itemgetter

from odoo import http, _
from odoo.exceptions import AccessError, MissingError
from odoo.http import request
from odoo.addons.portal.controllers.portal import CustomerPortal, pager as portal_pager
from odoo.tools import groupby as groupbyelem

from odoo.osv.expression import OR


class CustomerPortal(CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        if 'project_count' in counters:
            values['project_count'] = request.env['project.project'].search_count([]) \
                if request.env['project.project'].check_access_rights('read', raise_exception=False) else 0
        if 'task_count' in counters:
            values['task_count'] = request.env['project.task'].search_count([]) \
                if request.env['project.task'].check_access_rights('read', raise_exception=False) else 0
        return values

    # ------------------------------------------------------------
    # My Project
    # ------------------------------------------------------------
    def _project_get_page_view_values(self, project, access_token, **kwargs):
        values = {
            'page_name': 'project',
            'project': project,
        }
        return self._get_page_view_values(project, access_token, values, 'my_projects_history', False, **kwargs)

    @http.route(['/my/projects', '/my/projects/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_projects(self, page=1, date_begin=None, date_end=None, sortby=None, **kw):
        values = self._prepare_portal_layout_values()
        Project = request.env['project.project']
        domain = []

        searchbar_sortings = {
            'date': {'label': _('Newest'), 'order': 'create_date desc'},
            'name': {'label': _('Name'), 'order': 'name'},
        }
        if not sortby:
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        # projects count
        project_count = Project.search_count(domain)
        # pager
        pager = portal_pager(
            url="/my/projects",
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby},
            total=project_count,
            page=page,
            step=self._items_per_page
        )

        # content according to pager and archive selected
        projects = Project.search(domain, order=order, limit=self._items_per_page, offset=pager['offset'])
        request.session['my_projects_history'] = projects.ids[:100]

        values.update({
            'date': date_begin,
            'date_end': date_end,
            'projects': projects,
            'page_name': 'project',
            'default_url': '/my/projects',
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby
        })
        return request.render("project.portal_my_projects", values)

    @http.route(['/my/project/<int:project_id>'], type='http', auth="public", website=True)
    def portal_my_project(self, project_id=None, access_token=None, **kw):
        try:
            project_sudo = self._document_check_access('project.project', project_id, access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        values = self._project_get_page_view_values(project_sudo, access_token, **kw)
        return request.render("project.portal_my_project", values)

    # ------------------------------------------------------------
    # My Task
    # ------------------------------------------------------------
    def _task_get_page_view_values(self, task, access_token, **kwargs):
        values = {
            'page_name': 'task',
            'task': task,
            'user': request.env.user
        }
        return self._get_page_view_values(task, access_token, values, 'my_tasks_history', False, **kwargs)

    @http.route(['/my/tasks', '/my/tasks/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_tasks(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, search=None, search_in='content', groupby=None, **kw):
        values = self._prepare_portal_layout_values()
        searchbar_sortings = {
            'date': {'label': _('Newest'), 'order': 'create_date desc'},
            'name': {'label': _('Title'), 'order': 'name'},
            'stage': {'label': _('Stage'), 'order': 'stage_id, project_id'},
            'project': {'label': _('Project'), 'order': 'project_id, stage_id'},
            'update': {'label': _('Last Stage Update'), 'order': 'date_last_stage_update desc'},
        }
        searchbar_filters = {
            'all': {'label': _('All'), 'domain': []},
        }
        searchbar_inputs = {
            'content': {'input': 'content', 'label': _('Search <span class="nolabel"> (in Content)</span>')},
            'message': {'input': 'message', 'label': _('Search in Messages')},
            'customer': {'input': 'customer', 'label': _('Search in Customer')},
            'stage': {'input': 'stage', 'label': _('Search in Stages')},
            'project': {'input': 'project', 'label': _('Search in Project')},
            'all': {'input': 'all', 'label': _('Search in All')},
        }
        searchbar_groupby = {
            'none': {'input': 'none', 'label': _('None')},
            'project': {'input': 'project', 'label': _('Project')},
            'stage': {'input': 'stage', 'label': _('Stage')},
        }

        # extends filterby criteria with project the customer has access to
        projects = request.env['project.project'].search([])
        for project in projects:
            searchbar_filters.update({
                str(project.id): {'label': project.name, 'domain': [('project_id', '=', project.id)]}
            })

        # extends filterby criteria with project (criteria name is the project id)
        # Note: portal users can't view projects they don't follow
        project_groups = request.env['project.task'].read_group([('project_id', 'not in', projects.ids)],
                                                                ['project_id'], ['project_id'])
        for group in project_groups:
            proj_id = group['project_id'][0] if group['project_id'] else False
            proj_name = group['project_id'][1] if group['project_id'] else _('Others')
            searchbar_filters.update({
                str(proj_id): {'label': proj_name, 'domain': [('project_id', '=', proj_id)]}
            })

        # default sort by value
        if not sortby:
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']

        # default filter by value
        if not filterby:
            filterby = 'all'
        domain = searchbar_filters.get(filterby, searchbar_filters.get('all'))['domain']

        # default group by value
        if not groupby:
            groupby = 'project'

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        # search
        if search and search_in:
            search_domain = []
            if search_in in ('content', 'all'):
                search_domain = OR([search_domain, ['|', ('name', 'ilike', search), ('description', 'ilike', search)]])
            if search_in in ('customer', 'all'):
                search_domain = OR([search_domain, [('partner_id', 'ilike', search)]])
            if search_in in ('message', 'all'):
                search_domain = OR([search_domain, [('message_ids.body', 'ilike', search)]])
            if search_in in ('stage', 'all'):
                search_domain = OR([search_domain, [('stage_id', 'ilike', search)]])
            if search_in in ('project', 'all'):
                search_domain = OR([search_domain, [('project_id', 'ilike', search)]])
            domain += search_domain

        # task count
        task_count = request.env['project.task'].search_count(domain)
        # pager
        pager = portal_pager(
            url="/my/tasks",
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby, 'filterby': filterby, 'groupby': groupby, 'search_in': search_in, 'search': search},
            total=task_count,
            page=page,
            step=self._items_per_page
        )
        # content according to pager and archive selected
        if groupby == 'project':
            order = "project_id, %s" % order  # force sort on project first to group by project in view
        elif groupby == 'stage':
            order = "stage_id, %s" % order  # force sort on stage first to group by stage in view

        tasks = request.env['project.task'].search(domain, order=order, limit=self._items_per_page, offset=pager['offset'])
        request.session['my_tasks_history'] = tasks.ids[:100]

        if groupby == 'project':
            grouped_tasks = [request.env['project.task'].concat(*g) for k, g in groupbyelem(tasks, itemgetter('project_id'))]
        elif groupby == 'stage':
            grouped_tasks = [request.env['project.task'].concat(*g) for k, g in groupbyelem(tasks, itemgetter('stage_id'))]
        else:
            grouped_tasks = [tasks] if tasks else []

        values.update({
            'date': date_begin,
            'date_end': date_end,
            'grouped_tasks': grouped_tasks,
            'page_name': 'task',
            'default_url': '/my/tasks',
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'searchbar_groupby': searchbar_groupby,
            'searchbar_inputs': searchbar_inputs,
            'search_in': search_in,
            'search': search,
            'sortby': sortby,
            'groupby': groupby,
            'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
            'filterby': filterby,
        })
        return request.render("project.portal_my_tasks", values)

    @http.route(['/my/task/<int:task_id>'], type='http', auth="public", website=True)
    def portal_my_task(self, task_id, access_token=None, **kw):
        try:
            task_sudo = self._document_check_access('project.task', task_id, access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        # ensure attachment are accessible with access token inside template
        for attachment in task_sudo.attachment_ids:
            attachment.generate_access_token()
        values = self._task_get_page_view_values(task_sudo, access_token, **kw)
        return request.render("project.portal_my_task", values)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest.digest_digest_default" model="digest.digest">
            <field name="kpi_project_task_opened">True</field>
        </record>
    </data>

    <data>
        <record id="digest_tip_project_0" model="digest.tip">
            <field name="name">Tip: Customize tasks and stages according to the project</field>
            <field name="sequence">1200</field>
            <field name="group_id" ref="project.group_project_manager"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Customize tasks and stages according to the project</p>
    <p class="tip_content">Customize how tasks are named according to the project and create tailor made status messages for each step of the workflow. It helps to document your workflow: what should be done at which step.</p>
    <img src="/project/static/src/img/project-custom-tasks.gif" class="illustration_border" />
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\project_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

        <record id="ir_cron_rating_project" model="ir.cron">
            <field name="name">Project: Send rating</field>
            <field name="model_id" ref="project.model_project_project"/>
            <field name="state">code</field>
            <field name="code">model._send_rating_all()</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
        </record>

        <record id="ir_cron_recurring_tasks" model="ir.cron">
            <field name="name">Project: Create Recurring Tasks</field>
            <field name="model_id" ref="project.model_project_task_recurrence"/>
            <field name="state">code</field>
            <field name="code">model._cron_create_recurring_tasks()</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
            <field name="nextcall" eval="(DateTime.now().replace(hour=3, minute=0) + timedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')" />
        </record>

        <!-- Task-related subtypes for messaging / Chatter -->
        <record id="mt_task_new" model="mail.message.subtype">
            <field name="name">Task Created</field>
            <field name="res_model">project.task</field>
            <field name="default" eval="False"/>
            <field name="hidden" eval="True"/>
            <field name="description">Task Created</field>
        </record>
        <record id="mt_task_blocked" model="mail.message.subtype">
            <field name="name">Task Blocked</field>
            <field name="res_model">project.task</field>
            <field name="default" eval="False"/>
            <field name="description">Task blocked</field>
        </record>
        <record id="mt_task_ready" model="mail.message.subtype">
            <field name="name">Task Ready</field>
            <field name="res_model">project.task</field>
            <field name="default" eval="False"/>
            <field name="description">Task ready for Next Stage</field>
        </record>
        <record id="mt_task_stage" model="mail.message.subtype">
            <field name="name">Stage Changed</field>
            <field name="res_model">project.task</field>
            <field name="default" eval="False"/>
            <field name="description">Stage changed</field>
        </record>
        <record id="mt_task_rating" model="mail.message.subtype">
            <field name="name">Task Rating</field>
            <field name="res_model">project.task</field>
            <field name="default" eval="False"/>
            <field name="description">Ratings</field>
        </record>
        <!-- Project-related subtypes for messaging / Chatter -->
        <record id="mt_project_task_new" model="mail.message.subtype">
            <field name="name">Task Created</field>
            <field name="sequence">10</field>
            <field name="res_model">project.project</field>
            <field name="default" eval="False"/>
            <field name="parent_id" ref="mt_task_new"/>
            <field name="relation_field">project_id</field>
        </record>
        <record id="mt_project_task_blocked" model="mail.message.subtype">
            <field name="name">Task Blocked</field>
            <field name="sequence">11</field>
            <field name="res_model">project.project</field>
            <field name="default" eval="False"/>
            <field name="parent_id" ref="mt_task_blocked"/>
            <field name="relation_field">project_id</field>
        </record>
        <record id="mt_project_task_ready" model="mail.message.subtype">
            <field name="name">Task Ready</field>
            <field name="sequence">12</field>
            <field name="res_model">project.project</field>
            <field name="default" eval="False"/>
            <field name="parent_id" ref="mt_task_ready"/>
            <field name="relation_field">project_id</field>
        </record>
        <record id="mt_project_task_stage" model="mail.message.subtype">
            <field name="name">Task Stage Changed</field>
            <field name="sequence">13</field>
            <field name="res_model">project.project</field>
            <field name="default" eval="False"/>
            <field name="parent_id" ref="mt_task_stage"/>
            <field name="relation_field">project_id</field>
        </record>
        <record id="mt_project_task_rating" model="mail.message.subtype">
            <field name="name">Task Rating</field>
            <field name="sequence">14</field>
            <field name="res_model">project.project</field>
            <field name="default" eval="True"/>
            <field name="parent_id" ref="mt_task_rating"/>
            <field name="relation_field">project_id</field>
        </record>
</odoo>

```

## File: data\project_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Users -->
        <record id="base.user_demo" model="res.users">
            <field name="groups_id" eval="[(4, ref('group_project_manager'))]"/>
        </record>

        <!-- Categories -->
        <record id="project_tags_00" model="project.tags">
            <field name="name">Bug</field>
        </record>
        <record id="project_tags_01" model="project.tags">
            <field name="name">New Feature</field>
        </record>
        <record id="project_tags_02" model="project.tags">
            <field name="name">Experiment</field>
        </record>
        <record id="project_tags_03" model="project.tags">
            <field name="name">Usability</field>
        </record>

        <!-- Stages -->
        <record id="project_stage_0" model="project.task.type">
            <field name="sequence">1</field>
            <field name="name">New</field>
            <field name="legend_blocked">Blocked</field>
            <field name="mail_template_id" ref="project.mail_template_data_project_task"/>
        </record>
        <record id="project_stage_1" model="project.task.type">
            <field name="sequence">10</field>
            <field name="name">In Progress</field>
            <field name="legend_blocked">Need functional or technical help</field>
            <field name="legend_done">Buzz or set as done</field>
        </record>
        <record id="project_stage_2" model="project.task.type">
            <field name="sequence">20</field>
            <field name="name">Done</field>
            <field name="fold" eval="True"/>
            <field name="is_closed" eval="True"/>
        </record>
        <record id="project_stage_3" model="project.task.type">
            <field name="sequence">30</field>
            <field name="name">Cancelled</field>
            <field name="legend_done">Ready to reopen</field>
            <field name="fold" eval="True"/>
            <field name="is_closed" eval="True"/>
        </record>

        <record id="project_project_1" model="project.project">
            <field name="date_start" eval="time.strftime('%Y-%m-01 10:00:00')"/>
            <field name="name">Office Design</field>
            <field name="color">3</field>
            <field name="user_id" ref="base.user_demo"/>
            <field name="type_ids" eval="[(4, ref('project_stage_0')), (4, ref('project_stage_1')), (4, ref('project_stage_2')), (4, ref('project_stage_3'))]"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field name="privacy_visibility">portal</field>
            <field name="favorite_user_ids" eval="[(4, ref('base.user_admin'))]"/>
        </record>

        <record id="project_project_2" model="project.project">
            <field name="name">Research &amp; Development</field>
            <field name="privacy_visibility">followers</field>
            <field name="user_id" ref="base.user_demo"/>
            <field name="type_ids" eval="[(4, ref('project_stage_0')), (4, ref('project_stage_1')), (4, ref('project_stage_2')), (4, ref('project_stage_3'))]"/>
            <field name="favorite_user_ids" eval="[(4, ref('base.user_admin'))]"/>
        </record>

        <!-- Channel -->
        <record id="mail_channel_project_task" model="mail.channel">
            <field name="name">Projects &amp; Tasks</field>
            <field name="group_public_id" ref="group_project_user"/>
            <field name="group_ids" eval="[(4, ref('group_project_user'))]"/>
        </record>

        <!-- Channel follows all above projects -->
        <function model="project.project" name="message_subscribe"
            eval="[ref('project_project_1'), ref('project_project_2')
            ], None, [ref('mail_channel_project_task')]"/>

        <!-- Tasks -->
        <record id="project_task_1" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Meeting Room Furnitures</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="color">3</field>
        </record>
        <record id="project_task_2" model="project.task">
            <field name="planned_hours" eval="32.0"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Lunch Room: kitchen</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_task_3" model="project.task">
            <field name="planned_hours" eval="10.0"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Noise Reduction</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-24')"/>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">4</field>
        </record>
        <record id="project_task_4" model="project.task">
            <field name="planned_hours" eval="60.0"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Black Chairs for managers</field>
            <field name="description">Use the account_budget module</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-19')"/>
            <field name="color">5</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_01')])]"/>
        </record>
        <record id="project_task_5" model="project.task">
            <field name="planned_hours" eval="76.0"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Room 1: Decoration</field>
            <field name="kanban_state">done</field>
            <field name="priority">0</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-%d')"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_01')])]"/>
        </record>
        <record id="project_task_6" model="project.task">
            <field name="planned_hours" eval="24.0"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Room 2: Decoration</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_task_7" model="project.task">
            <field name="planned_hours" eval="15.0"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Energy Certificate</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_task_8" model="project.task">
            <field name="planned_hours" eval="22.0"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">New portal system</field>
            <field name="priority">0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project.project_tags_02')])]"/>
        </record>
        <record id="project_task_9" model="project.task">
            <field name="planned_hours" eval="18.0"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Document management</field>
            <field name="stage_id" ref="project_stage_0"/>
        </record>
        <record id="project_task_10" model="project.task">
            <field name="planned_hours" eval="38.0"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Social network integration</field>
            <field name="kanban_state">blocked</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_task_11" model="project.task">
            <field name="planned_hours" eval="16.0"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">User interface improvements</field>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project.project_tags_01'),
                    ref('project.project_tags_03')])]"/>
            <field name="stage_id" ref="project_stage_1"/>
        </record>

        <record id="project_task_12" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Planning and budget</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="color">6</field>
        </record>

        <record id="project_task_19" model="project.task">
            <field name="planned_hours">24.0</field>
            <field name="stage_id" ref="project_stage_3"/>
            <field name="user_id" eval="False"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Basic outline</field>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_02')])]"/>
        </record>

        <record id="project_task_20" model="project.task">
            <field name="planned_hours">42.0</field>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Create new components</field>
        </record>

        <record id="project_task_21" model="project.task">
            <field name="planned_hours">14.0</field>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Useablity review</field>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_03')])]"/>
        </record>

        <record id="project_task_22" model="project.task">
            <field name="planned_hours">12.0</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="user_id" eval="False"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Customer analysis + Architecture</field>
            <field name="color">7</field>
        </record>
        <record id="project_task_24" model="project.task">
            <field name="sequence">17</field>
            <field name="planned_hours">8.0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="user_id" eval="False"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Modifications asked by the customer</field>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_00')])]"/>
        </record>

        <record id="project_task_25" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">20.0</field>
            <field name="user_id" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Office planning</field>
            <field name="stage_id" ref="project_stage_0"/>
        </record>

        <record id="project_task_26" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">35.0</field>
            <field name="user_id" eval="False"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Unit Testing</field>
            <field name="stage_id" ref="project_stage_0"/>
        </record>

        <record id="message_task_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_task_22"/>
            <field name="body">Hello Demo,
There is a change in customer requirement.
Can you check the document from customer again.
Thanks,</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_root"/>
        </record>
        <record id="message_task_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_task_22"/>
            <field name="parent_id" ref="message_task_1"/>
            <field name="body">Ok, I have checked the mail,
I will update the document and let you know.</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
        </record>
        <record id="message_task_3" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_task_22"/>
            <field name="parent_id" ref="message_task_2"/>
            <field name="body">Fine!
Send it ASAP, its urgent.</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_root"/>
        </record>

        <function model="project.project" name="message_subscribe"
            eval="[ref('project.project_project_1')], [ref('base.partner_demo_portal')]"/>


        <!-- Rating Demo Data -->
        <record id="rating_task_1" model="rating.rating">
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="rated_partner_id" ref="base.partner_root"/>
            <field name="partner_id" ref="base.partner_demo"/>
            <field name="res_id" ref="project.project_task_3"/>
        </record>
        <record id="rating_task_2" model="rating.rating">
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="rated_partner_id" ref="base.partner_demo"/>
            <field name="partner_id" ref="base.partner_demo"/>
            <field name="res_id" ref="project.project_task_8"/>
        </record>
        <record id="rating_task_5" model="rating.rating">
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="rated_partner_id" ref="base.partner_root"/>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="res_id" ref="project.project_task_24"/>
        </record>

        <!-- add the email template as value for the project stage 2 -->
        <record id="project.project_stage_2" model="project.task.type">
            <field name="rating_template_id" ref="rating_project_request_email_template"/>
        </record>

        <function model="project.task" name="rating_apply" eval="([ref('project.project_task_3')], 5)"/>
        <function model="project.task" name="rating_apply" eval="([ref('project.project_task_24')], 5)"/>
        <function model="project.task" name="rating_apply" eval="([ref('project.project_task_8')], 1)"/>
    </data>
</odoo>

```

## File: data\project_mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Sample stage-related template -->
        <record id="mail_template_data_project_task" model="mail.template">
            <field name="name">Task: Reception Acknowledgment</field>
            <field name="model_id" ref="project.model_project_task"/>
            <field name="subject">Reception of ${object.name}</field>
            <field name="use_default_to" eval="True"/>
            <field name="body_html" type="html">
<div>
    Dear ${object.partner_id.name or 'customer'},<br/>
    Thank you for your enquiry.<br />
    If you have any questions, please let us know.
    <br/><br/>
    Thank you,
    % if user.signature
        <br />
        ${user.signature | safe}
    % endif
</div>
        </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Mail sent to request a rating for a task -->
        <record id="rating_project_request_email_template" model="mail.template">
            <field name="name">Task: Rating Request</field>
            <field name="model_id" ref="project.model_project_task"/>
            <field name="subject">${object.project_id.company_id.name}: Satisfaction Survey</field>
            <field name="email_from">${(object.rating_get_rated_partner_id().email_formatted if object.rating_get_rated_partner_id() else user.email_formatted) | safe}</field>
            <field name="partner_to" >${object.rating_get_partner_id().id}</field>
            <field name="body_html" type="html">
<div>
    % set access_token = object.rating_get_access_token()
    % set partner = object.rating_get_partner_id()
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="width:100%; margin:0px auto;">
    <tbody>
        <tr><td valign="top" style="font-size: 13px;">
            % if partner.name:
                Hello ${partner.name},<br/><br/>
            % else:
                Hello,<br/><br/>
            % endif
            Please take a moment to rate our services related to the task "<strong>${object.name}</strong>"
            % if object.rating_get_rated_partner_id().name:
                assigned to <strong>${object.rating_get_rated_partner_id().name}</strong>.<br/>
            % else:
                .<br/>
            % endif
        </td></tr>
        <tr><td style="text-align: center;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" summary="o_mail_notification" style="width:100%; margin: 32px 0px 32px 0px;">
                <tr><td style="font-size: 13px;">
                    <strong>Tell us how you feel about our service</strong><br/>
                    <span style="text-color: #888888">(click on one of these smileys)</span>
                </td></tr>
                <tr><td style="font-size: 13px;">
                    <table style="width:100%;text-align:center;">
                        <tr>
                            <td>
                                <a href="/rate/${access_token}/5">
                                    <img alt="Satisfied" src="/rating/static/src/img/rating_5.png" title="Satisfied"/>
                                </a>
                            </td>
                            <td>
                                <a href="/rate/${access_token}/3">
                                    <img alt="Not satisfied" src="/rating/static/src/img/rating_3.png" title="Not satisfied"/>
                                </a>
                            </td>
                            <td>
                                <a href="/rate/${access_token}/1">
                                    <img alt="Highly Dissatisfied" src="/rating/static/src/img/rating_1.png" title="Highly Dissatisfied"/>
                                </a>
                            </td>
                        </tr>
                    </table>
                </td></tr>
            </table>
        </td></tr>
        <tr><td valign="top" style="font-size: 13px;">
            We appreciate your feedback. It helps us to improve continuously.
            % if object.project_id.rating_status == 'stage':
                <br/><span style="margin: 0px 0px 0px 0px; font-size: 12px; opacity: 0.5; color: #454748;">This customer survey has been sent because your task has been moved to the stage <b>${object.stage_id.name}</b></span>
            % endif
            % if object.project_id.rating_status == 'periodic':
                <br/><span style="margin: 0px 0px 0px 0px; font-size: 12px; opacity: 0.5; color: #454748;">This customer survey is sent <b>${object.project_id.rating_status_period}</b> as long as the task is in the <b>${object.stage_id.name}</b> stage.</span>
            % endif
        </td></tr>
    </tbody>
    </table>
</div>
            </field>
            <field name="lang">${object.rating_get_partner_id().lang}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: doc\changelog.rst

```rst
.. _changelog:

Changelog
=========

`trunk (saas-2)`
----------------

- Stage/state update

  - ``project.task``: removed inheritance from ``base_stage`` class and removed
    ``state`` field. Added ``date_last_stage_update`` field holding last stage_id
    modification. Updated reports.
  - ``project.task.type``: removed ``state`` field.

- Removed ``project.task.reevaluate`` wizard.

```

## File: doc\index.rst

```rst
=====================
Project DevDoc
=====================

Project module documentation
===================================

Documentation topics
''''''''''''''''''''

.. toctree::
   :maxdepth: 1
   
   stage_status.rst

Changelog
'''''''''

.. toctree::
   :maxdepth: 1

   changelog.rst

```

## File: doc\stage_status.rst

```rst
.. _stage_status:

Stage and Status
================

.. versionchanged:: 8.0 saas-2 state/stage cleaning

Stage
+++++

This revision removed the concept of state on project.task objects. The ``state``
field has been totally removed and replaced by stages, using ``stage_id``. The
following models are impacted:

 - ``project.task`` now use only stages. However a convention still exists about
   'New' stage. A task is consdered as ``new`` when it has the following
   properties:

   - ``stage_id and stage_id.sequence = 1``

 - ``project.task.type`` do not have any ``state`` field anymore. 
 - ``project.task.report`` do not have any ``state`` field anymore. 

By default a newly created task is in a new stage. It means that it will
fetch the stage having ``sequence = 1``. Stage mangement is done using the
kanban view or the clikable statusbar. It is not done using buttons anymore.

Stage analysis
++++++++++++++

Stage analysis can be performed using the newly introduced ``date_last_stage_update``
datetime field. This field is updated everytime ``stage_id`` is updated.

``project.task.report`` model also uses the ``date_last_stage_update`` field.
This allows to group and analyse the time spend in the various stages.

Open / Assignment date
+++++++++++++++++++++++

The ``date_open`` field meaning has been updated. It is now set when the ``user_id``
(responsible) is set. It is therefore the assignment date.

Subtypes
++++++++

The following subtypes are triggered on ``project.task``:

 - ``mt_task_new``: new tasks. Condition: ``obj.stage_id and obj.stage_id.sequence == 1``
 - ``mt_task_stage``: stage changed. Condition: ``obj.stage_id and obj.stage_id.sequence != 1``
 - ``mt_task_assigned``: user assigned. condition: ``obj.user_id and obj.user_id.id``
 - ``mt_task_blocked``: kanban state blocked. Condition: ``obj.kanban_state == 'blocked'``


Those subtypes are also available on the ``project.project`` model and are used
for the auto subscription.

```

## File: models\analytic_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class AccountAnalyticAccount(models.Model):
    _inherit = 'account.analytic.account'
    _description = 'Analytic Account'

    project_ids = fields.One2many('project.project', 'analytic_account_id', string='Projects')
    project_count = fields.Integer("Project Count", compute='_compute_project_count')

    @api.depends('project_ids')
    def _compute_project_count(self):
        project_data = self.env['project.project'].read_group([('analytic_account_id', 'in', self.ids)], ['analytic_account_id'], ['analytic_account_id'])
        mapping = {m['analytic_account_id'][0]: m['analytic_account_id_count'] for m in project_data}
        for account in self:
            account.project_count = mapping.get(account.id, 0)

    @api.constrains('company_id')
    def _check_company_id(self):
        for record in self:
            if record.company_id and not all(record.company_id == c for c in record.project_ids.mapped('company_id')):
                raise UserError(_('You cannot change the company of an analytical account if it is related to a project.'))

    def unlink(self):
        projects = self.env['project.project'].search([('analytic_account_id', 'in', self.ids)])
        has_tasks = self.env['project.task'].search_count([('project_id', 'in', projects.ids)])
        if has_tasks:
            raise UserError(_('Please remove existing tasks in the project linked to the accounts you want to delete.'))
        return super(AccountAnalyticAccount, self).unlink()

    def action_view_projects(self):
        kanban_view_id = self.env.ref('project.view_project_kanban').id
        result = {
            "type": "ir.actions.act_window",
            "res_model": "project.project",
            "views": [[kanban_view_id, "kanban"], [False, "form"]],
            "domain": [['analytic_account_id', '=', self.id]],
            "context": {"create": False},
            "name": "Projects",
        }
        if len(self.project_ids) == 1:
            result['views'] = [(False, "form")]
            result['res_id'] = self.project_ids.id
        return result

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import AccessError


class Digest(models.Model):
    _inherit = 'digest.digest'

    kpi_project_task_opened = fields.Boolean('Open Tasks')
    kpi_project_task_opened_value = fields.Integer(compute='_compute_project_task_opened_value')

    def _compute_project_task_opened_value(self):
        if not self.env.user.has_group('project.group_project_user'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            record.kpi_project_task_opened_value = self.env['project.task'].search_count([
                ('stage_id.fold', '=', False),
                ('create_date', '>=', start),
                ('create_date', '<', end),
                ('company_id', '=', company.id)
            ])

    def _compute_kpis_actions(self, company, user):
        res = super(Digest, self)._compute_kpis_actions(company, user)
        res['kpi_project_task_opened'] = 'project.open_view_project_all&menu_id=%s' % self.env.ref('project.menu_main_pm').id
        return res

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
from collections import defaultdict
from datetime import timedelta
from random import randint

from odoo import api, fields, models, tools, SUPERUSER_ID, _
from odoo.exceptions import UserError, ValidationError
from odoo.osv.expression import OR
from odoo.tools.misc import get_lang

from .project_task_recurrence import DAYS, WEEKS

class ProjectTaskType(models.Model):
    _name = 'project.task.type'
    _description = 'Task Stage'
    _order = 'sequence, id'

    def _get_default_project_ids(self):
        default_project_id = self.env.context.get('default_project_id')
        return [default_project_id] if default_project_id else None

    active = fields.Boolean('Active', default=True)
    name = fields.Char(string='Stage Name', required=True, translate=True)
    description = fields.Text(translate=True)
    sequence = fields.Integer(default=1)
    project_ids = fields.Many2many('project.project', 'project_task_type_rel', 'type_id', 'project_id', string='Projects',
        default=_get_default_project_ids)
    legend_blocked = fields.Char(
        'Red Kanban Label', default=lambda s: _('Blocked'), translate=True, required=True,
        help='Override the default value displayed for the blocked state for kanban selection, when the task or issue is in that stage.')
    legend_done = fields.Char(
        'Green Kanban Label', default=lambda s: _('Ready'), translate=True, required=True,
        help='Override the default value displayed for the done state for kanban selection, when the task or issue is in that stage.')
    legend_normal = fields.Char(
        'Grey Kanban Label', default=lambda s: _('In Progress'), translate=True, required=True,
        help='Override the default value displayed for the normal state for kanban selection, when the task or issue is in that stage.')
    mail_template_id = fields.Many2one(
        'mail.template',
        string='Email Template',
        domain=[('model', '=', 'project.task')],
        help="If set an email will be sent to the customer when the task or issue reaches this step.")
    fold = fields.Boolean(string='Folded in Kanban',
        help='This stage is folded in the kanban view when there are no records in that stage to display.')
    rating_template_id = fields.Many2one(
        'mail.template',
        string='Rating Email Template',
        domain=[('model', '=', 'project.task')],
        help="If set and if the project's rating configuration is 'Rating when changing stage', then an email will be sent to the customer when the task reaches this step.")
    auto_validation_kanban_state = fields.Boolean('Automatic kanban status', default=False,
        help="Automatically modify the kanban state when the customer replies to the feedback for this stage.\n"
            " * A good feedback from the customer will update the kanban state to 'ready for the new stage' (green bullet).\n"
            " * A medium or a bad feedback will set the kanban state to 'blocked' (red bullet).\n")
    is_closed = fields.Boolean('Closing Stage', help="Tasks in this stage are considered as closed.")
    disabled_rating_warning = fields.Text(compute='_compute_disabled_rating_warning')

    def unlink_wizard(self, stage_view=False):
        self = self.with_context(active_test=False)
        # retrieves all the projects with a least 1 task in that stage
        # a task can be in a stage even if the project is not assigned to the stage
        readgroup = self.with_context(active_test=False).env['project.task'].read_group([('stage_id', 'in', self.ids)], ['project_id'], ['project_id'])
        project_ids = list(set([project['project_id'][0] for project in readgroup] + self.project_ids.ids))

        wizard = self.with_context(project_ids=project_ids).env['project.task.type.delete.wizard'].create({
            'project_ids': project_ids,
            'stage_ids': self.ids
        })

        context = dict(self.env.context)
        context['stage_view'] = stage_view
        return {
            'name': _('Delete Stage'),
            'view_mode': 'form',
            'res_model': 'project.task.type.delete.wizard',
            'views': [(self.env.ref('project.view_project_task_type_delete_wizard').id, 'form')],
            'type': 'ir.actions.act_window',
            'res_id': wizard.id,
            'target': 'new',
            'context': context,
        }

    def write(self, vals):
        if 'active' in vals and not vals['active']:
            self.env['project.task'].search([('stage_id', 'in', self.ids)]).write({'active': False})
        return super(ProjectTaskType, self).write(vals)

    @api.depends('project_ids', 'project_ids.rating_active')
    def _compute_disabled_rating_warning(self):
        for stage in self:
            disabled_projects = stage.project_ids.filtered(lambda p: not p.rating_active)
            if disabled_projects:
                stage.disabled_rating_warning = '\n'.join('- %s' % p.name for p in disabled_projects)
            else:
                stage.disabled_rating_warning = False


class Project(models.Model):
    _name = "project.project"
    _description = "Project"
    _inherit = ['portal.mixin', 'mail.alias.mixin', 'mail.thread', 'rating.parent.mixin']
    _order = "sequence, name, id"
    _rating_satisfaction_days = False  # takes all existing ratings
    _check_company_auto = True

    def _compute_attached_docs_count(self):
        Attachment = self.env['ir.attachment']
        for project in self:
            project.doc_count = Attachment.search_count([
                '|',
                '&',
                ('res_model', '=', 'project.project'), ('res_id', '=', project.id),
                '&',
                ('res_model', '=', 'project.task'), ('res_id', 'in', project.task_ids.ids)
            ])

    def _compute_task_count(self):
        task_data = self.env['project.task'].read_group([('project_id', 'in', self.ids), '|', '&', ('stage_id.is_closed', '=', False), ('stage_id.fold', '=', False), ('stage_id', '=', False)], ['project_id'], ['project_id'])
        result = dict((data['project_id'][0], data['project_id_count']) for data in task_data)
        for project in self:
            project.task_count = result.get(project.id, 0)

    def attachment_tree_view(self):
        action = self.env['ir.actions.act_window']._for_xml_id('base.action_attachment')
        action['domain'] = str([
            '|',
            '&',
            ('res_model', '=', 'project.project'),
            ('res_id', 'in', self.ids),
            '&',
            ('res_model', '=', 'project.task'),
            ('res_id', 'in', self.task_ids.ids)
        ])
        action['context'] = "{'default_res_model': '%s','default_res_id': %d}" % (self._name, self.id)
        return action

    def _compute_is_favorite(self):
        for project in self:
            project.is_favorite = self.env.user in project.favorite_user_ids

    def _inverse_is_favorite(self):
        favorite_projects = not_fav_projects = self.env['project.project'].sudo()
        for project in self:
            if self.env.user in project.favorite_user_ids:
                favorite_projects |= project
            else:
                not_fav_projects |= project

        # Project User has no write access for project.
        not_fav_projects.write({'favorite_user_ids': [(4, self.env.uid)]})
        favorite_projects.write({'favorite_user_ids': [(3, self.env.uid)]})

    def _get_default_favorite_user_ids(self):
        return [(6, 0, [self.env.uid])]

    name = fields.Char("Name", index=True, required=True, tracking=True)
    description = fields.Html()
    active = fields.Boolean(default=True,
        help="If the active field is set to False, it will allow you to hide the project without removing it.")
    sequence = fields.Integer(default=10, help="Gives the sequence order when displaying a list of Projects.")
    partner_id = fields.Many2one('res.partner', string='Customer', auto_join=True, tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    partner_email = fields.Char(
        compute='_compute_partner_email', inverse='_inverse_partner_email',
        string='Email', readonly=False, store=True, copy=False)
    partner_phone = fields.Char(
        compute='_compute_partner_phone', inverse='_inverse_partner_phone',
        string="Phone", readonly=False, store=True, copy=False)
    company_id = fields.Many2one('res.company', string='Company', required=True, default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', related="company_id.currency_id", string="Currency", readonly=True)
    analytic_account_id = fields.Many2one('account.analytic.account', string="Analytic Account", copy=False, ondelete='set null',
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", check_company=True,
        help="Analytic account to which this project is linked for financial management. "
             "Use an analytic account to record cost and revenue on your project.")

    favorite_user_ids = fields.Many2many(
        'res.users', 'project_favorite_user_rel', 'project_id', 'user_id',
        default=_get_default_favorite_user_ids,
        string='Members')
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite', string='Show Project on dashboard',
        help="Whether this project should be displayed on your dashboard.")
    label_tasks = fields.Char(string='Use Tasks as', default='Tasks', help="Label used for the tasks of the project.", translate=True)
    tasks = fields.One2many('project.task', 'project_id', string="Task Activities")
    resource_calendar_id = fields.Many2one(
        'resource.calendar', string='Working Time',
        related='company_id.resource_calendar_id')
    type_ids = fields.Many2many('project.task.type', 'project_task_type_rel', 'project_id', 'type_id', string='Tasks Stages')
    task_count = fields.Integer(compute='_compute_task_count', string="Task Count")
    task_ids = fields.One2many('project.task', 'project_id', string='Tasks',
                               domain=['|', ('stage_id.fold', '=', False), ('stage_id', '=', False)])
    color = fields.Integer(string='Color Index')
    user_id = fields.Many2one('res.users', string='Project Manager', default=lambda self: self.env.user, tracking=True)
    alias_enabled = fields.Boolean(string='Use email alias', compute='_compute_alias_enabled', readonly=False)
    alias_id = fields.Many2one('mail.alias', string='Alias', ondelete="restrict", required=True,
        help="Internal email associated with this project. Incoming emails are automatically synchronized "
             "with Tasks (or optionally Issues if the Issue Tracker module is installed).")
    privacy_visibility = fields.Selection([
            ('followers', 'Invited internal users'),
            ('employees', 'All internal users'),
            ('portal', 'Invited portal users and all internal users'),
        ],
        string='Visibility', required=True,
        default='portal',
        help="People to whom this project and its tasks will be visible.\n\n"
            "- Invited internal users: when following a project, internal users will get access to all of its tasks without distinction. "
            "Otherwise, they will only get access to the specific tasks they are following.\n "
            "A user with the project > administrator access right level can still access this project and its tasks, even if they are not explicitly part of the followers.\n\n"
            "- All internal users: all internal users can access the project and all of its tasks without distinction.\n\n"
            "- Invited portal users and all internal users: all internal users can access the project and all of its tasks without distinction.\n"
            "When following a project, portal users will get access to all of its tasks without distinction. Otherwise, they will only get access to the specific tasks they are following.")

    allowed_user_ids = fields.Many2many('res.users', compute='_compute_allowed_users', inverse='_inverse_allowed_user')
    allowed_internal_user_ids = fields.Many2many('res.users', 'project_allowed_internal_users_rel',
                                                 string="Allowed Internal Users", default=lambda self: self.env.user, domain=[('share', '=', False)])
    allowed_portal_user_ids = fields.Many2many('res.users', 'project_allowed_portal_users_rel', string="Allowed Portal Users", domain=[('share', '=', True)])
    doc_count = fields.Integer(compute='_compute_attached_docs_count', string="Number of documents attached")
    date_start = fields.Date(string='Start Date')
    date = fields.Date(string='Expiration Date', index=True, tracking=True)
    subtask_project_id = fields.Many2one('project.project', string='Sub-task Project', ondelete="restrict",
        help="Project in which sub-tasks of the current project will be created. It can be the current project itself.")
    allow_subtasks = fields.Boolean('Sub-tasks', default=lambda self: self.env.user.has_group('project.group_subtask_project'))
    allow_recurring_tasks = fields.Boolean('Recurring Tasks', default=lambda self: self.env.user.has_group('project.group_project_recurring_tasks'))

    # rating fields
    rating_request_deadline = fields.Datetime(compute='_compute_rating_request_deadline', store=True)
    rating_active = fields.Boolean('Customer Ratings', default=lambda self: self.env.user.has_group('project.group_project_rating'))
    rating_status = fields.Selection(
        [('stage', 'Rating when changing stage'),
         ('periodic', 'Periodical Rating')
        ], 'Customer Ratings Status', default="stage", required=True,
        help="How to get customer feedback?\n"
             "- Rating when changing stage: an email will be sent when a task is pulled in another stage.\n"
             "- Periodical Rating: email will be sent periodically.\n\n"
             "Don't forget to set up the mail templates on the stages for which you want to get the customer's feedbacks.")
    rating_status_period = fields.Selection([
        ('daily', 'Daily'),
        ('weekly', 'Weekly'),
        ('bimonthly', 'Twice a Month'),
        ('monthly', 'Once a Month'),
        ('quarterly', 'Quarterly'),
        ('yearly', 'Yearly')], 'Rating Frequency', required=True, default='monthly')

    _sql_constraints = [
        ('project_date_greater', 'check(date >= date_start)', 'Error! project start-date must be lower than project end-date.')
    ]

    @api.depends('partner_id.email')
    def _compute_partner_email(self):
        for project in self:
            if project.partner_id and project.partner_id.email != project.partner_email:
                project.partner_email = project.partner_id.email

    def _inverse_partner_email(self):
        for project in self:
            if project.partner_id and project.partner_email != project.partner_id.email:
                project.partner_id.email = project.partner_email

    @api.depends('partner_id.phone')
    def _compute_partner_phone(self):
        for project in self:
            if project.partner_id and project.partner_phone != project.partner_id.phone:
                project.partner_phone = project.partner_id.phone

    def _inverse_partner_phone(self):
        for project in self:
            if project.partner_id and project.partner_phone != project.partner_id.phone:
                project.partner_id.phone = project.partner_phone

    @api.onchange('alias_enabled')
    def _onchange_alias_name(self):
        if not self.alias_enabled:
            self.alias_name = False

    def _compute_alias_enabled(self):
        for project in self:
            project.alias_enabled = project.alias_domain and project.alias_id.alias_name

    @api.depends('allowed_internal_user_ids', 'allowed_portal_user_ids')
    def _compute_allowed_users(self):
        for project in self:
            users = project.allowed_internal_user_ids | project.allowed_portal_user_ids
            project.allowed_user_ids = users

    def _inverse_allowed_user(self):
        for project in self:
            allowed_users = project.allowed_user_ids
            project.allowed_portal_user_ids = allowed_users.filtered('share')
            project.allowed_internal_user_ids = allowed_users - project.allowed_portal_user_ids

    def _compute_access_url(self):
        super(Project, self)._compute_access_url()
        for project in self:
            project.access_url = '/my/project/%s' % project.id

    def _compute_access_warning(self):
        super(Project, self)._compute_access_warning()
        for project in self.filtered(lambda x: x.privacy_visibility != 'portal'):
            project.access_warning = _(
                "The project cannot be shared with the recipient(s) because the privacy of the project is too restricted. Set the privacy to 'Visible by following customers' in order to make it accessible by the recipient(s).")

    @api.depends('rating_status', 'rating_status_period')
    def _compute_rating_request_deadline(self):
        periods = {'daily': 1, 'weekly': 7, 'bimonthly': 15, 'monthly': 30, 'quarterly': 90, 'yearly': 365}
        for project in self:
            project.rating_request_deadline = fields.datetime.now() + timedelta(days=periods.get(project.rating_status_period, 0))

    @api.model
    def _map_tasks_default_valeus(self, task, project):
        """ get the default value for the copied task on project duplication """
        return {
            'stage_id': task.stage_id.id,
            'name': task.name,
            'company_id': project.company_id.id,
        }

    def map_tasks(self, new_project_id):
        """ copy and map tasks from old to new project """
        project = self.browse(new_project_id)
        tasks = self.env['project.task']
        # We want to copy archived task, but do not propagate an active_test context key
        task_ids = self.env['project.task'].with_context(active_test=False).search([('project_id', '=', self.id)], order='parent_id').ids
        old_to_new_tasks = {}
        for task in self.env['project.task'].browse(task_ids):
            # preserve task name and stage, normally altered during copy
            defaults = self._map_tasks_default_valeus(task, project)
            if task.parent_id:
                # set the parent to the duplicated task
                defaults['parent_id'] = old_to_new_tasks.get(task.parent_id.id, False)
            new_task = task.copy(defaults)
            # If child are created before parent (ex sub_sub_tasks)
            new_child_ids = [old_to_new_tasks[child.id] for child in task.child_ids if child.id in old_to_new_tasks]
            tasks.browse(new_child_ids).write({'parent_id': new_task.id})
            old_to_new_tasks[task.id] = new_task.id
            tasks += new_task

        return project.write({'tasks': [(6, 0, tasks.ids)]})

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default is None:
            default = {}
        if not default.get('name'):
            default['name'] = _("%s (copy)") % (self.name)
        project = super(Project, self).copy(default)
        if self.subtask_project_id == self:
            project.subtask_project_id = project
        for follower in self.message_follower_ids:
            project.message_subscribe(partner_ids=follower.partner_id.ids, subtype_ids=follower.subtype_ids.ids)
        if 'tasks' not in default:
            self.map_tasks(project.id)
        return project

    @api.model
    def create(self, vals):
        # Prevent double project creation
        self = self.with_context(mail_create_nosubscribe=True)
        project = super(Project, self).create(vals)
        if not vals.get('subtask_project_id'):
            project.subtask_project_id = project.id
        if project.privacy_visibility == 'portal' and project.partner_id.user_ids:
            project.allowed_user_ids |= project.partner_id.user_ids
        return project

    def write(self, vals):
        allowed_users_changed = 'allowed_portal_user_ids' in vals or 'allowed_internal_user_ids' in vals
        if allowed_users_changed:
            allowed_users = {project: project.allowed_user_ids for project in self}
        # directly compute is_favorite to dodge allow write access right
        if 'is_favorite' in vals:
            vals.pop('is_favorite')
            self._fields['is_favorite'].determine_inverse(self)
        res = super(Project, self).write(vals) if vals else True

        if allowed_users_changed:
            for project in self:
                permission_removed = allowed_users.get(project) - project.allowed_user_ids
                allowed_portal_users_removed = permission_removed.filtered('share')
                project.message_unsubscribe(allowed_portal_users_removed.partner_id.commercial_partner_id.ids)
                for task in project.task_ids:
                    task.allowed_user_ids -= permission_removed

        if 'allow_recurring_tasks' in vals and not vals.get('allow_recurring_tasks'):
            self.env['project.task'].search([('project_id', 'in', self.ids), ('recurring_task', '=', True)]).write({'recurring_task': False})

        if 'active' in vals:
            # archiving/unarchiving a project does it on its tasks, too
            self.with_context(active_test=False).mapped('tasks').write({'active': vals['active']})
        if vals.get('partner_id') or vals.get('privacy_visibility'):
            for project in self.filtered(lambda project: project.privacy_visibility == 'portal'):
                project.allowed_user_ids |= project.partner_id.user_ids

        return res

    def action_unlink(self):
        wizard = self.env['project.delete.wizard'].create({
            'project_ids': self.ids
        })

        return {
            'name': _('Confirmation'),
            'view_mode': 'form',
            'res_model': 'project.delete.wizard',
            'views': [(self.env.ref('project.project_delete_wizard_form').id, 'form')],
            'type': 'ir.actions.act_window',
            'res_id': wizard.id,
            'target': 'new',
            'context': self.env.context,
        }

    def unlink(self):
        # Check project is empty
        for project in self.with_context(active_test=False):
            if project.tasks:
                raise UserError(_('You cannot delete a project containing tasks. You can either archive it or first delete all of its tasks.'))
        # Delete the empty related analytic account
        analytic_accounts_to_delete = self.env['account.analytic.account']
        for project in self:
            if project.analytic_account_id and not project.analytic_account_id.line_ids:
                analytic_accounts_to_delete |= project.analytic_account_id
        result = super(Project, self).unlink()
        analytic_accounts_to_delete.unlink()
        return result

    def message_subscribe(self, partner_ids=None, channel_ids=None, subtype_ids=None):
        """
        Subscribe to all existing active tasks when subscribing to a project
        And add the portal user subscribed to allowed portal users
        """
        res = super(Project, self).message_subscribe(partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=subtype_ids)
        project_subtypes = self.env['mail.message.subtype'].browse(subtype_ids) if subtype_ids else None
        task_subtypes = (project_subtypes.mapped('parent_id') | project_subtypes.filtered(lambda sub: sub.internal or sub.default)).ids if project_subtypes else None
        if not subtype_ids or task_subtypes:
            self.mapped('tasks').message_subscribe(
                partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=task_subtypes)
        if partner_ids:
            all_users = self.env['res.partner'].browse(partner_ids).user_ids
            portal_users = all_users.filtered('share')
            internal_users = all_users - portal_users
            self.allowed_portal_user_ids |= portal_users
            self.allowed_internal_user_ids |= internal_users
        return res

    def message_unsubscribe(self, partner_ids=None, channel_ids=None):
        """ Unsubscribe from all tasks when unsubscribing from a project """
        self.mapped('tasks').message_unsubscribe(partner_ids=partner_ids, channel_ids=channel_ids)
        return super(Project, self).message_unsubscribe(partner_ids=partner_ids, channel_ids=channel_ids)

    def _alias_get_creation_values(self):
        values = super(Project, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('project.task').id
        if self.id:
            values['alias_defaults'] = defaults = ast.literal_eval(self.alias_defaults or "{}")
            defaults['project_id'] = self.id
        return values

    # ---------------------------------------------------
    #  Actions
    # ---------------------------------------------------

    def toggle_favorite(self):
        favorite_projects = not_fav_projects = self.env['project.project'].sudo()
        for project in self:
            if self.env.user in project.favorite_user_ids:
                favorite_projects |= project
            else:
                not_fav_projects |= project

        # Project User has no write access for project.
        not_fav_projects.write({'favorite_user_ids': [(4, self.env.uid)]})
        favorite_projects.write({'favorite_user_ids': [(3, self.env.uid)]})

    def action_view_tasks(self):
        action = self.with_context(active_id=self.id, active_ids=self.ids) \
            .env.ref('project.act_project_project_2_project_task_all') \
            .sudo().read()[0]
        action['display_name'] = self.name
        return action

    def action_view_account_analytic_line(self):
        """ return the action to see all the analytic lines of the project's analytic account """
        action = self.env["ir.actions.actions"]._for_xml_id("analytic.account_analytic_line_action")
        action['context'] = {'default_account_id': self.analytic_account_id.id}
        action['domain'] = [('account_id', '=', self.analytic_account_id.id)]
        return action

    def action_view_all_rating(self):
        """ return the action to see all the rating of the project and activate default filters"""
        action = self.env['ir.actions.act_window']._for_xml_id('project.rating_rating_action_view_project_rating')
        action['name'] = _('Ratings of %s') % (self.name,)
        action_context = ast.literal_eval(action['context']) if action['context'] else {}
        action_context.update(self._context)
        action_context['search_default_parent_res_name'] = self.name
        action_context.pop('group_by', None)
        return dict(action, context=action_context)

    # ---------------------------------------------------
    #  Business Methods
    # ---------------------------------------------------

    @api.model
    def _create_analytic_account_from_values(self, values):
        analytic_account = self.env['account.analytic.account'].create({
            'name': values.get('name', _('Unknown Analytic Account')),
            'company_id': values.get('company_id') or self.env.company.id,
            'partner_id': values.get('partner_id'),
            'active': True,
        })
        return analytic_account

    def _create_analytic_account(self):
        for project in self:
            analytic_account = self.env['account.analytic.account'].create({
                'name': project.name,
                'company_id': project.company_id.id,
                'partner_id': project.partner_id.id,
                'active': True,
            })
            project.write({'analytic_account_id': analytic_account.id})

    # ---------------------------------------------------
    # Rating business
    # ---------------------------------------------------

    # This method should be called once a day by the scheduler
    @api.model
    def _send_rating_all(self):
        projects = self.search([
            ('rating_active', '=', True),
            ('rating_status', '=', 'periodic'),
            ('rating_request_deadline', '<=', fields.Datetime.now())
        ])
        for project in projects:
            project.task_ids._send_task_rating_mail()
            project._compute_rating_request_deadline()
            self.env.cr.commit()


class Task(models.Model):
    _name = "project.task"
    _description = "Task"
    _date_name = "date_assign"
    _inherit = ['portal.mixin', 'mail.thread.cc', 'mail.activity.mixin', 'rating.mixin']
    _mail_post_access = 'read'
    _order = "priority desc, sequence, id desc"
    _check_company_auto = True

    def _get_default_stage_id(self):
        """ Gives default stage_id """
        project_id = self.env.context.get('default_project_id')
        if not project_id:
            return False
        return self.stage_find(project_id, [('fold', '=', False), ('is_closed', '=', False)])

    @api.model
    def _default_company_id(self):
        if self._context.get('default_project_id'):
            return self.env['project.project'].browse(self._context['default_project_id']).company_id
        return self.env.company

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        search_domain = [('id', 'in', stages.ids)]
        if 'default_project_id' in self.env.context:
            search_domain = ['|', ('project_ids', '=', self.env.context['default_project_id'])] + search_domain

        stage_ids = stages._search(search_domain, order=order, access_rights_uid=SUPERUSER_ID)
        return stages.browse(stage_ids)

    active = fields.Boolean(default=True)
    name = fields.Char(string='Title', tracking=True, required=True, index=True)
    description = fields.Html(string='Description')
    priority = fields.Selection([
        ('0', 'Normal'),
        ('1', 'Important'),
    ], default='0', index=True, string="Priority")
    sequence = fields.Integer(string='Sequence', index=True, default=10,
        help="Gives the sequence order when displaying a list of tasks.")
    stage_id = fields.Many2one('project.task.type', string='Stage', compute='_compute_stage_id',
        store=True, readonly=False, ondelete='restrict', tracking=True, index=True,
        default=_get_default_stage_id, group_expand='_read_group_stage_ids',
        domain="[('project_ids', '=', project_id)]", copy=False)
    tag_ids = fields.Many2many('project.tags', string='Tags')
    kanban_state = fields.Selection([
        ('normal', 'In Progress'),
        ('done', 'Ready'),
        ('blocked', 'Blocked')], string='Kanban State',
        copy=False, default='normal', required=True)
    kanban_state_label = fields.Char(compute='_compute_kanban_state_label', string='Kanban State Label', tracking=True)
    create_date = fields.Datetime("Created On", readonly=True, index=True)
    write_date = fields.Datetime("Last Updated On", readonly=True, index=True)
    date_end = fields.Datetime(string='Ending Date', index=True, copy=False)
    date_assign = fields.Datetime(string='Assigning Date', index=True, copy=False, readonly=True)
    date_deadline = fields.Date(string='Deadline', index=True, copy=False, tracking=True)
    date_last_stage_update = fields.Datetime(string='Last Stage Update',
        index=True,
        copy=False,
        readonly=True)
    project_id = fields.Many2one('project.project', string='Project',
        compute='_compute_project_id', store=True, readonly=False,
        index=True, tracking=True, check_company=True, change_default=True)
    planned_hours = fields.Float("Initially Planned Hours", help='Time planned to achieve this task (including its sub-tasks).', tracking=True)
    subtask_planned_hours = fields.Float("Sub-tasks Planned Hours", compute='_compute_subtask_planned_hours', help="Sum of the time planned of all the sub-tasks linked to this task. Usually less or equal to the initially time planned of this task.")
    user_id = fields.Many2one('res.users',
        string='Assigned to',
        default=lambda self: self.env.uid,
        index=True, tracking=True)
    partner_id = fields.Many2one('res.partner',
        string='Customer',
        compute='_compute_partner_id', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    partner_is_company = fields.Boolean(related='partner_id.is_company', readonly=True)
    commercial_partner_id = fields.Many2one(related='partner_id.commercial_partner_id')
    partner_email = fields.Char(
        compute='_compute_partner_email', inverse='_inverse_partner_email',
        string='Email', readonly=False, store=True, copy=False)
    partner_phone = fields.Char(
        compute='_compute_partner_phone', inverse='_inverse_partner_phone',
        string="Phone", readonly=False, store=True, copy=False)
    ribbon_message = fields.Char('Ribbon message', compute='_compute_ribbon_message')
    partner_city = fields.Char(related='partner_id.city', readonly=False)
    manager_id = fields.Many2one('res.users', string='Project Manager', related='project_id.user_id', readonly=True)
    company_id = fields.Many2one(
        'res.company', string='Company', compute='_compute_company_id', store=True, readonly=False,
        required=True, copy=True, default=_default_company_id)
    color = fields.Integer(string='Color Index')
    user_email = fields.Char(related='user_id.email', string='User Email', readonly=True, related_sudo=False)
    attachment_ids = fields.One2many('ir.attachment', compute='_compute_attachment_ids', string="Main Attachments",
        help="Attachment that don't come from message.")
    # In the domain of displayed_image_id, we couln't use attachment_ids because a one2many is represented as a list of commands so we used res_model & res_id
    displayed_image_id = fields.Many2one('ir.attachment', domain="[('res_model', '=', 'project.task'), ('res_id', '=', id), ('mimetype', 'ilike', 'image')]", string='Cover Image')
    legend_blocked = fields.Char(related='stage_id.legend_blocked', string='Kanban Blocked Explanation', readonly=True, related_sudo=False)
    legend_done = fields.Char(related='stage_id.legend_done', string='Kanban Valid Explanation', readonly=True, related_sudo=False)
    legend_normal = fields.Char(related='stage_id.legend_normal', string='Kanban Ongoing Explanation', readonly=True, related_sudo=False)
    is_closed = fields.Boolean(related="stage_id.is_closed", string="Closing Stage", readonly=True, related_sudo=False)
    parent_id = fields.Many2one('project.task', string='Parent Task', index=True)
    child_ids = fields.One2many('project.task', 'parent_id', string="Sub-tasks", context={'active_test': False})
    subtask_project_id = fields.Many2one('project.project', related="project_id.subtask_project_id", string='Sub-task Project', readonly=True)
    allow_subtasks = fields.Boolean(string="Allow Sub-tasks", related="project_id.allow_subtasks", readonly=True)
    subtask_count = fields.Integer("Sub-task count", compute='_compute_subtask_count')
    email_from = fields.Char(string='Email From', help="These people will receive email.", index=True,
        compute='_compute_email_from', store="True", readonly=False, copy=False)
    allowed_user_ids = fields.Many2many('res.users', string="Visible to", groups='project.group_project_manager', compute='_compute_allowed_user_ids', store=True, readonly=False, copy=False)
    project_privacy_visibility = fields.Selection(related='project_id.privacy_visibility', string="Project Visibility")
    # Computed field about working time elapsed between record creation and assignation/closing.
    working_hours_open = fields.Float(compute='_compute_elapsed', string='Working hours to assign', store=True, group_operator="avg")
    working_hours_close = fields.Float(compute='_compute_elapsed', string='Working hours to close', store=True, group_operator="avg")
    working_days_open = fields.Float(compute='_compute_elapsed', string='Working days to assign', store=True, group_operator="avg")
    working_days_close = fields.Float(compute='_compute_elapsed', string='Working days to close', store=True, group_operator="avg")
    # customer portal: include comment and incoming emails in communication history
    website_message_ids = fields.One2many(domain=lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment'])])

    # recurrence fields
    allow_recurring_tasks = fields.Boolean(related='project_id.allow_recurring_tasks')
    recurring_task = fields.Boolean(string="Recurrent")
    recurring_count = fields.Integer(string="Tasks in Recurrence", compute='_compute_recurring_count')
    recurrence_id = fields.Many2one('project.task.recurrence', copy=False)
    recurrence_update = fields.Selection([
        ('this', 'This task'),
        ('subsequent', 'This and following tasks'),
        ('all', 'All tasks'),
    ], default='this', store=False)
    recurrence_message = fields.Char(string='Next Recurrencies', compute='_compute_recurrence_message')

    repeat_interval = fields.Integer(string='Repeat Every', default=1, compute='_compute_repeat', readonly=False)
    repeat_unit = fields.Selection([
        ('day', 'Days'),
        ('week', 'Weeks'),
        ('month', 'Months'),
        ('year', 'Years'),
    ], default='week', compute='_compute_repeat', readonly=False)
    repeat_type = fields.Selection([
        ('forever', 'Forever'),
        ('until', 'End Date'),
        ('after', 'Number of Repetitions'),
    ], default="forever", string="Until", compute='_compute_repeat', readonly=False)
    repeat_until = fields.Date(string="End Date", compute='_compute_repeat', readonly=False)
    repeat_number = fields.Integer(string="Repetitions", default=1, compute='_compute_repeat', readonly=False)

    repeat_on_month = fields.Selection([
        ('date', 'Date of the Month'),
        ('day', 'Day of the Month'),
    ], default='date', compute='_compute_repeat', readonly=False)

    repeat_on_year = fields.Selection([
        ('date', 'Date of the Year'),
        ('day', 'Day of the Year'),
    ], default='date', compute='_compute_repeat', readonly=False)

    mon = fields.Boolean(string="Mon", compute='_compute_repeat', readonly=False)
    tue = fields.Boolean(string="Tue", compute='_compute_repeat', readonly=False)
    wed = fields.Boolean(string="Wed", compute='_compute_repeat', readonly=False)
    thu = fields.Boolean(string="Thu", compute='_compute_repeat', readonly=False)
    fri = fields.Boolean(string="Fri", compute='_compute_repeat', readonly=False)
    sat = fields.Boolean(string="Sat", compute='_compute_repeat', readonly=False)
    sun = fields.Boolean(string="Sun", compute='_compute_repeat', readonly=False)

    repeat_day = fields.Selection([
        (str(i), str(i)) for i in range(1, 32)
    ], compute='_compute_repeat', readonly=False)
    repeat_week = fields.Selection([
        ('first', 'First'),
        ('second', 'Second'),
        ('third', 'Third'),
        ('last', 'Last'),
    ], default='first', compute='_compute_repeat', readonly=False)
    repeat_weekday = fields.Selection([
        ('mon', 'Monday'),
        ('tue', 'Tuesday'),
        ('wed', 'Wednesday'),
        ('thu', 'Thursday'),
        ('fri', 'Friday'),
        ('sat', 'Saturday'),
        ('sun', 'Sunday'),
    ], string='Day Of The Week', compute='_compute_repeat', readonly=False)
    repeat_month = fields.Selection([
        ('january', 'January'),
        ('february', 'February'),
        ('march', 'March'),
        ('april', 'April'),
        ('may', 'May'),
        ('june', 'June'),
        ('july', 'July'),
        ('august', 'August'),
        ('september', 'September'),
        ('october', 'October'),
        ('november', 'November'),
        ('december', 'December'),
    ], compute='_compute_repeat', readonly=False)

    repeat_show_dow = fields.Boolean(compute='_compute_repeat_visibility')
    repeat_show_day = fields.Boolean(compute='_compute_repeat_visibility')
    repeat_show_week = fields.Boolean(compute='_compute_repeat_visibility')
    repeat_show_month = fields.Boolean(compute='_compute_repeat_visibility')

    @api.model
    def _get_recurrence_fields(self):
        return ['repeat_interval', 'repeat_unit', 'repeat_type', 'repeat_until', 'repeat_number',
                'repeat_on_month', 'repeat_on_year', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat',
                'sun', 'repeat_day', 'repeat_week', 'repeat_month', 'repeat_weekday']

    @api.depends('recurring_task', 'repeat_unit', 'repeat_on_month', 'repeat_on_year')
    def _compute_repeat_visibility(self):
        for task in self:
            task.repeat_show_day = task.recurring_task and (task.repeat_unit == 'month' and task.repeat_on_month == 'date') or (task.repeat_unit == 'year' and task.repeat_on_year == 'date')
            task.repeat_show_week = task.recurring_task and (task.repeat_unit == 'month' and task.repeat_on_month == 'day') or (task.repeat_unit == 'year' and task.repeat_on_year == 'day')
            task.repeat_show_dow = task.recurring_task and task.repeat_unit == 'week'
            task.repeat_show_month = task.recurring_task and task.repeat_unit == 'year'

    @api.depends('recurring_task')
    def _compute_repeat(self):
        rec_fields = self._get_recurrence_fields()
        defaults = self.default_get(rec_fields)
        for task in self:
            for f in rec_fields:
                if task.recurrence_id:
                    task[f] = task.recurrence_id[f]
                else:
                    if task.recurring_task:
                        task[f] = defaults.get(f)
                    else:
                        task[f] = False

    def _get_weekdays(self, n=1):
        self.ensure_one()
        if self.repeat_unit == 'week':
            return [fn(n) for day, fn in DAYS.items() if self[day]]
        return [DAYS.get(self.repeat_weekday)(n)]

    def _get_recurrence_start_date(self):
        return fields.Date.today()

    @api.depends(
        'recurring_task', 'repeat_interval', 'repeat_unit', 'repeat_type', 'repeat_until',
        'repeat_number', 'repeat_on_month', 'repeat_on_year', 'mon', 'tue', 'wed', 'thu', 'fri',
        'sat', 'sun', 'repeat_day', 'repeat_week', 'repeat_month', 'repeat_weekday')
    def _compute_recurrence_message(self):
        self.recurrence_message = False
        for task in self.filtered(lambda t: t.recurring_task and t._is_recurrence_valid()):
            date = task._get_recurrence_start_date()
            number_occurrences = min(5, task.repeat_number if task.repeat_type == 'after' else 5)
            delta = task.repeat_interval if task.repeat_unit == 'day' else 1
            recurring_dates = self.env['project.task.recurrence']._get_next_recurring_dates(
                date + timedelta(days=delta),
                task.repeat_interval,
                task.repeat_unit,
                task.repeat_type,
                task.repeat_until,
                task.repeat_on_month,
                task.repeat_on_year,
                task._get_weekdays(WEEKS.get(task.repeat_week)),
                task.repeat_day,
                task.repeat_week,
                task.repeat_month,
                count=number_occurrences)
            date_format = self.env['res.lang']._lang_get(self.env.user.lang).date_format or get_lang(self.env).date_format
            task.recurrence_message = '<ul>'
            for date in recurring_dates[:5]:
                task.recurrence_message += '<li>%s</li>' % date.strftime(date_format)
            if task.repeat_type == 'after' and task.repeat_number > 5 or task.repeat_type == 'forever' or len(recurring_dates) > 5:
                task.recurrence_message += '<li>...</li>'
            task.recurrence_message += '</ul>'
            if task.repeat_type == 'until':
                task.recurrence_message += _('<p><em>Number of tasks: %(tasks_count)s</em></p>') % {'tasks_count': len(recurring_dates)}

    def _is_recurrence_valid(self):
        self.ensure_one()
        return self.repeat_interval > 0 and\
                (not self.repeat_show_dow or self._get_weekdays()) and\
                (self.repeat_type != 'after' or self.repeat_number) and\
                (self.repeat_type != 'until' or self.repeat_until and self.repeat_until > fields.Date.today())

    @api.depends('recurrence_id')
    def _compute_recurring_count(self):
        self.recurring_count = 0
        recurring_tasks = self.filtered(lambda l: l.recurrence_id)
        count = self.env['project.task'].read_group([('recurrence_id', 'in', recurring_tasks.recurrence_id.ids)], ['id'], 'recurrence_id')
        tasks_count = {c.get('recurrence_id')[0]: c.get('recurrence_id_count') for c in count}
        for task in recurring_tasks:
            task.recurring_count = tasks_count.get(task.recurrence_id.id, 0)

    @api.depends('partner_id.email')
    def _compute_partner_email(self):
        for task in self:
            if task.partner_id and task.partner_id.email != task.partner_email:
                task.partner_email = task.partner_id.email

    def _inverse_partner_email(self):
        for task in self:
            if task.partner_id and task.partner_email != task.partner_id.email:
                task.partner_id.email = task.partner_email

    @api.depends('partner_id.phone')
    def _compute_partner_phone(self):
        for task in self:
            if task.partner_id and task.partner_phone != task.partner_id.phone:
                task.partner_phone = task.partner_id.phone

    def _inverse_partner_phone(self):
        for task in self:
            if task.partner_id and task.partner_phone != task.partner_id.phone:
                task.partner_id.phone = task.partner_phone

    @api.depends('partner_email', 'partner_phone', 'partner_id')
    def _compute_ribbon_message(self):
        for task in self:
            will_write_email = task.partner_id and task.partner_email != task.partner_id.email
            will_write_phone = task.partner_id and task.partner_phone != task.partner_id.phone

            if will_write_email and will_write_phone:
                task.ribbon_message = _('By saving this change, the customer email and phone number will also be updated.')
            elif will_write_email:
                task.ribbon_message = _('By saving this change, the customer email will also be updated.')
            elif will_write_phone:
                task.ribbon_message = _('By saving this change, the customer phone number will also be updated.')
            else:
                task.ribbon_message = False

    @api.constrains('parent_id')
    def _check_parent_id(self):
        if not self._check_recursion():
            raise ValidationError(_('Error! You cannot create recursive hierarchy of tasks.'))

    @api.constrains('allowed_user_ids')
    def _check_no_portal_allowed(self):
        for task in self.filtered(lambda t: t.project_id.privacy_visibility != 'portal'):
            portal_users = task.allowed_user_ids.filtered('share')
            if portal_users:
                user_names = ', '.join(portal_users[:10].mapped('name'))
                raise ValidationError(_("The project visibility setting doesn't allow portal users to see the project's tasks. (%s)", user_names))

    def _compute_attachment_ids(self):
        for task in self:
            attachment_ids = self.env['ir.attachment'].search([('res_id', '=', task.id), ('res_model', '=', 'project.task')]).ids
            message_attachment_ids = task.mapped('message_ids.attachment_ids').ids  # from mail_thread
            task.attachment_ids = [(6, 0, list(set(attachment_ids) - set(message_attachment_ids)))]

    @api.depends('project_id.allowed_user_ids', 'project_id.privacy_visibility')
    def _compute_allowed_user_ids(self):
        for task in self.with_context(prefetch_fields=False):
            portal_users = task.allowed_user_ids.filtered('share')
            internal_users = task.allowed_user_ids - portal_users
            if task.project_id.privacy_visibility == 'followers':
                task.allowed_user_ids |= task.project_id.allowed_internal_user_ids
                task.allowed_user_ids -= portal_users
            elif task.project_id.privacy_visibility == 'portal':
                task.allowed_user_ids |= task.project_id.allowed_portal_user_ids
            if task.project_id.privacy_visibility != 'portal':
                task.allowed_user_ids -= portal_users
            elif task.project_id.privacy_visibility != 'followers':
                task.allowed_user_ids -= internal_users

    @api.depends('create_date', 'date_end', 'date_assign')
    def _compute_elapsed(self):
        task_linked_to_calendar = self.filtered(
            lambda task: task.project_id.resource_calendar_id and task.create_date
        )
        for task in task_linked_to_calendar:
            dt_create_date = fields.Datetime.from_string(task.create_date)

            if task.date_assign:
                dt_date_assign = fields.Datetime.from_string(task.date_assign)
                duration_data = task.project_id.resource_calendar_id.get_work_duration_data(dt_create_date, dt_date_assign, compute_leaves=True)
                task.working_hours_open = duration_data['hours']
                task.working_days_open = duration_data['days']
            else:
                task.working_hours_open = 0.0
                task.working_days_open = 0.0

            if task.date_end:
                dt_date_end = fields.Datetime.from_string(task.date_end)
                duration_data = task.project_id.resource_calendar_id.get_work_duration_data(dt_create_date, dt_date_end, compute_leaves=True)
                task.working_hours_close = duration_data['hours']
                task.working_days_close = duration_data['days']
            else:
                task.working_hours_close = 0.0
                task.working_days_close = 0.0

        (self - task_linked_to_calendar).update(dict.fromkeys(
            ['working_hours_open', 'working_hours_close', 'working_days_open', 'working_days_close'], 0.0))

    @api.depends('stage_id', 'kanban_state')
    def _compute_kanban_state_label(self):
        for task in self:
            if task.kanban_state == 'normal':
                task.kanban_state_label = task.legend_normal
            elif task.kanban_state == 'blocked':
                task.kanban_state_label = task.legend_blocked
            else:
                task.kanban_state_label = task.legend_done

    def _compute_access_url(self):
        super(Task, self)._compute_access_url()
        for task in self:
            task.access_url = '/my/task/%s' % task.id

    def _compute_access_warning(self):
        super(Task, self)._compute_access_warning()
        for task in self.filtered(lambda x: x.project_id.privacy_visibility != 'portal'):
            task.access_warning = _(
                "The task cannot be shared with the recipient(s) because the privacy of the project is too restricted. Set the privacy of the project to 'Visible by following customers' in order to make it accessible by the recipient(s).")

    @api.depends('child_ids.planned_hours')
    def _compute_subtask_planned_hours(self):
        for task in self:
            task.subtask_planned_hours = sum(child_task.planned_hours + child_task.subtask_planned_hours for child_task in task.child_ids)

    @api.depends('child_ids')
    def _compute_subtask_count(self):
        for task in self:
            task.subtask_count = len(task._get_all_subtasks())

    @api.onchange('company_id')
    def _onchange_task_company(self):
        if self.project_id.company_id != self.company_id:
            self.project_id = False

    @api.depends('project_id.company_id')
    def _compute_company_id(self):
        for task in self.filtered(lambda task: task.project_id):
            task.company_id = task.project_id.company_id

    @api.depends('project_id')
    def _compute_stage_id(self):
        for task in self:
            if task.project_id:
                if task.project_id not in task.stage_id.project_ids:
                    task.stage_id = task.stage_find(task.project_id.id, [
                        ('fold', '=', False), ('is_closed', '=', False)])
            else:
                task.stage_id = False

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default is None:
            default = {}
        if not default.get('name'):
            default['name'] = _("%s (copy)", self.name)
        if self.recurrence_id:
            default['recurrence_id'] = self.recurrence_id.copy().id
        return super(Task, self).copy(default)

    @api.constrains('parent_id')
    def _check_parent_id(self):
        for task in self:
            if not task._check_recursion():
                raise ValidationError(_('Error! You cannot create recursive hierarchy of task(s).'))

    @api.model
    def get_empty_list_help(self, help):
        tname = _("task")
        project_id = self.env.context.get('default_project_id', False)
        if project_id:
            name = self.env['project.project'].browse(project_id).label_tasks
            if name: tname = name.lower()

        self = self.with_context(
            empty_list_help_id=self.env.context.get('default_project_id'),
            empty_list_help_model='project.project',
            empty_list_help_document_name=tname,
        )
        return super(Task, self).get_empty_list_help(help)

    def message_subscribe(self, partner_ids=None, channel_ids=None, subtype_ids=None):
        """
        Add the users subscribed to allowed portal users
        """
        res = super(Task, self).message_subscribe(partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=subtype_ids)
        if partner_ids:
            new_allowed_users = self.env['res.partner'].browse(partner_ids).user_ids.filtered('share')
            tasks = self.filtered(lambda task: task.project_id.privacy_visibility == 'portal')
            tasks.sudo().allowed_user_ids |= new_allowed_users
        return res

    # ----------------------------------------
    # Case management
    # ----------------------------------------

    def stage_find(self, section_id, domain=[], order='sequence'):
        """ Override of the base.stage method
            Parameter of the stage search taken from the lead:
            - section_id: if set, stages must belong to this section or
              be a default stage; if not set, stages must be default
              stages
        """
        # collect all section_ids
        section_ids = []
        if section_id:
            section_ids.append(section_id)
        section_ids.extend(self.mapped('project_id').ids)
        search_domain = []
        if section_ids:
            search_domain = [('|')] * (len(section_ids) - 1)
            for section_id in section_ids:
                search_domain.append(('project_ids', '=', section_id))
        search_domain += list(domain)
        # perform search, return the first found
        return self.env['project.task.type'].search(search_domain, order=order, limit=1).id

    # ------------------------------------------------
    # CRUD overrides
    # ------------------------------------------------
    @api.model
    def default_get(self, default_fields):
        vals = super(Task, self).default_get(default_fields)

        days = list(DAYS.keys())
        week_start = fields.Datetime.today().weekday()

        if all(d in default_fields for d in days):
            vals[days[week_start]] = True
        if 'repeat_day' in default_fields:
            vals['repeat_day'] = str(fields.Datetime.today().day)
        if 'repeat_month' in default_fields:
            vals['repeat_month'] = self._fields.get('repeat_month').selection[fields.Datetime.today().month - 1][0]
        if 'repeat_until' in default_fields:
            vals['repeat_until'] = fields.Date.today() + timedelta(days=7)
        if 'repeat_weekday' in default_fields:
            vals['repeat_weekday'] = self._fields.get('repeat_weekday').selection[week_start][0]

        return vals

    @api.model_create_multi
    def create(self, vals_list):
        default_stage = dict()
        for vals in vals_list:
            project_id = vals.get('project_id') or self.env.context.get('default_project_id')
            if project_id and not "company_id" in vals:
                vals["company_id"] = self.env["project.project"].browse(
                    project_id
                ).company_id.id or self.env.company.id
            if project_id and "stage_id" not in vals:
                # 1) Allows keeping the batch creation of tasks
                # 2) Ensure the defaults are correct (and computed once by project),
                # by using default get (instead of _get_default_stage_id or _stage_find),
                if project_id not in default_stage:
                    default_stage[project_id] = self.with_context(
                        default_project_id=project_id
                    ).default_get(['stage_id']).get('stage_id')
                vals["stage_id"] = default_stage[project_id]
            # user_id change: update date_assign
            if vals.get('user_id'):
                vals['date_assign'] = fields.Datetime.now()
            # Stage change: Update date_end if folded stage and date_last_stage_update
            if vals.get('stage_id'):
                vals.update(self.update_date_end(vals['stage_id']))
                vals['date_last_stage_update'] = fields.Datetime.now()
            # recurrence
            rec_fields = vals.keys() & self._get_recurrence_fields()
            if rec_fields and vals.get('recurring_task') is True:
                rec_values = {rec_field: vals[rec_field] for rec_field in rec_fields}
                rec_values['next_recurrence_date'] = fields.Datetime.today()
                recurrence = self.env['project.task.recurrence'].create(rec_values)
                vals['recurrence_id'] = recurrence.id
        tasks = super().create(vals_list)
        for task in tasks:
            if task.project_id.privacy_visibility == 'portal':
                task._portal_ensure_token()
        return tasks

    def _load_records_create(self, values):
        tasks = super()._load_records_create(values)
        stage_ids_per_project = defaultdict(list)
        for task in tasks:
            if task.stage_id and task.stage_id not in task.project_id.type_ids and task.stage_id.id not in stage_ids_per_project[task.project_id]:
                stage_ids_per_project[task.project_id].append(task.stage_id.id)

        for project, stage_ids in stage_ids_per_project.items():
            project.write({'type_ids': [(4, stage_id) for stage_id in stage_ids]})

        return tasks

    def write(self, vals):
        now = fields.Datetime.now()
        if 'parent_id' in vals and vals['parent_id'] in self.ids:
            raise UserError(_("Sorry. You can't set a task as its parent task."))
        if 'active' in vals and not vals.get('active') and any(self.mapped('recurrence_id')):
            # TODO: show a dialog to stop the recurrence
            raise UserError(_('You cannot archive recurring tasks. Please, disable the recurrence first.'))
        # stage change: update date_last_stage_update
        if 'stage_id' in vals:
            vals.update(self.update_date_end(vals['stage_id']))
            vals['date_last_stage_update'] = now
            # reset kanban state when changing stage
            if 'kanban_state' not in vals:
                vals['kanban_state'] = 'normal'
        # user_id change: update date_assign
        if vals.get('user_id') and 'date_assign' not in vals:
            vals['date_assign'] = now

        # recurrence fields
        rec_fields = vals.keys() & self._get_recurrence_fields()
        if rec_fields:
            rec_values = {rec_field: vals[rec_field] for rec_field in rec_fields}
            for task in self:
                if task.recurrence_id:
                    task.recurrence_id.write(rec_values)
                elif vals.get('recurring_task'):
                    rec_values['next_recurrence_date'] = fields.Datetime.today()
                    recurrence = self.env['project.task.recurrence'].create(rec_values)
                    task.recurrence_id = recurrence.id

        if not vals.get('recurring_task', True) and self.recurrence_id:
            tasks_in_recurrence = self.recurrence_id.task_ids
            self.recurrence_id.unlink()
            tasks_in_recurrence.write({'recurring_task': False})

        tasks = self
        recurrence_update = vals.pop('recurrence_update', 'this')
        if recurrence_update != 'this':
            recurrence_domain = []
            if recurrence_update == 'subsequent':
                for task in self:
                    recurrence_domain = OR([recurrence_domain, ['&', ('recurrence_id', '=', task.recurrence_id.id), ('create_date', '>=', task.create_date)]])
            else:
                recurrence_domain = [('recurrence_id', 'in', self.recurrence_id.ids)]
            tasks |= self.env['project.task'].search(recurrence_domain)

        result = super(Task, tasks).write(vals)
        # rating on stage
        if 'stage_id' in vals and vals.get('stage_id'):
            self.filtered(lambda x: x.project_id.rating_active and x.project_id.rating_status == 'stage')._send_task_rating_mail(force_send=True)
        return result

    def update_date_end(self, stage_id):
        project_task_type = self.env['project.task.type'].browse(stage_id)
        if project_task_type.fold or project_task_type.is_closed:
            return {'date_end': fields.Datetime.now()}
        return {'date_end': False}

    def unlink(self):
        if any(self.mapped('recurrence_id')):
            # TODO: show a dialog to stop the recurrence
            raise UserError(_('You cannot delete recurring tasks. Please, disable the recurrence first.'))
        return super().unlink()

    # ---------------------------------------------------
    # Subtasks
    # ---------------------------------------------------

    @api.depends('parent_id.partner_id', 'project_id.partner_id')
    def _compute_partner_id(self):
        """
        If a task has no partner_id, use the project partner_id if any, or else the parent task partner_id.
        Once the task partner_id has been set:
            1) if the project partner_id changes, the task partner_id is automatically changed also.
            2) if the parent task partner_id changes, the task partner_id remains the same.
        """
        for task in self:
            if task.partner_id:
                if task.project_id.partner_id:
                    task.partner_id = task.project_id.partner_id
            else:
                task.partner_id = task.project_id.partner_id or task.parent_id.partner_id 

    @api.depends('partner_id.email', 'parent_id.email_from')
    def _compute_email_from(self):
        for task in self:
            task.email_from = task.partner_id.email or ((task.partner_id or task.parent_id) and task.email_from) or task.parent_id.email_from

    @api.depends('parent_id.project_id.subtask_project_id')
    def _compute_project_id(self):
        for task in self:
            if not task.project_id:
                task.project_id = task.parent_id.project_id.subtask_project_id

    # ---------------------------------------------------
    # Mail gateway
    # ---------------------------------------------------

    def _track_template(self, changes):
        res = super(Task, self)._track_template(changes)
        test_task = self[0]
        if 'stage_id' in changes and test_task.stage_id.mail_template_id:
            res['stage_id'] = (test_task.stage_id.mail_template_id, {
                'auto_delete_message': True,
                'subtype_id': self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note'),
                'email_layout_xmlid': 'mail.mail_notification_light'
            })
        return res

    def _creation_subtype(self):
        return self.env.ref('project.mt_task_new')

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'kanban_state_label' in init_values and self.kanban_state == 'blocked':
            return self.env.ref('project.mt_task_blocked')
        elif 'kanban_state_label' in init_values and self.kanban_state == 'done':
            return self.env.ref('project.mt_task_ready')
        elif 'stage_id' in init_values:
            return self.env.ref('project.mt_task_stage')
        return super(Task, self)._track_subtype(init_values)

    def _notify_get_groups(self, msg_vals=None):
        """ Handle project users and managers recipients that can assign
        tasks and create new one directly from notification emails. Also give
        access button to portal users and portal customers. If they are notified
        they should probably have access to the document. """
        groups = super(Task, self)._notify_get_groups(msg_vals=msg_vals)
        local_msg_vals = dict(msg_vals or {})
        self.ensure_one()

        project_user_group_id = self.env.ref('project.group_project_user').id
        project_manager_group_id = self.env.ref('project.group_project_manager').id

        group_func = lambda pdata: pdata['type'] == 'user' and project_user_group_id in pdata['groups']
        if self.project_id.privacy_visibility == 'followers':
            allowed_user_ids = self.project_id.allowed_internal_user_ids.partner_id.ids
            group_func = lambda pdata:\
                pdata['type'] == 'user'\
                and (
                        project_manager_group_id in pdata['groups']\
                        or (project_user_group_id in pdata['groups'] and pdata['id'] in allowed_user_ids)
                )
        new_group = ('group_project_user', group_func, {})

        if not self.user_id and not self.stage_id.fold:
            take_action = self._notify_get_action_link('assign', **local_msg_vals)
            project_actions = [{'url': take_action, 'title': _('I take it')}]
            new_group[2]['actions'] = project_actions

        groups = [new_group] + groups

        if self.project_id.privacy_visibility == 'portal':
            allowed_user_ids = self.project_id.allowed_portal_user_ids.partner_id.ids
            groups.insert(0, (
                'allowed_portal_users',
                lambda pdata: pdata['type'] == 'portal' and pdata['id'] in allowed_user_ids,
                {}
            ))

        portal_privacy = self.project_id.privacy_visibility == 'portal'
        for group_name, group_method, group_data in groups:
            if group_name in ('customer', 'user') or group_name == 'portal_customer' and not portal_privacy:
                group_data['has_button_access'] = False
            elif group_name == 'portal_customer' and portal_privacy:
                group_data['has_button_access'] = True

        return groups

    def _notify_get_reply_to(self, default=None, records=None, company=None, doc_names=None):
        """ Override to set alias of tasks to their project if any. """
        aliases = self.sudo().mapped('project_id')._notify_get_reply_to(default=default, records=None, company=company, doc_names=None)
        res = {task.id: aliases.get(task.project_id.id) for task in self}
        leftover = self.filtered(lambda rec: not rec.project_id)
        if leftover:
            res.update(super(Task, leftover)._notify_get_reply_to(default=default, records=None, company=company, doc_names=doc_names))
        return res

    def email_split(self, msg):
        email_list = tools.email_split((msg.get('to') or '') + ',' + (msg.get('cc') or ''))
        # check left-part is not already an alias
        aliases = self.mapped('project_id.alias_name')
        return [x for x in email_list if x.split('@')[0] not in aliases]

    @api.model
    def message_new(self, msg, custom_values=None):
        """ Overrides mail_thread message_new that is called by the mailgateway
            through message_process.
            This override updates the document according to the email.
        """
        # remove default author when going through the mail gateway. Indeed we
        # do not want to explicitly set user_id to False; however we do not
        # want the gateway user to be responsible if no other responsible is
        # found.
        create_context = dict(self.env.context or {})
        create_context['default_user_id'] = False
        if custom_values is None:
            custom_values = {}
        defaults = {
            'name': msg.get('subject') or _("No Subject"),
            'email_from': msg.get('from'),
            'planned_hours': 0.0,
            'partner_id': msg.get('author_id')
        }
        defaults.update(custom_values)

        task = super(Task, self.with_context(create_context)).message_new(msg, custom_values=defaults)
        email_list = task.email_split(msg)
        partner_ids = [p.id for p in self.env['mail.thread']._mail_find_partner_from_emails(email_list, records=task, force_create=False) if p]
        customer_ids = [p.id for p in self.env['mail.thread']._mail_find_partner_from_emails(tools.email_split(defaults['email_from']), records=task) if p]
        partner_ids += customer_ids
        task.message_subscribe(partner_ids)
        return task

    def message_update(self, msg, update_vals=None):
        """ Override to update the task according to the email. """
        email_list = self.email_split(msg)
        partner_ids = [p.id for p in self.env['mail.thread']._mail_find_partner_from_emails(email_list, records=self, force_create=False) if p]
        self.message_subscribe(partner_ids)
        return super(Task, self).message_update(msg, update_vals=update_vals)

    def _message_get_suggested_recipients(self):
        recipients = super(Task, self)._message_get_suggested_recipients()
        for task in self:
            if task.partner_id:
                reason = _('Customer Email') if task.partner_id.email else _('Customer')
                task._message_add_suggested_recipient(recipients, partner=task.partner_id, reason=reason)
            elif task.email_from:
                task._message_add_suggested_recipient(recipients, email=task.email_from, reason=_('Customer Email'))
        return recipients

    def _notify_email_header_dict(self):
        headers = super(Task, self)._notify_email_header_dict()
        if self.project_id:
            current_objects = [h for h in headers.get('X-Odoo-Objects', '').split(',') if h]
            current_objects.insert(0, 'project.project-%s, ' % self.project_id.id)
            headers['X-Odoo-Objects'] = ','.join(current_objects)
        if self.tag_ids:
            headers['X-Odoo-Tags'] = ','.join(self.tag_ids.mapped('name'))
        return headers

    def _message_post_after_hook(self, message, msg_vals):
        if message.attachment_ids and not self.displayed_image_id:
            image_attachments = message.attachment_ids.filtered(lambda a: a.mimetype == 'image')
            if image_attachments:
                self.displayed_image_id = image_attachments[0]

        if self.email_from and not self.partner_id:
            # we consider that posting a message with a specified recipient (not a follower, a specific one)
            # on a document without customer means that it was created through the chatter using
            # suggested recipients. This heuristic allows to avoid ugly hacks in JS.
            email_normalized = tools.email_normalize(self.email_from)
            new_partner = message.partner_ids.filtered(
                lambda partner: partner.email == self.email_from or (email_normalized and partner.email_normalized == email_normalized)
            )
            if new_partner:
                if new_partner[0].email_normalized:
                    email_domain = ('email_from', 'in', [new_partner[0].email, new_partner[0].email_normalized])
                else:
                    email_domain = ('email_from', '=', new_partner[0].email)
                self.search([
                    ('partner_id', '=', False), email_domain, ('stage_id.fold', '=', False)
                ]).write({'partner_id': new_partner[0].id})
        return super(Task, self)._message_post_after_hook(message, msg_vals)

    def action_assign_to_me(self):
        self.write({'user_id': self.env.user.id})

    # If depth == 1, return only direct children
    # If depth == 3, return children to third generation
    # If depth <= 0, return all children without depth limit
    def _get_all_subtasks(self, depth=0):
        children = self.mapped('child_ids').filtered(lambda children: children.active)
        if not children:
            return self.env['project.task']
        if depth == 1:
            return children
        return children + children._get_all_subtasks(depth - 1)

    def action_open_parent_task(self):
        return {
            'name': _('Parent Task'),
            'view_mode': 'form',
            'res_model': 'project.task',
            'res_id': self.parent_id.id,
            'type': 'ir.actions.act_window',
            'context': dict(self._context, create=False)
        }

    def action_subtask(self):
        action = self.env["ir.actions.actions"]._for_xml_id("project.project_task_action_sub_task")

        # display all subtasks of current task
        action['domain'] = [('id', 'child_of', self.id), ('id', '!=', self.id)]

        # update context, with all default values as 'quick_create' does not contains all field in its view
        if self._context.get('default_project_id'):
            default_project = self.env['project.project'].browse(self.env.context['default_project_id'])
        else:
            default_project = self.project_id.subtask_project_id or self.project_id
        ctx = dict(self.env.context)
        ctx = {k: v for k, v in ctx.items() if not k.startswith('search_default_')}
        ctx.update({
            'default_name': self.env.context.get('name', self.name) + ':',
            'default_parent_id': self.id,  # will give default subtask field in `default_get`
            'default_company_id': default_project.company_id.id if default_project else self.env.company.id,
        })

        action['context'] = ctx

        return action

    def action_recurring_tasks(self):
        return {
            'name': 'Tasks in Recurrence',
            'type': 'ir.actions.act_window',
            'res_model': 'project.task',
            'view_mode': 'tree,form',
            'domain': [('recurrence_id', 'in', self.recurrence_id.ids)],
        }

    # ---------------------------------------------------
    # Rating business
    # ---------------------------------------------------

    def _send_task_rating_mail(self, force_send=False):
        for task in self:
            rating_template = task.stage_id.rating_template_id
            if rating_template:
                task.rating_send_request(rating_template, lang=task.partner_id.lang, force_send=force_send)

    def rating_get_partner_id(self):
        res = super(Task, self).rating_get_partner_id()
        if not res and self.project_id.partner_id:
            return self.project_id.partner_id
        return res

    def rating_apply(self, rate, token=None, feedback=None, subtype_xmlid=None):
        return super(Task, self).rating_apply(rate, token=token, feedback=feedback, subtype_xmlid="project.mt_task_rating")

    def _rating_get_parent_field_name(self):
        return 'project_id'


class ProjectTags(models.Model):
    """ Tags of project's tasks """
    _name = "project.tags"
    _description = "Project Tags"

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char('Name', required=True)
    color = fields.Integer(string='Color', default=_get_default_color)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

```

## File: models\project_task_recurrence.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.exceptions import ValidationError

from calendar import monthrange
from dateutil.relativedelta import relativedelta
from dateutil.rrule import rrule, rruleset, DAILY, WEEKLY, MONTHLY, YEARLY, MO, TU, WE, TH, FR, SA, SU

MONTHS = {
    'january': 31,
    'february': 28,
    'march': 31,
    'april': 30,
    'may': 31,
    'june': 30,
    'july': 31,
    'august': 31,
    'september': 30,
    'october': 31,
    'november': 30,
    'december': 31,
}

DAYS = {
    'mon': MO,
    'tue': TU,
    'wed': WE,
    'thu': TH,
    'fri': FR,
    'sat': SA,
    'sun': SU,
}

WEEKS = {
    'first': 1,
    'second': 2,
    'third': 3,
    'last': 4,
}

class ProjectTaskRecurrence(models.Model):
    _name = 'project.task.recurrence'
    _description = 'Task Recurrence'

    task_ids = fields.One2many('project.task', 'recurrence_id')
    next_recurrence_date = fields.Date()
    recurrence_left = fields.Integer(string="Number of tasks left to create")

    repeat_interval = fields.Integer(string='Repeat Every', default=1)
    repeat_unit = fields.Selection([
        ('day', 'Days'),
        ('week', 'Weeks'),
        ('month', 'Months'),
        ('year', 'Years'),
    ], default='week')
    repeat_type = fields.Selection([
        ('forever', 'Forever'),
        ('until', 'End Date'),
        ('after', 'Number of Repetitions'),
    ], default="forever", string="Until")
    repeat_until = fields.Date(string="End Date")
    repeat_number = fields.Integer(string="Repetitions")

    repeat_on_month = fields.Selection([
        ('date', 'Date of the Month'),
        ('day', 'Day of the Month'),
    ])

    repeat_on_year = fields.Selection([
        ('date', 'Date of the Year'),
        ('day', 'Day of the Year'),
    ])

    mon = fields.Boolean(string="Mon")
    tue = fields.Boolean(string="Tue")
    wed = fields.Boolean(string="Wed")
    thu = fields.Boolean(string="Thu")
    fri = fields.Boolean(string="Fri")
    sat = fields.Boolean(string="Sat")
    sun = fields.Boolean(string="Sun")

    repeat_day = fields.Selection([
        (str(i), str(i)) for i in range(1, 32)
    ])
    repeat_week = fields.Selection([
        ('first', 'First'),
        ('second', 'Second'),
        ('third', 'Third'),
        ('last', 'Last'),
    ])
    repeat_weekday = fields.Selection([
        ('mon', 'Monday'),
        ('tue', 'Tuesday'),
        ('wed', 'Wednesday'),
        ('thu', 'Thursday'),
        ('fri', 'Friday'),
        ('sat', 'Saturday'),
        ('sun', 'Sunday'),
    ], string='Day Of The Week', readonly=False)
    repeat_month = fields.Selection([
        ('january', 'January'),
        ('february', 'February'),
        ('march', 'March'),
        ('april', 'April'),
        ('may', 'May'),
        ('june', 'June'),
        ('july', 'July'),
        ('august', 'August'),
        ('september', 'September'),
        ('october', 'October'),
        ('november', 'November'),
        ('december', 'December'),
    ])

    @api.constrains('repeat_unit', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun')
    def _check_recurrence_days(self):
        for project in self.filtered(lambda p: p.repeat_unit == 'week'):
            if not any([project.mon, project.tue, project.wed, project.thu, project.fri, project.sat, project.sun]):
                raise ValidationError('You should select a least one day')

    @api.constrains('repeat_interval')
    def _check_repeat_interval(self):
        if self.filtered(lambda t: t.repeat_interval <= 0):
            raise ValidationError('The interval should be greater than 0')

    @api.constrains('repeat_number', 'repeat_type')
    def _check_repeat_number(self):
        if self.filtered(lambda t: t.repeat_type == 'after' and t.repeat_number <= 0):
            raise ValidationError('Should repeat at least once')

    @api.constrains('repeat_type', 'repeat_until')
    def _check_repeat_until_date(self):
        today = fields.Date.today()
        if self.filtered(lambda t: t.repeat_type == 'until' and t.repeat_until < today):
            raise ValidationError('The end date should be in the future')

    @api.constrains('repeat_unit', 'repeat_on_month', 'repeat_day', 'repeat_type', 'repeat_until')
    def _check_repeat_until_month(self):
        if self.filtered(lambda r: r.repeat_type == 'until' and r.repeat_unit == 'month' and r.repeat_until and r.repeat_on_month == 'date'
           and int(r.repeat_day) > r.repeat_until.day and monthrange(r.repeat_until.year, r.repeat_until.month)[1] != r.repeat_until.day):
            raise ValidationError('The end date should be after the day of the month or the last day of the month')

    @api.model
    def _get_recurring_fields(self):
        return ['allowed_user_ids', 'company_id', 'description', 'displayed_image_id', 'email_cc',
                'parent_id', 'partner_email', 'partner_id', 'partner_phone', 'planned_hours',
                'project_id', 'project_privacy_visibility', 'sequence', 'tag_ids', 'recurrence_id',
                'name', 'recurring_task']

    def _get_weekdays(self, n=1):
        self.ensure_one()
        if self.repeat_unit == 'week':
            return [fn(n) for day, fn in DAYS.items() if self[day]]
        return [DAYS.get(self.repeat_weekday)(n)]

    @api.model
    def _get_next_recurring_dates(self, date_start, repeat_interval, repeat_unit, repeat_type, repeat_until, repeat_on_month, repeat_on_year, weekdays, repeat_day, repeat_week, repeat_month, **kwargs):
        count = kwargs.get('count', 1)
        rrule_kwargs = {'interval': repeat_interval or 1, 'dtstart': date_start}
        repeat_day = int(repeat_day)
        start = False
        dates = []
        if repeat_type == 'until':
            rrule_kwargs['until'] = repeat_until if repeat_until else fields.Date.today()
        else:
            rrule_kwargs['count'] = count

        if repeat_unit == 'week'\
            or (repeat_unit == 'month' and repeat_on_month == 'day')\
            or (repeat_unit == 'year' and repeat_on_year == 'day'):
            rrule_kwargs['byweekday'] = weekdays

        if repeat_unit == 'day':
            rrule_kwargs['freq'] = DAILY
        elif repeat_unit == 'month':
            rrule_kwargs['freq'] = MONTHLY
            if repeat_on_month == 'date':
                start = date_start - relativedelta(days=1)
                start = start.replace(day=min(repeat_day, monthrange(start.year, start.month)[1]))
                if start < date_start:
                    # Ensure the next recurrence is in the future
                    start += relativedelta(months=repeat_interval)
                    start = start.replace(day=min(repeat_day, monthrange(start.year, start.month)[1]))
                can_generate_date = (lambda: start <= repeat_until) if repeat_type == 'until' else (lambda: len(dates) < count)
                while can_generate_date():
                    dates.append(start)
                    start += relativedelta(months=repeat_interval)
                    start = start.replace(day=min(repeat_day, monthrange(start.year, start.month)[1]))
                return dates
        elif repeat_unit == 'year':
            rrule_kwargs['freq'] = YEARLY
            month = list(MONTHS.keys()).index(repeat_month) + 1
            rrule_kwargs['bymonth'] = month
            if repeat_on_year == 'date':
                rrule_kwargs['bymonthday'] = min(repeat_day, MONTHS.get(repeat_month))
                rrule_kwargs['bymonth'] = month
        else:
            rrule_kwargs['freq'] = WEEKLY

        rules = rrule(**rrule_kwargs)
        return list(rules) if rules else []

    def _new_task_values(self, task):
        self.ensure_one()
        fields_to_copy = self._get_recurring_fields()
        task_values = task.read(fields_to_copy).pop()
        create_values = {
            field: value[0] if isinstance(value, tuple) else value for field, value in task_values.items()
        }
        create_values['stage_id'] = task.project_id.type_ids[0].id if task.project_id.type_ids else task.stage_id.id
        create_values['user_id'] = False
        return create_values

    def _create_next_task(self):
        for recurrence in self:
            task = max(recurrence.sudo().task_ids, key=lambda t: t.id)
            create_values = recurrence._new_task_values(task)
            new_task = self.env['project.task'].sudo().create(create_values)
            if not new_task.parent_id and task.child_ids:
                children = []
                # copy the subtasks of the original task
                for child in task.child_ids:
                    child_values = recurrence._new_task_values(child)
                    child_values['parent_id'] = new_task.id
                    children.append(child_values)
                self.env['project.task'].create(children)

    def _set_next_recurrence_date(self):
        today = fields.Date.today()
        tomorrow = today + relativedelta(days=1)
        for recurrence in self.filtered(
            lambda r:
            r.repeat_type == 'after' and r.recurrence_left >= 0
            or r.repeat_type == 'until' and r.repeat_until >= today
            or r.repeat_type == 'forever'
        ):
            if recurrence.repeat_type == 'after' and recurrence.recurrence_left == 0:
                recurrence.next_recurrence_date = False
            else:
                next_date = self._get_next_recurring_dates(tomorrow, recurrence.repeat_interval, recurrence.repeat_unit, recurrence.repeat_type, recurrence.repeat_until, recurrence.repeat_on_month, recurrence.repeat_on_year, recurrence._get_weekdays(), recurrence.repeat_day, recurrence.repeat_week, recurrence.repeat_month, count=1)
                recurrence.next_recurrence_date = next_date[0] if next_date else False

    @api.model
    def _cron_create_recurring_tasks(self):
        if not self.env.user.has_group('project.group_project_recurring_tasks'):
            return
        today = fields.Date.today()
        recurring_today = self.search([('next_recurrence_date', '<=', today)])
        recurring_today._create_next_task()
        for recurrence in recurring_today.filtered(lambda r: r.repeat_type == 'after'):
            recurrence.recurrence_left -= 1
        recurring_today._set_next_recurrence_date()

    @api.model
    def create(self, vals):
        if vals.get('repeat_number'):
            vals['recurrence_left'] = vals.get('repeat_number')
        res = super(ProjectTaskRecurrence, self).create(vals)
        res._set_next_recurrence_date()
        return res

    def write(self, vals):
        if vals.get('repeat_number'):
            vals['recurrence_left'] = vals.get('repeat_number')

        res = super(ProjectTaskRecurrence, self).write(vals)

        if 'next_recurrence_date' not in vals:
            self._set_next_recurrence_date()
        return res

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_project_forecast = fields.Boolean(string="Planning")
    module_hr_timesheet = fields.Boolean(string="Task Logs")
    group_subtask_project = fields.Boolean("Sub-tasks", implied_group="project.group_subtask_project")
    group_project_rating = fields.Boolean("Customer Ratings", implied_group='project.group_project_rating')
    group_project_recurring_tasks = fields.Boolean("Recurring Tasks", implied_group="project.group_project_recurring_tasks")

    def set_values(self):

        # Ensure that settings on existing projects match the above fields
        projects = self.env["project.project"].search([])
        features = (
            # Pairs of associated (config_flag, project_flag)
            ("group_subtask_project", "allow_subtasks"),
            ("group_project_rating", "rating_active"),
            ("group_project_recurring_tasks", "allow_recurring_tasks"),
            )
        for (config_flag, project_flag) in features:
            config_flag_global = "project." + config_flag
            config_feature_enabled = self[config_flag]
            if (self.user_has_groups(config_flag_global)
                    is not config_feature_enabled):
                projects[project_flag] = config_feature_enabled

        super(ResConfigSettings, self).set_values()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    """ Inherits partner and adds Tasks information in the partner form """
    _inherit = 'res.partner'

    task_ids = fields.One2many('project.task', 'partner_id', string='Tasks')
    task_count = fields.Integer(compute='_compute_task_count', string='# Tasks')

    def _compute_task_count(self):
        fetch_data = self.env['project.task'].read_group([('partner_id', 'in', self.ids)], ['partner_id'], ['partner_id'])
        result = dict((data['partner_id'][0], data['partner_id_count']) for data in fetch_data)
        for partner in self:
            partner.task_count = result.get(partner.id, 0) + sum(c.task_count for c in partner.child_ids)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import analytic_account
from . import project_task_recurrence
from . import project
from . import res_config_settings
from . import res_partner
from . import digest

```

## File: populate\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import collections

from odoo import models
from odoo.tools import populate

_logger = logging.getLogger(__name__)

class ProjectStage(models.Model):
    _inherit = "project.task.type"
    _populate_sizes = {"small": 10, "medium": 50, "large": 500}

    def _populate_factories(self):
        return [
            ("name", populate.constant('stage_{counter}')),
            ("sequence", populate.randomize([False] + [i for i in range(1, 101)])),
            ("description", populate.constant('project_stage_description_{counter}')),
            ("active", populate.randomize([True, False], [0.8, 0.2])),
            ("fold", populate.randomize([True, False], [0.9, 0.1]))
        ]

class ProjectProject(models.Model):
    _inherit = "project.project"
    _populate_sizes = {"small": 10, "medium": 50, "large": 1000}
    _populate_dependencies = ["res.company", "project.task.type"]

    def _populate_factories(self):
        company_ids = self.env.registry.populated_models["res.company"]
        stage_ids = self.env.registry.populated_models["project.task.type"]

        def get_company_id(random, **kwargs):
            return random.choice(company_ids)
            # user_ids from company.user_ids ?
            # Also add a partner_ids on res_company ?

        def get_stage_ids(random, **kwargs):
            return [
                (6, 0, [
                    random.choice(stage_ids)
                    for i in range(random.choice([j for j in range(1, 10)]))
                ])
            ]

        return [
            ("name", populate.constant('project_{counter}')),
            ("sequence", populate.randomize([False] + [i for i in range(1, 101)])),
            ("active", populate.randomize([True, False], [0.8, 0.2])),
            ("company_id", populate.compute(get_company_id)),
            ("type_ids", populate.compute(get_stage_ids)),
            ('color', populate.randomize([False] + [i for i in range(1, 7)])),
            # TODO user_id but what about multi-company coherence ??
        ]


class ProjectTask(models.Model):
    _inherit = "project.task"
    _populate_sizes = {"small": 500, "medium": 5000, "large": 50000}
    _populate_dependencies = ["project.project"]

    def _populate_factories(self):
        project_ids = self.env.registry.populated_models["project.project"]
        stage_ids = self.env.registry.populated_models["project.task.type"]
        def get_project_id(random, **kwargs):
            return random.choice([False, False, False] + project_ids)
        def get_stage_id(random, **kwargs):
            return random.choice([False, False] + stage_ids)
        return [
            ("name", populate.constant('project_task_{counter}')),
            ("sequence", populate.randomize([False] + [i for i in range(1, 101)])),
            ("active", populate.randomize([True, False], [0.8, 0.2])),
            ("color", populate.randomize([False] + [i for i in range(1, 7)])),
            ("kanban_state", populate.randomize(['normal', 'done', 'blocked'])),
            ("project_id", populate.compute(get_project_id)),
            ("stage_id", populate.compute(get_stage_id)),
        ]

    def _populate(self, size):
        records = super()._populate(size)
        # set parent_ids
        self._populate_set_children_tasks(records, size)
        return records

    def _populate_set_children_tasks(self, tasks, size):
        _logger.info('Setting parent tasks')
        rand = populate.Random('project.task+children_generator')
        parents = self.env["project.task"]
        for task in tasks:
            if not rand.getrandbits(4):
                parents |= task
        parent_ids = parents.ids
        tasks -= parents
        parent_childs = collections.defaultdict(lambda: self.env['project.task'])
        for count, task in enumerate(tasks):
            if not rand.getrandbits(4):
                parent_childs[rand.choice(parent_ids)] |= task

        for count, (parent, childs) in enumerate(parent_childs.items()):
            if (count + 1) % 100 == 0:
                _logger.info('Setting parent: %s/%s', count + 1, len(parent_childs))
            childs.write({'parent_id': parent})

```

## File: populate\__init__.py

```python
from . import project

```

## File: report\project_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools


class ReportProjectTaskUser(models.Model):
    _name = "report.project.task.user"
    _description = "Tasks Analysis"
    _order = 'name desc, project_id'
    _auto = False

    name = fields.Char(string='Task Title', readonly=True)
    user_id = fields.Many2one('res.users', string='Assigned To', readonly=True)
    date_assign = fields.Datetime(string='Assignment Date', readonly=True)
    date_end = fields.Datetime(string='Ending Date', readonly=True)
    date_deadline = fields.Date(string='Deadline', readonly=True)
    date_last_stage_update = fields.Datetime(string='Last Stage Update', readonly=True)
    project_id = fields.Many2one('project.project', string='Project', readonly=True)
    working_days_close = fields.Float(string='# Working Days to Close',
        digits=(16,2), readonly=True, group_operator="avg",
        help="Number of Working Days to close the task")
    working_days_open = fields.Float(string='# Working Days to Assign',
        digits=(16,2), readonly=True, group_operator="avg",
        help="Number of Working Days to Open the task")
    delay_endings_days = fields.Float(string='# Days to Deadline', digits=(16,2), readonly=True)
    nbr = fields.Integer('# of Tasks', readonly=True)  # TDE FIXME master: rename into nbr_tasks
    priority = fields.Selection([
        ('0', 'Low'),
        ('1', 'Normal'),
        ('2', 'High')
        ], readonly=True, string="Priority")
    state = fields.Selection([
            ('normal', 'In Progress'),
            ('blocked', 'Blocked'),
            ('done', 'Ready for next stage')
        ], string='Kanban State', readonly=True)
    company_id = fields.Many2one('res.company', string='Company', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    stage_id = fields.Many2one('project.task.type', string='Stage', readonly=True)

    def _select(self):
        select_str = """
             SELECT
                    (select 1 ) AS nbr,
                    t.id as id,
                    t.date_assign as date_assign,
                    t.date_end as date_end,
                    t.date_last_stage_update as date_last_stage_update,
                    t.date_deadline as date_deadline,
                    t.user_id,
                    t.project_id,
                    t.priority,
                    t.name as name,
                    t.company_id,
                    t.partner_id,
                    t.stage_id as stage_id,
                    t.kanban_state as state,
                    t.working_days_close as working_days_close,
                    t.working_days_open  as working_days_open,
                    (extract('epoch' from (t.date_deadline-(now() at time zone 'UTC'))))/(3600*24)  as delay_endings_days
        """
        return select_str

    def _group_by(self):
        group_by_str = """
                GROUP BY
                    t.id,
                    t.create_date,
                    t.write_date,
                    t.date_assign,
                    t.date_end,
                    t.date_deadline,
                    t.date_last_stage_update,
                    t.user_id,
                    t.project_id,
                    t.priority,
                    t.name,
                    t.company_id,
                    t.partner_id,
                    t.stage_id
        """
        return group_by_str

    def init(self):
        tools.drop_view_if_exists(self._cr, self._table)
        self._cr.execute("""
            CREATE view %s as
              %s
              FROM project_task t
                WHERE t.active = 'true'
                AND t.project_id IS NOT NULL
                %s
        """ % (self._table, self._select(), self._group_by()))

```

## File: report\project_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="view_task_project_user_pivot" model="ir.ui.view">
            <field name="name">report.project.task.user.pivot</field>
            <field name="model">report.project.task.user</field>
            <field name="arch" type="xml">
                <pivot string="Tasks Analysis" display_quantity="true" disable_linking="True" sample="1">
                    <field name="project_id" type="row"/>
                </pivot>
            </field>
        </record>

        <record id="view_task_project_user_graph" model="ir.ui.view">
            <field name="name">report.project.task.user.graph</field>
            <field name="model">report.project.task.user</field>
            <field name="arch" type="xml">
                <graph string="Tasks Analysis" type="bar" sample="1" disable_linking="1">
                     <field name="project_id" type="row"/>
                     <field name="user_id" type="col"/>
                     <field name="nbr" type="measure"/>
                 </graph>
             </field>
        </record>

        <record id="view_task_project_user_search" model="ir.ui.view">
            <field name="name">report.project.task.user.search</field>
            <field name="model">report.project.task.user</field>
            <field name="arch" type="xml">
                <search string="Tasks Analysis">
                    <field name="name" />
                    <field name="date_assign"/>
                    <field name="date_end"/>
                    <field name="date_deadline"/>
                    <field name="date_last_stage_update"/>
                    <field name="project_id"/>
                    <field name="user_id"/>
                    <field name="partner_id" filter_domain="[('partner_id', 'child_of', self)]"/>
                    <field name="stage_id"/>
                    <filter string="Unassigned" name="unassigned" domain="[('user_id', '=', False)]"/>
                    <separator/>
                    <filter name="filter_date_deadline" date="date_deadline"/>
                    <filter name="filter_date_assign" date="date_assign"/>
                    <filter name="filter_date_last_stage_update" date="date_last_stage_update"/>
                    <group expand="0" string="Extended Filters">
                        <field name="priority"/>
                        <field name="company_id" groups="base.group_multi_company"/>
                    </group>
                    <group expand="1" string="Group By">
                        <filter string="Project" name="project" context="{'group_by': 'project_id'}"/>
                        <filter string="Assigned to" name="User" context="{'group_by': 'user_id'}"/>
                        <filter string="Stage" name="Stage" context="{'group_by': 'stage_id'}"/>
                    </group>
                </search>
            </field>
        </record>

       <record id="action_project_task_user_tree" model="ir.actions.act_window">
            <field name="name">Tasks Analysis</field>
            <field name="res_model">report.project.task.user</field>
            <field name="view_mode">graph,pivot</field>
            <field name="search_view_id" ref="view_task_project_user_search"/>
            <field name="context">{'group_by_no_leaf':1,'group_by':[]}</field>
            <field name="help">This report allows you to analyse the performance of your projects and users. You can analyse the quantities of tasks, the hours spent compared to the planned hours, the average number of days to open or close a task, etc.</field>
        </record>

</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_project_project,project.project,model_project_project,project.group_project_user,1,0,0,0
access_project_project_manager,project.project,model_project_project,project.group_project_manager,1,1,1,1
access_project_task_type_user,project.task.type.user,model_project_task_type,base.group_user,1,0,0,0
access_project_task_type_project_user,project.task.type.project.user,model_project_task_type,project.group_project_user,1,0,0,0
access_project_task_type_manager,project.task.type manager,model_project_task_type,project.group_project_manager,1,1,1,1
access_project_task_type_portal,task_type_portal,project.model_project_task_type,base.group_portal,1,0,0,0
access_project_task,project.task,model_project_task,project.group_project_user,1,1,1,1
access_report_project_task_user,report.project.task.user,model_report_project_task_user,project.group_project_manager,1,1,1,1
access_partner_task_user,base.res.partner user,base.model_res_partner,project.group_project_user,1,0,0,0
access_task_on_partner,project.task on partners,model_project_task,base.group_user,1,0,0,0
access_task_portal,task_portal,project.model_project_task,base.group_portal,1,0,0,0
access_project_user,project.project on partners,model_project_project,base.group_user,1,0,0,0
access_project_portal,project_portal,project.model_project_project,base.group_portal,1,0,0,0
access_resource_calendar,project.resource_calendar user,resource.model_resource_calendar,project.group_project_user,1,0,0,0
access_resource_calendar_attendance,project.resource_calendar_attendance user,resource.model_resource_calendar_attendance,project.group_project_user,1,0,0,0
access_resource_calendar_leaves_user,resource.calendar.leaves user,resource.model_resource_calendar_leaves,project.group_project_user,1,1,1,1
access_project_tags_all,project.project_tags_all,model_project_tags,,1,0,0,0
access_project_tags_manager,project.project_tags_manager,model_project_tags,project.group_project_manager,1,1,1,1
access_project_tags_portal,project_tags_portal,project.model_project_tags,base.group_portal,1,0,0,0
access_mail_activity_type_project_manager,mail.activity.type.project.manager,mail.model_mail_activity_type,project.group_project_manager,1,1,1,1
access_account_analytic_account_user,account.analytic.account,analytic.model_account_analytic_account,project.group_project_user,1,0,0,0
access_account_analytic_account_manager,account.analytic.account,analytic.model_account_analytic_account,project.group_project_manager,1,1,1,1
access_account_analytic_line_project,account.analytic.line project,analytic.model_account_analytic_line,project.group_project_manager,1,1,1,1
access_project_delete_wizard,access_project_delete_wizard,model_project_delete_wizard,project.group_project_manager,1,1,1,1
access_project_task_type_delete_wizard,project.task.type.delete.wizard,model_project_task_type_delete_wizard,project.group_project_manager,1,1,1,1
access_project_task_recurrence,project.task.recurrence,model_project_task_recurrence,project.group_project_user,1,1,1,1

```

## File: security\project_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="base.module_category_services_project" model="ir.module.category">
        <field name="description">Helps you manage your projects and tasks by tracking them, generating plannings, etc...</field>
        <field name="sequence">3</field>
    </record>

    <record id="group_project_user" model="res.groups">
        <field name="name">User</field>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
        <field name="category_id" ref="base.module_category_services_project"/>
    </record>

    <record id="group_project_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_services_project"/>
        <field name="implied_ids" eval="[(4, ref('group_project_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="group_subtask_project" model="res.groups">
        <field name="name">Use Subtasks</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_project_rating" model="res.groups">
        <field name="name">Use Rating on Project</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_project_recurring_tasks" model="res.groups">
        <field name="name">Use Recurring Tasks</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

<data noupdate="1">
    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('project.group_project_manager'))]"/>
    </record>

    <record model="ir.rule" id="project_comp_rule">
        <field name="name">Project: multi-company</field>
        <field name="model_id" ref="model_project_project"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="project_project_manager_rule">
        <field name="name">Project: project manager: see all</field>
        <field name="model_id" ref="model_project_project"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('project.group_project_manager'))]"/>
    </record>

    <record model="ir.rule" id="project_public_members_rule">
        <field name="name">Project: Only invited users</field>
        <field name="model_id" ref="model_project_project"/>
        <field name="domain_force">[
        '|',
            ('privacy_visibility', '!=', 'followers'),
            ('allowed_internal_user_ids', 'in', user.ids),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record model="ir.rule" id="task_comp_rule">
        <field name="name">Project/Task: multi-company</field>
        <field name="model_id" ref="model_project_task"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="task_visibility_rule">
        <field name="name">Project/Task: employees: follow required for follower-only projects</field>
        <field name="model_id" ref="model_project_task"/>
        <field name="domain_force">[
        '|',
            ('project_id.privacy_visibility', '!=', 'followers'),
            ('allowed_user_ids', 'in', user.ids),
        ]</field>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record model="ir.rule" id="project_manager_all_project_tasks_rule">
        <field name="name">Project/Task: project manager: see all</field>
        <field name="model_id" ref="model_project_task"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('project.group_project_manager'))]"/>
    </record>

    <record model="ir.rule" id="report_project_task_user_report_comp_rule">
        <field name="name">Task Analysis multi-company</field>
        <field name="model_id" ref="model_report_project_task_user"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

     <!-- Portal -->
    <record id="project_project_rule_portal" model="ir.rule">
        <field name="name">Project: portal users: portal and following</field>
        <field name="model_id" ref="project.model_project_project"/>
        <field name="domain_force">[
            ('privacy_visibility', '=', 'portal'),
            ('allowed_portal_user_ids', 'in', user.ids),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record id="project_task_rule_portal" model="ir.rule">
        <field name="name">Project/Task: portal users: (portal and following project) or (portal and following task)</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="domain_force">[
            ('project_id.privacy_visibility', '=', 'portal'),
            ('allowed_user_ids', 'in', user.ids),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="98.162%" x2="0%" y1="1.838%" y2="100%"><stop offset="0%" stop-color="#797DA5"/><stop offset="50.799%" stop-color="#6D7194"/><stop offset="100%" stop-color="#626584"/></linearGradient><path id="d" d="M55.253 36.993c-3.23 0-4.629 2.41-6.384 2.41-4.666 0-.419-13.45-.419-13.45s-15.27 6.106-15.27-.251c0-2.735 2.822-3.53 2.822-6.563 0-2.709-2.187-4.176-4.781-4.176-2.696 0-5.163 1.442-5.163 4.3 0 3.158 2.467 4.525 2.467 6.24 0 5.313-13.684 2.187-13.684 2.187v25.432s13.898 3.133 13.898-2.187c0-1.715-3.112-3.061-3.112-6.218 0-2.859 2.275-4.3 4.946-4.3 2.62 0 4.807 1.466 4.807 4.176 0 3.032-2.823 3.828-2.823 6.562 0 4.64 10.088 1.963 14.1 1.963 0 0-2.702-9.165 2.009-9.165 2.797 0 3.611 2.759 6.714 2.759 2.773 0 4.273-2.138 4.273-4.698 0-2.61-1.475-5.021-4.4-5.021z"/><path id="e" d="M55.253 34.993c-3.23 0-4.629 2.41-6.384 2.41-4.666 0 1.522-11.755-.419-13.45-1.94-1.695-15.27 6.106-15.27-.251 0-2.735 2.822-3.53 2.822-6.563 0-2.709-2.187-4.176-4.781-4.176-2.696 0-5.163 1.442-5.163 4.3 0 3.158 2.467 4.525 2.467 6.24 0 5.313-13.684 2.187-13.684 2.187v25.432s13.898 3.133 13.898-2.187c0-1.715-3.112-3.061-3.112-6.218 0-2.859 2.275-4.3 4.946-4.3 2.62 0 4.807 1.466 4.807 4.176 0 3.032-2.823 3.828-2.823 6.562 0 4.64 12.677 2.6 14.1 1.963 1.422-.636-2.702-9.165 2.009-9.165 2.797 0 3.611 2.759 6.714 2.759 2.773 0 4.273-2.138 4.273-4.698 0-2.61-1.475-5.021-4.4-5.021z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 70c-2 0-4-.148-4-4.15V39.197L27 15l11 24.84 6 .066 11.113-.066 4.21 2.112L42.511 70H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\portal_rating.js

```javascript
odoo.define('website_rating_project.rating', function (require) {
'use strict';

var time = require('web.time');
var publicWidget = require('web.public.widget');

publicWidget.registry.ProjectRatingImage = publicWidget.Widget.extend({
    selector: '.o_portal_project_rating .o_rating_image',

    /**
     * @override
     */
    start: function () {
        this.$el.popover({
            placement: 'bottom',
            trigger: 'hover',
            html: true,
            content: function () {
                var $elem = $(this);
                var id = $elem.data('id');
                var ratingDate = $elem.data('rating-date');
                var baseDate = time.auto_str_to_date(ratingDate);
                var duration = moment(baseDate).fromNow();
                var $rating = $('#rating_' + id);
                $rating.find('.rating_timeduration').text(duration);
                return $rating.html();
            },
        });
        return this._super.apply(this, arguments);
    },
});
});

```

## File: static\src\js\project_calendar.js

```javascript
odoo.define('project.ProjectCalendarView', function (require) {
"use strict";

const CalendarController = require('web.CalendarController');
const CalendarView = require('web.CalendarView');
const viewRegistry = require('web.view_registry');

const ProjectCalendarController = CalendarController.extend({
    _renderButtonsParameters() {
        return _.extend({}, this._super(...arguments),  {scaleDrop: true});
    },
});

const ProjectCalendarView = CalendarView.extend({
        config: _.extend({}, CalendarView.prototype.config, {
            Controller: ProjectCalendarController,
        }),
    });

viewRegistry.add('project_calendar', ProjectCalendarView);
return ProjectCalendarView;
});

```

## File: static\src\js\project_kanban.js

```javascript
odoo.define('project.project_kanban', function (require) {
'use strict';

var KanbanController = require('web.KanbanController');
var KanbanView = require('web.KanbanView');
var KanbanColumn = require('web.KanbanColumn');
var view_registry = require('web.view_registry');
var KanbanRecord = require('web.KanbanRecord');

KanbanRecord.include({
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
     // YTI TODO: Should be transformed into a extend and specific to project
    _openRecord: function () {
        if (this.selectionMode !== true && this.modelName === 'project.project' &&
            this.$(".o_project_kanban_boxes a").length) {
            this.$('.o_project_kanban_boxes a').first().click();
        } else {
            this._super.apply(this, arguments);
        }
    },
});

var ProjectKanbanController = KanbanController.extend({
    custom_events: _.extend({}, KanbanController.prototype.custom_events, {
        'kanban_column_delete_wizard': '_onDeleteColumnWizard',
    }),

    _onDeleteColumnWizard: function (ev) {
        ev.stopPropagation();
        const self = this;
        const column_id = ev.target.id;
        var state = this.model.get(this.handle, {raw: true});
        this._rpc({
            model: 'project.task.type',
            method: 'unlink_wizard',
            args: [column_id],
            context: state.getContext(),
        }).then(function (res) {
            self.do_action(res);
        });
    }
});

var ProjectKanbanView = KanbanView.extend({
    config: _.extend({}, KanbanView.prototype.config, {
        Controller: ProjectKanbanController
    }),
});

KanbanColumn.include({
    _onDeleteColumn: function (event) {
        if (this.modelName === 'project.task' && this.groupedBy === 'stage_id') {
            event.preventDefault();
            this.trigger_up('kanban_column_delete_wizard');
            return;
        }
        this._super.apply(this, arguments);
    }
});

view_registry.add('project_kanban', ProjectKanbanView);

return ProjectKanbanController;
});

```

## File: static\src\js\project_rating_reporting.js

```javascript
odoo.define('project.project_rating_reporting', function (require) {
'use strict';

const core = require('web.core');
const _t = core._t;

const viewRegistry = require('web.view_registry');

const PivotController = require('web.PivotController');
const PivotView = require('web.PivotView');

const GraphController = require('web.GraphController');
const GraphView = require('web.GraphView');

var ProjectPivotController = PivotController.extend({
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        var measures = JSON.parse(JSON.stringify(this.measures));
        if ('res_id' in measures) {
            measures.res_id.string = _t('Task');
        }
        if ('parent_res_id' in measures) {
            measures.parent_res_id.string = _t('Project');
        }
        if ('rating' in measures) {
            measures.rating.string = _t('Rating Value (/5)');
        }
        this.measures = measures;
    },
});

var ProjectPivotView = PivotView.extend({
    config: _.extend({}, PivotView.prototype.config, {
        Controller: ProjectPivotController,
    }),
});

viewRegistry.add('project_rating_pivot', ProjectPivotView);

var ProjectGraphController = GraphController.extend({
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        _.each(this.measures, measure => {
            if (measure.fieldName === 'res_id') {
                measure.description = _t('Task');
            } else if (measure.fieldName === 'parent_res_id') {
                measure.description = _t('Project');
            } else if (measure.fieldName === 'rating') {
                measure.description = _t('Rating Value (/5)');
            }
        });
    },
});

var ProjectGraphView = GraphView.extend({
    config: _.extend({}, GraphView.prototype.config, {
        Controller: ProjectGraphController,
    }),
});

viewRegistry.add('project_rating_graph', ProjectGraphView);

});

```

## File: static\src\js\project_task_kanban_examples.js

```javascript
odoo.define('project.task_kanban_examples', function (require) {
'use strict';

var core = require('web.core');
var kanbanExamplesRegistry = require('web.kanban_examples_registry');

var _lt = core._lt;

var greenBullet = '<span class="o_status o_status_green"></span>';
var redBullet = '<span class="o_status o_status_red"></span>';
var star = '<a style="color: gold;" class="fa fa-star"/>';
var clock = '<a class="fa fa-clock-o" />'

var description_activities = escFormat(_lt('%s Use the %s icon to organize your daily activities.'), '<br/>', clock);
var description = escFormat(_lt('Prioritize Tasks by using the %s icon.'+
            '%s Use the %s button to signalize to your colleagues that a task is ready for the next stage.'+
            '%s Use the %s to signalize a problem or a need for discussion on a task.'+
            '%s'), star, '<br/>', greenBullet, '<br/>', redBullet, description_activities);

/**
 * Helper function to escape a text before formatting it.
 *
 * First argument is the string to format and the other arguments are the values
 * to inject into the string.
 *
 * Sort of 'lazy escaping' as it is used alongside _lt.
 *
 * @returns {string} the formatted and escaped string
 */
function escFormat() {
    var args = arguments;
    return {
        toString: function () {
            args[0] = _.escape(args[0]);
            return _.str.sprintf.apply(_.str, args);
        },
    };
}

kanbanExamplesRegistry.add('project', {
    ghostColumns: [_lt('New'), _lt('Assigned'), _lt('In Progress'), _lt('Done')],
    applyExamplesText: _lt("Use This For My Project"),
    examples:[{
        name: _lt('Software Development'),
        columns: [_lt('Backlog'), _lt('Specifications'), _lt('Development'), _lt('Tests'), _lt('Delivered')],
        description: escFormat(_lt('Prioritize Tasks by using the %s icon.'+
            '%s Use the %s button to inform your colleagues that a task is ready for the next stage.'+
            '%s Use the %s to indicate a problem or a need for discussion on a task.'+
            '%s'), star, '<br/>', greenBullet, '<br/>', redBullet, description_activities),
        bullets: [greenBullet, redBullet, star],
    }, {
        name: _lt('Agile Scrum'),
        columns: [_lt('Backlog'), _lt('Sprint Backlog'), _lt('Sprint in Progress'), _lt('Sprint Complete'), _lt('Old Completed Sprint')],
        description: escFormat(_lt('Waiting for the next stage: use %s and %s bullets. %s'), greenBullet, redBullet, description_activities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Digital Marketing'),
        columns: [_lt('Ideas'), _lt('Researching'), _lt('Writing'), _lt('Editing'), _lt('Done')],
        description: escFormat(_lt('Everyone can propose ideas, and the Editor marks the best ones ' +
            'as %s. Attach all documents or links to the task directly, to have all information about ' +
            'a research centralized. %s'), greenBullet, description_activities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Customer Feedback'),
        columns: [_lt('New'), _lt('In development'), _lt('Done'), _lt('Refused')],
        description: escFormat(_lt('Customers propose feedbacks by email; Odoo creates tasks ' +
            'automatically, and you can communicate on the task directly. Your managers decide which ' +
            'feedback is accepted %s and which feedback is moved to the %s column. %s'), greenBullet, _lt('"Refused"'), description_activities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Getting Things Done (GTD)'),
        columns: [_lt('Inbox'), _lt('Today'), _lt('This Week'), _lt('This Month'), _lt('Long Term')],
        description: escFormat(_lt('Fill your Inbox easily with the email gateway. Periodically review your ' +
            'Inbox and schedule tasks by moving them to others columns. Every day, you review the ' +
            '%s column to move important tasks %s. Every Monday, you review the %s column. %s'), _lt('"This Week"'), _lt('"Today"'), _lt('"This Month"'), description_activities),
    }, {
        name: _lt('Consulting'),
        columns: [_lt('New Projects'), _lt('Resources Allocation'), _lt('In Progress'), _lt('Done')],
        description: escFormat(_lt('Manage the lifecycle of your project using the kanban view. Add newly acquired projects, assign them and use the %s and %s to define if the project is ready for the next step. %s'), greenBullet, redBullet, description_activities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Research Project'),
        columns: [_lt('Brainstorm'), _lt('Research'), _lt('Draft'), _lt('Final Document')],
        description: escFormat(_lt('Handle your idea gathering within Tasks of your new Project and discuss them in the chatter of the tasks. Use the %s and %s to signalize what is the current status of your Idea. %s'), greenBullet, redBullet, description_activities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Website Redesign'),
        columns: [_lt('Page Ideas'), _lt('Copywriting'), _lt('Design'), _lt('Live')],
        description: escFormat(_lt('Handle your idea gathering within Tasks of your new Project and discuss them in the chatter of the tasks. Use the %s and %s to signalize what is the current status of your Idea. %s'), greenBullet, redBullet, description_activities),
    }, {
        name: _lt('T-shirt Printing'),
        columns: [_lt('New Orders'), _lt('Logo Design'), _lt('To Print'), _lt('Done')],
        description: escFormat(_lt('Communicate with customers on the task using the email gateway. ' +
            'Attach logo designs to the task, so that information flow from designers to the workers ' +
            'who print the t-shirt. Organize priorities amongst orders %s using the icon. %s'), star, description_activities),
        bullets: [star],
    }, {
        name: _lt('Design'),
        columns: [_lt('New Request'), _lt('Design'), _lt('Client Review'), _lt('Handoff')],
        description: description,
        bullets: [greenBullet, redBullet, star, clock],
    }, {
        name: _lt('Publishing'),
        columns: [_lt('Ideas'), _lt('Writing'), _lt('Editing'), _lt('Published')],
        description: description,
        bullets: [greenBullet, redBullet, star, clock],
    }, {
        name: _lt('Manufacturing'),
        columns: [_lt('New Orders'), _lt('Material Sourcing'), _lt('Manufacturing'), _lt('Assembling'), _lt('Delivered')],
        description: description,
        bullets: [greenBullet, redBullet, star, clock],
    }, {
        name: _lt('Podcast and Video Production'),
        columns: [_lt('Research'), _lt('Script'), _lt('Recording'), _lt('Mixing'), _lt('Publishing')],
        description: description,
        bullets: [greenBullet, redBullet, star, clock],
    }],
});

});

```

## File: static\src\js\tours\project.js

```javascript
odoo.define('project.tour', function(require) {
"use strict";

var core = require('web.core');
var tour = require('web_tour.tour');

var _t = core._t;

tour.register('project_tour', {
    sequence: 110,
    url: "/web",
    rainbowManMessage: "Congratulations, you are now a master of project management.",
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="project.menu_main_pm"]',
    content: _t('Want a better way to <b>manage your projects</b>? <i>It starts here.</i>'),
    position: 'right',
    edition: 'community',
}, {
    trigger: '.o_app[data-menu-xmlid="project.menu_main_pm"]',
    content: _t('Want a better way to <b>manage your projects</b>? <i>It starts here.</i>'),
    position: 'bottom',
    edition: 'enterprise',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_project_kanban',
    content: _t('Let\'s create your first <b>project</b>.'),
    position: 'bottom',
    width: 200,
}, {
    trigger: 'input.o_project_name',
    content: _t('Choose a <b>name</b> for your project. <i>It can be anything you want: the name of a customer,\
     of a product, of a team, of a construction site...</i>'),
    position: 'right',
}, {
    trigger: '.o_open_tasks',
    content: _t('Let\'s create your first <b>project</b>.'),
    position: 'top',
    run: function (actions) {
        actions.auto('.modal:visible .btn.btn-primary');
    },
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create input",
    content: _t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>."),
    position: 'bottom',
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    auto: true,
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create input",
    extra_trigger: '.o_kanban_group',
    content: _t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>."),
    position: 'bottom',
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    auto: true,
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_kanban_group:eq(1)',
    content: _t("Let's create your first <b>task</b>."),
    position: 'bottom',
    width: 200,
}, {
    trigger: '.o_kanban_quick_create input.o_field_char[name=name]',
    extra_trigger: '.o_kanban_project_tasks',
    content: _t('Choose a task <b>name</b> <i>(e.g. Website Design, Purchase Goods...)</i>'),
    position: 'right',
}, {
    trigger: '.o_kanban_quick_create .o_kanban_add',
    extra_trigger: '.o_kanban_project_tasks',
    content: _t("Add your task once it is ready."),
    position: "bottom",
}, {
    trigger: ".o_kanban_record .oe_kanban_content",
    extra_trigger: '.o_kanban_project_tasks',
    content: _t("<b>Drag &amp; drop</b> the card to change your task from stage."),
    position: "bottom",
    run: "drag_and_drop .o_kanban_group:eq(1) ",
}, {
    trigger: ".o_kanban_record:first",
    extra_trigger: '.o_kanban_project_tasks',
    content: _t("Let's start working on your task."),
    position: "bottom",
}, {
    trigger: ".o_ChatterTopbar_buttonSendMessage",
    extra_trigger: '.o_form_project_tasks',
    content: _t("Use this chatter to <b>send emails</b> and communicate efficently with your customers. \
    Add new people in the followers list to make them aware about the main changes about this task."),
    width: 350,
    position: "bottom",
}, {
    trigger: ".o_ChatterTopbar_buttonLogNote",
    extra_trigger: '.o_form_project_tasks',
    content: _t("<b>Log notes</b> for internal communications <i>(the people following this task won't be notified \
    of the note you are logging unless you specifically tag them)</i>. Use @ <b>mentions</b> to ping a colleague \
    or # <b>mentions</b> to reach an entire team."),
    width: 350,
    position: "bottom"
}, {
    trigger: ".o_ChatterTopbar_buttonScheduleActivity",
    extra_trigger: '.o_form_project_tasks',
    content: _t("Use <b>activities</b> to organize your daily work."),
}, {
    trigger: ".modal-dialog .btn-primary",
    extra_trigger: '.o_form_project_tasks',
    content: "Schedule your activity once it is ready",
    position: "bottom",
    run: "click",
}, {
    trigger: ".breadcrumb-item:not(.active):last",
    extra_trigger: '.o_form_project_tasks.o_form_readonly',
    content: _t("Let's go back to your <b>kanban view</b> to have an overview of your next tasks."),
    position: "right",
    run: 'click',
}]);

});

```

## File: static\src\xml\project_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="day_of_week_widget">
        <table class="o_dow_widget">
            <thead>
                <tr>
                    <th t-foreach="widget.labels" t-as="label">
                        <t t-esc="label"/>
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr class="o_dow_days" />
            </tbody>
        </table>
    </t>
</templates>

```

## File: views\analytic_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--
        Analytic Accounts with project
    -->
    <record id="account_analytic_account_view_form_inherit" model="ir.ui.view">
        <field name="name">account.analytic.account.form.inherit</field>
        <field name="model">account.analytic.account</field>
        <field name="inherit_id" ref="analytic.view_account_analytic_account_form"/>
        <field eval="18" name="priority"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button" type="object" name="action_view_projects"
                    icon="fa-puzzle-piece" attrs="{'invisible': [('project_count', '=', 0)]}">
                    <field string="Projects" name="project_count" widget="statinfo"/>
                </button>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\digest_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record id="digest_digest_view_form" model="ir.ui.view">
        <field name="name">digest.digest.view.form.inherit.project.task</field>
        <field name="model">digest.digest</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpi_general']" position="after">
                <group name="kpi_project" string="Project" groups="project.group_project_user">
                    <field name="kpi_project_task_opened"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Activity types config -->
    <record id="mail_activity_type_action_config_project_types" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model_id', '=', False), ('res_model_id.model', '=', 'project.task')]</field>
        <field name="context">{'default_res_model': 'project.task'}</field>
    </record>
    <menuitem id="project_menu_config_activity_type"
        action="mail_activity_type_action_config_project_types"
        parent="menu_project_config"/>
</odoo>
```

## File: views\project_assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="assets_backend" name="project assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <link rel="stylesheet" href="/project/static/src/css/project.css"/>
                <script type="text/javascript" src="/project/static/src/js/project_kanban.js"></script>
                <script type="text/javascript" src="/project/static/src/js/project_rating_reporting.js"></script>
                <script type="text/javascript" src="/project/static/src/js/project_task_kanban_examples.js"></script>
                <script type="text/javascript" src="/project/static/src/js/tours/project.js"></script>
                <script type="text/javascript" src="/project/static/src/js/project_calendar.js"></script>
                <link rel="stylesheet" type="text/scss" href="/project/static/src/scss/project_dashboard.scss"/>
            </xpath>
        </template>

        <template id="assets_frontend" inherit_id="web.assets_frontend" name="Rating Project Assets">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/project/static/src/scss/portal_rating.scss"/>
                <script type="text/javascript" src="/project/static/src/js/portal_rating.js"/>
            </xpath>
        </template>
</odoo>

```

## File: views\project_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="portal_layout" name="Portal layout: project menu entry" inherit_id="portal.portal_breadcrumbs" priority="40">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <li t-if="page_name == 'project' or project" class="col-lg-2" t-attf-class="breadcrumb-item #{'active ' if not project else ''}">
                <a t-if="project" t-attf-href="/my/projects?{{ keep_query() }}">Projects</a>
                <t t-else="">Projects</t>
            </li>
            <li t-if="project" class="breadcrumb-item active text-truncate col-8 col-lg-10">
                <t t-esc="project.name"/>
            </li>
            <li t-if="page_name == 'task' or task" t-attf-class="breadcrumb-item #{'active ' if not task else ''}">
                <a t-if="task" t-attf-href="/my/tasks?{{ keep_query() }}">Tasks</a>
                <t t-else="">Tasks</t>
            </li>
            <li t-if="task" class="breadcrumb-item active col-8 col-lg-10 text-truncate">
                <span t-field="task.name"/>
            </li>
        </xpath>
    </template>

    <template id="portal_my_home" name="Show Projects / Tasks" customize_show="True" inherit_id="portal.portal_my_home" priority="40">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="title">Projects</t>
                <t t-set="url" t-value="'/my/projects'"/>
                <t t-set="placeholder_count" t-value="'project_count'"/>
            </t>
            <t t-call="portal.portal_docs_entry">
                <t t-set="title">Tasks</t>
                <t t-set="url" t-value="'/my/tasks'"/>
                <t t-set="placeholder_count" t-value="'task_count'"/>
            </t>
        </xpath>
    </template>

    <template id="portal_my_projects" name="My Projects">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Projects</t>
            </t>
            <t t-if="not projects">
                <div class="alert alert-warning mt8" role="alert">
                    There are no projects.
                </div>
            </t>
            <t t-if="projects" t-call="portal.portal_table">
                <tbody>
                    <tr t-foreach="projects" t-as="project">
                        <td>
                            <a t-attf-href="/my/project/#{project.id}?{{ keep_query() }}"><span t-field="project.name"/></a>
                        </td>
                        <td class="text-right">
                            <a t-attf-href="/my/tasks?{{keep_query(filterby=project.id)}}">
                                <t t-esc="project.task_count" />
                                <t t-esc="project.label_tasks" />
                            </a>
                        </td>
                    </tr>
                </tbody>
            </t>
        </t>
    </template>

    <template id="portal_my_project" name="My Project">
        <t t-call="portal.portal_layout">
            <t t-set="o_portal_fullwidth_alert" groups="project.group_project_user">
                <t t-call="portal.portal_back_in_edit_mode">
                    <t t-set="backend_url" t-value="'/web#model=project.project&amp;id=%s&amp;view_type=form' % (project.id)"/>
                </t>
            </t>

            <t t-call="portal.portal_record_layout">
                <t t-set="card_header">
                    <h5 class="mb-0">
                        <span t-field="project.name" class="text-truncate"/>
                        <span class="float-right">
                            <a role="button" t-attf-href="/my/tasks?filterby=#{project.id}" class="btn btn-sm btn-secondary">
                                <span class="fa fa-tasks" role="img" aria-label="Tasks" title="Tasks"/>
                                <span t-esc="project.task_count" />
                                <span t-field="project.label_tasks" />
                            </a>
                        </span>
                    </h5>
                </t>
                <t t-set="card_body">
                    <div class="row">
                        <div t-if="project.partner_id" class="col-12 col-md-6 mb-2 mb-md-0">
                            <h6>Customer</h6>
                            <div class="row">
                                <div class="col flex-grow-0 pr-3">
                                    <img t-if="project.partner_id.image_1024" class="rounded-circle mt-1 o_portal_contact_img" t-att-src="image_data_uri(project.partner_id.image_1024)" alt="Contact"/>
                                    <img t-else="" class="rounded-circle mt-1 o_portal_contact_img" src="/web/static/src/img/user_menu_avatar.png" alt="Contact"/>
                                </div>
                                <div class="col pl-sm-0">
                                    <address t-field="project.partner_id" t-options='{"widget": "contact", "fields": ["name", "email", "phone"]}'/>
                                </div>
                            </div>
                        </div>
                        <div t-if="project.user_id" class="col-12 col-md-6">
                            <h6>Project Manager</h6>
                            <div class="row">
                                <div class="col flex-grow-0 pr-3">
                                    <img t-if="project.user_id.image_1024" class="rounded-circle mt-1 o_portal_contact_img" t-att-src="image_data_uri(project.user_id.image_1024)" alt="Contact"/>
                                    <img t-else="" class="rounded-circle mt-1 o_portal_contact_img" src="/web/static/src/img/user_menu_avatar.png" alt="Contact"/>
                                </div>
                                <div class="col pl-sm-0">
                                    <address t-field="project.user_id" t-options='{"widget": "contact", "fields": ["name", "email", "phone"]}'/>
                                </div>
                            </div>
                        </div>
                    </div>
                </t>
            </t>
        </t>
    </template>

    <template id="portal_my_tasks" name="My Tasks">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Tasks</t>
            </t>
            <t t-if="not grouped_tasks">
                <div class="alert alert-warning mt8" role="alert">
                    There are no tasks.
                </div>
            </t>
            <t t-if="grouped_tasks">
                <t t-call="portal.portal_table">
                    <t t-foreach="grouped_tasks" t-as="tasks">
                        <thead>
                            <tr t-attf-class="{{'thead-light' if not groupby == 'none' else ''}}">
                                <th class="text-left">Ref</th>
                                <th t-if="groupby == 'none'">Name</th>
                                <th t-if="groupby == 'project'">
                                    <em class="font-weight-normal text-muted"><span t-field="tasks[0].sudo().project_id.label_tasks"/> for project:</em>
                                    <span t-field="tasks[0].sudo().project_id.name"/></th>
                                <th t-if="groupby == 'stage'">
                                    <em class="font-weight-normal text-muted"><span t-field="tasks[0].sudo().project_id.label_tasks"/> in stage:</em>
                                    <span class="text-truncate" t-field="tasks[0].sudo().stage_id.name"/></th>
                                <th t-if="groupby != 'project'" class="text-center">Project</th>
                                <th t-if="groupby != 'stage'" class="text-center">Stage</th>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="tasks" t-as="task">
                                <tr>
                                    <td class="text-left">
                                        #<span t-esc="task.id"/>
                                    </td>
                                    <td>
                                        <a t-attf-href="/my/task/#{task.id}?{{ keep_query() }}"><span t-field="task.name"/></a>
                                    </td>
                                    <td t-if="groupby != 'project'" class="text-center">
                                        <span class="badge badge-pill badge-info" title="Current project of the task" t-esc="task.project_id.name" />
                                    </td>
                                    <td t-if="groupby != 'stage'" class="text-center">
                                        <span class="badge badge-pill badge-info" title="Current stage of the task" t-esc="task.stage_id.name" />
                                    </td>
                                </tr>
                            </t>
                        </tbody>
                    </t>
                </t>
            </t>
        </t>
    </template>

    <template id="portal_my_task" name="My Task">
        <t t-call="portal.portal_layout">
            <t t-set="o_portal_fullwidth_alert" groups="project.group_project_user">
                <t t-call="portal.portal_back_in_edit_mode">
                    <t t-set="backend_url" t-value="'/web#model=project.task&amp;id=%s&amp;view_type=form' % (task.id)"/>
                </t>
            </t>

            <t t-call="portal.portal_record_layout">
                <t t-set="card_header">
                    <div class="row no-gutters">
                        <div class="col-12">
                            <h5 class="d-flex mb-1 mb-md-0 row">
                                <span t-field="task.name" class="col-9 text-truncate"/>
                                <small class="text-muted d-none d-md-inline"> (#<span t-field="task.id"/>)</small>
                                <div class="col-3 col-md-2 text-right">
                                    <small class="text-right">Status:</small>
                                    <span t-field="task.stage_id.name" class=" badge badge-pill badge-info" title="Current stage of this task"/>
                                </div>
                            </h5>
                        </div>
                    </div>
                </t>
                <t t-set="card_body">
                    <div class="mb-1" t-if="user in task.sudo().project_id.allowed_user_ids">
                        <strong>Project:</strong> <a t-attf-href="/my/project/#{task.project_id.id}" t-field="task.project_id.name"/>
                    </div>

                    <div class="row mb-4">
                        <div class="col-12 col-md-6 mb-1">
                            <strong>Date:</strong> <span t-field="task.create_date" t-options='{"widget": "date"}'/>
                        </div>
                        <div class="col-12 col-md-6" t-if="task.date_deadline">
                            <strong>Deadline:</strong> <span t-field="task.date_deadline" t-options='{"widget": "date"}'/>
                        </div>
                    </div>

                    <div class="row mb-4" t-if="task.user_id or task.partner_id">
                        <div class="col-12 col-md-6 pb-2" t-if="task.user_id">
                            <strong>Assigned to</strong>
                            <div class="row">
                                <div class="col flex-grow-0 pr-3">
                                    <img t-if="task.user_id.image_1024" class="rounded-circle mt-1 o_portal_contact_img" t-att-src="image_data_uri(task.user_id.image_1024)" alt="Contact"/>
                                    <img t-else="" class="rounded-circle mt-1 o_portal_contact_img" src="/web/static/src/img/user_menu_avatar.png" alt="Contact"/>
                                </div>
                                <div class="col pl-md-0">
                                    <div t-field="task.user_id" t-options='{"widget": "contact", "fields": ["name", "email", "phone"]}'/>
                                </div>
                            </div>
                        </div>
                        <div class="coll-12 col-md-6 pb-2" t-if="task.partner_id">
                            <strong>Reported by</strong>
                            <div class="row">
                                <div class="col flex-grow-0 pr-3">
                                    <img t-if="task.partner_id.image_1024" class="rounded-circle mt-1 o_portal_contact_img" t-att-src="image_data_uri(task.partner_id.image_1024)" alt="Contact"/>
                                    <img t-else="" class="rounded-circle mt-1 o_portal_contact_img" src="/web/static/src/img/user_menu_avatar.png" alt="Contact"/>
                                </div>
                                <div class="col pl-md-0">
                                    <div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["name", "email", "phone"]}'/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="row" t-if="task.description or task.attachment_ids">
                        <div t-if="task.description" t-attf-class="col-12 col-lg-7 mb-4 mb-md-0 {{'col-lg-7' if task.attachment_ids else 'col-lg-12'}}">
                            <hr class="mb-1"/>
                            <div class="d-flex my-2">
                                <strong>Description</strong>
                            </div>
                            <div class="py-1 px-2 bg-100 small" t-field="task.description"/>
                        </div>
                        <div t-if="task.attachment_ids" t-attf-class="col-12 col-lg-5 o_project_portal_attachments {{'col-lg-5' if task.description else 'col-lg-12'}}">
                            <hr class="mb-1 d-none d-lg-block"/>
                            <strong class="d-block mb-2">Attachments</strong>
                            <div class="row">
                                <div t-attf-class="col {{'col-lg-6' if not task.description else 'col-lg-12'}}">
                                    <ul class="list-group">
                                        <a class="list-group-item list-group-item-action d-flex align-items-center oe_attachments py-1 px-2" t-foreach='task.attachment_ids' t-as='attachment' t-attf-href="/web/content/#{attachment.id}?download=true&amp;access_token=#{attachment.access_token}" target="_blank" data-no-post-process="">
                                            <div class='oe_attachment_embedded o_image o_image_small mr-2 mr-lg-3' t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype" t-attf-data-src="/web/image/#{attachment.id}/50x40?access_token=#{attachment.access_token}"/>
                                            <div class='oe_attachment_name text-truncate'><t t-esc='attachment.name'/></div>
                                        </a>
                                    </ul>
                                </div>
                            </div>
                        </div>
                    </div>
                </t>
            </t>

            <div class="mt32">
                <h4><strong>Message and communication history</strong></h4>
                <t t-call="portal.message_thread">
                    <t t-set="object" t-value="task"/>
                    <t t-set="token" t-value="task.access_token"/>
                    <t t-set="pid" t-value="pid"/>
                    <t t-set="hash" t-value="hash"/>
                </t>
            </div>
        </t>
    </template>
</odoo>

```

## File: views\project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Top menu item -->
        <menuitem name="Project"
            id="menu_main_pm"
            groups="group_project_manager,group_project_user"
            web_icon="project,static/description/icon.png"
            sequence="50"/>

        <menuitem id="menu_project_config" name="Configuration" parent="menu_main_pm"
            sequence="100" groups="project.group_project_manager"/>

        <record id="view_task_search_form" model="ir.ui.view">
            <field name="name">project.task.search.form</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
               <search string="Tasks">
                    <field name="name" string="Task"/>
                    <field name="tag_ids"/>
                    <field name="user_id"/>
                    <field name="partner_id" operator="child_of"/>
                    <field name="stage_id"/>
                    <field name="project_id"/>
                    <field name="parent_id" groups="project.group_subtask_project"/>
                    <filter string="My Tasks" name="my_tasks" domain="[('user_id', '=', uid)]"/>
                    <filter string="Followed Tasks" name="my_followed_tasks" domain="[('message_is_follower', '=', True)]" />
                    <filter string="Unassigned" name="unassigned" domain="[('user_id', '=', False)]"/>
                    <separator/>
                    <filter string="Starred" name="starred" domain="[('priority', 'in', [1, 2])]"/>
                    <filter string="Late Tasks" name="late" domain="[('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter string="Rated tasks" name="rating_task" domain="[('rating_last_value', '!=', 0.0)]" groups="project.group_project_rating"/>
                    <separator/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}"/>
                        <filter string="Assigned to" name="user" context="{'group_by': 'user_id'}"/>
                        <filter string="Project" name="project" context="{'group_by': 'project_id'}"/>
                        <filter string="Creation Date" name="group_create_date" context="{'group_by': 'create_date'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="act_project_project_2_project_task_all" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form,calendar,pivot,graph,activity</field>
            <field name="domain">[('project_id', '=', active_id)]</field>
            <field name="context">{
                'pivot_row_groupby': ['user_id'],
                'default_project_id': active_id,
            }</field>
            <field name="search_view_id" ref="view_task_search_form"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p><p>
                    To get things done, use activities and status on tasks.<br/>
                    Chat in real time or by email to collaborate efficiently.
                </p>
            </field>
        </record>

        <record id="project_task_action_sub_task" model="ir.actions.act_window">
            <field name="name">Sub-tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,kanban,form,calendar,pivot,graph,activity</field>
            <field name="search_view_id" ref="project.view_task_search_form"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p><p>
                    To get things done, use activities and status on tasks.<br/>
                    Chat in real time or by email to collaborate efficiently.
                </p>
            </field>
        </record>

        <!-- Task types -->
        <record id="task_type_search" model="ir.ui.view">
            <field name="name">project.task.type.search</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <search string="Tasks Stages">
                   <field name="name" string="Tasks Stages"/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <record id="task_type_edit" model="ir.ui.view">
            <field name="name">project.task.type.form</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <form string="Task Stage" delete="0">
                    <field name="active" invisible="1" />
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}" />
                        <group>
                            <group>
                                <field name="name"/>
                                <field name="mail_template_id"/>
                                <field name="rating_template_id" groups="project.group_project_rating"/>
                                <div class="alert alert-warning" role="alert" colspan='2' attrs="{'invisible': ['|', ('rating_template_id','=', False), ('disabled_rating_warning', '=', False)]}">
                                    <i class="fa fa-warning" title="Customer disabled on projects"/><b> Customer Ratings</b> are disabled on the following project(s) : <br/>
                                    <field name="disabled_rating_warning" class="mb-0" />
                                </div>
                                <field name="auto_validation_kanban_state" attrs="{'invisible': [('rating_template_id','=', False)]}" groups="project.group_project_rating"/>
                            </group>
                            <group>
                                <field name="fold"/>
                                <field name="is_closed" groups="base.group_no_one"/>
                                <field name="project_ids" widget="many2many_tags" groups="base.group_no_one"/>
                                <field name="sequence" groups="base.group_no_one"/>
                            </group>
                        </group>
                        <group string="Stage Description and Tooltips">
                            <p class="text-muted" colspan="2">
                                At each stage employees can block or make task/issue ready for next stage.
                                You can define here labels that will be displayed for the state instead
                                of the default labels.
                            </p>
                            <label for="legend_normal" string=" " class="o_status oe_project_kanban_legend"
                                title="Task in progress. Click to block or set as done."
                                aria-label="Task in progress. Click to block or set as done." role="img"/>
                            <field name="legend_normal" nolabel="1"/>
                            <label for="legend_blocked" string=" " class="o_status o_status_red oe_project_kanban_legend"
                                title="Task is blocked. Click to unblock or set as done."
                                aria-label="Task is blocked. Click to unblock or set as done." role="img"/>
                            <field name="legend_blocked" nolabel="1"/>
                            <label for="legend_done" string=" " class="o_status o_status_green oe_project_kanban_legend"
                                title="This step is done. Click to block or set in progress."
                                aria-label="This step is done. Click to block or set in progress." role="img"/>
                            <field name="legend_done" nolabel="1"/>

                            <p class="text-muted" colspan="2">
                                You can also add a description to help your coworkers understand the meaning and purpose of the stage.
                            </p>
                            <field name="description" placeholder="Add a description..." nolabel="1" colspan="2"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="task_type_tree" model="ir.ui.view">
            <field name="name">project.task.type.tree</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <tree string="Task Stage" delete="0" sample="1">
                    <field name="sequence" widget="handle" optional="show"/>
                    <field name="name"/>
                    <field name="fold" optional="show"/>
                    <field name="mail_template_id" optional="show"/>
                    <field name="rating_template_id" optional="show"/>
                    <field name="auto_validation_kanban_state" optional="hide" />
                    <field name="description" optional="hide"/>
                    <field name="project_ids" optional="show" widget="many2many_tags" options="{'color_field': 'color'}" />
                </tree>
            </field>
        </record>

        <record id="view_project_task_type_kanban" model="ir.ui.view">
            <field name="name">project.task.type.kanban</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1">
                    <field name="name"/>
                    <field name="fold"/>
                    <field name="description"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-12">
                                        <strong><t t-esc="record.name.value"/></strong>
                                    </div>
                                </div>
                                <t t-if="record.description.value">
                                    <hr class="mt8 mb8"/>
                                    <t t-esc="record.description.value"/>
                                </t>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="open_task_type_form" model="ir.actions.act_window">
            <field name="name">Stages</field>
            <field name="res_model">project.task.type</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_id" ref="task_type_tree"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new stage in the task pipeline
              </p><p>
                Define the steps that will be used in the project from the
                creation of the task, up to the closing of the task or issue.
                You will use these stages in order to track the progress in
                solving a task or an issue.
              </p>
            </field>
        </record>

        <record id="open_task_type_form_domain" model="ir.actions.act_window">
            <field name="name">Stages</field>
            <field name="res_model">project.task.type</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="domain">[('project_ids','=', project_id)]</field>
            <field name="view_id" ref="task_type_tree"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                Create a new stage in the task pipeline
                </p><p>
                Define the steps that will be used in the project from the
                creation of the task, up to the closing of the task or issue.
                You will use these stages in order to track the progress in
                solving a task or an issue.
                </p>
            </field>
        </record>

        <record id="unlink_task_type_action" model="ir.actions.server">
            <field name="name">Delete</field>
            <field name="model_id" ref="project.model_project_task_type"/>
            <field name="binding_model_id" ref="project.model_project_task_type"/>
            <field name="binding_view_types">form,list</field>
            <field name="state">code</field>
            <field name="code">action = records.unlink_wizard(stage_view=True)</field>
        </record>

        <!-- Project -->
        <record id="edit_project" model="ir.ui.view">
            <field name="name">project.project.form</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <form string="Project" delete="0">
                    <header>
                        <button name="%(portal.portal_share_action)d" string="Share" type="action" class="oe_highlight oe_read_only"/>
                    </header>
                <sheet string="Project">
                    <div class="oe_button_box" name="button_box" groups="base.group_user">
                        <button  class="oe_stat_button" name="attachment_tree_view" type="object" icon="fa-file-text-o">
                            <field string="Documents" name="doc_count" widget="statinfo"/>
                        </button>
                        <button class="oe_stat_button" type="action"
                            name="%(act_project_project_2_project_task_all)d" icon="fa-tasks">
                            <field string="Tasks In Progress" name="task_count" widget="statinfo" options="{'label_field': 'label_tasks'}"/>
                        </button>
                        <button name="action_view_all_rating" type="object" attrs="{'invisible': ['|', ('rating_active', '=', False), ('rating_percentage_satisfaction', '=', -1)]}" class="oe_stat_button oe_percent oe_read_only" icon="fa-smile-o" groups="project.group_project_rating">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="rating_percentage_satisfaction" nolabel="1"/>
                                    %
                                </span>
                                <span class="o_stat_text">
                                    Customer Satisfaction
                                </span>
                            </div>
                        </button>                            
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <h1>
                            <field name="name" placeholder="Project Name"/>
                        </h1>
                        <div name="options_active">
                            <div>
                                <label for="label_tasks" class="oe_inline" string="Name of the tasks:"/>
                                <field name="label_tasks" class="oe_inline oe_input_align"/>
                            </div>
                        </div>
                    </div>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" nolabel="1" placeholder="Describe your project..." type="html"/>
                            <div class="d-none oe_clear"/>
                        </page>
                        <page name="settings" string="Settings">
                            <group>
                                <group>
                                    <field name="active" invisible="1"/>
                                    <field name="user_id" string="Project Manager" widget="many2one_avatar_user" attrs="{'readonly':[('active','=',False)]}" domain="[('share', '=', False)]"/>
                                    <field name="partner_id" string="Customer" widget="res_partner_many2one"/>
                                    <field name="partner_phone" widget="phone"
                                           attrs="{'invisible': [('partner_id', '=', False)]}"/>
                                    <field name="partner_email" widget="email"
                                           attrs="{'invisible': [('partner_id', '=', False)]}"/>
                                    <div name="alias_def" colspan="2" attrs="{'invisible': [('alias_domain', '=', False)]}">
                                        <!-- Always display the whole alias in edit mode. It depends in read only -->
                                        <label for="alias_enabled" class="font-weight-bold" string="Create tasks by sending an email"/>
                                        <field name="alias_enabled" invisible="1"/>
                                        <span>
                                            <span class="font-weight-bold oe_read_only" attrs="{'invisible':  [('alias_name', '=', False)]}">to </span>
                                            <span class="font-weight-bold oe_edit_only">to </span>
                                            <field name="alias_name" class="oe_inline"/>
                                            <span class="oe_read_only" attrs="{'invisible':  [('alias_name', '=', False)]}">@</span>
                                            <span class="oe_edit_only">@</span>
                                            <field name="alias_domain"
                                                   attrs="{'invisible':  [('alias_name', '=', False)]}"
                                                   class="oe_inline oe_read_only"
                                                   readonly="1"/>
                                            <field name="alias_domain"
                                                   class="oe_inline oe_edit_only"
                                                   readonly="1"/>
                                        </span>
                                    </div>
                                    <!-- the alias contact must appear when the user start typing and it must disappear
                                        when the string is deleted. -->
                                    <field name="alias_contact" class="oe_inline oe_edit_only" string="Accept Emails From"
                                           attrs="{'invisible': ['|', ('alias_name', '=', ''), ('alias_name', '=', False)]}"/>
                                </group>
                                <group>
                                    <field name="analytic_account_id" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]" context="{'default_partner_id': partner_id}" groups="analytic.group_analytic_accounting"/>
                                    <field name="privacy_visibility" widget="radio"/>
                                    <field name="allowed_internal_user_ids" widget="many2many_tags" attrs="{'invisible': [('privacy_visibility', '!=', 'followers')]}"/>
                                    <field name="allowed_portal_user_ids" widget="many2many_tags" options="{'no_create': True}" attrs="{'invisible': [('privacy_visibility', '!=', 'portal')]}"/>

                                    <field name="company_id" groups="base.group_multi_company"/>
                                </group>
                                <group name="extra_settings">
                                </group>
                            </group>
                            <div class="row mt16 o_settings_container">
                                <div id="rating_settings" class="col-lg-6 o_setting_box" groups="project.group_project_rating">
                                    <div class="o_setting_left_pane">
                                        <field name="rating_active"/>
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label for="rating_active" />
                                        <div class="text-muted">
                                            Get customer feedback
                                        </div>
                                        <div class="mt16" attrs="{'invisible':[('rating_active','==',False)]}">
                                            <field name="rating_status" widget="radio" />
                                            <div  attrs="{'required': [('rating_status','=','periodic')], 'invisible': [('rating_status','!=','periodic')]}">
                                                <label for="rating_status_period"/>
                                                <field name="rating_status_period"/>
                                            </div>
                                            <div class="content-group">
                                                    <div class="mt8">
                                                        <button name="%(project.open_task_type_form_domain)d" context="{'project_id':id}" icon="fa-arrow-right" type="action" string="Set a Rating Email Template on Stages" class="btn-link"/>
                                                    </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-lg-6 o_setting_box"  id="subtask_settings" groups="project.group_subtask_project">
                                    <div class="o_setting_left_pane">
                                        <field name="allow_subtasks" />
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label for="allow_subtasks" />
                                        <div class="text-muted">
                                            Split your tasks to organize your work into sub-milestones
                                        </div>
                                        <div class="mt8" attrs="{'invisible': [('allow_subtasks', '=', False)]}">
                                            <label for="subtask_project_id" />
                                            <field name="subtask_project_id" />
                                        </div>
                                    </div>
                                </div>
                                <div class="col-lg-6 o_setting_box"  id="recurring_tasks_setting" groups="project.group_project_recurring_tasks">
                                    <div class="o_setting_left_pane">
                                        <field name="allow_recurring_tasks" />
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label for="allow_recurring_tasks" />
                                        <div class="text-muted">
                                            Auto-generate tasks for regular activities
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </page>
                    </notebook>

                    <div class="oe_chatter">
                        <field name="message_follower_ids" options="{'post_refresh':True}" help="Follow this project to automatically track the events associated to tasks and issues of this project." groups="base.group_user"/>
                    </div>
                </sheet>
                </form>
            </field>
        </record>

        <record id="view_project_project_filter" model="ir.ui.view">
            <field name="name">project.project.select</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <search string="Search Project">
                    <field name="name" string="Project"/>
                    <field name="user_id" string="Project Manager"/>
                    <field name="partner_id" string="Customer" filter_domain="[('partner_id', 'child_of', self)]"/>
                    <filter string="My Favorites" name="my_projects" domain="[('favorite_user_ids', 'in', uid)]"/>
                    <separator/>
                    <filter string="Followed" name="followed_by_me" domain="[('message_is_follower', '=', True)]"/>
                    <filter string="My Projects" name="own_projects" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Project Manager" name="Manager" context="{'group_by': 'user_id'}"/>
                        <filter string="Customer" name="Partner" context="{'group_by': 'partner_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="view_project" model="ir.ui.view">
            <field name="name">project.project.tree</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <tree decoration-muted="active == False" string="Projects" delete="0" multi_edit="1" sample="1">
                    <field name="sequence" optional="show" widget="handle"/>
                    <field name="message_needaction" invisible="1"/>
                    <field name="active" invisible="1"/>
                    <field name="name" string="Name" class="font-weight-bold"/>
                    <field name="user_id" optional="show" string="Project Manager" widget="many2one_avatar_user"/>
                    <field name="partner_id" optional="show" string="Customer"/>
                    <field name="analytic_account_id" optional="hide"/>
                    <field name="privacy_visibility" optional="hide"/>
                    <field name="subtask_project_id" optional="hide"/>
                    <field name="label_tasks" optional="hide"/>
                    <field name="company_id" optional="show"  groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="project_project_view_form_simplified" model="ir.ui.view">
            <field name="name">project.project.view.form.simplified</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <form string="Project">
                    <group>
                        <field name="name" class="o_project_name oe_inline"
                            string="Project Name" placeholder="e.g. Office Party"/>
                        <field name="user_id" invisible="1"/>
                        <field name="allow_recurring_tasks" groups="project.group_project_recurring_tasks" />
                    </group>
                    <div name="alias_def" colspan="2" attrs="{'invisible': [('alias_domain', '=', False)]}">
                        <label for="alias_name" class="oe_inline" string="Create tasks by sending an email to"/>
                        <field name="alias_enabled" invisible="1"/>
                        <span>
                            <field name="alias_name" class="oe_inline" placeholder="e.g. office-party"/>@<field name="alias_domain" class="oe_inline" readonly="1" />
                        </span>
                    </div>
                </form>
            </field>
        </record>

        <record id="project_project_view_form_simplified_footer" model="ir.ui.view">
            <field name="name">project.project.view.form.simplified</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.project_project_view_form_simplified"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//group" position="after">
                    <footer>
                        <button string="Create" name="action_view_tasks" type="object" class="btn-primary o_open_tasks"/>
                        <button string="Discard" class="btn-secondary" special="cancel"/>
                    </footer>
                </xpath>
            </field>
        </record>

        <record id="open_create_project" model="ir.actions.act_window">
            <field name="name">Create a Project</field>
            <field name="res_model">project.project</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="project_project_view_form_simplified_footer"/>
            <field name="target">new</field>
        </record>

        <record model="ir.ui.view" id="view_project_kanban">
            <field name="name">project.project.kanban</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <kanban class="oe_background_grey o_kanban_dashboard o_project_kanban o_emphasize_colors" on_create="project.open_create_project" sample="1">
                    <field name="name"/>
                    <field name="partner_id"/>
                    <field name="color"/>
                    <field name="task_count"/>
                    <field name="label_tasks"/>
                    <field name="alias_id"/>
                    <field name="alias_name"/>
                    <field name="alias_domain"/>
                    <field name="is_favorite"/>
                    <field name="rating_percentage_satisfaction"/>
                    <field name="rating_status"/>
                    <field name="rating_active" />
                    <field name="analytic_account_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{kanban_color(record.color.raw_value)} oe_kanban_global_click o_has_icon">
                                <div class="o_project_kanban_main">
                                    <div class="o_kanban_card_content">
                                        <div class="o_kanban_primary_left">
                                            <div class="o_primary">
                                                <span><t t-esc="record.name.value"/></span>
                                                <span t-if="record.partner_id.value">
                                                    <strong><t t-esc="record.partner_id.value"/></strong>
                                                </span>
                                            </div>
                                            <div t-if="record.alias_name.value and record.alias_domain.value">
                                                <span><i class="fa fa-envelope" role="img" aria-label="Domain Alias" title="Domain Alias"></i> <a t-att-href="'mailto:' + record.alias_id.value" title="Send an email"><t t-esc="record.alias_id.value"/></a></span>
                                            </div>
                                            <div t-if="record.rating_active.raw_value" class="mt8 text-primary" title="Percentage of happy ratings over the past 30 days." groups="project.group_project_rating">
                                                <b>
                                                    <a name="action_view_all_rating" type="object" context="{'search_default_rating_last_30_days':1}">
                                                        <t t-if="record.rating_percentage_satisfaction.value != -1">
                                                            <i class="fa fa-smile-o" role="img" aria-label="Percentage of satisfaction" title="Percentage of satisfaction"/> <t t-esc="record.rating_percentage_satisfaction.value"/>%
                                                        </t>
                                                    </a>
                                                </b>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="o_kanban_card_manage_pane dropdown-menu" groups="project.group_project_manager" role="menu">
                                        <div class="o_kanban_card_manage_section o_kanban_manage_reports">
                                            <div role="menuitem">
                                                <a name="%(portal.portal_share_action)d" type="action">Share</a>
                                            </div>
                                            <div role="menuitem">
                                                <a type="edit">Edit</a>
                                            </div>
                                        </div>
                                        <div role="menuitem" aria-haspopup="true" class="o_no_padding_kanban_colorpicker">
                                            <ul class="oe_kanban_colorpicker" data-field="color" role="popup"/>
                                        </div>
                                    </div>
                                    <a class="o_kanban_manage_toggle_button o_left" href="#" groups="project.group_project_manager"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
                                    <span class="o_right"><field name="is_favorite" widget="boolean_favorite" nolabel="1" force_save="1" /></span>
                                </div>

                                <div class="o_project_kanban_boxes">
                                    <a class="o_project_kanban_box" name="action_view_tasks" type="object">
                                        <div>
                                            <span class="o_value"><t t-esc="record.task_count.value"/></span>
                                            <span class="o_label"><t t-esc="record.label_tasks.value"/></span>
                                        </div>
                                    </a>
                                    <a t-if="record.analytic_account_id.raw_value" class="o_project_kanban_box o_project_timesheet_box" name="action_view_account_analytic_line" type="object" groups="analytic.group_analytic_accounting">
                                        <div>
                                            <span class="o_label">Profitability</span>
                                        </div>
                                    </a>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="open_view_project_all" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="domain">[]</field>
            <field name="view_mode">kanban,form</field>
            <field name="view_id" ref="view_project_kanban"/>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="target">main</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No projects found. Let's create one!
                </p><p>
                    Projects regroup tasks on the same topic and each have their own dashboard.
                </p>
            </field>
        </record>

        <record id="open_view_project_all_config" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="domain">[]</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('view_project')}),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('view_project_kanban')})]"/>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="context">{}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                   No projects found. Let's create one!
                </p><p>
                    Projects regroup tasks on the same topic and each have their own dashboard.
                </p>
            </field>
        </record>

        <!-- Task -->
        <record id="view_task_form2" model="ir.ui.view">
            <field name="name">project.task.form</field>
            <field name="model">project.task</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <form string="Task" class="o_form_project_tasks">
                    <field name="allow_subtasks" invisible="1" />
                    <field name="is_closed" invisible="1" />
                    <field name="allow_recurring_tasks" invisible="1" />
                    <field name="repeat_show_dow" invisible="1" />
                    <field name="repeat_show_day" invisible="1" />
                    <field name="repeat_show_week" invisible="1" />
                    <field name="repeat_show_month" invisible="1" />
                    <field name="recurrence_id" invisible="1" />
                    <header>
                        <button name="action_assign_to_me" string="Assign to Me" type="object" class="oe_highlight"
                            attrs="{'invisible' : [('user_id', '!=', False)]}"/>
                        <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" attrs="{'invisible': [('project_id', '=', False)]}"/>
                    </header>
                    <div class="text-center alert alert-primary" role="alert" attrs="{'invisible': ['|', ('ribbon_message', '=', False), ('ribbon_message', '=', '')]}">
                        <field name="ribbon_message"/>
                    </div>
                    <div class="alert alert-info oe_edit_only" role="status" attrs="{'invisible': ['|', ('recurring_task', '=', False), ('recurrence_id', '=', False)]}">
                        <p>Edit recurring task</p>
                        <field name="recurrence_update" widget="radio"/>
                    </div>
                    <sheet string="Task">
                    <div class="oe_button_box" name="button_box">
                        <button name="action_subtask" type="object" class="oe_stat_button" icon="fa-tasks"
                            attrs="{'invisible' : ['|', ('allow_subtasks', '=', False), ('id', '=', False)]}" context="{'default_user_id': user_id, 'default_parent_id': id, 'default_project_id': subtask_project_id}">
                            <field string="Sub-tasks" name="subtask_count" widget="statinfo"/>
                        </button>
                        <button name="%(rating_rating_action_task)d" type="action" attrs="{'invisible': [('rating_count', '=', 0)]}" class="oe_stat_button" icon="fa-smile-o" groups="project.group_project_rating">
                            <field name="rating_count" string="Rating" widget="statinfo"/>
                        </button>
                        <button name="action_recurring_tasks" type="object" attrs="{'invisible': [('recurrence_id', '=', False)]}" class="oe_stat_button" icon="fa-repeat" groups="project.group_project_recurring_tasks">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="recurring_count" widget="statinfo" nolabel="1" />
                                    Tasks
                                </span>
                                <span class="o_stat_text">in Recurrence</span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title pr-0">
                        <h1 class="d-flex flex-row justify-content-between">
                            <field name="priority" widget="priority" class="mr-3"/>
                            <field name="name" class="o_task_name text-truncate" placeholder="Task Title..." default_focus="1" />
                            <field name="kanban_state" widget="state_selection" class="ml-auto"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="project_id" required="1" domain="[('active', '=', True), ('company_id', '=', company_id)]"/>
                            <field name="user_id"
                                class="o_task_user_field"
                                domain="[('share', '=', False)]"/>
                            <field
                                name="parent_id"
                                domain="[('parent_id', '=', False)]"
                                attrs="{'invisible' : [('allow_subtasks', '=', False)]}"
                            />
                            <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                            <field name="recurring_task" attrs="{'invisible': [('allow_recurring_tasks', '=', False)]}" />
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="partner_id" widget="res_partner_many2one" class="o_task_customer_field"/>
                            <field name="partner_email" widget="email" invisible="1"/>
                            <field name="partner_phone" widget="phone" attrs="{'invisible': True}"/>
                            <field name="legend_blocked" invisible="1"/>
                            <field name="legend_normal" invisible="1"/>
                            <field name="legend_done" invisible="1"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" type="html"/>
                            <div class="d-none oe_clear"/>
                        </page>
                        <page name="recurrence" string="Recurrence" attrs="{'invisible': [('recurring_task', '=', False)]}">
                            <group>
                                <group>
                                    <label for="repeat_interval" />
                                    <div class="o_row">
                                        <field name="repeat_interval" attrs="{'required': [('recurring_task', '=', True)]}" />
                                        <field name="repeat_unit" attrs="{'required': [('recurring_task', '=', True)]}" />
                                    </div>

                                    <label for="repeat_on_month" string="Repeat On" attrs="{'invisible': [('repeat_unit', 'not in', ('month', 'year'))]}" />
                                    <div class="o_row">
                                        <field name="repeat_on_month" attrs="{'invisible': [('repeat_unit', '!=', 'month')], 'required': [('repeat_unit', '=', 'month')]}" />
                                        <field name="repeat_on_year" attrs="{'invisible': [('repeat_unit', '!=', 'year')], 'required': [('repeat_unit', '=', 'year')]}" />

                                        <field name="repeat_day" attrs="{'invisible': [('repeat_show_day', '=', False)], 'required': [('repeat_show_day', '=', True)]}" />
                                        <field name="repeat_week" attrs="{'invisible': [('repeat_show_week', '=', False)], 'required': [('repeat_show_week', '=', True)]}" />
                                        <field name="repeat_weekday" attrs="{'invisible': [('repeat_show_week', '=', False)], 'required': [('repeat_show_week', '=', True)]}" />
                                        <span attrs="{'invisible': ['|', ('repeat_show_week', '=', False), ('repeat_show_month', '=', False)]}">of</span>
                                        <field name="repeat_month" attrs="{'invisible': [('repeat_show_month', '=', False)], 'required': [('repeat_show_month', '=', True)]}" />
                                    </div>


                                    <field name="mon" attrs="{'invisible': [('repeat_show_dow', '=', False)]}" />
                                    <field name="tue" attrs="{'invisible': [('repeat_show_dow', '=', False)]}" />
                                    <field name="wed" attrs="{'invisible': [('repeat_show_dow', '=', False)]}" />
                                    <field name="thu" attrs="{'invisible': [('repeat_show_dow', '=', False)]}" />
                                    <field name="fri" attrs="{'invisible': [('repeat_show_dow', '=', False)]}" />
                                    <field name="sat" attrs="{'invisible': [('repeat_show_dow', '=', False)]}" />
                                    <field name="sun" attrs="{'invisible': [('repeat_show_dow', '=', False)]}" />

                                    <label for="repeat_type" />
                                    <div class="o_row">
                                        <field name="repeat_type" attrs="{'required': [('recurring_task', '=', True)]}" />
                                        <field name="repeat_until" attrs="{'invisible': [('repeat_type', '!=', 'until')], 'required': [('repeat_type', '=', 'until')]}" />
                                        <field name="repeat_number" attrs="{'invisible': [('repeat_type', '!=', 'after')], 'required': [('repeat_type', '=', 'after')]}" />
                                    </div>
                                </group>
                            </group>
                            <group attrs="{'invisible': ['|', ('recurring_task', '=', False), ('recurrence_message', '=', False)]}">
                                <div class="alert alert-success o_form_project_recurrence_message" role="status">
                                    <p><span class="fa fa-check-circle"></span> Next Occurrences:</p>
                                    <field name="recurrence_message" widget="html" class="mb-0" />
                                </div>
                            </group>
                        </page>
                        <page name="extra_info" string="Extra Info" groups="base.group_no_one">
                            <group>
                                <group>
                                    <field name="sequence" groups="base.group_no_one"/>
                                    <field name="email_from" invisible="1"/>
                                    <field name="email_cc" groups="base.group_no_one"/>
                                    <field name="project_privacy_visibility" groups="base.group_no_one"/>
                                    <field name="allowed_user_ids" widget="many2many_tags"
                                           groups="base.group_no_one" attrs="{'invisible': [('project_privacy_visibility', 'not in', ('followers', 'portal'))]}"/>
                                    <field name="child_ids" invisible="1" />
                                    <field name="subtask_project_id" invisible="1" />
                                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                    <field name="displayed_image_id" groups="base.group_no_one"/>
                                </group>
                                <group>
                                    <field name="date_assign" groups="base.group_no_one"/>
                                    <field name="date_last_stage_update" groups="base.group_no_one"/>
                                </group>
                                <group string="Working Time to Assign" attrs="{'invisible': [('working_hours_open', '=', 0.0)]}">
                                    <field name="working_hours_open" string="Hours"/>
                                    <field name="working_days_open" string="Days"/>
                                </group>
                                <group string="Working Time to Close" attrs="{'invisible': [('working_hours_close', '=', 0.0)]}">
                                    <field name="working_hours_close" string="Hours"/>
                                    <field name="working_days_close" string="Days"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" options="{'post_refresh':True}" groups="base.group_user"/>
                        <field name="activity_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="portal_share_action" model="ir.actions.act_window">
            <field name="name">Share</field>
            <field name="res_model">portal.share</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="binding_model_id" ref="model_project_task"/>
            <field name="binding_view_types">form</field>
        </record>

        <record id="unlink_project_action" model="ir.actions.server">
            <field name="name">Delete</field>
            <field name="model_id" ref="project.model_project_project"/>
            <field name="binding_model_id" ref="project.model_project_project"/>
            <field name="binding_view_types">form,list</field>
            <field name="state">code</field>
            <field name="code">action = records.action_unlink()</field>
        </record>

        <record id="quick_create_task_form" model="ir.ui.view">
            <field name="name">project.task.form.quick_create</field>
            <field name="model">project.task</field>
            <field name="priority">1000</field>
            <field name="arch" type="xml">
                <form>
                    <group>
                        <field name="name" string = "Task Title"/>
                        <field name="user_id" options="{'no_open': True,'no_create': True}" domain="[('share', '=', False)]"/>
                        <field name="project_id" required="1" invisible="context.get('all_task', 1)"/>
                        <field name="company_id" invisible="1"/>
                        <field name="parent_id" invisible="1"/>
                        <field name="description" invisible="1"/>
                    </group>
                </form>
            </field>
        </record>

        <!-- Project Task Kanban View -->
        <record model="ir.ui.view" id="view_task_kanban">
            <field name="name">project.task.kanban</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <kanban default_group_by="stage_id" class="o_kanban_small_column o_kanban_project_tasks" on_create="quick_create" quick_create_view="project.quick_create_task_form" examples="project" js_class="project_kanban" sample="1">
                    <field name="color"/>
                    <field name="priority"/>
                    <field name="stage_id" options='{"group_by_tooltip": {"description": "Description"}}'/>
                    <field name="user_id"/>
                    <field name="partner_id"/>
                    <field name="sequence"/>
                    <field name="is_closed"/>
                    <field name="partner_is_company"/>
                    <field name="displayed_image_id"/>
                    <field name="active"/>
                    <field name="legend_blocked"/>
                    <field name="legend_normal"/>
                    <field name="legend_done"/>
                    <field name="activity_ids"/>
                    <field name="activity_state"/>
                    <field name="rating_last_value"/>
                    <field name="rating_ids"/>
                    <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger", "normal": "muted"}'/>
                    <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click">
                            <div class="oe_kanban_content">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title"><field name="name"/></strong>
                                        <span  invisible="context.get('default_project_id', False) or context.get('fsm_mode', False)"><br/><field name="project_id" required="1"/></span>
                                        <br />
                                        <t t-if="record.partner_id.value">
                                            <span t-if="!record.partner_is_company.raw_value">
                                                <field name="commercial_partner_id"/>
                                            </span>
                                            <span t-else="">
                                                <field name="partner_id"/>
                                            </span>
                                        </t>
                                        <t t-else="record.email_from.raw_value"><span><field name="email_from"/></span></t>
                                    </div>
                                    <div class="o_dropdown_kanban dropdown" t-if="!selection_mode" groups="base.group_user">
                                        <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                            <span class="fa fa-ellipsis-v"/>
                                        </a>
                                        <div class="dropdown-menu" role="menu">
                                            <a t-if="widget.editable" role="menuitem" type="set_cover" class="dropdown-item" data-field="displayed_image_id">Set Cover Image</a>
                                            <a name="%(portal.portal_share_action)d" role="menuitem" type="action" class="dropdown-item">Share</a>
                                            <a t-if="widget.editable" role="menuitem" type="edit" class="dropdown-item">Edit</a>
                                            <a t-if="widget.editable" role="menuitem" class="dropdown-item" name="toggle_active" type="object">
                                                <t t-if="record.active.raw_value">Archive</t>
                                                <t t-if="!record.active.raw_value">Restore</t>
                                            </a>
                                            <a t-if="widget.deletable" role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                            <div role="separator" class="dropdown-divider"></div>
                                            <ul class="oe_kanban_colorpicker" data-field="color"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="o_kanban_record_body">
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" invisible="context.get('fsm_mode', False)"/>
                                    <div t-if="record.displayed_image_id.value">
                                        <field name="displayed_image_id" widget="attachment_image"/>
                                    </div>
                                </div>
                                <div class="o_kanban_record_bottom" t-if="!selection_mode">
                                    <div class="oe_kanban_bottom_left">
                                        <field name="priority" widget="priority"/>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                        <b t-if="record.rating_ids.raw_value.length">
                                            <span style="font-weight:bold;" class="fa fa-fw mt4 fa-smile-o text-success" t-if="record.rating_last_value.value == 5" title="Latest Rating: Satisfied" role="img" aria-label="Happy face"/>
                                            <span style="font-weight:bold;" class="fa fa-fw mt4 fa-meh-o text-warning" t-if="record.rating_last_value.value == 3" title="Latest Rating: Not Satisfied" role="img" aria-label="Neutral face"/>
                                            <span style="font-weight:bold;" class="fa fa-fw mt4 fa-frown-o text-danger" t-if="record.rating_last_value.value == 1" title="Latest Rating: Higly Dissatisfied" role="img" aria-label="Sad face"/>
                                        </b>
                                        <div name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}">
                                            <field name="date_deadline" widget="remaining_days"/>
                                        </div>
                                    </div>
                                    <div class="oe_kanban_bottom_right" t-if="!selection_mode">
                                        <field name="kanban_state" widget="state_selection" groups="base.group_user" invisible="context.get('fsm_mode', False)"/>
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>
                                </div>
                            </div>
                            <div class="oe_clear"></div>
                        </div>
                    </t>
                    </templates>
                </kanban>
            </field>
         </record>

        <record id="view_task_tree2" model="ir.ui.view">
            <field name="name">project.task.tree</field>
            <field name="model">project.task</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <tree string="Tasks" multi_edit="1" sample="1">
                    <field name="message_needaction" invisible="1" readonly="1"/>
                    <field name="is_closed" invisible="1" />
                    <field name="sequence" invisible="1" readonly="1"/>
                    <field name="name"/>
                    <field name="project_id" optional="show" readonly="1"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="parent_id" groups="project.group_subtask_project" optional="hide"/>
                    <field name="user_id" optional="show" widget="many2one_avatar_user"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="date_deadline" optional="hide" widget="remaining_days" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                    <field name="kanban_state" widget="state_selection" optional="hide" readonly="1"/>
                    <field name="stage_id" invisible="context.get('set_visible',False)" optional="show" readonly="1"/>
                </tree>
            </field>
        </record>

        <record id="project_task_view_tree_activity" model="ir.ui.view">
            <field name="name">project.task.tree.activity</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <tree string="Next Activities" decoration-danger="not is_closed and activity_date_deadline &lt; current_date" default_order="activity_date_deadline" multi_edit="1">
                    <field name="is_closed"/>
                    <field name="name"/>
                    <field name="project_id"/>
                    <field name="activity_date_deadline"/>
                    <field name="activity_type_id"/>
                    <field name="activity_summary"/>
                    <field name="stage_id"/>
                </tree>
            </field>
        </record>

        <record id="view_task_calendar" model="ir.ui.view">
            <field name="name">project.task.calendar</field>
            <field name="model">project.task</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <calendar date_start="date_deadline" string="Tasks" mode="month" color="user_id" event_limit="5"
                          hide_time="true" quick_add="False" js_class="project_calendar">
                    <field name="user_id" avatar_field="image_128" filters="1"/>
                    <field name="date_deadline"/>
                    <field name="project_id"/>
                    <field name="priority" widget="priority"/>
                    <field name="stage_id"/>
                </calendar>
            </field>
        </record>

        <record id="view_project_task_pivot" model="ir.ui.view">
            <field name="name">project.task.pivot</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <pivot string="Project Tasks" sample="1">
                    <field name="project_id" type="row"/>
                    <field name="stage_id" type="col"/>
                    <field name="color" invisible="1"/>
                </pivot>
            </field>
        </record>

        <record id="view_project_task_graph" model="ir.ui.view">
            <field name="name">project.task.graph</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <graph string="Project Tasks" sample="1">
                    <field name="project_id"/>
                    <field name="stage_id"/>
                </graph>
            </field>
        </record>

        <record id="project_task_view_activity" model="ir.ui.view">
            <field name="name">project.task.activity</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <activity string="Project Tasks">
                    <field name="user_id"/>
                    <templates>
                        <div t-name="activity-box">
                            <img t-att-src="activity_image('res.users', 'image_128', record.user_id.raw_value)" t-att-title="record.user_id.value" t-att-alt="record.user_id.value"/>
                            <div>
                                <field name="name" display="full"/>
                                <field name="project_id" muted="1" display="full" invisible="context.get('default_project_id', False)"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="action_view_task" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form,calendar,pivot,graph,activity</field>
            <field name="context">{'search_default_my_tasks': 1}</field>
            <field name="search_view_id" ref="view_task_search_form"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p><p>
                    To get things done, use activities and status on tasks.<br/>
                    Chat in real time or by email to collaborate efficiently.
                </p>
            </field>
        </record>

        <record id="action_view_all_task" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,kanban,form,calendar,pivot,graph,activity</field>
            <field name="context">{'search_default_my_tasks': 1, 'all_task': 0}</field>
            <field name="search_view_id" ref="view_task_search_form"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p><p>
                     To get things done, use activities and status on tasks.<br/>
                    Chat in real time or by email to collaborate efficiently.
                </p>
            </field>
        </record>

        <record id="open_view_task_list_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="0"/>
            <field name="view_mode">kanban</field>
            <field name="act_window_id" ref="action_view_task"/>
        </record>
        <record id="open_view_task_list_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="act_window_id" ref="action_view_task"/>
        </record>

        <menuitem name="Tasks" id="menu_project_management" parent="menu_main_pm"
            action="action_view_all_task" sequence="2" groups="base.group_no_one,group_project_user"/>

        <record id="project_task_action_from_partner" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="search_view_id" ref="view_task_search_form"/>
        </record>

        <record id="action_view_task_overpassed_draft" model="ir.actions.act_window">
            <field name="name">Overpassed Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,form,calendar,graph,kanban</field>
            <field name="domain">[('is_closed', '=', False), ('date_deadline','&lt;',time.strftime('%Y-%m-%d'))]</field>
            <field name="filter" eval="True"/>
            <field name="search_view_id" ref="view_task_search_form"/>
        </record>

        <!-- Opening task when double clicking on project -->
        <record id="dblc_proj" model="ir.actions.act_window">
            <field name="res_model">project.task</field>
            <field name="name">Project's tasks</field>
            <field name="view_mode">tree,form,calendar,graph,kanban</field>
            <field name="domain">[('project_id', '=', active_id)]</field>
            <field name="context">{'project_id':active_id}</field>
        </record>

        <!-- Menu item for project -->
        <menuitem id="menu_tasks_config" name="GTD" parent="menu_project_config" sequence="2"/>

        <menuitem action="open_task_type_form" id="menu_project_config_project" name="Stages" parent="menu_project_config" sequence="3" groups="base.group_no_one"/>

        <menuitem action="open_view_project_all" id="menu_projects" name="Projects" parent="menu_main_pm" sequence="1"/>
        <menuitem action="open_view_project_all_config" id="menu_projects_config" name="Projects" parent="menu_project_config" sequence="10"/>

        <!-- User Form -->
        <record id="act_res_users_2_project_task_opened" model="ir.actions.act_window">
            <field name="name">Assigned Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,form,calendar,graph</field>
            <field name="context">{'search_default_user_id': [active_id], 'default_user_id': active_id}</field>
            <field name="binding_model_id" ref="base.model_res_users"/>
            <field name="binding_view_types">form</field>
        </record>

        <!-- Tags -->
        <record model="ir.ui.view" id="project_tags_search_view">
            <field name="name">Tags</field>
            <field name="model">project.tags</field>
            <field name="arch" type="xml">
                <search string="Issue Version">
                    <field name="name"/>
                </search>
            </field>
        </record>

        <record model="ir.ui.view" id="project_tags_form_view">
            <field name="name">Tags</field>
            <field name="model">project.tags</field>
            <field name="arch" type="xml">
                <form string="Tags">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="color" widget="color_picker"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="project_tags_tree_view">
            <field name="name">Tags</field>
            <field name="model">project.tags</field>
            <field name="arch" type="xml">
                <tree string="Tags" editable="bottom" sample="1">
                    <field name="name"/>
                    <field name="color" widget="color_picker"/>
                </tree>
            </field>
        </record>

        <record id="project_tags_action" model="ir.actions.act_window">
            <field name="name">Tags</field>
            <field name="res_model">project.tags</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No tags found. Let's create one!
              </p>
              <p>
                  Tags are perfect to categorize your tasks.
              </p>
            </field>
        </record>
        <menuitem action="project_tags_action" id="menu_project_tags_act" parent="menu_project_config"/>

        <!-- Reporting menus -->
        <menuitem id="menu_project_report" name="Reporting"
            groups="project.group_project_manager"
            parent="menu_main_pm" sequence="99"/>

        <menuitem id="menu_project_report_task_analysis"
            name="Tasks Analysis"
            action="project.action_project_task_user_tree"
            parent="menu_project_report"
            sequence="10"/>

        <menuitem id="rating_rating_menu_project"
            action="rating_rating_action_project_report"
            parent="menu_project_report"
            groups="project.group_project_rating"
            sequence="40"/>

</odoo>

```

## File: views\rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="rating_rating_view_tree_project" model="ir.ui.view">
        <field name="name">rating.rating.tree.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <field name="res_name" position="attributes">
                <attribute name="string">Task</attribute>
            </field>
            <field name="parent_res_name" position="attributes">
                <attribute name="string">Project</attribute>
            </field>
            <field name="rated_partner_id" position="attributes">
                <attribute name="string">Assigned to</attribute>
            </field>
        </field>
    </record>

    <record id="rating_rating_view_form_project" model="ir.ui.view">
        <field name="name">rating.rating.form.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_form"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <field name="res_name" position="attributes">
                <attribute name="string">Task</attribute>
            </field>
            <field name="resource_ref" position="attributes">
                <attribute name="string">Task</attribute>
            </field>
            <field name="parent_ref" position="attributes">
                <attribute name="string">Project</attribute>
            </field>
            <field name="parent_res_name" position="attributes">
                <attribute name="string">Project</attribute>
            </field>
            <field name="rated_partner_id" position="attributes">
                <attribute name="string">Assigned to</attribute>
            </field>
            <div name="rating_image_container" position="replace">
                <field name="rating_text" string="Rating"/>
            </div>
            <xpath expr="//field[@name='is_internal']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_view_pivot" model="ir.ui.view">
        <field name="name">rating.rating.view.pivot.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_pivot"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//pivot" position="attributes">
                <attribute name="js_class">project_rating_pivot</attribute>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_view_graph" model="ir.ui.view">
        <field name="name">rating.rating.view.graph.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_graph"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="attributes">
                <attribute name="js_class">project_rating_graph</attribute>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_view_search_project" model="ir.ui.view">
        <field name="name">rating.rating.search.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_search"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='resource']" position="after">
                <filter string="Project" name="groupby_project" context="{'group_by': 'parent_res_name'}"/>
            </xpath>
            <xpath expr="/search" position="inside">
                <filter string="Last 30 Days" name="rating_last_30_days" domain="[
                    ('create_date', '>=', (datetime.datetime.combine(context_today() + relativedelta(days=-30), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S'))]"
                />
                <separator/>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_action_view_project_rating" model="ir.actions.act_window">
        <field name="name">Rating</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,graph,pivot,form</field>
        <field name="domain">[('consumed','=',True), ('parent_res_model','=','project.project')]</field>
        <field name="search_view_id" ref="rating_rating_view_search_project"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There are no ratings for this project at the moment
            </p>
        </field>
    </record>

    <record id="rating_rating_action_view_project_rating_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="rating_rating_action_view_project_rating"/>
        <field name="view_id" ref="rating_rating_view_tree_project"/>
    </record>

    <record id="rating_rating_action_view_project_rating_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="rating_rating_action_view_project_rating"/>
        <field name="view_id" ref="rating_rating_view_form_project"/>
    </record>

    <record id="rating_rating_action_view_project_rating_pivot" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">pivot</field>
        <field name="act_window_id" ref="rating_rating_action_view_project_rating"/>
        <field name="view_id" ref="rating_rating_view_pivot"/>
    </record>

    <record id="rating_rating_action_view_project_rating_graph" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">graph</field>
        <field name="act_window_id" ref="rating_rating_action_view_project_rating"/>
        <field name="view_id" ref="rating_rating_view_graph"/>
    </record>

    <record id="rating_rating_action_task" model="ir.actions.act_window">
        <field name="name">Customer Ratings on Task</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,pivot,graph,form</field>
        <field name="domain">[('res_model', '=', 'project.task'), ('res_id', '=', active_id), ('consumed', '=', True)]</field>
        <field name="search_view_id" ref="rating_rating_view_search_project"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No customer ratings yet
            </p><p>
                Let's wait for your customers to manifest themselves.
            </p>
        </field>
    </record>

    <record id="rating_rating_action_task_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="rating_rating_action_task"/>
        <field name="view_id" ref="rating_rating_view_tree_project"/>
    </record>

    <record id="rating_rating_action_task_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="rating_rating_action_task"/>
        <field name="view_id" ref="rating_rating_view_form_project"/>
    </record>

    <record id="rating_rating_action_task_pivot" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">pivot</field>
        <field name="act_window_id" ref="rating_rating_action_task"/>
        <field name="view_id" ref="rating_rating_view_pivot"/>
    </record>

    <record id="rating_rating_action_task_graph" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">graph</field>
        <field name="act_window_id" ref="rating_rating_action_task"/>
        <field name="view_id" ref="rating_rating_view_graph"/>
    </record>

    <record id="rating_rating_action_project_report" model="ir.actions.act_window">
        <field name="name">Customer Ratings</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,pivot,graph,form</field>
        <field name="domain">[('parent_res_model','=','project.project'), ('consumed', '=', True)]</field>
        <field name="search_view_id" ref="rating_rating_view_search_project"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No customer ratings yet
            </p><p>
                Let's wait for your customers to manifest themselves.
            </p>
        </field>
        <field name="context">{}</field>
    </record>

    <record id="rating_rating_action_project_report_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="rating_rating_action_project_report"/>
        <field name="view_id" ref="rating_rating_view_tree_project"/>
    </record>

    <record id="rating_rating_action_project_report_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="rating_rating_action_project_report"/>
        <field name="view_id" ref="rating_rating_view_form_project"/>
    </record>

    <record id="rating_rating_action_project_report_pivot" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">pivot</field>
        <field name="act_window_id" ref="rating_rating_action_project_report"/>
        <field name="view_id" ref="rating_rating_view_pivot"/>
    </record>

    <record id="rating_rating_action_project_report_graph" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">graph</field>
        <field name="act_window_id" ref="rating_rating_action_project_report"/>
        <field name="view_id" ref="rating_rating_view_graph"/>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.project</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="50"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form" />
            <field name="arch" type="xml">
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Project" string="Project" data-key="project" groups="project.group_project_manager">
                        <h2>Tasks Management</h2>
                        <div class="row mt16 o_settings_container" id="tasks_management">
                            <div id="use_collaborative_pad" class="col-12 col-lg-6 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="module_pad"/>
                                </div>
                                <div class="o_setting_right_pane" name="pad_project_right_pane">
                                    <label for="module_pad"/>
                                    <div class="text-muted">
                                        Edit tasks' description collaboratively in real time. See each author's text in a distinct color.
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="group_subtask_project"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_subtask_project"/>
                                    <div class="text-muted">
                                        Split your tasks to organize your work into sub-milestones
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="track_customer_satisfaction_setting">
                                <div class="o_setting_left_pane">
                                    <field name="group_project_rating"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_project_rating"/>
                                    <div class="text-muted">
                                        Track customer satisfaction on tasks
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('group_project_rating', '=', False)]}">
                                        <div class="mt8">
                                            <button name="%(project.open_task_type_form)d" icon="fa-arrow-right" type="action" string="Set a Rating Email Template on Stages" class="btn-link"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="recurring_tasks_setting">
                                <div class="o_setting_left_pane">
                                    <field name="group_project_recurring_tasks"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_project_recurring_tasks"/>
                                    <div class="text-muted">
                                        Auto-generate tasks for regular activities
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Time Management</h2>
                        <div class="row mt16 o_settings_container" name="project_time">
                            <div class="col-12 col-lg-6 o_setting_box" name="project_time_management">
                                <div class="o_setting_left_pane">
                                    <field name="module_project_forecast" widget="upgrade_boolean"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_project_forecast"/>
                                    <div class="text-muted" name="project_forecast_msg">
                                        Plan resource allocation across projets and tasks, and estimate deadlines more accurately
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="log_time_tasks_setting">
                                <div class="o_setting_left_pane">
                                    <field name="module_hr_timesheet"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_hr_timesheet"/>
                                    <div class="text-muted">
                                        Track time spent on projects and tasks
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="project_config_settings_action" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'project', 'bin_size': False}</field>
        </record>

        <menuitem id="project_config_settings_menu_action" name="Settings" parent="menu_project_config"
            sequence="0" action="project_config_settings_action" groups="base.group_system"/>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

         <!--  Partners inherited form -->
        <record id="view_task_partner_info_form" model="ir.ui.view">
            <field name="name">res.partner.task.buttons</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="priority" eval="7"/>
            <field name="groups_id" eval="[(4, ref('project.group_project_user'))]"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" type="action" name="%(project_task_action_from_partner)d"
                        context="{'search_default_partner_id': active_id, 'default_partner_id': active_id}" attrs="{'invisible': [('task_count', '=', 0)]}"
                        icon="fa-tasks">
                        <field  string="Tasks" name="task_count" widget="statinfo"/>
                    </button>
                </div>
            </field>
       </record>

</odoo>

```

## File: wizard\project_delete_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProjectDelete(models.TransientModel):
    _name = 'project.delete.wizard'
    _description = 'Project Delete Wizard'

    project_ids = fields.Many2many('project.project', string='Projects')
    task_count = fields.Integer(compute='_compute_task_count')
    projects_archived = fields.Boolean(compute='_compute_projects_archived')

    def _compute_projects_archived(self):
        for wizard in self.with_context(active_test=False):
            wizard.projects_archived = all(not p.active for p in wizard.project_ids)

    def _compute_task_count(self):
        for wizard in self:
            wizard.task_count = sum(wizard.with_context(active_test=False).project_ids.mapped('task_count'))

    def action_archive(self):
        self.project_ids.write({'active': False})

    def confirm_delete(self):
        self.with_context(active_test=False).project_ids.unlink()
        return self.env["ir.actions.actions"]._for_xml_id("project.open_view_project_all_config")

```

## File: wizard\project_delete_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="project_delete_wizard_form" model="ir.ui.view">
            <field name="name">Delete Project</field>
            <field name="model">project.delete.wizard</field>
            <field name="arch" type="xml">
                <form string="Delete Project">
                    <field name="task_count" invisible="1" />
                    <field name="projects_archived" invisible="1" />
                    <group attrs="{'invisible': [('task_count', '=', 0)]}">
                        <span>You cannot delete a project containing tasks. You can either archive it or first delete all of its tasks.</span>
                    </group>
                    <group attrs="{'invisible': [('task_count', '>', 0)]}">
                        <span>Are you sure you want to delete this project ?</span>
                    </group>
                    <footer attrs="{'invisible': [('task_count', '=', 0)]}">
                        <button string="Archive project" type="object" name="action_archive" class="btn btn-primary" attrs="{'invisible': [('projects_archived', '=', True)]}" />
                        <button string="Discard" special="cancel" />
                    </footer>
                    <footer attrs="{'invisible': [('task_count', '>', 0)]}">
                        <button string="Ok" type="object" name="confirm_delete" class="btn btn-primary" />
                        <button string="Cancel" special="cancel" />
                    </footer>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\project_task_type_delete.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

import ast


class ProjectTaskTypeDelete(models.TransientModel):
    _name = 'project.task.type.delete.wizard'
    _description = 'Project Stage Delete Wizard'

    project_ids = fields.Many2many('project.project', domain="['|', ('active', '=', False), ('active', '=', True)]", string='Projects', ondelete='cascade')
    stage_ids = fields.Many2many('project.task.type', string='Stages To Delete', ondelete='cascade')
    tasks_count = fields.Integer('Number of tasks', compute='_compute_tasks_count')
    stages_active = fields.Boolean(compute='_compute_stages_active')

    @api.depends('project_ids')
    def _compute_tasks_count(self):
        for wizard in self:
            wizard.tasks_count = self.with_context(active_test=False).env['project.task'].search_count([('stage_id', 'in', wizard.stage_ids.ids)])

    @api.depends('stage_ids')
    def _compute_stages_active(self):
        for wizard in self:
            wizard.stages_active = all(wizard.stage_ids.mapped('active'))

    def action_archive(self):
        if len(self.project_ids) <= 1:
            return self.action_confirm()

        return {
            'name': _('Confirmation'),
            'view_mode': 'form',
            'res_model': 'project.task.type.delete.wizard',
            'views': [(self.env.ref('project.view_project_task_type_delete_confirmation_wizard').id, 'form')],
            'type': 'ir.actions.act_window',
            'res_id': self.id,
            'target': 'new',
            'context': self.env.context,
        }

    def action_confirm(self):
        tasks = self.with_context(active_test=False).env['project.task'].search([('stage_id', 'in', self.stage_ids.ids)])
        tasks.write({'active': False})
        self.stage_ids.write({'active': False})
        return self._get_action()

    def action_unlink(self):
        self.stage_ids.unlink()
        return self._get_action()

    def _get_action(self):
        project_id = self.env.context.get('default_project_id')

        if project_id:
            action = self.env["ir.actions.actions"]._for_xml_id("project.action_view_task")
            action['domain'] = [('project_id', '=', project_id)]
            action['context'] = str({
                'pivot_row_groupby': ['user_id'],
                'default_project_id': project_id,
            })
        elif self.env.context.get('stage_view'):
            action = self.env["ir.actions.actions"]._for_xml_id("project.open_task_type_form")
        else:
            action = self.env["ir.actions.actions"]._for_xml_id("project.action_view_all_task")

        context = dict(ast.literal_eval(action.get('context')), active_test=True)
        action['context'] = context
        action['target'] = 'main'
        return action

```

## File: wizard\project_task_type_delete_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_project_task_type_delete_wizard" model="ir.ui.view">
        <field name="name">project.task.type.delete.wizard.form</field>
        <field name="model">project.task.type.delete.wizard</field>
        <field name="arch" type="xml">
            <form string="Delete Stage">
                <field name="tasks_count" invisible="1" />
                <field name="stages_active" invisible="1" />
                <div attrs="{'invisible': [('tasks_count', '>', 0)]}">
                    <p>Are you sure you want to delete those stages ?</p>
                </div>
                <div attrs="{'invisible': ['|', ('stages_active', '=', False), ('tasks_count', '=', 0)]}">
                    <p>You cannot delete stages containing tasks. You can either archive them or first delete all of their tasks.</p>
                </div>
                <div attrs="{'invisible': ['|', ('stages_active', '=', True), ('tasks_count', '=', 0)]}">
                    <p>You cannot delete stages containing tasks. You should first delete all of their tasks.</p>
                </div>
                <footer>
                    <button string="Archive Stages" type="object" name="action_archive" class="btn btn-primary" attrs="{'invisible': ['|', ('stages_active', '=', False), ('tasks_count', '=', 0)]}" />
                    <button string="Delete" type="object" name="action_unlink" class="btn btn-primary" attrs="{'invisible': [('tasks_count', '>', 0)]}" />
                    <button string="Discard" special="cancel" />
                </footer>
            </form>
        </field>
    </record>

    <record id="view_project_task_type_delete_confirmation_wizard" model="ir.ui.view">
        <field name="name">project.task.type.delete.wizard.form</field>
        <field name="model">project.task.type.delete.wizard</field>
        <field name="arch" type="xml">
            <form string="Delete Stage">
                <div>
                    <p>This will archive the stages and all the tasks they contain from the following projects:</p>
                    <field name="project_ids" readonly="1">
                        <tree>
                            <field name="name"/>
                        </tree>
                    </field>
                    <p>Are you sure you want to continue?</p>
                </div>
                <footer>
                    <button string="Confirm" type="object" name="action_confirm" class="btn btn-primary"/>
                    <button string="Discard" special="cancel" />
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_delete_wizard
from . import project_task_type_delete

```

