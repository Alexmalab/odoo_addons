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

from odoo import api, SUPERUSER_ID


def _check_exists_collaborators_for_project_sharing(env):
    """ Check if it exists at least a collaborator in a shared project

        If it is the case we need to active the portal rules added only for this feature.
    """
    collaborator = env['project.collaborator'].search([], limit=1)
    if collaborator:
        # Then we need to enable the access rights linked to project sharing for the portal user
        env['project.collaborator']._toggle_project_sharing_portal_rules(True)


def _project_post_init(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    _check_exists_collaborators_for_project_sharing(env)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Project',
    'version': '1.2',
    'website': 'https://www.odoo.com/app/project',
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
        'security/ir.model.access.xml',
        'data/digest_data.xml',
        'report/project_report_views.xml',
        'report/project_task_burndown_chart_report_views.xml',
        'views/analytic_views.xml',
        'views/digest_views.xml',
        'views/rating_views.xml',
        'views/project_update_views.xml',
        'views/project_update_templates.xml',
        'views/project_project_stage_views.xml',
        'wizard/project_share_wizard_views.xml',
        'views/project_collaborator_views.xml',
        'views/project_views.xml',
        'views/res_partner_views.xml',
        'views/res_config_settings_views.xml',
        'views/mail_activity_views.xml',
        'views/project_sharing_views.xml',
        'views/project_portal_templates.xml',
        'views/project_task_templates.xml',
        'views/project_sharing_templates.xml',
        'data/ir_cron_data.xml',
        'data/mail_data.xml',
        'data/mail_template_data.xml',
        'data/project_data.xml',
        'wizard/project_delete_wizard_views.xml',
        'wizard/project_task_type_delete_views.xml',
    ],
    'demo': ['data/project_demo.xml'],
    'test': [
    ],
    'installable': True,
    'auto_install': False,
    'application': True,
    'post_init_hook': '_project_post_init',
    'assets': {
        'web.assets_backend': [
            'project/static/src/burndown_chart/*',
            'project/static/src/project_control_panel/*',
            'project/static/src/css/project.css',
            'project/static/src/js/project_activity.js',
            'project/static/src/js/project_control_panel.js',
            'project/static/src/js/project_form.js',
            'project/static/src/js/project_graph_view.js',
            'project/static/src/js/project_kanban.js',
            'project/static/src/js/project_list.js',
            'project/static/src/js/project_pivot_view.js',
            'project/static/src/js/project_rating_graph_view.js',
            'project/static/src/js/project_rating_pivot_view.js',
            'project/static/src/js/project_task_kanban_examples.js',
            'project/static/src/js/tours/project.js',
            'project/static/src/js/project_calendar.js',
            'project/static/src/js/right_panel/*',
            'project/static/src/js/update/*',
            'project/static/src/js/widgets/*',
            'project/static/src/scss/project_dashboard.scss',
            'project/static/src/scss/project_form.scss',
            'project/static/src/scss/project_rightpanel.scss',
            'project/static/src/scss/project_widgets.scss',
        ],
        "web.assets_backend_legacy_lazy": [
            'project/static/src/js/*_legacy.js',
        ],
        'web.assets_frontend': [
            'project/static/src/scss/portal_rating.scss',
            'project/static/src/js/portal_rating.js',
        ],
        'web.assets_qweb': [
            'project/static/src/xml/**/*',
            'project/static/src/burndown_chart/**/*.xml',
            'project/static/src/project_control_panel/**/*.xml',
        ],
        'web.qunit_suite_tests': [
            'project/static/tests/burndown_chart_tests.js',
            'project/static/tests/project_form_tests.js',
        ],
        'web.assets_tests': [
            'project/static/tests/tours/**/*',
        ],
        'project.assets_qweb': [
            ('include', 'web.assets_qweb'),
            'project/static/src/project_sharing/**/*.xml',
        ],
        'project.webclient': [
            ('include', 'web.assets_backend'),

            # Remove Longpolling bus and packages needed this bus
            ('remove', 'bus/static/src/js/services/assets_watchdog_service.js'),
            ('remove', 'mail/static/src/services/messaging/messaging.js'),

            ('remove', 'mail/static/src/components/dialog_manager/dialog_manager.js'),
            ('remove', 'mail/static/src/services/dialog_service/dialog_service.js'),
            ('remove', 'mail/static/src/components/chat_window_manager/chat_window_manager.js'),
            ('remove', 'mail/static/src/services/chat_window_service/chat_window_service.js'),

            'web/static/src/legacy/js/public/public_widget.js',
            'portal/static/src/js/portal_chatter.js',
            'portal/static/src/js/portal_composer.js',
            'project/static/src/project_sharing/search/favorite_menu/custom_favorite_item.xml',
            'project/static/src/project_sharing/**/*.js',
            'project/static/src/scss/project_sharing/*',
            'web/static/src/start.js',
            'web/static/src/legacy/legacy_setup.js',
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
from operator import itemgetter
from markupsafe import Markup

from odoo import conf, http, _
from odoo.exceptions import AccessError, MissingError
from odoo.http import request
from odoo.addons.portal.controllers.portal import CustomerPortal, pager as portal_pager
from odoo.tools import groupby as groupbyelem

from odoo.osv.expression import OR, AND

from odoo.addons.web.controllers.main import HomeStaticTemplateHelpers


class ProjectCustomerPortal(CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        if 'project_count' in counters:
            values['project_count'] = request.env['project.project'].search_count([]) \
                if request.env['project.project'].check_access_rights('read', raise_exception=False) else 0
        if 'task_count' in counters:
            values['task_count'] = request.env['project.task'].search_count([('project_id', '!=', False)]) \
                if request.env['project.task'].check_access_rights('read', raise_exception=False) else 0
        return values

    # ------------------------------------------------------------
    # My Project
    # ------------------------------------------------------------
    def _project_get_page_view_values(self, project, access_token, page=1, date_begin=None, date_end=None, sortby=None, search=None, search_in='content', groupby=None, **kwargs):
        # TODO: refactor this because most of this code is duplicated from portal_my_tasks method
        values = self._prepare_portal_layout_values()
        searchbar_sortings = self._task_get_searchbar_sortings()

        searchbar_inputs = self._task_get_searchbar_inputs()
        searchbar_groupby = self._task_get_searchbar_groupby()

        # default sort by value
        if not sortby or sortby not in searchbar_sortings:
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']

        # default filter by value
        domain = [('project_id', '=', project.id)]

        # default group by value
        if not groupby:
            groupby = 'project'

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        # search
        if search and search_in:
            domain += self._task_get_search_domain(search_in, search)

        Task = request.env['project.task']
        if access_token:
            Task = Task.sudo()
        elif not request.env.user._is_public():
            domain = AND([domain, request.env['ir.rule']._compute_domain(Task._name, 'read')])
            Task = Task.sudo()

        # task count
        task_count = Task.search_count(domain)
        # pager
        url = "/my/project/%s" % project.id
        pager = portal_pager(
            url=url,
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby, 'groupby': groupby, 'search_in': search_in, 'search': search, 'access_token': access_token},
            total=task_count,
            page=page,
            step=self._items_per_page
        )
        # content according to pager and archive selected
        order = self._task_get_order(order, groupby)

        tasks = Task.search(domain, order=order, limit=self._items_per_page, offset=pager['offset'])
        request.session['my_project_tasks_history'] = tasks.ids[:100]

        groupby_mapping = self._task_get_groupby_mapping()
        group = groupby_mapping.get(groupby)
        if group:
            grouped_tasks = [Task.concat(*g) for k, g in groupbyelem(tasks, itemgetter(group))]
        else:
            grouped_tasks = [tasks]

        values.update(
            date=date_begin,
            date_end=date_end,
            grouped_tasks=grouped_tasks,
            page_name='project',
            default_url=url,
            pager=pager,
            searchbar_sortings=searchbar_sortings,
            searchbar_groupby=searchbar_groupby,
            searchbar_inputs=searchbar_inputs,
            search_in=search_in,
            search=search,
            sortby=sortby,
            groupby=groupby,
            project=project,
        )
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
        if not sortby or sortby not in searchbar_sortings:
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

    @http.route(['/my/project/<int:project_id>', '/my/project/<int:project_id>/page/<int:page>'], type='http', auth="public", website=True)
    def portal_my_project(self, project_id=None, access_token=None, page=1, date_begin=None, date_end=None, sortby=None, search=None, search_in='content', groupby=None, **kw):
        try:
            project_sudo = self._document_check_access('project.project', project_id, access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')
        if project_sudo.collaborator_count and project_sudo.with_user(request.env.user)._check_project_sharing_access():
            return request.render("project.project_sharing_portal", {'project_id': project_id})
        project_sudo = project_sudo if access_token else project_sudo.with_user(request.env.user)
        values = self._project_get_page_view_values(project_sudo, access_token, page, date_begin, date_end, sortby, search, search_in, groupby, **kw)
        values['task_url'] = 'project/%s/task' % project_id
        return request.render("project.portal_my_project", values)

    def _prepare_project_sharing_session_info(self, project):
        session_info = request.env['ir.http'].session_info()
        user_context = request.session.get_context() if request.session.uid else {}
        mods = conf.server_wide_modules or []
        qweb_checksum = HomeStaticTemplateHelpers.get_qweb_templates_checksum(debug=request.session.debug, bundle="project.assets_qweb")
        if request.env.lang:
            lang = request.env.lang
            session_info['user_context']['lang'] = lang
            # Update Cache
            user_context['lang'] = lang
        lang = user_context.get("lang")
        translation_hash = request.env['ir.translation'].get_web_translations_hash(mods, lang)
        cache_hashes = {
            "qweb": qweb_checksum,
            "translations": translation_hash,
        }

        project_company = project.company_id
        session_info.update(
            cache_hashes=cache_hashes,
            action_name='project.project_sharing_project_task_action',
            project_id=project.id,
            user_companies={
                'current_company': project_company.id,
                'allowed_companies': {
                    project_company.id: {
                        'id': project_company.id,
                        'name': project_company.name,
                    },
                },
            },
            # FIXME: See if we prefer to give only the currency that the portal user just need to see the correct information in project sharing
            currencies=request.env['ir.http'].get_currencies(),
        )
        return session_info

    @http.route("/my/project/<int:project_id>/project_sharing", type="http", auth="user", methods=['GET'])
    def render_project_backend_view(self, project_id):
        project = request.env['project.project'].sudo().browse(project_id)
        if not project.exists() or not project.with_user(request.env.user)._check_project_sharing_access():
            return request.not_found()

        return request.render(
            'project.project_sharing_embed',
            {'session_info': self._prepare_project_sharing_session_info(project)},
        )

    @http.route('/my/project/<int:project_id>/task/<int:task_id>', type='http', auth='public', website=True)
    def portal_my_project_task(self, project_id=None, task_id=None, access_token=None, **kw):
        try:
            project_sudo = self._document_check_access('project.project', project_id, access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')
        Task = request.env['project.task']
        if access_token:
            Task = Task.sudo()
        task_sudo = Task.search([('project_id', '=', project_id), ('id', '=', task_id)], limit=1).sudo()
        task_sudo.attachment_ids.generate_access_token()
        values = self._task_get_page_view_values(task_sudo, access_token, project=project_sudo, **kw)
        values['project'] = project_sudo
        return request.render("project.portal_my_task", values)

    # ------------------------------------------------------------
    # My Task
    # ------------------------------------------------------------
    def _task_get_page_view_values(self, task, access_token, **kwargs):
        project = kwargs.get('project')
        if project:
            project_accessible = True
            page_name = 'project_task'
            history = 'my_project_tasks_history'
        else:
            page_name = 'task'
            history = 'my_tasks_history'
            try:
                project_accessible = bool(task.project_id.id and self._document_check_access('project.project', task.project_id.id))
            except (AccessError, MissingError):
                project_accessible = False
        values = {
            'page_name': page_name,
            'task': task,
            'user': request.env.user,
            'project_accessible': project_accessible,
        }

        values = self._get_page_view_values(task, access_token, values, history, False, **kwargs)
        if project:
            history = request.session.get('my_project_tasks_history', [])
            try:
                current_task_index = history.index(task.id)
            except ValueError:
                return values

            total_task = len(history)
            task_url = f"{task.project_id.access_url}/task/%s?model=project.project&res_id={values['user'].id}&access_token={access_token}"

            values['prev_record'] = current_task_index != 0 and task_url % history[current_task_index - 1]
            values['next_record'] = current_task_index < total_task - 1 and task_url % history[current_task_index + 1]

        return values

    def _task_get_searchbar_sortings(self):
        return {
            'date': {'label': _('Newest'), 'order': 'create_date desc', 'sequence': 1},
            'name': {'label': _('Title'), 'order': 'name', 'sequence': 2},
            'project': {'label': _('Project'), 'order': 'project_id, stage_id', 'sequence': 3},
            'stage': {'label': _('Stage'), 'order': 'stage_id, project_id', 'sequence': 5},
            'status': {'label': _('Status'), 'order': 'kanban_state', 'sequence': 6},
            'priority': {'label': _('Priority'), 'order': 'priority desc', 'sequence': 7},
            'date_deadline': {'label': _('Deadline'), 'order': 'date_deadline asc', 'sequence': 8},
            'update': {'label': _('Last Stage Update'), 'order': 'date_last_stage_update desc', 'sequence': 10},
        }

    def _task_get_searchbar_groupby(self):
        values = {
            'none': {'input': 'none', 'label': _('None'), 'order': 1},
            'project': {'input': 'project', 'label': _('Project'), 'order': 2},
            'stage': {'input': 'stage', 'label': _('Stage'), 'order': 4},
            'status': {'input': 'status', 'label': _('Status'), 'order': 5},
            'priority': {'input': 'priority', 'label': _('Priority'), 'order': 6},
            'customer': {'input': 'customer', 'label': _('Customer'), 'order': 9},
        }
        return dict(sorted(values.items(), key=lambda item: item[1]["order"]))

    def _task_get_groupby_mapping(self):
        return {
            'project': 'project_id',
            'stage': 'stage_id',
            'customer': 'partner_id',
            'priority': 'priority',
            'status': 'kanban_state',
        }

    def _task_get_order(self, order, groupby):
        groupby_mapping = self._task_get_groupby_mapping()
        field_name = groupby_mapping.get(groupby, '')
        if not field_name:
            return order
        return '%s, %s' % (field_name, order)

    def _task_get_searchbar_inputs(self):
        values = {
            'all': {'input': 'all', 'label': _('Search in All'), 'order': 1},
            'content': {'input': 'content', 'label': Markup(_('Search <span class="nolabel"> (in Content)</span>')), 'order': 1},
            'ref': {'input': 'ref', 'label': _('Search in Ref'), 'order': 1},
            'project': {'input': 'project', 'label': _('Search in Project'), 'order': 2},
            'users': {'input': 'users', 'label': _('Search in Assignees'), 'order': 3},
            'stage': {'input': 'stage', 'label': _('Search in Stages'), 'order': 4},
            'status': {'input': 'status', 'label': _('Search in Status'), 'order': 5},
            'priority': {'input': 'priority', 'label': _('Search in Priority'), 'order': 6},
            'message': {'input': 'message', 'label': _('Search in Messages'), 'order': 10},
        }
        return dict(sorted(values.items(), key=lambda item: item[1]["order"]))

    def _task_get_search_domain(self, search_in, search):
        search_domain = []
        if search_in in ('content', 'all'):
            search_domain.append([('name', 'ilike', search)])
            search_domain.append([('description', 'ilike', search)])
        if search_in in ('customer', 'all'):
            search_domain.append([('partner_id', 'ilike', search)])
        if search_in in ('message', 'all'):
            search_domain.append([('message_ids.body', 'ilike', search)])
        if search_in in ('stage', 'all'):
            search_domain.append([('stage_id', 'ilike', search)])
        if search_in in ('project', 'all'):
            search_domain.append([('project_id', 'ilike', search)])
        if search_in in ('ref', 'all'):
            search_domain.append([('id', 'ilike', search)])
        if search_in in ('users', 'all'):
            user_ids = request.env['res.users'].sudo().search([('name', 'ilike', search)])
            search_domain.append([('user_ids', 'in', user_ids.ids)])
        if search_in in ('priority', 'all'):
            search_domain.append([('priority', 'ilike', search == 'normal' and '0' or '1')])
        if search_in in ('status', 'all'):
            search_domain.append([
                ('kanban_state', 'ilike', 'normal' if search == 'In Progress' else 'done' if search == 'Ready' else 'blocked' if search == 'Blocked' else search)
            ])
        return OR(search_domain)

    @http.route(['/my/tasks', '/my/tasks/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_tasks(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, search=None, search_in='content', groupby=None, **kw):
        values = self._prepare_portal_layout_values()
        searchbar_sortings = self._task_get_searchbar_sortings()
        searchbar_sortings = dict(sorted(self._task_get_searchbar_sortings().items(),
                                         key=lambda item: item[1]["sequence"]))

        searchbar_filters = {
            'all': {'label': _('All'), 'domain': [('project_id', '!=', False)]},
        }

        searchbar_inputs = self._task_get_searchbar_inputs()
        searchbar_groupby = self._task_get_searchbar_groupby()

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
        if not sortby or sortby not in searchbar_sortings:
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
            domain += self._task_get_search_domain(search_in, search)

        TaskSudo = request.env['project.task'].sudo()
        domain = AND([domain, request.env['ir.rule']._compute_domain(TaskSudo._name, 'read')])

        # task count
        task_count = TaskSudo.search_count(domain)
        # pager
        pager = portal_pager(
            url="/my/tasks",
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby, 'filterby': filterby, 'groupby': groupby, 'search_in': search_in, 'search': search},
            total=task_count,
            page=page,
            step=self._items_per_page
        )
        # content according to pager and archive selected
        order = self._task_get_order(order, groupby)

        tasks = TaskSudo.search(domain, order=order, limit=self._items_per_page, offset=pager['offset'])
        request.session['my_tasks_history'] = tasks.ids[:100]

        groupby_mapping = self._task_get_groupby_mapping()
        group = groupby_mapping.get(groupby)
        if group:
            grouped_tasks = [request.env['project.task'].concat(*g) for k, g in groupbyelem(tasks, itemgetter(group))]
        else:
            grouped_tasks = [tasks] if tasks else []

        task_states = dict(request.env['project.task']._fields['kanban_state']._description_selection(request.env))
        if sortby == 'status':
            if groupby == 'none' and grouped_tasks:
                grouped_tasks[0] = grouped_tasks[0].sorted(lambda tasks: task_states.get(tasks.kanban_state))
            else:
                grouped_tasks.sort(key=lambda tasks: task_states.get(tasks[0].kanban_state))

        values.update({
            'date': date_begin,
            'date_end': date_end,
            'grouped_tasks': grouped_tasks,
            'page_name': 'task',
            'default_url': '/my/tasks',
            'task_url': 'task',
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

## File: controllers\project_sharing_chatter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import Forbidden

from odoo.http import request, route

from odoo.addons.portal.controllers.mail import PortalChatter
from .portal import ProjectCustomerPortal


class ProjectSharingChatter(PortalChatter):
    def _check_project_access_and_get_token(self, project_id, res_model, res_id, token):
        """ Check if the chatter in project sharing can be accessed

            If the portal user is in the project sharing, then we do not have the access token of the task
            but we can have the one of the project (if the user accessed to the project sharing views via the shared link).
            So, we need to check if the chatter is for a task and if the res_id is a task
            in the project shared. Then, if we had the project token and this one is the one in the project
            then we return the token of the task to continue the portal chatter process.
            If we do not have any token, then we need to check if the portal user is a follower of the project shared.
            If it is the case, then we give the access token of the task.
        """
        project_sudo = ProjectCustomerPortal._document_check_access(self, 'project.project', project_id, token)
        can_access = project_sudo and res_model == 'project.task' and project_sudo.with_user(request.env.user)._check_project_sharing_access()
        task = None
        if can_access:
            task = request.env['project.task'].sudo().search([('id', '=', res_id), ('project_id', '=', project_sudo.id)])
        if not can_access or not task:
            raise Forbidden()
        return task[task._mail_post_token_field]

    # ============================================================ #
    # Note concerning the methods portal_chatter_(init/post/fetch)
    # ============================================================ #
    #
    # When the project is shared to a portal user with the edit rights,
    # he has the read/write access to the related tasks. So it could be
    # possible to call directly the message_post method on a task.
    #
    # This change is considered as safe, as we only willingly expose
    # records, for some assumed fields only, and this feature is
    # optional and opt-in. (like the public employee model for example).
    # It doesn't allow portal users to access other models, like
    # a timesheet or an invoice.
    #
    # It could seem odd to use those routes, and converting the project
    # access token into the task access token, as the user has actually
    # access to the records.
    #
    # However, it has been decided that it was the less hacky way to
    # achieve this, as:
    #
    # - We're reusing the existing routes, that convert all the data
    #   into valid arguments for the methods we use (message_post, ...).
    #   That way, we don't have to reinvent the wheel, duplicating code
    #   from mail/portal that surely will lead too desynchronization
    #   and inconsistencies over the time.
    #
    # - We don't define new routes, to do the exact same things than portal,
    #   considering that the portal user can use message_post for example
    #   because he has access to the record.
    #   Let's suppose that we remove this in a future development, those
    #   new routes won't be valid anymore.
    #
    # - We could have reused the mail widgets, as we already reuse the
    #   form/list/kanban views, etc. However, we only want to display
    #   the messages and allow to post. We don't need the next activities
    #   the followers system, etc. This required to override most of the
    #   mail.thread basic methods, without being sure that this would
    #   work with other installed applications or customizations

    @route()
    def portal_chatter_init(self, res_model, res_id, domain=False, limit=False, **kwargs):
        project_sharing_id = kwargs.get('project_sharing_id')
        if project_sharing_id:
            # if there is a token in `kwargs` then it should be the access_token of the project shared
            token = self._check_project_access_and_get_token(project_sharing_id, res_model, res_id, kwargs.get('token'))
            if token:
                del kwargs['project_sharing_id']
                kwargs['token'] = token
        return super().portal_chatter_init(res_model, res_id, domain=domain, limit=limit, **kwargs)

    @route()
    def portal_chatter_post(self, res_model, res_id, message, attachment_ids=None, attachment_tokens=None, **kw):
        project_sharing_id = kw.get('project_sharing_id')
        if project_sharing_id:
            token = self._check_project_access_and_get_token(project_sharing_id, res_model, res_id, kw.get('token'))
            if token:
                del kw['project_sharing_id']
                kw['token'] = token
        return super().portal_chatter_post(res_model, res_id, message, attachment_ids=attachment_ids, attachment_tokens=attachment_tokens, **kw)

    @route()
    def portal_message_fetch(self, res_model, res_id, domain=False, limit=10, offset=0, **kw):
        project_sharing_id = kw.get('project_sharing_id')
        if project_sharing_id:
            token = self._check_project_access_and_get_token(project_sharing_id, res_model, res_id, kw.get('token'))
            if token:
                kw['token'] = token
        return super().portal_message_fetch(res_model, res_id, domain=domain, limit=limit, offset=offset, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal
from . import project_sharing_chatter

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

## File: data\ir_cron_data.xml

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
</odoo>

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
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
    </record>
    <record id="mt_task_dependency_change" model="mail.message.subtype">
        <field name="name">Task Dependency Changes</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="hidden" eval="True"/>
    </record>
    <!-- Project-related subtypes for messaging / Chatter -->
    <record id="mt_project_stage_change" model="mail.message.subtype">
        <field name="name">Project Stage Changed</field>
        <field name="sequence">9</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="hidden" eval="True"/>
    </record>
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
    <record id="mt_project_task_dependency_change" model="mail.message.subtype">
        <field name="name">Task Dependency Changes</field>
        <field name="sequence">15</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_dependency_change"/>
        <field name="relation_field">project_id</field>
        <field name="hidden" eval="True"/>
    </record>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Sample stage-related template -->
        <record id="mail_template_data_project_task" model="mail.template">
            <field name="name">Task: Reception Acknowledgment</field>
            <field name="model_id" ref="project.model_project_task"/>
            <field name="subject">Reception of {{ object.name }}</field>
            <field name="use_default_to" eval="True"/>
            <field name="body_html" type="html">
<div>
    Dear <t t-out="object.partner_id.name or 'customer'">Brandon Freeman</t>,<br/>
    Thank you for your enquiry.<br />
    If you have any questions, please let us know.
    <br/><br/>
    Thank you,
    <t t-if="user.signature">
        <br />
        <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
    </t>
</div>
        </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Mail sent to request a rating for a task -->
        <record id="rating_project_request_email_template" model="mail.template">
            <field name="name">Task: Rating Request</field>
            <field name="model_id" ref="project.model_project_task"/>
            <field name="subject">{{ object.project_id.company_id.name }}: Satisfaction Survey</field>
            <field name="email_from">{{ (object.rating_get_rated_partner_id().email_formatted if object.rating_get_rated_partner_id() else user.email_formatted) }}</field>
            <field name="partner_to" >{{ object.rating_get_partner_id().id }}</field>
            <field name="body_html" type="html">
<div>
    <t t-set="access_token" t-value="object.rating_get_access_token()"/>
    <t t-set="partner" t-value="object.rating_get_partner_id()"/>
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="width:100%; margin:0px auto;">
    <tbody>
        <tr><td valign="top" style="font-size: 13px;">
            <t t-if="partner.name">
                Hello <t t-out="partner.name or ''">Brandon Freeman</t>,<br/><br/>
            </t>
            <t t-else="">
                Hello,<br/><br/>
            </t>
            Please take a moment to rate our services related to the task "<strong t-out="object.name or ''">Planning and budget</strong>"
            <t t-if="object.rating_get_rated_partner_id().name">
                assigned to <strong t-out="object.rating_get_rated_partner_id().name or ''">Mitchell Admin</strong>.<br/>
            </t>
            <t t-else="">
                .<br/>
            </t>
        </td></tr>
        <tr><td style="text-align: center;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" summary="o_mail_notification" style="width:100%; margin: 32px 0px 32px 0px;">
                <tr><td style="font-size: 13px;">
                    <strong>Tell us how you feel about our service</strong><br/>
                    <span style="text-color: #888888">(click on one of these smileys)</span>
                </td></tr>
                <tr><td style="font-size: 13px;">
                    <table style="width:100%;text-align:center;margin-top:2rem;">
                        <tr>
                            <td>
                                <a t-attf-href="/rate/{{ access_token }}/5">
                                    <img alt="Satisfied" src="/rating/static/src/img/rating_5.png" title="Satisfied"/>
                                </a>
                            </td>
                            <td>
                                <a t-attf-href="/rate/{{ access_token }}/3">
                                    <img alt="Okay" src="/rating/static/src/img/rating_3.png" title="Okay"/>
                                </a>
                            </td>
                            <td>
                                <a t-attf-href="/rate/{{ access_token }}/1">
                                    <img alt="Dissatisfied" src="/rating/static/src/img/rating_1.png" title="Dissatisfied"/>
                                </a>
                            </td>
                        </tr>
                    </table>
                </td></tr>
            </table>
        </td></tr>
        <tr><td valign="top" style="font-size: 13px;">
            We appreciate your feedback. It helps us to improve continuously.
            <t t-if="object.project_id.rating_status == 'stage'">
                <br/><span style="margin: 0px 0px 0px 0px; font-size: 12px; opacity: 0.5; color: #454748;">This customer survey has been sent because your task has been moved to the stage <b t-out="object.stage_id.name or ''">In progress</b></span>
            </t>
            <t t-if="object.project_id.rating_status == 'periodic'">
                <br/><span style="margin: 0px 0px 0px 0px; font-size: 12px; opacity: 0.5; color: #454748;">This customer survey is sent <b t-out="object.project_id.rating_status_period or ''">Weekly</b> as long as the task is in the <b t-out="object.stage_id.name or ''">In progress</b> stage.</span>
            </t>
        </td></tr>
    </tbody>
    </table>
</div>
            </field>
            <field name="lang">{{ object.rating_get_partner_id().lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- You have been assigned email -->
        <template id="project_message_user_assigned">
<p style="margin: 0px;">
    <span>Dear <t t-esc="assignee_name"/>,</span><br />
    <span style="margin-top: 8px;">You have been assigned to the <t t-esc="model_description or 'document'"/> <t t-esc="object.display_name"/>.</span>
</p>
<p style="padding-top: 24px; padding-bottom: 16px;">
    <a t-att-href="access_link" t-att-data-oe-model="object._name" t-att-data-oe-id="object.id" style="background-color:#875A7B; padding: 10px; text-decoration: none; color: #fff; border-radius: 5px;">
            View <t t-esc="model_description or 'document'"/>
    </a>
</p>
        </template>
    </data>
</odoo>

```

## File: data\project_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Project Stages -->
    <record id="project_project_stage_0" model="project.project.stage">
        <field name="sequence">10</field>
        <field name="name">To Do</field>
    </record>

    <record id="project_project_stage_1" model="project.project.stage">
        <field name="sequence">15</field>
        <field name="name">In Progress</field>
    </record>

    <record id="project_project_stage_2" model="project.project.stage">
        <field name="sequence">20</field>
        <field name="name">Done</field>
        <field name="fold" eval="True"/>
    </record>

    <record id="project_project_stage_3" model="project.project.stage">
        <field name="sequence">25</field>
        <field name="name">Canceled</field>
        <field name="fold" eval="True"/>
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

        <!-- Groups : Add subtasks, recurring tasks and task dependencies by default -->
        <record id="base.group_user" model="res.groups">
            <field name="implied_ids"
                   eval="[(4, ref('group_subtask_project')), (4, ref('group_project_recurring_tasks')), (4, ref('group_project_task_dependencies'))]"/>
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
        <record id="project_tags_04" model="project.tags">
            <field name="name">Internal</field>
        </record>
        <record id="project_tags_05" model="project.tags">
            <field name="name">External</field>
        </record>

        <!-- Task Stages -->
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
            <field name="tag_ids" eval="[(4, ref('project.project_tags_05'))]"/>
            <field name="stage_id" ref="project.project_project_stage_1"/>
        </record>

        <record id="project_project_2" model="project.project">
            <field name="name">Research &amp; Development</field>
            <field name="privacy_visibility">followers</field>
            <field name="user_id" ref="base.user_demo"/>
            <field name="type_ids" eval="[(4, ref('project_stage_0')), (4, ref('project_stage_1')), (4, ref('project_stage_2')), (4, ref('project_stage_3'))]"/>
            <field name="favorite_user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="tag_ids" eval="[(4, ref('project.project_tags_04'))]"/>
            <field name="stage_id" ref="project.project_project_stage_1"/>
        </record>

        <record id="project_project_3" model="project.project">
            <field name="date_start" eval="(DateTime.today() + relativedelta(months=1)).strftime('%Y-%m-%d 10:00:00')"/>
            <field name="name">Renovations</field>
            <field name="color">4</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="type_ids" eval="[(4, ref('project_stage_0')), (4, ref('project_stage_1')), (4, ref('project_stage_2')), (4, ref('project_stage_3'))]"/>
            <field name="favorite_user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="tag_ids" eval="[(4, ref('project_tags_04')), (4, ref('project_tags_02'))]"/>
            <field name="stage_id" ref="project.project_project_stage_2"/>
        </record>

        <!-- Personal Stages: Mitchell Admin-->
        <record id="project_personal_stage_admin_0" model="project.task.type">
            <field name="sequence">1</field>
            <field name="name">Inbox</field>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="project_personal_stage_admin_1" model="project.task.type">
            <field name="sequence">2</field>
            <field name="name">Today</field>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="project_personal_stage_admin_2" model="project.task.type">
            <field name="sequence">3</field>
            <field name="name">This Week</field>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="project_personal_stage_admin_3" model="project.task.type">
            <field name="sequence">4</field>
            <field name="name">This Month</field>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="project_personal_stage_admin_4" model="project.task.type">
            <field name="sequence">5</field>
            <field name="name">Later</field>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="project_personal_stage_admin_5" model="project.task.type">
            <field name="sequence">6</field>
            <field name="name">Done</field>
            <field name="fold" eval="True"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="project_personal_stage_admin_6" model="project.task.type">
            <field name="sequence">7</field>
            <field name="name">Canceled</field>
            <field name="fold" eval="True"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>

        <!-- Personal Stages: Marc Demo -->
        <record id="project_personal_stage_demo_0" model="project.task.type">
            <field name="sequence">1</field>
            <field name="name">Inbox</field>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="project_personal_stage_demo_1" model="project.task.type">
            <field name="sequence">2</field>
            <field name="name">Today</field>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="project_personal_stage_demo_2" model="project.task.type">
            <field name="sequence">3</field>
            <field name="name">This Week</field>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="project_personal_stage_demo_3" model="project.task.type">
            <field name="sequence">4</field>
            <field name="name">This Month</field>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="project_personal_stage_demo_4" model="project.task.type">
            <field name="sequence">5</field>
            <field name="name">Later</field>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="project_personal_stage_demo_5" model="project.task.type">
            <field name="sequence">6</field>
            <field name="name">Done</field>
            <field name="fold" eval="True"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="project_personal_stage_demo_6" model="project.task.type">
            <field name="sequence">7</field>
            <field name="name">Canceled</field>
            <field name="fold" eval="True"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>

        <!-- Tasks -->
        <record id="project_task_1" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Meeting Room Furnitures</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="color">3</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_2" model="project.task">
            <field name="planned_hours" eval="32.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Lunch Room: kitchen</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_2_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_2"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=2)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_2_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_2"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=1)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_2_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_2_mail_message_1"/>
        </record>
        <record id="project_task_2_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Prpgress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_task_2_mail_message_2"/>
        </record>

        <record id="project_task_3" model="project.task">
            <field name="planned_hours" eval="10.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Noise Reduction</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-24')"/>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">4</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_3_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_3"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_3_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_3"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_3_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_3_mail_message_1"/>
        </record>
        <record id="project_task_3_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_task_3_mail_message_2"/>
        </record>

        <record id="project_task_4" model="project.task">
            <field name="planned_hours" eval="60.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Black Chairs for managers</field>
            <field name="description">Use the account_budget module</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-19')"/>
            <field name="color">5</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_01')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_4_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_4"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=1)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_4_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_4_mail_message_1"/>
        </record>

        <record id="project_task_5" model="project.task">
            <field name="planned_hours" eval="76.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Room 1: Decoration</field>
            <field name="kanban_state">done</field>
            <field name="priority">0</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-%d')"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_01')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_5_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_5"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=2)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_5_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_5_mail_message_1"/>
        </record>

        <record id="project_task_6" model="project.task">
            <field name="planned_hours" eval="24.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Room 2: Decoration</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_6_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_6"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=3)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_6_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_6_mail_message_1"/>
        </record>

        <record id="project_task_7" model="project.task">
            <field name="planned_hours" eval="15.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Energy Certificate</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_7_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_7"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_7_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_7_mail_message_1"/>
        </record>

        <record id="project_task_22" model="project.task">
            <field name="planned_hours">12.0</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin')), (4, ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Customer analysis + Architecture</field>
            <field name="color">7</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_22_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_22"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=2)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_22_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_22_mail_message_1"/>
        </record>

        <record id="project_task_12" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Planning and budget</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="color">6</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[(4, ref('project.project_task_22'))]"/>
        </record>

        <record id="project_task_8" model="project.task">
            <field name="planned_hours" eval="22.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">New portal system</field>
            <field name="priority">0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project.project_tags_02')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[(4, ref('project.project_task_12'))]"/>
        </record>
        <record id="project_task_8_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_8"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(days=3)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_8_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_8"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(days=1)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_8_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_8_mail_message_1"/>
        </record>
        <record id="project_task_8_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_task_8_mail_message_2"/>
        </record>

        <record id="project_task_10" model="project.task">
            <field name="planned_hours" eval="38.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Social network integration</field>
            <field name="kanban_state">blocked</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[(4, ref('project.project_task_12'))]"/>
        </record>
        <record id="project_task_10_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_10"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4, days=5)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_10_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_10_mail_message_1"/>
        </record>

        <record id="project_task_11" model="project.task">
            <field name="planned_hours" eval="16.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">User interface improvements</field>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project.project_tags_01'),
                    ref('project.project_tags_03')])]"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[(4, ref('project.project_task_12'))]"/>
        </record>
        <record id="project_task_11_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_11"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_11_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_11_mail_message_1"/>
        </record>

        <record id="project_task_20" model="project.task">
            <field name="planned_hours">42.0</field>
            <field name="user_ids" eval="False"/>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Create new components</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[(4, ref('project.project_task_12'))]"/>
        </record>

        <record id="project_task_21" model="project.task">
            <field name="planned_hours">14.0</field>
            <field name="user_ids" eval="[(4, ref('base.user_admin')), (4, ref('base.user_demo'))]"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Usability review</field>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_03')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids"
                   eval="[(4, ref('project.project_task_8')), (4, ref('project.project_task_10')), (4, ref('project.project_task_11')), (4, ref('project.project_task_20'))]"/>
        </record>
        <record id="project_task_21_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_21"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=1)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_21_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_21_mail_message_1"/>
        </record>

        <record id="project_task_9" model="project.task">
            <field name="planned_hours" eval="18.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin')), (4, ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Document management</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[(4, ref('project.project_task_21'))]"/>
        </record>

        <record id="project_task_19" model="project.task">
            <field name="planned_hours">24.0</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Basic outline</field>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_02')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[(4, ref('project.project_task_22'))]"/>
        </record>
        <record id="project_task_19_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_19"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_19_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_19_mail_message_1"/>
        </record>

        <record id="project_task_24" model="project.task">
            <field name="sequence">17</field>
            <field name="planned_hours">8.0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="user_ids" eval="False"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Modifications asked by the customer</field>
            <field name="tag_ids" eval="[(6, 0, [
                    ref('project_tags_00')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_24_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_24"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_24_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_24"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=3)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_24_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_24_mail_message_1"/>
        </record>
        <record id="project_task_24_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_task_24_mail_message_2"/>
        </record>

        <record id="project_task_25" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">20.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Office planning</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>

        <record id="project_task_26" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">35.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Unit Testing</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[(4, ref('project.project_task_21'))]"/>
        </record>
        <record id="project_task_26_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_26"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_26_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_task_26"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=3)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_task_26_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_task_26_mail_message_1"/>
        </record>
        <record id="project_task_26_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_task_26_mail_message_2"/>
        </record>

        <record id="project_task_30" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Entry Hall</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">3</field>
        </record>
        <record id="project_task_31" model="project.task">
            <field name="planned_hours" eval="10.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Check Lift</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-24')"/>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">4</field>
        </record>
        <record id="project_task_32" model="project.task">
            <field name="planned_hours" eval="24.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Room 1: Paint</field>
            <field name="kanban_state">done</field>
            <field name="priority">0</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-%d')"/>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="tag_ids" eval="[(6, 0, [ref('project_tags_01')])]"/>
        </record>
        <record id="project_task_33" model="project.task">
            <field name="planned_hours" eval="76.0"/>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Bathroom</field>
            <field name="stage_id" ref="project_stage_2"/>
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

        <record id="project_milestone_1" model="project.milestone">
            <field name="is_reached" eval="True"/>
            <field name="deadline" eval="time.strftime('%Y-%m-10')"/>
            <field name="name">First Phase</field>
            <field name="reached_date" eval="time.strftime('%Y-%m-10')"/>
            <field name="project_id" ref="project.project_project_1"/>
        </record>
        <record id="project_milestone_2" model="project.milestone">
            <field name="is_reached" eval="False"/>
            <field name="deadline" eval="(DateTime.now() + relativedelta(years=1)).strftime('%Y-%m-15')"/>
            <field name="name">Second Phase</field>
            <field name="project_id" ref="project.project_project_1"/>
        </record>
        <record id="project_milestone_3" model="project.milestone">
            <field name="is_reached" eval="False"/>
            <field name="deadline" eval="(DateTime.now() + relativedelta(years=2)).strftime('%Y-%m-%d')"/>
            <field name="name">Final Phase</field>
            <field name="project_id" ref="project.project_project_1"/>
        </record>

        <record id="project_update_1" model="project.update" context="{'default_project_id': ref('project.project_project_1')}">
            <field name="name">Review of the situation</field>
            <field name="user_id" eval="ref('base.user_demo')"/>
            <field name="progress" eval="15"/>
            <field name="status">at_risk</field>
        </record>
        <record id="project_update_2" model="project.update" context="{'default_project_id': ref('project.project_project_2')}">
            <field name="name">Weekly review</field>
            <field name="user_id" eval="ref('base.user_admin')"/>
            <field name="progress" eval="35"/>
            <field name="status">at_risk</field>
        </record>

        <!-- Recurring tasks and subtasks -->
        <record id="project_task_recurrence_1" model="project.task.recurrence">
            <field name="recurrence_left">3</field>
            <field name="repeat_unit">month</field>
            <field name="repeat_on_month">date</field>
            <field name="repeat_type">after</field>
            <field name="repeat_number">4</field>
            <field name="repeat_day">1</field>
            <field name="repeat_weekday">mon</field>
            <field name="create_date" eval="DateTime.now() + relativedelta(weeks=-2)"/>
        </record>
        <record id="project_task_recurrence_1" model="project.task.recurrence">
            <field name="next_recurrence_date" eval="DateTime.now() + relativedelta(weeks=-2)"/>
        </record>
        <record id="project_task_recurrence_2" model="project.task.recurrence">
            <field name="recurrence_left">20</field>
            <field name="mon" eval="True"/>
            <field name="tue" eval="True"/>
            <field name="wed" eval="True"/>
            <field name="thu" eval="True"/>
            <field name="fri" eval="True"/>
            <field name="repeat_type">after</field>
            <field name="repeat_number">20</field>
        </record>
        <record id="project_task_recurrence_2" model="project.task.recurrence">
            <field name="next_recurrence_date" eval="DateTime.now() + relativedelta(weeks=-2)"/>
        </record>
        <record id="project_task_27" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">20.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Customer review</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="recurring_task" eval="True"/>
            <field name="recurrence_id" ref="project_task_recurrence_1"/>
            <field name="create_date" eval="DateTime.now() + relativedelta(weeks=-2)"/>
        </record>
        <record id="project_task_28" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">0.25</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_task_27"/>
            <field name="name">Daily stand-up meeting - Send minutes</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="recurring_task" eval="True"/>
            <field name="recurrence_id" ref="project_task_recurrence_2"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(weeks=-1)"/>
        </record>
        <record id="project_task_29" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">8.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_task_27"/>
            <field name="name">Customer Meeting</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_task_34" model="project.task">
            <field name="sequence">10</field>
            <field name="planned_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_task_29"/>
            <field name="name">Daily Meetings summary</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_task_35" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_task_29"/>
            <field name="name">Preparation</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_task_36" model="project.task">
            <field name="sequence">30</field>
            <field name="planned_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_task_29"/>
            <field name="name">Minutes</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>

        <function model="project.task.recurrence" name="_cron_create_recurring_tasks"/>
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
                raise UserError(_('You cannot change the company of an analytic account if it is related to a project.'))

    @api.ondelete(at_uninstall=False)
    def _unlink_except_existing_tasks(self):
        projects = self.env['project.project'].search([('analytic_account_id', 'in', self.ids)])
        has_tasks = self.env['project.task'].search_count([('project_id', 'in', projects.ids)])
        if has_tasks:
            raise UserError(_('Please remove existing tasks in the project linked to the accounts you want to delete.'))

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

## File: models\analytic_account_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class AccountAnalyticTag(models.Model):
    _inherit = 'account.analytic.tag'

    task_ids = fields.Many2many('project.task', string='Tasks')

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
                ('company_id', '=', company.id),
                ('display_project_id', '!=', False),
            ])

    def _compute_kpis_actions(self, company, user):
        res = super(Digest, self)._compute_kpis_actions(company, user)
        res['kpi_project_task_opened'] = 'project.open_view_project_all&menu_id=%s' % self.env.ref('project.menu_main_pm').id
        return res

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
        if self.env.user.has_group('project.group_project_stages'):
            res.append(self.env.ref('project.menu_projects').id)
        return res

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import json
from collections import defaultdict
from datetime import timedelta, datetime
from random import randint

from odoo import api, Command, fields, models, tools, SUPERUSER_ID, _, _lt
from odoo.addons.web_editor.controllers.main import handle_history_divergence
from odoo.exceptions import UserError, ValidationError, AccessError
from odoo.tools import format_amount
from odoo.osv.expression import OR, TRUE_LEAF, FALSE_LEAF

from .project_task_recurrence import DAYS, WEEKS
from .project_update import STATUS_COLOR
from odoo.tools.misc import get_lang

PROJECT_TASK_READABLE_FIELDS = {
    'id',
    'active',
    'description',
    'priority',
    'kanban_state_label',
    'project_id',
    'display_project_id',
    'color',
    'partner_is_company',
    'commercial_partner_id',
    'allow_subtasks',
    'subtask_count',
    'child_text',
    'is_closed',
    'email_from',
    'create_date',
    'write_date',
    'company_id',
    'displayed_image_id',
    'display_name',
    'portal_user_names',
    'legend_normal',
    'legend_blocked',
    'legend_done',
    'user_ids',
}

PROJECT_TASK_WRITABLE_FIELDS = {
    'name',
    'partner_id',
    'date_deadline',
    'tag_ids',
    'sequence',
    'stage_id',
    'kanban_state',
    'child_ids',
    'parent_id',
    'priority',
}

class ProjectTaskType(models.Model):
    _name = 'project.task.type'
    _description = 'Task Stage'
    _order = 'sequence, id'

    def _get_default_project_ids(self):
        default_project_id = self.env.context.get('default_project_id')
        return [default_project_id] if default_project_id else None

    active = fields.Boolean('Active', default=True)
    name = fields.Char(string='Name', required=True, translate=True)
    description = fields.Text(translate=True)
    sequence = fields.Integer(default=1)
    project_ids = fields.Many2many('project.project', 'project_task_type_rel', 'type_id', 'project_id', string='Projects',
        default=lambda self: self._get_default_project_ids())
    legend_blocked = fields.Char(
        'Red Kanban Label', default=lambda s: _('Blocked'), translate=True, required=True,
        help='Override the default value displayed for the blocked state for kanban selection when the task or issue is in that stage.')
    legend_done = fields.Char(
        'Green Kanban Label', default=lambda s: _('Ready'), translate=True, required=True,
        help='Override the default value displayed for the done state for kanban selection when the task or issue is in that stage.')
    legend_normal = fields.Char(
        'Grey Kanban Label', default=lambda s: _('In Progress'), translate=True, required=True,
        help='Override the default value displayed for the normal state for kanban selection when the task or issue is in that stage.')
    mail_template_id = fields.Many2one(
        'mail.template',
        string='Email Template',
        domain=[('model', '=', 'project.task')],
        help="If set, an email will be sent to the customer when the task or issue reaches this step.")
    fold = fields.Boolean(string='Folded in Kanban',
        help='This stage is folded in the kanban view when there are no records in that stage to display.')
    rating_template_id = fields.Many2one(
        'mail.template',
        string='Rating Email Template',
        domain=[('model', '=', 'project.task')],
        help="If set and if the project's rating configuration is 'Rating when changing stage', then an email will be sent to the customer when the task reaches this step.")
    auto_validation_kanban_state = fields.Boolean('Automatic kanban status', default=False,
        help="Automatically modify the kanban state when the customer replies to the feedback for this stage.\n"
            " * Good feedback from the customer will update the kanban state to 'ready for the new stage' (green bullet).\n"
            " * Neutral or bad feedback will set the kanban state to 'blocked' (red bullet).\n")
    is_closed = fields.Boolean('Closing Stage', help="Tasks in this stage are considered as closed.")
    disabled_rating_warning = fields.Text(compute='_compute_disabled_rating_warning')

    user_id = fields.Many2one('res.users', 'Stage Owner', index=True)

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

    @api.constrains('user_id', 'project_ids')
    def _check_personal_stage_not_linked_to_projects(self):
        if any(stage.user_id and stage.project_ids for stage in self):
            raise UserError(_('A personal stage cannot be linked to a project because it is only visible to its corresponding user.'))

    def remove_personal_stage(self):
        """
        Remove a personal stage, tasks using that stage will move to the first
        stage with a lower priority if it exists higher if not.
        This method will not allow to delete the last personal stage.
        Having no personal_stage_type_id makes the task not appear when grouping by personal stage.
        """
        self.ensure_one()
        assert self.user_id == self.env.user or self.env.su

        users_personal_stages = self.env['project.task.type']\
            .search([('user_id', '=', self.user_id.id)], order='sequence DESC')
        if len(users_personal_stages) == 1:
            raise ValidationError(_("You should at least have one personal stage. Create a new stage to which the tasks can be transferred after this one is deleted."))

        # Find the most suitable stage, they are already sorted by sequence
        new_stage = self.env['project.task.type']
        for stage in users_personal_stages:
            if stage == self:
                continue
            if stage.sequence > self.sequence:
                new_stage = stage
            elif stage.sequence <= self.sequence:
                new_stage = stage
                break

        self.env['project.task.stage.personal'].search([('stage_id', '=', self.id)]).write({
            'stage_id': new_stage.id,
        })
        self.unlink()

class Project(models.Model):
    _name = "project.project"
    _description = "Project"
    _inherit = ['portal.mixin', 'mail.alias.mixin', 'mail.thread', 'mail.activity.mixin', 'rating.parent.mixin']
    _order = "sequence, name, id"
    _rating_satisfaction_days = False  # takes all existing ratings
    _check_company_auto = True

    def _compute_attached_docs_count(self):
        docs_count = {}
        if self.ids:
            self.env.cr.execute(
                """
                WITH docs AS (
                     SELECT res_id as id, count(*) as count
                       FROM ir_attachment
                      WHERE res_model = 'project.project'
                        AND res_id IN %(project_ids)s
                   GROUP BY res_id

                  UNION ALL

                     SELECT t.project_id as id, count(*) as count
                       FROM ir_attachment a
                       JOIN project_task t ON a.res_model = 'project.task' AND a.res_id = t.id
                      WHERE t.project_id IN %(project_ids)s
                   GROUP BY t.project_id
                )
                SELECT id, sum(count)
                  FROM docs
              GROUP BY id
                """,
                {"project_ids": tuple(self.ids)}
            )
            docs_count = dict(self.env.cr.fetchall())
        for project in self:
            project.doc_count = docs_count.get(project.id, 0)

    def _compute_task_count(self):
        task_data = self.env['project.task'].read_group(
            [('project_id', 'in', self.ids),
             '|',
                ('stage_id.fold', '=', False),
                ('stage_id', '=', False)],
            ['project_id', 'display_project_id:count'], ['project_id'])
        result_wo_subtask = defaultdict(int)
        result_with_subtasks = defaultdict(int)
        for data in task_data:
            result_wo_subtask[data['project_id'][0]] += data['display_project_id']
            result_with_subtasks[data['project_id'][0]] += data['project_id_count']
        for project in self:
            project.task_count = result_wo_subtask[project.id]
            project.task_count_with_subtasks = result_with_subtasks[project.id]

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

    def _default_stage_id(self):
        # Since project stages are order by sequence first, this should fetch the one with the lowest sequence number.
        return self.env['project.project.stage'].search([], limit=1)

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

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        return self.env['project.project.stage'].search([], order=order)

    name = fields.Char("Name", index=True, required=True, tracking=True, translate=True)
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
    commercial_partner_id = fields.Many2one(related="partner_id.commercial_partner_id")
    company_id = fields.Many2one('res.company', string='Company', required=True, default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', related="company_id.currency_id", string="Currency", readonly=True)
    analytic_account_id = fields.Many2one('account.analytic.account', string="Analytic Account", copy=False, ondelete='set null',
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", check_company=True,
        help="Analytic account to which this project is linked for financial management. "
             "Use an analytic account to record cost and revenue on your project.")
    analytic_account_balance = fields.Monetary(related="analytic_account_id.balance")

    favorite_user_ids = fields.Many2many(
        'res.users', 'project_favorite_user_rel', 'project_id', 'user_id',
        default=_get_default_favorite_user_ids,
        string='Members')
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite',
        string='Show Project on Dashboard',
        help="Whether this project should be displayed on your dashboard.")
    label_tasks = fields.Char(string='Use Tasks as', default=lambda s: _('Tasks'), help="Label used for the tasks of the project.", translate=True)
    tasks = fields.One2many('project.task', 'project_id', string="Task Activities")
    resource_calendar_id = fields.Many2one(
        'resource.calendar', string='Working Time',
        related='company_id.resource_calendar_id')
    type_ids = fields.Many2many('project.task.type', 'project_task_type_rel', 'project_id', 'type_id', string='Tasks Stages')
    task_count = fields.Integer(compute='_compute_task_count', string="Task Count")
    task_count_with_subtasks = fields.Integer(compute='_compute_task_count')
    task_ids = fields.One2many('project.task', 'project_id', string='Tasks',
                               domain=['|', ('stage_id.fold', '=', False), ('stage_id', '=', False)])
    color = fields.Integer(string='Color Index')
    user_id = fields.Many2one('res.users', string='Project Manager', default=lambda self: self.env.user, tracking=True)
    alias_enabled = fields.Boolean(string='Use Email Alias', compute='_compute_alias_enabled', readonly=False)
    alias_id = fields.Many2one('mail.alias', string='Alias', ondelete="restrict", required=True,
        help="Internal email associated with this project. Incoming emails are automatically synchronized "
             "with Tasks (or optionally Issues if the Issue Tracker module is installed).")
    alias_value = fields.Char(string='Alias email', compute='_compute_alias_value')
    privacy_visibility = fields.Selection([
            ('followers', 'Invited employees'),
            ('employees', 'All employees'),
            ('portal', 'Invited portal users and all employees'),
        ],
        string='Visibility', required=True,
        default='portal',
        help="People to whom this project and its tasks will be visible.\n\n"
            "- Invited internal users: when following a project, internal users will get access to all of its tasks without distinction. "
            "Otherwise, they will only get access to the specific tasks they are following.\n "
            "A user with the project > administrator access right level can still access this project and its tasks, even if they are not explicitly part of the followers.\n\n"
            "- All internal users: all internal users can access the project and all of its tasks without distinction.\n\n"
            "- Invited portal users and all internal users: all internal users can access the project and all of its tasks without distinction.\n"
            "When following a project, portal users will get access to all of its tasks without distinction. Otherwise, they will only get access to the specific tasks they are following.\n\n"
            "When a project is shared in read-only, the portal user is redirected to their portal. They can view the tasks, but not edit them.\n"
            "When a project is shared in edit, the portal user is redirected to the kanban and list views of the tasks. They can modify a selected number of fields on the tasks.\n\n"
            "In any case, an internal user with no project access rights can still access a task, "
            "provided that they are given the corresponding URL (and that they are part of the followers if the project is private).")
    doc_count = fields.Integer(compute='_compute_attached_docs_count', string="Number of documents attached")
    date_start = fields.Date(string='Start Date')
    date = fields.Date(string='Expiration Date', index=True, tracking=True)
    allow_subtasks = fields.Boolean('Sub-tasks', default=lambda self: self.env.user.has_group('project.group_subtask_project'))
    allow_recurring_tasks = fields.Boolean('Recurring Tasks', default=lambda self: self.env.user.has_group('project.group_project_recurring_tasks'))
    allow_task_dependencies = fields.Boolean('Task Dependencies', default=lambda self: self.env.user.has_group('project.group_project_task_dependencies'))
    tag_ids = fields.Many2many('project.tags', relation='project_project_project_tags_rel', string='Tags')

    # Project Sharing fields
    collaborator_ids = fields.One2many('project.collaborator', 'project_id', string='Collaborators', copy=False)
    collaborator_count = fields.Integer('# Collaborators', compute='_compute_collaborator_count', compute_sudo=True)

    # rating fields
    rating_request_deadline = fields.Datetime(compute='_compute_rating_request_deadline', store=True)
    rating_active = fields.Boolean('Customer Ratings', default=lambda self: self.env.user.has_group('project.group_project_rating'))
    rating_status = fields.Selection(
        [('stage', 'Rating when changing stage'),
         ('periodic', 'Periodic rating')
        ], 'Customer Ratings Status', default="stage", required=True,
        help="How to get customer feedback?\n"
             "- Rating when changing stage: an email will be sent when a task is pulled to another stage.\n"
             "- Periodic rating: an email will be sent periodically.\n\n"
             "Don't forget to set up the email templates on the stages for which you want to get customer feedback.")
    rating_status_period = fields.Selection([
        ('daily', 'Daily'),
        ('weekly', 'Weekly'),
        ('bimonthly', 'Twice a Month'),
        ('monthly', 'Once a Month'),
        ('quarterly', 'Quarterly'),
        ('yearly', 'Yearly')], 'Rating Frequency', required=True, default='monthly')

    # Not `required` since this is an option to enable in project settings.
    stage_id = fields.Many2one('project.project.stage', string='Stage', ondelete='restrict', groups="project.group_project_stages",
        tracking=True, index=True, copy=False, default=_default_stage_id, group_expand='_read_group_stage_ids')

    update_ids = fields.One2many('project.update', 'project_id')
    last_update_id = fields.Many2one('project.update', string='Last Update', copy=False)
    last_update_status = fields.Selection(selection=[
        ('on_track', 'On Track'),
        ('at_risk', 'At Risk'),
        ('off_track', 'Off Track'),
        ('on_hold', 'On Hold')
    ], default='on_track', compute='_compute_last_update_status', store=True)
    last_update_color = fields.Integer(compute='_compute_last_update_color')
    milestone_ids = fields.One2many('project.milestone', 'project_id')
    milestone_count = fields.Integer(compute='_compute_milestone_count')

    _sql_constraints = [
        ('project_date_greater', 'check(date >= date_start)', 'Error! Project start date must be before project end date.')
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

    @api.depends('last_update_id.status')
    def _compute_last_update_status(self):
        for project in self:
            project.last_update_status = project.last_update_id.status or 'on_track'

    @api.depends('last_update_status')
    def _compute_last_update_color(self):
        for project in self:
            project.last_update_color = STATUS_COLOR[project.last_update_status]

    @api.depends('milestone_ids')
    def _compute_milestone_count(self):
        read_group = self.env['project.milestone'].read_group([('project_id', 'in', self.ids)], ['project_id'], ['project_id'])
        mapped_count = {group['project_id'][0]: group['project_id_count'] for group in read_group}
        for project in self:
            project.milestone_count = mapped_count.get(project.id, 0)

    @api.depends('alias_name', 'alias_domain')
    def _compute_alias_value(self):
        for project in self:
            if not project.alias_name or not project.alias_domain:
                project.alias_value = ''
            else:
                project.alias_value = "%s@%s" % (project.alias_name, project.alias_domain)

    @api.depends('collaborator_ids', 'privacy_visibility')
    def _compute_collaborator_count(self):
        project_sharings = self.filtered(lambda project: project.privacy_visibility == 'portal')
        collaborator_read_group = self.env['project.collaborator'].read_group(
            [('project_id', 'in', project_sharings.ids)],
            ['project_id'],
            ['project_id'],
        )
        collaborator_count_by_project = {res['project_id'][0]: res['project_id_count'] for res in collaborator_read_group}
        for project in self:
            project.collaborator_count = collaborator_count_by_project.get(project.id, 0)

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
        self.ensure_one()
        project = self.browse(new_project_id)
        tasks = self.env['project.task']
        # We want to copy archived task, but do not propagate an active_test context key
        task_ids = self.env['project.task'].with_context(active_test=False).search([('project_id', '=', self.id)], order='parent_id').ids
        old_to_new_tasks = {}
        all_tasks = self.env['project.task'].browse(task_ids)
        for task in all_tasks:
            # preserve task name and stage, normally altered during copy
            defaults = self._map_tasks_default_valeus(task, project)
            if task.parent_id:
                # set the parent to the duplicated task
                parent_id = old_to_new_tasks.get(task.parent_id.id, False)
                defaults['parent_id'] = parent_id
                if not parent_id or task.display_project_id:
                    defaults['project_id'] = project.id if task.display_project_id == self else False
                    defaults['display_project_id'] = project.id if task.display_project_id == self else False
            elif task.display_project_id == self:
                defaults['project_id'] = project.id
                defaults['display_project_id'] = project.id
            new_task = task.copy(defaults)
            # If child are created before parent (ex sub_sub_tasks)
            new_child_ids = [old_to_new_tasks[child.id] for child in task.child_ids if child.id in old_to_new_tasks]
            tasks.browse(new_child_ids).write({'parent_id': new_task.id})
            old_to_new_tasks[task.id] = new_task.id
            if task.allow_task_dependencies:
                depend_on_ids = [t.id if t not in all_tasks else old_to_new_tasks.get(t.id, None) for t in
                                 task.depend_on_ids]
                new_task.write({'depend_on_ids': [Command.link(tid) for tid in depend_on_ids if tid]})
                dependent_ids = [t.id if t not in all_tasks else old_to_new_tasks.get(t.id, None) for t in
                                 task.dependent_ids]
                new_task.write({'dependent_ids': [Command.link(tid) for tid in dependent_ids if tid]})
            tasks += new_task

        return project.write({'tasks': [(6, 0, tasks.ids)]})

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default is None:
            default = {}
        if not default.get('name'):
            default['name'] = _("%s (copy)") % (self.name)
        project = super(Project, self).copy(default)
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
        if project.privacy_visibility == 'portal' and project.partner_id:
            project.message_subscribe(project.partner_id.ids)
        return project

    def write(self, vals):
        # directly compute is_favorite to dodge allow write access right
        if 'is_favorite' in vals:
            vals.pop('is_favorite')
            self._fields['is_favorite'].determine_inverse(self)
        res = super(Project, self).write(vals) if vals else True

        if 'allow_recurring_tasks' in vals and not vals.get('allow_recurring_tasks'):
            self.env['project.task'].search([('project_id', 'in', self.ids), ('recurring_task', '=', True)]).write({'recurring_task': False})

        if 'active' in vals:
            # archiving/unarchiving a project does it on its tasks, too
            self.with_context(active_test=False).mapped('tasks').write({'active': vals['active']})
        if vals.get('partner_id') or vals.get('privacy_visibility'):
            for project in self.filtered(lambda project: project.privacy_visibility == 'portal'):
                project.message_subscribe(project.partner_id.ids)
        if vals.get('privacy_visibility'):
            self._change_privacy_visibility()
        if 'name' in vals and self.analytic_account_id:
            projects_read_group = self.env['project.project'].read_group(
                [('analytic_account_id', 'in', self.analytic_account_id.ids)],
                ['analytic_account_id'],
                ['analytic_account_id']
            )
            analytic_account_to_update = self.env['account.analytic.account'].browse([
                res['analytic_account_id'][0]
                for res in projects_read_group
                if res['analytic_account_id'] and res['analytic_account_id_count'] == 1
            ])
            analytic_account_to_update.write({'name': self.name})
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

    @api.ondelete(at_uninstall=False)
    def _unlink_except_contains_tasks(self):
        # Check project is empty
        for project in self.with_context(active_test=False):
            if project.tasks:
                raise UserError(_('You cannot delete a project containing tasks. You can either archive it or first delete all of its tasks.'))

    def unlink(self):
        # Delete the empty related analytic account
        analytic_accounts_to_delete = self.env['account.analytic.account']
        for project in self:
            if project.analytic_account_id and not project.analytic_account_id.line_ids:
                analytic_accounts_to_delete |= project.analytic_account_id
        result = super(Project, self).unlink()
        analytic_accounts_to_delete.unlink()
        return result

    def message_subscribe(self, partner_ids=None, subtype_ids=None):
        """
        Subscribe to all existing active tasks when subscribing to a project
        """
        res = super(Project, self).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)
        project_subtypes = self.env['mail.message.subtype'].browse(subtype_ids) if subtype_ids else None
        task_subtypes = (project_subtypes.mapped('parent_id') | project_subtypes.filtered(lambda sub: sub.internal or sub.default)).ids if project_subtypes else None
        if not subtype_ids or task_subtypes:
            self.mapped('tasks').message_subscribe(
                partner_ids=partner_ids, subtype_ids=task_subtypes)
        return res

    def message_unsubscribe(self, partner_ids=None):
        """ Unsubscribe from all tasks when unsubscribing from a project """
        self.mapped('tasks').message_unsubscribe(partner_ids=partner_ids)
        return super(Project, self).message_unsubscribe(partner_ids=partner_ids)

    def _alias_get_creation_values(self):
        values = super(Project, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('project.task').id
        if self.id:
            values['alias_defaults'] = defaults = ast.literal_eval(self.alias_defaults or "{}")
            defaults['project_id'] = self.id
        return values

    # ---------------------------------------------------
    # Mail gateway
    # ---------------------------------------------------

    def _track_template(self, changes):
        res = super()._track_template(changes)
        project = self[0]
        if self.user_has_groups('project.group_project_stages') and 'stage_id' in changes and project.stage_id.mail_template_id:
            res['stage_id'] = (project.stage_id.mail_template_id, {
                'auto_delete_message': True,
                'subtype_id': self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note'),
                'email_layout_xmlid': 'mail.mail_notification_light',
            })
        return res

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'stage_id' in init_values:
            return self.env.ref('project.mt_project_stage_change')
        return super()._track_subtype(init_values)

    def _mail_get_message_subtypes(self):
        res = super()._mail_get_message_subtypes()
        if len(self) == 1:
            dependency_subtype = self.env.ref('project.mt_project_task_dependency_change')
            if not self.allow_task_dependencies and dependency_subtype in res:
                res -= dependency_subtype
        return res

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

    def action_view_all_rating(self):
        """ return the action to see all the rating of the project and activate default filters"""
        action = self.env['ir.actions.act_window']._for_xml_id('project.rating_rating_action_view_project_rating')
        action['name'] = _('Ratings of %s') % (self.name,)
        action_context = ast.literal_eval(action['context']) if action['context'] else {}
        action_context.update(self._context)
        action_context['search_default_parent_res_name'] = self.name
        action_context.pop('group_by', None)
        return dict(action, context=action_context)

    def action_view_tasks_analysis(self):
        """ return the action to see the tasks analysis report of the project """
        action = self.env['ir.actions.act_window']._for_xml_id('project.action_project_task_user_tree')
        action_context = ast.literal_eval(action['context']) if action['context'] else {}
        action_context['search_default_project_id'] = self.id
        return dict(action, context=action_context)

    @api.model
    def _action_open_all_projects(self):
        action = self.env['ir.actions.act_window']._for_xml_id(
            'project.open_view_project_all' if not self.user_has_groups('project.group_project_stages') else
            'project.open_view_project_all_group_stage')
        return action

    def action_view_analytic_account_entries(self):
        self.ensure_one()
        return {
            'res_model': 'account.analytic.line',
            'type': 'ir.actions.act_window',
            'name': _("Gross Margin"),
            'domain': [('account_id', '=', self.analytic_account_id.id)],
            'views': [(self.env.ref('analytic.view_account_analytic_line_tree').id, 'list'),
                      (self.env.ref('analytic.view_account_analytic_line_form').id, 'form'),
                      (self.env.ref('analytic.view_account_analytic_line_graph').id, 'graph'),
                      (self.env.ref('analytic.view_account_analytic_line_pivot').id, 'pivot')],
            'view_mode': 'tree,form,graph,pivot',
            'context': {'search_default_group_date': 1, 'default_account_id': self.analytic_account_id.id}
        }

    def action_view_kanban_project(self):
        # [XBO] TODO: remove me in master
        return

    # ---------------------------------------------
    #  PROJECT UPDATES
    # ---------------------------------------------

    def get_last_update_or_default(self):
        self.ensure_one()
        labels = dict(self._fields['last_update_status']._description_selection(self.env))
        return {
            'status': labels[self.last_update_status],
            'color': self.last_update_color,
        }

    def get_panel_data(self):
        self.ensure_one()
        return {
            'user': self._get_user_values(),
            'tasks_analysis': self._get_tasks_analysis(),
            'milestones': self._get_milestones(),
            'buttons': sorted(self._get_stat_buttons(), key=lambda k: k['sequence']),
        }

    def _get_user_values(self):
        return {
            'is_project_user': self.user_has_groups('project.group_project_user'),
        }

    def _get_tasks_analysis(self):
        # Deprecated
        return {
            'data': [],
        }

    def _get_milestones(self):
        self.ensure_one()
        return {
            'data': self.milestone_ids._get_data_list(),
        }

    def _get_stat_buttons(self):
        self.ensure_one()
        buttons = [{
            'icon': 'tasks',
            'text': _lt('Tasks'),
            'number': self.task_count,
            'action_type': 'action',
            'action': 'project.act_project_project_2_project_task_all',
            'additional_context': json.dumps({
                'active_id': self.id,
            }),
            'show': True,
            'sequence': 2,
        }]
        if self.user_has_groups('project.group_project_rating'):
            buttons.append({
                'icon': 'smile-o',
                'text': _lt('Customer Satisfaction'),
                'number': '%s %%' % (self.rating_percentage_satisfaction),
                'action_type': 'object',
                'action': 'action_view_all_rating',
                'show': self.rating_active and self.rating_percentage_satisfaction > -1,
                'sequence': 5,
            })
        if self.user_has_groups('project.group_project_manager'):
            buttons.append({
                'icon': 'area-chart',
                'text': _lt('Burndown Chart'),
                'action_type': 'action',
                'action': 'project.action_project_task_burndown_chart_report',
                'additional_context': json.dumps({
                    'active_id': self.id,
                }),
                'show': True,
                'sequence': 7,
            })
            buttons.append({
                'icon': 'users',
                'text': _lt('Collaborators'),
                'number': self.collaborator_count,
                'action_type': 'action',
                'action': 'project.project_collaborator_action',
                'additional_context': json.dumps({
                    'active_id': self.id,
                }),
                'show': True,
                'sequence': 23,
            })
        if self.user_has_groups('analytic.group_analytic_accounting'):
            buttons.append({
                'icon': 'usd',
                'text': _lt('Gross Margin'),
                'number': format_amount(self.env, self.analytic_account_balance, self.company_id.currency_id),
                'action_type': 'object',
                'action': 'action_view_analytic_account_entries',
                'show': bool(self.analytic_account_id),
                'sequence': 18,
            })
        return buttons

    def _get_tasks_analysis_counts(self, created=False, updated=False):
        # Deprecated
        return {}

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

    # ---------------------------------------------------
    # Privacy
    # ---------------------------------------------------

    def _change_privacy_visibility(self):
        """
        Unsubscribe non-internal users from the project and tasks if the project privacy visibility
        goes to a value different than 'portal'
        """
        for project in self.filtered(lambda p: p.privacy_visibility != 'portal'):
            portal_users = project.message_partner_ids.user_ids.filtered('share')
            project.message_unsubscribe(partner_ids=portal_users.partner_id.ids)
            project.mapped('tasks')._change_project_privacy_visibility()

    # ---------------------------------------------------
    # Project sharing
    # ---------------------------------------------------
    def _check_project_sharing_access(self):
        self.ensure_one()
        if self.privacy_visibility != 'portal':
            return False
        if self.env.user.has_group('base.group_portal'):
            return self.env['project.collaborator'].search([('project_id', '=', self.sudo().id), ('partner_id', '=', self.env.user.partner_id.id)])
        return self.env.user.has_group('base.group_user')

    def _add_collaborators(self, partners):
        self.ensure_one()
        user_group_id = self.env['ir.model.data']._xmlid_to_res_id('base.group_user')
        all_collaborators = self.collaborator_ids.partner_id
        new_collaborators = partners.filtered(
            lambda partner:
                partner not in all_collaborators
                and (not partner.user_ids or user_group_id not in partner.user_ids[0].groups_id.ids)
        )
        if not new_collaborators:
            # Then we have nothing to do
            return
        self.write({'collaborator_ids': [
            Command.create({
                'partner_id': collaborator.id,
            }) for collaborator in new_collaborators],
        })

class Task(models.Model):
    _name = "project.task"
    _description = "Task"
    _date_name = "date_assign"
    _inherit = ['portal.mixin', 'mail.thread.cc', 'mail.activity.mixin', 'rating.mixin']
    _mail_post_access = 'read'
    _order = "priority desc, sequence, id desc"
    _check_company_auto = True

    @api.model
    def _get_default_partner_id(self, project=None, parent=None):
        if parent and parent.partner_id:
            return parent.partner_id.id
        if project and project.partner_id:
            return project.partner_id.id
        return False

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

    @api.model
    def _read_group_personal_stage_type_ids(self, stages, domain, order):
        return stages.search(['|', ('id', 'in', stages.ids), ('user_id', '=', self.env.user.id)])

    active = fields.Boolean(default=True)
    name = fields.Char(string='Title', tracking=True, required=True, index=True)
    description = fields.Html(string='Description', sanitize_attributes=False)
    priority = fields.Selection([
        ('0', 'Normal'),
        ('1', 'Important'),
    ], default='0', index=True, string="Starred", tracking=True)
    sequence = fields.Integer(string='Sequence', index=True, default=10,
        help="Gives the sequence order when displaying a list of tasks.")
    stage_id = fields.Many2one('project.task.type', string='Stage', compute='_compute_stage_id',
        store=True, readonly=False, ondelete='restrict', tracking=True, index=True,
        default=_get_default_stage_id, group_expand='_read_group_stage_ids',
        domain="[('project_ids', '=', project_id)]", copy=False, task_dependency_tracking=True)
    tag_ids = fields.Many2many('project.tags', string='Tags')
    kanban_state = fields.Selection([
        ('normal', 'In Progress'),
        ('done', 'Ready'),
        ('blocked', 'Blocked')], string='Status',
        copy=False, default='normal', required=True)
    kanban_state_label = fields.Char(compute='_compute_kanban_state_label', string='Kanban State Label', tracking=True, task_dependency_tracking=True)
    create_date = fields.Datetime("Created On", readonly=True, index=True)
    write_date = fields.Datetime("Last Updated On", readonly=True, index=True)
    date_end = fields.Datetime(string='Ending Date', index=True, copy=False)
    date_assign = fields.Datetime(string='Assigning Date', index=True, copy=False, readonly=True)
    date_deadline = fields.Date(string='Deadline', index=True, copy=False, tracking=True, task_dependency_tracking=True)
    date_last_stage_update = fields.Datetime(string='Last Stage Update',
        index=True,
        copy=False,
        readonly=True)
    project_id = fields.Many2one('project.project', string='Project',
        compute='_compute_project_id', recursive=True, store=True, readonly=False,
        index=True, tracking=True, check_company=True, change_default=True)
    # Defines in which project the task will be displayed / taken into account in statistics.
    # Example: 1 task A with 1 subtask B in project P
    # A -> project_id=P, display_project_id=P
    # B -> project_id=P (to inherit from ACL/security rules), display_project_id=False
    display_project_id = fields.Many2one('project.project', index=True)
    planned_hours = fields.Float("Initially Planned Hours", help='Time planned to achieve this task (including its sub-tasks).', tracking=True)
    subtask_planned_hours = fields.Float("Sub-tasks Planned Hours", compute='_compute_subtask_planned_hours',
        help="Sum of the time planned of all the sub-tasks linked to this task. Usually less than or equal to the initially planned time of this task.")
    # Tracking of this field is done in the write function
    user_ids = fields.Many2many('res.users', relation='project_task_user_rel', column1='task_id', column2='user_id',
        string='Assignees', default=lambda self: not self.env.user.share and self.env.user, context={'active_test': False}, tracking=True)
    # User names displayed in project sharing views
    portal_user_names = fields.Char(compute='_compute_portal_user_names', compute_sudo=True, search='_search_portal_user_names')
    # Second Many2many containing the actual personal stage for the current user
    # See project_task_stage_personal.py for the model defininition
    personal_stage_type_ids = fields.Many2many('project.task.type', 'project_task_user_rel', column1='task_id', column2='stage_id',
        ondelete='restrict', group_expand='_read_group_personal_stage_type_ids', copy=False,
        domain="[('user_id', '=', user.id)]", depends=['user_ids'], string='Personal Stage')
    # Personal Stage computed from the user
    personal_stage_id = fields.Many2one('project.task.stage.personal', string='Personal Stage State', compute_sudo=False,
        compute='_compute_personal_stage_id', help="The current user's personal stage.")
    # This field is actually a related field on personal_stage_id.stage_id
    # However due to the fact that personal_stage_id is computed, the orm throws out errors
    # saying the field cannot be searched.
    personal_stage_type_id = fields.Many2one('project.task.type', string='Personal User Stage',
        compute='_compute_personal_stage_type_id', inverse='_inverse_personal_stage_type_id', store=False,
        search='_search_personal_stage_type_id',
        help="The current user's personal task stage.")
    partner_id = fields.Many2one('res.partner',
        string='Customer',
        compute='_compute_partner_id', recursive=True, store=True, readonly=False, tracking=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    partner_is_company = fields.Boolean(related='partner_id.is_company', readonly=True)
    commercial_partner_id = fields.Many2one(related='partner_id.commercial_partner_id')
    partner_email = fields.Char(
        compute='_compute_partner_email', inverse='_inverse_partner_email',
        string='Email', readonly=False, store=True, copy=False)
    partner_phone = fields.Char(
        compute='_compute_partner_phone', inverse='_inverse_partner_phone',
        string="Phone", readonly=False, store=True, copy=False)
    partner_city = fields.Char(related='partner_id.city', readonly=False)
    manager_id = fields.Many2one('res.users', string='Project Manager', related='project_id.user_id', readonly=True)
    company_id = fields.Many2one(
        'res.company', string='Company', compute='_compute_company_id', store=True, readonly=False,
        required=True, copy=True, default=_default_company_id)
    color = fields.Integer(string='Color Index')
    attachment_ids = fields.One2many('ir.attachment', compute='_compute_attachment_ids', string="Main Attachments",
        help="Attachments that don't come from a message.")
    # In the domain of displayed_image_id, we couln't use attachment_ids because a one2many is represented as a list of commands so we used res_model & res_id
    displayed_image_id = fields.Many2one('ir.attachment', domain="[('res_model', '=', 'project.task'), ('res_id', '=', id), ('mimetype', 'ilike', 'image')]", string='Cover Image')
    legend_blocked = fields.Char(related='stage_id.legend_blocked', string='Kanban Blocked Explanation', readonly=True, related_sudo=False)
    legend_done = fields.Char(related='stage_id.legend_done', string='Kanban Valid Explanation', readonly=True, related_sudo=False)
    legend_normal = fields.Char(related='stage_id.legend_normal', string='Kanban Ongoing Explanation', readonly=True, related_sudo=False)
    is_closed = fields.Boolean(related="stage_id.is_closed", string="Closing Stage", readonly=True, related_sudo=False)
    parent_id = fields.Many2one('project.task', string='Parent Task', index=True)
    child_ids = fields.One2many('project.task', 'parent_id', string="Sub-tasks")
    child_text = fields.Char(compute="_compute_child_text")
    allow_subtasks = fields.Boolean(string="Allow Sub-tasks", related="project_id.allow_subtasks", readonly=True)
    subtask_count = fields.Integer("Sub-task Count", compute='_compute_subtask_count')
    email_from = fields.Char(string='Email From', help="These people will receive email.", index=True,
        compute='_compute_email_from', recursive=True, store=True, readonly=False, copy=False)
    project_privacy_visibility = fields.Selection(related='project_id.privacy_visibility', string="Project Visibility")
    # Computed field about working time elapsed between record creation and assignation/closing.
    working_hours_open = fields.Float(compute='_compute_elapsed', string='Working Hours to Assign', digits=(16, 2), store=True, group_operator="avg")
    working_hours_close = fields.Float(compute='_compute_elapsed', string='Working Hours to Close', digits=(16, 2), store=True, group_operator="avg")
    working_days_open = fields.Float(compute='_compute_elapsed', string='Working Days to Assign', store=True, group_operator="avg")
    working_days_close = fields.Float(compute='_compute_elapsed', string='Working Days to Close', store=True, group_operator="avg")
    # customer portal: include comment and incoming emails in communication history
    website_message_ids = fields.One2many(domain=lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment'])])
    is_private = fields.Boolean(compute='_compute_is_private')

    # Task Dependencies fields
    allow_task_dependencies = fields.Boolean(related='project_id.allow_task_dependencies')
    # Tracking of this field is done in the write function
    depend_on_ids = fields.Many2many('project.task', relation="task_dependencies_rel", column1="task_id",
                                     column2="depends_on_id", string="Blocked By", tracking=True, copy=False,
                                     domain="[('allow_task_dependencies', '=', True), ('id', '!=', id)]")
    dependent_ids = fields.Many2many('project.task', relation="task_dependencies_rel", column1="depends_on_id",
                                     column2="task_id", string="Block", copy=False,
                                     domain="[('allow_task_dependencies', '=', True), ('id', '!=', id)]")
    dependent_tasks_count = fields.Integer(string="Dependent Tasks", compute='_compute_dependent_tasks_count')

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

    # Account analytic
    analytic_account_id = fields.Many2one('account.analytic.account', ondelete='set null',
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", check_company=True,
        help="Analytic account to which this task is linked for financial management. "
             "Use an analytic account to record cost and revenue on your task. "
             "If empty, the analytic account of the project will be used.")
    project_analytic_account_id = fields.Many2one('account.analytic.account', string='Project Analytic Account', related='project_id.analytic_account_id')
    analytic_tag_ids = fields.Many2many('account.analytic.tag',
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", check_company=True)

    @property
    def SELF_READABLE_FIELDS(self):
        return PROJECT_TASK_READABLE_FIELDS | self.SELF_WRITABLE_FIELDS

    @property
    def SELF_WRITABLE_FIELDS(self):
        return PROJECT_TASK_WRITABLE_FIELDS

    @api.depends('project_id', 'parent_id')
    def _compute_is_private(self):
        # Modify accordingly, this field is used to display the lock on the task's kanban card
        for task in self:
            task.is_private = not task.project_id and not task.parent_id

    @api.depends_context('uid')
    @api.depends('user_ids')
    def _compute_personal_stage_id(self):
        # An user may only access his own 'personal stage' and there can only be one pair (user, task_id)
        personal_stages = self.env['project.task.stage.personal'].search([('user_id', '=', self.env.uid), ('task_id', 'in', self.ids)])
        self.personal_stage_id = False
        for personal_stage in personal_stages:
            personal_stage.task_id.personal_stage_id = personal_stage

    @api.depends('personal_stage_id')
    def _compute_personal_stage_type_id(self):
        for task in self:
            task.personal_stage_type_id = task.personal_stage_id.stage_id

    def _inverse_personal_stage_type_id(self):
        for task in self:
            task.personal_stage_id.stage_id = task.personal_stage_type_id

    @api.model
    def _search_personal_stage_type_id(self, operator, value):
        return [('personal_stage_type_ids', operator, value)]

    @api.model
    def _get_default_personal_stage_create_vals(self, user_id):
        return [
            {'sequence': 1, 'name': _('Inbox'), 'user_id': user_id, 'fold': False},
            {'sequence': 2, 'name': _('Today'), 'user_id': user_id, 'fold': False},
            {'sequence': 3, 'name': _('This Week'), 'user_id': user_id, 'fold': False},
            {'sequence': 4, 'name': _('This Month'), 'user_id': user_id, 'fold': False},
            {'sequence': 5, 'name': _('Later'), 'user_id': user_id, 'fold': False},
            {'sequence': 6, 'name': _('Done'), 'user_id': user_id, 'fold': True},
            {'sequence': 7, 'name': _('Canceled'), 'user_id': user_id, 'fold': True},
        ]

    def _populate_missing_personal_stages(self):
        # Assign the default personal stage for those that are missing
        personal_stages_without_stage = self.env['project.task.stage.personal'].sudo().search([('task_id', 'in', self.ids), ('stage_id', '=', False)])
        if personal_stages_without_stage:
            user_ids = personal_stages_without_stage.user_id
            personal_stage_by_user = defaultdict(lambda: self.env['project.task.stage.personal'])
            for personal_stage in personal_stages_without_stage:
                personal_stage_by_user[personal_stage.user_id] |= personal_stage
            for user_id in user_ids:
                stage = self.env['project.task.type'].sudo().search([('user_id', '=', user_id.id)], limit=1)
                # In the case no stages have been found, we create the default stages for the user
                if not stage:
                    stages = self.env['project.task.type'].sudo().with_context(lang=user_id.partner_id.lang, default_project_ids=False).create(
                        self.with_context(lang=user_id.partner_id.lang)._get_default_personal_stage_create_vals(user_id.id)
                    )
                    stage = stages[0]
                personal_stage_by_user[user_id].sudo().write({'stage_id': stage.id})

    @api.constrains('depend_on_ids')
    def _check_no_cyclic_dependencies(self):
        if not self._check_m2m_recursion('depend_on_ids'):
            raise ValidationError(_("You cannot create cyclic dependency."))

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
            recurrence_left = task.recurrence_id.recurrence_left if task.recurrence_id  else task.repeat_number
            number_occurrences = min(5, recurrence_left if task.repeat_type == 'after' else 5)
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
            if recurrence_left == 0:
                recurrence_title = _('There are no more occurrences.')
            else:
                recurrence_title = _('Next Occurrences:')
            task.recurrence_message = '<p><span class="fa fa-check-circle"></span> %s</p><ul>' % recurrence_title
            task.recurrence_message += ''.join(['<li>%s</li>' % date.strftime(date_format) for date in recurring_dates[:5]])
            if task.repeat_type == 'after' and recurrence_left > 5 or task.repeat_type == 'forever' or len(recurring_dates) > 5:
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

    @api.depends('dependent_ids')
    def _compute_dependent_tasks_count(self):
        tasks_with_dependency = self.filtered('allow_task_dependencies')
        (self - tasks_with_dependency).dependent_tasks_count = 0
        if tasks_with_dependency:
            group_dependent = self.env['project.task'].read_group([
                ('depend_on_ids', 'in', tasks_with_dependency.ids),
            ], ['depend_on_ids'], ['depend_on_ids'])
            dependent_tasks_count_dict = {
                group['depend_on_ids'][0]: group['depend_on_ids_count']
                for group in group_dependent
            }
            for task in tasks_with_dependency:
                task.dependent_tasks_count = dependent_tasks_count_dict.get(task.id, 0)

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

    @api.constrains('parent_id')
    def _check_parent_id(self):
        if not self._check_recursion():
            raise ValidationError(_('Error! You cannot create a recursive hierarchy of tasks.'))

    def _compute_attachment_ids(self):
        for task in self:
            attachment_ids = self.env['ir.attachment'].search([('res_id', '=', task.id), ('res_model', '=', 'project.task')]).ids
            message_attachment_ids = task.mapped('message_ids.attachment_ids').ids  # from mail_thread
            task.attachment_ids = [(6, 0, list(set(attachment_ids) - set(message_attachment_ids)))]

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
    def _compute_child_text(self):
        for task in self:
            if not task.subtask_count:
                task.child_text = False
            elif task.subtask_count == 1:
                task.child_text = _("(+ 1 task)")
            else:
                task.child_text = _("(+ %(child_count)s tasks)", child_count=task.subtask_count)

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

    @api.depends('user_ids')
    def _compute_portal_user_names(self):
        """ This compute method allows to see all the names of assigned users to each task contained in `self`.

            When we are in the project sharing feature, the `user_ids` contains only the users if we are a portal user.
            That is, only the users in the same company of the current user.
            So this compute method is a related of `user_ids.name` but with more records that the portal user
            can normally see.
            (In other words, this compute is only used in project sharing views to see all assignees for each task)
        """
        if self.ids:
            # fetch 'user_ids' in superuser mode (and override value in cache
            # browse is useful to avoid miscache because of the newIds contained in self
            self.browse(self.ids)._read(['user_ids'])
        for task in self.with_context(prefetch_fields=False):
            task.portal_user_names = ', '.join(task.user_ids.mapped('name'))

    def _search_portal_user_names(self, operator, value):
        if operator != 'ilike' and not isinstance(value, str):
            raise ValidationError('Not Implemented.')

        query = """
            SELECT task_user.task_id
              FROM project_task_user_rel task_user
        INNER JOIN res_users users ON task_user.user_id = users.id
        INNER JOIN res_partner partners ON partners.id = users.partner_id
             WHERE partners.name ILIKE %s
        """
        return [('id', 'inselect', (query, [f'%{value}%']))]

    @api.constrains('recurring_task')
    def _check_recurring_task(self):
        # Deprecated method
        # TODO : Remove me in master
        return

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default is None:
            default = {}
        if not default.get('name'):
            default['name'] = _("%s (copy)", self.name)
        if self.recurrence_id:
            default['recurrence_id'] = self.recurrence_id.copy().id
        return super(Task, self).copy(default)

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

    def _valid_field_parameter(self, field, name):
        # If the field has `task_dependency_tracking` on we track the changes made in the dependent task on the parent task
        return name == 'task_dependency_tracking' or super()._valid_field_parameter(field, name)

    @tools.ormcache('self.env.uid', 'self.env.su')
    def _get_depends_tracked_fields(self):
        """ Returns the set of tracked field names for the current model.
        Those fields are the ones tracked in the parent task when using task dependencies.

        See :meth:`mail.models.MailThread._get_tracked_fields`"""
        fields = {name for name, field in self._fields.items() if getattr(field, 'task_dependency_tracking', None)}
        return fields and set(self.fields_get(fields))

    # ----------------------------------------
    # Case management
    # ----------------------------------------

    def stage_find(self, section_id, domain=[], order='sequence, id'):
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
    def fields_get(self, allfields=None, attributes=None):
        fields = super().fields_get(allfields=allfields, attributes=attributes)
        if not self.env.user.has_group('base.group_portal'):
            return fields
        readable_fields = self.SELF_READABLE_FIELDS
        public_fields = {field_name: description for field_name, description in fields.items() if field_name in readable_fields}

        writable_fields = self.SELF_WRITABLE_FIELDS
        for field_name, description in public_fields.items():
            if field_name not in writable_fields and not description.get('readonly', False):
                # If the field is not in Writable fields and it is not readonly then we force the readonly to True
                description['readonly'] = True

        return public_fields

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

        if 'partner_id' in vals and not vals['partner_id']:
            # if the default_partner_id=False or no default_partner_id then we search the partner based on the project and parent
            project_id = vals.get('project_id')
            parent_id = vals.get('parent_id', self.env.context.get('default_parent_id'))
            if project_id or parent_id:
                partner_id = self._get_default_partner_id(
                    project_id and self.env['project.project'].browse(project_id),
                    parent_id and self.env['project.task'].browse(parent_id)
                )
                if partner_id:
                    vals['partner_id'] = partner_id

        return vals

    def _ensure_fields_are_accessible(self, fields, operation='read', check_group_user=True):
        """" ensure all fields are accessible by the current user

            This method checks if the portal user can access to all fields given in parameter.
            By default, it checks if the current user is a portal user and then checks if all fields are accessible for this user.

            :param fields: list of fields to check if the current user can access.
            :param operation: contains either 'read' to check readable fields or 'write' to check writable fields.
            :param check_group_user: contains boolean value.
                - True, if the method has to check if the current user is a portal one.
                - False if we are sure the user is a portal user,
        """
        assert operation in ('read', 'write'), 'Invalid operation'
        if fields and (not check_group_user or self.env.user.has_group('base.group_portal')) and not self.env.su:
            unauthorized_fields = set(fields) - (self.SELF_READABLE_FIELDS if operation == 'read' else self.SELF_WRITABLE_FIELDS)
            if unauthorized_fields:
                if operation == 'read':
                    error_message = _('You cannot read %s fields in task.', ', '.join(unauthorized_fields))
                else:
                    error_message = _('You cannot write on %s fields in task.', ', '.join(unauthorized_fields))
                raise AccessError(error_message)

    def _get_portal_sudo_vals(self, vals, defaults=False):
        """ returns the values which must be written without and with sudo when a portal user creates / writes a task.
            :param vals: dict of {field: value}, the values to create/write
            :return: a tuple with 2 dicts:
                - the first with the values to write without sudo
                - the second with the values to write with sudo
        """
        vals_no_sudo = {key: val for key, val in vals.items() if self._fields[key].type in ('one2many', 'many2many')}
        if defaults:
            vals_no_sudo.update({
                key[8:]: value
                for key, value in self.env.context.items()
                if key.startswith('default_') and key[8:] in self.SELF_WRITABLE_FIELDS and self._fields[key[8:]].type in ('one2many', 'many2many')
            })
        vals_sudo = {key: val for key, val in vals.items() if key not in vals_no_sudo}
        return vals_no_sudo, vals_sudo

    @api.model
    def _get_portal_sudo_context(self):
        return {
            key: value for key, value in self.env.context.items()
            if key == 'default_project_id'
            or not key.startswith('default_')
            or key[8:] in (field for field in self.SELF_WRITABLE_FIELDS if self._fields[field].type not in ('one2many', 'many2many'))
        }

    def read(self, fields=None, load='_classic_read'):
        self._ensure_fields_are_accessible(fields)
        return super(Task, self).read(fields=fields, load=load)

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        fields_list = ([f.split(':')[0] for f in fields] or [])
        if groupby:
            fields_groupby = [groupby] if isinstance(groupby, str) else groupby
            # only take field name when having ':' e.g 'date_deadline:week' => 'date_deadline'
            fields_list += [f.split(':')[0] for f in fields_groupby]
        if domain:
            fields_list += [term[0].split('.')[0] for term in domain if isinstance(term, (tuple, list)) and term not in [TRUE_LEAF, FALSE_LEAF]]
        self._ensure_fields_are_accessible(fields_list)
        return super(Task, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        fields_list = {term[0] for term in args if isinstance(term, (tuple, list)) and term not in [TRUE_LEAF, FALSE_LEAF]}
        self._ensure_fields_are_accessible(fields_list)
        return super(Task, self)._search(args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

    def mapped(self, func):
        # Note: This will protect the filtered method too
        if func and isinstance(func, str):
            fields_list = func.split('.')
            self._ensure_fields_are_accessible(fields_list)
        return super(Task, self).mapped(func)

    def filtered_domain(self, domain):
        fields_list = [term[0] for term in domain if isinstance(term, (tuple, list)) and term not in [TRUE_LEAF, FALSE_LEAF]]
        self._ensure_fields_are_accessible(fields_list)
        return super(Task, self).filtered_domain(domain)

    def copy_data(self, default=None):
        defaults = super().copy_data(default=default)
        if self.env.user.has_group('project.group_project_user'):
            return defaults
        return [{k: v for k, v in default.items() if k in self.SELF_READABLE_FIELDS} for default in defaults]

    @api.model
    def _ensure_portal_user_can_write(self, fields):
        for field in fields:
            if field not in self.SELF_WRITABLE_FIELDS:
                raise AccessError(_('You have not write access of %s field.') % field)

    def _load_records_create(self, vals_list):
        projects_with_recurrence = self.env['project.project'].search([('allow_recurring_tasks', '=', True)])
        for vals in vals_list:
            if vals.get('recurring_task'):
                if vals.get('project_id') in projects_with_recurrence.ids and not vals.get('recurrence_id'):
                    default_val = self.default_get(self._get_recurrence_fields())
                    vals.update(**default_val)
                else:
                    for field_name in self._get_recurrence_fields() + ['recurring_task']:
                        vals.pop(field_name, None)
        tasks = super()._load_records_create(vals_list)
        stage_ids_per_project = defaultdict(list)
        for task in tasks:
            if task.stage_id and task.stage_id not in task.project_id.type_ids and task.stage_id.id not in stage_ids_per_project[task.project_id]:
                stage_ids_per_project[task.project_id].append(task.stage_id.id)

        for project, stage_ids in stage_ids_per_project.items():
            project.write({'type_ids': [Command.link(stage_id) for stage_id in stage_ids]})

        return tasks

    @api.model_create_multi
    def create(self, vals_list):
        is_portal_user = self.env.user.has_group('base.group_portal')
        if is_portal_user:
            self.check_access_rights('create')
        default_stage = dict()
        for vals in vals_list:
            if is_portal_user:
                self._ensure_fields_are_accessible(vals.keys(), operation='write', check_group_user=False)

            project_id = vals.get('project_id') or self.env.context.get('default_project_id')
            if not vals.get('parent_id'):
                # 1) We must initialize display_project_id to follow project_id if there is no parent_id
                vals['display_project_id'] = project_id
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
            # user_ids change: update date_assign
            if vals.get('user_ids'):
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
        # The sudo is required for a portal user as the record creation
        # requires the read access on other models, as mail.template
        # in order to compute the field tracking
        was_in_sudo = self.env.su
        if is_portal_user:
            vals_list_no_sudo, vals_list = zip(*(self._get_portal_sudo_vals(vals, defaults=True) for vals in vals_list))
            self_no_sudo, self = self, self.with_context(self._get_portal_sudo_context()).sudo()
        tasks = super(Task, self).create(vals_list)
        if is_portal_user:
            for task, vals in zip(tasks.with_env(self_no_sudo.env), vals_list_no_sudo):
                task.write(vals)
        tasks._populate_missing_personal_stages()
        self._task_message_auto_subscribe_notify({task: task.user_ids - self.env.user for task in tasks})

        # in case we were already in sudo, we don't check the rights.
        if is_portal_user and not was_in_sudo:
            # since we use sudo to create tasks, we need to check
            # if the portal user could really create the tasks based on the ir rule.
            tasks.with_user(self.env.user).check_access_rule('create')
        for task in tasks:
            if task.project_id.privacy_visibility == 'portal':
                task._portal_ensure_token()
        return tasks

    def write(self, vals):
        if len(self) == 1:
            handle_history_divergence(self, 'description', vals)
        portal_can_write = False
        if self.env.user.has_group('base.group_portal') and not self.env.su:
            # Check if all fields in vals are in SELF_WRITABLE_FIELDS
            self._ensure_fields_are_accessible(vals.keys(), operation='write', check_group_user=False)
            self.check_access_rights('write')
            self.check_access_rule('write')
            portal_can_write = True

        now = fields.Datetime.now()
        if 'parent_id' in vals and vals['parent_id'] in self.ids:
            raise UserError(_("Sorry. You can't set a task as its parent task."))
        if 'active' in vals and not vals.get('active') and any(self.mapped('recurrence_id')):
            # TODO: show a dialog to stop the recurrence
            raise UserError(_('You cannot archive recurring tasks. Please disable the recurrence first.'))
        if 'recurrence_id' in vals and vals.get('recurrence_id') and any(not task.active for task in self):
            raise UserError(_('Archived tasks cannot be recurring. Please unarchive the task first.'))
        # stage change: update date_last_stage_update
        if 'stage_id' in vals:
            vals.update(self.update_date_end(vals['stage_id']))
            vals['date_last_stage_update'] = now
            # reset kanban state when changing stage
            if 'kanban_state' not in vals:
                vals['kanban_state'] = 'normal'
        # user_ids change: update date_assign
        if vals.get('user_ids') and 'date_assign' not in vals:
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

        # The sudo is required for a portal user as the record update
        # requires the write access on others models, as rating.rating
        # in order to keep the same name than the task.
        if portal_can_write:
            tasks_no_sudo, tasks = tasks, tasks.sudo()
            vals_no_sudo, vals = self._get_portal_sudo_vals(vals)

        # Track user_ids to send assignment notifications
        old_user_ids = {t: t.user_ids for t in self}

        result = super(Task, tasks).write(vals)
        if portal_can_write:
            super(Task, tasks_no_sudo).write(vals_no_sudo)

        self._task_message_auto_subscribe_notify({task: task.user_ids - old_user_ids[task] - self.env.user for task in self})

        if 'user_ids' in vals:
            tasks._populate_missing_personal_stages()

        # rating on stage
        if 'stage_id' in vals and vals.get('stage_id'):
            tasks.filtered(lambda x: x.project_id.rating_active and x.project_id.rating_status == 'stage')._send_task_rating_mail(force_send=True)
        for task in self:
            if task.display_project_id != task.project_id and not task.parent_id:
                # We must make the display_project_id follow the project_id if no parent_id set
                task.display_project_id = task.project_id
        return result

    def update_date_end(self, stage_id):
        project_task_type = self.env['project.task.type'].browse(stage_id)
        if project_task_type.fold or project_task_type.is_closed:
            return {'date_end': fields.Datetime.now()}
        return {'date_end': False}

    @api.ondelete(at_uninstall=False)
    def _unlink_except_recurring(self):
        if any(self.mapped('recurrence_id')):
            # TODO: show a dialog to stop the recurrence
            raise UserError(_('You cannot delete recurring tasks. Please disable the recurrence first.'))

    # ---------------------------------------------------
    # Subtasks
    # ---------------------------------------------------

    @api.depends('parent_id', 'project_id', 'display_project_id')
    def _compute_partner_id(self):
        """ Compute the partner_id when the tasks have no partner_id.

            Use the project partner_id if any, or else the parent task partner_id.
        """
        for task in self.filtered(lambda task: not task.partner_id):
            # When the task has a parent task, the display_project_id can be False or the project choose by the user for this task.
            project = task.display_project_id if task.parent_id and task.display_project_id else task.project_id
            task.partner_id = self._get_default_partner_id(project, task.parent_id)

    @api.depends('partner_id.email', 'parent_id.email_from')
    def _compute_email_from(self):
        for task in self:
            task.email_from = task.partner_id.email or ((task.partner_id or task.parent_id) and task.email_from) or task.parent_id.email_from

    @api.depends('parent_id.project_id', 'display_project_id')
    def _compute_project_id(self):
        for task in self:
            if task.parent_id:
                task.project_id = task.display_project_id or task.parent_id.project_id

    # ---------------------------------------------------
    # Mail gateway
    # ---------------------------------------------------

    @api.model
    def _task_message_auto_subscribe_notify(self, users_per_task):
        # Utility method to send assignation notification upon writing/creation.
        template_id = self.env['ir.model.data']._xmlid_to_res_id('project.project_message_user_assigned', raise_if_not_found=False)
        if not template_id:
            return
        view = self.env['ir.ui.view'].browse(template_id)
        task_model_description = self.env['ir.model']._get(self._name).display_name
        for task, users in users_per_task.items():
            if not users:
                continue
            values = {
                'object': task,
                'model_description': task_model_description,
                'access_link': task._notify_get_action_link('view'),
            }
            for user in users:
                values.update(assignee_name=user.sudo().name)
                assignation_msg = view._render(values, engine='ir.qweb', minimal_qcontext=True)
                assignation_msg = self.env['mail.render.mixin']._replace_local_links(assignation_msg)
                task.message_notify(
                    subject=_('You have been assigned to %s', task.display_name),
                    body=assignation_msg,
                    partner_ids=user.partner_id.ids,
                    record_name=task.display_name,
                    email_layout_xmlid='mail.mail_notification_light',
                    model_description=task_model_description,
                )

    def _message_auto_subscribe_followers(self, updated_values, default_subtype_ids):
        if 'user_ids' not in updated_values:
            return []
        # Since the changes to user_ids becoming a m2m, the default implementation of this function
        #  could not work anymore, override the function to keep the functionality.
        new_followers = []
        # Normalize input to tuple of ids
        value = self._fields['user_ids'].convert_to_cache(updated_values.get('user_ids', []), self.env['project.task'], validate=False)
        users = self.env['res.users'].browse(value)
        for user in users:
            try:
                if user.partner_id:
                    # The you have been assigned notification is handled separately
                    new_followers.append((user.partner_id.id, default_subtype_ids, False))
            except:
                pass
        return new_followers

    def _mail_track(self, tracked_fields, initial_values):
        changes, tracking_value_ids  = super()._mail_track(tracked_fields, initial_values)
        # Many2many tracking
        if len(changes) > len(tracking_value_ids):
            for changed_field in changes:
                if tracked_fields[changed_field]['type'] in ['one2many', 'many2many']:
                    field = self.env['ir.model.fields']._get(self._name, changed_field)
                    vals = {
                        'field': field.id,
                        'field_desc': field.field_description,
                        'field_type': field.ttype,
                        'tracking_sequence': field.tracking,
                        'old_value_char': ', '.join(initial_values[changed_field].mapped('name')),
                        'new_value_char': ', '.join(self[changed_field].mapped('name')),
                    }
                    tracking_value_ids.append(Command.create(vals))
        # Track changes on depending tasks
        depends_tracked_fields = self._get_depends_tracked_fields()
        depends_changes = changes & depends_tracked_fields
        if depends_changes and self.allow_task_dependencies and self.user_has_groups('project.group_project_task_dependencies'):
            parent_ids = self.dependent_ids
            if parent_ids:
                fields_to_ids = self.env['ir.model.fields']._get_ids('project.task')
                field_ids = [fields_to_ids.get(name) for name in depends_changes]
                depends_tracking_value_ids = [
                    tracking_values for tracking_values in tracking_value_ids
                    if tracking_values[2]['field'] in field_ids
                ]
                subtype = self.env['ir.model.data']._xmlid_to_res_id('project.mt_task_dependency_change')
                # We want to include the original subtype message coming from the child task
                # for example when the stage changes the message in the chatter starts with 'Stage Changed'
                child_subtype = self._track_subtype(dict((col_name, initial_values[col_name]) for col_name in changes))
                child_subtype_info = child_subtype.description or child_subtype.name if child_subtype else False
                # NOTE: the subtype does not have a description on purpose, otherwise the description would be put
                #  at the end of the message instead of at the top, we use the name here
                body = self.env['ir.qweb']._render('project.task_track_depending_tasks', {
                    'child': self,
                    'child_subtype': child_subtype_info,
                })
                for p in parent_ids:
                    p.message_post(body=body, subtype_id=subtype, tracking_value_ids=depends_tracking_value_ids)
        return changes, tracking_value_ids

    def _track_template(self, changes):
        res = super(Task, self)._track_template(changes)
        test_task = self[0]
        if 'stage_id' in changes and test_task.stage_id.mail_template_id:
            res['stage_id'] = (test_task.stage_id.mail_template_id, {
                'auto_delete_message': True,
                'subtype_id': self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note'),
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

    def _mail_get_message_subtypes(self):
        res = super()._mail_get_message_subtypes()
        if len(self) == 1:
            dependency_subtype = self.env.ref('project.mt_task_dependency_change')
            if ((self.project_id and not self.project_id.allow_task_dependencies)\
                or (not self.project_id and not self.user_has_groups('project.group_project_task_dependencies')))\
                and dependency_subtype in res:
                res -= dependency_subtype
        return res

    def _notify_get_groups(self, msg_vals=None):
        """ Handle project users and managers recipients that can assign
        tasks and create new one directly from notification emails. Also give
        access button to portal users and portal customers. If they are notified
        they should probably have access to the document. """
        groups = super(Task, self)._notify_get_groups(msg_vals=msg_vals)
        local_msg_vals = dict(msg_vals or {})
        self.ensure_one()

        project_user_group_id = self.env.ref('project.group_project_user').id
        new_group = ('group_project_user', lambda pdata: pdata['type'] == 'user' and project_user_group_id in pdata['groups'], {})
        groups = [new_group] + groups

        if self.project_privacy_visibility == 'portal':
            groups.insert(0, (
                'allowed_portal_users',
                lambda pdata: pdata['type'] == 'portal',
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
        create_context['default_user_ids'] = False
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
        self.write({'user_ids': [(4, self.env.user.id)]})

    def action_unassign_me(self):
        self.write({'user_ids': [Command.unlink(self.env.uid)]})

    # If depth == 1, return only direct children
    # If depth == 3, return children to third generation
    # If depth <= 0, return all children without depth limit
    def _get_all_subtasks(self, depth=0):
        children = self.mapped('child_ids')
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
            'context': self._context
        }

    # ------------
    # Actions
    # ------------

    def action_open_task(self):
        return {
            'view_mode': 'form',
            'res_model': 'project.task',
            'res_id': self.id,
            'type': 'ir.actions.act_window',
            'context': self._context
        }

    def action_dependent_tasks(self):
        self.ensure_one()
        action = {
            'res_model': 'project.task',
            'type': 'ir.actions.act_window',
            'context': {**self._context, 'default_depend_on_ids': [Command.link(self.id)], 'show_project_update': False},
            'domain': [('depend_on_ids', '=', self.id)],
        }
        if self.dependent_tasks_count == 1:
            action['view_mode'] = 'form'
            action['res_id'] = self.dependent_ids.id
            action['views'] = [(False, 'form')]
        else:
            action['name'] = _('Dependent Tasks')
            action['view_mode'] = 'tree,form,kanban,calendar,pivot,graph,activity'
        return action

    def action_recurring_tasks(self):
        return {
            'name': 'Tasks in Recurrence',
            'type': 'ir.actions.act_window',
            'res_model': 'project.task',
            'view_mode': 'tree,form,kanban,calendar,pivot,graph,activity',
            'domain': [('recurrence_id', 'in', self.recurrence_id.ids)],
        }

    def action_stop_recurrence(self):
        tasks = self.env['project.task'].with_context(active_test=False).search([('recurrence_id', 'in', self.recurrence_id.ids)])
        tasks.write({'recurring_task': False})
        self.recurrence_id.unlink()

    def action_continue_recurrence(self):
        self.recurrence_id = False
        self.recurring_task = False

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

    def rating_get_rated_partner_id(self):
        """ Overwrite since we have user_ids and not user_id """
        tasks_with_one_user = self.filtered(lambda task: len(task.user_ids) == 1 and task.user_ids.partner_id)
        return tasks_with_one_user.user_ids.partner_id or self.env['res.partner']

    # ---------------------------------------------------
    # Privacy
    # ---------------------------------------------------
    def _change_project_privacy_visibility(self):
        for task in self.filtered(lambda t: t.project_privacy_visibility != 'portal'):
            portal_users = task.message_partner_ids.user_ids.filtered('share')
            task.message_unsubscribe(partner_ids=portal_users.partner_id.ids)

    # ---------------------------------------------------
    # Analytic accounting
    # ---------------------------------------------------
    def _get_task_analytic_account_id(self):
        self.ensure_one()
        return self.analytic_account_id or self.project_analytic_account_id

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

## File: models\project_collaborator.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProjectCollaborator(models.Model):
    _name = 'project.collaborator'
    _description = 'Collaborators in project shared'

    project_id = fields.Many2one('project.project', 'Project Shared', domain=[('privacy_visibility', '=', 'portal')], required=True, readonly=True)
    partner_id = fields.Many2one('res.partner', 'Collaborator', required=True, readonly=True)

    _sql_constraints = [
        ('unique_collaborator', 'UNIQUE(project_id, partner_id)', 'A collaborator cannot be selected more than once in the project sharing access. Please remove duplicate(s) and try again.'),
    ]

    def name_get(self):
        collaborator_search_read = self.search_read([('id', 'in', self.ids)], ['id', 'project_id', 'partner_id'])
        return [(collaborator['id'], '%s - %s' % (collaborator['project_id'][1], collaborator['partner_id'][1])) for collaborator in collaborator_search_read]

    @api.model_create_multi
    def create(self, vals_list):
        collaborator = self.env['project.collaborator'].search([], limit=1)
        project_collaborators = super().create(vals_list)
        if not collaborator:
            self._toggle_project_sharing_portal_rules(True)
        return project_collaborators

    def unlink(self):
        res = super().unlink()
        # Check if it remains at least a collaborator in all shared projects.
        collaborator = self.env['project.collaborator'].search([], limit=1)
        if not collaborator:  # then disable the project sharing feature
            self._toggle_project_sharing_portal_rules(False)
        return res

    @api.model
    def _toggle_project_sharing_portal_rules(self, active):
        """ Enable/disable project sharing feature

            When the first collaborator is added in the model then we need to enable the feature.
            In the inverse case, if no collaborator is stored in the model then we disable the feature.
            To enable/disable the feature, we just need to enable/disable the ir.model.access and ir.rule
            added to portal user that we do not want to give when we know the project sharing is unused.

            :param active: contains boolean value, True to enable the project sharing feature, otherwise we disable the feature.
        """
        access_project_sharing_portal = self.env.ref('project.access_project_sharing_task_portal').sudo()
        if access_project_sharing_portal.active != active:
            access_project_sharing_portal.write({'active': active})

        task_portal_ir_rule = self.env.ref('project.project_task_rule_portal_project_sharing').sudo()
        if task_portal_ir_rule.active != active:
            task_portal_ir_rule.write({'active': active})

```

## File: models\project_milestone.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

class ProjectMilestone(models.Model):
    _name = 'project.milestone'
    _description = "Project Milestone"
    _inherit = ['mail.thread']
    _order = 'deadline, is_reached desc, name'

    def _get_default_project_id(self):
        return self.env.context.get('default_project_id') or self.env.context.get('active_id')

    name = fields.Char(required=True)
    project_id = fields.Many2one('project.project', required=True, default=_get_default_project_id)
    deadline = fields.Date(tracking=True)
    is_reached = fields.Boolean(string="Reached", default=False)
    reached_date = fields.Date(compute='_compute_reached_date', store=True)

    # computed non-stored fields
    is_deadline_exceeded = fields.Boolean(compute="_compute_is_deadline_exceeded")
    is_deadline_future = fields.Boolean(compute="_compute_is_deadline_future")

    @api.depends('is_reached')
    def _compute_reached_date(self):
        for ms in self:
            ms.reached_date = ms.is_reached and fields.Date.context_today(self)

    @api.depends('is_reached', 'deadline')
    def _compute_is_deadline_exceeded(self):
        today = fields.Date.context_today(self)
        for ms in self:
            ms.is_deadline_exceeded = not ms.is_reached and ms.deadline and ms.deadline < today

    @api.depends('deadline')
    def _compute_is_deadline_future(self):
        for ms in self:
            ms.is_deadline_future = ms.deadline and ms.deadline > fields.Date.context_today(self)

    def toggle_is_reached(self, is_reached):
        self.ensure_one()
        self.update({'is_reached': is_reached})
        return self._get_data()

    @api.model
    def _get_fields_to_export(self):
        return ['id', 'name', 'deadline', 'is_reached', 'reached_date', 'is_deadline_exceeded', 'is_deadline_future']

    def _get_data(self):
        self.ensure_one()
        return {field: self[field] for field in self._get_fields_to_export()}

    def _get_data_list(self):
        return [ms._get_data() for ms in self]

```

## File: models\project_project_stage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ProjectProjectStage(models.Model):
    _name = 'project.project.stage'
    _description = 'Project Stage'
    _order = 'sequence, id'

    active = fields.Boolean(default=True)
    sequence = fields.Integer(default=50)
    name = fields.Char(required=True, translate=True)
    mail_template_id = fields.Many2one('mail.template', string='Email Template', domain=[('model', '=', 'project.project')],
        help="If set, an email will be sent to the customer when the project reaches this step.")
    fold = fields.Boolean('Folded in Kanban', help="This stage is folded in the kanban view.")

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

    task_ids = fields.One2many('project.task', 'recurrence_id', copy=False)
    next_recurrence_date = fields.Date()
    recurrence_left = fields.Integer(string="Number of Tasks Left to Create", copy=False)

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
        return ['message_partner_ids', 'company_id', 'description', 'displayed_image_id', 'email_cc',
                'parent_id', 'partner_email', 'partner_id', 'partner_phone', 'planned_hours',
                'project_id', 'display_project_id', 'project_privacy_visibility', 'sequence', 'tag_ids', 'recurrence_id',
                'name', 'recurring_task', 'analytic_account_id']

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
            month = list(MONTHS.keys()).index(repeat_month) + 1 if repeat_month else date_start.month
            repeat_month = repeat_month or list(MONTHS.keys())[month - 1]
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
        create_values['user_ids'] = False
        return create_values

    def _create_subtasks(self, task, new_task, depth=3):
        if depth == 0 or not task.child_ids:
            return
        children = []
        child_recurrence = []
        # copy the subtasks of the original task
        for child in task.child_ids:
            if child.recurrence_id and child.recurrence_id.id in child_recurrence:
                # The subtask has been generated by another subtask in the childs
                # This subtasks is skipped as it will be meant to be a copy of the first
                # task of the recurrence we just created.
                continue
            child_values = self._new_task_values(child)
            child_values['parent_id'] = new_task.id
            if child.recurrence_id:
                # The subtask has a recurrence, the recurrence is thus copied rather than used
                # with raw reference in order to decouple the recurrence of the initial subtask
                # from the recurrence of the copied subtask which will live its own life and generate
                # subsequent tasks.
                child_recurrence += [child.recurrence_id.id]
                child_values['recurrence_id'] = child.recurrence_id.copy().id
            if child.child_ids and depth > 1:
                # If child has childs in the following layer and we will have to copy layer, we have to
                # first create the new_child record in order to have a new parent_id reference for the
                # "grandchildren" tasks
                new_child = self.env['project.task'].sudo().create(child_values)
                self._create_subtasks(child, new_child, depth=depth - 1)
            else:
                children.append(child_values)
        children_tasks = self.env['project.task'].sudo().create(children)

    def _create_next_task(self):
        for recurrence in self:
            task = max(recurrence.sudo().task_ids, key=lambda t: t.id)
            create_values = recurrence._new_task_values(task)
            new_task = self.env['project.task'].sudo().create(create_values)
            recurrence._create_subtasks(task, new_task, depth=3)

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
                next_date = self._get_next_recurring_dates(tomorrow, recurrence.repeat_interval, recurrence.repeat_unit, recurrence.repeat_type, recurrence.repeat_until, recurrence.repeat_on_month, recurrence.repeat_on_year, recurrence._get_weekdays(WEEKS.get(recurrence.repeat_week)), recurrence.repeat_day, recurrence.repeat_week, recurrence.repeat_month, count=1)
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

## File: models\project_task_stage_personal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ProjectTaskStagePersonal(models.Model):
    _name = 'project.task.stage.personal'
    _description = 'Personal Task Stage'
    _table = 'project_task_user_rel'
    _rec_name = 'stage_id'

    task_id = fields.Many2one('project.task', required=True, ondelete='cascade', index=True)
    user_id = fields.Many2one('res.users', required=True, ondelete='cascade', index=True)
    stage_id = fields.Many2one('project.task.type', domain="[('user_id', '=', user_id)]", ondelete='restrict')

    _sql_constraints = [
        ('project_personal_stage_unique', 'UNIQUE (task_id, user_id)', 'A task can only have a single personal stage per user.'),
    ]

```

## File: models\project_update.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
from dateutil.relativedelta import relativedelta
from werkzeug.urls import url_encode

from odoo import api, fields, models
from odoo.osv import expression
from odoo.tools import formatLang

STATUS_COLOR = {
    'on_track': 20,  # green / success
    'at_risk': 2,  # orange
    'off_track': 23,  # red / danger
    'on_hold': 4,  # light blue
    False: 0,  # default grey -- for studio
}

class ProjectUpdate(models.Model):
    _name = 'project.update'
    _description = 'Project Update'
    _order = 'date desc'
    _inherit = ['mail.thread.cc', 'mail.activity.mixin']

    def default_get(self, fields):
        result = super().default_get(fields)
        if 'project_id' in fields and not result.get('project_id'):
            result['project_id'] = self.env.context.get('active_id')
        if result.get('project_id'):
            project = self.env['project.project'].browse(result['project_id'])
            if 'progress' in fields and not result.get('progress'):
                result['progress'] = project.last_update_id.progress
            if 'description' in fields and not result.get('description'):
                result['description'] = self._build_description(project)
            if 'status' in fields and not result.get('status'):
                result['status'] = project.last_update_status
        return result

    name = fields.Char("Title", required=True, tracking=True)
    status = fields.Selection(selection=[
        ('on_track', 'On Track'),
        ('at_risk', 'At Risk'),
        ('off_track', 'Off Track'),
        ('on_hold', 'On Hold')
    ], required=True, tracking=True)
    color = fields.Integer(compute='_compute_color')
    progress = fields.Integer(tracking=True)
    progress_percentage = fields.Float(compute='_compute_progress_percentage')
    user_id = fields.Many2one('res.users', string='Author', required=True, default=lambda self: self.env.user)
    description = fields.Html()
    date = fields.Date(default=fields.Date.context_today, tracking=True)
    project_id = fields.Many2one('project.project', required=True)
    name_cropped = fields.Char(compute="_compute_name_cropped")

    @api.depends('status')
    def _compute_color(self):
        for update in self:
            update.color = STATUS_COLOR[update.status]

    @api.depends('progress')
    def _compute_progress_percentage(self):
        for u in self:
            u.progress_percentage = u.progress / 100

    @api.depends('name')
    def _compute_name_cropped(self):
        for u in self:
            u.name_cropped = (u.name[:57] + '...') if len(u.name) > 60 else u.name

    # ---------------------------------
    # ORM Override
    # ---------------------------------
    @api.model
    def create(self, vals):
        update = super().create(vals)
        update.project_id.sudo().last_update_id = update
        return update

    def unlink(self):
        projects = self.project_id
        res = super().unlink()
        for project in projects:
            project.last_update_id = self.search([('project_id', "=", project.id)], order="date desc", limit=1)
        return res

    # ---------------------------------
    # Build default description
    # ---------------------------------
    @api.model
    def _build_description(self, project):
        template = self.env.ref('project.project_update_default_description')
        return template._render(self._get_template_values(project), engine='ir.qweb')

    @api.model
    def _get_template_values(self, project):
        milestones = self._get_milestone_values(project)
        return {
            'user': self.env.user,
            'project': project,
            'show_activities': milestones['show_section'],
            'milestones': milestones,
            'format_lang': lambda value, digits: formatLang(self.env, value, digits=digits),
        }

    @api.model
    def _get_milestone_values(self, project):
        Milestone = self.env['project.milestone']
        list_milestones = Milestone.search(
            [('project_id', '=', project.id),
             '|', ('deadline', '<', fields.Date.context_today(self) + relativedelta(years=1)), ('deadline', '=', False)])._get_data_list()
        updated_milestones = self._get_last_updated_milestone(project)
        domain = [('project_id', '=', project.id)]
        if project.last_update_id.create_date:
            domain = expression.AND([domain, [('create_date', '>', project.last_update_id.create_date)]])
        created_milestones = Milestone.search(domain)._get_data_list()
        return {
            'show_section': (list_milestones or updated_milestones or created_milestones) and True or False,
            'list': list_milestones,
            'updated': updated_milestones,
            'last_update_date': project.last_update_id.create_date or None,
            'created': created_milestones,
        }

    @api.model
    def _get_last_updated_milestone(self, project):
        query = """
            SELECT DISTINCT pm.id as milestone_id,
                            pm.deadline as deadline,
                            FIRST_VALUE(old_value_datetime::date) OVER w_partition as old_value,
                            pm.deadline as new_value
                       FROM mail_message mm
                 INNER JOIN mail_tracking_value mtv
                         ON mm.id = mtv.mail_message_id
                 INNER JOIN ir_model_fields imf
                         ON mtv.field = imf.id
                        AND imf.model = 'project.milestone'
                        AND imf.name = 'deadline'
                 INNER JOIN project_milestone pm
                         ON mm.res_id = pm.id
                      WHERE mm.model = 'project.milestone'
                        AND mm.message_type = 'notification'
                        AND pm.project_id = %(project_id)s
         """
        if project.last_update_id.create_date:
            query = query + "AND mm.date > %(last_update_date)s"
        query = query + """
                     WINDOW w_partition AS (
                             PARTITION BY pm.id
                             ORDER BY mm.date ASC
                            )
                   ORDER BY pm.deadline ASC
                   LIMIT 1;
        """
        query_params = {'project_id': project.id}
        if project.last_update_id.create_date:
            query_params['last_update_date'] = project.last_update_id.create_date
        self.env.cr.execute(query, query_params)
        results = self.env.cr.dictfetchall()
        mapped_result = {res['milestone_id']: {'new_value': res['new_value'], 'old_value': res['old_value']} for res in results}
        milestones = self.env['project.milestone'].search([('id', 'in', list(mapped_result.keys()))])
        return [{
            **milestone._get_data(),
            'new_value': mapped_result[milestone.id]['new_value'],
            'old_value': mapped_result[milestone.id]['old_value'],
        } for milestone in milestones]

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
    group_project_stages = fields.Boolean("Project Stages", implied_group="project.group_project_stages")
    group_project_recurring_tasks = fields.Boolean("Recurring Tasks", implied_group="project.group_project_recurring_tasks")
    group_project_task_dependencies = fields.Boolean("Task Dependencies", implied_group="project.group_project_task_dependencies")

    @api.model
    def _get_basic_project_domain(self):
        return []

    def set_values(self):
        # Ensure that settings on existing projects match the above fields
        projects = self.env["project.project"].search([])
        basic_projects = projects.filtered_domain(self._get_basic_project_domain())

        features = {
            # key: (config_flag, is_global), value: project_flag
            ("group_project_rating", True): "rating_active",
            ("group_project_recurring_tasks", True): "allow_recurring_tasks",
            ("group_subtask_project", False): "allow_subtasks",
            ("group_project_task_dependencies", False): "allow_task_dependencies",
        }

        for (config_flag, is_global), project_flag in features.items():
            config_flag_global = f"project.{config_flag}"
            config_feature_enabled = self[config_flag]
            if self.user_has_groups(config_flag_global) != config_feature_enabled:
                if config_feature_enabled and not is_global:
                    basic_projects[project_flag] = config_feature_enabled
                else:
                    projects[project_flag] = config_feature_enabled

        # Hide the task dependency changes subtype when the dependency setting is disabled
        task_dep_change_subtype_id = self.env.ref('project.mt_task_dependency_change')
        project_task_dep_change_subtype_id = self.env.ref('project.mt_project_task_dependency_change')
        if task_dep_change_subtype_id.hidden != (not self['group_project_task_dependencies']):
            task_dep_change_subtype_id.hidden = not self['group_project_task_dependencies']
            project_task_dep_change_subtype_id.hidden = not self['group_project_task_dependencies']
        # Hide Project Stage Changed mail subtype according to the settings
        project_stage_change_mail_type = self.env.ref('project.mt_project_stage_change')
        if project_stage_change_mail_type.hidden == self['group_project_stages']:
            project_stage_change_mail_type.hidden = not self['group_project_stages']

        super(ResConfigSettings, self).set_values()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools import email_normalize


class ResPartner(models.Model):
    """ Inherits partner and adds Tasks information in the partner form """
    _inherit = 'res.partner'

    task_ids = fields.One2many('project.task', 'partner_id', string='Tasks')
    task_count = fields.Integer(compute='_compute_task_count', string='# Tasks')

    def _compute_task_count(self):
        # retrieve all children partners and prefetch 'parent_id' on them
        all_partners = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])
        all_partners.read(['parent_id'])

        task_data = self.env['project.task'].read_group(
            domain=[('partner_id', 'in', all_partners.ids)],
            fields=['partner_id'], groupby=['partner_id']
        )

        self.task_count = 0
        for group in task_data:
            partner = self.browse(group['partner_id'][0])
            while partner:
                if partner in self:
                    partner.task_count += group['partner_id_count']
                partner = partner.parent_id

# Deprecated: remove me in MASTER
    def _create_portal_users(self):
        partners_without_user = self.filtered(lambda partner: not partner.user_ids)
        if not partners_without_user:
            return self.env['res.users']
        created_users = self.env['res.users']
        for partner in partners_without_user:
            created_users += self.env['res.users'].with_context(no_reset_password=True).sudo()._create_user_from_template({
                'email': email_normalize(partner.email),
                'login': email_normalize(partner.email),
                'partner_id': partner.id,
                'company_id': self.env.company.id,
                'company_ids': [(6, 0, self.env.company.ids)],
                'active': True,
            })
        return created_users

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import analytic_account
from . import analytic_account_tag
from . import project_milestone
from . import project_project_stage
from . import project_task_recurrence
# `project_task_stage_personal` has to be loaded before `project`
from . import project_task_stage_personal
from . import project
from . import project_collaborator
from . import project_update
from . import res_config_settings
from . import res_partner
from . import digest
from . import ir_ui_menu

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

    name = fields.Char(string='Task', readonly=True)
    user_ids = fields.Many2many('res.users', relation='project_task_user_rel', column1='task_id', column2='user_id',
                                string='Assignees', readonly=True)
    create_date = fields.Datetime("Create Date", readonly=True)
    date_assign = fields.Datetime(string='Assignment Date', readonly=True)
    date_end = fields.Datetime(string='Ending Date', readonly=True)
    date_deadline = fields.Date(string='Deadline', readonly=True)
    date_last_stage_update = fields.Datetime(string='Last Stage Update', readonly=True)
    project_id = fields.Many2one('project.project', string='Project', readonly=True)
    working_days_close = fields.Float(string='Working Days to Close',
        digits=(16,2), readonly=True, group_operator="avg",
        help="Number of Working Days to close the task")
    working_days_open = fields.Float(string='Working Days to Assign',
        digits=(16,2), readonly=True, group_operator="avg",
        help="Number of Working Days to open the task")
    delay_endings_days = fields.Float(string='Days to Deadline', digits=(16, 2), group_operator="avg", readonly=True)
    nbr = fields.Integer('# of Tasks', readonly=True)  # TDE FIXME master: rename into nbr_tasks
    working_hours_open = fields.Float(string='Working Hours to Assign', digits=(16, 2), readonly=True, group_operator="avg", help="Number of Working Hours to open the task")
    working_hours_close = fields.Float(string='Working Hours to Close', digits=(16, 2), readonly=True, group_operator="avg", help="Number of Working Hours to close the task")
    rating_last_value = fields.Float('Rating Value (/5)', group_operator="avg", readonly=True, groups="project.group_project_rating")
    priority = fields.Selection([
        ('0', 'Low'),
        ('1', 'Normal'),
        ('2', 'High')
        ], readonly=True, string="Priority")
    state = fields.Selection([
            ('normal', 'In Progress'),
            ('blocked', 'Blocked'),
            ('done', 'Ready for Next Stage')
        ], string='Kanban State', readonly=True)
    company_id = fields.Many2one('res.company', string='Company', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    stage_id = fields.Many2one('project.task.type', string='Stage', readonly=True)
    task_id = fields.Many2one('project.task', string='Tasks', readonly=True)

    def _select(self):
        select_str = """
             SELECT
                    (select 1 ) AS nbr,
                    t.id as id,
                    t.id as task_id,
                    t.create_date as create_date,
                    t.date_assign as date_assign,
                    t.date_end as date_end,
                    t.date_last_stage_update as date_last_stage_update,
                    t.date_deadline as date_deadline,
                    t.project_id,
                    t.priority,
                    t.name as name,
                    t.company_id,
                    t.partner_id,
                    t.stage_id as stage_id,
                    t.kanban_state as state,
                    NULLIF(t.rating_last_value, 0) as rating_last_value,
                    t.working_days_close as working_days_close,
                    t.working_days_open  as working_days_open,
                    t.working_hours_open as working_hours_open,
                    t.working_hours_close as working_hours_close,
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
              LEFT JOIN project_task_user_rel tu on t.id=tu.task_id
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
                <pivot string="Tasks Analysis" display_quantity="1" sample="1">
                    <field name="project_id" type="row"/>
                </pivot>
            </field>
        </record>

        <record id="view_task_project_user_graph" model="ir.ui.view">
            <field name="name">report.project.task.user.graph</field>
            <field name="model">report.project.task.user</field>
            <field name="arch" type="xml">
                <graph string="Tasks Analysis" sample="1">
                     <field name="project_id"/>
                     <field name="stage_id"/>
                     <field name="user_ids" invisible="1"/>
                     <field name="nbr" invisible="1"/>
                 </graph>
             </field>
        </record>

        <record id="report_project_task_user_view_tree" model="ir.ui.view">
            <field name="name">report.project.task.user.view.tree</field>
            <field name="model">report.project.task.user</field>
            <field name="arch" type="xml">
                <tree string="Tasks Analysis" editable="top" delete="false">
                    <field name="name"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="project_id" optional="show"/>
                    <field name="user_ids" optional="show" widget="many2many_avatar_user"/>
                    <field name="stage_id" optional="show"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="view_task_project_user_search" model="ir.ui.view">
            <field name="name">report.project.task.user.search</field>
            <field name="model">report.project.task.user</field>
            <field name="arch" type="xml">
                <search string="Tasks Analysis">
                    <field name="project_id"/>
                    <field name="name" />
                    <field name="stage_id"/>
                    <field name="user_ids"/>
                    <field name="partner_id" filter_domain="[('partner_id', 'child_of', self)]"/>
                    <field name="name" string="Tags" filter_domain="[('task_id.tag_ids', 'ilike', self)]"/>
                    <field name="date_assign"/>
                    <field name="date_end"/>
                    <field name="date_deadline"/>
                    <field name="date_last_stage_update"/>
                    <filter string="My Projects" name="own_projects" domain="[('project_id.user_id', '=', uid)]"/>
                    <filter string="My Tasks" name="my_tasks" domain="[('task_id.user_ids', 'in', uid)]"/>
                    <separator/>
                    <filter string="Starred" name="starred" domain="[('task_id.priority', 'in', [0, 1])]"/>
                    <separator/>
                    <filter string="Tasks Late" name="late" domain="[('task_id.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d')), ('task_id.is_closed', '=', False)]"/>
                    <separator/>
                    <filter string="Unassigned Tasks" name="unassigned" domain="[('user_ids', '=', False)]"/>
                    <separator/>
                    <filter string="Open tasks" name="open_tasks" domain="[('stage_id.fold', '=', False), ('stage_id.is_closed', '=', False)]"/>
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
                        <filter string="Stage" name="Stage" context="{'group_by': 'stage_id'}"/>
                        <filter string="Assignees" name="User" context="{'group_by': 'user_ids'}"/>
                        <filter string="Customer" name="Customer" context="{'group_by': 'partner_id'}"/>
                        <filter string="Deadline" name="deadline" context="{'group_by': 'date_deadline'}"/>
                    </group>
                </search>
            </field>
        </record>

       <record id="action_project_task_user_tree" model="ir.actions.act_window">
            <field name="name">Tasks Analysis</field>
            <field name="res_model">report.project.task.user</field>
            <field name="view_mode">graph,pivot</field>
            <field name="search_view_id" ref="view_task_project_user_search"/>
            <field name="context">{'group_by_no_leaf':1, 'group_by':[], 'graph_measure': '__count__'}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_empty_folder">
                    No data yet!
                </p><p>
                    Analyze the performance of your tasks and your workers.
                </p>
            </field>
        </record>

</odoo>

```

## File: report\project_task_burndown_chart_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from psycopg2 import sql

from odoo import api, fields, models, tools
from odoo.osv import expression
from odoo.tools import OrderedSet


class ReportProjectTaskBurndownChart(models.Model):
    _name = 'project.task.burndown.chart.report'
    _description = 'Burndown Chart'
    _auto = False
    _order = 'date'

    project_id = fields.Many2one('project.project', readonly=True)
    display_project_id = fields.Many2one('project.project', readonly=True)
    stage_id = fields.Many2one('project.task.type', readonly=True)
    date = fields.Datetime('Date', readonly=True)
    user_ids = fields.Many2many('res.users', relation='project_task_user_rel', column1='task_id', column2='user_id',
                                string='Assignees', readonly=True)
    date_assign = fields.Datetime(string='Assignment Date', readonly=True)
    date_deadline = fields.Date(string='Deadline', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    nb_tasks = fields.Integer('# of Tasks', readonly=True, group_operator="sum")
    date_group_by = fields.Selection(
        [
            ('day', 'By Day'),
            ('month', 'By Month'),
            ('quarter', 'By quarter'),
            ('year', 'By Year')
        ], string="Date Group By", readonly=True)

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        date_group_bys = []
        groupby = [groupby] if isinstance(groupby, str) else list(OrderedSet(groupby))
        for gb in groupby:
            if gb.startswith('date:'):
                date_group_bys.append(gb.split(':')[-1])

        date_domains = []
        for gb in date_group_bys:
            date_domains = expression.OR([date_domains, [('date_group_by', '=', gb)]])
        domain = expression.AND([domain, date_domains])

        res = super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)
        return res

    def init(self):
        query = """
WITH all_moves_stage_task AS (
    -- Here we compute all previous stage in tracking values
    -- We're missing the last reached stage
    -- And the tasks without any stage change (which, by definition, are at the last stage)
    SELECT pt.project_id,
           pt.id as task_id,
           pt.display_project_id,
           COALESCE(LAG(mm.date) OVER (PARTITION BY mm.res_id ORDER BY mm.id), pt.create_date) as date_begin,
           mm.date as date_end,
           mtv.old_value_integer as stage_id,
           pt.date_assign,
           pt.date_deadline,
           pt.partner_id
      FROM project_task pt
      JOIN mail_message mm ON mm.res_id = pt.id
                          AND mm.message_type = 'notification'
                          AND mm.model = 'project.task'
      JOIN mail_tracking_value mtv ON mm.id = mtv.mail_message_id
      JOIN ir_model_fields imf ON mtv.field = imf.id
                              AND imf.model = 'project.task'
                              AND imf.name = 'stage_id'
      JOIN project_task_type_rel pttr ON pttr.type_id = mtv.old_value_integer
                              AND pttr.project_id = pt.project_id
     WHERE pt.active

    --We compute the last reached stage
    UNION ALL

    SELECT pt.project_id,
           pt.id as task_id,
           pt.display_project_id,
           COALESCE(md.date, pt.create_date) as date_begin,
           (CURRENT_DATE + interval '1 month')::date as date_end,
           pt.stage_id,
           pt.date_assign,
           pt.date_deadline,
           pt.partner_id
      FROM project_task pt
      LEFT JOIN LATERAL (SELECT mm.date
                      FROM mail_message mm
                      JOIN mail_tracking_value mtv ON mm.id = mtv.mail_message_id
                      JOIN ir_model_fields imf ON mtv.field = imf.id
                                              AND imf.model = 'project.task'
                                              AND imf.name = 'stage_id'
                     WHERE mm.res_id = pt.id
                       AND mm.message_type = 'notification'
                       AND mm.model = 'project.task'
                  ORDER BY mm.id DESC
                     FETCH FIRST ROW ONLY) md ON TRUE
     WHERE pt.active
)
SELECT (task_id*10^7 + 10^6 + to_char(d, 'YYMMDD')::integer)::bigint as id,
       project_id,
       task_id,
       display_project_id,
       stage_id,
       d as date,
       date_assign,
       date_deadline,
       partner_id,
       'day' AS date_group_by,
       1 AS nb_tasks
  FROM all_moves_stage_task t
  JOIN LATERAL generate_series(t.date_begin, t.date_end-interval '1 day', '1 day') d ON TRUE

UNION ALL

SELECT (task_id*10^7 + 2*10^6 + to_char(d, 'YYMMDD')::integer)::bigint as id,
       project_id,
       task_id,
       display_project_id,
       stage_id,
       date_trunc('week', d) as date,
       date_assign,
       date_deadline,
       partner_id,
       'week' AS date_group_by,
       1 AS nb_tasks
  FROM all_moves_stage_task t
  JOIN LATERAL generate_series(t.date_begin, t.date_end, '1 week') d ON TRUE
 WHERE date_trunc('week', t.date_begin) <= date_trunc('week', d)
   AND date_trunc('week', t.date_end) > date_trunc('week', d)

UNION ALL

SELECT (task_id*10^7 + 3*10^6 + to_char(d, 'YYMMDD')::integer)::bigint as id,
       project_id,
       task_id,
       display_project_id,
       stage_id,
       date_trunc('month', d) as date,
       date_assign,
       date_deadline,
       partner_id,
       'month' AS date_group_by,
       1 AS nb_tasks
  FROM all_moves_stage_task t
  JOIN LATERAL generate_series(t.date_begin, t.date_end, '1 month') d ON TRUE
 WHERE date_trunc('month', t.date_begin) <= date_trunc('month', d)
   AND date_trunc('month', t.date_end) > date_trunc('month', d)

UNION ALL

SELECT (task_id*10^7 + 4*10^6 + to_char(d, 'YYMMDD')::integer)::bigint as id,
       project_id,
       task_id,
       display_project_id,
       stage_id,
       date_trunc('quarter', d) as date,
       date_assign,
       date_deadline,
       partner_id,
       'quarter' AS date_group_by,
       1 AS nb_tasks
  FROM all_moves_stage_task t
  JOIN LATERAL generate_series(t.date_begin, t.date_end, '3 month') d ON TRUE
 WHERE date_trunc('quarter', t.date_begin) <= date_trunc('quarter', d)
   AND date_trunc('quarter', t.date_end) > date_trunc('quarter', d)

UNION ALL

SELECT (task_id*10^7 + 5*10^6 + to_char(d, 'YYMMDD')::integer)::bigint as id,
       project_id,
       task_id,
       display_project_id,
       stage_id,
       date_trunc('year', d) as date,
       date_assign,
       date_deadline,
       partner_id,
       'year' AS date_group_by,
       1 AS nb_tasks
  FROM all_moves_stage_task t
  JOIN LATERAL generate_series(t.date_begin, t.date_end, '1 year') d ON TRUE
 WHERE date_trunc('year', t.date_begin) <= date_trunc('year', d)
   AND date_trunc('year', t.date_end) > date_trunc('year', d)
        """

        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute(
            sql.SQL("CREATE or REPLACE VIEW {} as ({})").format(
                sql.Identifier(self._table),
                sql.SQL(query)
            )
        )

```

## File: report\project_task_burndown_chart_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_task_burndown_chart_report_view_search" model="ir.ui.view">
        <field name="name">project.task.burndown.chart.report.view.search</field>
        <field name="model">project.task.burndown.chart.report</field>
        <field name="arch" type="xml">
            <search string="Burndown Chart">
                <field name="project_id" />
                <field name="date_assign"/>
                <field name="date_deadline"/>
                <field name="partner_id" filter_domain="[('partner_id', 'child_of', self)]"/>
                <field name="stage_id" />
                <separator/>
                <filter name="filter_date" date="date" string="Date" />
                <filter name="filter_date_deadline" date="date_deadline"/>
                <filter name="filter_date_assign" date="date_assign"/>
                <filter string="Last Month" invisible="1" name="last_month" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=30)).strftime('%%Y-%%m-%%d'))]"/>
                <filter string="Open tasks" name="open_tasks" domain="[('stage_id.fold', '=', False), ('stage_id.is_closed', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Date" name="date" context="{'group_by': 'date'}" />
                    <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}" />
                    <filter string="Project" name="project" context="{'group_by': 'project_id'}" />
                </group>
            </search>
        </field>
    </record>

    <record id="project_task_burndown_chart_report_view_graph" model="ir.ui.view">
        <field name="name">project.task.burndown.chart.report.view.graph</field>
        <field name="model">project.task.burndown.chart.report</field>
        <field name="arch" type="xml">
            <graph string="Burndown Chart" type="line" sample="1" disable_linking="1" js_class="burndown_chart">
                <field name="date" string="Date" interval="month"/>
                <field name="stage_id"/>
                <field name="nb_tasks" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="project_task_burndown_chart_report_view_pivot" model="ir.ui.view">
        <field name="name">project.task.burndown.chart.report.view.pivot</field>
        <field name="model">project.task.burndown.chart.report</field>
        <field name="arch" type="xml">
            <pivot string="Burndown Chart" display_quantity="1" disable_linking="1" sample="1">
                <field name="date" type="row"/>
                <field name="stage_id" type="row"/>
            </pivot>
        </field>
    </record>

    <record id="action_project_task_burndown_chart_report" model="ir.actions.act_window">
        <field name="name">Burndown Chart</field>
        <field name="res_model">project.task.burndown.chart.report</field>
        <field name="view_mode">graph,pivot</field>
        <field name="search_view_id" ref="project_task_burndown_chart_report_view_search"/>
        <field name="context">{'search_default_project_id': active_id}</field>
        <field name="domain">[('display_project_id', '!=', False)]</field>
        <field name="help">
        </field>
    </record>

</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_report
from . import project_task_burndown_chart_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_project_project,project.project,model_project_project,project.group_project_user,1,0,0,0
access_project_project_manager,project.project,model_project_project,project.group_project_manager,1,1,1,1
access_project_project_stage,project.project_stage,model_project_project_stage,base.group_user,1,0,0,0
access_project_project_stage_manager,project.project_stage.manager,model_project_project_stage,project.group_project_manager,1,1,1,1
access_project_task_type_user,project.task.type.user,model_project_task_type,base.group_user,1,0,0,0
access_project_task_type_project_user,project.task.type.project.user,model_project_task_type,project.group_project_user,1,1,1,1
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
project.access_project_task_burndown_chart_report,access_project_task_burndown_chart_report,project.model_project_task_burndown_chart_report,project.group_project_manager,1,1,1,1
access_project_update_user,project.update.user,model_project_update,base.group_user,1,0,0,0
access_project_update_portal,project.update.portal,model_project_update,base.group_portal,0,0,0,0
access_project_update_project_user,project.update.project.user,model_project_update,project.group_project_user,1,1,1,1
access_project_update_project_manager,project.update.project.manager,model_project_update,project.group_project_manager,1,1,1,1
access_project_milestone_user,project.milestone.user,model_project_milestone,base.group_user,1,0,0,0
access_project_milestone_portal,project.milestone.portal,model_project_milestone,base.group_portal,0,0,0,0
access_project_milestone_project_user,project.milestone.project.user,model_project_milestone,project.group_project_user,1,0,0,0
access_project_milestone_project_manager,project.milestone.project.manager,model_project_milestone,project.group_project_manager,1,1,1,1
access_project_collaborator_manager,project.collaborator.manager,model_project_collaborator,project.group_project_manager,1,1,1,1
access_project_collaborator_user,project.collaborator.user,model_project_collaborator,project.group_project_user,1,0,0,0
access_project_collaborator_portal,project.collaborator.portal,model_project_collaborator,base.group_portal,1,0,0,0
access_project_share_manager,project.share.wizard.manager,model_project_share_wizard,project.group_project_manager,1,1,1,0
access_project_personal_stage,project.personal.stage.user,model_project_task_stage_personal,base.group_user,1,1,1,1

```

## File: security\ir.model.access.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <data noupdate="1">

        <record id="access_project_sharing_task_portal" model="ir.model.access">
            <field name="name">project_sharing_task_portal</field>
            <field name="model_id" ref="model_project_task"/>
            <field name="group_id" ref="base.group_portal"/>
            <field name="active">0</field>
            <field name="perm_read">0</field>
            <field name="perm_write">1</field>
            <field name="perm_create">1</field>
            <field name="perm_unlink">0</field>
        </record>

    </data>

</odoo>

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

    <record id="group_project_stages" model="res.groups">
        <field name="name">Use Stages on Project</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_project_recurring_tasks" model="res.groups">
        <field name="name">Use Recurring Tasks</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_project_task_dependencies" model="res.groups">
        <field name="name">Use Task Dependencies</field>
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
        <field name="name">Project: employees: following required for follower-only projects</field>
        <field name="model_id" ref="model_project_project"/>
        <field name="domain_force">['|',
                                        ('privacy_visibility', '!=', 'followers'),
                                        ('message_partner_ids', 'in', [user.partner_id.id])
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
                '&amp;',
                    ('project_id', '!=', False),
                    '|',
                        ('project_id.privacy_visibility', '!=', 'followers'),
                        ('project_id.message_partner_ids', 'in', [user.partner_id.id]),
                '|',
                    ('message_partner_ids', 'in', [user.partner_id.id]),
                    # to subscribe check access to the record, follower is not enough at creation
                    ('user_ids', 'in', user.id)
        ]</field>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record model="ir.rule" id="project_manager_all_project_tasks_rule">
        <field name="name">Project/Task: project manager: see all</field>
        <field name="model_id" ref="model_project_task"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('project.group_project_manager'))]"/>
    </record>

    <record model="ir.rule" id="task_type_manager_rule">
        <field name="name">Project/Task Type: manager sees all</field>
        <field name="model_id" ref="model_project_task_type"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('project.group_project_manager'))]"/>
    </record>

    <record model="ir.rule" id="task_type_visibility_rule">
        <field name="name">Project/Task Type: see own or unowned stages</field>
        <field name="model_id" ref="model_project_task_type"/>
        <field name="domain_force">[('user_id', 'in', (False, user.id))]</field>
    </record>

    <record model="ir.rule" id="task_type_own_write_rule">
        <field name="name">Project/Task Type: write own stages</field>
        <field name="model_id" ref="model_project_task_type"/>
        <field name="domain_force">[('user_id', '=', user.id)]</field>
        <field name="perm_read" eval="False"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_unlink" eval="True"/>
        <field name="groups" eval="[(4,ref('project.group_project_user'))]"/>
    </record>

    <record model="ir.rule" id="report_project_task_user_report_comp_rule">
        <field name="name">Task Analysis multi-company</field>
        <field name="model_id" ref="model_report_project_task_user"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record id="ir_rule_project_personal_stage_my" model="ir.rule">
        <field name="name">Project: See my own personal stage</field>
        <field name="model_id" ref="project.model_project_task_stage_personal"/>
        <field name="domain_force">[('user_id', '=', user.id)]</field>
    </record>

    <record id="ir_rule_private_task" model="ir.rule">
        <field name="name">Project: See private tasks</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="domain_force">[
            ('project_id.privacy_visibility', '!=', 'followers'),
            '|', '|', ('project_id', '!=', False),
                      ('parent_id', '!=', False),
                 ('user_ids', 'in', user.id),
        ]</field>
        <field name="groups" eval="[(4,ref('project.group_project_user'))]"/>
    </record>

     <!-- Portal -->
    <record id="project_project_rule_portal" model="ir.rule">
        <field name="name">Project: portal users: portal and following</field>
        <field name="model_id" ref="project.model_project_project"/>
        <field name="domain_force">[
            '&amp;',
                ('privacy_visibility', '=', 'portal'),
                ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record id="project_collaborator_rule_portal" model="ir.rule">
        <field name="name">Project/Collaborator: portal users: can only see his own collobaroration in shared projects</field>
        <field name="model_id" ref="project.model_project_collaborator"/>
        <field name="domain_force">[
            ('project_id.privacy_visibility', '=', 'portal'),
            ('partner_id', '=', user.partner_id.id),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record id="project_task_rule_portal" model="ir.rule">
        <field name="name">Project/Task: portal users: (portal and following project) or (portal and following task)</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="domain_force">[
        ('project_id.privacy_visibility', '=', 'portal'),
        ('active', '=', True),
        '|',
            ('project_id.message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
            ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
        ]</field>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record id="project_task_rule_portal_project_sharing" model="ir.rule">
        <field name="name">Project/Task: portal users: portal user can edit with project sharing feature</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="active">0</field>
        <field name="domain_force">[
            ('project_id.privacy_visibility', '=', 'portal'),
            ('active', '=', True),
            '|',
                ('project_id.message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
                ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
            ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="False"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record model="ir.rule" id="update_comp_rule">
        <field name="name">Project/Updates: multi-company</field>
        <field name="model_id" ref="model_project_update"/>
        <field name="domain_force">[('project_id.company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="update_visibility_rule">
        <field name="name">Project/Update: employees: follow required for follower-only projects</field>
        <field name="model_id" ref="model_project_update"/>
        <field name="domain_force">[
        '|',
            ('project_id.privacy_visibility', '!=', 'followers'),
            '|',
                ('project_id.message_partner_ids', 'in', [user.partner_id.id]),
                '|',
                    ('user_id', '=', user.id),
                    ('project_id.user_id', '=', user.id)
        ]</field>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="update_visibility_project_admin" model="ir.rule">
        <field name="name">Project updates : Project user can see all project updates</field>
        <field name="model_id" ref="project.model_project_update"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('project.group_project_manager'))]"/>
    </record>

    <record model="ir.rule" id="milestone_comp_rule">
        <field name="name">Project/Milestone: multi-company</field>
        <field name="model_id" ref="model_project_milestone"/>
        <field name="domain_force">[('project_id.company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="milestone_visibility_rule">
        <field name="name">Project/Milestone: employees: follow required for follower-only projects</field>
        <field name="model_id" ref="model_project_milestone"/>
        <field name="domain_force">[
        '|',
            ('project_id.privacy_visibility', '!=', 'followers'),
            '|',
                ('project_id.message_partner_ids', 'in', [user.partner_id.id]),
                ('project_id.user_id', '=', user.id),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="milestone_visibility_project_admin" model="ir.rule">
        <field name="name">Project/Milestone: Project manager can see all project milestones</field>
        <field name="model_id" ref="project.model_project_milestone"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('project.group_project_manager'))]"/>
    </record>
</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="98.162%" x2="0%" y1="1.838%" y2="100%"><stop offset="0%" stop-color="#797DA5"/><stop offset="50.799%" stop-color="#6D7194"/><stop offset="100%" stop-color="#626584"/></linearGradient><path id="d" d="M55.253 36.993c-3.23 0-4.629 2.41-6.384 2.41-4.666 0-.419-13.45-.419-13.45s-15.27 6.106-15.27-.251c0-2.735 2.822-3.53 2.822-6.563 0-2.709-2.187-4.176-4.781-4.176-2.696 0-5.163 1.442-5.163 4.3 0 3.158 2.467 4.525 2.467 6.24 0 5.313-13.684 2.187-13.684 2.187v25.432s13.898 3.133 13.898-2.187c0-1.715-3.112-3.061-3.112-6.218 0-2.859 2.275-4.3 4.946-4.3 2.62 0 4.807 1.466 4.807 4.176 0 3.032-2.823 3.828-2.823 6.562 0 4.64 10.088 1.963 14.1 1.963 0 0-2.702-9.165 2.009-9.165 2.797 0 3.611 2.759 6.714 2.759 2.773 0 4.273-2.138 4.273-4.698 0-2.61-1.475-5.021-4.4-5.021z"/><path id="e" d="M55.253 34.993c-3.23 0-4.629 2.41-6.384 2.41-4.666 0 1.522-11.755-.419-13.45-1.94-1.695-15.27 6.106-15.27-.251 0-2.735 2.822-3.53 2.822-6.563 0-2.709-2.187-4.176-4.781-4.176-2.696 0-5.163 1.442-5.163 4.3 0 3.158 2.467 4.525 2.467 6.24 0 5.313-13.684 2.187-13.684 2.187v25.432s13.898 3.133 13.898-2.187c0-1.715-3.112-3.061-3.112-6.218 0-2.859 2.275-4.3 4.946-4.3 2.62 0 4.807 1.466 4.807 4.176 0 3.032-2.823 3.828-2.823 6.562 0 4.64 12.677 2.6 14.1 1.963 1.422-.636-2.702-9.165 2.009-9.165 2.797 0 3.611 2.759 6.714 2.759 2.773 0 4.273-2.138 4.273-4.698 0-2.61-1.475-5.021-4.4-5.021z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 70c-2 0-4-.148-4-4.15V39.197L27 15l11 24.84 6 .066 11.113-.066 4.21 2.112L42.511 70H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\burndown_chart\burndown_chart_renderer.js

```javascript
/** @odoo-module **/

import { hexToRGBA } from "@web/views/graph/colors";
import { GraphRenderer } from "@web/views/graph/graph_renderer";

export class BurndownChartRenderer extends GraphRenderer {
    /**
     * @override
     */
    getLineChartData() {
        const data = super.getLineChartData();
        const { stacked } = this.model.metaData;
        if (stacked) {
            for (const dataset of data.datasets) {
                dataset.backgroundColor = hexToRGBA(dataset.borderColor, 0.4);
            }
        }
        return data;
    }

    /**
     * @override
     */
    getElementOptions() {
        const elementOptions = super.getElementOptions();
        const { mode, stacked } = this.model.metaData;
        if (mode === "line") {
            elementOptions.line.fill = stacked;
        }
        return elementOptions;
    }

    /**
     * @override
     */
    getScaleOptions() {
        const { xAxes, yAxes } = super.getScaleOptions();
        const { mode, stacked } = this.model.metaData;
        if (mode === "line") {
            for (const y of yAxes) {
                y.stacked = stacked;
            }
        }
        return { xAxes, yAxes };
    }
}

```

## File: static\src\burndown_chart\burndown_chart_view.js

```javascript
/** @odoo-module **/

import { BurndownChartRenderer } from "./burndown_chart_renderer";
import { GraphView } from "@web/views/graph/graph_view";
import { registry } from "@web/core/registry";

const viewRegistry = registry.category("views");

class BurndownChartView extends GraphView {}
BurndownChartView.components = { ...GraphView.components, Renderer: BurndownChartRenderer };
BurndownChartView.buttonTemplate = "project.BurndownChartView.Buttons";

viewRegistry.add("burndown_chart", BurndownChartView);

```

## File: static\src\burndown_chart\burndown_chart_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.BurndownChartView.Buttons" t-inherit="web.GraphView.Buttons" t-inherit-mode="primary" owl="1">
        <xpath expr="//button[@data-mode='pie']" position="replace">
        </xpath>
        <xpath expr="//div[@role='toolbar'][3]" position="attributes">
            <attribute name="t-if">true</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\js\portal_rating.js

```javascript
/** @odoo-module **/

import time from 'web.time';
import publicWidget from 'web.public.widget';

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

```

## File: static\src\js\project_activity.js

```javascript
/** @odoo-module **/

import ActivityView from '@mail/js/views/activity/activity_view';
import { ProjectControlPanel } from '@project/js/project_control_panel';
import viewRegistry from 'web.view_registry';

const ProjectActivityView = ActivityView.extend({
    config: Object.assign({}, ActivityView.prototype.config, {
        ControlPanel: ProjectControlPanel,
    }),
});

viewRegistry.add('project_activity', ProjectActivityView);

```

## File: static\src\js\project_calendar.js

```javascript
/** @odoo-module **/

import CalendarController from 'web.CalendarController';
import CalendarView from 'web.CalendarView';
import viewRegistry from 'web.view_registry';
import { ProjectControlPanel } from '@project/js/project_control_panel';

const ProjectCalendarController = CalendarController.extend({
    _renderButtonsParameters() {
        return Object.assign({}, this._super(...arguments), {scaleDrop: true});
    },

    /**
     * @private
     * @param {OdooEvent} event
     */
    _onChangeDate: function (event) {
        this._super.apply(this, arguments);
        this.model.setScale('month');
        this.model.setDate(event.data.date);
        this.reload();
    },
});

export const ProjectCalendarView = CalendarView.extend({
    config: Object.assign({}, CalendarView.prototype.config, {
        Controller: ProjectCalendarController,
        ControlPanel: ProjectControlPanel,
    }),
});

viewRegistry.add('project_calendar', ProjectCalendarView);

```

## File: static\src\js\project_control_panel.js

```javascript
/** @odoo-module **/

import ControlPanel from 'web.ControlPanel';
import session from 'web.session';

export class ProjectControlPanel extends ControlPanel {

    constructor() {
        super(...arguments);
        this.show_project_update = this.env.view.type === "form" || this.props.action.context.show_project_update;
        this.project_id = this.show_project_update ? this.props.action.context.active_id : false;
    }

    async willStart() {
        const promises = [];
        promises.push(super.willStart(...arguments));
        promises.push(this._loadWidgetData());
        return Promise.all(promises);
    }

    async willUpdateProps() {
        const promises = [];
        promises.push(super.willUpdateProps(...arguments));
        promises.push(this._loadWidgetData());
        return Promise.all(promises);
    }

    async _loadWidgetData() {
        if (this.show_project_update) {
            this.data = await this.rpc({
                model: 'project.project',
                method: 'get_last_update_or_default',
                args: [this.project_id],
            });
            this.is_project_user = await session.user_has_group('project.group_project_user');
        }
    }

    async onStatusClick(ev) {
        ev.preventDefault();
        await this.trigger('do-action', {
            action: "project.project_update_all_action",
            options: {
                additional_context: {
                    default_project_id: this.project_id,
                    active_id: this.project_id
                }
            }
        });
    }
}

```

## File: static\src\js\project_form.js

```javascript
/** @odoo-module **/

import Dialog from 'web.Dialog';
import FormView from 'web.FormView';
import FormController from 'web.FormController';
import { bus, _t } from 'web.core';
import { device } from 'web.config';
import viewRegistry from 'web.view_registry';

const ProjectFormController = FormController.extend({
    on_attach_callback() {
        this._super(...arguments);
        if (!device.isMobile) {
            bus.on("DOM_updated", this, this._onDomUpdated);
        }
    },
    _onDomUpdated() {
        const $editable = this.$el.find('[name="description"] .note-editable');
        if ($editable.length) {
            const minHeight = window.innerHeight - $editable.offset().top - 42;
            $editable.css('min-height', minHeight + 'px');
        }
    },
    on_detach_callback() {
        this._super(...arguments);
        bus.off('DOM_updated', this._onDomUpdated);
    },
    _getActionMenuItems(state) {
        if (!this.archiveEnabled || !state.data['recurrence_id']) {
            return this._super(...arguments);
        }

        this.archiveEnabled = false;
        let actions = this._super(...arguments);
        this.archiveEnabled = true;

        if (actions) {
            const activeField = this.model.getActiveField(state);
            actions.items.other.unshift({
                description: state.data[activeField] ? _t('Archive') : _t('Unarchive'),
                callback: () => this._stopRecurrence(state.data['id'], state.data[activeField] ? 'archive' : 'unarchive'),
            });
        }

        return actions;
    },

    _onDeleteRecord() {
        const record = this.model.get(this.handle);

        if (!record.data.recurrence_id) {
            return this._super(...arguments);
        }
        this._stopRecurrence(record.res_id, 'delete');
    },

    _countTasks(recurrence_id) {
        return this._rpc({
            model: 'project.task',
            method: 'search_count',
            args: [[["recurrence_id", "=", recurrence_id.res_id]]],
        });
    },

    async _stopRecurrence(resId, mode) {
        const record = this.model.get(this.handle);
        const recurrence_id = record.data.recurrence_id;
        const count = await this._countTasks(recurrence_id);
        const allowContinue = count != 1;

        const alert = allowContinue
            ? _t('It seems that this task is part of a recurrence.')
            : _t('It seems that this task is part of a recurrence. You must keep it as a model to create the next occurences.');
        const dialog = new Dialog(this, {
            buttons: [
                {
                    classes: 'btn-primary',
                    click: () => {
                        this._rpc({
                            model: 'project.task',
                            method: 'action_stop_recurrence',
                            args: [resId],
                        }).then(() => {
                            if (mode === 'archive') {
                                this._toggleArchiveState(true);
                            } else if (mode === 'unarchive') {
                                this._toggleArchiveState(false);
                            } else if (mode === 'delete') {
                                this._deleteRecords([this.handle]);
                            }
                        });
                    },
                    close: true,
                    text: _t('Stop Recurrence'),
                },
                {
                    close: true,
                    text: _t('Discard'),
                }
            ],
            size: 'medium',
            title: _t('Confirmation'),
            $content: $('<main/>', {
                role: 'alert',
                text: alert,
            }),
        });

        if (allowContinue) {
            dialog.buttons.splice(1, 0,
                {
                    click: () => {
                        this._rpc({
                            model: 'project.task',
                            method: 'action_continue_recurrence',
                            args: [resId],
                        }).then(() => {
                            if (mode === 'archive') {
                                this._toggleArchiveState(true);
                            } else if (mode === 'unarchive') {
                                this._toggleArchiveState(false);
                            } else if (mode === 'delete') {
                                this._deleteRecords([this.handle]);
                            }
                        });
                    },
                    close: true,
                    text: _t('Continue Recurrence'),
                })
        };

        dialog.open();
    }
});

export const ProjectFormView = FormView.extend({
    config: Object.assign({}, FormView.prototype.config, {
        Controller: ProjectFormController,
    }),
});

viewRegistry.add('project_form', ProjectFormView);

```

## File: static\src\js\project_graph_legacy.js

```javascript
/** @odoo-module **/

import GraphView from 'web.GraphView';
import { ProjectControlPanel } from '@project/js/project_control_panel';
import viewRegistry from 'web.view_registry';

export const ProjectGraphView = GraphView.extend({
  config: Object.assign({}, GraphView.prototype.config, {
    ControlPanel: ProjectControlPanel,
  }),
});

viewRegistry.add('project_graph', ProjectGraphView);

```

## File: static\src\js\project_graph_view.js

```javascript
/** @odoo-module **/

import { ProjectControlPanel } from "@project/project_control_panel/project_control_panel";
import { registry } from "@web/core/registry";
import { GraphView } from "@web/views/graph/graph_view";

const viewRegistry = registry.category("views");

export class ProjectGraphView extends GraphView {}
ProjectGraphView.ControlPanel = ProjectControlPanel;

viewRegistry.add("project_graph", ProjectGraphView);

```

## File: static\src\js\project_kanban.js

```javascript
/** @odoo-module **/

import KanbanController from 'web.KanbanController';
import KanbanRenderer from 'web.KanbanRenderer';
import KanbanView from 'web.KanbanView';
import KanbanColumn from 'web.KanbanColumn';
import KanbanRecord from 'web.KanbanRecord';
import KanbanModel from 'web.KanbanModel';
import viewRegistry from 'web.view_registry';
import { ProjectControlPanel } from '@project/js/project_control_panel';
import viewUtils from 'web.viewUtils';
import { Domain } from '@web/core/domain';
import view_dialogs from 'web.view_dialogs';
import core from 'web.core';

const _t = core._t;

// PROJECTS

const ProjectProjectKanbanRecord = KanbanRecord.extend({
    /**
     * @override
     * @private
     */
    _openRecord: function () {
        const kanbanBoxesElement = this.el.querySelectorAll('.o_project_kanban_boxes a');
        if (this.selectionMode !== true && kanbanBoxesElement.length) {
            kanbanBoxesElement[0].click();
        } else {
            this._super.apply(this, arguments);
        }
    },
    /**
     * @override
     * @private
     */
    _onManageTogglerClicked: function (event) {
        this._super.apply(this, arguments);
        const thisSettingToggle = this.el.querySelector('.o_kanban_manage_toggle_button');
        this.el.parentNode.querySelectorAll('.o_kanban_manage_toggle_button.show').forEach(el => {
            if (el !== thisSettingToggle) {
                el.classList.remove('show');
            }
        });
        thisSettingToggle.classList.toggle('show');
    },
});

const ProjectProjectKanbanRenderer = KanbanRenderer.extend({
    config: _.extend({}, KanbanRenderer.prototype.config, {
        KanbanRecord: ProjectProjectKanbanRecord,
    }),
});

const ProjectProjectKanbanView = KanbanView.extend({
    config: Object.assign({}, KanbanView.prototype.config, {
        Renderer: ProjectProjectKanbanRenderer,
    })
});

viewRegistry.add('project_project_kanban', ProjectProjectKanbanView);

// TASKS

const ProjectTaskKanbanColumn = KanbanColumn.extend({
    /**
     * @override
     * @private
     */
    _onDeleteColumn: function (event) {
        if (this.groupedBy === 'stage_id') {
            event.preventDefault();
            this.trigger_up('kanban_column_delete_wizard');
        } else {
            this._super(...arguments);
        }
    },

    /**
     * Open alternative view when editing personal stages.
     *
     * @private
     * @override
     */
    _onEditColumn: function (event) {
        if (this.groupedBy !== 'personal_stage_type_ids') {
            this._super(...arguments);
            return;
        }
        event.preventDefault();
        const context = Object.assign({}, this.getSession().user_context, {
            form_view_ref: 'project.personal_task_type_edit',
        });
        new view_dialogs.FormViewDialog(this, {
            res_model: this.relation,
            res_id: this.id,
            context: context,
            title: _t("Edit Personal Stage"),
            on_saved: this.trigger_up.bind(this, 'reload'),
        }).open();
    },
});

const ProjectTaskKanbanRenderer = KanbanRenderer.extend({
    config: Object.assign({}, KanbanRenderer.prototype.config, {
        KanbanColumn: ProjectTaskKanbanColumn,
    }),

    init: function () {
        this._super.apply(this, arguments);
        this.isProjectManager = false;
    },

    willStart: function () {
        const superPromise = this._super.apply(this, arguments);

        const isProjectManager = this.getSession().user_has_group('project.group_project_manager').then((hasGroup) => {
            this.isProjectManager = hasGroup;
            this._setState();
            return Promise.resolve();
        });

        return Promise.all([superPromise, isProjectManager]);
    },

    /**
     * Allows record drag when grouping by `personal_stage_type_ids`
     *
     * @override
     */
    _setState() {
        this._super(...arguments);
        const groupedBy = this.state.groupedBy[0];
        const groupByFieldName = viewUtils.getGroupByField(groupedBy);
        const field = this.state.fields[groupByFieldName] || {};
        const fieldInfo = this.state.fieldsInfo.kanban[groupByFieldName] || {};

        const grouped_by_date = ["date", "datetime"].includes(field.type);
        const grouped_by_m2m = field.type === "many2many";
        const readonly = !!field.readonly || !!fieldInfo.readonly;
        const groupedByPersonalStage = (groupByFieldName === 'personal_stage_type_ids');

        const draggable = !readonly && (!grouped_by_m2m || groupedByPersonalStage) &&
            (!grouped_by_date || fieldInfo.allowGroupRangeValue);

        // When grouping by personal stage we allow any project user to create
        let editable = this.columnOptions.editable;
        let deletable = this.columnOptions.deletable;
        if (['stage_id', 'personal_stage_type_ids'].includes(groupByFieldName)) {
            this.groupedByM2O = groupedByPersonalStage || this.groupedByM2O;
            const allow_crud = this.isProjectManager || groupedByPersonalStage;
            this.createColumnEnabled = editable = deletable = allow_crud;
        }

        Object.assign(this.columnOptions, {
            draggable,
            grouped_by_m2o: this.groupedByM2O,
            editable: editable,
            deletable: deletable,
        });
    }
});

export const ProjectKanbanController = KanbanController.extend({
    custom_events: Object.assign({}, KanbanController.prototype.custom_events, {
        'kanban_column_delete_wizard': '_onDeleteColumnWizard',
    }),

    _onDeleteColumnWizard: function (ev) {
        ev.stopPropagation();
        const self = this;
        const columnId = ev.target.id;
        const state = this.model.get(this.handle, {raw: true});
        this._rpc({
            model: 'project.task.type',
            method: 'unlink_wizard',
            args: [columnId],
            context: state.getContext(),
        }).then(function (res) {
            self.do_action(res);
        });
    },

    /**
     * @override
     */
    _onDeleteColumn: function (ev) {
        const state = this.model.get(this.handle, {raw: true});
        const groupedByFieldname = state.groupedBy[0];
        if (groupedByFieldname !== 'personal_stage_type_ids') {
            this._super(...arguments);
            return;
        }
        const column = ev.target;
        this._rpc({
            model: 'project.task.type',
            method: 'remove_personal_stage',
            args: [[column.id]],
        }).then(this.update.bind(this, {}, {}));
    },
});

const ProjectTaskKanbanModel = KanbanModel.extend({

    /**
     * Upon updating `personal_stage_type_ids` we actually want to update the `personal_stage_type_id` field.
     *
     * @override
     * @private
     */
    moveRecord: function (recordID, groupID, parentID) {
        const self = this;
        const parent = this.localData[parentID];
        const new_group = this.localData[groupID];
        const changes = {};
        const groupedFieldName = viewUtils.getGroupByField(parent.groupedBy[0]);
        const groupedField = parent.fields[groupedFieldName];
        // for a date/datetime field, we take the last moment of the group as the group value
        if (['date', 'datetime'].includes(groupedField.type)) {
            changes[groupedFieldName] = viewUtils.getGroupValue(new_group, groupedFieldName);
        } else if (groupedField.type === 'many2one') {
            changes[groupedFieldName] = {
                id: new_group.res_id,
                display_name: new_group.value,
            };
        } else if (groupedField.type === 'selection') {
            const value = _.findWhere(groupedField.selection, {1: new_group.value});
            changes[groupedFieldName] = value && value[0] || false;
        } else if (groupedField.type == 'many2many' && groupedFieldName == 'personal_stage_type_ids') {
            changes['personal_stage_type_id'] = {
                id: new_group.res_id,
                display_name: new_group.value,
            }
        } else {
            changes[groupedFieldName] = new_group.value;
        }

        // Manually updates groups data. Note: this is done before the actual
        // save as it might need to perform a read group in some cases so those
        // updated data might be overridden again.
        const record = self.localData[recordID];
        const resID = record.res_id;
        // Remove record from its current group
        let old_group;
        for (let i = 0; i < parent.data.length; i++) {
            old_group = self.localData[parent.data[i]];
            const index = _.indexOf(old_group.data, recordID);
            if (index >= 0) {
                old_group.data.splice(index, 1);
                old_group.count--;
                if (!old_group.activeFilter || old_group.activeFilter.value === record.data[parent.progressBar.field]) {
                    // Here, the record leaving the old group matches its domain,
                    // so we must decrease the domainCount too.
                    old_group.domainCount--;
                }
                old_group.res_ids = _.without(old_group.res_ids, resID);
                self._updateParentResIDs(old_group);
                break;
            }
        }
        // Add record to its new group
        new_group.data.push(recordID);
        new_group.res_ids.push(resID);
        new_group.count++;

        return this.notifyChanges(recordID, changes).then(function () {
            return self.save(recordID);
        }).then(function () {
            record.parentID = new_group.id;
            return [old_group.id, new_group.id];
        });
    },

    /**
     * When grouped by personal stage create a new personal stage instead of
     * a regular stage.
     * Meaning setting `user_id` on the stage.
     *
     * @override
     */
    createGroup: function (name, parentID) {
        const parent = this.localData[parentID];
        const groupedFieldName = viewUtils.getGroupByField(parent.groupedBy[0]);
        if (groupedFieldName !== 'personal_stage_type_ids') {
            return this._super(...arguments);
        }
        const groupBy = parent.groupedBy[0];
        const context = Object.assign({}, parent.context, {
            default_user_id: this.getSession().uid,
        });
        // In case it's a personal stage we don't want to assign it to the project.
        delete context.default_project_id;
        return this._rpc({
                model: 'project.task.type',
                method: 'name_create',
                args: [name],
                context: context,
            })
            .then((result) => {
                const createGroupDataPoint = (model, parent) => {
                    const newGroup = model._makeDataPoint({
                        modelName: parent.model,
                        context: parent.context,
                        domain: parent.domain.concat([[groupBy, "=", result[0]]]),
                        fields: parent.fields,
                        fieldsInfo: parent.fieldsInfo,
                        isOpen: true,
                        limit: parent.limit,
                        parentID: parent.id,
                        openGroupByDefault: true,
                        orderedBy: parent.orderedBy,
                        value: result,
                        viewType: parent.viewType,
                    });
                    if (parent.progressBar) {
                        newGroup.progressBarValues = _.extend({
                            counts: {},
                        }, parent.progressBar);
                    }
                    return newGroup;
                };
                const newGroup = createGroupDataPoint(this, parent);
                parent.data.push(newGroup.id);
                if (this.isInSampleMode()) {
                    // in sample mode, create the new group in both models (main + sample)
                    const sampleParent = this.sampleModel.localData[parentID];
                    const newSampleGroup = createGroupDataPoint(this.sampleModel, sampleParent);
                    sampleParent.data.push(newSampleGroup.id);
                }
                return newGroup.id;
            });
    },

    /**
     * Force tasks assigned to the user when grouping by personal stage.
     *
     * @override
     * @private
     */
    _readGroup: function (list) {
        const groupedBy = list.groupedBy[0];
        if (groupedBy === 'personal_stage_type_ids') {
            list.domain = Domain.and([
                [['user_ids', 'in', this.getSession().uid]],
                list.domain
            ]).toList();
        }
        return this._super(...arguments);
    },
})

const ProjectKanbanView = KanbanView.extend({
    config: _.extend({}, KanbanView.prototype.config, {
        Model: ProjectTaskKanbanModel,
        Controller: ProjectKanbanController,
        Renderer: ProjectTaskKanbanRenderer,
        ControlPanel: ProjectControlPanel,
    }),
});

viewRegistry.add('project_task_kanban', ProjectKanbanView);

```

## File: static\src\js\project_list.js

```javascript
/** @odoo-module **/

import { _t } from 'web.core';
import Dialog from 'web.Dialog';
import ListView from 'web.ListView';
import ListController from 'web.ListController';
import viewRegistry from 'web.view_registry';
import { ProjectControlPanel } from '@project/js/project_control_panel';

const ProjectListController = ListController.extend({
    _getActionMenuItems(state) {
        if (!this.archiveEnabled) {
            return this._super(...arguments);
        }

        const recurringRecords = this.getSelectedRecords().filter(rec => rec.data.recurrence_id).map(rec => rec.data.id);
        this.archiveEnabled = recurringRecords.length == 0;
        let actions = this._super(...arguments);
        this.archiveEnabled = true;

        if (actions && recurringRecords.length > 0) {
            actions.items.other.unshift({
                description: _t('Archive'),
                callback: () => this._stopRecurrence(recurringRecords, this.selectedRecords, 'archive'),
            }, {
                description: _t('Unarchive'),
                callback: () => this._toggleArchiveState(false)
            });
        }
        return actions;
    },

    _onDeleteSelectedRecords() {
        const recurringRecords = this.getSelectedRecords().filter(rec => rec.data.recurrence_id).map(rec => rec.data.id);
        if (recurringRecords.length > 0) {
            return this._stopRecurrence(recurringRecords, this.selectedRecords, 'delete');
        }

        return this._super(...arguments);
    },

    _countRecordsPerReccurence(recurrenceIds, resIds) {
        return this._rpc({
            model: 'project.task',
            method: 'read_group',
            args: [
                [['recurrence_id', 'in', recurrenceIds], ['id', 'not in', resIds]],
                ['recurrence_id'],
                ['recurrence_id'],
            ],
        });
    },

    async _stopRecurrence(recurringResIds, resIds, mode) {
        const recurrenceIdsSet = new Set();
        for (const record of this.getSelectedRecords()) {
            const recurrenceId = record.data.recurrence_id;
            if (recurrenceId) {
                recurrenceIdsSet.add(recurrenceId);
            }
        }
        const recurrenceIds = Array.from(recurrenceIdsSet);
        // list recurrences that have tasks left after deleting/archiving
        let countsLeft = await this._countRecordsPerReccurence(recurrenceIds, recurringResIds);
        countsLeft = countsLeft.map(rec => rec.recurrence_id[0]);
        // so we check that no recurrence is absent, as it would mean no task is left
        const allowContinue = recurrenceIds.every(rec => countsLeft.includes(rec));

        let warning;
        if (resIds.length > 1) {
            warning = allowContinue
                    ? _t('It seems that some tasks are part of a recurrence.')
                    : _t('It seems that some tasks are part of a recurrence. At least one of them must be kept as a model to create the next occurences.');
        } else {
            warning = allowContinue
                    ? _t('It seems that this task is part of a recurrence.')
                    : _t('It seems that this task is part of a recurrence. You must keep it as a model to create the next occurences.');
        }

        const dialog = new Dialog(this, {
            buttons: [
                {
                    classes: 'btn-primary',
                    click: () => {
                        this._rpc({
                            model: 'project.task',
                            method: 'action_stop_recurrence',
                            args: [recurringResIds],
                        }).then(() => {
                            if (mode === 'archive') {
                                this._toggleArchiveState(true);
                            } else if (mode === 'delete') {
                                this._deleteRecords(resIds);
                            }
                        });
                    },
                    close: true,
                    text: _t('Stop Recurrence'),
                },
                {
                    close: true,
                    text: _t('Discard'),
                }
            ],
            size: 'medium',
            title: _t('Confirmation'),
            $content: $('<main/>', {
                role: 'alert',
                text: warning,
            }),
        });

        if (allowContinue) {
            Dialog.buttons.splice(1, 0,
                {
                    click: () => {
                        this._rpc({
                            model: 'project.task',
                            method: 'action_continue_recurrence',
                            args: [recurringResIds],
                        }).then(() => {
                            if (mode === 'archive') {
                                this._toggleArchiveState(true);
                            } else if (mode === 'delete') {
                                this._deleteRecords(resIds);
                            }
                        });
                },
                close: true,
                text: _t('Continue Recurrence'),
            });
        }

        dialog.open();
    }
});

export const ProjectListView = ListView.extend({
    config: Object.assign({}, ListView.prototype.config, {
        Controller: ProjectListController,
        ControlPanel: ProjectControlPanel,
    }),
});

viewRegistry.add('project_list', ProjectListView);

```

## File: static\src\js\project_pivot_legacy.js

```javascript
/** @odoo-module **/

import PivotView from 'web.PivotView';
import { ProjectControlPanel } from '@project/js/project_control_panel';
import viewRegistry from 'web.view_registry';

export const ProjectPivotView = PivotView.extend({
  config: Object.assign({}, PivotView.prototype.config, {
    ControlPanel: ProjectControlPanel,
  }),
});

viewRegistry.add('project_pivot', ProjectPivotView);

```

## File: static\src\js\project_pivot_view.js

```javascript
/** @odoo-module **/

import { ProjectControlPanel } from "@project/project_control_panel/project_control_panel";
import { registry } from "@web/core/registry";
import { PivotView } from "@web/views/pivot/pivot_view";

const viewRegistry = registry.category("views");

class ProjectPivotView extends PivotView {}
ProjectPivotView.ControlPanel = ProjectControlPanel;

viewRegistry.add("project_pivot", ProjectPivotView);

```

## File: static\src\js\project_rating_graph_view.js

```javascript
/** @odoo-module **/

import { _lt } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { GraphArchParser } from "@web/views/graph/graph_arch_parser";
import { GraphView } from "@web/views/graph/graph_view";

const viewRegistry = registry.category("views");

const MEASURE_STRINGS = {
    parent_res_id: _lt("Project"),
    rating: _lt("Rating Value (/5)"),
    res_id: _lt("Task"),
};

class ProjectRatingArchParser extends GraphArchParser {
    parse() {
        const archInfo = super.parse(...arguments);
        for (const [key, val] of Object.entries(MEASURE_STRINGS)) {
            archInfo.fieldAttrs[key] = {
                ...archInfo.fieldAttrs[key],
                string: val.toString(),
            };
        }
        return archInfo;
    }
}

// Would it be not better achiedved by using a proper arch directly?

class ProjectRatingGraphView extends GraphView {}
ProjectRatingGraphView.ArchParser = ProjectRatingArchParser;

viewRegistry.add("project_rating_graph", ProjectRatingGraphView);

```

## File: static\src\js\project_rating_pivot_view.js

```javascript
/** @odoo-module **/

import { _lt } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { PivotArchParser } from "@web/views/pivot/pivot_arch_parser";
import { PivotView } from "@web/views/pivot/pivot_view";

const viewRegistry = registry.category("views");

const MEASURE_STRINGS = {
    parent_res_id: _lt("Project"),
    rating: _lt("Rating Value (/5)"),
    res_id: _lt("Task"),
};

class ProjectRatingArchParser extends PivotArchParser {
    parse() {
        const archInfo = super.parse(...arguments);
        for (const [key, val] of Object.entries(MEASURE_STRINGS)) {
            archInfo.fieldAttrs[key] = {
                ...archInfo.fieldAttrs[key],
                string: val.toString(),
            };
        }
        return archInfo;
    }
}

// Would it be not better achiedved by using a proper arch directly?

class ProjectRatingPivotView extends PivotView {}
ProjectRatingPivotView.ArchParser = ProjectRatingArchParser;

viewRegistry.add("project_rating_pivot", ProjectRatingPivotView);

```

## File: static\src\js\project_task_kanban_examples.js

```javascript
/** @odoo-module **/

import { _lt } from 'web.core';
import {Markup} from 'web.utils';
import kanbanExamplesRegistry from 'web.kanban_examples_registry';

const greenBullet = Markup`<span class="o_status d-inline-block o_status_green"></span>`;
const redBullet = Markup`<span class="o_status d-inline-block o_status_red"></span>`;
const star = Markup`<a style="color: gold;" class="fa fa-star"/>`;
const clock = Markup`<a class="fa fa-clock-o" />`;

const descriptionActivities = escFormat(_lt('%s Use the %s icon to organize your daily activities.'), '<br/>', clock);
const description = escFormat(_lt('Prioritize Tasks by using the %s icon.' +
            '%s Use the %s button to signalize to your colleagues that a task is ready for the next stage.' +
            '%s Use the %s to signalize a problem or a need for discussion on a task.' +
            '%s'), star, '<br/>', greenBullet, '<br/>', redBullet, descriptionActivities);

/**
 * Helper function to escape the format string, but not the values formatted in,
 * rougly equivalent to `_.str.sprintf(_.escape(fmt), ...)`.
 *
 * Performs the escaping lazily (at point of use) as it's designed to be used
 * with `_lt` as the source of the format string.
 *
 * @param fmt format string, to escape
 * @param args objects to format into `fmt`, unescaped
 * @returns {Object} a stringifiable object returning the escaped then formatted string
 */
function escFormat(fmt, ...args) {
    return {
        [_.escapeMethod]() {
            return this;
        },
        toString() {
            return _.str.sprintf(_.escape(fmt), ...args);
        },
    };
}

kanbanExamplesRegistry.add('project', {
    ghostColumns: [_lt('New'), _lt('Assigned'), _lt('In Progress'), _lt('Done')],
    applyExamplesText: _lt("Use This For My Project"),
    examples:[{
        name: _lt('Software Development'),
        columns: [_lt('Backlog'), _lt('Specifications'), _lt('Development'), _lt('Tests'), _lt('Delivered')],
        description: escFormat(_lt('Prioritize Tasks by using the %s icon.' +
            '%s Use the %s button to inform your colleagues that a task is ready for the next stage.' +
            '%s Use the %s to indicate a problem or a need for discussion on a task.' +
            '%s'), star, '<br/>', greenBullet, '<br/>', redBullet, descriptionActivities),
        bullets: [greenBullet, redBullet, star],
    }, {
        name: _lt('Agile Scrum'),
        columns: [_lt('Backlog'), _lt('Sprint Backlog'), _lt('Sprint in Progress'), _lt('Sprint Complete'), _lt('Old Completed Sprint')],
        description: escFormat(_lt('Use %s and %s bullets to indicate the status of a task. %s'), greenBullet, redBullet, descriptionActivities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Digital Marketing'),
        columns: [_lt('Ideas'), _lt('Researching'), _lt('Writing'), _lt('Editing'), _lt('Done')],
        description: escFormat(_lt('Everyone can propose ideas, and the Editor marks the best ones ' +
            'as %s. Attach all documents or links to the task directly, to have all research information centralized. %s'), greenBullet, descriptionActivities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Customer Feedback'),
        columns: [_lt('New'), _lt('In development'), _lt('Done'), _lt('Refused')],
        description: escFormat(_lt('Customers propose feedbacks by email; Odoo creates tasks ' +
            'automatically, and you can communicate on the task directly. Your managers decide which ' +
            'feedback is accepted %s and which feedback is moved to the %s column. %s'), greenBullet, _lt('"Refused"'), descriptionActivities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Getting Things Done (GTD)'),
        columns: [_lt('Inbox'), _lt('Today'), _lt('This Week'), _lt('This Month'), _lt('Long Term')],
        description: escFormat(_lt('Fill your Inbox easily with the email gateway. Periodically review your ' +
            'Inbox and schedule tasks by moving them to other columns. Every day, you review the ' +
            '%s column to move important tasks %s. Every Monday, you review the %s column. %s'), _lt('"This Week"'), _lt('"Today"'), _lt('"This Month"'), descriptionActivities),
    }, {
        name: _lt('Consulting'),
        columns: [_lt('New Projects'), _lt('Resources Allocation'), _lt('In Progress'), _lt('Done')],
        description: escFormat(_lt('Manage the lifecycle of your project using the kanban view. Add newly acquired projects, assign them and use the %s and %s to define if the project is ready for the next step. %s'), greenBullet, redBullet, descriptionActivities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Research Project'),
        columns: [_lt('Brainstorm'), _lt('Research'), _lt('Draft'), _lt('Final Document')],
        description: escFormat(_lt('Handle your idea gathering within Tasks of your new Project and discuss them in the chatter of the tasks. Use the %s and %s to signalize what is the current status of your Idea. %s'), greenBullet, redBullet, descriptionActivities),
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Website Redesign'),
        columns: [_lt('Page Ideas'), _lt('Copywriting'), _lt('Design'), _lt('Live')],
        description: escFormat(_lt('Handle your idea gathering within Tasks of your new Project and discuss them in the chatter of the tasks. Use the %s and %s to signalize what is the current status of your Idea. %s'), greenBullet, redBullet, descriptionActivities),
    }, {
        name: _lt('T-shirt Printing'),
        columns: [_lt('New Orders'), _lt('Logo Design'), _lt('To Print'), _lt('Done')],
        description: escFormat(_lt('Communicate with customers on the task using the email gateway. ' +
            'Attach logo designs to the task, so that information flows from designers to the workers ' +
            'who print the t-shirt. Organize priorities amongst orders using the %s icon. %s'), star, descriptionActivities),
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
        columns: [_lt('Research'), _lt('Script'), _lt('Recording'), _lt('Mixing'), _lt('Published')],
        description: description,
        bullets: [greenBullet, redBullet, star, clock],
    }],
});

```

## File: static\src\js\right_panel\project_right_panel.js

```javascript
/** @odoo-module **/

import { AddMilestone, OpenMilestone } from '@project/js/right_panel/project_utils';
import { formatFloat } from "@web/fields/formatters";
const { useState } = owl.hooks;

export default class ProjectRightPanel extends owl.Component {
    constructor() {
        super(...arguments);
        this.context = this.props.action.context;
        this.domain = this.props.action.domain;
        this.project_id = this.context.active_id;
        this.state = useState({
            data: {
                milestones: {
                    data: []
                },
                user: {},
            }
        });
    }

    formatFloat(value) {
        return formatFloat(value, { digits: [false, 1] });
    }

    async willStart() {
        await super.willStart(...arguments);
        await this._loadQwebContext();
    }

    async willUpdateProps() {
        await super.willUpdateProps(...arguments);
        await this._loadQwebContext();
    }

    async _loadQwebContext() {
        if (!this.project_id){ // If this is called from notif, multiples updates but no specific project
            return {};
        }
        const data = await this.rpc({
            model: 'project.project',
            method: 'get_panel_data',
            args: [this.project_id],
            kwargs: {
                context: this.context
            }
        });
        this.state.data = data;
        return data;
    }

    async onProjectActionClick(event) {
        event.stopPropagation();
        let action = event.currentTarget.dataset.action;
        const additionalContext = JSON.parse(event.currentTarget.dataset.additional_context || "{}");
        if (event.currentTarget.dataset.type === "object") {
            action = await this.rpc({
                // Use the call_button method in order to have an action
                // with the correct view naming, i.e. list view is named
                // 'list' rather than 'tree'.
                route: '/web/dataset/call_button',
                params: {
                    model: 'project.project',
                    method: event.currentTarget.dataset.action,
                    args: [this.project_id],
                    kwargs: {
                        context: this.context
                    }
                }
            });
        }
        this._doAction(action, {
            additional_context: additionalContext
        });
    }

    _doAction(action, options) {
        this.trigger('do-action', {
            action,
            options
        });
    }
}

ProjectRightPanel.template = "project.ProjectRightPanel";
ProjectRightPanel.components = {AddMilestone, OpenMilestone};

```

## File: static\src\js\right_panel\project_right_panel_mixin.js

```javascript
/** @odoo-module **/

import { ComponentWrapper } from 'web.OwlCompatibility';

export const RightPanelRendererMixin = {
    /**
     * Add RightPanel css class to renderer in order to handle panel style.
     *
     * @override
     */
    async start() {
        this.$el.addClass('o_renderer_with_rightpanel');
        await Promise.all([this._render(), this._super()]);
    },
};

export const RightPanelControllerMixin = {
    rightPanelPosition: 'last-child',
    /**
     * Init the rightSidePanel from the config parameters
     *
     * @override
     */
    init: function (parent, model, renderer, params) {
        this._super.apply(this, arguments);
        this.rightSidePanel = params.rightSidePanel;
    },
    /**
     * Add the Rightpanel component through the component wrapper.
     *
     * @override
     */
    start: async function () {
        const promises = [this._super(...arguments)];
        this._rightPanelWrapper = new ComponentWrapper(this, this.rightSidePanel.Component, this.rightSidePanel.props);
        const content = this.el.querySelector(':scope .o_content');
        content.classList.add('o_controller_with_rightpanel');
        promises.push(this._rightPanelWrapper.mount(content, { position: this.rightPanelPosition }));
        await Promise.all(promises);
    },
    /**
     * Make sure the right panel is updated each time the view is reloaded.
     *
     * @override
     */
    async _update() {
        this._rightPanelWrapper.update(this.rightSidePanel.props);
        await this._super.apply(this, arguments);
    },
};

export const RightPanelViewMixin = {
    searchMenuTypes: ['filter', 'favorite'],
    /**
     * Adds the rightSidePanel in the view.
     * This extention is assuming the rightSidePanel is always given in the case this Mixin is used.
     *
     * @override
     */
    _createSearchModel: function (params) {
        const result = this._super.apply(this, arguments);
        const props = {
            action: params.action,
        };
        this.controllerParams.rightSidePanel = {
            Component: this.config.RightSidePanel,
            props: props,
        };
        return result;
    }
};

```

## File: static\src\js\right_panel\project_utils.js

```javascript
/** @odoo-module **/

import { _lt } from 'web.core';
import fieldUtils from 'web.field_utils';
import { ComponentAdapter } from 'web.OwlCompatibility';
import { FormViewDialog } from 'web.view_dialogs';
const { useState, useRef } = owl.hooks;

class MilestoneComponent extends owl.Component {
    constructor() {
        super(...arguments);
        this.contextValue = Object.assign({}, {
            'default_project_id': this.props.context.active_id,
        }, this.props.context);
        this.FormViewDialog = FormViewDialog;
        this.state = useState({
            openDialog: false
        });
        this._dialogRef = useRef('dialog');
        this._isDialogOpen = false;
        this._createContext = this._createContext.bind(this);
        this._onDialogSaved = this._onDialogSaved.bind(this);
    }

    get context() {
        return this.contextValue;
    }

    set context(value) {
        this.contextValue = Object.assign({}, {
            'default_project_id': value.active_id,
        }, value);
    }

    dialogClosed() {
        this._isDialogOpen = false;
        this.state.openDialog = false;
    }

    patched() {
        if (this.state.openDialog && !this._isDialogOpen) {
            this._isDialogOpen = true;
            this._dialogRef.comp.widget.on('closed', this, () => {
                this.dialogClosed();
            });
            this._dialogRef.comp.widget.open();
        }
    }

    _createContext() {
        return Object.assign({}, {
            'default_project_id': this.contextValue.active_id,
        }, this.contextValue);
    }

    async _onDialogSaved() {
        await this.__owl__.parent.willUpdateProps();
    }
}
MilestoneComponent.components = { ComponentAdapter };

export class AddMilestone extends MilestoneComponent {
    get NEW_PROJECT_MILESTONE() {
        return _lt("New Milestone");
    }

    onAddMilestoneClick(event) {
        if (!this._isDialogOpen) {
            event.stopPropagation();
            this.state.openDialog = true;
        }
    }
}
AddMilestone.template = 'project.AddMilestone';

export class OpenMilestone extends MilestoneComponent {

    constructor() {
        super(...arguments);
        this.milestone = useState(this.props.milestone);
        this.state.colorClass = this.milestone.is_deadline_exceeded ? "o_milestone_danger" : "";
        this.state.checkboxIcon = this.milestone.is_reached ? "fa-check-square-o" : "fa-square-o";
    }

    get OPEN_PROJECT_MILESTONE() {
        return _lt("Milestone");
    }

    get deadline() {
        return fieldUtils.format.date(moment(this.milestone.deadline));
    }

    willUpdateProps(nextProps) {
        if (nextProps.milestone) {
            this.milestone = nextProps.milestone;
            this.state.colorClass = this.milestone.is_deadline_exceeded ? "o_milestone_danger" : "";
            this.state.checkboxIcon = this.milestone.is_reached ? "fa-check-square-o" : "fa-square-o";
        }
        if (nextProps.context) {
            this.contextValue = nextProps.context;
        }
    }

    dialogClosed() {
        super.dialogClosed();
        this.write_mutex = false;
    }

    async onDeleteMilestone() {
        await this.rpc({
            model: 'project.milestone',
            method: 'unlink',
            args: [this.milestone.id]
        });
        await this.__owl__.parent.willUpdateProps();
    }

    onOpenMilestone() {
        if (!this._isDialogOpen && !this.write_mutex) {
            this.write_mutex = true;
            this.state.openDialog = true;
        }
    }

    async onMilestoneClick() {
        if (!this.write_mutex) {
            this.write_mutex = true;
            this.milestone = await this.rpc({
                model: 'project.milestone',
                method: 'toggle_is_reached',
                args: [[this.milestone.id], !this.milestone.is_reached],
            });
            this.state.colorClass = this.milestone.is_deadline_exceeded ? "o_milestone_danger" : "";
            this.state.checkboxIcon = this.milestone.is_reached ? "fa-check-square-o" : "fa-square-o";
            this.write_mutex = false;
        }
    }
}
OpenMilestone.template = 'project.OpenMilestone';

```

## File: static\src\js\tours\project.js

```javascript
odoo.define('project.tour', function(require) {
"use strict";

const {_t} = require('web.core');
const {Markup} = require('web.utils');
var tour = require('web_tour.tour');

tour.register('project_tour', {
    sequence: 110,
    url: "/web",
    rainbowManMessage: "Congratulations, you are now a master of project management.",
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="project.menu_main_pm"]',
    content: Markup(_t('Want a better way to <b>manage your projects</b>? <i>It starts here.</i>')),
    position: 'right',
    edition: 'community',
}, {
    trigger: '.o_app[data-menu-xmlid="project.menu_main_pm"]',
    content: Markup(_t('Want a better way to <b>manage your projects</b>? <i>It starts here.</i>')),
    position: 'bottom',
    edition: 'enterprise',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_project_kanban',
    content: Markup(_t('Let\'s create your first <b>project</b>.')),
    position: 'bottom',
    width: 200,
}, {
    trigger: 'input.o_project_name',
    content: Markup(_t('Choose a <b>name</b> for your project. <i>It can be anything you want: the name of a customer,\
     of a product, of a team, of a construction site, etc.</i>')),
    position: 'right',
}, {
    trigger: '.o_open_tasks',
    content: Markup(_t('Let\'s create your first <b>project</b>.')),
    position: 'top',
    run: function (actions) {
        actions.auto('.modal:visible .btn.btn-primary');
    },
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .input-group",
    content: Markup(_t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>.")),
    position: 'right',
    run: function (actions) {
        actions.text("Test", this.$anchor.find("input"));
    },
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    auto: true,
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .input-group",
    extra_trigger: '.o_kanban_group',
    content: Markup(_t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>.")),
    position: 'right',
    run: function (actions) {
        actions.text("Test", this.$anchor.find("input"));
    },
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    auto: true,
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_kanban_group:eq(1)',
    content: Markup(_t("Let's create your first <b>task</b>.")),
    position: 'bottom',
    width: 200,
}, {
    trigger: '.o_kanban_quick_create input.o_field_char[name=name]',
    extra_trigger: '.o_kanban_project_tasks',
    content: Markup(_t('Choose a task <b>name</b> <i>(e.g. Website Design, Purchase Goods...)</i>')),
    position: 'right',
}, {
    trigger: '.o_kanban_quick_create .o_kanban_add',
    extra_trigger: '.o_kanban_project_tasks',
    content: _t("Add your task once it is ready."),
    position: "bottom",
}, {
    trigger: ".o_kanban_record .oe_kanban_content",
    extra_trigger: '.o_kanban_project_tasks',
    content: Markup(_t("<b>Drag &amp; drop</b> the card to change your task from stage.")),
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
    content: Markup(_t("Use the chatter to <b>send emails</b> and communicate efficiently with your customers. \
    Add new people to the followers' list to make them aware of the main changes about this task.")),
    width: 350,
    position: "bottom",
}, {
    trigger: ".o_ChatterTopbar_buttonLogNote",
    extra_trigger: '.o_form_project_tasks',
    content: Markup(_t("<b>Log notes</b> for internal communications <i>(the people following this task won't be notified \
    of the note you are logging unless you specifically tag them)</i>. Use @ <b>mentions</b> to ping a colleague \
    or # <b>mentions</b> to reach an entire team.")),
    width: 350,
    position: "bottom"
}, {
    trigger: ".o_ChatterTopbar_buttonScheduleActivity",
    extra_trigger: '.o_form_project_tasks',
    content: Markup(_t("Use <b>activities</b> to organize your daily work.")),
}, {
    trigger: ".modal-dialog .btn-primary",
    extra_trigger: '.o_form_project_tasks',
    content: "Schedule your activity once it is ready.",
    position: "bottom",
    run: "click",
}, {
    trigger: ".breadcrumb-item:not(.active):last",
    extra_trigger: '.o_form_project_tasks.o_form_readonly',
    content: Markup(_t("Let's go back to the <b>kanban view</b> to have an overview of your next tasks.")),
    position: "right",
    run: 'click',
}]);

});

```

## File: static\src\js\update\project_update_view_kanban.js

```javascript
/** @odoo-module **/

import KanbanController from 'web.KanbanController';
import KanbanRenderer from 'web.KanbanRenderer';
import KanbanView from 'web.KanbanView';
import viewRegistry from 'web.view_registry';
import ProjectRightSidePanel from '@project/js/right_panel/project_right_panel';
import {
    RightPanelControllerMixin,
    RightPanelRendererMixin,
    RightPanelViewMixin,
} from '@project/js/right_panel/project_right_panel_mixin';

const ProjectUpdateKanbanRenderer = KanbanRenderer.extend(RightPanelRendererMixin);

const ProjectUpdateKanbanController = KanbanController.extend(RightPanelControllerMixin);

export const ProjectUpdateKanbanView = KanbanView.extend(RightPanelViewMixin).extend({
    config: Object.assign({}, KanbanView.prototype.config, {
        Controller: ProjectUpdateKanbanController,
        Renderer: ProjectUpdateKanbanRenderer,
        RightSidePanel: ProjectRightSidePanel,
    }),
});

viewRegistry.add('project_update_kanban', ProjectUpdateKanbanView);

```

## File: static\src\js\update\project_update_view_list.js

```javascript
/** @odoo-module **/

import ListController from 'web.ListController';
import ListRenderer from 'web.ListRenderer';
import ListView from 'web.ListView';
import viewRegistry from 'web.view_registry';
import ProjectRightSidePanel from '@project/js/right_panel/project_right_panel';
import {
    RightPanelControllerMixin,
    RightPanelRendererMixin,
    RightPanelViewMixin,
} from '@project/js/right_panel/project_right_panel_mixin';

const ProjectUpdateListRenderer = ListRenderer.extend(RightPanelRendererMixin);

const ProjectUpdateListController = ListController.extend(RightPanelControllerMixin);

export const ProjectUpdateListView = ListView.extend(RightPanelViewMixin).extend({
    config: Object.assign({}, ListView.prototype.config, {
        Controller: ProjectUpdateListController,
        Renderer: ProjectUpdateListRenderer,
        RightSidePanel: ProjectRightSidePanel,
    }),
});

viewRegistry.add('project_update_list', ProjectUpdateListView);

```

## File: static\src\js\widgets\html_with_action_widget.js

```javascript
/** @odoo-module **/

import { device } from 'web.config';
import { bus } from 'web.core';
import fieldRegistry from 'web.field_registry';
import FieldHtml from 'web_editor.field.html';

export const FieldHtmlWithAction = FieldHtml.extend({
    events: Object.assign({}, FieldHtml.prototype.events, {
        'click a[type="object"]': '_onClickObject'
    }),
    init() {
        this._super(...arguments);
        if (!device.isMobile) {
            bus.on("DOM_updated", this, () => {
                const $editable = this.$el.find('.note-editable');
                if ($editable.length) {
                    const minHeight = window.innerHeight - $editable.offset().top - 30;
                    $editable.css('min-height', minHeight + 'px');
                }
            });
        }
    },
    /**
     * Handle the click on a link with object as type
     * @param {*} event
     */
    _onClickObject: async function (event) {
        event.preventDefault();
        const action = await this._rpc({
            model: this.model,
            method: event.target.name,
            args: [this.res_id],
            context: this.record.context,
        });
        if (action) {
            this.do_action(action);
        }
    }
});

fieldRegistry.add('html_with_action', FieldHtmlWithAction);

```

## File: static\src\js\widgets\project_name_with_subtask_count_widget.js

```javascript
/** @odoo-module **/

import fieldRegistry from 'web.field_registry';
import { FieldChar } from 'web.basic_fields';

export const FieldNameWithSubTaskCount = FieldChar.extend({
    /**
     * @override
     */
    init() {
        this._super(...arguments);
        if (this.viewType === 'kanban') {
            // remove click event handler
            const {click, ...events} = this.events;
            this.events = events;
        }
    },

    _render: function () {
        let result = this._super.apply(this, arguments);
        if (this.recordData.allow_subtasks && this.recordData.child_text) {
            this.$el.append($('<span>')
                    .addClass("text-muted ml-2")
                    .text(this.recordData.child_text)
                    .css('font-weight', 'normal'));
        }
        return result;
    }
});

fieldRegistry.add('name_with_subtask_count', FieldNameWithSubTaskCount);

```

## File: static\src\js\widgets\status_with_color_widget.js

```javascript
/** @odoo-module **/

import { qweb } from 'web.core';
import fieldRegistry from 'web.field_registry';
import { FieldSelection } from 'web.relational_fields';

/**
 * options :
 * `color_field` : The field that must be use to color the bubble. It must be in the view. (from 0 to 11). Default : grey.
 */
export const StatusWithColor = FieldSelection.extend({
    _template: 'project.statusWithColor',

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.color = this.recordData[this.nodeOptions.color_field];
        if (this.nodeOptions.no_quick_edit) {
            this._canQuickEdit = false;
        }
    },

    /**
     * @override
     */
    _renderReadonly() {
        this._super.apply(this, arguments);
        if (this.value) {
            this.$el.addClass('o_status_with_color');
            this.$el.prepend(qweb.render(this._template, {
                color: this.color,
            }));
        }
    },
});

fieldRegistry.add('status_with_color', StatusWithColor);

```

## File: static\src\project_control_panel\project_control_panel.js

```javascript
/** @odoo-module **/

import { ControlPanel } from "@web/search/control_panel/control_panel";
import { useService } from "@web/core/utils/hooks";

export class ProjectControlPanel extends ControlPanel {
    constructor() {
        super(...arguments);
        this.orm = useService("orm");
        this.user = useService("user");
        const { active_id, show_project_update } = this.env.searchModel.globalContext;
        this.showProjectUpdate = this.env.config.viewType === "form" || show_project_update;
        this.projectId = this.showProjectUpdate ? active_id : false;
    }

    async willStart() {
        const proms = [super.willStart(...arguments)];
        if (this.showProjectUpdate) {
            proms.push(this.loadData());
        }
        await Promise.all(proms);
    }

    async willUpdateProps() {
        const proms = [super.willUpdateProps(...arguments)];
        if (this.showProjectUpdate) {
            proms.push(this.loadData());
        }
        await Promise.all(proms);
    }

    async loadData() {
        const [data, isProjectUser] = await Promise.all([
            this.orm.call("project.project", "get_last_update_or_default", [this.projectId]),
            this.user.hasGroup("project.group_project_user"),
        ]);
        this.data = data;
        this.isProjectUser = isProjectUser;
    }

    async onStatusClick(ev) {
        ev.preventDefault();
        this.actionService.doAction("project.project_update_all_action", {
            additionalContext: {
                default_project_id: this.projectId,
                active_id: this.projectId,
            },
        });
    }
}

ProjectControlPanel.template = "project.ProjectControlPanel";

```

## File: static\src\project_control_panel\project_control_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectControlPanelContentBadge" owl="1">
        <t t-tag="isProjectUser ? 'button' : 'span'" class="badge border d-flex p-2 ml-2 bg-white">
            <span t-attf-class="o_status_bubble o_color_bubble_{{data.color}}"/>
            <span class="font-weight-normal ml-1" t-esc="data.status"/>
        </t>
    </t>

    <t t-name="project.ProjectControlPanelContent" owl="1">
        <t t-if="showProjectUpdate">
            <li t-if="isProjectUser" class="o_project_updates_breadcrumb pl-3" t-on-click="onStatusClick">
                <t t-call="project.ProjectControlPanelContentBadge"></t>
            </li>
            <li t-else="" class="o_project_updates_breadcrumb pl-3">
                <t t-call="project.ProjectControlPanelContentBadge"></t>
            </li>
        </t>
    </t>

    <t t-name="project.Breadcrumbs" t-inherit="web.Breadcrumbs" t-inherit-mode="primary" owl="1">
        <xpath expr="//ol" position="inside">
            <t t-call="project.ProjectControlPanelContent"/>
        </xpath>
    </t>

    <t t-name="project.ProjectControlPanel" t-inherit="web.ControlPanel" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-call='web.Breadcrumbs']" position="replace">
            <t t-call="project.Breadcrumbs"/>
        </xpath>
    </t>

</templates>

```

## File: static\src\project_sharing\main.js

```javascript
/** @odoo-module **/
import { startWebClient } from '@web/start';
import { ProjectSharingWebClient } from './project_sharing';
import { prepareFavoriteMenuRegister } from './components/favorite_menu_registry';

prepareFavoriteMenuRegister();
startWebClient(ProjectSharingWebClient);

```

## File: static\src\project_sharing\project_sharing.js

```javascript
/** @odoo-module **/

import { useBus, useEffect, useService } from '@web/core/utils/hooks';
import { ActionContainer } from '@web/webclient/actions/action_container';
import { MainComponentsContainer } from "@web/core/main_components_container";
import { useOwnDebugContext } from "@web/core/debug/debug_context";
import { ErrorHandler, NotUpdatable } from "@web/core/utils/components";
import { session } from '@web/session';

const { Component } = owl;

export class ProjectSharingWebClient extends Component {
    setup() {
        window.parent.document.body.style.margin = "0"; // remove the margin in the parent body
        this.actionService = useService('action');
        this.user = useService("user");
        useService("legacy_service_provider");
        useOwnDebugContext({ categories: ["default"] });
        useBus(this.env.bus, "ACTION_MANAGER:UI-UPDATED", (mode) => {
            if (mode !== "new") {
                this.el.classList.toggle("o_fullscreen", mode === "fullscreen");
            }
        });
        useEffect(
            () => {
                this._showView();
            },
            () => []
        );
    }

    mounted() { }

    handleComponentError(error, C) {
        // remove the faulty component
        this.Components.splice(this.Components.indexOf(C), 1);
        /**
         * we rethrow the error to notify the user something bad happened.
         * We do it after a tick to make sure owl can properly finish its
         * rendering
         */
        Promise.resolve().then(() => {
            throw error;
        });
    }

    async _showView() {
        const { action_name, project_id } = session;
        await this.actionService.doAction(
            action_name,
            {
                clearBreadcrumbs: true,
                additionalContext: {
                    active_id: project_id,
                }
            }
        );
    }
}

ProjectSharingWebClient.components = { ActionContainer, ErrorHandler, NotUpdatable, MainComponentsContainer };
ProjectSharingWebClient.template = 'project.ProjectSharingWebClient';

```

## File: static\src\project_sharing\project_sharing.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectSharingWebClient" owl="1">
        <body class="o_project_sharing">
            <NotUpdatable>
                <ActionContainer />
            </NotUpdatable>
            <div class="o_dialog_container"/>
            <MainComponentsContainer/>
        </body>
    </t>

</templates>

```

## File: static\src\project_sharing\components\chatter.js

```javascript
/** @odoo-module **/

import { PortalChatter } from 'portal.chatter';
import Composer from './composer';

export default PortalChatter.extend({
    messageFetch(domain) {
        if (!this.options.res_id) {
            return Promise.resolve();
        }
        return this._super.apply(this, arguments);
    },
    _chatterInit() {
        if (!this.options.res_id) {
            this.result = { messages: [] };
            return Promise.resolve();
        }
        return this._super.apply(this, arguments);
    },
    _messageFetchPrepareParams() {
        const data = this._super.apply(this, arguments);
        if (this.options.project_sharing_id) {
            data.project_sharing_id = this.options.project_sharing_id;
        }
        return data;
    },
    _createComposerWidget() {
        return new Composer(this, this.options);
    },
    update(props) {
        this._setOptions(props);
        this._reloadChatterContent();
    },
});

```

## File: static\src\project_sharing\components\chatter.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-extend="portal.chatter_messages">
        <t t-jquery="t[t-as='message']" t-operation="attributes">
            <attribute name="t-foreach">!!widget.options.res_id &amp;&amp; widget.get('messages') || []</attribute>
        </t>
    </t>
</templates>

```

## File: static\src\project_sharing\components\composer.js

```javascript
/** @odoo-module **/

import { PortalComposer } from 'portal.composer';

export default PortalComposer.extend({
    _prepareAttachmentData() {
        const data = this._super.apply(this, arguments);
        const newData = {};
        if (this.options.display_composer && typeof this.options.display_composer == 'string') {
            // then we should have the access_token of the task
            newData.access_token = this.options.display_composer;
        } else {
            newData.project_sharing_id = this.options.project_sharing_id;
        }
        return Object.assign(data, newData);
    },
});

```

## File: static\src\project_sharing\components\favorite_menu_registry.js

```javascript
/** @odoo-module **/

import FavoriteMenuLegacy from 'web.FavoriteMenu';
import CustomFavoriteItemLegacy from 'web.CustomFavoriteItem';
import { registry } from "@web/core/registry";


/**
 * Remove all components contained in the favorite menu registry except the CustomFavoriteItem
 * component for only the project sharing feature.
 */
export function prepareFavoriteMenuRegister() {
    let customFavoriteItemKey = 'favorite-generator-menu';
    const keys = FavoriteMenuLegacy.registry.keys().filter(key => key !== customFavoriteItemKey);
    FavoriteMenuLegacy.registry = Object.assign(FavoriteMenuLegacy.registry, {
        map: {},
        _scoreMapping: {},
        _sortedKeys: null,
    });
    FavoriteMenuLegacy.registry.add(customFavoriteItemKey, CustomFavoriteItemLegacy, 0);
    // notify the listeners, we keep only one key in this registry.
    for (const key of keys) {
        for (const callback of FavoriteMenuLegacy.registry.listeners) {
            callback(key, undefined);
        }
    }

    customFavoriteItemKey = 'custom-favorite-item';
    const favoriteMenuRegistry = registry.category("favoriteMenu");
    for (const [key, _] of favoriteMenuRegistry.getEntries()) {
        if (key !== customFavoriteItemKey) {
            favoriteMenuRegistry.remove(key);
        }
    }
}

```

## File: static\src\project_sharing\search\favorite_menu\custom_favorite_item.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">

    <t t-inherit="web.CustomFavoriteItem" t-inherit-mode="extension">
        <xpath expr="//CheckBox[@value='state.isShared']" position="replace"/>
    </t>

</templates>

```

## File: static\src\project_sharing\views\registry.js

```javascript
/** @odoo-module **/

import ViewRegistry from 'web.view_registry';

import FormView from './form/view';
import ListView from './list/view';

ViewRegistry
    .add('form', FormView)
    .add('list', ListView);

```

## File: static\src\project_sharing\views\form\controller.js

```javascript
/** @odoo-module **/

import FormController from 'web.FormController';
import BasicController from 'web.BasicController';

export default FormController.extend({
    _getActionMenuItems: BasicController.prototype._getActionMenuItems,
});

```

## File: static\src\project_sharing\views\form\renderer.js

```javascript
/** @odoo-module **/

import FormRenderer from 'web.FormRenderer';
import Chatter from '@project/project_sharing/components/chatter';
import { session } from '@web/session';

export default FormRenderer.extend({
    _makeChatterContainerComponent() {
        const props = this._makeChatterContainerProps();
        this._chatterContainerComponent = new Chatter(this, props);
    },
    _makeChatterContainerProps() {
        // FIXME: perhaps check if we have a parent in the window ?
        // Call the parent window to get the url displayed in the browser since we are in an iframe
        const query = new URLSearchParams(window.parent.location.search);
        return {
            token: query.get('access_token') || '',
            res_model: 'project.task',
            pid: '',
            hash: '',
            res_id: this.state.res_id,
            pager_step: 10,
            allow_composer: !!this.state.res_id,
            two_columns: false,
            project_sharing_id: session.project_id,
        };
    },
    _makeChatterContainerTarget() {
        const $el = $('<div class="o_FormRenderer_chatterContainer"/>');
        this._chatterContainerTarget = $el[0];
        return $el;
    },
    _mountChatterContainerComponent() {
        this._chatterContainerComponent.appendTo(this._chatterContainerTarget);
    },
    _renderNode(node) {
        if (node.tag === 'div' && node.attrs.class === 'oe_project_sharing_chatter') {
            let isVisible = true;
            if (node.attrs.modifiers && node.attrs.modifiers.invisible) {
                const record = this._getRecord(this.state.id);
                if (record) {
                    isVisible = !record.evalModifiers(node.attrs.modifiers).invisible;
                }
            }
            if (isVisible) {
                if (this._isFromFormViewDialog) {
                    return $('<div/>');
                }
                return this._makeChatterContainerTarget();
            }
        }
        return this._super(...arguments);
    },
});

```

## File: static\src\project_sharing\views\form\view.js

```javascript
/** @odoo-module **/

import FormView from 'web.FormView';
import Controller from './controller';
import Renderer from './renderer';

export default FormView.extend({
    config: _.extend({}, FormView.prototype.config, {
        Controller,
        Renderer,
    }),
});

```

## File: static\src\project_sharing\views\list\controller.js

```javascript
/** @odoo-module **/

import ListController from 'web.ListController';
import BasicController from 'web.BasicController';

export default ListController.extend({
    _getActionMenuItems: BasicController.prototype._getActionMenuItems,
});

```

## File: static\src\project_sharing\views\list\view.js

```javascript
/** @odoo-module **/

import ListView from 'web.ListView';
import Controller from './controller';

export default ListView.extend({
    config: Object.assign({}, ListView.prototype.config, {
        Controller,
    }),

    init: function (viewInfo, params) {
        return this._super(viewInfo, {...params, hasSelectors: false});
    },
    getRenderer (parent, state) {
        const columnInvisibleFields = {};
        for (const child of this.arch.children) {
            if (child.attrs && child.attrs.modifiers && child.attrs.modifiers.column_invisible) {
                columnInvisibleFields[child.attrs.name] = child.attrs.modifiers.column_invisible;
            }
        }
        this.rendererParams.columnInvisibleFields = Object.entries(columnInvisibleFields).reduce((acc, entry) => {
            const [fieldName, domains] = entry;
            if (domains instanceof Array && state.data.length) {
                const record = state.data[0];
                const data = Object.entries(record.data).reduce((acc, entry) => {
                    const [fieldName, value] = entry;
                    const field = record.fields[fieldName];
                    if (field.type === 'one2many' || field.type === 'many2many') {
                        if (value instanceof Object && value.type === 'list') {
                            acc[fieldName] = value.id;
                        }
                    }
                    return acc;
                }, record.data);
                if (Object.keys(data).length) {
                    // We assume the field in this domain is a related to the project shared.
                    acc[fieldName] = this.model._evalModifiers({ ...record, data }, { column_invisible: domains }).column_invisible;
                }
            } else {
                acc[fieldName] = domains;
            }
            return acc;
        }, {});
        return this._super(parent, state);
    },
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

    <span t-name="project.statusWithColor" t-att-class="'o_status_bubble mr-2 o_color_bubble_' + color"></span>

    <div t-name="project.ControlPanel" t-inherit="web.Legacy.ControlPanel" t-inherit-mode="extension" owl="1">
        <xpath expr="//ol[hasclass('breadcrumb')]" position="inside">
            <t t-call="project.ProjectControlPanelContent">
                <t t-set="showProjectUpdate" t-value="show_project_update"/>
                <t t-set="isProjectUser" t-value="is_project_user"/>
            </t>
        </xpath>
    </div>

    <t t-name="project.ProjectRightPanel" owl="1">
        <div t-if="project_id" class="o_rightpanel">
            <t t-call="project.StatButtonsSection"/>
            <t t-call="project.MilestoneSection"/>
        </div>
        <!-- If this is called from notif, multiples updates but no specific project -->
        <div t-else=""/>
    </t>

    <t t-name="project.StatButtonsSection" owl="1">
        <div class="o_rightpanel_section">
            <div class="o_form_view">
                <div class="oe_button_box o_full">
                    <t t-foreach="state.data.buttons" t-as="button" t-key="button.order">
                        <button class="btn oe_stat_button" t-if="button.show"
                            t-on-click="onProjectActionClick" 
                            t-att-data-action="button.action"
                            t-att-data-additional_context="button.additional_context"
                            t-att-data-type="button.action_type">
                            <i t-att-class="'fa fa-fw o_button_icon fa-' + button.icon"/>
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <t t-esc="button.number"/>
                                </span>
                                <span class="o_stat_text">
                                    <t t-esc="button.text"/>
                                </span>
                            </div>
                        </button>
                    </t>
                </div>
            </div>
        </div>
    </t>

    <t t-name="project.MilestoneSection" owl="1">
        <t t-set="milestones" t-value="state.data.milestones"/>
        <div class="o_rightpanel_section" name="milestone">
            <div class="o_rightpanel_header">
                <div class="o_rightpanel_title">
                    Milestones
                </div>
                <span class="o_rightpanel_button">
                    <AddMilestone context="context"/>
                </span>
            </div>
            <div class="o_rightpanel_data">
                <div t-foreach="milestones.data" t-as="milestone" t-key="milestone.id" class="o_rightpanel_data_row">
                    <OpenMilestone context="context" milestone="milestone" t-key="milestone.id"/>
                </div>
                <span t-if="milestones.data.length === 0" class="text-muted font-italic">
                    Track major progress points that must be reached to achieve success.
                </span>
            </div>
        </div>
    </t>

    <t t-name="project.AddMilestone" owl="1">
        <div class="o_add_milestone">
            <a t-on-click="onAddMilestoneClick">Add Milestone</a>
            <span t-if="state.openDialog" class="o_rightpanel_hidden">
                <ComponentAdapter
                    Component="FormViewDialog"
                    params="{
                        context: context,
                        _createContext: _createContext,
                        on_saved: _onDialogSaved,
                        res_id: false,
                        res_model: 'project.milestone',
                        title: NEW_PROJECT_MILESTONE,
                        multi_select: true,
                    }"
                    t-ref="dialog"
                />
            </span>
        </div>
    </t>

    <t t-name="project.OpenMilestone" owl="1">
        <div class="o_open_milestone row" t-att-class="state.colorClass">
            <span class="o_milestone_checkbox col-1" t-on-click="onMilestoneClick">
                <i class="fa" t-att-class="state.checkboxIcon"></i>
            </span>
            <div class="o_milestone_detail col-10 p-0" t-on-click="onOpenMilestone">
                <span class="text-truncate col-8 px-0" t-att-title="milestone.name" style="flex: 10%">
                    <t t-esc="milestone.name"/>
                </span>
                <span class="o_rightpanel_center_col col-4">
                    <t t-esc="deadline"/>
                </span>
            </div>
            <span class="o_rightpanel_right_col col-1 px-0">
                <a class="o_delete_icon" t-on-click="onDeleteMilestone" title="Delete Milestone"><i class="fa fa-trash"></i></a>
            </span>
            <span t-if="state.openDialog" class="o_rightpanel_hidden">
                <ComponentAdapter
                    Component="FormViewDialog"
                    params="{
                        context: context,
                        on_saved: _onDialogSaved,
                        res_id: milestone.id,
                        res_model: 'project.milestone',
                        title: OPEN_PROJECT_MILESTONE,
                        disable_multiple_selection: true,
                        deletable: true
                    }"
                    t-ref="dialog"
                />
            </span>
        </div>
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
        <field eval="40" name="priority"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(analytic.account_analytic_line_action)d']" position="before">
                <button class="oe_stat_button" type="object" name="action_view_projects"
                    icon="fa-puzzle-piece" attrs="{'invisible': [('project_count', '=', 0)]}">
                    <field string="Projects" name="project_count" widget="statinfo"/>
                </button>
            </xpath>
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
        <field name="domain">['|', ('res_model', '=', False), ('res_model', '=', 'project.task')]</field>
        <field name="context">{'default_res_model': 'project.task'}</field>
    </record>
    <menuitem id="project_menu_config_activity_type"
        action="mail_activity_type_action_config_project_types"
        parent="menu_project_config"/>
</odoo>
```

## File: views\project_collaborator_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_collaborator_view_form" model="ir.ui.view">
        <field name="name">project.collaborator.view.form</field>
        <field name="model">project.collaborator</field>
        <field name="arch" type="xml">
            <form string="Project Collaborator" create="0" edit="0">
                <sheet>
                    <group>
                        <field name="project_id" options="{'no_create': True}" />
                        <field name="partner_id" options="{'no_create': True}" />
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="project_sharing_access_view_tree" model="ir.ui.view">
        <field name="name">project.collaborator.view.tree</field>
        <field name="model">project.collaborator</field>
        <field name="arch" type="xml">
            <tree string="Project Collaborators" create="0">
                <field name="project_id" options="{'no_create': True}"/>
                <field name="partner_id" options="{'no_create': True}"/>
            </tree>
        </field>
    </record>

    <record id="project_collaborator_view_search" model="ir.ui.view">
        <field name="name">project.collaborator.view.search</field>
        <field name="model">project.collaborator</field>
        <field name="arch" type="xml">
            <search>
                <field name="partner_id" />
                <field name="project_id" />
                <group expand="0" string="Group By">
                    <filter name="project" string="Project" context="{'group_by': 'project_id'}" />
                    <filter name="collaborator" string="Collaborator" context="{'group_by': 'partner_id'}" />
                </group>
            </search>
        </field>
    </record>

    <record id="project_collaborator_action" model="ir.actions.act_window">
        <field name="name">Project Collaborators</field>
        <field name="res_model">project.collaborator</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('project_id', '=', active_id)]</field>
        <field name="search_view_id" ref="project_collaborator_view_search"/>
    </record>

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
            <li t-if="page_name == 'project_task' and project" class="breadcrumb-item active">
                <a t-if="project" t-attf-href="/my/project/{{ project.id }}?{{ keep_query() }}"><t t-esc="project.name"/></a>
            </li>
            <li t-elif="project" t-attf-class="breadcrumb-item #{'active ' if not project else ''} text-truncate col-8 col-lg-10">
                <t t-esc="project.name"/>
            </li>
            <li t-if="page_name == 'task' or (task and not project)" t-attf-class="breadcrumb-item #{'active ' if not task else ''}">
                <a t-if="task" t-attf-href="/my/tasks?{{ keep_query() }}">Tasks</a>
                <t t-else="">Tasks</t>
            </li>
            <li t-if="task" class="breadcrumb-item active text-truncate">
                <span t-field="task.name"/>
            </li>
        </xpath>
    </template>

    <template id="portal_my_tasks_priority_widget_template" name="Priority Widget Template">
        <span t-attf-class="o_priority_star fa fa-star#{'' if task.priority == '1' else '-o'}"/>
    </template>

    <template id="portal_my_tasks_state_widget_template" name="Status Widget Template">
        <span t-att-title="task.kanban_state_label" t-attf-class="o_status rounded-circle #{'bg-success' if task.kanban_state == 'done' else 'bg-danger' if task.kanban_state == 'blocked' else ''}"/>
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
                            <t t-out="project.task_count_with_subtasks" />
                            <t t-out="project.label_tasks" />
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
                    <t t-set="backend_url" t-value="'/web#model=project.project&amp;id=%s&amp;view_type=kanban' % (project.id)"/>
                </t>
            </t>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Tasks</t>
            </t>
            <t t-if="not grouped_tasks">
                <div class="alert alert-warning mt8" role="alert">
                    There are no tasks.
                </div>
            </t>

            <t t-call="project.portal_tasks_list"/>
        </t>
    </template>

    <template id="portal_tasks_list" name="Tasks List">
        <t t-if="grouped_tasks">
            <t t-call="portal.portal_table">
                <t t-foreach="grouped_tasks" t-as="tasks">
                    <thead>
                        <tr t-attf-class="{{'thead-light' if not groupby == 'none' else ''}}">
                            <th class="text-left">Ref</th>
                            <th t-if="groupby != 'priority'">Priority</th>
                            <th t-if="groupby == 'none'">Name</th>
                            <th t-if="groupby == 'project'">
                                <em class="font-weight-normal text-muted"><span t-field="tasks[0].sudo().project_id.label_tasks"/> for project:</em>
                                <span t-field="tasks[0].sudo().project_id.name"/></th>
                            <th t-if="groupby == 'stage'">
                                <em class="font-weight-normal text-muted"><span t-field="tasks[0].sudo().project_id.label_tasks"/> in stage:</em>
                                <span class="text-truncate" t-field="tasks[0].sudo().stage_id.name"/></th>
                            <th t-if="groupby == 'priority'">
                                <em class="font-weight-normal text-muted"><span t-field="tasks[0].sudo().project_id.label_tasks"/> in priority:</em>
                                <span class="text-truncate" t-field="tasks[0].sudo().priority"/></th>
                            <th t-if="groupby == 'status'">
                                <em class="font-weight-normal text-muted"><span t-field="tasks[0].sudo().project_id.label_tasks"/> in status:</em>
                                <span class="text-truncate" t-field="tasks[0].sudo().kanban_state"/></th>
                            <th t-if="groupby == 'customer'">
                                <em class="font-weight-normal text-muted" t-if="tasks[0].sudo().partner_id"><span t-field="tasks[0].sudo().project_id.label_tasks"/> for customer:</em>
                                <span class="text-truncate" t-field="tasks[0].sudo().partner_id.name"/></th>
                            <th name="project_portal_assignees">Assignees</th>
                            <th t-if="groupby != 'status'">Status</th>
                            <th t-if="groupby != 'project'">Project</th>
                            <th t-if="groupby != 'stage'">Stage</th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="tasks" t-as="task">
                            <tr>
                                <td class="text-left">
                                    #<span t-esc="task.id"/>
                                </td>
                                <td t-if="groupby != 'priority'">
                                    <t t-call="project.portal_my_tasks_priority_widget_template"/>
                                </td>
                                <td>
                                    <a t-attf-href="/my/#{task_url}/#{task.id}?{{ keep_query() }}"><span t-field="task.name"/></a>
                                </td>
                                <td name="project_portal_assignees">
                                    <t t-set="assignees" t-value="task.sudo().user_ids"/>
                                    <span t-if="assignees" t-out="'%s%s' % (assignees[:1].name, ' + %s others' % len(assignees[1:]) if len(assignees.user_ids) > 1 else '')" t-att-title="'\n'.join(assignees.mapped('name'))"/>
                                </td>
                                <td t-if="groupby != 'status'">
                                    <t t-call="project.portal_my_tasks_state_widget_template">
                                        <t t-set="path" t-value="'tasks'"/>
                                    </t>
                                </td>
                                <td t-if="groupby != 'project'">
                                    <span class="badge badge-pill badge-info mw-100 text-truncate" title="Current project of the task" t-esc="task.project_id.name" />
                                </td>
                                <td t-if="groupby != 'stage'">
                                    <span class="badge badge-pill badge-info" title="Current stage of the task" t-esc="task.stage_id.name" />
                                </td>
                            </tr>
                        </t>
                    </tbody>
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
            <t t-call="project.portal_tasks_list"/>
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
                                <div class="col-9">
                                    <t t-call="project.portal_my_tasks_priority_widget_template"/>
                                    <span t-field="task.name" class="text-truncate"/>
                                    <small class="text-muted d-none d-md-inline"> (#<span t-field="task.id"/>)</small>
                                </div>
                                <div class="col-3 text-right">
                                    <small class="text-right">Stage:</small>
                                    <span t-field="task.stage_id.name" class=" badge badge-pill badge-info" title="Current stage of this task"/>
                                </div>
                            </h5>
                        </div>
                    </div>
                </t>
                <t t-set="card_body">
                    <div class="float-right">
                        <t t-call="project.portal_my_tasks_state_widget_template">
                            <t t-set="path" t-value="'task'"/>
                        </t>
                    </div>
                    <div class="row mt-3" t-if="task.user_ids or task.partner_id">
                        <div class="col-12 col-md-6 pb-2" t-if="task.user_ids">
                            <strong>Assignees</strong>
                            <div class="row">
                                <t t-foreach="task.user_ids" t-as="user">
                                    <div class="col d-flex align-items-center flex-grow-0 pr-3">
                                        <img class="rounded-circle mt-1 o_portal_contact_img" t-att-src="image_data_uri(user.avatar_1024)" alt="Contact"/>
                                    </div>
                                    <div class="col pl-md-0">
                                        <div t-esc="user" t-options='{"widget": "contact", "fields": ["name"]}'/>
                                        <a t-attf-href="mailto:{{user.email}}" t-if="user.email"><div t-esc="user" t-options='{"widget": "contact", "fields": ["email"]}'/></a>
                                        <a t-attf-href="tel:{{user.phone}}" t-if="user.phone"><div t-esc="user" t-options='{"widget": "contact", "fields": ["phone"]}'/></a>
                                    </div>
                                </t>
                            </div>
                        </div>
                        <div class="col-12 col-md-6 pb-2" t-if="task.partner_id">
                            <strong>Customer</strong>
                            <div class="row">
                                <div class="col d-flex align-items-center flex-grow-0 pr-3">
                                    <img class="rounded-circle mt-1 o_portal_contact_img" t-att-src="image_data_uri(task.partner_id.avatar_1024)" alt="Contact"/>
                                </div>
                                <div class="col pl-md-0">
                                    <div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["name"]}'/>
                                    <a t-attf-href="mailto:{{task.partner_id.email}}" t-if="task.partner_id.email"><div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["email"]}'/></a>
                                    <a t-attf-href="tel:{{task.partner_id.phone}}" t-if="task.partner_id.phone"><div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["phone"]}'/></a>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="row mb-4">
                        <div class="col-12 col-md-6">
                            <div t-if="project_accessible"><strong>Project:</strong> <a t-attf-href="/my/project/#{task.project_id.id}" t-field="task.project_id"/></div>
                            <div t-else=""><strong>Project:</strong> <a t-field="task.project_id"/></div>
                            <div t-if="task.date_deadline"><strong>Deadline:</strong> <span t-field="task.date_deadline" t-options='{"widget": "date"}'/></div>
                            <div name="portal_my_task_planned_hours">
                                <t t-call="project.portal_my_task_planned_hours_template"/>
                            </div>
                        </div>
                        <div class="col-12 col-md-6" name="portal_my_task_second_column"></div>
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

    <template id="portal_my_task_planned_hours_template">
        <strong>Planned Hours:</strong> <span t-esc="task.planned_hours" t-options='{"widget": "float_time"}'/>
    </template>
</odoo>

```

## File: views\project_project_stage_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_project_stage_view_tree" model="ir.ui.view">
        <field name="name">project.project.stage.view.tree</field>
        <field name="model">project.project.stage</field>
        <field name="arch" type="xml">
            <tree editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="mail_template_id" optional="hide" context="{'default_model': 'project.project'}"/>
                <field name="fold" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="project_project_stage_view_form_quick_create" model="ir.ui.view">
        <field name="name">project.project.stage.view.form.quick.create</field>
        <field name="model">project.project.stage</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="name"/>
                    <field name="mail_template_id"/>
                    <field name="fold"/>
                </group>
            </form>
        </field>
    </record>

    <record id="project_project_stage_view_form" model="ir.ui.view">
        <field name="name">project.project.stage.view.form</field>
        <field name="model">project.project.stage</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <h1><field name="name" placeholder="New"/></h1>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="mail_template_id" context="{'default_model': 'project.project'}"/>
                            <field name="sequence" groups="base.group_no_one"/>
                        </group>
                        <group>
                            <field name="fold"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="project_project_stage_view_kanban" model="ir.ui.view">
        <field name="name">project.project.stage.view.kanban</field>
        <field name="model">project.project.stage</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1" quick_create_view="project.project_project_stage_view_form_quick_create">
                <field name="name"/>
                <field name="mail_template_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="o_kanban_record oe_kanban_global_click">
                            <div class="row">
                                <div class="col-12">
                                    <strong><field name="name"/></strong>
                                    <br/>
                                    <span class="text-muted"><field name="mail_template_id"/></span>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="project_project_stage_view_search" model="ir.ui.view">
        <field name="name">project.project.stage.view.search</field>
        <field name="model">project.project.stage</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="mail_template_id"/>
                <filter string="Archived" name="archived" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="project_project_stage_configure" model="ir.actions.act_window">
        <field name="name">Project Stages</field>
        <field name="res_model">project.project.stage</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
              No stages found. Let's create one!
            </p><p>
              Track the progress of your projects from their creation to their closing.
            </p>
        </field>
    </record>

    <record id="project_project_stage_configure_view_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="project_project_stage_view_tree"/>
        <field name="act_window_id" ref="project_project_stage_configure"/>
    </record>

    <record id="project_project_stage_configure_view_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="2"/>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="project_project_stage_view_kanban"/>
        <field name="act_window_id" ref="project_project_stage_configure"/>
    </record>

    <record id="project_project_stage_configure_view_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="3"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="project_project_stage_view_form"/>
        <field name="act_window_id" ref="project_project_stage_configure"/>
    </record>
</odoo>

```

## File: views\project_sharing_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="project_sharing_portal" name="Project Sharing View in Portal">
        <t t-call="portal.frontend_layout">
            <t t-set="no_footer" t-value="true"/>
            <t t-call="project.project_sharing"/>
        </t>
    </template>

    <template id="project_sharing" name="Project Sharing View">
        <!--    We need to forward the request lang to ensure that the lang set on the portal match the lang delivered -->
        <iframe width="100%" height="100%" frameborder="0" t-attf-src="/{{ request.context['lang'] }}/my/project/{{ str(project_id) }}/project_sharing"/>
    </template>

    <template id="project_sharing_embed" name="Project Sharing View Embed">
        <t t-call="web.layout">
            <t t-set="head_project_sharing">
                <script type="text/javascript">
                    odoo.__session_info__ = <t t-out="json.dumps(session_info)"/>;
                    // Prevent the menu_service to load anything. In an ideal world, Project Sharing assets would only contain
                    // what is genuinely necessary, and not the whole backend.
                    odoo.loadMenusPromise = Promise.resolve();
                    odoo.loadTemplatesPromise = fetch(`/web/webclient/qweb/${odoo.__session_info__.cache_hashes.qweb}?bundle=project.assets_qweb`).then(doc => doc.text());
                </script>
                <base target="_parent"/>
                <t t-call-assets="web.assets_common" t-js="false"/>
                <t t-call-assets="project.webclient" t-js="false"/>
                <t t-call-assets="web.assets_common" t-css="false"/>
                <t t-call-assets="project.webclient" t-css="false"/>
                <t t-call="web.conditional_assets_tests"/>
            </t>
            <t t-set="head" t-value="head_project_sharing + (head or '')"/>
            <t t-set="body_classname" t-value="'o_web_client'"/>
        </t>
    </template>

</odoo>

```

## File: views\project_sharing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_sharing_quick_create_task_form" model="ir.ui.view">
        <field name="name">project.task.form.quick_create</field>
        <field name="model">project.task</field>
        <field name="priority">999</field>
        <field name="groups_id" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="name" string="Task Title" placeholder="e.g. New Design"/>
                </group>
            </form>
        </field>
    </record>

    <record id="project_sharing_project_task_view_kanban" model="ir.ui.view">
        <field name="name">project.sharing.project.task.view.kanban</field>
        <field name="model">project.task</field>
        <field name="priority">999</field>
        <field name="groups_id" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
        <field name="arch" type="xml">
            <kanban
                class="o_kanban_small_column o_kanban_project_tasks"
                default_group_by="stage_id"
                on_create="quick_create"
                quick_create_view="project.project_sharing_quick_create_task_form"
                archivable="0"
                import="0"
            >
                <field name="color"/>
                <field name="priority"/>
                <field name="stage_id" options='{"group_by_tooltip": {"description": "Description"}}'/>
                <!-- TODO: [XBO] remove me in master -->
                <field name="user_ids" invisible="1"/>
                <field name="portal_user_names"/>
                <field name="partner_id"/>
                <field name="sequence"/>
                <field name="is_closed"/>
                <field name="partner_is_company"/>
                <field name="displayed_image_id"/>
                <field name="active"/>
                <field name="allow_subtasks"/>
                <field name="child_text"/>
                <field name="legend_blocked" invisible="1"/>
                <field name="legend_normal" invisible="1"/>
                <field name="legend_done" invisible="1"/>
                <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger", "normal": "muted"}'/>
                <templates>
                <t t-name="kanban-box">
                    <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click">
                        <div class="oe_kanban_content">
                            <div class="o_kanban_record_top">
                                <div class="o_kanban_record_headings">
                                    <strong class="o_kanban_record_title">
                                        <s t-if="!record.active.raw_value"><field name="name" widget="name_with_subtask_count"/></s>
                                        <t t-else=""><field name="name" widget="name_with_subtask_count"/></t>
                                    </strong>
                                    <span invisible="context.get('default_project_id', False)"><br/><field name="project_id" required="1"/></span>
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
                                <div class="o_dropdown_kanban dropdown" t-if="!selection_mode">
                                    <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                        <span class="fa fa-ellipsis-v"/>
                                    </a>
                                    <div class="dropdown-menu" role="menu">
                                        <!-- [XBO] TODO: remove me in master -->
                                        <a t-if="widget.editable" role="menuitem" type="set_cover" class="dropdown-item d-none" data-field="displayed_image_id">Set Cover Image</a>
                                        <a t-if="widget.editable" role="menuitem" type="edit" class="dropdown-item">Edit</a>
                                        <div role="separator" class="dropdown-divider"></div>
                                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                                    </div>
                                </div>
                            </div>
                            <div class="o_kanban_record_body">
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                <div t-if="record.date_deadline.raw_value" name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}">
                                    <field name="date_deadline" widget="remaining_days"/>
                                </div>
                                <div t-if="record.displayed_image_id.value">
                                    <field name="displayed_image_id" widget="attachment_image"/>
                                </div>
                            </div>
                            <div class="o_kanban_record_bottom" t-if="!selection_mode">
                                <div class="oe_kanban_bottom_left">
                                    <field name="priority" widget="priority"/>
                                </div>
                                <div class="oe_kanban_bottom_right" t-if="!selection_mode">
                                    <span t-if="record.portal_user_names.raw_value.length > 0" class="pr-2" t-att-title="record.portal_user_names.raw_value">
                                        <t t-set="user_count" t-value="record.portal_user_names.raw_value.split(',').length"/>
                                        <t t-out="user_count"/>
                                        <t t-if="user_count > 1"> assignees</t>
                                        <t t-else=""> assignee</t>
                                        <t t-if="1 == 0" t-set="display_nb_assignees" t-value="user_count + ' assignee' + (user_count > 1 ? 's' : '')"/>
                                        <t t-if="1 == 0" t-out="display_nb_assignees"/>
                                    </span>
                                    <field name="kanban_state" widget="state_selection"/>
                                    <!-- TODO: [XBO] remove me in master -->
                                    <field name="user_ids" string="Assignees" widget="many2many_avatar_user" options="{'no_open_chat': True}" invisible="1"/>
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

    <record id="project_sharing_project_task_view_tree" model="ir.ui.view">
        <field name="name">project.sharing.project.task.tree</field>
        <field name="model">project.task</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <tree string="Tasks" sample="1" delete="0" import="0">
                <field name="is_closed" invisible="1" />
                <field name="allow_subtasks" invisible="1" />
                <field name="sequence" invisible="1" readonly="1"/>
                <field name="priority" widget="priority" optional="show" nolabel="1"/>
                <field name="name" widget="name_with_subtask_count"/>
                <field name="child_text" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <field name="partner_id" optional="hide"/>
                <field name="portal_user_names" string="Assignees" optional="show"/>
                <!-- TODO: [XBO] remove me in master -->
                <field name="user_ids" invisible="1" />
                <field name="date_deadline" optional="hide" widget="remaining_days" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                <field name="kanban_state" widget="state_selection" optional="hide"/>
                <field name="legend_blocked" invisible="1"/>
                <field name="legend_normal" invisible="1"/>
                <field name="legend_done" invisible="1"/>
                <field name="stage_id" invisible="context.get('set_visible',False)" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="project_sharing_project_task_view_form" model="ir.ui.view">
        <field name="name">project.sharing.project.task.view.form</field>
        <field name="model">project.task</field>
        <field name="priority">999</field>
        <field name="groups_id" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
        <field name="arch" type="xml">
            <form string="Project Sharing: Task" class="o_form_project_tasks">
                <header>
                    <button name="action_assign_to_me" string="Assign to Me" type="object" class="oe_highlight"
                            attrs="{'invisible' : &quot;[('user_ids', 'in', [uid])]&quot;}" data-hotkey="q" groups="base.group_user"/>
                    <button name="action_unassign_me" string="Unassign Me" type="object" class="oe_highlight"
                            attrs="{'invisible' : &quot;[('user_ids', 'not in', [uid])]&quot;}" data-hotkey="q"/>
                    <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" attrs="{'invisible': [('project_id', '=', False), ('stage_id', '=', False)]}" />
                </header>
                <sheet string="Task">
                    <div class="oe_button_box" name="button_box">
                        <button name="action_open_parent_task" type="object" special="cancel" class="oe_stat_button o_debounce_disabled" icon="fa-tasks" string="Parent Task" attrs="{'invisible': [('parent_id', '=', False)]}"/>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title pr-0">
                        <h1 class="d-flex flex-row justify-content-between">
                            <field name="priority" widget="priority" class="mr-3"/>
                            <field name="name" class="o_task_name text-truncate" placeholder="Task Title..."/>
                            <field name="kanban_state" widget="state_selection" class="ml-auto"/>
                            <field name="legend_blocked" invisible="1"/>
                            <field name="legend_normal" invisible="1"/>
                            <field name="legend_done" invisible="1"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="project_id" invisible="1"/>
                            <field name="display_project_id" string="Project" invisible="1"/>
                            <field name="user_ids" invisible="1" />
                            <field name="portal_user_names"
                                string="Assignees"
                                class="o_task_user_field"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="parent_id" invisible="1" />
                            <field name="company_id" invisible="1" />
                            <field name="is_closed" invisible="1" />
                            <field name="allow_subtasks" invisible="1" />
                            <field name="partner_id" options="{'no_open': True, 'no_create': True, 'no_edit': True}"/>
                            <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True, 'no_edit_color': True}"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" type="html" options="{'collaborative': true}"/>
                        </page>
                        <page name="sub_tasks_page" string="Sub-tasks" attrs="{'invisible': [('allow_subtasks', '=', False)]}">
                            <field name="child_ids" context="{'default_project_id': project_id if not parent_id or not display_project_id else display_project_id, 'default_parent_id': id, 'default_partner_id': partner_id, 'form_view_ref' : 'project.project_sharing_project_task_view_form'}">
                                <tree editable="bottom">
                                    <field name="project_id" invisible="1"/>
                                    <field name="is_closed" invisible="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="name"/>
                                    <field name="display_project_id" string="Project" optional="hide" invisible="1"/>
                                    <field name="company_id" invisible="1"/>
                                    <field name="partner_id" options="{'no_open': True, 'no_create': True, 'no_edit': True}" optional="hide"/>
                                    <!-- TODO: [XBO] remove me in master -->
                                    <field name="user_ids" invisible="1"/>
                                    <field name="portal_user_names" string="Assignees"/>
                                    <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}" optional="show"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="kanban_state" widget="state_selection" optional="hide"/>
                                    <field name="stage_id" optional="show"/>
                                    <button name="action_open_task" type="object" title="View Task" string="View Task" class="btn btn-link pull-right"
                                            context="{'form_view_ref': 'project.project_sharing_project_task_view_form'}"
                                            attrs="{'invisible': &quot;[('display_project_id', '!=', False), ('display_project_id', '!=', active_id)]&quot;}"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter"/>
            </form>
        </field>
    </record>

    <record id="project_sharing_project_task_view_search" model="ir.ui.view">
        <field name="name">project.task.search.form</field>
        <field name="model">project.task</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <search string="Tasks">
                <field name="name" string="Task"/>
                <field name="tag_ids"/>
                <!-- TODO: [XBO] remove me in master -->
                <field name="user_ids" invisible="1"/>
                <field name="portal_user_names" string="Assignees"/>
                <field name="partner_id" operator="child_of"/>
                <field name="stage_id"/>
                <field string="Project" name="display_project_id"/>
                <filter string="Unassigned" name="unassigned" domain="[('user_ids', '=', False)]"/>
                <separator/>
                <filter string="Starred" name="starred" domain="[('priority', 'in', [1, 2])]"/>
                <separator/>
                <filter string="Late Tasks" name="late" domain="[('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d')), ('is_closed', '=', False)]"/>
                <separator/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}"/>
                    <!-- TODO: [XBO] remove me in master -->
                    <filter string="Assignees" name="user" context="{'group_by': 'user_ids'}" invisible="1"/>
                    <!-- [XBO] TODO: remove me in master -->
                    <filter string="Project" name="project" context="{'group_by': 'project_id'}" invisible="1"/>
                    <filter string="Customer" name="customer" context="{'group_by': 'partner_id'}"/>
                    <filter string="Kanban State" name="kanban_state" context="{'group_by': 'kanban_state'}"/>
                    <filter string="Deadline" name="date_deadline" context="{'group_by': 'date_deadline'}"/>
                    <!-- [XBO] TODO: remove me in master -->
                    <filter string="Creation Date" name="group_create_date" context="{'group_by': 'create_date'}" invisible="1"/>
                </group>
            </search>
        </field>
    </record>

    <record id="project_sharing_project_task_action" model="ir.actions.act_window">
        <field name="name">Project Sharing</field>
        <field name="res_model">project.task</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="search_view_id" ref="project.project_sharing_project_task_view_search"/>
        <field name="domain">[('display_project_id', '=', active_id)]</field>
        <field name="context">{
            'default_project_id': active_id,
            'delete': 0,
        }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No tasks found. Let's create one!
            </p><p>
                To get things done, use activities and status on tasks.<br/>
                Chat in real time or by email to collaborate efficiently.
            </p>
        </field>
    </record>

    <record id="project_sharing_kanban_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_kanban"/>
    </record>

    <record id="project_sharing_tree_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_tree"/>
    </record>

    <record id="project_sharing_form_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_form"/>
    </record>

</odoo>

```

## File: views\project_task_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="task_track_depending_tasks">
        <span>Task: <a href="#" data-oe-model="project.task" t-att-data-oe-id="child.id" t-esc="child.name"/></span>
        <br/>
        <t t-if="child_subtype">
            <span><t t-esc="child_subtype"/></span><br/>
        </t>
    </template>
</odoo>

```

## File: views\project_update_templates.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="project.milestone_deadline">
<t t-if="milestone['deadline']">
(due <t t-esc="milestone['deadline']" t-options='{"widget": "date"}'/><t t-if="not milestone['is_reached'] or not milestone['reached_date']">)</t><t t-else=""> - reached on<t t-if="milestone['reached_date'] &gt; milestone['deadline']">
<font t-att-style="'color: rgb(' + str(color_level) + ', 0, 0)'"><b><t t-esc="milestone['reached_date']" t-options='{"widget": "date"}'/></b></font>)</t><t t-else="">
<font t-att-style="'color: rgb(0, ' + str(color_level) + ', 0)'"><b><t t-esc="milestone['reached_date']" t-options='{"widget": "date"}'/></b></font>)</t></t>
</t>
    </template>

    <template id="project_update_default_description" name="Project Update Description">
<!--As this template is rendered in an html field, the spaces may be interpreted as nbsp while editing. -->
<div name="summary">
<br/><h1 style="font-weight: bolder;">Sprint Summary</h1>
<br/><p>How’s this project going?</p><br/><br/>
</div>

<div name="activities" t-if="show_activities">
<h1 style="font-weight: bolder;">Activities</h1>
</div>

<div name="milestone" t-if="milestones['show_section']">
<t t-if="milestones['show_section']">
<br/>
<h3 style="font-weight: bolder"><u>Milestones</u></h3>

<ul class="o_checklist" t-if="milestones['list']">
<t t-foreach="milestones['list']" t-as="milestone" t-key="milestone['id']">
<li t-attf-class="{{milestone['is_reached'] and 'o_checked' or ''}}">
<t t-esc="milestone['name']"/>
<span t-if="milestone['is_deadline_future'] and not milestone['is_reached']"><font style="color: rgb(190, 190, 190);"><t t-set="color_level" t-value="64"/><t t-call="project.milestone_deadline"/></font></span>
<span t-else=""><t t-set="color_level" t-value="128"/><t t-call="project.milestone_deadline"/></span>
</li>
</t>
</ul>

<t t-if="milestones['updated']">
<t t-if="milestones['last_update_date']">Since <t t-esc="milestones['last_update_date']" t-options='{"widget": "date"}'/> (last project update), </t>
<t t-if="len(milestones['updated']) > 1">the deadline of the following milestones has been updated:</t>
<t t-else="">the deadline of the following milestone has been updated:</t>
<ul>
<t t-foreach="milestones['updated']" t-as="milestone" t-key="milestone['id']">
<li>
<t t-esc="milestone['name']"/> (<t t-esc="milestone['old_value']" t-options='{"widget": "date"}'/> =&gt; <t t-esc="milestone['new_value']"  t-options='{"widget": "date"}'/>)
</li>
</t>
</ul>
</t>

<t t-if="milestones['created']">
<t t-if="len(milestones['created']) > 1">The following milestones have been added:</t>
<t t-else="">The following milestone has been added:</t>
<ul>
<t t-foreach="milestones['created']" t-as="milestone" t-key="milestone['id']">
<li>
<t t-esc="milestone['name']"/><span t-if="milestone['is_deadline_future'] and not milestone['is_reached']">
<font style="color: rgb(190, 190, 190);">
<t t-set="color_level" t-value="64"/>
<t t-call="project.milestone_deadline"/>
</font>
</span>
<span t-else="">
<t t-set="color_level" t-value="128"/>
<t t-call="project.milestone_deadline"/>
</span>
</li>
</t>
</ul>
</t>
</t>
</div>
    </template>

</odoo>

```

## File: views\project_update_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_update_view_search" model="ir.ui.view">
        <field name="name">project.update.view.search</field>
        <field name="model">project.update</field>
        <field name="arch" type="xml">
            <search string="Search Update">
                <field name="name"/>
                <field name="project_id" invisible="1"/>
            </search>
        </field>
    </record>

    <record id="project_update_view_form" model="ir.ui.view">
        <field name="name">project.update.view.form</field>
        <field name="model">project.update</field>
        <field name="arch" type="xml">
            <form string="Project Update" class="o_form_project_update">
                <sheet>
                    <div class="oe_title">
                        <h1>
                            <field name="name" class="o_text_overflow" placeholder="Title of the Update"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="project_id" invisible="1"/>
                            <field name="color" invisible="1"/>
                            <field name="status" widget="status_with_color" options="{'color_field': 'color'}"/>
                            <field name="progress" widget="progressbar" options="{'current_value': 'progress', 'max_value': '100', 'editable': true}"/>
                        </group>
                        <group>
                            <field name="user_id" widget="many2one_avatar_user" readonly="1"/>
                            <field name="date"/>
                        </group>
                    </group>
                    <separator/>
                    <notebook>
                        <page string="Description" name="description">
                            <field name="description" widget="html_with_action" nolabel="1" class="o_project_update_description" options="{'collaborative': true}"/>
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

    <record id="project_update_view_kanban" model="ir.ui.view">
        <field name="name">project.update.view.kanban</field>
        <field name="model">project.update</field>
        <field name="arch" type="xml">
            <kanban class="o_pupdate_kanban" sample="1" js_class="project_update_kanban">
                <field name="color"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + record.color.raw_value : ''}} oe_kanban_global_click o_pupdate_kanban_card">
                            <!-- Project Update Kanban View is always ungrouped - see js_class -->
                            <div class="o_kanban_detail_ungrouped row">
                                <div class="col-lg-4 o_pupdate_name">
                                    <b><field name="name_cropped"/></b>
                                    <div>
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                        <t t-esc="record.user_id.value"/>
                                    </div>
                                </div>
                                <div class="col-lg-2">
                                    <field name="color" invisible="1"/>
                                    <b><field name="status" widget="status_with_color" options="{'color_field': 'color'}"/></b>
                                </div>
                                <div class="col-lg-2">
                                    <b><field name="progress_percentage" widget="percentage"/></b>
                                    <div>Progress</div>
                                </div>
                                <div class="col-lg-2">
                                    <b><field name="date"/></b>
                                    <div>Date</div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="project_update_view_tree" model="ir.ui.view">
        <field name="name">project.update.view.tree</field>
        <field name="model">project.update</field>
        <field name="arch" type="xml">
            <tree sample="1" js_class="project_update_list">
                <field name="name"/>
                <field name="user_id" widget="many2one_avatar_user" optional="show"/>
                <field name="progress_percentage" string="Progress" widget="percentage" optional="show"/>
                <field name="date" optional="show"/>
                <field name="color" invisible="1"/>
                <field name="status" widget="status_with_color" options="{'color_field': 'color'}"/>
            </tree>
        </field>
    </record>

    <record id="project_update_all_action" model="ir.actions.act_window">
        <field name="name">Project Updates</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">project.update</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="domain">[('project_id', '=', active_id)]</field>
        <field name="search_view_id" ref="project_update_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            No updates found. Let's create one!
            </p><p>
            Get a snapshot of the status of your project and share its progress with key stakeholders.
            </p>
        </field>
    </record>

    <record id="project_milestone_view_form" model="ir.ui.view">
        <field name="name">project.milestone.view.form</field>
        <field name="model">project.milestone</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <group>
                            <field name="project_id" invisible="1"/>
                            <field name="name" placeholder="E.g: Product Launch"/>
                            <field name="deadline"/>
                            <field name="is_reached"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="project_milestone_view_tree" model="ir.ui.view">
        <field name="name">project.milestone.view.tree</field>
        <field name="model">project.milestone</field>
        <field name="arch" type="xml">
            <tree decoration-danger="is_deadline_exceeded" decoration-muted="is_reached" editable="bottom">
                <field name="name"/>
                <field name="deadline" optional="show"/>
                <field name="is_reached" optional="show"/>
                <field name="is_deadline_exceeded" invisible="1"/>
            </tree>
        </field>
    </record>

    <record id="project_milestone_all" model="ir.actions.act_window">
        <field name="name">Milestones</field>
        <field name="res_model">project.milestone</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'default_project_id': active_id}</field>
        <field name="domain">[('project_id', '=', active_id)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            No milestones found. Let's create one!
            </p><p>
            Track major progress points that must be reached to achieve success.
            </p>
        </field>
    </record>
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
            sequence="70"/>

        <menuitem id="menu_project_config" name="Configuration" parent="menu_main_pm"
            sequence="100" groups="project.group_project_manager"/>

        <record id="view_task_search_form" model="ir.ui.view">
            <field name="name">project.task.search.form</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
               <search string="Tasks">
                    <field name="name" string="Task"/>
                    <field name="tag_ids"/>
                    <field name="user_ids" filter_domain="[('user_ids.name', 'ilike', self), ('user_ids.active', 'in', [True, False])]"/>
                    <field string="Project" name="display_project_id"/>
                    <field name="stage_id"/>
                    <field name="partner_id" operator="child_of"/>
                    <field name="parent_id"/>
                    <filter string="My Tasks" name="my_tasks" domain="[('user_ids', 'in', uid)]"/>
                    <filter string="Unassigned" name="unassigned" domain="[('user_ids', '=', False)]"/>
                    <separator/>
                    <filter string="Starred" name="starred" domain="[('priority', 'in', [0, 1])]"/>
                    <separator/>
                    <filter string="Late Tasks" name="late" domain="[('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d')), ('is_closed', '=', False)]"/>
                    <separator/>
                    <filter string="Rated Tasks" name="rating_task" domain="[('rating_last_value', '!=', 0.0)]" groups="project.group_project_rating"/>
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
                        <filter string="Personal Stage" name="personal_stage" context="{'group_by': 'personal_stage_type_ids'}"/>
                        <filter string="Assignees" name="user" context="{'group_by': 'user_ids'}"/>
                        <filter string="Project" name="project" context="{'group_by': 'project_id'}"/>
                        <filter string="Customer" name="customer" context="{'group_by': 'partner_id'}"/>
                        <filter string="Kanban State" name="kanban_state" context="{'group_by': 'kanban_state'}"/>
                        <filter string="Deadline" name="date_deadline" context="{'group_by': 'date_deadline'}"/>
                        <filter string="Creation Date" name="group_create_date" context="{'group_by': 'create_date'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="view_task_search_form_extended" model="ir.ui.view">
            <field name="name">project.task.search.form.extended</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_search_form"></field>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
               <xpath expr="//filter[@name='unassigned']" position="after">
                    <separator/>
                    <filter string="My Projects" name="my_projects" domain="[('project_id.user_id', '=', uid)]"/>
                    <filter string="My Favorite Projects" name="my_favorite_projects" domain="[('project_id.favorite_user_ids', 'in', [uid])]"/>
                    <filter string="Followed Projects" name="my_followed_projects" domain="[('project_id.message_is_follower', '=', True)]"/>
                </xpath>
            </field>
        </record>

        <record id="act_project_project_2_project_task_all" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form,calendar,pivot,graph,activity</field>
            <field name="domain">[('display_project_id', '=', active_id)]</field>
            <field name="context">{
                'pivot_row_groupby': ['user_ids'],
                'default_project_id': active_id,
                'show_project_update': True,
            }</field>
            <field name="search_view_id" ref="view_task_search_form"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p><p>
                    To get things done, use activities and status on tasks.<br/>
                    Chat in real-time or by email to collaborate efficiently.
                </p>
            </field>
        </record>

        <!-- Task types -->
        <record id="task_type_search" model="ir.ui.view">
            <field name="name">project.task.type.search</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <search string="Tasks Stages">
                   <field name="name" string="Name"/>
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
                                <field name="mail_template_id" context="{'default_model': 'project.task'}"/>
                                <field name="rating_template_id" groups="project.group_project_rating" context="{'default_model': 'project.task'}"/>
                                <field name="sequence" groups="base.group_no_one"/>
                                <div class="alert alert-warning" role="alert" colspan='2' attrs="{'invisible': ['|', ('rating_template_id','=', False), ('disabled_rating_warning', '=', False)]}">
                                    <i class="fa fa-warning" title="Customer disabled on projects"/><b> Customer Ratings</b> are disabled on the following project(s) : <br/>
                                    <field name="disabled_rating_warning" class="mb-0" />
                                </div>
                                <field name="auto_validation_kanban_state" attrs="{'invisible': [('rating_template_id','=', False)]}" groups="project.group_project_rating"/>
                            </group>
                            <group>
                                <field name="fold"/>
                                <field name="is_closed" groups="base.group_no_one"/>
                                <field name="project_ids" widget="many2many_tags" options="{'color_field': 'color'}" required="1"/>
                            </group>
                        </group>
                        <group string="Stage Description and Tooltips">
                            <group>
                                <p class="text-muted" colspan="2">
                                    At each stage, employees can block tasks or mark them as ready for the next step.
                                    You can customize here the labels for each state.
                                </p>
                                <div class="row ml-1" colspan="2">
                                    <label for="legend_normal" string=" " class="o_status mt4"
                                        title="Task in progress. Click to block or set as done."
                                        aria-label="Task in progress. Click to block or set as done." role="img"/>
                                    <div class="col-11 pl-0">
                                        <field name="legend_normal"/>
                                    </div>
                                </div>
                                <div class="row ml-1" colspan="2">
                                    <label for="legend_blocked" string=" " class="o_status o_status_red mt4"
                                        title="Task is blocked. Click to unblock or set as done."
                                        aria-label="Task is blocked. Click to unblock or set as done." role="img"/>
                                    <div class="col-11 pl-0">
                                        <field name="legend_blocked"/>
                                    </div>
                                </div>
                                <div class="row ml-1" colspan="2">
                                    <label for="legend_done" string=" " class="o_status o_status_green mt4"
                                        title="This step is done. Click to block or set in progress."
                                        aria-label="This step is done. Click to block or set in progress." role="img"/>
                                    <div class="col-11 pl-0">
                                        <field name="legend_done"/>
                                    </div>
                                </div>

                                <p class="text-muted mt-2" colspan="2">
                                    You can also add a description to help your coworkers understand the meaning and purpose of the stage.
                                </p>
                                <field name="description" placeholder="Add a description..." nolabel="1" colspan="2"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="personal_task_type_edit" model="ir.ui.view">
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
                                <field name="sequence" groups="base.group_no_one"/>
                            </group>
                            <group>
                                <field name="fold"/>
                            </group>
                        </group>
                        <group>
                            <group>
                                <field name="description" placeholder="Add a description..." nolabel="1" colspan="2"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="task_type_tree" model="ir.ui.view">
            <field name="name">project.task.type.tree</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <tree string="Task Stage" delete="0" sample="1" multi_edit="1">
                    <field name="sequence" widget="handle" optional="show"/>
                    <field name="name"/>
                    <field name="mail_template_id" optional="hide"/>
                    <field name="rating_template_id" optional="hide" groups="project.group_project_rating"/>
                    <field name="project_ids" optional="show" widget="many2many_tags" options="{'color_field': 'color'}" />
                    <field name="fold" optional="show"/>
                    <field name="user_id" optional="show"/>
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
                                <field name="project_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
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
            <field name="name">Task Stages</field>
            <field name="res_model">project.task.type</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_id" ref="task_type_tree"/>
            <field name="domain">[('user_id', '=', False)]</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No stages found. Let's create one!
              </p><p>
                Track the progress of your tasks from their creation to their closing.
              </p>
            </field>
        </record>

        <record id="open_task_type_form_domain" model="ir.actions.act_window">
            <field name="name">Task Stages</field>
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
                <form string="Project" class="o_form_project_project" delete="0">
                    <header>
                        <button name="%(project.project_share_wizard_action)d" string="Share Readonly" type="action" class="oe_highlight" groups="project.group_project_manager"
                        attrs="{'invisible': [('privacy_visibility', '!=', 'portal')]}" context="{'default_access_mode': 'read'}"/>
                        <button name="%(project.project_share_wizard_action)d" string="Share Editable" type="action" class="oe_highlight" groups="project.group_project_manager"
                        attrs="{'invisible': [('privacy_visibility', '!=', 'portal')]}" context="{'default_access_mode': 'edit'}"/>
                        <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" groups="project.group_project_stages"/>
                    </header>
                <sheet string="Project">
                    <div class="oe_button_box" name="button_box" groups="base.group_user">
                        <field name="currency_id" invisible="1"/>
                        <button class="oe_stat_button" type="action"
                            name="%(act_project_project_2_project_task_all)d" icon="fa-tasks">
                            <field string="Tasks In Progress" name="task_count" widget="statinfo" options="{'label_field': 'label_tasks'}"/>
                        </button>
                        <button class="oe_stat_button" name="%(project.project_milestone_all)d" type="action" icon="fa-check-square-o">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="milestone_count" nolabel="1"/>
                                </span>
                                <span class="o_stat_text">
                                    Milestones
                                </span>
                            </div>
                        </button>
                        <button name="action_view_all_rating" type="object" attrs="{'invisible': ['|', ('rating_active', '=', False), ('rating_percentage_satisfaction', '=', -1)]}" class="oe_stat_button oe_read_only" icon="fa-smile-o" groups="project.group_project_rating">
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
                        <button name="%(action_project_task_burndown_chart_report)d" type="action" class="oe_stat_button" icon="fa-area-chart" groups="project.group_project_manager">
                            <span class="o_stat_text">
                                Burndown Chart
                            </span>
                        </button>
                        <button class="oe_stat_button" type="object" name="action_view_analytic_account_entries" icon="fa-usd" attrs="{'invisible': [('analytic_account_id', '=', False)]}" groups="analytic.group_analytic_accounting">
                            <div class="o_form_field o_stat_info">
                                <span class="o_stat_value">
                                    <field name="analytic_account_balance"/>
                                </span>
                                <span class="o_stat_text">Gross Margin</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="%(project.project_collaborator_action)d" type="action" icon="fa-users" groups="project.group_project_manager">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="collaborator_count" nolabel="1"/>
                                </span>
                                <span class="o_stat_text">
                                    Collaborators
                                </span>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="%(project.project_update_all_action)d" type="action" groups="project.group_project_manager">
                            <div class="pl-4">
                                <field name="last_update_color" invisible="1"/>
                                <field name="last_update_status" widget="status_with_color" options="{'color_field': 'last_update_color'}"/>
                            </div>
                        </button>
                        <button class="oe_stat_button o_project_not_clickable" disabled="disabled" groups="!project.group_project_manager">
                            <div class="pl-4">
                                <field name="last_update_color" invisible="1"/>
                                <field name="last_update_status" widget="status_with_color" options="{'color_field': 'last_update_color', 'no_quick_edit': 1}"/>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <h1>
                            <field name="name" class="text-break" placeholder="Project Name"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="label_tasks" string="Name of the tasks"/>
                            <field name="partner_id" widget="res_partner_many2one"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="user_id" string="Project Manager" widget="many2one_avatar_user" attrs="{'readonly':[('active','=',False)]}" domain="[('share', '=', False)]"/>
                            <label for="date_start" string="Dates"/>
                            <div name="dates" class="o_row">
                                <field name="date_start" widget="daterange" options='{"related_end_date": "date"}'/>
                                <i class="fa fa-long-arrow-right mx-2 oe_edit_only" aria-label="Arrow icon" title="Arrow"/>
                                <i class="fa fa-long-arrow-right mx-2 oe_read_only" aria-label="Arrow icon" title="Arrow" attrs="{'invisible': [('date_start', '=', False), ('date', '=', False)]}"/>
                                <field name="date" widget="daterange" options='{"related_start_date": "date_start"}'/>
                            </div>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="description" string="Description">
                            <field name="description"/>
                        </page>
                        <page name="settings" string="Settings">
                            <group>
                                <group>
                                    <field name="analytic_account_id" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]" context="{'default_partner_id': partner_id}" groups="analytic.group_analytic_accounting"/>
                                    <field name="privacy_visibility" widget="radio"/>
                                </group>
                                <group>
                                    <div name="alias_def" colspan="2" class="pb-2" attrs="{'invisible': [('alias_domain', '=', False)]}">
                                        <!-- Always display the whole alias in edit mode. It depends in read only -->
                                        <field name="alias_enabled" invisible="1"/>
                                        <span class="font-weight-bold oe_read_only" attrs="{'invisible': [('alias_name', '!=', False)]}" style="opacity: 0.7;">Create tasks by sending an email to </span>
                                        <span class="font-weight-bold oe_read_only text-dark" attrs="{'invisible': [('alias_name', '=', False)]}">Create tasks by sending an email to </span>
                                        <span class="font-weight-bold oe_edit_only text-dark">Create tasks by sending an email to </span>
                                            <field name="alias_value" class="oe_read_only d-inline" readonly="1" widget="email" attrs="{'invisible':  [('alias_name', '=', False)]}" />
                                            <span class="oe_edit_only">
                                                <field name="alias_name" class="oe_inline"/>@<field name="alias_domain" class="oe_inline" readonly="1"/>
                                            </span>
                                    </div>
                                    <!-- the alias contact must appear when the user start typing and it must disappear
                                        when the string is deleted. -->
                                    <field name="alias_contact" class="oe_inline" string="Accept Emails From"
                                           attrs="{'invisible': ['|', ('alias_name', '=', ''), ('alias_name', '=', False)]}"/>
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
                                <div class="col-lg-6 o_setting_box" id="task_dependencies_setting" groups="project.group_project_task_dependencies">
                                    <div class="o_setting_left_pane">
                                        <field name="allow_task_dependencies"/>
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label for="allow_task_dependencies"/>
                                        <div class="text-muted">
                                            Determine the order in which to perform tasks
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" options="{'post_refresh':True}" help="Follow this project to automatically track the events associated to tasks and issues of this project." groups="base.group_user"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
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
                    <field name="analytic_account_id"/>
                    <field name="stage_id" groups="project.group_project_stages"/>
                    <field name="tag_ids"/>
                    <filter string="My Favorites" name="my_projects" domain="[('favorite_user_ids', 'in', uid)]"/>
                    <separator/>
                    <filter string="Followed" name="followed_by_me" domain="[('message_is_follower', '=', True)]"/>
                    <filter string="My Projects" name="own_projects" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter string="Open" name="open" domain="[('stage_id.fold', '=', False)]" groups="project.group_project_stages"/>
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
                        <filter string="Project Manager" name="Manager" context="{'group_by': 'user_id'}"/>
                        <filter string="Customer" name="Partner" context="{'group_by': 'partner_id'}"/>
                        <filter string="Status" name="status" context="{'group_by': 'last_update_status'}"/>
                        <filter string="Stage" name="groupby_stage" context="{'group_by': 'stage_id'}" groups="project.group_project_stages"/>
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
                    <field name="partner_id" optional="show" string="Customer"/>
                    <field name="privacy_visibility" optional="hide"/>
                    <field name="company_id" optional="show"  groups="base.group_multi_company" options="{'no_create': True, 'no_create_edit': True}"/>
                    <field name="analytic_account_id" optional="hide" groups="analytic.group_analytic_accounting"/>
                    <field name="date_start" string="Start Date" widget="daterange" options="{'related_end_date': 'date'}"/>
                    <field name="date" string="End Date" widget="daterange" options="{'related_start_date': 'date_start'}"/>
                    <field name="user_id" optional="show" string="Project Manager" widget="many2one_avatar_user" options="{'no_open':True, 'no_create': True, 'no_create_edit': True}"/>
                    <field name="last_update_color" invisible="1"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                    <field name="last_update_status" string="Status" optional="show" widget="status_with_color" options="{'color_field': 'last_update_color'}"/>
                    <field name="stage_id" options="{'no_open': True}" optional="show"/>
                </tree>
            </field>
        </record>

        <record id="project_view_kanban" model="ir.ui.view">
            <field name="name">project.project.kanban</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="user_id" string="Project Manager"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_content oe_kanban_global_click o_kanban_get_form">
                                <div class="row">
                                    <div class="col-12">
                                        <strong><field name="name" string="Project Name"/></strong>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-8">
                                        <field name="partner_id" string="Contact"/>
                                    </div>
                                    <div class="col-4">
                                        <div class="oe_kanban_bottom_right float-right">
                                            <field name="user_id" widget="many2one_avatar_user"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
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
                        <button string="Create" name="action_view_tasks" type="object" class="btn-primary o_open_tasks" data-hotkey="q"/>
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z"/>
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
            <field name="context">{"default_allow_billable": 1}</field>
        </record>

        <record model="ir.ui.view" id="view_project_kanban">
            <field name="name">project.project.kanban</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <kanban class="oe_background_grey o_kanban_dashboard o_project_kanban o_emphasize_colors" on_create="project.open_create_project" js_class="project_project_kanban" sample="1">
                    <field name="name"/>
                    <field name="partner_id"/>
                    <field name="commercial_partner_id"/>
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
                    <field name="date"/>
                    <field name="doc_count"/>
                    <field name="privacy_visibility"/>
                    <field name="last_update_color"/>
                    <field name="last_update_status"/>
                    <field name="tag_ids"/>
                    <progressbar field="last_update_status" colors='{"on_track": "success", "at_risk": "warning", "off_track": "danger", "on_hold": "info"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{kanban_color(record.color.raw_value)} oe_kanban_global_click o_has_icon oe_kanban_content oe_kanban_card">
                                <div class="o_project_kanban_main ">
                                    <div class="o_kanban_card_content mw-100">
                                        <div class="o_kanban_primary_left">
                                            <div class="o_primary">
                                                <span class="o_text_overflow"><t t-esc="record.name.value"/></span>
                                                <span class="o_text_overflow text-muted" t-if="record.partner_id.value">
                                                    <span class="fa fa-user mr-2" aria-label="Partner" title="Partner"></span><t t-esc="record.partner_id.value"/>
                                                </span>
                                                <div t-if="record.date.raw_value or record.date_start.raw_value" class="text-muted o_row">
                                                    <span class="fa fa-clock-o mr-2" title="Dates"></span><field name="date_start"/>
                                                    <i t-if="record.date.raw_value and record.date_start.raw_value" class="fa fa-long-arrow-right mx-2 oe_read_only" aria-label="Arrow icon" title="Arrow"/>
                                                    <field name="date"/>
                                                </div>
                                                <div t-if="record.alias_name.value and record.alias_domain.value" class="text-muted">
                                                    <span class="fa fa-envelope-o mr-2" aria-label="Domain Alias" title="Domain Alias"></span><t t-esc="record.alias_id.value"/>
                                                </div>
                                                <div t-if="record.rating_active.raw_value" class="text-muted" title="Percentage of happy ratings over the past 30 days." groups="project.group_project_rating">
                                                    <b>
                                                        <t t-if="record.rating_percentage_satisfaction.value != -1">
                                                            <i class="fa fa-smile-o" role="img" aria-label="Percentage of satisfaction" title="Percentage of satisfaction"/> <t t-esc="record.rating_percentage_satisfaction.value"/>%
                                                        </t>
                                                    </b>
                                                </div>
                                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="container o_kanban_card_manage_pane dropdown-menu" role="menu" groups="base.group_user">
                                        <div class="row">
                                            <div class="col-6 o_kanban_card_manage_section o_kanban_manage_view">
                                                <div role="menuitem" class="o_kanban_card_manage_title">
                                                    <span>View</span>
                                                </div>
                                                <div role="menuitem">
                                                    <a name="action_view_tasks" type="object">Tasks</a>
                                                </div>
                                                <div role="menuitem" t-if="record.doc_count.raw_value">
                                                    <a name="attachment_tree_view" type="object">Documents</a>
                                                </div>
                                            </div>
                                            <div class="col-6 o_kanban_card_manage_section o_kanban_manage_reporting" groups="project.group_project_manager">
                                                <div role="menuitem" class="o_kanban_card_manage_title">
                                                    <span>Reporting</span>
                                                </div>
                                                <div role="menuitem">
                                                    <a name="action_view_tasks_analysis" type="object">Tasks Analysis</a>
                                                </div>
                                                <div role="menuitem">
                                                    <a name="%(action_project_task_burndown_chart_report)d" type="action">Burndown Chart</a>
                                                </div>
                                            </div>
                                        </div>
                                        <div class="o_kanban_card_manage_settings row" groups="project.group_project_manager">
                                            <div role="menuitem" aria-haspopup="true" class="col-8">
                                                <ul class="oe_kanban_colorpicker" data-field="color" role="popup"/>
                                            </div>
                                            <div role="menuitem" class="col-4">
                                                <a t-if="record.privacy_visibility.raw_value == 'portal'" class="dropdown-item" role="menuitem" name="%(project.project_share_wizard_action)d" type="action">Share</a>
                                                <!-- [XBO] TODO: remove the name attribute in this a tag in master -->
                                                <a class="dropdown-item" role="menuitem" type="edit" name="action_view_kanban_project">Edit</a>
                                            </div>
                                        </div>
                                    </div>
                                    <a class="o_kanban_manage_toggle_button o_dropdown_kanban" href="#" groups="base.group_user">
                                        <i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/>
                                    </a>
                                    <span>
                                       <field name="is_favorite" widget="boolean_favorite" nolabel="1" force_save="1" />
                                    </span>
                                </div>
                                <div class="o_kanban_record_bottom mt-3">
                                    <div class="oe_kanban_bottom_left">
                                        <div class="o_project_kanban_boxes">
                                            <a class="o_project_kanban_box" name="action_view_tasks" type="object">
                                                <div>
                                                    <span class="o_value"><t t-esc="record.task_count.value"/></span>
                                                    <span class="o_label"><t t-esc="record.label_tasks.value"/></span>
                                                </div>
                                            </a>
                                        </div>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <span t-att-class="'o_status_bubble mx-0 o_color_bubble_' + record.last_update_color.value" t-att-title="record.last_update_status.value"></span>
                                        <field name="user_id" widget="many2one_avatar_user" t-if="record.user_id.raw_value"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <!-- Please update both act_window when modifying one (open_view_project_all or open_view_project_all_group_stage) as one or the other is used in the menu menu_project_config -->
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
                    Projects regroup tasks on the same topic, and each has its dashboard.
                </p>
            </field>
        </record>

        <record id="open_view_project_all_group_stage" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="domain">[]</field>
            <field name="view_mode">kanban,form</field>
            <field name="view_id" ref="view_project_kanban"/>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="target">main</field>
            <field name="context">{'search_default_groupby_stage': 1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No projects found. Let's create one!
                </p><p>
                    Projects regroup tasks on the same topic, and each has its dashboard.
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
                    Projects regroup tasks on the same topic, and each has its dashboard.
                </p>
            </field>
        </record>

        <!-- Task -->
        <record id="view_task_form2" model="ir.ui.view">
            <field name="name">project.task.form</field>
            <field name="model">project.task</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <form string="Task" class="o_form_project_tasks" js_class="project_form">
                    <field name="allow_subtasks" invisible="1" />
                    <field name="is_closed" invisible="1" />
                    <field name="allow_recurring_tasks" invisible="1" />
                    <field name="repeat_show_dow" invisible="1" />
                    <field name="repeat_show_day" invisible="1" />
                    <field name="repeat_show_week" invisible="1" />
                    <field name="repeat_show_month" invisible="1" />
                    <field name="recurrence_id" invisible="1" />
                    <field name="allow_task_dependencies" invisible="1" />
                    <header>
                        <button name="action_assign_to_me" string="Assign to Me" type="object" class="oe_highlight"
                            attrs="{'invisible' : &quot;[('user_ids', 'in', uid)]&quot;}" data-hotkey="q"/>
                        <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" attrs="{'invisible': [('project_id', '=', False), ('stage_id', '=', False)]}"/>
                    </header>
                    <div class="alert alert-info oe_edit_only mb-0" role="status" attrs="{'invisible': ['|', ('recurring_task', '=', False), ('recurrence_id', '=', False)]}">
                        <p>Edit recurring task</p>
                        <field name="recurrence_update" widget="radio"/>
                    </div>
                    <sheet string="Task">
                    <div class="oe_button_box" name="button_box">
                        <button name="action_open_parent_task" type="object" class="oe_stat_button" icon="fa-tasks" string="Parent Task" attrs="{'invisible': ['|', ('allow_subtasks', '=', False), ('parent_id', '=', False)]}" groups="project.group_subtask_project"/>
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
                        <button name="action_dependent_tasks" type="object" attrs="{'invisible': [('dependent_tasks_count', '=', 0)]}" class="oe_stat_button" icon="fa-tasks" groups="project.group_project_task_dependencies">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Blocking</span>
                                <span class="o_stat_value ">
                                    <field name="dependent_tasks_count" widget="statinfo" nolabel="1" />
                                    Tasks
                                </span>
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
                            <label for="project_id" attrs="{'invisible': [('parent_id', '!=', False)]}" class="oe_read_only"/>
                            <div colspan="1" class="oe_read_only">
                                <i class="o_project_task_project_field text-danger" attrs="{'invisible': [('project_id', '!=', False)]}">Private</i>
                                <field name="project_id" attrs="{'invisible': ['|', ('parent_id', '!=', False), ('project_id', '=', False)]}" nolabel="1"/>
                            </div>
                            <field name="project_id" attrs="{'invisible': [('parent_id', '!=', False)]}" domain="[('active', '=', True), ('company_id', '=', company_id)]"
                                placeholder="Private" class="o_project_task_project_field oe_edit_only"/>
                            <field name="display_project_id" string="Project" attrs="{'invisible': [('parent_id', '=', False)]}" domain="[('active', '=', True), ('company_id', '=', company_id)]"/>
                            <field name="user_ids"
                                class="o_task_user_field"
                                options="{'no_open': True}"
                                widget="many2many_avatar_user"
                                domain="[('share', '=', False), ('active', '=', True)]"/>
                            <field name="parent_id"
                                attrs="{'invisible' : [('allow_subtasks', '=', False)]}"
                                groups="base.group_no_one"
                            />
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="partner_id" widget="res_partner_many2one" class="o_task_customer_field"/>
                            <field name="partner_phone" widget="phone" attrs="{'invisible': True}"/>
                            <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                            <field name="recurring_task" attrs="{'invisible': ['|', ('allow_recurring_tasks', '=', False), ('active', '=', False)]}"/>
                            <field name="legend_blocked" invisible="1"/>
                            <field name="legend_normal" invisible="1"/>
                            <field name="legend_done" invisible="1"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" type="html" options="{'collaborative': true}"/>
                        </page>
                        <page name="sub_tasks_page" string="Sub-tasks" attrs="{'invisible': [('allow_subtasks', '=', False)]}">
                            <field name="child_ids" context="{'default_project_id': project_id if not parent_id or not display_project_id else display_project_id, 'default_user_ids': user_ids, 'default_parent_id': id, 'default_partner_id': partner_id}">
                                <tree editable="bottom">
                                    <field name="project_id" invisible="1"/>
                                    <field name="is_closed" invisible="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="name"/>
                                    <field name="display_project_id" string="Project" optional="hide"/>
                                    <field name="partner_id" optional="hide"/>
                                    <field name="user_ids" widget="many2many_avatar_user" optional="show" domain="[('share', '=', False), ('active', '=', True)]"/>
                                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                                    <field name="activity_ids" widget="list_activity" optional="hide"/>
                                    <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}" optional="show"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="kanban_state" widget="state_selection" optional="hide"/>
                                    <field name="stage_id" optional="show" context="{'default_project_id': project_id}"/>
                                    <button name="action_open_task" type="object" title="View Task" string="View Task" class="btn btn-link pull-right"/>
                                </tree>
                            </field>
                        </page>
                        <page name="task_dependencies" string="Blocked By" attrs="{'invisible': [('allow_task_dependencies', '=', False)]}" groups="project.group_project_task_dependencies">
                            <field name="depend_on_ids" nolabel="1">
                                <tree editable="bottom">
                                    <field name="is_closed" invisible="1" />
                                    <field name="name" />
                                    <field name="project_id" optional="hide" />
                                    <field name="user_ids" widget="many2many_avatar_user" optional="show" domain="[('share', '=', False), ('active', '=', True)]"/>
                                    <field name="company_id" optional="hide" groups="base.group_multi_company" />
                                    <field name="activity_ids" widget="list_activity" optional="hide"/>
                                    <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}" optional="show" />
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="kanban_state" widget="state_selection" optional="hide" />
                                    <field name="stage_id" optional="show" />
                                    <button class="oe_link float-right" string="View Task" name="action_open_task" type="object"/>
                                </tree>
                            </field>
                        </page>
                        <page name="recurrence" string="Recurrence" attrs="{'invisible': [('recurring_task', '=', False)]}">
                            <group>
                                <group>
                                    <label for="repeat_interval" />
                                    <div class="o_col">
                                        <div class="o_row">
                                            <field name="repeat_interval" attrs="{'required': [('recurring_task', '=', True)]}" />
                                            <field name="repeat_unit" attrs="{'required': [('recurring_task', '=', True)]}" />
                                        </div>
                                        <widget name="week_days" attrs="{'invisible': [('repeat_show_dow', '=', False)]}"/>
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
                                    <!-- Those fields are added to trigger the compute method for the recurrence feature. -->
                                    <field name="mon" invisible="1"/>
                                    <field name="tue" invisible="1"/>
                                    <field name="wed" invisible="1"/>
                                    <field name="thu" invisible="1"/>
                                    <field name="fri" invisible="1"/>
                                    <field name="sat" invisible="1"/>
                                    <field name="sun" invisible="1"/>

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
                                    <field name="recurrence_message" widget="html" class="mb-0" />
                                </div>
                            </group>
                        </page>
                        <page name="extra_info" string="Extra Info" groups="base.group_no_one">
                            <group>
                                <group>
                                    <field name="analytic_account_id" groups="analytic.group_analytic_accounting" context="{'default_partner_id': partner_id}"/>
                                    <field name="analytic_tag_ids" groups="analytic.group_analytic_tags" widget="many2many_tags"/>
                                    <field name="sequence" groups="base.group_no_one"/>
                                    <field name="email_from" invisible="1"/>
                                    <field name="email_cc" groups="base.group_no_one"/>
                                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                    <field name="displayed_image_id" groups="base.group_no_one" options="{'no_create': True}"/>
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
                        <field name="name" string = "Task Title" placeholder="e.g. New Design"/>
                        <field name="user_ids" options="{'no_open': True,'no_create': True}" domain="[('share', '=', False), ('active', '=', True)]"
                            widget="many2many_avatar_user"/>
                        <field name="company_id" invisible="1"/>
                        <field name="parent_id" invisible="1" groups="base.group_no_one"/>
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
                <kanban default_group_by="stage_id" class="o_kanban_small_column o_kanban_project_tasks" on_create="quick_create"
                    quick_create_view="project.quick_create_task_form" examples="project" js_class="project_task_kanban" sample="1">
                    <field name="color"/>
                    <field name="priority"/>
                    <field name="stage_id" options='{"group_by_tooltip": {"description": "Description"}}'/>
                    <field name="user_ids"/>
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
                    <field name="allow_subtasks"/>
                    <field name="child_text"/>
                    <field name="is_private"/>
                    <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger", "normal": "muted"}'/>
                    <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click">
                            <div class="oe_kanban_content">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title">
                                            <s t-if="!record.active.raw_value"><field name="name" widget="name_with_subtask_count"/></s>
                                            <t t-else=""><field name="name" widget="name_with_subtask_count"/></t>
                                        </strong>
                                        <t t-if="!record.is_private.raw_value">
                                            <span invisible="context.get('default_project_id', False)"><br/><field name="project_id" required="1"/></span>
                                        </t>
                                        <t t-else="">
                                            <br/>
                                            <span class="fa fa-lock text-muted"/><span class="text-muted"> Private</span>
                                        </t>
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
                                            <div role="separator" class="dropdown-divider"></div>
                                            <ul class="oe_kanban_colorpicker" data-field="color"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="o_kanban_record_body">
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                    <div t-if="record.date_deadline.raw_value" name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}">
                                        <field name="date_deadline" widget="remaining_days"/>
                                    </div>
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
                                            <span style="font-weight:bold;" class="fa fa-fw mt4 fa-meh-o text-warning" t-if="record.rating_last_value.value == 3" title="Latest Rating: Okay" role="img" aria-label="Neutral face"/>
                                            <span style="font-weight:bold;" class="fa fa-fw mt4 fa-frown-o text-danger" t-if="record.rating_last_value.value == 1" title="Latest Rating: Dissatisfied" role="img" aria-label="Sad face"/>
                                        </b>
                                    </div>
                                    <div class="oe_kanban_bottom_right" t-if="!selection_mode">
                                        <field name="kanban_state" widget="state_selection" groups="base.group_user" invisible="context.get('fsm_mode', False)"/>
                                        <t t-if="record.user_ids.raw_value"><field name="user_ids" widget="many2many_avatar_user"/></t>
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
                <tree string="Tasks" multi_edit="1" sample="1" js_class="project_list">
                    <field name="message_needaction" invisible="1" readonly="1"/>
                    <field name="is_closed" invisible="1" />
                    <field name="allow_subtasks" invisible="1" />
                    <field name="sequence" invisible="1" readonly="1"/>
                    <field name="priority" widget="priority" optional="show" nolabel="1"/>
                    <field name="name" widget="name_with_subtask_count"/>
                    <field name="child_text" invisible="1"/>
                    <field name="project_id" optional="show" readonly="1"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="parent_id" optional="hide" attrs="{'invisible': [('allow_subtasks', '=', False)]}" groups="base.group_no_one"/>
                    <field name="user_ids" optional="show" widget="many2many_avatar_user" domain="[('share', '=', False), ('active', '=', True)]"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="date_deadline" optional="hide" widget="remaining_days" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                    <field name="kanban_state" widget="selection" optional="hide" />
                    <field name="stage_id" invisible="context.get('set_visible',False)" optional="show" readonly="not context.get('default_project_id')"/>
                    <field name="recurrence_id" invisible="1" />
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
                <calendar date_start="date_deadline" string="Tasks" mode="month" color="user_ids" event_limit="5"
                          hide_time="true" js_class="project_calendar" event_open_popup="true" quick_add="false">
                    <field name="project_id" filters="1"/>
                    <field name="user_ids" widget="many2many_avatar_user"/>
                    <field name="partner_id" attrs="{'invisible': [('partner_id', '=', False)]}"/>
                    <field name="priority" widget="priority"/>
                    <field name="date_deadline"/>
                    <field name="tag_ids" widget="many2many_tags" attrs="{'invisible': [('tag_ids', '=', [])]}"/>
                    <field name="stage_id"/>
                    <field name="kanban_state"/>
                </calendar>
            </field>
        </record>

        <record id="view_project_task_pivot" model="ir.ui.view">
            <field name="name">project.task.pivot</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <pivot string="Tasks" sample="1" js_class="project_pivot">
                    <field name="project_id" type="row"/>
                    <field name="stage_id" type="col"/>
                    <field name="color" invisible="1"/>
                    <field name="rating_last_value" type="measure" string="Rating (/5)"/>
                    <field name="sequence" invisible="1"/>
                    <field name="planned_hours" widget="float_time"/>
                    <field name="working_hours_close" widget="float_time"/>
                    <field name="working_hours_open" widget="float_time"/>
                </pivot>
            </field>
        </record>

        <record id="view_project_task_graph" model="ir.ui.view">
            <field name="name">project.task.graph</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <graph string="Tasks" sample="1" js_class="project_graph">
                    <field name="project_id"/>
                    <field name="stage_id"/>
                    <field name="color" invisible="1"/>
                    <field name="sequence" invisible="1"/>
                    <field name="rating_last_value" type="measure" string="Rating (/5)"/>
                </graph>
            </field>
        </record>

        <record id="project_task_view_activity" model="ir.ui.view">
            <field name="name">project.task.activity</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <activity string="Project Tasks" js_class="project_activity">
                    <field name="user_ids"/>
                    <templates>
                        <div class="justify-content-between" t-name="activity-box">
                            <field name="user_ids" widget="many2many_avatar_user"/>
                            <div class="text-right">
                                <span t-att-title="record.name.value">
                                    <field name="name" display="full"/>
                                </span>
                                <span t-att-title="record.project_id.value">
                                    <field name="project_id" muted="1" display="full" invisible="context.get('default_project_id', False)"/>
                                </span>
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
            <field name="domain">[('display_project_id', '!=', False)]</field>
            <field name="search_view_id" ref="view_task_search_form"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p><p>
                    To get things done, use activities and status on tasks.<br/>
                    Chat in real-time or by email to collaborate efficiently.
                </p>
            </field>
        </record>

        <record id="action_view_all_task" model="ir.actions.act_window">
            <field name="name">My Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form,calendar,pivot,graph,activity</field>
            <field name="context">{'search_default_my_tasks': 1, 'search_default_personal_stage': 1, 'all_task': 0}</field>
            <field name="search_view_id" ref="view_task_search_form_extended"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p><p>
                     To get things done, use activities and status on tasks.<br/>
                    Chat in real-time or by email to collaborate efficiently.
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

        <menuitem name="My Tasks" id="menu_project_management" parent="menu_main_pm"
            action="action_view_all_task" sequence="2" groups="base.group_no_one,group_project_user"/>

        <record id="project_task_action_from_partner" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="search_view_id" ref="view_task_search_form_extended"/>
        </record>

        <record id="action_view_task_overpassed_draft" model="ir.actions.act_window">
            <field name="name">Overpassed Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,form,calendar,graph,kanban</field>
            <field name="domain">[('is_closed', '=', False), ('date_deadline','&lt;',time.strftime('%Y-%m-%d')), ('display_project_id', '!=', False)]</field>
            <field name="filter" eval="True"/>
            <field name="search_view_id" ref="view_task_search_form_extended"/>
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

        <menuitem action="project_project_stage_configure" id="menu_project_config_project_stage" name="Project Stages" parent="menu_project_config" sequence="9" groups="project.group_project_stages"/>

        <menuitem action="open_task_type_form" id="menu_project_config_project" name="Task Stages" parent="menu_project_config" sequence="10" groups="base.group_no_one"/>

        <menuitem action="open_view_project_all" id="menu_projects" name="Projects" parent="menu_main_pm" sequence="1"/>
        <menuitem action="open_view_project_all_group_stage" id="menu_projects_group_stage" name="Projects" parent="menu_main_pm" sequence="1" groups="project.group_project_stages"/>
        <menuitem action="open_view_project_all_config" id="menu_projects_config" name="Projects" parent="menu_project_config" sequence="5"/>

        <!-- User Form -->
        <record id="act_res_users_2_project_task_opened" model="ir.actions.act_window">
            <field name="name">Assigned Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,form,calendar,graph</field>
            <field name="context">{'default_user_ids': [(6, 0, [active_id])]}</field>
            <field name="domain">[('display_project_id', '!=', False), ('user_ids', 'in', [active_id])]</field>
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
                <tree string="Tags" editable="top" sample="1">
                    <field name="name"/>
                    <field name="color" widget="color_picker" optional="show"/>
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
                  Use tags to categorize your tasks.
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

        <record id="project_view_kanban_inherit_project" model="ir.ui.view">
            <field name="name">project.kanban.inherit.project</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project_kanban"/>
            <field name="priority">200</field>
            <field name="arch" type="xml">
                <xpath expr="/kanban" position="inside">
                    <field name="id"/>
                </xpath>
                <xpath expr="//div[hasclass('o_kanban_manage_view')]" position="inside">
                    <div role="menuitem" groups="project.group_project_manager">
                        <a name="%(project.project_update_all_action)d" type="action" t-attf-context="{'active_id': #{record.id.raw_value}}">Project Updates</a>
                    </div>
                </xpath>
            </field>
        </record>
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
            <xpath expr="//filter[@name='responsible']" position="replace"></xpath>
            <xpath expr="/search" position="inside">
                <filter string="Last 30 Days" name="rating_last_30_days" invisible="1" domain="[
                    ('write_date', '>=', (datetime.datetime.combine(context_today() + relativedelta(days=-30), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S'))]"
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
        <field name="context">{'search_default_last_month': 1, 'search_default_responsible': 1}</field>
    </record>

    <record id="rating_rating_action_project_report_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="rating_rating_action_project_report"/>
        <field name="view_id" ref="rating.rating_rating_view_kanban"/>
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
                            <div class="col-12 col-lg-6 o_setting_box" id="task_dependencies_setting">
                                <div class="o_setting_left_pane">
                                    <field name="group_project_task_dependencies"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_project_task_dependencies"/>
                                    <div class="text-muted">
                                        Determine the order in which to perform tasks
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="project_stages">
                                <div class="o_setting_left_pane">
                                    <field name="group_project_stages"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_project_stages"/>
                                    <div class="text-muted">
                                        Track the progress of your projects
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('group_project_stages', '=', False)]}">
                                        <div class="mt8">
                                            <button name="%(project.project_project_stage_configure)d" icon="fa-arrow-right" type="action" string="Configure Stages" class="btn-link"/>
                                        </div>
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
                        <h2>Analytics</h2>
                        <div class="row mt16 o_settings_container" name="analytic">
                            <div class="col-12 col-lg-6 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="group_analytic_accounting"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_analytic_accounting" string="Profitability"/>
                                    <div class="text-muted">
                                        Track the costs and revenues linked to your projects
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
        return self.env["project.project"]._action_open_all_projects()

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
                        <button string="Archive project" type="object" name="action_archive" class="btn btn-primary" attrs="{'invisible': [('projects_archived', '=', True)]}" data-hotkey="q"/>
                        <button string="Discard" special="cancel" data-hotkey="z" />
                    </footer>
                    <footer attrs="{'invisible': [('task_count', '>', 0)]}">
                        <button string="Ok" type="object" name="confirm_delete" class="btn btn-primary" data-hotkey="q"/>
                        <button string="Cancel" special="cancel" data-hotkey="z"/>
                    </footer>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\project_share_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProjectShareWizard(models.TransientModel):
    _name = 'project.share.wizard'
    _inherit = 'portal.share'
    _description = 'Project Sharing'

    @api.model
    def default_get(self, fields):
        result = super().default_get(fields)
        if not result.get('access_mode'):
            result.update(
                access_mode='read',
                display_access_mode=True,
            )
        return result

    @api.model
    def _selection_target_model(self):
        project_model = self.env['ir.model']._get('project.project')
        return [(project_model.model, project_model.name)]

    access_mode = fields.Selection([('read', 'Readonly'), ('edit', 'Edit')])
    display_access_mode = fields.Boolean()

    @api.depends('res_model', 'res_id')
    def _compute_resource_ref(self):
        for wizard in self:
            if wizard.res_model and wizard.res_model == 'project.project':
                wizard.resource_ref = '%s,%s' % (wizard.res_model, wizard.res_id or 0)
            else:
                wizard.resource_ref = None

    def action_send_mail(self):
        self.ensure_one()
        if self.access_mode == 'edit':
            portal_partners = self.partner_ids.filtered('user_ids')
            note = self._get_note()
            self.resource_ref._add_collaborators(self.partner_ids)
            self._send_public_link(note, portal_partners)
            self._send_signup_link(note, partners=self.partner_ids - portal_partners)
            self.resource_ref.message_subscribe(partner_ids=self.partner_ids.ids)
            return {'type': 'ir.actions.act_window_close'}
        return super().action_send_mail()

```

## File: wizard\project_share_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_share_wizard_view_form" model="ir.ui.view">
        <field name="name">project.share.wizard.view.form</field>
        <field name="model">project.share.wizard</field>
        <field name="arch" type="xml">
            <form string="Share Project">
                <field name="res_model" invisible="1"/>
                <field name="res_id" invisible="1"/>
                <field name="display_access_mode" invisible="1" />
                <p class="alert alert-warning" attrs="{'invisible': [('access_warning', '=', '')]}" role="alert"><field name="access_warning"/></p>
                <group attrs="{'invisible': [('display_access_mode', '=', False)]}">
                    <field class="flex-row" name="access_mode" widget="radio"/>
                </group>
                <group name="share_link" attrs="{'invisible': [('access_mode', '=', 'edit')]}">
                    <field name="share_link" widget="CopyClipboardChar" options="{'string': 'Copy Link'}"/>
                </group>
                <group>
                    <div class="o_td_label">
                        <label for="partner_ids" string="Invite People" attrs="{'invisible': [('access_mode', '=', 'read')]}"/>
                        <label for="partner_ids" attrs="{'invisible': [('access_mode', '=', 'edit')]}"/>
                    </div>
                    <field name="partner_ids" widget="many2many_tags_email" placeholder="Add contacts to share the project..." nolabel="1"/>
                </group>
                <group>
                    <field name="note" placeholder="Add a note"/>
                </group>
                <footer>
                    <button string="Send" name="action_send_mail" attrs="{'invisible': [('access_warning', '!=', '')]}" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-default" special="cancel" data-hotkey="z" />
                </footer>
            </form>
        </field>
    </record>

    <record id="project_share_wizard_action" model="ir.actions.act_window">
        <field name="name">Share Project</field>
        <field name="res_model">project.share.wizard</field>
        <field name="binding_model_id" ref="model_project_project"/>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>

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
    tasks_count = fields.Integer('Number of Tasks', compute='_compute_tasks_count')
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
                'pivot_row_groupby': ['user_ids'],
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
                    <button string="Archive Stages" type="object" name="action_archive" class="btn btn-primary" attrs="{'invisible': ['|', ('stages_active', '=', False), ('tasks_count', '=', 0)]}" data-hotkey="q"/>
                    <button string="Delete" type="object" name="action_unlink" class="btn btn-primary" attrs="{'invisible': [('tasks_count', '>', 0)]}" data-hotkey="w"/>
                    <button string="Discard" special="cancel" data-hotkey="z" />
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
                    <button string="Confirm" type="object" name="action_confirm" class="btn btn-primary" data-hotkey="q"/>
                    <button string="Discard" special="cancel" data-hotkey="z" />
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
from . import project_share_wizard

```

