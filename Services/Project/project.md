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
from odoo.tools.sql import create_index


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

    # Index to improve the performance of burndown chart.
    project_task_stage_field_id = env['ir.model.fields']._get_ids('project.task').get('stage_id')
    create_index(
        cr,
        'mail_tracking_value_mail_message_id_old_value_integer_task_stage',
        env['mail.tracking.value']._table,
        ['mail_message_id', 'old_value_integer'],
        where=f'field={project_task_stage_field_id}'
    )

def _project_uninstall_hook(cr, registry):
    """Since the m2m table for the project share wizard's `partner_ids` field is not dropped at uninstall, it is
    necessary to ensure it is emptied, else re-installing the module will fail due to foreign keys constraints."""
    env = api.Environment(cr, SUPERUSER_ID, {})
    env['project.share.wizard'].search([("partner_ids", "!=", False)]).partner_ids = False

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Project',
    'version': '1.3',
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
    'data': [
        'security/project_security.xml',
        'security/ir.model.access.csv',
        'security/ir.model.access.xml',
        'data/digest_data.xml',
        'report/project_report_views.xml',
        'report/project_task_burndown_chart_report_views.xml',
        'views/analytic_views.xml',
        'views/digest_views.xml',
        'views/rating_rating_views.xml',
        'views/project_update_views.xml',
        'views/project_update_templates.xml',
        'views/project_project_stage_views.xml',
        'wizard/project_share_wizard_views.xml',
        'views/project_collaborator_views.xml',
        'views/project_views.xml',
        'views/project_milestone_views.xml',
        'views/res_partner_views.xml',
        'views/res_config_settings_views.xml',
        'views/mail_activity_views.xml',
        'views/project_sharing_views.xml',
        'views/project_portal_templates.xml',
        'views/project_task_templates.xml',
        'views/project_sharing_templates.xml',
        'data/ir_cron_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_template_data.xml',
        'data/project_data.xml',
        'wizard/project_task_type_delete_views.xml',
    ],
    'demo': [
        'data/mail_template_demo.xml',
        'data/project_demo.xml',
    ],
    'installable': True,
    'application': True,
    'post_init_hook': '_project_post_init',
    'uninstall_hook': '_project_uninstall_hook',
    'assets': {
        'web.assets_backend': [
            'project/static/src/css/project.css',
            'project/static/src/utils/**/*',
            'project/static/src/services/**/*',
            'project/static/src/components/**/*',
            'project/static/src/views/**/*',
            'project/static/src/js/project_activity.js',
            'project/static/src/js/project_control_panel.js',
            'project/static/src/js/project_graph_view.js',
            'project/static/src/js/project_pivot_view.js',
            'project/static/src/js/project_rating_graph_view.js',
            'project/static/src/js/project_rating_pivot_view.js',
            'project/static/src/js/project_task_kanban_examples.js',
            'project/static/src/js/tours/project.js',
            'project/static/src/js/widgets/*',
            'project/static/src/scss/project_dashboard.scss',
            'project/static/src/scss/project_form.scss',
            'project/static/src/scss/project_widgets.scss',
            'project/static/src/xml/**/*',
        ],
        'web.assets_frontend': [
            'project/static/src/scss/portal_rating.scss',
            'project/static/src/scss/project_sharing_frontend.scss',
            'project/static/src/js/portal_rating.js',
        ],
        'web.qunit_suite_tests': [
            'project/static/src/project_sharing/components/portal_file_input/portal_file_input.js',
            'project/static/tests/**/*.js',
        ],
        'web.assets_tests': [
            'project/static/tests/tours/**/*',
        ],
        'project.webclient': [
            ('include', 'web._assets_helpers'),
            ('include', 'web._assets_backend_helpers'),

            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',

            ('include', 'web._assets_bootstrap'),

            'base/static/src/css/modules.css',

            'web/static/src/core/utils/transitions.scss',
            'web/static/src/core/**/*',
            'web/static/src/search/**/*',
            'web/static/src/webclient/icons.scss', # variables required in list_controller.scss
            'web/static/src/views/*.js',
            'web/static/src/views/*.xml',
            'web/static/src/views/*.scss',
            'web/static/src/views/fields/**/*',
            ('remove', 'web/static/src/views/fields/journal_dashboard_graph/**/*'),  # only works with graph view in assets
            'web/static/src/views/form/**/*',
            'web/static/src/views/kanban/**/*',
            'web/static/src/views/list/**/*',
            'web/static/src/views/view_button/**/*',
            'web/static/src/views/view_dialogs/**/*',
            'web/static/src/views/widgets/**/*',
            'web/static/src/webclient/**/*',
            ('remove', 'web/static/src/webclient/navbar/navbar.scss'),  # already in assets_common
            ('remove', 'web/static/src/webclient/clickbot/clickbot.js'), # lazy loaded
            ('remove', 'web/static/src/views/form/button_box/*.scss'),

            # remove the report code and whitelist only what's needed
            ('remove', 'web/static/src/webclient/actions/reports/**/*'),
            'web/static/src/webclient/actions/reports/*.js',
            'web/static/src/webclient/actions/reports/*.xml',

            'web/static/src/env.js',

            'web/static/lib/jquery.scrollTo/jquery.scrollTo.js',
            'web/static/lib/py.js/lib/py.js',
            'web/static/lib/py.js/lib/py_extras.js',
            'web/static/lib/jquery.ba-bbq/jquery.ba-bbq.js',

            'web/static/src/legacy/scss/fields.scss',
            'web/static/src/legacy/scss/views.scss',
            'web/static/src/legacy/scss/form_view.scss',
            'web/static/src/legacy/scss/list_view.scss',
            'web/static/src/legacy/scss/kanban_dashboard.scss',
            'web/static/src/legacy/scss/kanban_examples_dialog.scss',
            'web/static/src/legacy/scss/kanban_column_progressbar.scss',
            'web/static/src/legacy/scss/kanban_view.scss',

            'base/static/src/scss/res_partner.scss',

            # Form style should be computed before
            'web/static/src/views/form/button_box/*.scss',

            'web/static/src/legacy/action_adapters.js',
            'web/static/src/legacy/debug_manager.js',
            'web/static/src/legacy/legacy_service_provider.js',
            'web/static/src/legacy/legacy_client_actions.js',
            'web/static/src/legacy/legacy_dialog.js',
            'web/static/src/legacy/legacy_load_views.js',
            'web/static/src/legacy/legacy_promise_error_handler.js',
            'web/static/src/legacy/legacy_rpc_error_handler.js',
            'web/static/src/legacy/root_widget.js',
            'web/static/src/legacy/legacy_setup.js',
            'web/static/src/legacy/root_widget.js',
            'web/static/src/legacy/backend_utils.js',
            'web/static/src/legacy/utils.js',
            'web/static/src/legacy/web_client.js',
            'web/static/src/legacy/js/_deprecated/data.js',
            'web/static/src/legacy/js/chrome/*',
            'web/static/src/legacy/js/components/*',
            'web/static/src/legacy/js/control_panel/*',
            'web/static/src/legacy/js/core/domain.js',
            'web/static/src/legacy/js/core/mvc.js',
            'web/static/src/legacy/js/core/py_utils.js',
            'web/static/src/legacy/js/core/context.js',
            'web/static/src/legacy/js/core/misc.js',
            'web/static/src/legacy/js/fields/abstract_field.js',
            'web/static/src/legacy/js/fields/abstract_field_owl.js',
            'web/static/src/legacy/js/_deprecated/basic_fields.js',
            'web/static/src/legacy/js/fields/basic_fields.js',
            'web/static/src/legacy/js/fields/basic_fields_owl.js',
            'web/static/src/legacy/js/fields/field_utils.js',
            'web/static/src/legacy/js/fields/relational_fields.js',
            'web/static/src/legacy/js/fields/special_fields.js',
            'web/static/src/legacy/js/fields/field_registry.js',
            'web/static/src/legacy/js/fields/field_registry_owl.js',
            'web/static/src/legacy/js/fields/field_utils.js',
            'web/static/src/legacy/js/fields/field_wrapper.js',
            'web/static/src/legacy/js/views/sample_server.js',
            'web/static/src/legacy/js/views/abstract_model.js',
            'web/static/src/legacy/js/views/basic/basic_model.js',
            'web/static/src/legacy/js/views/action_model.js',
            'web/static/src/legacy/js/views/view_utils.js',
            'web/static/src/legacy/js/services/data_manager.js',
            'web/static/src/legacy/js/services/session.js',
            'web/static/src/legacy/js/tools/tools.js',
            'web/static/src/legacy/js/views/**/*',
            'web/static/src/legacy/js/widgets/data_export.js',
            'web/static/src/legacy/js/widgets/date_picker.js',
            'web/static/src/legacy/js/widgets/domain_selector_dialog.js',
            'web/static/src/legacy/js/widgets/domain_selector.js',
            'web/static/src/legacy/js/widgets/model_field_selector.js',
            'web/static/src/legacy/js/widgets/model_field_selector_popover.js',
            'web/static/src/legacy/js/env.js',
            'web/static/src/legacy/js/model.js',
            'web/static/src/legacy/js/owl_compatibility.js',

            'web_editor/static/src/components/**/*',
            'web_editor/static/src/scss/web_editor.common.scss',
            'web_editor/static/src/scss/web_editor.backend.scss',

            'web_editor/static/src/js/wysiwyg/dialog.js',
            'web_editor/static/src/js/frontend/loader.js',
            'web_editor/static/src/js/backend/**/*',
            'web_editor/static/src/xml/backend.xml',

            'mail/static/src/scss/variables/*.scss',
            'mail/static/src/widgets/**/*.scss',

            'project/static/src/components/project_task_name_with_subtask_count_char_field/*',
            'project/static/src/views/project_task_form/*.scss',

            'project/static/src/project_sharing/search/favorite_menu/custom_favorite_item.xml',
            'project/static/src/project_sharing/**/*',
            'web/static/src/start.js',
            'web/static/src/legacy/legacy_setup.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
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
        # default filter by value
        domain = [('project_id', '=', project.id)]
        # pager
        url = "/my/projects/%s" % project.id
        values = self._prepare_tasks_values(page, date_begin, date_end, sortby, search, search_in, groupby, url, domain, su=bool(access_token))
        # adding the access_token to the pager's url args,
        # so we are not prompted for loging when switching pages
        # if access_token is None, the arg is not present in the URL
        values['pager']['url_args']['access_token'] = access_token
        pager = portal_pager(**values['pager'])

        values.update(
            grouped_tasks=values['grouped_tasks'](pager['offset']),
            page_name='project',
            pager=pager,
            project=project,
            task_url=f'projects/{project.id}/task',
        )
        # default value is set to 'project' in _prepare_tasks_values, so we have to set it to 'none' here.
        if not groupby:
            values['groupby'] = 'none'

        return self._get_page_view_values(project, access_token, values, 'my_projects_history', False, **kwargs)

    def _prepare_project_domain(self):
        return []

    def _prepare_searchbar_sortings(self):
        return {
            'date': {'label': _('Newest'), 'order': 'create_date desc'},
            'name': {'label': _('Name'), 'order': 'name'},
        }

    @http.route(['/my/projects', '/my/projects/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_projects(self, page=1, date_begin=None, date_end=None, sortby=None, **kw):
        values = self._prepare_portal_layout_values()
        Project = request.env['project.project']
        domain = self._prepare_project_domain()

        searchbar_sortings = self._prepare_searchbar_sortings()
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

    @http.route(['/my/project/<int:project_id>',
                 '/my/project/<int:project_id>/page/<int:page>',
                 '/my/project/<int:project_id>/task/<int:task_id>',
                 '/my/project/<int:project_id>/project_sharing'], type='http', auth="public")
    def portal_project_routes_outdated(self, **kwargs):
        """ Redirect the outdated routes to the new routes. """
        return request.redirect(request.httprequest.full_path.replace('/my/project/', '/my/projects/'))

    @http.route(['/my/task',
                 '/my/task/page/<int:page>',
                 '/my/task/<int:task_id>'], type='http', auth='public')
    def portal_my_task_routes_outdated(self, **kwargs):
        """ Redirect the outdated routes to the new routes. """
        return request.redirect(request.httprequest.full_path.replace('/my/task', '/my/tasks'))

    @http.route(['/my/projects/<int:project_id>', '/my/projects/<int:project_id>/page/<int:page>'], type='http', auth="public", website=True)
    def portal_my_project(self, project_id=None, access_token=None, page=1, date_begin=None, date_end=None, sortby=None, search=None, search_in='content', groupby=None, task_id=None, **kw):
        try:
            project_sudo = self._document_check_access('project.project', project_id, access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')
        if project_sudo.collaborator_count and project_sudo.with_user(request.env.user)._check_project_sharing_access():
            values = {'project_id': project_id}
            if task_id:
                values['task_id'] = task_id
            return request.render("project.project_sharing_portal", values)
        project_sudo = project_sudo if access_token else project_sudo.with_user(request.env.user)
        values = self._project_get_page_view_values(project_sudo, access_token, page, date_begin, date_end, sortby, search, search_in, groupby, **kw)
        return request.render("project.portal_my_project", values)

    def _prepare_project_sharing_session_info(self, project, task=None):
        session_info = request.env['ir.http'].session_info()
        user_context = dict(request.env.context) if request.session.uid else {}
        mods = conf.server_wide_modules or []
        if request.env.lang:
            lang = request.env.lang
            session_info['user_context']['lang'] = lang
            # Update Cache
            user_context['lang'] = lang
        lang = user_context.get("lang")
        translation_hash = request.env['ir.http'].get_web_translations_hash(mods, lang)
        cache_hashes = {
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
        if task:
            session_info['open_task_action'] = task.action_project_sharing_open_task()
        return session_info

    @http.route("/my/projects/<int:project_id>/project_sharing", type="http", auth="user", methods=['GET'])
    def render_project_backend_view(self, project_id, task_id=None):
        project = request.env['project.project'].sudo().browse(project_id)
        if not project.exists() or not project.with_user(request.env.user)._check_project_sharing_access():
            return request.not_found()
        task = task_id and request.env['project.task'].browse(int(task_id))
        return request.render(
            'project.project_sharing_embed',
            {'session_info': self._prepare_project_sharing_session_info(project, task)},
        )

    @http.route('/my/projects/<int:project_id>/task/<int:task_id>', type='http', auth='public', website=True)
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

    @http.route('/my/projects/<int:project_id>/task/<int:task_id>/subtasks', type='http', auth='user', methods=['GET'], website=True)
    def portal_my_project_subtasks(self, project_id, task_id, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, search=None, search_in='content', groupby=None, **kw):
        try:
            project_sudo = self._document_check_access('project.project', project_id)
            task_sudo = request.env['project.task'].search([('project_id', '=', project_id), ('id', '=', task_id)]).sudo()
            task_domain = [('id', 'child_of', task_id), ('id', '!=', task_id)]
            searchbar_filters = self._get_my_tasks_searchbar_filters([('id', '=', task_sudo.project_id.id)], task_domain)

            if not filterby:
                filterby = 'all'
            domain = searchbar_filters.get(filterby, searchbar_filters.get('all'))['domain']

            values = self._prepare_tasks_values(page, date_begin, date_end, sortby, search, search_in, groupby, url=f'/my/projects/{project_id}/task/{task_id}/subtasks', domain=AND([task_domain, domain]))
            values['page_name'] = 'project_subtasks'

            # pager
            pager_vals = values['pager']
            pager_vals['url_args'].update(filterby=filterby)
            pager = portal_pager(**pager_vals)

            values.update({
                'project': project_sudo,
                'task': task_sudo,
                'grouped_tasks': values['grouped_tasks'](pager['offset']),
                'pager': pager,
                'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
                'filterby': filterby,
            })
            return request.render("project.portal_my_tasks", values)
        except (AccessError, MissingError):
            return request.not_found()

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
            'task_link_section': [],
        }

        values = self._get_page_view_values(task, access_token, values, history, False, **kwargs)
        if project:
            values['project_id'] = project.id
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

    def _task_get_searchbar_sortings(self, milestones_allowed):
        values = {
            'date': {'label': _('Newest'), 'order': 'create_date desc', 'sequence': 1},
            'name': {'label': _('Title'), 'order': 'name', 'sequence': 2},
            'project': {'label': _('Project'), 'order': 'project_id, stage_id', 'sequence': 3},
            'stage': {'label': _('Stage'), 'order': 'stage_id, project_id', 'sequence': 5},
            'status': {'label': _('Status'), 'order': 'kanban_state', 'sequence': 6},
            'priority': {'label': _('Priority'), 'order': 'priority desc', 'sequence': 8},
            'date_deadline': {'label': _('Deadline'), 'order': 'date_deadline asc', 'sequence': 9},
            'update': {'label': _('Last Stage Update'), 'order': 'date_last_stage_update desc', 'sequence': 11},
        }
        if milestones_allowed:
            values['milestone'] = {'label': _('Milestone'), 'order': 'milestone_id', 'sequence': 7}
        return values

    def _task_get_searchbar_groupby(self, milestones_allowed):
        values = {
            'none': {'input': 'none', 'label': _('None'), 'order': 1},
            'project': {'input': 'project', 'label': _('Project'), 'order': 2},
            'stage': {'input': 'stage', 'label': _('Stage'), 'order': 4},
            'status': {'input': 'status', 'label': _('Status'), 'order': 5},
            'priority': {'input': 'priority', 'label': _('Priority'), 'order': 7},
            'customer': {'input': 'customer', 'label': _('Customer'), 'order': 10},
        }
        if milestones_allowed:
            values['milestone'] = {'input': 'milestone', 'label': _('Milestone'), 'order': 6}
        return dict(sorted(values.items(), key=lambda item: item[1]["order"]))

    def _task_get_groupby_mapping(self):
        return {
            'project': 'project_id',
            'stage': 'stage_id',
            'customer': 'partner_id',
            'milestone': 'milestone_id',
            'priority': 'priority',
            'status': 'kanban_state',
        }

    def _task_get_order(self, order, groupby):
        groupby_mapping = self._task_get_groupby_mapping()
        field_name = groupby_mapping.get(groupby, '')
        if not field_name:
            return order
        return '%s, %s' % (field_name, order)

    def _task_get_searchbar_inputs(self, milestones_allowed):
        values = {
            'all': {'input': 'all', 'label': _('Search in All'), 'order': 1},
            'content': {'input': 'content', 'label': Markup(_('Search <span class="nolabel"> (in Content)</span>')), 'order': 1},
            'ref': {'input': 'ref', 'label': _('Search in Ref'), 'order': 1},
            'project': {'input': 'project', 'label': _('Search in Project'), 'order': 2},
            'users': {'input': 'users', 'label': _('Search in Assignees'), 'order': 3},
            'stage': {'input': 'stage', 'label': _('Search in Stages'), 'order': 4},
            'status': {'input': 'status', 'label': _('Search in Status'), 'order': 5},
            'priority': {'input': 'priority', 'label': _('Search in Priority'), 'order': 7},
            'message': {'input': 'message', 'label': _('Search in Messages'), 'order': 11},
        }
        if milestones_allowed:
            values['milestone'] = {'input': 'milestone', 'label': _('Search in Milestone'), 'order': 6}

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
        if search_in in ('milestone', 'all'):
            search_domain.append([('milestone_id', 'ilike', search)])
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

    def _prepare_tasks_values(self, page, date_begin, date_end, sortby, search, search_in, groupby, url="/my/tasks", domain=None, su=False):
        values = self._prepare_portal_layout_values()

        Task = request.env['project.task']
        milestone_domain = AND([domain, [('allow_milestones', '=', 'True')]])
        milestones_allowed = Task.sudo().search_count(milestone_domain, limit=1) == 1
        searchbar_sortings = dict(sorted(self._task_get_searchbar_sortings(milestones_allowed).items(),
                                         key=lambda item: item[1]["sequence"]))
        searchbar_inputs = self._task_get_searchbar_inputs(milestones_allowed)
        searchbar_groupby = self._task_get_searchbar_groupby(milestones_allowed)

        if not domain:
            domain = []
        if not su and Task.check_access_rights('read'):
            domain = AND([domain, request.env['ir.rule']._compute_domain(Task._name, 'read')])
        Task_sudo = Task.sudo()

        # default sort by value
        if not sortby or sortby not in searchbar_sortings or (sortby == 'milestone' and not milestones_allowed):
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']

        # default group by value
        if not groupby or (groupby == 'milestone' and not milestones_allowed):
            groupby = 'project'

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        # search reset if needed
        if not milestones_allowed and search_in == 'milestone':
            search_in = 'all'
        # search
        if search and search_in:
            domain += self._task_get_search_domain(search_in, search)

        # content according to pager and archive selected
        order = self._task_get_order(order, groupby)

        def get_grouped_tasks(pager_offset):
            tasks = Task_sudo.search(domain, order=order, limit=self._items_per_page, offset=pager_offset)
            request.session['my_project_tasks_history' if url.startswith('/my/projects') else 'my_tasks_history'] = tasks.ids[:100]

            tasks_project_allow_milestone = tasks.filtered(lambda t: t.allow_milestones)
            tasks_no_milestone = tasks - tasks_project_allow_milestone

            groupby_mapping = self._task_get_groupby_mapping()
            group = groupby_mapping.get(groupby)
            if group:
                if group == 'milestone_id':
                    grouped_tasks = [Task_sudo.concat(*g) for k, g in groupbyelem(tasks_project_allow_milestone, itemgetter(group))]

                    if not grouped_tasks:
                        if tasks_no_milestone:
                            grouped_tasks = [tasks_no_milestone]
                    else:
                        if grouped_tasks[len(grouped_tasks) - 1][0].milestone_id and tasks_no_milestone:
                            grouped_tasks.append(tasks_no_milestone)
                        else:
                            grouped_tasks[len(grouped_tasks) - 1] |= tasks_no_milestone

                else:
                    grouped_tasks = [Task_sudo.concat(*g) for k, g in groupbyelem(tasks, itemgetter(group))]
            else:
                grouped_tasks = [tasks] if tasks else []

            task_states = dict(Task_sudo._fields['kanban_state']._description_selection(request.env))
            if sortby == 'status':
                if groupby == 'none' and grouped_tasks:
                    grouped_tasks[0] = grouped_tasks[0].sorted(lambda tasks: task_states.get(tasks.kanban_state))
                else:
                    grouped_tasks.sort(key=lambda tasks: task_states.get(tasks[0].kanban_state))
            return grouped_tasks

        values.update({
            'date': date_begin,
            'date_end': date_end,
            'grouped_tasks': get_grouped_tasks,
            'allow_milestone': milestones_allowed,
            'page_name': 'task',
            'default_url': url,
            'task_url': 'tasks',
            'pager': {
                "url": url,
                "url_args": {'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby, 'groupby': groupby, 'search_in': search_in, 'search': search},
                "total": Task_sudo.search_count(domain),
                "page": page,
                "step": self._items_per_page
            },
            'searchbar_sortings': searchbar_sortings,
            'searchbar_groupby': searchbar_groupby,
            'searchbar_inputs': searchbar_inputs,
            'search_in': search_in,
            'search': search,
            'sortby': sortby,
            'groupby': groupby,
        })
        return values

    def _get_my_tasks_searchbar_filters(self, project_domain=None, task_domain=None):
        searchbar_filters = {
            'all': {'label': _('All'), 'domain': [('project_id', '!=', False)]},
        }

        # extends filterby criteria with project the customer has access to
        projects = request.env['project.project'].search(project_domain or [])
        for project in projects:
            searchbar_filters.update({
                str(project.id): {'label': project.name, 'domain': [('project_id', '=', project.id)]}
            })

        # extends filterby criteria with project (criteria name is the project id)
        # Note: portal users can't view projects they don't follow
        project_groups = request.env['project.task'].read_group(AND([[('project_id', 'not in', projects.ids)], task_domain or []]),
                                                                ['project_id'], ['project_id'])
        for group in project_groups:
            proj_id = group['project_id'][0] if group['project_id'] else False
            proj_name = group['project_id'][1] if group['project_id'] else _('Others')
            searchbar_filters.update({
                str(proj_id): {'label': proj_name, 'domain': [('project_id', '=', proj_id)]}
            })
        return searchbar_filters

    @http.route(['/my/tasks', '/my/tasks/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_tasks(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, search=None, search_in='content', groupby=None, **kw):
        searchbar_filters = self._get_my_tasks_searchbar_filters()

        if not filterby:
            filterby = 'all'
        domain = searchbar_filters.get(filterby, searchbar_filters.get('all'))['domain']

        values = self._prepare_tasks_values(page, date_begin, date_end, sortby, search, search_in, groupby, domain=domain)

        # pager
        pager_vals = values['pager']
        pager_vals['url_args'].update(filterby=filterby)
        pager = portal_pager(**pager_vals)

        values.update({
            'grouped_tasks': values['grouped_tasks'](pager['offset']),
            'pager': pager,
            'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
            'filterby': filterby,
        })
        return request.render("project.portal_my_tasks", values)

    def _show_task_report(self, task_sudo, report_type, download):
        # This method is to be overriden to report timesheets if the module is installed.
        # The route should not be called if at least hr_timesheet is not installed
        raise MissingError(_('There is nothing to report.'))

    @http.route(['/my/tasks/<int:task_id>'], type='http', auth="public", website=True)
    def portal_my_task(self, task_id, report_type=None, access_token=None, project_sharing=False, **kw):
        try:
            task_sudo = self._document_check_access('project.task', task_id, access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        if report_type in ('pdf', 'html', 'text'):
            return self._show_task_report(task_sudo, report_type, download=kw.get('download'))

        # ensure attachment are accessible with access token inside template
        for attachment in task_sudo.attachment_ids:
            attachment.generate_access_token()
        if project_sharing is True:
            # Then the user arrives to the stat button shown in form view of project.task and the portal user can see only 1 task
            # so the history should be reset.
            request.session['my_tasks_history'] = task_sudo.ids
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
            if token is not None:
                kw['token'] = token # Update token (either string which contains token value or False)
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
    <img src="https://download.odoocdn.com/digests/project/static/src/img/project-custom-tasks.gif" class="illustration_border" />
</div>
            </field>
        </record>

        <record id="digest_tip_project_1" model="digest.tip">
            <field name="name">Tip: Create tasks from incoming emails</field>
            <field name="sequence">1300</field>
            <field name="group_id" ref="project.group_project_user"/>
            <field name="tip_description" type="html">
<div>
    <t t-set="project_record" t-value="object.env['project.project'].search([('alias_name', '!=', False)], limit=1, order='sequence asc')"/>
    <p class="tip_title">Tip: Create tasks from incoming emails</p>
    <t t-if="project_record and project_record.alias_domain">
        <p class="tip_content">Emails sent to <a t-attf-href="mailto:{{project_record.alias_value}}" target="_blank" style="color: #875a7b; text-decoration: none;"><t t-out="project_record.alias_value" /></a> will generate tasks in your <t t-out="project_record.name"></t> project.</p>
    </t>
    <t t-else="">
        <p class="tip_content">Create tasks by sending an email to the email address of your project.</p>
    </t>
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

## File: data\mail_message_subtype_data.xml

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
    <record id="mt_task_stage" model="mail.message.subtype">
        <field name="name">Stage Changed</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="description">Stage changed</field>
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
    <record id="mt_task_progress" model="mail.message.subtype">
        <field name="name">Task in Progress</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
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
    <!-- Update-related subtypes for messaging / Chatter -->
    <record id="mt_update_create" model="mail.message.subtype">
        <field name="name">Update Created</field>
        <field name="res_model">project.update</field>
        <field name="default" eval="False"/>
        <field name="description">Update Created</field>
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
    <record id="mt_project_update_create" model="mail.message.subtype">
        <field name="name">Update Created</field>
        <field name="sequence">16</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_update_create"/>
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
            <field name="name">Project: Request Acknowledgment</field>
            <field name="model_id" ref="project.model_project_task"/>
            <field name="subject">Reception of {{ object.name }}</field>
            <field name="use_default_to" eval="True"/>
            <field name="description">Set this template on a project's stage to automate email when tasks reach stages</field>
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
            <field name="name">Project: Task Rating Request</field>
            <field name="model_id" ref="project.model_project_task"/>
            <field name="subject">{{ object.project_id.company_id.name }}: Satisfaction Survey</field>
            <field name="email_from">{{ (object._rating_get_operator().email_formatted if object._rating_get_operator() else user.email_formatted) }}</field>
            <field name="partner_to" >{{ object._rating_get_partner().id }}</field>
            <field name="description">Set this template on a project stage to request feedback from your customers. Enable the "customer ratings" feature on the project</field>
            <field name="body_html" type="html">
<div>
    <t t-set="access_token" t-value="object._rating_get_access_token()"/>
    <t t-set="partner" t-value="object._rating_get_partner()"/>
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
            <t t-if="object._rating_get_operator().name">
                assigned to <strong t-out="object._rating_get_operator().name or ''">Mitchell Admin</strong>.<br/>
            </t>
            <t t-else="">
                .<br/>
            </t>
        </td></tr>
        <tr><td style="text-align: center;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" summary="o_mail_notification" style="width:100%; margin: 32px 0px 32px 0px;">
                <tr><td style="font-size: 13px;">
                    <strong>Tell us how you feel about our service</strong><br/>
                    <span style="font-size: 12px; opacity: 0.5; color: #454748;">(click on one of these smileys)</span>
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
                <br/><br/><span style="margin: 0px 0px 0px 0px; font-size: 12px; opacity: 0.5; color: #454748;">This customer survey has been sent because your task has been moved to the stage <b t-out="object.stage_id.name or ''">In progress</b></span>
            </t>
            <t t-if="object.project_id.rating_status == 'periodic'">
                <br/><span style="margin: 0px 0px 0px 0px; font-size: 12px; opacity: 0.5; color: #454748;">This customer survey is sent <b t-out="object.project_id.rating_status_period or ''">Weekly</b> as long as the task is in the <b t-out="object.stage_id.name or ''">In progress</b> stage.</span>
            </t>
        </td></tr>
    </tbody>
    </table>
</div>
            </field>
            <field name="lang">{{ object._rating_get_partner().lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- You have been assigned email -->
        <template id="project_message_user_assigned">
    <span>Dear <t t-esc="assignee_name"/>,</span>
    <br/><br/>
    <span style="margin-top: 8px;">You have been assigned to the <t t-esc="model_description or 'document'"/> <t t-esc="object.display_name"/>.</span>
    <br/>
        </template>
    </data>
</odoo>

```

## File: data\mail_template_demo.xml

```xml
<odoo>
    <data>        
        <record id="project_done_email_template" model="mail.template">
            <field name="name">Project: Project Completed</field>
            <field name="model_id" ref="project.model_project_project"/>
            <field name="subject">Project status - {{ object.name }}</field>
            <field name="email_from">{{ (object.partner_id.email_formatted if object.partner_id else user.email_formatted) }}</field>
            <field name="partner_to" >{{ object.partner_id.id }}</field>
            <field name="description">Set on project's stages to inform customers when a project reaches that stage</field>
            <field name="body_html" type="html">
<div>
    Dear <t t-out="object.partner_id.name or 'customer'">Brandon Freeman</t>,<br/>
    It is my pleasure to let you know that we have successfully completed the project "<strong t-out="object.name or ''">Renovations</strong>".
    <t t-if="user.signature">
        <br />
        <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
    </t>
</div>
<br/><span style="margin: 0px 0px 0px 0px; font-size: 12px; opacity: 0.5; color: #454748;" groups="project.group_project_stages">You are receiving this email because your project has been moved to the stage <b t-out="object.stage_id.name or ''">Done</b></span>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
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
            <field name="groups_id" eval="[Command.link(ref('group_project_user'))]"/>
        </record>

        <!-- Groups : Add milestones feature by default -->
        <record id="base.group_user" model="res.groups">
            <field name="implied_ids" eval="[Command.link(ref('group_project_milestone'))]"/>
        </record>
        <!-- The feature is also enabled to portal user because the feature is displayed in the Project Sharing feature -->
        <record id="base.group_portal" model="res.groups">
            <field name="implied_ids" eval="[Command.link(ref('group_project_milestone'))]"/>
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

        <!-- Analytic Accounts -->
        <!-- Needed so that we can have the same analytic accounts on hr_timesheet and project_account_budget -->
        <record id="analytic_office_design" model="account.analytic.account">
            <field name="name">Office Design</field>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>

        <record id="analytic_research_development" model="account.analytic.account">
            <field name="name">Research &amp; Development</field>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>

        <record id="analytic_renovations" model="account.analytic.account">
            <field name="name">Renovations</field>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>

        <!-- Stage templates -->
        <record id="project.project_project_stage_2" model="project.project.stage">
            <field name="mail_template_id" ref="project.project_done_email_template"/>
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
        </record>
        <record id="project_stage_3" model="project.task.type">
            <field name="sequence">30</field>
            <field name="name">Canceled</field>
            <field name="legend_done">Ready to reopen</field>
            <field name="fold" eval="True"/>
        </record>

        <record id="project_project_1" model="project.project">
            <field name="date_start" eval="DateTime.today() - relativedelta(weeks=9)"/>
            <field name="date" eval="DateTime.today() + relativedelta(weekday=4,weeks=1)"/>
            <field name="name">Office Design</field>
            <field name="color">3</field>
            <field name="user_id" ref="base.user_demo"/>
            <field name="type_ids" eval="[Command.link(ref('project_stage_0')), Command.link(ref('project_stage_1')), Command.link(ref('project_stage_2')), Command.link(ref('project_stage_3'))]"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field name="privacy_visibility">portal</field>
            <field name="favorite_user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="tag_ids" eval="[Command.link(ref('project.project_tags_05'))]"/>
            <field name="stage_id" ref="project.project_project_stage_1"/>
            <field name="analytic_account_id" ref="project.analytic_office_design"/>
        </record>
        <record id="project_1_follower_admin" model="mail.followers">
            <field name="res_model">project.project</field>
            <field name="res_id" ref="project_project_1"/>
            <field name="partner_id" ref="base.partner_admin"/>
        </record>

        <record id="project_project_2" model="project.project">
            <field name="name">Research &amp; Development</field>
            <field name="privacy_visibility">followers</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="type_ids" eval="[Command.link(ref('project_stage_0')), Command.link(ref('project_stage_1')), Command.link(ref('project_stage_2')), Command.link(ref('project_stage_3'))]"/>
            <field name="favorite_user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="tag_ids" eval="[Command.link(ref('project.project_tags_04'))]"/>
            <field name="stage_id" ref="project.project_project_stage_1"/>
            <field name="analytic_account_id" ref="project.analytic_research_development"/>
        </record>
        <record id="project_2_activity_1" model="mail.activity">
            <field name="res_id" ref="project_project_2"/>
            <field name="res_model_id" ref="project.model_project_project"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_meeting"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=13)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Examine project status</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>

        <record id="project_project_3" model="project.project">
            <field name="date_start" eval="(DateTime.today() + relativedelta(months=-2)).strftime('%Y-%m-%d 10:00:00')"/>
            <field name="date" eval="(DateTime.today() + relativedelta(days=-5)).strftime('%Y-%m-%d 17:00:00')"/>
            <field name="name">Renovations</field>
            <field name="description">Renovation work at the YourCompany headquarters.</field>
            <field name="color">4</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="type_ids" eval="[Command.link(ref('project_stage_0')), Command.link(ref('project_stage_1')), Command.link(ref('project_stage_2')), Command.link(ref('project_stage_3'))]"/>
            <field name="favorite_user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="tag_ids" eval="[Command.link(ref('project_tags_04')), Command.link(ref('project_tags_02'))]"/>
            <field name="stage_id" ref="project.project_project_stage_2"/>
            <field name="analytic_account_id" ref="project.analytic_renovations"/>
            <field name="privacy_visibility">employees</field>
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

        <!-- Project 1 Milestones -->
        <record id="project_1_milestone_1" model="project.milestone">
            <field name="is_reached" eval="True"/>
            <field name="deadline" eval="time.strftime('%Y-%m-10')"/>
            <field name="name">First Phase</field>
            <field name="reached_date" eval="time.strftime('%Y-%m-10')"/>
            <field name="project_id" ref="project.project_project_1"/>
        </record>
        <record id="project_1_milestone_2" model="project.milestone">
            <field name="is_reached" eval="False"/>
            <field name="deadline" eval="(DateTime.now() + relativedelta(years=1)).strftime('%Y-%m-15')"/>
            <field name="name">Second Phase</field>
            <field name="project_id" ref="project.project_project_1"/>
        </record>
        <record id="project_1_milestone_3" model="project.milestone">
            <field name="is_reached" eval="False"/>
            <field name="deadline" eval="(DateTime.now() + relativedelta(years=2)).strftime('%Y-%m-%d')"/>
            <field name="name">Final Phase</field>
            <field name="project_id" ref="project.project_project_1"/>
        </record>

        <!-- Project 1 Tasks -->
        <record id="project_1_task_1" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">20.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Office planning</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">7</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="milestone_id" ref="project.project_1_milestone_1" />
        </record>

        <record id="project_1_task_2" model="project.task">
            <field name="planned_hours" eval="32.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Lunch Room: kitchen</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="milestone_id" ref="project.project_1_milestone_1" />
        </record>
        <record id="project_1_task_2_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_2"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=2)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_2_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_2"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=1)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_2_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_2_mail_message_1"/>
        </record>
        <record id="project_1_task_2_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_1_task_2_mail_message_2"/>
        </record>

        <record id="project_1_task_3" model="project.task">
            <field name="planned_hours" eval="10.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Noise Reduction</field>
            <field name="description">Installation of acoustic ceiling clouds and wall panels.</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-24')"/>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">4</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="milestone_id" ref="project.project_1_milestone_2" />
        </record>
        <record id="project_1_task_3_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_3"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_3_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_3"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_3_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_3_mail_message_1"/>
        </record>
        <record id="project_1_task_3_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_1_task_3_mail_message_2"/>
        </record>

        <record id="project_1_task_4" model="project.task">
            <field name="sequence">17</field>
            <field name="planned_hours">8.0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="user_ids" eval="False"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Modifications asked by the customer</field>
            <field name="description">Modifications to the kitchen of the lunch room</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_00')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="milestone_id" ref="project.project_1_milestone_2" />
        </record>
        <record id="project_1_task_4_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_4"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_4_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_4"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=3)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_4_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_4_mail_message_1"/>
        </record>
        <record id="project_1_task_4_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_1_task_4_mail_message_2"/>
        </record>

        <record id="project_1_task_5" model="project.task">
            <field name="planned_hours" eval="15.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Energy Certificate</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="kanban_state">blocked</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=2)"/>
            <field name="color">1</field>
            <field name="milestone_id" ref="project.project_1_milestone_3" />
        </record>
        <record id="project_1_task_5_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_5"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_5_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_5_mail_message_1"/>
        </record>
        <record id="project_1_task_5_activity_1" model="mail.activity">
            <field name="res_id" ref="project_1_task_5"/>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(hours=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Follow-up email</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>

        <record id="project_1_task_6" model="project.task">
            <field name="planned_hours" eval="76.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Room 1: Decoration</field>
            <field name="kanban_state">done</field>
            <field name="priority">0</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-%d')"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_01')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="color">11</field>
            <field name="milestone_id" ref="project.project_1_milestone_3" />
        </record>
        <record id="project_1_task_6_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_6"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=2)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_6_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_6_mail_message_1"/>
        </record>
        <record id="project_1_task_6_activity_1" model="mail.activity">
            <field name="res_id" ref="project_1_task_6"/>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(hours=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Call Joel Willis</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>

        <record id="project_1_task_7" model="project.task">
            <field name="planned_hours" eval="24.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Room 2: Decoration</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=6)"/>
            <field name="color">9</field>
            <field name="milestone_id" ref="project.project_1_milestone_3" />
        </record>
        <record id="project_1_task_7_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_7"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=3)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_7_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_7_mail_message_1"/>
        </record>

        <record id="project_1_task_8" model="project.task">
            <field name="planned_hours" eval="60.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Black Chairs for managers</field>
            <field name="description">Use the account_budget module</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-19')"/>
            <field name="color">5</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="kanban_state">blocked</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_01')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="milestone_id" ref="project.project_1_milestone_3" />
        </record>
        <record id="project_1_task_8_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_1_task_8"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(months=1)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_1_task_8_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_8_mail_message_1"/>
        </record>
        <record id="project_1_task_8_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_1_task_8"/>
            <field name="body">Hello Admin,
                Can we discuss this? Having nicer chairs for managers doesn't sit right with me.
            </field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="date" eval="(DateTime.now() - relativedelta(days=3)).strftime('%Y-%m-%d 09:43:27')"/>
        </record>
        <record id="project_1_task_8_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_1_task_8"/>
            <field name="parent_id" ref="project_1_task_8_message_1"/>
            <field name="body">We have already discussed, and I stand by my decision.</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="date" eval="(DateTime.now() - relativedelta(days=3)).strftime('%Y-%m-%d 11:52:03')"/>
        </record>

        <record id="project_1_task_9" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Meeting Room Furnitures</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="color">3</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="milestone_id" ref="project.project_1_milestone_3" />
        </record>
        <record id="project_1_task_9_activity_1" model="mail.activity">
            <field name="res_id" ref="project_1_task_9"/>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Check furniture</field>
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>

        <!-- Project 1 Recurring tasks and subtasks -->
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
        <record id="project_1_task_10" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">20.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Customer review</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="recurring_task" eval="True"/>
            <field name="recurrence_id" ref="project_task_recurrence_1"/>
            <field name="create_date" eval="DateTime.now() + relativedelta(weeks=-2)"/>
            <field name="milestone_id" ref="project.project_1_milestone_3" />
        </record>
        <record id="project_1_task_11" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">0.25</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_10"/>
            <field name="name">Daily stand-up meeting - Send minutes</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="recurring_task" eval="True"/>
            <field name="recurrence_id" ref="project_task_recurrence_2"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(weeks=-1)"/>
        </record>
        <record id="project_1_task_12" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">8.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_10"/>
            <field name="name">Customer Meeting</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_1_task_13" model="project.task">
            <field name="sequence">10</field>
            <field name="planned_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_12"/>
            <field name="name">Daily Meetings summary</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_1_task_14" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_12"/>
            <field name="name">Preparation</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_1_task_15" model="project.task">
            <field name="sequence">30</field>
            <field name="planned_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_12"/>
            <field name="name">Minutes</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>

        <function model="project.task.recurrence" name="_cron_create_recurring_tasks"/>

        <!-- Project 2 Tasks-->
        <record id="project_2_task_1" model="project.task">
            <field name="planned_hours">12.0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Customer analysis + Architecture</field>
            <field name="color">7</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_2_task_1_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_1"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(weeks=16)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_1_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_1_mail_message_1"/>
        </record>
        <record id="project_2_task_1_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_1"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() + relativedelta(weeks=-16, days=4)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_1_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_2_task_1_mail_message_2"/>
        </record>

        <record id="project_2_task_2" model="project.task">
            <field name="planned_hours">24.0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Basic outline</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_02')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_1'))]"/>
        </record>
        <record id="project_2_task_2_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_2"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(weeks=15)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_2_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_2_mail_message_1"/>
        </record>
        <record id="project_2_task_2_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_2"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(weeks=14)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_2_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_2_task_2_mail_message_2"/>
        </record>

        <record id="project_2_task_3" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Planning and budget</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="color">6</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_2'))]"/>
            <field name="date_deadline" eval="DateTime.now() - relativedelta(days=4)"/>
        </record>
        <record id="project_2_task_3_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_3"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() + relativedelta(weeks=-14, days=1)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_3_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_3_mail_message_1"/>
        </record>
        <record id="project_2_task_3_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_2_task_3"/>
            <field name="body">Hello Demo,
There is a change in customer requirement.
Can you check the document from customer again.
Thanks,</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_root"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weeks=-9, days=2)).strftime('%Y-%m-%d 11:23:17')"/>
        </record>
        <record id="project_2_task_3_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_2_task_3"/>
            <field name="parent_id" ref="project_2_task_3_message_1"/>
            <field name="body">Ok, I have checked the mail,
I will update the document and let you know.</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weeks=-9, days=2)).strftime('%Y-%m-%d 12:04:58')"/>
        </record>
        <record id="project_2_task_3_message_3" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_2_task_3"/>
            <field name="parent_id" ref="project_2_task_3_message_2"/>
            <field name="body">Fine!
Send it ASAP, its urgent.</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_root"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weeks=-9, days=2)).strftime('%Y-%m-%d 12:15:26')"/>
        </record>

        <record id="project_2_task_4" model="project.task">
            <field name="planned_hours" eval="16.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">User interface improvements</field>
            <field name="tag_ids" eval="[Command.set([
                    ref('project.project_tags_01'),
                    ref('project.project_tags_03')])]"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="kanban_state">done</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_3'))]"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=1)"/>
            <field name="color">2</field>
        </record>
        <record id="project_2_task_4_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_4"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() + relativedelta(weeks=-11)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_4_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_4_mail_message_1"/>
        </record>

        <record id="project_2_task_5" model="project.task">
            <field name="planned_hours" eval="38.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Social network integration</field>
            <field name="description">Facebook and Twitter integration</field>
            <field name="kanban_state">blocked</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_3'))]"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=5)"/>
            <field name="color">2</field>
        </record>
        <record id="project_2_task_5_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_5"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() + relativedelta(weeks=-12, days=6)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_5_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_5_mail_message_1"/>
        </record>

        <record id="project_2_task_6" model="project.task">
            <field name="planned_hours">42.0</field>
            <field name="user_ids" eval="False"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Create new components</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_3'))]"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=4)"/>
            <field name="color">11</field>
        </record>
        <record id="project_2_task_6_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_6"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() + relativedelta(weeks=-11, days=3)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_6_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_6_mail_message_1"/>
        </record>

        <record id="project_2_task_7" model="project.task">
            <field name="planned_hours" eval="22.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">New portal system</field>
            <field name="priority">0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="tag_ids" eval="[Command.set([ref('project.project_tags_02')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_3'))]"/>
        </record>
        <record id="project_2_task_7_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_7"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() + relativedelta(weeks=-12, days=5)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_7_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_7_mail_message_1"/>
        </record>
        <record id="project_2_task_7_mail_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_7"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() + relativedelta(weeks=-9, days=3)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_7_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_2_task_7_mail_message_2"/>
        </record>

        <record id="project_2_task_8" model="project.task">
            <field name="planned_hours">14.0</field>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Usability review</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_03')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids"
                   eval="[Command.link(ref('project.project_2_task_7')), Command.link(ref('project.project_2_task_5')), Command.link(ref('project.project_2_task_4')), Command.link(ref('project.project_2_task_6'))]"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=9)"/>
            <field name="color">7</field>
        </record>
        <record id="project_2_task_8_mail_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project.project_2_task_8"/>
            <field name="message_type">notification</field>
            <field name="subtype_id" ref="mt_task_stage"/>
            <field name="date" eval="DateTime.now() - relativedelta(weeks=10)"/>
            <field name="author_id" ref="base.partner_admin"/>
        </record>
        <record id="project_2_task_8_mail_message_1_track_1" model="mail.tracking.value">
            <field name="field" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="field_desc">Stage</field>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="field_type">many2one</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_8_mail_message_1"/>
        </record>

        <record id="project_2_task_9" model="project.task">
            <field name="planned_hours" eval="18.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Document management</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="kanban_state">done</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_8'))]"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=15)"/>
            <field name="color">4</field>
        </record>

        <record id="project_2_task_10" model="project.task">
            <field name="sequence">20</field>
            <field name="planned_hours">35.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Unit Testing</field>
            <field name="description">The most important part!</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_8'))]"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=15)"/>
            <field name="color">5</field>
        </record>

        <record id="project_2_task_11" model="project.task">
            <field name="planned_hours" eval="20.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="stage_id" ref="project_stage_3"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Code Documentation</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="active" eval="False"/>
        </record>

        <!-- Project 3 Tasks -->
        <record id="project_3_task_1" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Entry Hall</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">3</field>
        </record>

        <record id="project_3_task_2" model="project.task">
            <field name="planned_hours" eval="10.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Check Lift</field>
            <field name="date_deadline" eval="DateTime.today() + relativedelta(days=-10)"/>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">4</field>
        </record>
        <record id="project_3_task_2_message_1" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_3_task_2"/>
            <field name="body">The elevator's state leaves much to be desired on many levels, we might need to take steps to repair it.</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="date" eval="(DateTime.now() - relativedelta(weeks=3)).strftime('%Y-%m-%d 09:42:13')"/>
        </record>
        <record id="project_3_task_2_message_2" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_3_task_2"/>
            <field name="parent_id" ref="project_3_task_2_message_1"/>
            <field name="body">This is not very uplifting, it would probably raise the expenses by a lot. 😕</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="date" eval="(DateTime.now() - relativedelta(weeks=3)).strftime('%Y-%m-%d 10:23:48')"/>
        </record>
        <record id="project_3_task_2_message_3" model="mail.message">
            <field name="model">project.task</field>
            <field name="res_id" ref="project_3_task_2"/>
            <field name="parent_id" ref="project_3_task_2_message_2"/>
            <field name="body">I know, it's driving me up the wall.</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="date" eval="(DateTime.now() - relativedelta(weeks=3)).strftime('%Y-%m-%d 10:57:04')"/>
        </record>

        <record id="project_3_task_3" model="project.task">
            <field name="planned_hours" eval="24.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Room 1: Paint</field>
            <field name="description">Repaint the walls with the hex color #0FF1CE</field>
            <field name="kanban_state">done</field>
            <field name="priority">0</field>
            <field name="date_deadline" eval="DateTime.today() - relativedelta(days=5)"/>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_01')])]"/>
            <field name="color">9</field>
        </record>

        <record id="project_3_task_4" model="project.task">
            <field name="planned_hours" eval="76.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Bathroom</field>
            <field name="stage_id" ref="project_stage_2"/>
        </record>

        <record id="project_3_task_5" model="project.task">
            <field name="planned_hours" eval="40.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Room 2: Paint</field>
            <field name="stage_id" ref="project_stage_3"/>
            <field name="active">False</field>
        </record>

        <!-- Private tasks -->
        <record id="project_private_task_1" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Buy a gift for Marc Demo's birthday</field>
        </record>
        <record id="project_private_task_2" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Change left screen cable</field>
        </record>
        <record id="project_private_task_3" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="project_id"/>
            <field name="name">Clean kitchen fridge</field>
        </record>
        <record id="project_private_task_4" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="project_id"/>
            <field name="name">Check employees lunch accounts</field>
        </record>

        <!-- Tasks personal stages -->
        <!-- Admin -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_9')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_0')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_1_task_6')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_1')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_3')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_2')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_2')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_2')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_1')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_3')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_1_task_5')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_4')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_1_task_7')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_4')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_4')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_4')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_8')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_4')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_1_task_3')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_1')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_2')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_7')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_3_task_2')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_3_task_3')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_3_task_4')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
        </function>

        <!-- Demo -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_1_task_9')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_1')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_4')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_1')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_3')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_2')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_3')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_2')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_1_task_8')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_3')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_8')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_3')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_5')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_4')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_1_task_2')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_1')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_2_task_7')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_3_task_1')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_3_task_5')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_6')}"/>
        </function>


        <!-- Rating Demo Data -->
        <record id="rating_task_1" model="rating.rating">
            <field name="access_token">PROJECT_1</field>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="rated_partner_id" ref="base.partner_root"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field name="res_id" ref="project.project_1_task_3"/>
        </record>
        <function model="project.task" name="rating_apply"
            eval="([ref('project.project_1_task_3')], 5, 'PROJECT_1', None, 'Good Job')"/>

        <record id="rating_task_2" model="rating.rating">
            <field name="access_token">PROJECT_2</field>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="rated_partner_id" ref="base.partner_demo"/>
            <field name="partner_id" ref="base.partner_demo"/>
            <field name="res_id" ref="project.project_2_task_7"/>
        </record>
        <function model="project.task" name="rating_apply"
            eval="([ref('project.project_2_task_7')], 1, 'PROJECT_2', None, 'Not as good as expected')"/>

        <record id="rating_task_3" model="rating.rating">
            <field name="access_token">PROJECT_3</field>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="rated_partner_id" ref="base.partner_root"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field name="res_id" ref="project.project_1_task_4"/>
        </record>
        <function model="project.task" name="rating_apply"
            eval="([ref('project.project_1_task_4')], 5, 'PROJECT_3', None, 'Exactly what I asked for, thank you!')"/>

        <record id="rating_task_4" model="rating.rating">
            <field name="access_token">PROJECT_4</field>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="rated_partner_id" ref="base.partner_root"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field name="res_id" ref="project.project_1_task_2"/>
        </record>
        <function model="project.task" name="rating_apply"
            eval="([ref('project.project_1_task_2')], 1, 'PROJECT_4', None, 'There must have been some miscomunication, because the result is not quite what I had in mind. I would like to request some modifications.')"/>


        <!-- add the email template as value for the project stage 2 -->
        <record id="project.project_stage_2" model="project.task.type">
            <field name="rating_template_id" ref="rating_project_request_email_template"/>
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
        project_data = self.env['project.project']._read_group([('analytic_account_id', 'in', self.ids)], ['analytic_account_id'], ['analytic_account_id'])
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
            "name": _("Projects"),
        }
        if len(self.project_ids) == 1:
            result['views'] = [(False, "form")]
            result['res_id'] = self.project_ids.id
        return result

```

## File: models\company.py

```python
# -*- coding: utf-8 -*-
from odoo import fields, models


class ResCompany(models.Model):
    _name = "res.company"
    _inherit = "res.company"

    analytic_plan_id = fields.Many2one(
        'account.analytic.plan',
        string="Default Plan",
        check_company=True,
        readonly=False,
        compute="_compute_analytic_plan_id",
        help="Default Plan for a new analytic account for projects")

    def _compute_analytic_plan_id(self):
        for company in self:
            default_plan = self.env['ir.config_parameter'].with_company(company).sudo().get_param("default_analytic_plan_id_%s" % company.id)
            company.analytic_plan_id = int(default_plan) if default_plan else False
            if not company.analytic_plan_id:
                company.analytic_plan_id = self.env['account.analytic.plan'].with_company(company)._get_default()

    def write(self, values):
        for company in self:
            if 'analytic_plan_id' in values:
                self.env['ir.config_parameter'].sudo().set_param("default_analytic_plan_id_%s" % company.id, values['analytic_plan_id'])
        return super().write(values)

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
        if not self.env.user.has_group('project.group_project_manager'):
            res.append(self.env.ref('project.rating_rating_menu_project').id)
        if self.env.user.has_group('project.group_project_stages'):
            res.append(self.env.ref('project.menu_projects').id)
            res.append(self.env.ref('project.menu_projects_config').id)
        return res

```

## File: models\mail_message.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools.sql import create_index


class MailMessage(models.Model):
    _inherit = 'mail.message'

    def init(self):
        super().init()
        create_index(
            self._cr,
            'mail_message_date_res_id_id_for_burndown_chart',
            self._table,
            ['date', 'res_id', 'id'],
            where="model='project.task' AND message_type='notification'"
        )

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import json
from pytz import UTC
from collections import defaultdict
from datetime import timedelta, datetime, time
from random import randint

from odoo import api, Command, fields, models, tools, SUPERUSER_ID, _, _lt
from odoo.addons.rating.models import rating_data
from odoo.addons.web_editor.controllers.main import handle_history_divergence
from odoo.exceptions import UserError, ValidationError, AccessError
from odoo.osv import expression
from odoo.tools.misc import get_lang

from .project_task_recurrence import DAYS, WEEKS
from .project_update import STATUS_COLOR

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
    'display_parent_task_button',
    'allow_milestones',
    'milestone_id',
    'has_late_and_unreached_milestone',
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
        default=lambda self: self._get_default_project_ids(),
        help="Projects in which this stage is present. If you follow a similar workflow in several projects,"
            " you can share this stage among them and get consolidated information this way.")
    legend_blocked = fields.Char(
        'Red Kanban Label', default=lambda s: _('Blocked'), translate=True, required=True)
    legend_done = fields.Char(
        'Green Kanban Label', default=lambda s: _('Ready'), translate=True, required=True)
    legend_normal = fields.Char(
        'Grey Kanban Label', default=lambda s: _('In Progress'), translate=True, required=True)
    mail_template_id = fields.Many2one(
        'mail.template',
        string='Email Template',
        domain=[('model', '=', 'project.task')],
        help="If set, an email will be automatically sent to the customer when the task reaches this stage.")
    fold = fields.Boolean(string='Folded in Kanban',
        help='If enabled, this stage will be displayed as folded in the Kanban view of your tasks. Tasks in a folded stage are considered as closed (not applicable to personal stages).')
    rating_template_id = fields.Many2one(
        'mail.template',
        string='Rating Email Template',
        domain=[('model', '=', 'project.task')],
        help="If set, a rating request will automatically be sent by email to the customer when the task reaches this stage. \n"
             "Alternatively, it will be sent at a regular interval as long as the task remains in this stage, depending on the configuration of your project. \n"
             "To use this feature make sure that the 'Customer Ratings' option is enabled on your project.")
    auto_validation_kanban_state = fields.Boolean('Automatic Kanban Status', default=False,
        help="Automatically modify the kanban state when the customer replies to the feedback for this stage.\n"
            " * Good feedback from the customer will update the kanban state to 'ready for the new stage' (green bullet).\n"
            " * Neutral or bad feedback will set the kanban state to 'blocked' (red bullet).\n")
    disabled_rating_warning = fields.Text(compute='_compute_disabled_rating_warning')

    user_id = fields.Many2one('res.users', 'Stage Owner', index=True)

    def unlink_wizard(self, stage_view=False):
        self = self.with_context(active_test=False)
        # retrieves all the projects with a least 1 task in that stage
        # a task can be in a stage even if the project is not assigned to the stage
        readgroup = self.with_context(active_test=False).env['project.task']._read_group([('stage_id', 'in', self.ids)], ['project_id'], ['project_id'])
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

    def toggle_active(self):
        res = super().toggle_active()
        stage_active = self.filtered('active')
        inactive_tasks = self.env['project.task'].with_context(active_test=False).search(
            [('active', '=', False), ('stage_id', 'in', stage_active.ids)], limit=1)
        if stage_active and inactive_tasks:
            wizard = self.env['project.task.type.delete.wizard'].create({
                'stage_ids': stage_active.ids,
            })

            return {
                'name': _('Unarchive Tasks'),
                'view_mode': 'form',
                'res_model': 'project.task.type.delete.wizard',
                'views': [(self.env.ref('project.view_project_task_type_unarchive_wizard').id, 'form')],
                'type': 'ir.actions.act_window',
                'res_id': wizard.id,
                'target': 'new',
            }
        return res

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
    _rating_satisfaction_days = 30  # takes 30 days by default
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
        domain = [('project_id', 'in', self.ids), ('is_closed', '=', False)]
        fields = ['project_id', 'display_project_id:count']
        groupby = ['project_id', 'active']
        result_wo_subtask = defaultdict(int)
        result_with_subtasks = defaultdict(int)
        task_all_data = self.env['project.task'].with_context(active_test=False)._read_group(domain, fields, groupby, lazy=False)
        active_project_ids = self.filtered('active').ids
        for data in task_all_data:
            project_id = data['project_id'][0]
            if data['active'] or project_id not in active_project_ids:
                # count active tasks only of all if the project is archived
                result_wo_subtask[project_id] += data['display_project_id']
            if data['active'] or not self.env.context.get('active_test', True):
                # count subtasks only for active tasks
                result_with_subtasks[project_id] += data['__count']

        for project in self:
            project.task_count = result_wo_subtask[project.id]
            project.task_count_with_subtasks = result_with_subtasks[project.id]

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

    name = fields.Char("Name", index='trigram', required=True, tracking=True, translate=True, default_export_compatible=True,
        help="Name of your project. It can be anything you want e.g. the name of a customer or a service.")
    description = fields.Html(help="Description to provide more information and context about this project")
    active = fields.Boolean(default=True,
        help="If the active field is set to False, it will allow you to hide the project without removing it.")
    sequence = fields.Integer(default=10)
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
        help="Analytic account to which this project, its tasks and its timesheets are linked. \n"
            "Track the costs and revenues of your project by setting this analytic account on your related documents (e.g. sales orders, invoices, purchase orders, vendor bills, expenses etc.).\n"
            "This analytic account can be changed on each task individually if necessary.\n"
            "An analytic account is required in order to use timesheets.")
    analytic_account_balance = fields.Monetary(related="analytic_account_id.balance")

    favorite_user_ids = fields.Many2many(
        'res.users', 'project_favorite_user_rel', 'project_id', 'user_id',
        default=_get_default_favorite_user_ids,
        string='Members')
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite', compute_sudo=True,
        string='Show Project on Dashboard')
    label_tasks = fields.Char(string='Use Tasks as', default=lambda s: _('Tasks'), translate=True,
        help="Name used to refer to the tasks of your project e.g. tasks, tickets, sprints, etc...")
    tasks = fields.One2many('project.task', 'project_id', string="Task Activities")
    resource_calendar_id = fields.Many2one(
        'resource.calendar', string='Working Time',
        related='company_id.resource_calendar_id')
    type_ids = fields.Many2many('project.task.type', 'project_task_type_rel', 'project_id', 'type_id', string='Tasks Stages')
    task_count = fields.Integer(compute='_compute_task_count', string="Task Count")
    task_count_with_subtasks = fields.Integer(compute='_compute_task_count')
    task_ids = fields.One2many('project.task', 'project_id', string='Tasks',
                               domain=[('is_closed', '=', False)])
    color = fields.Integer(string='Color Index')
    user_id = fields.Many2one('res.users', string='Project Manager', default=lambda self: self.env.user, tracking=True)
    alias_enabled = fields.Boolean(string='Use Email Alias', compute='_compute_alias_enabled', readonly=False)
    alias_id = fields.Many2one('mail.alias', string='Alias', ondelete="restrict", required=True,
        help="Internal email associated with this project. Incoming emails are automatically synchronized "
             "with Tasks (or optionally Issues if the Issue Tracker module is installed).")
    alias_value = fields.Char(string='Alias email', compute='_compute_alias_value')
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
            "When following a project, portal users will get access to all of its tasks without distinction. Otherwise, they will only get access to the specific tasks they are following.\n\n"
            "When a project is shared in read-only, the portal user is redirected to their portal. They can view the tasks, but not edit them.\n"
            "When a project is shared in edit, the portal user is redirected to the kanban and list views of the tasks. They can modify a selected number of fields on the tasks.\n\n"
            "In any case, an internal user with no project access rights can still access a task, "
            "provided that they are given the corresponding URL (and that they are part of the followers if the project is private).")
    privacy_visibility_warning = fields.Char('Privacy Visibility Warning', compute='_compute_privacy_visibility_warning')
    access_instruction_message = fields.Char('Access Instruction Message', compute='_compute_access_instruction_message')
    doc_count = fields.Integer(compute='_compute_attached_docs_count', string="Number of documents attached")
    date_start = fields.Date(string='Start Date')
    date = fields.Date(string='Expiration Date', index=True, tracking=True,
        help="Date on which this project ends. The timeframe defined on the project is taken into account when viewing its planning.")
    allow_subtasks = fields.Boolean('Sub-tasks', default=lambda self: self.env.user.has_group('project.group_subtask_project'))
    allow_recurring_tasks = fields.Boolean('Recurring Tasks', default=lambda self: self.env.user.has_group('project.group_project_recurring_tasks'))
    allow_task_dependencies = fields.Boolean('Task Dependencies', default=lambda self: self.env.user.has_group('project.group_project_task_dependencies'))
    allow_milestones = fields.Boolean('Milestones', default=lambda self: self.env.user.has_group('project.group_project_milestone'))
    tag_ids = fields.Many2many('project.tags', relation='project_project_project_tags_rel', string='Tags')
    task_properties_definition = fields.PropertiesDefinition('Task Properties')

    # Project Sharing fields
    collaborator_ids = fields.One2many('project.collaborator', 'project_id', string='Collaborators', copy=False)
    collaborator_count = fields.Integer('# Collaborators', compute='_compute_collaborator_count', compute_sudo=True)

    # rating fields
    rating_request_deadline = fields.Datetime(compute='_compute_rating_request_deadline', store=True)
    rating_active = fields.Boolean('Customer Ratings', default=lambda self: self.env.user.has_group('project.group_project_rating'))
    allow_rating = fields.Boolean('Allow Customer Ratings', compute="_compute_allow_rating", default=lambda self: self.env.user.has_group('project.group_project_rating'))
    rating_status = fields.Selection(
        [('stage', 'Rating when changing stage'),
         ('periodic', 'Periodic rating')
        ], 'Customer Ratings Status', default="stage", required=True,
        help="Collect feedback from your customers by sending them a rating request when a task enters a certain stage. To do so, define a rating email template on the corresponding stages.\n"
             "Rating when changing stage: an email will be automatically sent when the task reaches the stage on which the rating email template is set.\n"
             "Periodic rating: an email will be automatically sent at regular intervals as long as the task remains in the stage in which the rating email template is set.")
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
        ('on_hold', 'On Hold'),
        ('to_define', 'Set Status'),
    ], default='to_define', compute='_compute_last_update_status', store=True, readonly=False, required=True)
    last_update_color = fields.Integer(compute='_compute_last_update_color')
    milestone_ids = fields.One2many('project.milestone', 'project_id')
    milestone_count = fields.Integer(compute='_compute_milestone_count', groups='project.group_project_milestone')
    milestone_count_reached = fields.Integer(compute='_compute_milestone_reached_count', groups='project.group_project_milestone')
    is_milestone_exceeded = fields.Boolean(compute="_compute_is_milestone_exceeded", search='_search_is_milestone_exceeded')

    _sql_constraints = [
        ('project_date_greater', 'check(date >= date_start)', "The project's start date must be before its end date.")
    ]

    @api.depends('partner_id.email')
    def _compute_partner_email(self):
        for project in self:
            if project.partner_id.email != project.partner_email:
                project.partner_email = project.partner_id.email

    def _inverse_partner_email(self):
        for project in self:
            if project.partner_id and project.partner_email != project.partner_id.email:
                project.partner_id.email = project.partner_email

    @api.depends('partner_id.phone')
    def _compute_partner_phone(self):
        for project in self:
            if project.partner_phone != project.partner_id.phone:
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
            project.access_url = f'/my/projects/{project.id}'

    def _compute_access_warning(self):
        super(Project, self)._compute_access_warning()
        for project in self.filtered(lambda x: x.privacy_visibility != 'portal'):
            project.access_warning = _(
                "The project cannot be shared with the recipient(s) because the privacy of the project is too restricted. Set the privacy to 'Visible by following customers' in order to make it accessible by the recipient(s).")

    @api.depends_context('uid')
    def _compute_allow_rating(self):
        self.allow_rating = self.env.user.has_group('project.group_project_rating')

    @api.depends('rating_status', 'rating_status_period')
    def _compute_rating_request_deadline(self):
        periods = {'daily': 1, 'weekly': 7, 'bimonthly': 15, 'monthly': 30, 'quarterly': 90, 'yearly': 365}
        for project in self:
            project.rating_request_deadline = fields.datetime.now() + timedelta(days=periods.get(project.rating_status_period, 0))

    @api.depends('last_update_id.status')
    def _compute_last_update_status(self):
        for project in self:
            project.last_update_status = project.last_update_id.status or 'to_define'

    @api.depends('last_update_status')
    def _compute_last_update_color(self):
        for project in self:
            project.last_update_color = STATUS_COLOR[project.last_update_status]

    @api.depends('milestone_ids')
    def _compute_milestone_count(self):
        read_group = self.env['project.milestone']._read_group([('project_id', 'in', self.ids)], ['project_id'], ['project_id'])
        mapped_count = {group['project_id'][0]: group['project_id_count'] for group in read_group}
        for project in self:
            project.milestone_count = mapped_count.get(project.id, 0)

    @api.depends('milestone_ids.is_reached')
    def _compute_milestone_reached_count(self):
        read_group = self.env['project.milestone']._read_group(
            [('project_id', 'in', self.ids), ('is_reached', '=', True)],
            ['project_id'],
            ['project_id'],
        )
        mapped_count = {group['project_id'][0]: group['project_id_count'] for group in read_group}
        for project in self:
            project.milestone_count_reached = mapped_count.get(project.id, 0)

    @api.depends('milestone_ids', 'milestone_ids.is_reached', 'milestone_ids.deadline', 'allow_milestones')
    def _compute_is_milestone_exceeded(self):
        today = fields.Date.context_today(self)
        read_group = self.env['project.milestone']._read_group([
            ('project_id', 'in', self.filtered('allow_milestones').ids),
            ('is_reached', '=', False),
            ('deadline', '<', today)], ['project_id'], ['project_id'])
        mapped_count = {group['project_id'][0]: group['project_id_count'] for group in read_group}
        for project in self:
            project.is_milestone_exceeded = bool(mapped_count.get(project.id, 0))

    @api.model
    def _search_is_milestone_exceeded(self, operator, value):
        if not isinstance(value, bool):
            raise ValueError(_('Invalid value: %s') % value)
        if operator not in ['=', '!=']:
            raise ValueError(_('Invalid operator: %s') % operator)

        query = """
            SELECT P.id
              FROM project_project P
         LEFT JOIN project_milestone M ON P.id = M.project_id
             WHERE M.is_reached IS false
               AND P.allow_milestones IS true
               AND M.deadline < CAST(now() AS date)
        """
        if (operator == '=' and value is True) or (operator == '!=' and value is False):
            operator_new = 'inselect'
        else:
            operator_new = 'not inselect'
        return [('id', operator_new, (query, ()))]

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
        collaborator_read_group = self.env['project.collaborator']._read_group(
            [('project_id', 'in', project_sharings.ids)],
            ['project_id'],
            ['project_id'],
        )
        collaborator_count_by_project = {res['project_id'][0]: res['project_id_count'] for res in collaborator_read_group}
        for project in self:
            project.collaborator_count = collaborator_count_by_project.get(project.id, 0)

    @api.depends('privacy_visibility')
    def _compute_privacy_visibility_warning(self):
        for project in self:
            if not project.ids:
                project.privacy_visibility_warning = ''
            elif project.privacy_visibility == 'portal' and project._origin.privacy_visibility != 'portal':
                project.privacy_visibility_warning = _('Customers will be added to the followers of their project and tasks.')
            elif project.privacy_visibility != 'portal' and project._origin.privacy_visibility == 'portal':
                project.privacy_visibility_warning = _('Portal users will be removed from the followers of the project and its tasks.')
            else:
                project.privacy_visibility_warning = ''

    @api.depends('privacy_visibility')
    def _compute_access_instruction_message(self):
        for project in self:
            if project.privacy_visibility == 'portal':
                project.access_instruction_message = _('Grant portal users access to your project or tasks by adding them as followers.')
            elif project.privacy_visibility == 'followers':
                project.access_instruction_message = _('Grant employees access to your project or tasks by adding them as followers.')
            else:
                project.access_instruction_message = ''

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
        new_tasks = self.env['project.task']
        # We want to copy archived task, but do not propagate an active_test context key
        task_ids = self.env['project.task'].with_context(active_test=False).search([('project_id', '=', self.id), ('parent_id', '=', False)]).ids
        if self.allow_task_dependencies and 'task_mapping' not in self.env.context:
            self = self.with_context(task_mapping=dict())
        for task in self.env['project.task'].browse(task_ids):
            # preserve task name and stage, normally altered during copy
            defaults = self._map_tasks_default_valeus(task, project)
            new_tasks |= task.copy(defaults)
        project.write({'tasks': [Command.set(new_tasks.ids)]})
        new_tasks._get_all_subtasks().filtered(
            lambda child: child.display_project_id == self
        ).write({
            'display_project_id': project.id
        })
        return True

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default is None:
            default = {}
        if not default.get('name'):
            default['name'] = _("%s (copy)") % (self.name)
        project = super(Project, self).copy(default)
        for follower in self.message_follower_ids:
            project.message_subscribe(partner_ids=follower.partner_id.ids, subtype_ids=follower.subtype_ids.ids)
        if self.allow_milestones:
            if 'milestone_mapping' not in self.env.context:
                self = self.with_context(milestone_mapping=dict())
            project.milestone_ids = [milestone.copy().id for milestone in self.milestone_ids]
        if 'tasks' not in default:
            self.map_tasks(project.id)

        return project

    @api.model
    def name_create(self, name):
        res = super().name_create(name)
        if res:
            # We create a default stage `new` for projects created on the fly.
            self.browse(res[0]).type_ids += self.env['project.task.type'].sudo().create({'name': _('New')})
        return res

    @api.model_create_multi
    def create(self, vals_list):
        # Prevent double project creation
        self = self.with_context(mail_create_nosubscribe=True)
        projects = super().create(vals_list)
        return projects

    def write(self, vals):
        if vals.get('access_token'):
            self.ensure_one()  # We are not supposed to add a single access token to multiple project
            if self.privacy_visibility != 'portal':
                vals['access_token'] = ''

        # directly compute is_favorite to dodge allow write access right
        if 'is_favorite' in vals:
            vals.pop('is_favorite')
            self._fields['is_favorite'].determine_inverse(self)

        if 'last_update_status' in vals and vals['last_update_status'] != 'to_define':
            for project in self:
                # This does not benefit from multi create, this is to allow the default description from being built.
                # This does seem ok since last_update_status should only be updated on one record at once.
                self.env['project.update'].with_context(default_project_id=project.id).create({
                    'name': _('Status Update - ') + fields.Date.today().strftime(get_lang(self.env).date_format),
                    'status': vals.get('last_update_status'),
                })
            vals.pop('last_update_status')
        if vals.get('privacy_visibility'):
            self._change_privacy_visibility(vals['privacy_visibility'])

        res = super(Project, self).write(vals) if vals else True

        if 'allow_recurring_tasks' in vals and not vals.get('allow_recurring_tasks'):
            self.env['project.task'].search([('project_id', 'in', self.ids), ('recurring_task', '=', True)]).write({'recurring_task': False})

        if 'active' in vals:
            # archiving/unarchiving a project does it on its tasks, too
            self.with_context(active_test=False).mapped('tasks').write({'active': vals['active']})
        if 'name' in vals and self.analytic_account_id:
            projects_read_group = self.env['project.project']._read_group(
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

    def unlink(self):
        # Delete the empty related analytic account
        analytic_accounts_to_delete = self.env['account.analytic.account']
        tasks = self.with_context(active_test=False).tasks
        for project in self:
            if project.analytic_account_id and not project.analytic_account_id.line_ids:
                analytic_accounts_to_delete |= project.analytic_account_id
        result = super(Project, self).unlink()
        tasks.unlink()
        analytic_accounts_to_delete.unlink()
        return result

    def message_subscribe(self, partner_ids=None, subtype_ids=None):
        """
        Subscribe to newly created task but not all existing active task when subscribing to a project.
        User update notification preference of project its propagated to all the tasks that the user is
        currently following.
        """
        res = super(Project, self).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)
        if subtype_ids:
            project_subtypes = self.env['mail.message.subtype'].browse(subtype_ids)
            task_subtypes = (project_subtypes.mapped('parent_id') | project_subtypes.filtered(lambda sub: sub.internal or sub.default)).ids
            if task_subtypes:
                for task in self.task_ids:
                    partners = set(task.message_partner_ids.ids) & set(partner_ids)
                    if partners:
                        task.message_subscribe(partner_ids=list(partners), subtype_ids=task_subtypes)
                self.update_ids.message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)
        return res

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

    def _notify_get_recipients_groups(self, msg_vals=None):
        """ Give access to the portal user/customer if the project visibility is portal. """
        groups = super()._notify_get_recipients_groups(msg_vals=msg_vals)
        if not self:
            return groups

        self.ensure_one()
        portal_privacy = self.privacy_visibility == 'portal'
        for group_name, _group_method, group_data in groups:
            if group_name in ['portal', 'portal_customer'] and not portal_privacy:
                group_data['has_button_access'] = False
        return groups

    # ---------------------------------------------------
    #  Actions
    # ---------------------------------------------------

    def action_project_task_burndown_chart_report(self):
        action = self.env['ir.actions.act_window']._for_xml_id('project.action_project_task_burndown_chart_report')
        action['display_name'] = _("%(name)s's Burndown Chart", name=self.name)
        return action

    # TODO to remove in master
    def action_project_timesheets(self):
        pass

    def project_update_all_action(self):
        action = self.env['ir.actions.act_window']._for_xml_id('project.project_update_all_action')
        action['display_name'] = _("%(name)s's Updates", name=self.name)
        return action

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
        action = self.env['ir.actions.act_window'].with_context(active_id=self.id)._for_xml_id('project.act_project_project_2_project_task_all')
        action['display_name'] = _("%(name)s", name=self.name)
        context = action['context'].replace('active_id', str(self.id))
        context = ast.literal_eval(context)
        context.update({
            'create': self.active,
            'active_test': self.active
            })
        action['context'] = context
        return action

    def action_view_all_rating(self):
        """ return the action to see all the rating of the project and activate default filters"""
        action = self.env['ir.actions.act_window']._for_xml_id('project.rating_rating_action_view_project_rating')
        action['display_name'] = _("%(name)s's Rating", name=self.name)
        action_context = ast.literal_eval(action['context']) if action['context'] else {}
        action_context.update(self._context)
        action_context['search_default_rating_last_30_days'] = 1
        action_context.pop('group_by', None)
        action['domain'] = [('consumed', '=', True), ('parent_res_model', '=', 'project.project'), ('parent_res_id', '=', self.id)]
        if self.rating_count == 1:
            action.update({
                'view_mode': 'form',
                'views': [(view_id, view_type) for view_id, view_type in action['views'] if view_type == 'form'],
                'res_id': self.rating_ids[0].id, # [0] since rating_ids might be > then rating_count
            })
        return dict(action, context=action_context)

    def action_view_tasks_analysis(self):
        """ return the action to see the tasks analysis report of the project """
        action = self.env['ir.actions.act_window']._for_xml_id('project.action_project_task_user_tree')
        action['display_name'] = _("%(name)s's Tasks Analysis", name=self.name)
        action_context = ast.literal_eval(action['context']) if action['context'] else {}
        action_context['search_default_project_id'] = self.id
        return dict(action, context=action_context)

    def action_get_list_view(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _("%(name)s's Milestones", name=self.name),
            'domain': [('project_id', '=', self.id)],
            'res_model': 'project.milestone',
            'views': [(self.env.ref('project.project_milestone_view_tree').id, 'tree')],
            'view_mode': 'tree',
            'help': _("""
                <p class="o_view_nocontent_smiling_face">
                    No milestones found. Let's create one!
                </p><p>
                    Track major progress points that must be reached to achieve success.
                </p>
            """),
            'context': {
                'default_project_id': self.id,
                **self.env.context
            }
        }

    # ---------------------------------------------
    #  PROJECT UPDATES
    # ---------------------------------------------

    def action_profitability_items(self, section_name, domain=None, res_id=False):
        return {}

    def get_last_update_or_default(self):
        self.ensure_one()
        labels = dict(self._fields['last_update_status']._description_selection(self.env))
        return {
            'status': labels.get(self.last_update_status, _('Set Status')),
            'color': self.last_update_color,
        }

    def get_panel_data(self):
        self.ensure_one()
        if not self.user_has_groups('project.group_project_user'):
            return {}
        panel_data = {
            'user': self._get_user_values(),
            'buttons': sorted(self._get_stat_buttons(), key=lambda k: k['sequence']),
            'currency_id': self.currency_id.id,
        }
        if self.allow_milestones:
            panel_data['milestones'] = self._get_milestones()
        if self._show_profitability():
            profitability_items = self._get_profitability_items()
            if self._get_profitability_sequence_per_invoice_type() and profitability_items and 'revenues' in profitability_items and 'costs' in profitability_items:  # sort the data values
                profitability_items['revenues']['data'] = sorted(profitability_items['revenues']['data'], key=lambda k: k['sequence'])
                profitability_items['costs']['data'] = sorted(profitability_items['costs']['data'], key=lambda k: k['sequence'])
            panel_data['profitability_items'] = profitability_items
            panel_data['profitability_labels'] = self._get_profitability_labels()
        return panel_data

    def get_milestones(self):
        if self.user_has_groups('project.group_project_user'):
            return self._get_milestones()
        return {}

    def _get_profitability_labels(self):
        return {}

    def _get_profitability_sequence_per_invoice_type(self):
        return {}

    def _get_already_included_profitability_invoice_line_ids(self):
        # To be extended to avoid account.move.line overlap between
        # profitability reports.
        return []

    def _get_user_values(self):
        return {
            'is_project_user': self.user_has_groups('project.group_project_user'),
        }

    def _show_profitability(self):
        self.ensure_one()
        return True

    def _get_profitability_aal_domain(self):
        return [('account_id', 'in', self.analytic_account_id.ids)]

    def _get_profitability_items(self, with_action=True):
        return {
            'revenues': {'data': [], 'total': {'invoiced': 0.0, 'to_invoice': 0.0}},
            'costs': {'data': [], 'total': {'billed': 0.0, 'to_bill': 0.0}},
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
            'action_type': 'object',
            'action': 'action_view_tasks',
            'show': True,
            'sequence': 3,
        }]
        if self.rating_count != 0 and self.user_has_groups('project.group_project_rating'):
            if self.rating_avg >= rating_data.RATING_AVG_TOP:
                icon = 'smile-o text-success'
            elif self.rating_avg >= rating_data.RATING_AVG_OK:
                icon = 'meh-o text-warning'
            else:
                icon = 'frown-o text-danger'
            buttons.append({
                'icon': icon,
                'text': _lt('Satisfaction'),
                'number': f'{round(100 * self.rating_avg_percentage, 2)} %',
                'action_type': 'object',
                'action': 'action_view_all_rating',
                'show': self.rating_active,
                'sequence': 15,
            })
        if self.user_has_groups('project.group_project_user'):
            buttons.append({
                'icon': 'area-chart',
                'text': _lt('Burndown Chart'),
                'action_type': 'action',
                'action': 'project.action_project_task_burndown_chart_report',
                'additional_context': json.dumps({
                    'active_id': self.id,
                }),
                'show': True,
                'sequence': 60,
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
                'show': self.privacy_visibility == "portal",
                'sequence': 66,
            })
        return buttons

    # ---------------------------------------------------
    #  Business Methods
    # ---------------------------------------------------

    @api.model
    def _create_analytic_account_from_values(self, values):
        company = self.env['res.company'].browse(values.get('company_id')) if values.get('company_id') else self.env.company
        analytic_account = self.env['account.analytic.account'].create({
            'name': values.get('name', _('Unknown Analytic Account')),
            'company_id': company.id,
            'partner_id': values.get('partner_id'),
            'plan_id': company.analytic_plan_id.id,
        })
        return analytic_account

    def _create_analytic_account(self):
        for project in self:
            analytic_account = self.env['account.analytic.account'].create({
                'name': project.name,
                'company_id': project.company_id.id,
                'partner_id': project.partner_id.id,
                'plan_id': project.company_id.analytic_plan_id.id,
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

    def _change_privacy_visibility(self, new_visibility):
        """
        Unsubscribe non-internal users from the project and tasks if the project privacy visibility
        goes from 'portal' to a different value.
        If the privacy visibility is set to 'portal', subscribe back project and tasks partners.
        """
        for project in self:
            if project.privacy_visibility == new_visibility:
                continue
            if new_visibility == 'portal':
                project.message_subscribe(partner_ids=project.partner_id.ids)
                for task in project.task_ids.filtered('partner_id'):
                    task.message_subscribe(partner_ids=task.partner_id.ids)
            elif project.privacy_visibility == 'portal':
                portal_users = project.message_partner_ids.user_ids.filtered('share')
                project.message_unsubscribe(partner_ids=portal_users.partner_id.ids)
                project.tasks._unsubscribe_portal_users()
                # revoke access_token since the project and its tasks are no longer accessible for portal/public users
                project.tasks.access_token = ''
                project.access_token = ''

    # ---------------------------------------------------
    # Project sharing
    # ---------------------------------------------------
    def _check_project_sharing_access(self):
        self.ensure_one()
        if self.privacy_visibility != 'portal':
            return False
        if self.env.user.has_group('base.group_portal'):
            return self.env['project.collaborator'].search([('project_id', '=', self.sudo().id), ('partner_id', '=', self.env.user.partner_id.id)])
        return self.env.user._is_internal()

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
    _primary_email = 'email_from'
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
        return self.stage_find(project_id, [('fold', '=', False)])

    @api.model
    def _default_personal_stage_type_id(self):
        return self.env['project.task.type'].search([('user_id', '=', self.env.user.id)], limit=1).id

    @api.model
    def _default_company_id(self):
        if self._context.get('default_project_id'):
            return self.env['project.project'].browse(self._context['default_project_id']).company_id
        return self.env.company

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        search_domain = [('id', 'in', stages.ids)]
        if 'default_project_id' in self.env.context and 'project_kanban' in self.env.context:
            search_domain = ['|', ('project_ids', '=', self.env.context['default_project_id'])] + search_domain

        stage_ids = stages._search(search_domain, order=order, access_rights_uid=SUPERUSER_ID)
        return stages.browse(stage_ids)

    @api.model
    def _read_group_personal_stage_type_ids(self, stages, domain, order):
        return stages.search(['|', ('id', 'in', stages.ids), ('user_id', '=', self.env.user.id)])

    active = fields.Boolean(default=True)
    name = fields.Char(string='Title', tracking=True, required=True, index='trigram')
    description = fields.Html(string='Description', sanitize_attributes=False)
    priority = fields.Selection([
        ('0', 'Low'),
        ('1', 'High'),
    ], default='0', index=True, string="Priority", tracking=True)
    sequence = fields.Integer(string='Sequence', default=10)
    stage_id = fields.Many2one('project.task.type', string='Stage', compute='_compute_stage_id',
        store=True, readonly=False, ondelete='restrict', tracking=True, index=True,
        default=_get_default_stage_id, group_expand='_read_group_stage_ids',
        domain="[('project_ids', '=', project_id)]", copy=False, task_dependency_tracking=True)
    tag_ids = fields.Many2many('project.tags', string='Tags',
        help="You can only see tags that are already present in your project. If you try creating a tag that is already existing in other projects, it won't generate any duplicates.")
    kanban_state = fields.Selection([
        ('normal', 'In Progress'),
        ('done', 'Ready'),
        ('blocked', 'Blocked')], string='Status',
        copy=False, default='normal', required=True, compute='_compute_kanban_state', readonly=False, store=True)
    kanban_state_label = fields.Char(compute='_compute_kanban_state_label', string='Kanban State Label', tracking=True, task_dependency_tracking=True)
    create_date = fields.Datetime("Created On", readonly=True)
    write_date = fields.Datetime("Last Updated On", readonly=True)
    date_end = fields.Datetime(string='Ending Date', index=True, copy=False)
    date_assign = fields.Datetime(string='Assigning Date', copy=False, readonly=True,
        help="Date on which this task was last assigned (or unassigned). Based on this, you can get statistics on the time it usually takes to assign tasks.")
    date_deadline = fields.Date(string='Deadline', index=True, copy=False, tracking=True, task_dependency_tracking=True)

    date_last_stage_update = fields.Datetime(string='Last Stage Update',
        index=True,
        copy=False,
        readonly=True,
        help="Date on which the stage of your task has last been modified.\n"
            "Based on this information you can identify tasks that are stalling and get statistics on the time it usually takes to move tasks from one stage to another.")
    project_id = fields.Many2one('project.project', string='Project', recursive=True,
        compute='_compute_project_id', store=True, readonly=False, precompute=True,
        index=True, tracking=True, check_company=True, change_default=True)
    task_properties = fields.Properties('Properties', definition='project_id.task_properties_definition', copy=True)
    # Defines in which project the task will be displayed / taken into account in statistics.
    # Example: 1 task A with 1 subtask B in project P
    # A -> project_id=P, display_project_id=P
    # B -> project_id=P (to inherit from ACL/security rules), display_project_id=False
    display_project_id = fields.Many2one('project.project', index=True)
    planned_hours = fields.Float("Initially Planned Hours", tracking=True)
    subtask_planned_hours = fields.Float("Sub-tasks Planned Hours", compute='_compute_subtask_planned_hours',
        help="Sum of the hours allocated for all the sub-tasks (and their own sub-tasks) linked to this task. Usually less than or equal to the allocated hours of this task.")
    # Tracking of this field is done in the write function
    user_ids = fields.Many2many('res.users', relation='project_task_user_rel', column1='task_id', column2='user_id', string='Assignees', context={'active_test': False}, tracking=True)
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
        search='_search_personal_stage_type_id', default=_default_personal_stage_type_id,
        help="The current user's personal task stage.")
    partner_id = fields.Many2one('res.partner',
        string='Customer', recursive=True, tracking=True,
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
    partner_city = fields.Char(related='partner_id.city', readonly=False)
    email_cc = fields.Char(help='Email addresses that were in the CC of the incoming emails from this task and that are not currently linked to an existing customer.')
    manager_id = fields.Many2one('res.users', string='Project Manager', related='project_id.user_id', readonly=True)
    company_id = fields.Many2one(
        'res.company', string='Company', compute='_compute_company_id', store=True, readonly=False,
        required=True, copy=True, default=_default_company_id)
    color = fields.Integer(string='Color Index')
    project_color = fields.Integer(related='project_id.color', string='Project Color')
    rating_active = fields.Boolean(string='Project Rating Status', related="project_id.rating_active")
    attachment_ids = fields.One2many('ir.attachment', compute='_compute_attachment_ids', string="Main Attachments",
        help="Attachments that don't come from a message.")
    # In the domain of displayed_image_id, we couln't use attachment_ids because a one2many is represented as a list of commands so we used res_model & res_id
    displayed_image_id = fields.Many2one('ir.attachment', domain="[('res_model', '=', 'project.task'), ('res_id', '=', id), ('mimetype', 'ilike', 'image')]", string='Cover Image')
    legend_blocked = fields.Char(related='stage_id.legend_blocked', string='Kanban Blocked Explanation', readonly=True)
    legend_done = fields.Char(related='stage_id.legend_done', string='Kanban Valid Explanation', readonly=True)
    legend_normal = fields.Char(related='stage_id.legend_normal', string='Kanban Ongoing Explanation', readonly=True)
    is_closed = fields.Boolean(related="stage_id.fold", string="Closing Stage", store=True, index=True, help="Folded in Kanban stages are closing stages.")
    parent_id = fields.Many2one('project.task', string='Parent Task', index=True)
    ancestor_id = fields.Many2one('project.task', string='Ancestor Task', compute='_compute_ancestor_id', index='btree_not_null', recursive=True, store=True)
    child_ids = fields.One2many('project.task', 'parent_id', string="Sub-tasks")
    child_text = fields.Char(compute="_compute_child_text")
    allow_subtasks = fields.Boolean(string="Allow Sub-tasks", related="project_id.allow_subtasks", readonly=True)
    subtask_count = fields.Integer("Sub-task Count", compute='_compute_subtask_count')
    email_from = fields.Char(string='Email From', help="These people will receive email.", index='trigram',
        compute='_compute_email_from', recursive=True, store=True, readonly=False, copy=False)
    project_privacy_visibility = fields.Selection(related='project_id.privacy_visibility', string="Project Visibility")
    # Computed field about working time elapsed between record creation and assignation/closing.
    working_hours_open = fields.Float(compute='_compute_elapsed', string='Working Hours to Assign', digits=(16, 2), store=True, group_operator="avg")
    working_hours_close = fields.Float(compute='_compute_elapsed', string='Working Hours to Close', digits=(16, 2), store=True, group_operator="avg")
    working_days_open = fields.Float(compute='_compute_elapsed', string='Working Days to Assign', store=True, group_operator="avg")
    working_days_close = fields.Float(compute='_compute_elapsed', string='Working Days to Close', store=True, group_operator="avg")
    # customer portal: include comment and incoming emails in communication history
    website_message_ids = fields.One2many(domain=lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment'])])
    is_private = fields.Boolean(compute='_compute_is_private', search='_search_is_private')
    allow_milestones = fields.Boolean(related='project_id.allow_milestones')
    milestone_id = fields.Many2one(
        'project.milestone',
        'Milestone',
        domain="[('project_id', '=', project_id)]",
        compute='_compute_milestone_id',
        readonly=False,
        store=True,
        tracking=True,
        index='btree_not_null',
        help="Deliver your services automatically when a milestone is reached by linking it to a sales order item."
    )
    has_late_and_unreached_milestone = fields.Boolean(
        compute='_compute_has_late_and_unreached_milestone',
        search='_search_has_late_and_unreached_milestone',
    )

    # Task Dependencies fields
    allow_task_dependencies = fields.Boolean(related='project_id.allow_task_dependencies')
    # Tracking of this field is done in the write function
    depend_on_ids = fields.Many2many('project.task', relation="task_dependencies_rel", column1="task_id",
                                     column2="depends_on_id", string="Blocked By", tracking=True, copy=False,
                                     domain="[('project_id', '!=', False), ('id', '!=', id)]")
    dependent_ids = fields.Many2many('project.task', relation="task_dependencies_rel", column1="depends_on_id",
                                     column2="task_id", string="Block", copy=False,
                                     domain="[('project_id', '!=', False), ('id', '!=', id)]")
    dependent_tasks_count = fields.Integer(string="Dependent Tasks", compute='_compute_dependent_tasks_count')
    is_blocked = fields.Boolean(compute='_compute_is_blocked', store=True, recursive=True)

    # Project sharing fields
    display_parent_task_button = fields.Boolean(compute='_compute_display_parent_task_button', compute_sudo=True)

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
    recurrence_message = fields.Char(string='Next Recurrencies', compute='_compute_recurrence_message', groups="project.group_project_user")

    repeat_interval = fields.Integer(string='Repeat Every', default=1, compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_unit = fields.Selection([
        ('day', 'Days'),
        ('week', 'Weeks'),
        ('month', 'Months'),
        ('year', 'Years'),
    ], default='week', compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_type = fields.Selection([
        ('forever', 'Forever'),
        ('until', 'End Date'),
        ('after', 'Number of Repetitions'),
    ], default="forever", string="Until", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_until = fields.Date(string="End Date", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_number = fields.Integer(string="Repetitions", default=1, compute='_compute_repeat', readonly=False, groups="project.group_project_user")

    repeat_on_month = fields.Selection([
        ('date', 'Date of the Month'),
        ('day', 'Day of the Month'),
    ], default='date', compute='_compute_repeat', readonly=False, groups="project.group_project_user")

    repeat_on_year = fields.Selection([
        ('date', 'Date of the Year'),
        ('day', 'Day of the Year'),
    ], default='date', compute='_compute_repeat', readonly=False, groups="project.group_project_user")

    mon = fields.Boolean(string="Mon", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    tue = fields.Boolean(string="Tue", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    wed = fields.Boolean(string="Wed", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    thu = fields.Boolean(string="Thu", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    fri = fields.Boolean(string="Fri", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    sat = fields.Boolean(string="Sat", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    sun = fields.Boolean(string="Sun", compute='_compute_repeat', readonly=False, groups="project.group_project_user")

    repeat_day = fields.Selection([
        (str(i), str(i)) for i in range(1, 32)
    ], compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_week = fields.Selection([
        ('first', 'First'),
        ('second', 'Second'),
        ('third', 'Third'),
        ('last', 'Last'),
    ], default='first', compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_weekday = fields.Selection([
        ('mon', 'Monday'),
        ('tue', 'Tuesday'),
        ('wed', 'Wednesday'),
        ('thu', 'Thursday'),
        ('fri', 'Friday'),
        ('sat', 'Saturday'),
        ('sun', 'Sunday'),
    ], string='Day Of The Week', compute='_compute_repeat', readonly=False, groups="project.group_project_user")
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
    ], compute='_compute_repeat', readonly=False, groups="project.group_project_user")

    repeat_show_dow = fields.Boolean(compute='_compute_repeat_visibility', groups="project.group_project_user")
    repeat_show_day = fields.Boolean(compute='_compute_repeat_visibility', groups="project.group_project_user")
    repeat_show_week = fields.Boolean(compute='_compute_repeat_visibility', groups="project.group_project_user")
    repeat_show_month = fields.Boolean(compute='_compute_repeat_visibility', groups="project.group_project_user")

    # Account analytic
    analytic_account_id = fields.Many2one('account.analytic.account', ondelete='set null', compute='_compute_analytic_account_id', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", check_company=True,
        help="Analytic account to which this task and its timesheets are linked.\n"
            "Track the costs and revenues of your task by setting its analytic account on your related documents (e.g. sales orders, invoices, purchase orders, vendor bills, expenses etc.).\n"
            "By default, the analytic account of the project is set. However, it can be changed on each task individually if necessary.")
    is_analytic_account_id_changed = fields.Boolean('Is Analytic Account Manually Changed', compute='_compute_is_analytic_account_id_changed', store=True)
    project_analytic_account_id = fields.Many2one('account.analytic.account', string='Project Analytic Account', related='project_id.analytic_account_id')

    @property
    def SELF_READABLE_FIELDS(self):
        return PROJECT_TASK_READABLE_FIELDS | self.SELF_WRITABLE_FIELDS

    @property
    def SELF_WRITABLE_FIELDS(self):
        return PROJECT_TASK_WRITABLE_FIELDS

    @api.depends('project_id.analytic_account_id')
    def _compute_analytic_account_id(self):
        self.env.remove_to_compute(self._fields['is_analytic_account_id_changed'], self)
        for task in self:
            if not task.is_analytic_account_id_changed:
                task.analytic_account_id = task.project_id.analytic_account_id

    @api.depends('analytic_account_id')
    def _compute_is_analytic_account_id_changed(self):
        for task in self:
            task.is_analytic_account_id_changed = task.project_id and task.analytic_account_id != task.project_id.analytic_account_id

    @api.depends('project_id', 'parent_id')
    def _compute_is_private(self):
        # Modify accordingly, this field is used to display the lock on the task's kanban card
        for task in self:
            task.is_private = not task.project_id and not task.parent_id

    def _search_is_private(self, operator, value):
        if not isinstance(value, bool):
            raise ValueError(_('Value should be True or False (not %s)'), value)
        if operator not in ['=', '!=']:
            raise NotImplementedError(_('Operation should be = or != (not %s)'), value)
        if (operator == '=' and value) or (operator == '!=' and not value):
            return [('project_id', '=', False)]
        else:
            return [('project_id', '!=', False)]

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        if view_type == 'search' and  self.env.user.notification_type == 'email':
            for node in arch.xpath("//filter[@name='message_needaction']"):
                node.set('invisible', '1')
        return arch, view

    @api.depends('stage_id', 'project_id')
    def _compute_kanban_state(self):
        self.kanban_state = 'normal'

    @api.depends('parent_id.ancestor_id')
    def _compute_ancestor_id(self):
        for task in self:
            task.ancestor_id = task.parent_id.ancestor_id or task.parent_id

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

    def message_subscribe(self, partner_ids=None, subtype_ids=None):
        """ Set task notification based on project notification preference if user follow the project"""
        if not subtype_ids:
            project_followers = self.project_id.sudo().message_follower_ids.filtered(lambda f: f.partner_id.id in partner_ids)
            for project_follower in project_followers:
                project_subtypes = project_follower.subtype_ids
                task_subtypes = (project_subtypes.mapped('parent_id') | project_subtypes.filtered(lambda sub: sub.internal or sub.default)).ids if project_subtypes else None
                partner_ids.remove(project_follower.partner_id.id)
                super().message_subscribe(project_follower.partner_id.ids, task_subtypes)
        return super().message_subscribe(partner_ids, subtype_ids)

    @api.constrains('depend_on_ids')
    def _check_no_cyclic_dependencies(self):
        if not self._check_m2m_recursion('depend_on_ids'):
            raise ValidationError(_("Two tasks cannot depend on each other."))

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
                recurrence_title = _('A new task will be created on the following dates:')
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
        count = self.env['project.task']._read_group([('recurrence_id', 'in', recurring_tasks.recurrence_id.ids)], ['id'], 'recurrence_id')
        tasks_count = {c.get('recurrence_id')[0]: c.get('recurrence_id_count') for c in count}
        for task in recurring_tasks:
            task.recurring_count = tasks_count.get(task.recurrence_id.id, 0)

    @api.depends('dependent_ids')
    def _compute_dependent_tasks_count(self):
        tasks_with_dependency = self.filtered('allow_task_dependencies')
        (self - tasks_with_dependency).dependent_tasks_count = 0
        if tasks_with_dependency:
            group_dependent = self.env['project.task']._read_group([
                ('depend_on_ids', 'in', tasks_with_dependency.ids),
            ], ['depend_on_ids'], ['depend_on_ids'])
            dependent_tasks_count_dict = {
                group['depend_on_ids'][0]: group['depend_on_ids_count']
                for group in group_dependent
            }
            for task in tasks_with_dependency:
                task.dependent_tasks_count = dependent_tasks_count_dict.get(task.id, 0)

    @api.depends('depend_on_ids.is_closed', 'depend_on_ids.is_blocked')
    def _compute_is_blocked(self):
        for task in self:
            task.is_blocked = any(not blocking_task.is_closed or blocking_task.is_blocked for blocking_task in task.depend_on_ids)

    @api.depends('partner_id.email')
    def _compute_partner_email(self):
        for task in self:
            if task.partner_id.email != task.partner_email:
                task.partner_email = task.partner_id.email

    def _inverse_partner_email(self):
        for task in self:
            if task.partner_id and task.partner_email != task.partner_id.email:
                task.partner_id.email = task.partner_email

    @api.depends('partner_id.phone')
    def _compute_partner_phone(self):
        for task in self:
            if task.partner_phone != task.partner_id.phone:
                task.partner_phone = task.partner_id.phone

    def _inverse_partner_phone(self):
        for task in self:
            if task.partner_id and task.partner_phone != task.partner_id.phone:
                task.partner_id.phone = task.partner_phone

    @api.constrains('parent_id')
    def _check_parent_id(self):
        if not self._check_recursion():
            raise ValidationError(_('Error! You cannot create a recursive hierarchy of tasks.'))

    def _get_attachments_search_domain(self):
        self.ensure_one()
        return [('res_id', '=', self.id), ('res_model', '=', 'project.task')]

    def _compute_attachment_ids(self):
        for task in self:
            attachment_ids = self.env['ir.attachment'].search(task._get_attachments_search_domain()).ids
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
            task.access_url = f'/my/tasks/{task.id}'

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
        subtasks_per_task = self._get_subtask_ids_per_task_id()
        for task in self:
            task.subtask_count = len(subtasks_per_task.get(task.id, []))

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
                    task.stage_id = task.stage_find(task.project_id.id, [('fold', '=', False)])
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
            self.invalidate_recordset(fnames=['user_ids'])
            self.browse(self.ids)._read(['user_ids'])
        for task in self.with_context(prefetch_fields=False):
            task.portal_user_names = ', '.join(task.user_ids.mapped('name'))

    def _search_portal_user_names(self, operator, value):
        if operator != 'ilike' and not isinstance(value, str):
            raise ValidationError(_('Not Implemented.'))

        query = """
            SELECT task_user.task_id
              FROM project_task_user_rel task_user
        INNER JOIN res_users users ON task_user.user_id = users.id
        INNER JOIN res_partner partners ON partners.id = users.partner_id
             WHERE partners.name ILIKE %s
        """
        return [('id', 'inselect', (query, [f'%{value}%']))]

    def _compute_display_parent_task_button(self):
        accessible_parent_tasks = self.parent_id.with_user(self.env.user)._filter_access_rules('read')
        for task in self:
            task.display_parent_task_button = task.parent_id in accessible_parent_tasks

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default is None:
            default = {}
        if self.allow_task_dependencies and 'task_mapping' not in self.env.context:
            self = self.with_context(task_mapping=dict())
        has_default_name = bool(default.get('name', ''))
        if not has_default_name:
            default['name'] = _("%s (copy)", self.name)
        if self.recurrence_id:
            default['recurrence_id'] = self.recurrence_id.copy().id
        if self.allow_subtasks:
            default_child_ids = []
            should_copy_stage_id = bool(default.get('stage_id', False))
            for child in self.child_ids:
                subtask_default = {}
                if has_default_name:
                    subtask_default['name'] = child.name
                if should_copy_stage_id:
                    subtask_default['stage_id'] = child.stage_id.id
                default_child_ids.append(child.copy(subtask_default).id)
            default['child_ids'] = default_child_ids
        task_copy = super(Task, self).copy(default)
        if self.allow_task_dependencies:
            task_mapping = self.env.context.get('task_mapping')
            task_mapping[self.id] = task_copy.id
            new_tasks = task_mapping.values()
            self.write({'depend_on_ids': [Command.unlink(t.id) for t in self.depend_on_ids if t.id in new_tasks]})
            self.write({'dependent_ids': [Command.unlink(t.id) for t in self.dependent_ids if t.id in new_tasks]})
            task_copy.write({'depend_on_ids': [Command.link(task_mapping.get(t.id, t.id)) for t in self.depend_on_ids]})
            task_copy.write({'dependent_ids': [Command.link(task_mapping.get(t.id, t.id)) for t in self.dependent_ids]})
        if self.allow_milestones:
            milestone_mapping = self.env.context.get('milestone_mapping', {})
            task_copy.milestone_id = milestone_mapping.get(task_copy.milestone_id.id, task_copy.milestone_id.id)
        return task_copy

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

        See :meth:`mail.models.MailThread._track_get_fields`"""
        fields = {name for name, field in self._fields.items() if getattr(field, 'task_dependency_tracking', None)}
        return fields and set(self.fields_get(fields))

    def _portal_get_parent_hash_token(self, pid):
        return self.project_id._sign_token(pid)

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
    def _get_view_cache_key(self, view_id=None, view_type='form', **options):
        """The override of fields_get making fields readonly for portal users
        makes the view cache dependent on the fact the user has the group portal or not

        The override of _get_view making the "Unread messages" filter invisible
        according to the user notification type
        makes the view cache dependent on the user notification type"""
        key = super()._get_view_cache_key(view_id, view_type, **options)
        key = key + (self.env.user.has_group('base.group_portal'),)
        if view_type == 'search':
            key += (self.env.user.notification_type,)
        return key

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
        project_id = vals.get('project_id', self.env.context.get('default_project_id'))
        if project_id:
            project = self.env['project.project'].browse(project_id)
            if project.analytic_account_id:
                vals['analytic_account_id'] = project.analytic_account_id.id
        elif 'default_user_ids' not in self.env.context:
            user_ids = vals.get('user_ids', [])
            user_ids.append(Command.link(self.env.user.id))
            vals['user_ids'] = user_ids

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
            or key == 'default_user_ids' and value is False \
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
            fields_list += [term[0].split('.')[0] for term in domain if isinstance(term, (tuple, list)) and term not in [expression.TRUE_LEAF, expression.FALSE_LEAF]]
        self._ensure_fields_are_accessible(fields_list)
        return super(Task, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        fields_list = {term[0] for term in args if isinstance(term, (tuple, list)) and term not in [expression.TRUE_LEAF, expression.FALSE_LEAF]}
        self._ensure_fields_are_accessible(fields_list)
        for index, leaf in enumerate(args):
            if leaf[0] == 'personal_stage_type_ids' and leaf[1] == '=' and not leaf[2]:
                types = self.env['project.task.type']._search([('user_id', '=', self.env.uid)])
                args[index] = ('personal_stage_type_ids', 'not in', types)
        return super(Task, self)._search(args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

    def mapped(self, func):
        # Note: This will protect the filtered method too
        if func and isinstance(func, str):
            fields_list = func.split('.')
            self._ensure_fields_are_accessible(fields_list)
        return super(Task, self).mapped(func)

    def filtered_domain(self, domain):
        fields_list = [term[0] for term in domain if isinstance(term, (tuple, list)) and term not in [expression.TRUE_LEAF, expression.FALSE_LEAF]]
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
            project_id = vals.get('project_id')
            if project_id:
                self = self.with_context(default_project_id=project_id)
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
        is_superuser = self._uid == SUPERUSER_ID
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
            if not project_id and ("stage_id" in vals or self.env.context.get('default_stage_id')):
                vals["stage_id"] = False

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
                if not project_id:
                    user_ids = self._fields['user_ids'].convert_to_cache(vals.get('user_ids', []), self)
                    if self.env.user.id not in user_ids and not is_superuser:
                        vals['user_ids'] = [Command.set(list(user_ids) + [self.env.uid])]
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
        current_partner = self.env.user.partner_id
        for task in tasks:
            if task.project_id.privacy_visibility == 'portal':
                task._portal_ensure_token()
            if current_partner not in task.message_partner_ids:
                task.message_subscribe(current_partner.ids)
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
            vals['recurring_task'] = False
        if 'recurrence_id' in vals and vals.get('recurrence_id') and any(not task.active for task in self):
            raise UserError(_('Archived tasks cannot be recurring. Please unarchive the task first.'))
        # stage change: update date_last_stage_update
        if 'stage_id' in vals:
            if not 'project_id' in vals and self.filtered(lambda t: not t.project_id):
                raise UserError(_('You can only set a personal stage on a private task.'))

            vals.update(self.update_date_end(vals['stage_id']))
            vals['date_last_stage_update'] = now
        task_ids_without_user_set = set()
        if 'user_ids' in vals and 'date_assign' not in vals:
            # prepare update of date_assign after super call
            task_ids_without_user_set = {task.id for task in self if not task.user_ids}

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
                    recurrence_domain = expression.OR([recurrence_domain, ['&', ('recurrence_id', '=', task.recurrence_id.id), ('create_date', '>=', task.create_date)]])
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

        if "personal_stage_type_id" in vals and not vals['personal_stage_type_id']:
            del vals['personal_stage_type_id']

        result = super(Task, tasks).write(vals)
        if portal_can_write:
            super(Task, tasks_no_sudo).write(vals_no_sudo)

        if 'user_ids' in vals:
            tasks._populate_missing_personal_stages()

        # user_ids change: update date_assign
        if 'user_ids' in vals:
            for task in self:
                if not task.user_ids and task.date_assign:
                    task.date_assign = False
                elif 'date_assign' not in vals and task.id in task_ids_without_user_set:
                    task.date_assign = now

        # rating on stage
        if 'stage_id' in vals and vals.get('stage_id'):
            tasks.filtered(lambda x: x.project_id.rating_active and x.project_id.rating_status == 'stage')._send_task_rating_mail(force_send=True)
        for task in tasks:
            if task.display_project_id != task.project_id and not task.parent_id:
                # We must make the display_project_id follow the project_id if no parent_id set
                task.display_project_id = task.project_id

        self._task_message_auto_subscribe_notify({task: task.user_ids - old_user_ids[task] - self.env.user for task in self})
        return result

    def update_date_end(self, stage_id):
        project_task_type = self.env['project.task.type'].browse(stage_id)
        if project_task_type.fold:
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
        # Avoid recomputing kanban_state
        self.env.remove_to_compute(self._fields['kanban_state'], self)
        for task in self:
            if task.parent_id:
                task.project_id = task.display_project_id or task.parent_id.project_id

    @api.depends('project_id')
    def _compute_milestone_id(self):
        for task in self:
            if task.project_id != task.milestone_id.project_id:
                task.milestone_id = False

    def _compute_has_late_and_unreached_milestone(self):
        if all(not task.allow_milestones for task in self):
            self.has_late_and_unreached_milestone = False
            return
        late_milestones = self.env['project.milestone'].sudo()._search([  # sudo is needed for the portal user in Project Sharing.
            ('id', 'in', self.milestone_id.ids),
            ('is_reached', '=', False),
            ('deadline', '<', fields.Date.today()),
        ])
        for task in self:
            task.has_late_and_unreached_milestone = task.allow_milestones and task.milestone_id.id in late_milestones

    def _search_has_late_and_unreached_milestone(self, operator, value):
        if operator not in ('=', '!=') or not isinstance(value, bool):
            raise NotImplementedError(_('The search does not support the %s operator or %s value.', operator, value))
        domain = [
            ('allow_milestones', '=', True),
            ('milestone_id', '!=', False),
            ('milestone_id.is_reached', '=', False),
            ('milestone_id.deadline', '!=', False), ('milestone_id.deadline', '<', fields.Date.today())
        ]
        if (operator == '!=' and value) or (operator == '=' and not value):
            domain.insert(0, expression.NOT_OPERATOR)
            domain = expression.distribute_not(domain)
        return domain

    # ---------------------------------------------------
    # Mail gateway
    # ---------------------------------------------------

    def _notify_by_email_prepare_rendering_context(self, message, msg_vals=False, model_description=False,
                                                   force_email_company=False, force_email_lang=False):
        render_context = super()._notify_by_email_prepare_rendering_context(
            message, msg_vals, model_description=model_description,
            force_email_company=force_email_company, force_email_lang=force_email_lang
        )
        if self.date_deadline:
            render_context['subtitles'].append(
                _('Deadline: %s', self.date_deadline.strftime(get_lang(self.env).date_format)))
        elif self.date_assign:
            render_context['subtitles'].append(
                _('Assigned On: %s', self.date_assign.strftime(get_lang(self.env).date_format)))
        return render_context

    @api.model
    def _task_message_auto_subscribe_notify(self, users_per_task):
        # Utility method to send assignation notification upon writing/creation.
        template_id = self.env['ir.model.data']._xmlid_to_res_id('project.project_message_user_assigned', raise_if_not_found=False)
        if not template_id:
            return
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
                assignation_msg = self.env['ir.qweb']._render('project.project_message_user_assigned', values, minimal_qcontext=True)
                assignation_msg = self.env['mail.render.mixin']._replace_local_links(assignation_msg)
                task.message_notify(
                    subject=_('You have been assigned to %s', task.display_name),
                    body=assignation_msg,
                    partner_ids=user.partner_id.ids,
                    record_name=task.display_name,
                    email_layout_xmlid='mail.mail_notification_layout',
                    model_description=task_model_description,
                    mail_auto_delete=False,
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
            except Exception:
                pass
        return new_followers

    def _mail_track(self, tracked_fields, initial_values):
        changes, tracking_value_ids = super()._mail_track(tracked_fields, initial_values)
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
        mail_message_subtype_per_kanban_state = {
            'blocked': 'project.mt_task_blocked',
            'done': 'project.mt_task_ready',
            'normal': 'project.mt_task_progress',
        }
        if 'stage_id' in init_values:
            return self.env.ref('project.mt_task_stage')
        elif 'kanban_state_label' in init_values and self.kanban_state in mail_message_subtype_per_kanban_state:
            return self.env.ref(mail_message_subtype_per_kanban_state[self.kanban_state])
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

    def _notify_get_recipients_groups(self, msg_vals=None):
        """ Handle project users and managers recipients that can assign
        tasks and create new one directly from notification emails. Also give
        access button to portal users and portal customers. If they are notified
        they should probably have access to the document. """
        groups = super(Task, self)._notify_get_recipients_groups(msg_vals=msg_vals)
        if not self:
            return groups

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
        for group_name, _group_method, group_data in groups:
            if group_name in ('customer', 'user') or group_name == 'portal_customer' and not portal_privacy:
                group_data['has_button_access'] = False
            elif group_name == 'portal_customer' and portal_privacy:
                group_data['has_button_access'] = True

        return groups

    def _notify_get_reply_to(self, default=None):
        """ Override to set alias of tasks to their project if any. """
        aliases = self.sudo().mapped('project_id')._notify_get_reply_to(default=default)
        res = {task.id: aliases.get(task.project_id.id) for task in self}
        leftover = self.filtered(lambda rec: not rec.project_id)
        if leftover:
            res.update(super(Task, leftover)._notify_get_reply_to(default=default))
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
            'planned_hours': 0.0,
            'partner_id': msg.get('author_id'),
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

    def _notify_by_email_get_headers(self):
        headers = super(Task, self)._notify_by_email_get_headers()
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
        # use the sanitized body of the email from the message thread to populate the task's description
        if (
           not self.description
           and message.subtype_id == self._creation_subtype()
           and self.partner_id == message.author_id
           and msg_vals['message_type'] == 'email'
        ):
            self.description = message.body
        return super(Task, self)._message_post_after_hook(message, msg_vals)

    def action_assign_to_me(self):
        self.write({'user_ids': [(4, self.env.user.id)]})

    def action_unassign_me(self):
        self.write({'user_ids': [Command.unlink(self.env.uid)]})

    def _get_all_subtasks(self, depth=0):
        return self.browse(set.union(set(), *self._get_subtask_ids_per_task_id().values()))

    def _get_subtask_ids_per_task_id(self):
        if not self:
            return {}

        res = dict.fromkeys(self._ids, [])
        if all(self._ids):
            self.env.cr.execute(
                """
         WITH RECURSIVE task_tree
                     AS (
                     SELECT id, id as supertask_id
                       FROM project_task
                      WHERE id IN %(ancestor_ids)s
                      UNION
                         SELECT t.id, tree.supertask_id
                           FROM project_task t
                           JOIN task_tree tree
                             ON tree.id = t.parent_id
                            AND t.active in (TRUE, %(active)s)
               ) SELECT supertask_id, ARRAY_AGG(id)
                   FROM task_tree
                  WHERE id != supertask_id
               GROUP BY supertask_id
                """,
                {
                    "ancestor_ids": tuple(self.ids),
                    "active": self._context.get('active_test', True),
                }
            )
            res.update(dict(self.env.cr.fetchall()))
        else:
            res.update({
                task.id: task._get_subtasks_recursively().ids
                for task in self
            })
        return res

    def _get_subtasks_recursively(self):
        children = self.child_ids
        if not children:
            return self.env['project.task']
        return children + children._get_subtasks_recursively()

    def action_open_parent_task(self):
        return {
            'name': _('Parent Task'),
            'view_mode': 'form',
            'res_model': 'project.task',
            'res_id': self.parent_id.id,
            'type': 'ir.actions.act_window',
            'context': self._context
        }

    def action_project_sharing_view_parent_task(self):
        if self.parent_id.project_id != self.project_id and self.user_has_groups('base.group_portal'):
            project = self.parent_id.project_id._filter_access_rules_python('read')
            if project:
                url = f"/my/projects/{self.parent_id.project_id.id}/task/{self.parent_id.id}"
                if project._check_project_sharing_access():
                    url = f"/my/projects/{self.parent_id.project_id.id}?task_id={self.parent_id.id}"
                return {
                    "name": "Portal Parent Task",
                    "type": "ir.actions.act_url",
                    "url": url,
                }
            elif self.display_parent_task_button:
                return self.parent_id.get_portal_url()
            # The portal user has no access to the parent task, so normally the button should be invisible.
            return {}
        action = self.action_open_parent_task()
        action['views'] = [(self.env.ref('project.project_sharing_project_task_view_form').id, 'form')]
        return action

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

    def action_project_sharing_open_task(self):
        action = self.action_open_task()
        action['views'] = [[self.env.ref('project.project_sharing_project_task_view_form').id, 'form']]
        return action

    def action_project_sharing_open_subtasks(self):
        self.ensure_one()
        subtasks = self.env['project.task'].search([('id', 'child_of', self.id), ('id', '!=', self.id)])
        if subtasks.project_id == self.project_id:
            action = self.env['ir.actions.act_window']._for_xml_id('project.project_sharing_project_task_action_sub_task')
            if len(subtasks) == 1:
                action['view_mode'] = 'form'
                action['views'] = [(view_id, view_type) for view_id, view_type in action['views'] if view_type == 'form']
                action['res_id'] = subtasks.id
            return action
        return {
            'name': 'Portal Sub-tasks',
            'type': 'ir.actions.act_url',
            'url': f'/my/projects/{self.project_id.id}/task/{self.id}/subtasks' if len(subtasks) > 1 else subtasks.get_portal_url(query_string='project_sharing=1'),
        }

    def action_dependent_tasks(self):
        self.ensure_one()
        action = {
            'res_model': 'project.task',
            'type': 'ir.actions.act_window',
            'context': {**self._context, 'default_depend_on_ids': [Command.link(self.id)], 'show_project_update': False},
        }
        if self.dependent_tasks_count == 1:
            action['view_mode'] = 'form'
            action['res_id'] = self.dependent_ids.id
            action['views'] = [(False, 'form')]
        else:
            action['domain'] = [('depend_on_ids', '=', self.id)]
            action['name'] = _('Dependent Tasks')
            action['view_mode'] = 'tree,form,kanban,calendar,pivot,graph,activity'
        return action

    def action_recurring_tasks(self):
        return {
            'name': _('Tasks in Recurrence'),
            'type': 'ir.actions.act_window',
            'res_model': 'project.task',
            'view_mode': 'tree,form,kanban,calendar,pivot,graph,activity',
            'context': {'create': False},
            'domain': [('recurrence_id', 'in', self.recurrence_id.ids)],
        }

    def action_open_ratings(self):
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id('project.rating_rating_action_task')
        if self.rating_count == 1:
            action['view_mode'] = 'form'
            action['res_id'] = self.rating_ids[0].id
            action['views'] = [[self.env.ref('project.rating_rating_view_form_project').id, 'form']]
            return action
        else:
            return action

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

    def _rating_get_partner(self):
        res = super(Task, self)._rating_get_partner()
        if not res and self.project_id.partner_id:
            return self.project_id.partner_id
        return res

    def rating_apply(self, rate, token=None, rating=None, feedback=None,
                     subtype_xmlid=None, notify_delay_send=False):
        rating = super(Task, self).rating_apply(
            rate, token=token, rating=rating, feedback=feedback,
            subtype_xmlid=subtype_xmlid, notify_delay_send=notify_delay_send)
        if self.stage_id and self.stage_id.auto_validation_kanban_state:
            kanban_state = 'done' if rating.rating >= rating_data.RATING_LIMIT_OK else 'blocked'
            self.write({'kanban_state': kanban_state})
        return rating

    def _rating_apply_get_default_subtype_id(self):
        return self.env['ir.model.data']._xmlid_to_res_id("project.mt_task_rating")

    def _rating_get_parent_field_name(self):
        return 'project_id'

    def _rating_get_operator(self):
        """ Overwrite since we have user_ids and not user_id """
        tasks_with_one_user = self.filtered(lambda task: len(task.user_ids) == 1 and task.user_ids.partner_id)
        return tasks_with_one_user.user_ids.partner_id or self.env['res.partner']

    # ---------------------------------------------------
    # Privacy
    # ---------------------------------------------------
    def _unsubscribe_portal_users(self):
        self.message_unsubscribe(partner_ids=self.message_partner_ids.filtered('user_ids.share').ids)

    # ---------------------------------------------------
    # Analytic accounting
    # ---------------------------------------------------
    def _get_task_analytic_account_id(self):
        self.ensure_one()
        return self.analytic_account_id or self.project_analytic_account_id

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        calendar = self.env.company.resource_calendar_id
        return calendar._get_unusual_days(
            datetime.combine(fields.Date.from_string(date_from), time.min).replace(tzinfo=UTC),
            datetime.combine(fields.Date.from_string(date_to), time.max).replace(tzinfo=UTC)
        )

class ProjectTags(models.Model):
    """ Tags of project's tasks """
    _name = "project.tags"
    _description = "Project Tags"

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char('Name', required=True, translate=True)
    color = fields.Integer(string='Color', default=_get_default_color,
        help="Transparent tags are not visible in the kanban view of your projects and tasks.")
    project_ids = fields.Many2many('project.project', 'project_project_project_tags_rel', string='Projects')
    task_ids = fields.Many2many('project.task', string='Tasks')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "A tag with the same name already exists."),
    ]

    def _get_project_tags_domain(self, domain, project_id):
        # TODO: Remove in master
        return domain

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if 'project_id' in self.env.context:
            tag_ids = self._name_search(limit=None)
            domain = expression.AND([domain, [('id', 'in', tag_ids)]])
        return super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    @api.model
    def search_read(self, domain=None, fields=None, offset=0, limit=None, order=None):
        if 'project_id' in self.env.context:
            tag_ids = self._name_search(limit=None)
            domain = expression.AND([domain, [('id', 'in', tag_ids)]])
            return self.arrange_tag_list_by_id(super().search_read(domain=domain, fields=fields, offset=offset, limit=limit), tag_ids)
        return super().search_read(domain=domain, fields=fields, offset=offset, limit=limit, order=order)

    @api.model
    def arrange_tag_list_by_id(self, tag_list, id_order):
        """arrange_tag_list_by_id re-order a list of record values (dict) following a given id sequence
           complexity: O(n)
           param:
                - tag_list: ordered (by id) list of record values, each record being a dict
                  containing at least an 'id' key
                - id_order: list of value (int) corresponding to the id of the records to re-arrange
           result:
                - Sorted list of record values (dict)
        """
        tags_by_id = {tag['id']: tag for tag in tag_list}
        return [tags_by_id[id] for id in id_order if id in tags_by_id]

    @api.model
    def _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):
        ids = []
        if not (name == '' and operator in ('like', 'ilike')):
            if args is None:
                args = []
            args += [('name', operator, name)]
        if self.env.context.get('project_id'):
            # optimisation for large projects, we look first for tags present on the last 1000 tasks of said project.
            # when not enough results are found, we complete them with a fallback on a regular search
            self.env.cr.execute("""
                SELECT DISTINCT project_tasks_tags.id
                FROM (
                    SELECT rel.project_tags_id AS id
                    FROM project_tags_project_task_rel AS rel
                    JOIN project_task AS task
                        ON task.id=rel.project_task_id
                        AND task.project_id=%(project_id)s
                    ORDER BY task.id DESC
                    LIMIT 1000
                ) AS project_tasks_tags
            """, {'project_id': self.env.context['project_id']})
            project_tasks_tags_domain = [('id', 'in', [row[0] for row in self.env.cr.fetchall()])]
            # we apply the args and limit to the ids we've already found
            ids += self.env['project.tags'].search(expression.AND([args, project_tasks_tags_domain]), limit=limit).ids
        if not limit or len(ids) < limit:
            limit = limit and limit - len(ids)
            ids += self.env['project.tags'].search(expression.AND([args, [('id', 'not in', ids)]]), limit=limit).ids
        return ids

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
    partner_email = fields.Char(related='partner_id.email')

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

from collections import defaultdict

from odoo import api, fields, models

class ProjectMilestone(models.Model):
    _name = 'project.milestone'
    _description = "Project Milestone"
    _inherit = ['mail.thread']
    _order = 'deadline, is_reached desc, name'

    def _get_default_project_id(self):
        return self.env.context.get('default_project_id') or self.env.context.get('active_id')

    name = fields.Char(required=True)
    project_id = fields.Many2one('project.project', required=True, default=_get_default_project_id, ondelete='cascade')
    deadline = fields.Date(tracking=True, copy=False)
    is_reached = fields.Boolean(string="Reached", default=False, copy=False)
    reached_date = fields.Date(compute='_compute_reached_date', store=True)
    task_ids = fields.One2many('project.task', 'milestone_id', 'Tasks')

    # computed non-stored fields
    is_deadline_exceeded = fields.Boolean(compute="_compute_is_deadline_exceeded")
    is_deadline_future = fields.Boolean(compute="_compute_is_deadline_future")
    task_count = fields.Integer('# of Tasks', compute='_compute_task_count', groups='project.group_project_milestone')
    can_be_marked_as_done = fields.Boolean(compute='_compute_can_be_marked_as_done', groups='project.group_project_milestone')

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

    @api.depends('task_ids.milestone_id')
    def _compute_task_count(self):
        task_read_group = self.env['project.task']._read_group([('milestone_id', 'in', self.ids), ('allow_milestones', '=', True)], ['milestone_id'], ['milestone_id'])
        task_count_per_milestone = {res['milestone_id'][0]: res['milestone_id_count'] for res in task_read_group}
        for milestone in self:
            milestone.task_count = task_count_per_milestone.get(milestone.id, 0)

    def _compute_can_be_marked_as_done(self):
        if not any(self._ids):
            for milestone in self:
                milestone.can_be_marked_as_done = not milestone.is_reached and all(milestone.task_ids.is_closed)
            return
        unreached_milestones = self.filtered(lambda milestone: not milestone.is_reached)
        (self - unreached_milestones).can_be_marked_as_done = False
        if unreached_milestones:
            task_read_group = self.env['project.task']._read_group(
                [('milestone_id', 'in', unreached_milestones.ids)],
                ['milestone_id', 'is_closed', 'task_count:count(id)'],
                ['milestone_id', 'is_closed'],
                lazy=False,
            )
            task_count_per_milestones = defaultdict(lambda: (0, 0))
            for res in task_read_group:
                opened_task_count, closed_task_count = task_count_per_milestones[res['milestone_id'][0]]
                if res['is_closed']:
                    closed_task_count += res['task_count']
                else:
                    opened_task_count += res['task_count']
                task_count_per_milestones[res['milestone_id'][0]] = opened_task_count, closed_task_count
            for milestone in unreached_milestones:
                opened_task_count, closed_task_count = task_count_per_milestones[milestone.id]
                milestone.can_be_marked_as_done = closed_task_count > 0 and not opened_task_count

    def toggle_is_reached(self, is_reached):
        self.ensure_one()
        self.update({'is_reached': is_reached})
        return self._get_data()

    def action_view_tasks(self):
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id('project.action_view_task_from_milestone')
        action['context'] = {'default_project_id': self.project_id.id, 'default_milestone_id': self.id}
        if self.task_count == 1:
            action['view_mode'] = 'form'
            action['res_id'] = self.task_ids.id
            if 'views' in action:
                action['views'] = [(view_id, view_type) for view_id, view_type in action['views'] if view_type == 'form']
        return action

    @api.model
    def _get_fields_to_export(self):
        return ['id', 'name', 'deadline', 'is_reached', 'reached_date', 'is_deadline_exceeded', 'is_deadline_future', 'can_be_marked_as_done']

    def _get_data(self):
        self.ensure_one()
        return {field: self[field] for field in self._get_fields_to_export()}

    def _get_data_list(self):
        return [ms._get_data() for ms in self]

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default is None:
            default = {}
        milestone_copy = super(ProjectMilestone, self).copy(default)
        if self.project_id.allow_milestones:
            milestone_mapping = self.env.context.get('milestone_mapping', {})
            milestone_mapping[self.id] = milestone_copy.id
        return milestone_copy

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
        help="If set, an email will be automatically sent to the customer when the project reaches this stage.")
    fold = fields.Boolean('Folded in Kanban',
        help="If enabled, this stage will be displayed as folded in the Kanban view of your projects. Projects in a folded stage are considered as closed.")

```

## File: models\project_task_recurrence.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
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
                raise ValidationError(_('You should select a least one day'))

    @api.constrains('repeat_interval')
    def _check_repeat_interval(self):
        if self.filtered(lambda t: t.repeat_interval <= 0):
            raise ValidationError(_('The interval should be greater than 0'))

    @api.constrains('repeat_number', 'repeat_type')
    def _check_repeat_number(self):
        if self.filtered(lambda t: t.repeat_type == 'after' and t.repeat_number <= 0):
            raise ValidationError(_('Should repeat at least once'))

    @api.constrains('repeat_type', 'repeat_until')
    def _check_repeat_until_date(self):
        today = fields.Date.today()
        if self.filtered(lambda t: t.repeat_type == 'until' and t.repeat_until < today):
            raise ValidationError(_('The end date should be in the future'))

    @api.constrains('repeat_unit', 'repeat_on_month', 'repeat_day', 'repeat_type', 'repeat_until')
    def _check_repeat_until_month(self):
        if self.filtered(lambda r: r.repeat_type == 'until' and r.repeat_unit == 'month' and r.repeat_until and r.repeat_on_month == 'date'
           and int(r.repeat_day) > r.repeat_until.day and monthrange(r.repeat_until.year, r.repeat_until.month)[1] != r.repeat_until.day):
            raise ValidationError(_('The end date should be after the day of the month or the last day of the month'))

    @api.model
    def _get_recurring_fields(self):
        return ['message_partner_ids', 'company_id', 'description', 'displayed_image_id', 'email_cc',
                'parent_id', 'partner_email', 'partner_id', 'partner_phone', 'planned_hours',
                'project_id', 'display_project_id', 'project_privacy_visibility', 'sequence', 'tag_ids', 'recurrence_id',
                'name', 'recurring_task', 'analytic_account_id', 'user_ids']

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
        self.env['project.task'].sudo().create(children)

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

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('repeat_number'):
                vals['recurrence_left'] = vals.get('repeat_number')
        recurrences = super().create(vals_list)
        recurrences._set_next_recurrence_date()
        return recurrences

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
    # Only used in project.task
    'to_define': 0,
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
                # `to_define` is not an option for self.status, here we actually want to default to `on_track`
                # the goal of `to_define` is for a project to start without an actual status.
                result['status'] = project.last_update_status if project.last_update_status != 'to_define' else 'on_track'
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
    @api.model_create_multi
    def create(self, vals_list):
        updates = super().create(vals_list)
        for update in updates:
            update.project_id.sudo().last_update_id = update
        return updates

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
        return self.env['ir.qweb']._render('project.project_update_default_description', self._get_template_values(project))

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
        if not project.allow_milestones:
            return {
                'show_section': False,
                'list': [],
                'updated': [],
                'last_update_date': None,
                'created': []
            }
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
    group_project_milestone = fields.Boolean('Milestones', implied_group='project.group_project_milestone', group='base.group_portal,base.group_user')

    # Analytic Accounting
    analytic_plan_id = fields.Many2one(
        comodel_name='account.analytic.plan',
        string="Default Plan",
        readonly=False,
        related='company_id.analytic_plan_id',
    )

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
            ("group_project_milestone", False): "allow_milestones",
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

        super().set_values()

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

        task_data = self.env['project.task']._read_group(
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
from . import mail_message
from . import project_milestone
from . import project_project_stage
from . import project_task_recurrence
# `project_task_stage_personal` has to be loaded before `project`
from . import project_task_stage_personal
from . import project
from . import project_collaborator
from . import project_update
from . import company
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

from odoo.addons.rating.models.rating_data import RATING_LIMIT_MIN, RATING_TEXT

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
        digits=(16, 2), readonly=True, group_operator="avg")
    working_days_open = fields.Float(string='Working Days to Assign',
        digits=(16, 2), readonly=True, group_operator="avg")
    delay_endings_days = fields.Float(string='Days to Deadline', digits=(16, 2), group_operator="avg", readonly=True)
    nbr = fields.Integer('# of Tasks', readonly=True)  # TDE FIXME master: rename into nbr_tasks
    working_hours_open = fields.Float(string='Working Hours to Assign', digits=(16, 2), readonly=True, group_operator="avg")
    working_hours_close = fields.Float(string='Working Hours to Close', digits=(16, 2), readonly=True, group_operator="avg")
    rating_last_value = fields.Float('Rating Value (/5)', group_operator="avg", readonly=True, groups="project.group_project_rating")
    rating_avg = fields.Float('Average Rating', readonly=True, group_operator='avg', groups="project.group_project_rating")
    priority = fields.Selection([
        ('0', 'Low'),
        ('1', 'High')
        ], readonly=True, string="Priority")
    state = fields.Selection([
            ('normal', 'In Progress'),
            ('blocked', 'Blocked'),
            ('done', 'Ready for Next Stage')
        ], string='Kanban State', readonly=True)
    company_id = fields.Many2one('res.company', string='Company', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    stage_id = fields.Many2one('project.task.type', string='Stage', readonly=True)
    is_closed = fields.Boolean("Closing Stage", readonly=True, help="Folded in Kanban stages are closing stages.")
    task_id = fields.Many2one('project.task', string='Tasks', readonly=True)
    active = fields.Boolean(readonly=True)
    tag_ids = fields.Many2many('project.tags', relation='project_tags_project_task_rel',
        column1='project_task_id', column2='project_tags_id',
        string='Tags', readonly=True)
    parent_id = fields.Many2one('project.task', string='Parent Task', readonly=True)
    ancestor_id = fields.Many2one('project.task', string="Ancestor Task", readonly=True)
    # We are explicitly not using a related field in order to prevent the recomputing caused by the depends as the model is a report.
    rating_last_text = fields.Selection(RATING_TEXT, string="Rating Last Text", compute="_compute_rating_last_text", search="_search_rating_last_text")
    personal_stage_type_ids = fields.Many2many('project.task.type', relation='project_task_user_rel',
        column1='task_id', column2='stage_id',
        string="Personal Stage", readonly=True)
    milestone_id = fields.Many2one('project.milestone', readonly=True)
    milestone_reached = fields.Boolean('Is Milestone Reached', readonly=True)
    milestone_deadline = fields.Date('Milestone Deadline', readonly=True)

    def _compute_rating_last_text(self):
        for task_analysis in self:
            task_analysis.rating_last_text = task_analysis.task_id.rating_last_text

    def _search_rating_last_text(self, operator, value):
        return [('task_id.rating_last_text', operator, value)]

    def _select(self):
        return """
                (select 1) AS nbr,
                t.id as id,
                t.id as task_id,
                t.active,
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
                t.parent_id as parent_id,
                t.ancestor_id as ancestor_id,
                t.stage_id as stage_id,
                t.is_closed as is_closed,
                t.kanban_state as state,
                t.milestone_id,
                pm.is_reached as milestone_reached,
                pm.deadline as milestone_deadline,
                NULLIF(t.rating_last_value, 0) as rating_last_value,
                AVG(rt.rating) as rating_avg,
                t.working_days_close as working_days_close,
                t.working_days_open  as working_days_open,
                t.working_hours_open as working_hours_open,
                t.working_hours_close as working_hours_close,
                (extract('epoch' from (t.date_deadline-(now() at time zone 'UTC'))))/(3600*24)  as delay_endings_days
        """

    def _group_by(self):
        return """
                t.id,
                t.active,
                t.create_date,
                t.date_assign,
                t.date_end,
                t.date_last_stage_update,
                t.date_deadline,
                t.project_id,
                t.ancestor_id,
                t.priority,
                t.name,
                t.company_id,
                t.partner_id,
                t.parent_id,
                t.stage_id,
                t.is_closed,
                t.kanban_state,
                t.rating_last_value,
                t.working_days_close,
                t.working_days_open,
                t.working_hours_open,
                t.working_hours_close,
                t.milestone_id,
                pm.is_reached,
                pm.deadline
        """

    def _from(self):
        return f"""
                project_task t
                    LEFT JOIN rating_rating rt ON rt.res_id = t.id
                        AND rt.res_model = 'project.task'
                        AND rt.consumed = True
                        AND rt.rating >= {RATING_LIMIT_MIN}
                    LEFT JOIN project_milestone pm ON pm.id = t.milestone_id
        """

    def _where(self):
        return """
                t.project_id IS NOT NULL
        """

    def init(self):
        tools.drop_view_if_exists(self._cr, self._table)
        self._cr.execute("""
    CREATE view %s as
         SELECT %s
           FROM %s
          WHERE %s
       GROUP BY %s
        """ % (self._table, self._select(), self._from(), self._where(), self._group_by()))

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
                <graph string="Tasks Analysis" sample="1" disable_linking="1">
                     <field name="project_id"/>
                     <field name="stage_id"/>
                     <field name="nbr" invisible="1"/>
                 </graph>
             </field>
        </record>

        <record id="report_project_task_user_view_tree" model="ir.ui.view">
            <field name="name">report.project.task.user.view.tree</field>
            <field name="model">report.project.task.user</field>
            <field name="arch" type="xml">
                <tree string="Tasks Analysis" create="false" editable="top" delete="false" edit="false">
                    <field name="name"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="project_id" options="{'no_open': True}" optional="show"/>
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
                    <field name="name" string="Task"/>
                    <field name="tag_ids"/>
                    <field name="user_ids" context="{'active_test': False}"/>
                    <field name="project_id"/>
                    <field name="milestone_id" groups="project.group_project_milestone"/>
                    <field name="ancestor_id" groups="project.group_subtask_project"/>
                    <field name="stage_id"/>
                    <field name="partner_id" operator="child_of"/>
                    <field name="active"/>
                    <field name="rating_last_text"/>
                    <field name="date_assign"/>
                    <field name="date_end"/>
                    <field name="date_deadline"/>
                    <field name="date_last_stage_update"/>
                    <filter string="My Tasks" name="my_tasks" domain="[('user_ids', 'in', uid)]"/>
                    <filter string="Followed Tasks" name="followed_by_me" domain="[('task_id.message_is_follower', '=', True)]"/>
                    <filter string="Unassigned" name="unassigned" domain="[('user_ids', '=', False)]"/>
                    <separator/>
                    <filter string="My Projects" name="own_projects" domain="[('project_id.user_id', '=', uid)]"/>
                    <filter string="My Favorite Projects" name="my_favorite_projects" domain="[('project_id.favorite_user_ids', 'in', [uid])]"/>
                    <separator/>
                    <filter string="High Priority" name="high_priority" domain="[('priority', '=', 1)]"/>
                    <filter string="Low Priority" name="low_priority" domain="[('priority', '=', 0)]"/>
                    <separator/>
                    <filter string="Open" name="open_tasks" domain="[('is_closed', '=', False)]"/>
                    <filter string="Closed" name="closed_tasks" domain="[('is_closed', '=', True)]"/>
                    <separator/>
                    <filter string="Late Milestones" name="late_milestone"
                        domain="[('project_id.allow_milestones', '=', True), ('is_closed', '=', False), ('milestone_reached', '=', False), ('milestone_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        groups="project.group_project_milestone"
                    />
                    <filter string="Late Tasks" name="late" domain="[('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d')), ('is_closed', '=', False)]"/>
                    <filter name="rating_satisfied" string="Satisfied" domain="[('rating_avg', '&gt;=', 3.66)]" groups="project.group_project_rating"/>
                    <filter name="rating_okay" string="Okay" domain="[('rating_avg', '&lt;', 3.66), ('rating_avg', '&gt;=', 2.33)]" groups="project.group_project_rating"/>
                    <filter name="dissatisfied" string="Dissatisfied" domain="[('rating_avg', '&lt;', 2.33), ('rating_last_value', '!=', 0)]" groups="project.group_project_rating"/>
                    <filter name="no_rating" string="No Rating" domain="[('rating_last_value', '=', 0)]" groups="project.group_project_rating"/>
                    <separator/>
                    <filter name="filter_date_deadline" date="date_deadline"/>
                    <filter name="filter_date_assign" date="date_assign"/>
                    <filter name="filter_date_last_stage_update" date="date_last_stage_update"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Extended Filters">
                        <field name="priority"/>
                        <field name="company_id" groups="base.group_multi_company"/>
                    </group>
                    <group expand="1" string="Group By">
                        <filter string="Stage" name="Stage" context="{'group_by': 'stage_id'}"/>
                        <filter string="Personal Stage" name="personal_stage" context="{'group_by': 'personal_stage_type_ids'}"/>
                        <filter string="Assignees" name="User" context="{'group_by': 'user_ids'}"/>
                        <filter string="Ancestor Task" name="groupby_ancestor_task" context="{'group_by': 'ancestor_id'}" groups="project.group_subtask_project"/>
                        <filter string="Milestone" name="milestone" context="{'group_by': 'milestone_id'}" groups="project.group_project_milestone"/>
                        <filter string="Customer" name="Customer" context="{'group_by': 'partner_id'}"/>
                        <filter string="Kanban State" name="kanban_state" context="{'group_by': 'state'}"/>
                        <filter string="Deadline" name="deadline" context="{'group_by': 'date_deadline'}"/>
                        <filter string="Creation Date" name="group_create_date" context="{'group_by': 'create_date'}"/>
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
                    Analyze the progress of your projects and the performance of your employees.
                </p>
            </field>
        </record>

</odoo>

```

## File: report\project_task_burndown_chart_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.models import regex_field_agg, VALID_AGGREGATE_FUNCTIONS
from odoo.exceptions import UserError
from odoo.osv.expression import AND_OPERATOR, OR_OPERATOR, NOT_OPERATOR, DOMAIN_OPERATORS, FALSE_LEAF, TRUE_LEAF, normalize_domain
from odoo.tools import OrderedSet


def remove_domain_leaf(domain, fields_to_remove):
    """ Make the provided domain insensitive to the fields provided in fields_to_remove. Fields that are part of
    `fields_to_remove` are replaced by either a `FALSE_LEAF` or a `TRUE_LEAF` in order to ensure the evaluation of the
    complete domain.

    :param domain: The domain to process.
    :param fields_to_remove: List of fields the domain has to be insensitive to.
    :return: The insensitive domain.
    """
    def _process_leaf(elements, index, operator, new_domain):
        leaf = elements[index]
        if len(leaf) == 3:
            if leaf[0] in fields_to_remove:
                if operator == AND_OPERATOR:
                    new_domain.append(TRUE_LEAF)
                elif operator == OR_OPERATOR:
                    new_domain.append(FALSE_LEAF)
            else:
                new_domain.append(leaf)
            return 1
        elif len(leaf) == 1 and leaf in DOMAIN_OPERATORS:
            # Special case to avoid OR ('|') that can never resolve to true
            if leaf == OR_OPERATOR \
                    and len(elements[index + 1]) == 3 and len(elements[index + 2]) == 3 \
                    and elements[index + 1][0] in fields_to_remove and elements[index + 2][0] in fields_to_remove:
                new_domain.append(TRUE_LEAF)
                return 3
            new_domain.append(leaf)
            if leaf[0] == NOT_OPERATOR:
                return 1 + _process_leaf(elements, index + 1, '&', new_domain)
            first_leaf_skip = _process_leaf(elements, index + 1, leaf, new_domain)
            second_leaf_skip = _process_leaf(elements, index + 1 + first_leaf_skip, leaf, new_domain)
            return 1 + first_leaf_skip + second_leaf_skip
        return 0

    if len(domain) == 0:
        return domain
    new_domain = []
    _process_leaf(normalize_domain(domain), 0, AND_OPERATOR, new_domain)
    return new_domain


class ReportProjectTaskBurndownChart(models.AbstractModel):
    _name = 'project.task.burndown.chart.report'
    _description = 'Burndown Chart'
    _auto = False
    _order = 'date'

    planned_hours = fields.Float(string='Allocated Hours', readonly=True)
    date = fields.Datetime('Date', readonly=True)
    date_assign = fields.Datetime(string='Assignment Date', readonly=True)
    date_deadline = fields.Date(string='Deadline', readonly=True)
    display_project_id = fields.Many2one('project.project', readonly=True)
    is_closed = fields.Boolean("Closing Stage", readonly=True)
    milestone_id = fields.Many2one('project.milestone', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    project_id = fields.Many2one('project.project', readonly=True)
    stage_id = fields.Many2one('project.task.type', readonly=True)
    user_ids = fields.Many2many('res.users', relation='project_task_user_rel', column1='task_id', column2='user_id',
                                string='Assignees', readonly=True)

    # Fake field required as used in the filters. It will however be managed through the `project.task` model.
    has_late_and_unreached_milestone = fields.Boolean(readonly=True)

    # This variable is used in order to distinguish conditions that can be set on `project.task` and thus being used
    # at a lower level than the "usual" query made by the `read_group_raw`. Indeed, the domain applied on those fields
    # will be performed on a `CTE` that will be later use in the `SQL` in order to limit the subset of data that is used
    # in the successive `GROUP BY` statements.
    task_specific_fields = [
        'date_assign',
        'date_deadline',
        'display_project_id',
        'has_late_and_unreached_milestone',
        'is_closed',
        'milestone_id',
        'partner_id',
        'project_id',
        'stage_id',
        'user_ids',
    ]

    def _get_group_by_SQL(self, task_specific_domain, count_field, select_terms, from_clause, where_clause,
                          where_clause_params, groupby_terms, orderby_terms, limit, offset, groupby, annotated_groupbys,
                          prefix_term, prefix_terms):
        """ Prepare and return the SQL to be used for the read_group. """

        # Build the query on `project.task` with the domain fields that are linked to that model. This is done in order
        # to be able to reduce the number of treated records in the query by limiting them to the one corresponding to
        # the ids that are returned from this sub query.
        project_task_query = self.env['project.task']._where_calc(task_specific_domain)
        project_task_from_clause, project_task_where_clause, project_task_where_clause_params = project_task_query.get_sql()

        # Get the stage_id `ir.model.fields`'s id in order to inject it directly in the query and avoid having to join
        # on `ir_model_fields` table.
        IrModelFieldsSudo = self.env['ir.model.fields'].sudo()
        field_id = IrModelFieldsSudo.search([('name', '=', 'stage_id'), ('model', '=', 'project.task')]).id

        # Get the date aggregation SQL statement in order to be able to inject it in the SQL.
        date_group_by_field = next(filter(lambda gb: gb.startswith('date'), groupby))
        date_annotated_groupby = [
            annotated_groupby for annotated_groupby in annotated_groupbys
            if annotated_groupby['groupby'] == date_group_by_field
        ][0]
        date_begin, date_end = (
            date_annotated_groupby['qualified_field'].replace(
                '"%s"."%s"' % (self._table, date_annotated_groupby['field']), '"%s_%s"' % (date_annotated_groupby['field'], field)
            )
            for field in ['begin', 'end']
        )

        # Insert `WHERE` clause parameter that apply on `project_task` prior to the one that apply on
        # `project_task_burndown_chart_report` as the `project_task` CTE is placed at the beginning of the `SQL`.
        for param in reversed(project_task_where_clause_params):
            where_clause_params.insert(0, param)

        # Computes the interval which needs to be used in the `SQL` depending on the date group by interval.
        if date_annotated_groupby['groupby'].split(':')[1] != 'quarter':
            interval = '1 %s' % date_annotated_groupby['groupby'].split(':')[1]
        else:
            interval = '3 month'

        burndown_chart_query = """
              WITH task_ids AS (
                 SELECT id
                 FROM %(task_query_from)s
                 %(task_query_where)s
              ),
              all_stage_task_moves AS (
                 SELECT count(*) as %(count_field)s,
                        sum(planned_hours) as planned_hours,
                        project_id,
                        display_project_id,
                        %(date_begin)s as date_begin,
                        %(date_end)s as date_end,
                        stage_id
                   FROM (
                            -- Gathers the stage_ids history per task_id. This query gets:
                            -- * All changes except the last one for those for which we have at least a mail
                            --   message and a mail tracking value on project.task stage_id.
                            -- * The stage at creation for those for which we do not have any mail message and a
                            --   mail tracking value on project.task stage_id.
                            SELECT DISTINCT task_id,
                                   planned_hours,
                                   project_id,
                                   display_project_id,
                                   %(date_begin)s as date_begin,
                                   %(date_end)s as date_end,
                                   first_value(stage_id) OVER task_date_begin_window AS stage_id
                              FROM (
                                     SELECT pt.id as task_id,
                                            pt.planned_hours,
                                            pt.project_id,
                                            pt.display_project_id,
                                            COALESCE(LAG(mm.date) OVER (PARTITION BY mm.res_id ORDER BY mm.id), pt.create_date) as date_begin,
                                            CASE WHEN mtv.id IS NOT NULL THEN mm.date
                                                ELSE (now() at time zone 'utc')::date + INTERVAL '%(interval)s'
                                            END as date_end,
                                            CASE WHEN mtv.id IS NOT NULL THEN mtv.old_value_integer
                                               ELSE pt.stage_id
                                            END as stage_id
                                       FROM project_task pt
                                                LEFT JOIN (
                                                    mail_message mm
                                                        JOIN mail_tracking_value mtv ON mm.id = mtv.mail_message_id
                                                                                     AND mtv.field = %(field_id)s
                                                                                     AND mm.model='project.task'
                                                                                     AND mm.message_type = 'notification'
                                                        JOIN project_task_type ptt ON ptt.id = mtv.old_value_integer
                                                ) ON mm.res_id = pt.id
                                      WHERE pt.active=true AND pt.id IN (SELECT id from task_ids)
                                   ) task_stage_id_history
                          GROUP BY task_id,
                                   planned_hours,
                                   project_id,
                                   display_project_id,
                                   %(date_begin)s,
                                   %(date_end)s,
                                   stage_id
                            WINDOW task_date_begin_window AS (PARTITION BY task_id, %(date_begin)s)
                          UNION ALL
                            -- Gathers the current stage_ids per task_id for those which values changed at least
                            -- once (=those for which we have at least a mail message and a mail tracking value
                            -- on project.task stage_id).
                            SELECT pt.id as task_id,
                                   pt.planned_hours,
                                   pt.project_id,
                                   pt.display_project_id,
                                   last_stage_id_change_mail_message.date as date_begin,
                                   (now() at time zone 'utc')::date + INTERVAL '%(interval)s' as date_end,
                                   pt.stage_id as old_value_integer
                              FROM project_task pt
                                   JOIN project_task_type ptt ON ptt.id = pt.stage_id
                                   JOIN LATERAL (
                                       SELECT mm.date
                                       FROM mail_message mm
                                       JOIN mail_tracking_value mtv ON mm.id = mtv.mail_message_id
                                       AND mtv.field = %(field_id)s
                                       AND mm.model='project.task'
                                       AND mm.message_type = 'notification'
                                       AND mm.res_id = pt.id
                                       ORDER BY mm.id DESC
                                       FETCH FIRST ROW ONLY
                                   ) AS last_stage_id_change_mail_message ON TRUE
                             WHERE pt.active=true AND pt.id IN (SELECT id from task_ids)
                        ) AS project_task_burndown_chart
               GROUP BY planned_hours,
                        project_id,
                        display_project_id,
                        %(date_begin)s,
                        %(date_end)s,
                        stage_id
              )
              SELECT (project_id*10^13 + stage_id*10^7 + to_char(date, 'YYMMDD')::integer)::bigint as id,
                     planned_hours,
                     project_id,
                     display_project_id,
                     stage_id,
                     date,
                     %(count_field)s
                FROM all_stage_task_moves t
                         JOIN LATERAL generate_series(t.date_begin, t.date_end-INTERVAL '1 day', '%(interval)s')
                            AS date ON TRUE
        """ % {
            'task_query_from': project_task_from_clause,
            'task_query_where': prefix_term('WHERE', project_task_where_clause),
            'count_field': count_field,
            'date_begin': date_begin,
            'date_end': date_end,
            'interval': interval,
            'field_id': field_id,
        }

        # Replace, in the `FROM` clause generated on `project_task_burndown_chart_report`, the
        # `project_task_burndown_chart_report` table name by the burndown_chart_query `SQL` aliased as
        # `project_task_burndown_chart_report`.
        from_clause = from_clause.replace('"project_task_burndown_chart_report"', '(%s) AS "project_task_burndown_chart_report"' % burndown_chart_query, 1)

        return """
            SELECT min("%(table)s".id) AS id, sum(%(table)s.%(count_field)s) AS "%(count_field)s" %(extra_fields)s
            FROM %(from)s
            %(where)s
            %(groupby)s
            %(orderby)s
            %(limit)s
            %(offset)s
        """ % {
            'table': self._table,
            'count_field': count_field,
            'extra_fields': prefix_terms(',', select_terms),
            'from': from_clause,
            'where': prefix_term('WHERE', where_clause),
            'groupby': prefix_terms('GROUP BY', groupby_terms),
            'orderby': prefix_terms('ORDER BY', orderby_terms),
            'limit': prefix_term('LIMIT', int(limit) if limit else None),
            'offset': prefix_term('OFFSET', int(offset) if limit else None),
        }

    @api.model
    def _validate_group_by(self, groupby):
        """ Check that the both `date` and `stage_id` are part of `group_by`, otherwise raise a `UserError`.

        :param groupby: List of group by fields.
        """
        stage_id_in_groupby = False
        date_in_groupby = False

        for gb in groupby:
            if gb.startswith('date'):
                date_in_groupby = True
            else:
                if gb == 'stage_id':
                    stage_id_in_groupby = True

        if not date_in_groupby or not stage_id_in_groupby:
            raise UserError(_('The view must be grouped by date and by stage_id'))

    @api.model
    def _determine_domains(self, domain):
        """ Compute two separated domain from the provided one:
        * A domain that only contains fields that are specific to `project.task.burndown.chart.report`
        * A domain that only contains fields that are specific to `project.task`

        Fields that are not part of the constraint are replaced by either a `FALSE_LEAF` or a `TRUE_LEAF` in order
        to ensure the complete domain evaluation. See `remove_domain_leaf` for more details.

        :param domain: The domain that has been passed to the read_group.
        :return: A tuple containing the non `project.task` specific domain and the `project.task` specific domain.
        """
        burndown_chart_specific_fields = list(set(self._fields) - set(self.task_specific_fields))
        task_specific_domain = remove_domain_leaf(domain, burndown_chart_specific_fields)
        non_task_specific_domain = remove_domain_leaf(domain, self.task_specific_fields)
        return non_task_specific_domain, task_specific_domain

    @api.model
    def _read_group_raw(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        """ Although not being a good practice, this code is, for a big part, duplicated from `read_group_raw` from
        `models.py`. In order to be able to use the report on big databases, it is necessary to inject `WHERE`
        statements at the lowest levels in the report `SQL`. As a result, using a view was no more an option as
        `Postgres` could not optimise the `SQL`.
        The code of `fill_temporal` has been removed from what's available in `models.py` as it is not relevant in the
        context of the Burndown Chart. Indeed, series are generated so no empty are returned by the `SQL`, except if
        explicitly specified in the domain through the `date` field, which is then expected.
        """

        # --- Below code is custom

        self._validate_group_by(groupby)
        burndown_specific_domain, task_specific_domain = self._determine_domains(domain)

        # --- Below code is from models.py read_group_raw

        self.check_access_rights('read')
        query = self._where_calc(burndown_specific_domain)
        fields = fields or [f.name for f in self._fields.values() if f.store]

        groupby = [groupby] if isinstance(groupby, str) else list(OrderedSet(groupby))
        groupby_list = groupby[:1] if lazy else groupby
        annotated_groupbys = [self._read_group_process_groupby(gb, query) for gb in groupby_list]
        groupby_fields = [g['field'] for g in annotated_groupbys]
        order = orderby or ','.join([g for g in groupby_list])
        groupby_dict = {gb['groupby']: gb for gb in annotated_groupbys}

        self._apply_ir_rules(query, 'read')
        for gb in groupby_fields:
            if gb not in self._fields:
                raise UserError(_("Unknown field %r in 'groupby'", gb))
            if not self._fields[gb].base_field.groupable:
                raise UserError(_(
                    "Field %s is not a stored field, only stored fields (regular or "
                    "many2many) are valid for the 'groupby' parameter", self._fields[gb],
                ))

        aggregated_fields = []
        select_terms = []
        fnames = []                     # list of fields to flush

        for fspec in fields:
            if fspec == 'sequence':
                continue
            if fspec == '__count':
                # the web client sometimes adds this pseudo-field in the list
                continue

            match = regex_field_agg.match(fspec)
            if not match:
                raise UserError(_("Invalid field specification %r.", fspec))

            name, func, fname = match.groups()
            if func:
                # we have either 'name:func' or 'name:func(fname)'
                fname = fname or name
                field = self._fields.get(fname)
                if not field:
                    raise ValueError(_("Invalid field %r on model %r", (fname, self._name)))
                if not (field.base_field.store and field.base_field.column_type):
                    raise UserError(_("Cannot aggregate field %r.", fname))
                if func not in VALID_AGGREGATE_FUNCTIONS:
                    raise UserError(_("Invalid aggregation function %r.", func))
            else:
                # we have 'name', retrieve the aggregator on the field
                field = self._fields.get(name)
                if not field:
                    raise ValueError(_("Invalid field %r on model %r", (name, self._name)))
                if not (field.base_field.store and
                        field.base_field.column_type and field.group_operator):
                    continue
                func, fname = field.group_operator, name

            fnames.append(fname)

            if fname in groupby_fields:
                continue
            if name in aggregated_fields:
                raise UserError(_("Output name %r is used twice.", name))
            aggregated_fields.append(name)

            expr = self._inherits_join_calc(self._table, fname, query)
            if func.lower() == 'count_distinct':
                term = 'COUNT(DISTINCT %s) AS "%s"' % (expr, name)
            else:
                term = '%s(%s) AS "%s"' % (func, expr, name)
            select_terms.append(term)

        for gb in annotated_groupbys:
            select_terms.append('%s as "%s" ' % (gb['qualified_field'], gb['groupby']))

        # --- Below code is custom
        # --- As the report is base on `project.task` we flush that specific model

        # self._flush_search(domain, fields=fnames + groupby_fields)
        self.env['project.task']._flush_search(task_specific_domain, fields=self.task_specific_fields)

        # --- Below code is from models.py read_group_raw

        groupby_terms, orderby_terms = self._read_group_prepare(order, aggregated_fields, annotated_groupbys, query)
        from_clause, where_clause, where_clause_params = query.get_sql()
        if lazy and (len(groupby_fields) >= 2 or not self._context.get('group_by_no_leaf')):
            count_field = groupby_fields[0] if len(groupby_fields) >= 1 else '_'
        else:
            count_field = '_'
        count_field += '_count'

        prefix_terms = lambda prefix, terms: (prefix + " " + ",".join(terms)) if terms else ''
        prefix_term = lambda prefix, term: ('%s %s' % (prefix, term)) if term else ''

        # --- Below code is custom

        query = self._get_group_by_SQL(task_specific_domain, count_field, select_terms, from_clause, where_clause,
                                       where_clause_params, groupby_terms, orderby_terms, limit, offset, groupby,
                                       annotated_groupbys, prefix_term, prefix_terms)

        # --- Below code is from models.py read_group_raw

        self._cr.execute(query, where_clause_params)
        fetched_data = self._cr.dictfetchall()

        if not groupby_fields:
            return fetched_data

        self._read_group_resolve_many2x_fields(fetched_data, annotated_groupbys)

        data = [{k: self._read_group_prepare_data(k, v, groupby_dict) for k, v in r.items()} for r in fetched_data]

        result = [self._read_group_format_result(d, annotated_groupbys, groupby, domain) for d in data]

        # --- Below code is custom
        # --- We removed fill_temporal handling as not relevant in the context of the Burndown Chart

        # --- Below code is from models.py read_group_raw

        if lazy:
            # Right now, read_group only fill results in lazy mode (by default).
            # If you need to have the empty groups in 'eager' mode, then the
            # method _read_group_fill_results need to be completely reimplemented
            # in a sane way
            result = self._read_group_fill_results(
                domain, groupby_fields[0], groupby[len(annotated_groupbys):],
                aggregated_fields, count_field, result, read_group_order=order,
            )
        return result

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
                <field name="stage_id" />
                <field name="project_id" />
                <field name="user_ids" />
                <field name="milestone_id" groups="project.group_project_milestone"/>
                <field name="date_assign"/>
                <field name="date_deadline"/>
                <field name="partner_id" filter_domain="[('partner_id', 'child_of', self)]"/>
                <separator/>
                <filter name="filter_date" date="date" string="Date" default_period="this_year,last_year" />
                <filter name="filter_date_deadline" date="date_deadline"/>
                <filter name="filter_date_assign" date="date_assign"/>
                <filter string="Last Month" invisible="1" name="last_month" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=30)).strftime('%Y-%m-%d'))]"/>
                <filter string="Open tasks" name="open_tasks" domain="[('is_closed', '=', False)]"/>
                <filter string="Late Milestones" name="late_milestone" domain="[('is_closed', '=', False), ('has_late_and_unreached_milestone', '=', True)]" groups="project.group_project_milestone"/>
                <group expand="0" string="Group By">
                    <filter string="Date" name="date" context="{'group_by': 'date'}" />
                    <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}" invisible="1"/>
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
            </graph>
        </field>
    </record>

    <record id="action_project_task_burndown_chart_report" model="ir.actions.act_window">
        <field name="name">Burndown Chart</field>
        <field name="res_model">project.task.burndown.chart.report</field>
        <field name="view_mode">graph</field>
        <field name="search_view_id" ref="project_task_burndown_chart_report_view_search"/>
        <field name="context">{'search_default_project_id': active_id, 'search_default_date': 1, 'search_default_stage': 1, 'search_default_filter_date': 1, 'search_default_open_tasks': 1}</field>
        <field name="domain">[('display_project_id', '!=', False)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No data yet!
            </p>
            <p>Analyze how quickly your team is completing your project's tasks and check if everything is progressing according to plan.</p>
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
access_report_project_task_user_project_user,report.project.task.user.project.user,model_report_project_task_user,project.group_project_user,1,0,0,0
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
access_project_task_type_delete_wizard,project.task.type.delete.wizard,model_project_task_type_delete_wizard,project.group_project_manager,1,1,1,1
access_project_task_recurrence,project.task.recurrence,model_project_task_recurrence,project.group_project_user,1,1,1,1
project.access_project_task_burndown_chart_report,access_project_task_burndown_chart_report,project.model_project_task_burndown_chart_report,project.group_project_manager,1,1,1,1
project.access_project_task_burndown_chart_report_user,access_project_task_burndown_chart_report_user,project.model_project_task_burndown_chart_report,project.group_project_user,1,0,0,0
access_project_update_user,project.update.user,model_project_update,base.group_user,1,0,0,0
access_project_update_portal,project.update.portal,model_project_update,base.group_portal,0,0,0,0
access_project_update_project_user,project.update.project.user,model_project_update,project.group_project_user,1,1,1,1
access_project_update_project_manager,project.update.project.manager,model_project_update,project.group_project_manager,1,1,1,1
access_project_milestone_user,project.milestone.user,model_project_milestone,base.group_user,1,0,0,0
access_project_milestone_portal,project.milestone.portal,model_project_milestone,base.group_portal,1,0,0,0
access_project_milestone_project_user,project.milestone.project.user,model_project_milestone,project.group_project_user,1,1,1,1
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

    <record id="group_project_milestone" model="res.groups">
        <field name="name">Use Milestones</field>
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
        <field name="name">Project/Task: project manager: see all tasks linked to a project or its own tasks</field>
        <field name="model_id" ref="model_project_task"/>
        <field name="domain_force">[
            '|', ('project_id', '!=', False),
                 ('user_ids', 'in', user.id),
        ]</field>
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

    <record model="ir.rule" id="report_project_task_user_rule">
        <field name="name">Tasks Analysis: project visibility User</field>
        <field name="model_id" ref="model_report_project_task_user"/>
        <field name="domain_force">[
        '|',
            ('project_id.privacy_visibility', '!=', 'followers'),
            '|',
                ('project_id.message_partner_ids', 'in', [user.partner_id.id]),
                '|',
                    ('task_id.message_partner_ids', 'in', [user.partner_id.id]),
                    ('user_ids', 'in', user.id),
        ]</field>
        <field name="groups" eval="[(4,ref('project.group_project_user'))]"/>
    </record>

    <record model="ir.rule" id="report_project_task_manager_rule">
        <field name="name">Tasks Analysis: project visibility Manager</field>
        <field name="model_id" ref="model_report_project_task_user"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('project.group_project_manager'))]"/>
    </record>

    <record id="update_visibility_project_admin" model="ir.rule">
        <field name="name">Project updates : Project user can see all project updates</field>
        <field name="model_id" ref="project.model_project_update"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('project.group_project_manager'))]"/>
    </record>

    <record model="ir.rule" id="burndown_chart_project_user_rule">
        <field name="name">Burndown chart: project visibility User</field>
        <field name="model_id" ref="model_project_task_burndown_chart_report"/>
        <field name="domain_force">[
        '|',
            ('project_id.privacy_visibility', '!=', 'followers'),
            '|',
                ('project_id.message_partner_ids', 'in', [user.partner_id.id]),
                ('user_ids', 'in', user.id),
        ]</field>
        <field name="groups" eval="[(4,ref('project.group_project_user'))]"/>
    </record>

    <record model="ir.rule" id="burndown_chart_project_manager_rule">
        <field name="name">Burndown chart: project visibility User</field>
        <field name="model_id" ref="model_project_task_burndown_chart_report"/>
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

    <record id="project_milestone_rule_portal_project_sharing" model="ir.rule">
        <field name="name">Project/milestone portal users: portal user can read with project sharing feature</field>
        <field name="model_id" ref="project.model_project_milestone"/>
        <field name="domain_force">[
            ('project_id.privacy_visibility', '=', 'portal'),
            ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),
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

## File: static\src\components\project_control_panel\project_control_panel.js

```javascript
/** @odoo-module **/

import { ControlPanel } from "@web/search/control_panel/control_panel";
import { useService } from "@web/core/utils/hooks";

const { onWillStart } = owl;

export class ProjectControlPanel extends ControlPanel {
    setup() {
        super.setup();
        this.orm = useService("orm");
        this.user = useService("user");
        const { active_id, show_project_update } = this.env.searchModel.globalContext;
        this.showProjectUpdate = this.env.config.viewType === "form" || show_project_update;
        this.projectId = this.showProjectUpdate ? active_id : false;

        onWillStart(async () => {
            if (this.showProjectUpdate) {
                await this.loadData();
            }
        });
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

## File: static\src\components\project_control_panel\project_control_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectControlPanelContentBadge" owl="1">
        <t t-tag="isProjectUser ? 'button' : 'span'" class="badge border d-flex p-2 ms-2 bg-view" data-hotkey="y">
            <span t-attf-class="o_status_bubble o_color_bubble_{{data.color}}"/>
            <span t-att-class="'fw-normal ms-1' + (data.color === 0 ? ' text-muted' : '')" t-esc="data.status"/>
        </t>
    </t>

    <t t-name="project.ProjectControlPanelContent" owl="1">
        <t t-if="showProjectUpdate">
            <li t-if="isProjectUser" class="o_project_updates_breadcrumb ps-3" t-on-click="onStatusClick">
                <t t-call="project.ProjectControlPanelContentBadge"></t>
            </li>
            <li t-else="" class="o_project_updates_breadcrumb ps-3">
                <t t-call="project.ProjectControlPanelContentBadge"></t>
            </li>
        </t>
    </t>

    <t t-name="project.Breadcrumbs" t-inherit="web.Breadcrumbs" t-inherit-mode="primary" owl="1">
        <xpath expr="//ol" position="inside">
            <t t-call="project.ProjectControlPanelContent"/>
        </xpath>
    </t>

    <t t-name="project.Breadcrumbs.Small" t-inherit="web.Breadcrumbs.Small" t-inherit-mode="primary" owl="1">
        <xpath expr="//ol" position="inside">
            <t t-call="project.ProjectControlPanelContent"/>
        </xpath>
    </t>

    <t t-name="project.ProjectControlPanel.Regular" t-inherit="web.ControlPanel.Regular" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-call='web.Breadcrumbs']" position="replace">
            <t t-call="project.Breadcrumbs"/>
        </xpath>
    </t>

    <t t-name="project.ProjectControlPanel.Small" t-inherit="web.ControlPanel.Small" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-call='web.Breadcrumbs.Small']" position="replace">
            <t t-call="project.Breadcrumbs.Small"/>
        </xpath>
    </t>

    <t t-name="project.ProjectControlPanel" t-inherit="web.ControlPanel" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-call='web.ControlPanel.Regular']" position="replace">
            <t t-call="project.ProjectControlPanel.Regular"/>
        </xpath>
        <xpath expr="//t[@t-call='web.ControlPanel.Small']" position="replace">
            <t t-call="project.ProjectControlPanel.Small"/>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\project_private_task_many2one_field\project_private_task_many2one_field.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';
import { Many2OneField } from '@web/views/fields/many2one/many2one_field';

export class ProjectPrivateTaskMany2OneField extends Many2OneField { }
ProjectPrivateTaskMany2OneField.template = 'project.ProjectPrivateTaskMany2OneField';

registry.category('fields').add('project_private_task', ProjectPrivateTaskMany2OneField);

```

## File: static\src\components\project_private_task_many2one_field\project_private_task_many2one_field.xml

```xml
<templates>

    <t t-name="project.ProjectPrivateTaskMany2OneField" t-inherit="web.Many2OneField" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-if='!props.canOpen']/span" position="attributes">
            <attribute name="t-if">props.value</attribute>
        </xpath>
        <xpath expr="//t[@t-if='!props.canOpen']/span" position="after">
            <span t-else="" class="text-danger fst-italic text-muted"><i class="fa fa-lock"></i> Private</span>
        </xpath>
        <xpath expr="//t[@t-else='']/a" position="attributes">
            <attribute name="t-if">displayName</attribute>
        </xpath>
        <xpath expr="//t[@t-else='']/a" position="after">
            <span t-else="" class="text-danger fst-italic text-muted"><i class="fa fa-lock"></i> Private</span>
        </xpath>
        <xpath expr="//div[hasclass('o_field_many2one_selection')]" position="attributes">
            <attribute name="class" separator=" " add="project_private_task_many2one_field"></attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\project_right_side_panel\project_right_side_panel.js

```javascript
/** @odoo-module */

import { useService } from '@web/core/utils/hooks';
import { formatFloat } from '@web/views/fields/formatters';
import { session } from '@web/session';
import { ViewButton } from '@web/views/view_button/view_button';
import { FormViewDialog } from '@web/views/view_dialogs/form_view_dialog';

import { ProjectRightSidePanelSection } from './components/project_right_side_panel_section';
import { ProjectMilestone } from './components/project_milestone';
import { ProjectProfitability } from './components/project_profitability';

const { Component, onWillStart, useState } = owl;

export class ProjectRightSidePanel extends Component {
    setup() {
        this.orm = useService('orm');
        this.actionService = useService('action');
        this.dialog = useService('dialog');
        this.state = useState({
            data: {
                milestones: {
                    data: [],
                },
                profitability_items: {
                    costs: { data: [], total: { billed: 0.0, to_bill: 0.0 } },
                    revenues: { data: [], total: { invoiced: 0.0, to_invoice: 0.0 } },
                },
                user: {},
                currency_id: false,
            }
        });

        onWillStart(() => this.loadData());
    }

    get context() {
        return this.props.context;
    }

    get domain() {
        return this.props.domain;
    }

    get projectId() {
        return this.context.active_id;
    }

    get currencyId() {
        return this.state.data.currency_id;
    }

    get sectionNames() {
        return {
            'milestones': this.env._t('Milestones'),
            'profitability': this.env._t('Profitability'),
        };
    }

    get showProjectProfitability() {
        return !!this.state.data.profitability_items
            && (
                this.state.data.profitability_items.revenues.data.length > 0
                || this.state.data.profitability_items.costs.data.length > 0
            );
    }

    formatFloat(value) {
        return formatFloat(value, { digits: [false, 1] });
    }

    formatMonetary(value, options = {}) {
        const valueFormatted = formatFloat(value, {
            ...options,
            'digits': [false, 0],
            'noSymbol': true,
        });
        const currency = session.currencies[this.currencyId];
        if (!currency) {
            return valueFormatted;
        }
        if (currency.position === "after") {
            return `${valueFormatted}\u00A0${currency.symbol}`;
        } else {
            return `${currency.symbol}\u00A0${valueFormatted}`;
        }
    }

    async loadData() {
        if (!this.projectId) { // If this is called from notif, multiples updates but no specific project
            return {};
        }
        const data = await this.orm.call(
            'project.project',
            'get_panel_data',
            [[this.projectId]],
            { context: this.context },
        );
        this.state.data = data;
        return data;
    }

    async loadMilestones() {
        const milestones = await this.orm.call(
            'project.project',
            'get_milestones',
            [[this.projectId]],
            { context: this.context },
        );
        this.state.data.milestones = milestones;
        return milestones;
    }

    addMilestone() {
        const context = {
            ...this.context,
            'default_project_id': this.projectId,
        };
        this.openFormViewDialog({
            context,
            title: this.env._t('New Milestone'),
            resModel: 'project.milestone',
            onRecordSaved: async () => {
                await this.loadMilestones();
            },
        });
    }

    async openFormViewDialog(params, options = {}) {
        this.dialog.add(FormViewDialog, params, options);
    }

    async onProjectActionClick(params) {
        this.actionService.doActionButton({
            type: 'action',
            resId: this.projectId,
            context: this.context,
            resModel: 'project.project',
            ...params,
        });
    }

    _getStatButtonClickParams(statButton) {
        return {
            type: statButton.action_type,
            name: statButton.action,
            context: statButton.additional_context || '{}',
        };
    }

    _getStatButtonRecordParams() {
        return {
            resId: this.projectId,
            context: JSON.stringify(this.context),
            resModel: 'project.project',
        };
    }
}

ProjectRightSidePanel.components = { ProjectRightSidePanelSection, ProjectMilestone, ViewButton, ProjectProfitability };
ProjectRightSidePanel.template = 'project.ProjectRightSidePanel';
ProjectRightSidePanel.props = {
    context: Object,
    domain: Array,
};

```

## File: static\src\components\project_right_side_panel\project_right_side_panel.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="project.ProjectRightSidePanel" owl="1">
        <div t-if="projectId" class="o_rightpanel pt-0 bg-view border-start overflow-auto">
            <ProjectRightSidePanelSection
                name="'stat_buttons'"
                header="false"
                show="!!state.data.buttons"
            >
                <div class="o_form_view">
                    <div class="oe_button_box o-form-buttonbox d-flex flex-wrap">
                        <t t-foreach="state.data.buttons" t-as="button" t-key="button.action">
                            <ViewButton
                                t-if="button.show"
                                defaultRank="'oe_stat_button'"
                                className="'h-auto py-2 border border-start-0 border-top-0 text-start'"
                                icon="`fa-${button.icon}`"
                                title="button.text"
                                clickParams="_getStatButtonClickParams(button)"
                                record="_getStatButtonRecordParams()"
                            >
                                <t t-set-slot="contents">
                                    <div class="o_field_widget o_stat_info">
                                        <span class="o_stat_value text-start">
                                            <t t-esc="button.number"/>
                                        </span>
                                        <span class="o_stat_text">
                                            <t t-esc="button.text"/>
                                        </span>
                                    </div>
                                </t>
                            </ViewButton>
                        </t>
                    </div>
                </div>
            </ProjectRightSidePanelSection>
            <ProjectRightSidePanelSection
                name="'profitability'"
                show="showProjectProfitability"
            >
                <t t-set-slot="title" owl="1">
                    Profitability
                </t>
                <ProjectProfitability
                    data="state.data.profitability_items"
                    labels="state.data.profitability_labels"
                    formatMonetary="formatMonetary.bind(this)"
                    onClick="(params) => this.onProjectActionClick(params)"
                />
            </ProjectRightSidePanelSection>
            <ProjectRightSidePanelSection
                name="'milestones'"
                show="!!state.data.milestones &amp;&amp; !!state.data.milestones.data"
            >
                <t t-set-slot="header" owl="1">
                    <span class="btn btn-secondary">
                        <div class="o_add_milestone">
                            <a t-on-click="addMilestone">Add Milestone</a>
                        </div>
                    </span>
                </t>
                <t t-set-slot="title" owl="1">
                    Milestones
                </t>
                <div t-foreach="state.data.milestones.data" t-as="milestone" t-key="milestone.id" class="o_rightpanel_data_row">
                    <ProjectMilestone context="context" milestone="milestone" open.bind="openFormViewDialog" load.bind="loadMilestones"/>
                </div>
                <span t-if="state.data.milestones.data.length === 0" class="text-muted fst-italic">
                    Track major progress points that must be reached to achieve success.
                </span>
            </ProjectRightSidePanelSection>
        </div>
        <!-- If this is called from notif, multiples updates but no specific project -->
        <div t-else=""/>
    </t>

</templates>

```

## File: static\src\components\project_right_side_panel\components\project_milestone.js

```javascript
/** @odoo-module  */

import { formatDate } from "@web/core/l10n/dates";
import { useService } from '@web/core/utils/hooks';

const { Component, useState, onWillUpdateProps, status } = owl;
const { DateTime } = luxon;

export class ProjectMilestone extends Component {
    setup() {
        this.orm = useService('orm');
        this.milestone = useState(this.props.milestone);
        this.state = useState({
            colorClass: this._getColorClass(),
            checkboxIcon: this._getCheckBoxIcon(),
        });
        onWillUpdateProps(this.onWillUpdateProps);
    }

    get resModel() {
        return 'project.milestone';
    }

    get deadline() {
        if (!this.milestone.deadline) return;
        return formatDate(DateTime.fromISO(this.milestone.deadline));
    }

    _getColorClass() {
        return this.milestone.is_deadline_exceeded && !this.milestone.can_be_marked_as_done ? "text-danger" : this.milestone.can_be_marked_as_done ? "text-success" : "";
    }

    _getCheckBoxIcon() {
        return this.milestone.is_reached ? "fa-check-square-o" : "fa-square-o";
    }

    onWillUpdateProps(nextProps) {
        if (nextProps.milestone) {
            this.milestone = nextProps.milestone;
            this.state.colorClass = this._getColorClass();
            this.state.checkboxIcon = this._getCheckBoxIcon();
        }
        if (nextProps.context) {
            this.contextValue = nextProps.context;
        }
    }

    async onDeleteMilestone() {
        await this.orm.call('project.milestone', 'unlink', [this.milestone.id]);
        await this.props.load();
    }

    async onOpenMilestone() {
        if (!this.write_mutex) {
            this.write_mutex = true;
            this.props.open({
                resModel: this.resModel,
                resId: this.milestone.id,
                title: this.env._t("Milestone"),
            }, {
                onClose: async () => {
                    if (status(this) === "mounted") {
                        await this.props.load();
                        this.write_mutex = false;
                    }
                },
            });
        }
    }

    async toggleIsReached() {
        if (!this.write_mutex) {
            this.write_mutex = true;
            this.milestone = await this.orm.call(
                this.resModel,
                'toggle_is_reached',
                [[this.milestone.id], !this.milestone.is_reached],
            );
            this.state.colorClass = this._getColorClass();
            this.state.checkboxIcon = this._getCheckBoxIcon();
            this.write_mutex = false;
        }
    }
}

ProjectMilestone.props = {
    context: Object,
    milestone: Object,
    open: Function,
    load: Function,
};
ProjectMilestone.template = 'project.ProjectMilestone';

```

## File: static\src\components\project_right_side_panel\components\project_milestone.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="project.ProjectMilestone" owl="1">
        <div class="list-group mb-2">
            <div class="o_rightpanel_milestone list-group-item list-group-item-action d-flex justify-content-evenly px-0 cursor-pointer" t-att-class="state.colorClass">
                <span t-on-click="toggleIsReached">
                    <i class="fa position-absolute pt-1" t-att-class="state.checkboxIcon"/>
                </span>
                <div class="o_milestone_detail d-flex justify-content-between ps-3 pe-2 col-11" t-on-click="onOpenMilestone">
                    <div class="text-truncate col-7" t-att-title="milestone.name">
                        <t t-esc="milestone.name"/>
                    </div>
                    <span class="d-flex justify-content-center align-items-center">
                        <t t-esc="deadline"/>
                    </span>
                </div>
                <span class="d-flex align-items-center">
                    <a t-on-click="onDeleteMilestone" title="Delete Milestone"><i class="fa fa-trash"/></a>
                </span>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\components\project_right_side_panel\components\project_profitability.js

```javascript
/** @odoo-module */

const { Component } = owl;

export class ProjectProfitability extends Component {
    get revenues() {
        return this.props.data.revenues;
    }

    get costs() {
        return this.props.data.costs;
    }

    get margin() {
        const invoiced_billed = this.revenues.total.invoiced + this.costs.total.billed;
        const to_invoice_to_bill = this.revenues.total.to_invoice + this.costs.total.to_bill;
        return {
            invoiced_billed,
            to_invoice_to_bill,
            total: invoiced_billed + to_invoice_to_bill,
        };
    }
}

ProjectProfitability.props = {
    data: Object,
    labels: Object,
    formatMonetary: Function,
    onClick: Function,
};
ProjectProfitability.template = 'project.ProjectProfitability';

```

## File: static\src\components\project_right_side_panel\components\project_profitability.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="project.ProjectProfitability" owl="1">
        <div class="o_rightpanel_subsection pb-3" t-if="revenues.data.length">
            <table class="table table-striped table-hover mb-0">
                <thead class="align-middle">
                    <tr>
                        <th>Revenues</th>
                        <th class="text-end">Invoiced</th>
                        <th class="text-end">To Invoice</th>
                        <th class="text-end">Expected</th>
                    </tr>
                </thead>
                <tbody>
                    <tr t-foreach="revenues.data" t-as="revenue" t-key="revenue.id" t-if="revenue.invoiced !== 0 || revenue.to_invoice !== 0">
                        <t t-set="revenue_label" t-value="props.labels[revenue.id] or revenue.id"/>
                        <td class="align-middle">
                            <a t-if="revenue.action" href="#"
                                t-on-click="() => this.props.onClick(revenue.action)"
                            >
                                <t t-esc="revenue_label"/>
                            </a>
                            <t t-esc="revenue_label" t-else=""/>
                        </td>
                        <td t-attf-class="text-end align-middle {{ revenue.invoiced === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.invoiced)"/></td>
                        <td t-attf-class="text-end align-middle {{ revenue.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.to_invoice)"/></td>
                        <td t-attf-class="text-end align-middle {{ revenue.invoiced + revenue.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.invoiced + revenue.to_invoice)"/></td>
                    </tr>
                </tbody>
                <tfoot>
                    <tr class="fw-bolder">
                        <td>Total</td>
                        <td t-attf-class="text-end {{ revenues.total.invoiced === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenues.total.invoiced)"/></td>
                        <td t-attf-class="text-end {{ revenues.total.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenues.total.to_invoice)"/></td>
                        <td t-attf-class="text-end {{ revenues.total.invoiced + revenues.total.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenues.total.invoiced + revenues.total.to_invoice)"/></td>
                    </tr>
                </tfoot>
            </table>
        </div>
        <div class="o_rightpanel_subsection pb-3" t-if="costs.data.length">
            <table class="table table-striped table-hover mb-0">
                <thead>
                    <tr>
                        <th>Costs</th>
                        <th class="text-end">Billed</th>
                        <th class="text-end">To Bill</th>
                        <th class="text-end">Expected</th>
                    </tr>
                </thead>
                <tbody>
                    <tr t-foreach="costs.data" t-as="cost" t-key="cost.id" t-if="cost.billed !== 0 || cost.to_bill !== 0">
                        <t t-set="cost_label" t-value="props.labels[cost.id] or cost.id"/>
                        <td class="align-middle">
                            <a t-if="cost.action" href="#"
                                t-on-click="() => this.props.onClick(cost.action)"
                            >
                                <t t-esc="cost_label"/>
                            </a>
                            <t t-esc="cost_label" t-else=""/>
                        </td>
                        <td t-attf-class="text-end align-middle {{ cost.billed === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(cost.billed)"/></td>
                        <td t-attf-class="text-end align-middle {{ cost.to_bill === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(cost.to_bill)"/></td>
                        <td t-attf-class="text-end align-middle {{ cost.billed + cost.to_bill === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(cost.billed + cost.to_bill)"/></td>
                    </tr>
                </tbody>
                <tfoot>
                    <tr class="fw-bolder">
                        <td>Total</td>
                        <td t-attf-class="text-end {{ costs.total.billed === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(costs.total.billed)"/></td>
                        <td t-attf-class="text-end {{ costs.total.to_bill === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(costs.total.to_bill)"/></td>
                        <td t-attf-class="text-end {{ costs.total.billed + costs.total.to_bill  === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(costs.total.billed + costs.total.to_bill)"/></td>
                    </tr>
                </tfoot>
            </table>
        </div>
        <div class="o_rightpanel_subsection">
            <table class="w-100 table table-borderless mb-4">
                <thead>
                    <tr>
                        <th>Margin</th>
                        <th class="text-end" t-att-class="margin.invoiced_billed &lt; 0 ? 'text-danger' : 'text-success'"><t t-esc="props.formatMonetary(margin.invoiced_billed)"/></th>
                        <th class="text-end" t-att-class="margin.to_invoice_to_bill &lt; 0 ? 'text-danger' : 'text-success'"><t t-esc="props.formatMonetary(margin.to_invoice_to_bill)"/></th>
                        <th class="text-end" t-att-class="margin.total &lt; 0 ? 'text-danger' : 'text-success'"><t t-esc="props.formatMonetary(margin.total)"/></th>
                    </tr>
                </thead>
            </table>
        </div>
    </t>
</templates>

```

## File: static\src\components\project_right_side_panel\components\project_right_side_panel_section.js

```javascript
/** @odoo-module */

const { Component } = owl;

export class ProjectRightSidePanelSection extends Component { }

ProjectRightSidePanelSection.props = {
    name: { type: String, optional: true },
    header: { type: Boolean, optional: true },
    show: Boolean,
    showData: { type: Boolean, optional: true },
    slots: {
        type: Object,
        shape: {
            default: Object, // Content is not optional
            header: { type: Object, optional: true },
            title: { type: Object, optional: true },
        },
    },
};
ProjectRightSidePanelSection.defaultProps = {
    header: true,
    showData: true,
};

ProjectRightSidePanelSection.template = 'project.ProjectRightSidePanelSection';

```

## File: static\src\components\project_right_side_panel\components\project_right_side_panel_section.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="project.ProjectRightSidePanelSection" owl="1">
        <div class="o_rightpanel_section py-0" t-att-name="props.name" t-if="props.show">
            <div class="o_rightpanel_header d-flex align-items-center justify-content-between py-4" t-if="props.header">
                <div class="o_rightpanel_title d-flex flex-row-reverse align-items-center" t-if="props.slots.title">
                    <h3 class="m-0 lh-lg"><t t-slot="title"/></h3>
                </div>
                <t t-slot="header"/>
            </div>
            <div class="o_rightpanel_data fs-6" t-if="props.showData">
                <t t-slot="default"/>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\components\project_state_selection\project_state_selection.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';
import { StateSelectionField } from '@web/views/fields/state_selection/state_selection_field';

import { STATUS_COLORS, STATUS_COLOR_PREFIX } from '../../utils/project_utils';

export class ProjectStateSelectionField extends StateSelectionField {
    setup() {
        super.setup();
        this.colorPrefix = STATUS_COLOR_PREFIX;
        this.colors = STATUS_COLORS;
    }

    /**
     * @override
     */
    get showLabel() {
        return !this.props.hideLabel;
    }

    /**
     * @override
     */
    get options() {
        return super.options.filter(o => o[0] !== 'to_define');
    }
}

registry.category('fields').add('kanban.project_state_selection', ProjectStateSelectionField);

```

## File: static\src\components\project_status_with_color_selection\project_status_with_color_selection_field.js

```javascript
/** @odoo-module */

import { SelectionField } from '@web/views/fields/selection/selection_field';
import { registry } from '@web/core/registry';

import { STATUS_COLORS, STATUS_COLOR_PREFIX } from '../../utils/project_utils';

export class ProjectStatusWithColorSelectionField extends SelectionField {
    setup() {
        super.setup();
        this.colorPrefix = STATUS_COLOR_PREFIX;
        this.colors = STATUS_COLORS;
    }

    get currentValue() {
        return this.props.value || this.options[0][0];
    }

    statusColor(value) {
        return this.colors[value] ? this.colorPrefix + this.colors[value] : "";
    }
}
ProjectStatusWithColorSelectionField.template = 'project.ProjectStatusWithColorSelectionField';

registry.category('fields').add('status_with_color', ProjectStatusWithColorSelectionField);

```

## File: static\src\components\project_status_with_color_selection\project_status_with_color_selection_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectStatusWithColorSelectionField" t-inherit="web.SelectionField" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-if='props.readonly']/span" position="replace">
            <div class="d-flex align-items-center">
                <span t-attf-class="o_status me-2 {{ statusColor(currentValue) }}"/>
                <span class="o_status_text text-wrap text-truncate" t-out="string" t-att-raw-value="value"/>
            </div>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\project_stop_recurrence_confirmation_dialog\project_stop_recurrence_confirmation_dialog.js

```javascript
/** @odoo-module */

import { ConfirmationDialog } from '@web/core/confirmation_dialog/confirmation_dialog';

export class ProjectStopRecurrenceConfirmationDialog extends ConfirmationDialog {
    _continueRecurrence() {
        if (this.props.continueRecurrence) {
            this.props.continueRecurrence();
        }
        this.props.close();
    }
}
ProjectStopRecurrenceConfirmationDialog.template = 'project.ProjectStopRecurrenceConfirmationDialog';
ProjectStopRecurrenceConfirmationDialog.props.continueRecurrence = { type: Function, optional: true };

```

## File: static\src\components\project_stop_recurrence_confirmation_dialog\project_stop_recurrence_confirmation_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="project.ProjectStopRecurrenceConfirmationDialog" t-inherit="web.ConfirmationDialog" t-inherit-mode="primary" owl="1">
        <xpath expr="//button[@t-on-click='_confirm']" position="replace">
            <button class="btn btn-primary" t-on-click="_confirm">
                Stop Recurrence
            </button>
        </xpath>
        <xpath expr="//button[@t-on-click='_confirm']" position="after">
            <button t-if="props.continueRecurrence" class="btn btn-secondary" t-on-click="_continueRecurrence">
                Continue Recurrence
            </button>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\project_task_name_with_subtask_count_char_field\project_task_name_with_subtask_count_char_field.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';
import { CharField } from '@web/views/fields/char/char_field';
import { formatChar } from '@web/views/fields/formatters';

class ProjectTaskNameWithSubtaskCountCharField extends CharField {
    get formattedSubtaskCount() {
        return formatChar(this.props.record.data.allow_subtasks && this.props.record.data.child_text || '');
    }
}

ProjectTaskNameWithSubtaskCountCharField.template = 'project.ProjectTaskNameWithSubtaskCountCharField';

registry.category('fields').add('name_with_subtask_count', ProjectTaskNameWithSubtaskCountCharField);

```

## File: static\src\components\project_task_name_with_subtask_count_char_field\project_task_name_with_subtask_count_char_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectTaskNameWithSubtaskCountCharField" t-inherit="web.CharField" t-inherit-mode="primary" owl="1">
        <xpath expr="//span[@t-esc='formattedValue']" position="after">
            <span
                class="text-muted ms-2"
                t-out="formattedSubtaskCount"
                style="font-weight: normal;"
            />
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

## File: static\src\js\project_control_panel.js

```javascript
/** @odoo-module **/

import ControlPanel from 'web.ControlPanel';
import session from 'web.session';

const { onWillStart, onWillUpdateProps } = owl;

export class ProjectControlPanel extends ControlPanel {

    setup() {
        super.setup();
        this.show_project_update = this.props.view.type === "form" || this.props.action.context.show_project_update;
        this.project_id = this.show_project_update ? this.props.action.context.active_id : false;

        onWillStart(() => this._loadWidgetData());
        onWillUpdateProps(() => this._loadWidgetData());
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

## File: static\src\js\project_graph_view.js

```javascript
/** @odoo-module **/

import { ProjectControlPanel } from "@project/components/project_control_panel/project_control_panel";
import { registry } from "@web/core/registry";
import { graphView } from "@web/views/graph/graph_view";

const viewRegistry = registry.category("views");

export const projectGraphView = {...graphView, ControlPanel: ProjectControlPanel};

viewRegistry.add("project_graph", projectGraphView);

```

## File: static\src\js\project_pivot_view.js

```javascript
/** @odoo-module **/

import { ProjectControlPanel } from "@project/components/project_control_panel/project_control_panel";
import { registry } from "@web/core/registry";
import { pivotView } from "@web/views/pivot/pivot_view";

const projectPivotView = {...pivotView, ControlPanel: ProjectControlPanel};

registry.category("views").add("project_pivot", projectPivotView);

```

## File: static\src\js\project_rating_graph_view.js

```javascript
/** @odoo-module **/

import { _lt } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { GraphArchParser } from "@web/views/graph/graph_arch_parser";
import { graphView } from "@web/views/graph/graph_view";

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
const projectRatingGraphView = {
    ...graphView,
    ArchParser: ProjectRatingArchParser,
};

viewRegistry.add("project_rating_graph", projectRatingGraphView);

```

## File: static\src\js\project_rating_pivot_view.js

```javascript
/** @odoo-module **/

import { _lt } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { PivotArchParser } from "@web/views/pivot/pivot_arch_parser";
import { pivotView } from "@web/views/pivot/pivot_view";

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

const projectRatingPivotView = {
    ...pivotView,
    ArchParser: ProjectRatingArchParser,
};

viewRegistry.add("project_rating_pivot", projectRatingPivotView);

```

## File: static\src\js\project_task_kanban_examples.js

```javascript
/** @odoo-module **/

import { _lt } from 'web.core';
import kanbanExamplesRegistry from 'web.kanban_examples_registry';
import { registry } from "@web/core/registry";
import { renderToMarkup } from '@web/core/utils/render';

const { markup } = owl;
const greenBullet = markup(`<span class="o_status d-inline-block o_status_green"></span>`);
const redBullet = markup(`<span class="o_status d-inline-block o_status_red"></span>`);
const star = markup(`<a style="color: gold;" class="fa fa-star"></a>`);
const clock = markup(`<a class="fa fa-clock-o"></a>`);

const exampleData = {
    ghostColumns: [_lt('New'), _lt('Assigned'), _lt('In Progress'), _lt('Done')],
    applyExamplesText: _lt("Use This For My Project"),
    allowedGroupBys: ['stage_id'],
    examples:[{
        name: _lt('Software Development'),
        columns: [_lt('Backlog'), _lt('Specifications'), _lt('Development'), _lt('Tests'), _lt('Delivered')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, redBullet, star],
    }, {
        name: _lt('Agile Scrum'),
        columns: [_lt('Backlog'), _lt('Sprint Backlog'), _lt('Sprint in Progress'), _lt('Sprint Complete'), _lt('Old Completed Sprint')],
        get description() {
            return renderToMarkup("project.example.agilescrum");
        },
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Digital Marketing'),
        columns: [_lt('Ideas'), _lt('Researching'), _lt('Writing'), _lt('Editing'), _lt('Done')],
        get description() {
            return renderToMarkup("project.example.digitalmarketing");
        },
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Customer Feedback'),
        columns: [_lt('New'), _lt('In development'), _lt('Done'), _lt('Refused')],
        get description() {
            return renderToMarkup("project.example.customerfeedback");
        },
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Consulting'),
        columns: [_lt('New Projects'), _lt('Resources Allocation'), _lt('In Progress'), _lt('Done')],
        get description() {
            return renderToMarkup("project.example.consulting");
        },
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Research Project'),
        columns: [_lt('Brainstorm'), _lt('Research'), _lt('Draft'), _lt('Final Document')],
        get description() {
            return renderToMarkup("project.example.researchproject");
        },
        bullets: [greenBullet, redBullet],
    }, {
        name: _lt('Website Redesign'),
        columns: [_lt('Page Ideas'), _lt('Copywriting'), _lt('Design'), _lt('Live')],
        get description() {
            return renderToMarkup("project.example.researchproject");
        },
    }, {
        name: _lt('T-shirt Printing'),
        columns: [_lt('New Orders'), _lt('Logo Design'), _lt('To Print'), _lt('Done')],
        get description() {
            return renderToMarkup("project.example.tshirtprinting");
        },
        bullets: [star],
    }, {
        name: _lt('Design'),
        columns: [_lt('New Request'), _lt('Design'), _lt('Client Review'), _lt('Handoff')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, redBullet, star, clock],
    }, {
        name: _lt('Publishing'),
        columns: [_lt('Ideas'), _lt('Writing'), _lt('Editing'), _lt('Published')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, redBullet, star, clock],
    }, {
        name: _lt('Manufacturing'),
        columns: [_lt('New Orders'), _lt('Material Sourcing'), _lt('Manufacturing'), _lt('Assembling'), _lt('Delivered')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, redBullet, star, clock],
    }, {
        name: _lt('Podcast and Video Production'),
        columns: [_lt('Research'), _lt('Script'), _lt('Recording'), _lt('Mixing'), _lt('Published')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, redBullet, star, clock],
    }],
};

kanbanExamplesRegistry.add('project', exampleData);
registry.category("kanban_examples").add('project', exampleData);

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
    rainbowManMessage: _t("Congratulations, you are now a master of project management."),
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
    trigger: '.o_project_name input',
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
    trigger: ".o_kanban_project_tasks .o_column_quick_create .input-group .o_input",
    content: Markup(_t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>.")),
    position: 'bottom',
    run: "text Test",
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    content: Markup(_t('Let\'s create your first <b>stage</b>.')),
    position: 'right',
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .input-group .o_input",
    extra_trigger: '.o_kanban_group',
    content: Markup(_t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>.")),
    position: 'bottom',
    run: "text Test",
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    content: Markup(_t('Let\'s create your second <b>stage</b>.')),
    position: 'right',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_kanban_group:eq(1)',
    content: Markup(_t("Let's create your first <b>task</b>.")),
    position: 'bottom',
    width: 200,
}, {
    trigger: '.o_kanban_quick_create div.o_field_char[name=name] input',
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
    run: "drag_and_drop_native .o_kanban_group:eq(1) ",
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
    content: Markup(_t("Create <b>activities</b> to set yourself to-dos or to schedule meetings.")),
}, {
    trigger: ".modal-dialog .btn-primary",
    extra_trigger: '.o_form_project_tasks',
    content: _t("Schedule your activity once it is ready."),
    position: "bottom",
    run: "click",
}, {
    trigger: ".o_field_widget[name='user_ids']",
    extra_trigger: '.o_form_project_tasks',
    content: _t("Assign a responsible to your task"),
    position: "right",
    run() {
        document.querySelector('.o_field_widget[name="user_ids"] input').click();
    }
}, {
    trigger: ".ui-autocomplete > li > a:not(:has(i.fa))",
    auto: true,
    mobile: false,
}, {
    trigger: "div[role='article']",
    mobile: true,
    run: "click",
}, {
    trigger: ".o_form_button_save",
    extra_trigger: '.o_form_project_tasks.o_form_dirty',
    content: Markup(_t("You have unsaved changes - no worries! Odoo will automatically save it as you navigate.<br/> You can discard these changes from here or manually save your task.<br/>Let's save it manually.")),
    position: "bottom",
}, {
    trigger: ".breadcrumb .o_back_button",
    extra_trigger: '.o_form_project_tasks',
    content: Markup(_t("Let's go back to the <b>kanban view</b> to have an overview of your next tasks.")),
    position: "right",
    run: 'click',
}, {
    trigger: '.o_kanban_renderer',
    // last step to confirm we've come back before considering the tour successful
    auto: true
}]);

});

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
            this.events = { ...this.events };
            delete this.events.click;
        }
    },

    _render: function () {
        let result = this._super.apply(this, arguments);
        if (this.recordData.allow_subtasks && this.recordData.child_text) {
            this.$el.append($('<span>')
                    .addClass("text-muted ms-2")
                    .text(this.recordData.child_text)
                    .css('font-weight', 'normal'));
        }
        return result;
    }
});

fieldRegistry.add('name_with_subtask_count', FieldNameWithSubTaskCount);

```

## File: static\src\js\widgets\project_private_task.js

```javascript
/** @odoo-module alias=project.project_private_task **/
"use strict";

import field_registry from 'web.field_registry';
import { FieldMany2One } from 'web.relational_fields';
import core from 'web.core';

const QWeb = core.qweb;

const ProjectPrivateTask = FieldMany2One.extend({
    /**
     * @override
     * @private
     */
    _renderReadonly: function() {
        this._super.apply(this, arguments);
        if (!this.m2o_value) {
            this.$el.empty();
            this.$el.append(QWeb.render('project.task.PrivateProjectName'));
            this.$el.addClass('pe-none');
        }
    },
});

field_registry.add('project_private_task', ProjectPrivateTask);

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
        this.hideLabel = this.nodeOptions.hide_label;
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
            if (this.hideLabel) {
                this.$el.attr('title', this.$el.text());
                this.$el.empty();
            }
            this.$el.prepend(qweb.render(this._template, {
                color: this.color,
            }));
        }
    },
});

fieldRegistry.add('status_with_color', StatusWithColor);

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

import { useBus, useService } from '@web/core/utils/hooks';
import { ActionContainer } from '@web/webclient/actions/action_container';
import { MainComponentsContainer } from "@web/core/main_components_container";
import { useOwnDebugContext } from "@web/core/debug/debug_context";
import { session } from '@web/session';

const { Component, useEffect, useExternalListener, useState } = owl;

export class ProjectSharingWebClient extends Component {
    setup() {
        window.parent.document.body.style.margin = "0"; // remove the margin in the parent body
        this.actionService = useService('action');
        this.user = useService("user");
        useService("legacy_service_provider");
        useOwnDebugContext({ categories: ["default"] });
        this.state = useState({
            fullscreen: false,
        });
        useBus(this.env.bus, "ACTION_MANAGER:UI-UPDATED", (mode) => {
            if (mode !== "new") {
                this.state.fullscreen = mode === "fullscreen";
            }
        });
        useEffect(
            () => {
                this._showView();
            },
            () => []
        );
        useExternalListener(window, "click", this.onGlobalClick, { capture: true });
    }

    async _showView() {
        const { action_name, project_id, open_task_action } = session;
        await this.actionService.doAction(
            action_name,
            {
                clearBreadcrumbs: true,
                additionalContext: {
                    active_id: project_id,
                }
            }
        );
        if (open_task_action) {
            await this.actionService.doAction(open_task_action);
        }
    }

    /**
     * @param {MouseEvent} ev
     */
    onGlobalClick(ev) {
        // When a ctrl-click occurs inside an <a href/> element
        // we let the browser do the default behavior and
        // we do not want any other listener to execute.
        if (
            ev.ctrlKey &&
            ((ev.target instanceof HTMLAnchorElement && ev.target.href) ||
                (ev.target instanceof HTMLElement && ev.target.closest("a[href]:not([href=''])")))
        ) {
            ev.stopImmediatePropagation();
            return;
        }
    }
}

ProjectSharingWebClient.components = { ActionContainer, MainComponentsContainer };
ProjectSharingWebClient.template = 'project.ProjectSharingWebClient';

```

## File: static\src\project_sharing\project_sharing.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectSharingWebClient" owl="1">
        <ActionContainer />
        <MainComponentsContainer/>
    </t>

</templates>

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
    for (const [key] of favoriteMenuRegistry.getEntries()) {
        if (key !== customFavoriteItemKey) {
            favoriteMenuRegistry.remove(key);
        }
    }
}

```

## File: static\src\project_sharing\components\chatter\chatter_attachments_viewer.js

```javascript
/** @odoo-module */

const { Component } = owl;

export class ChatterAttachmentsViewer extends Component {}

ChatterAttachmentsViewer.template = 'project.ChatterAttachmentsViewer';
ChatterAttachmentsViewer.props = {
    attachments: Array,
    canDelete: { type: Boolean, optional: true },
    delete: { type: Function, optional: true },
};
ChatterAttachmentsViewer.defaultProps = {
    delete: async () => {},
};

```

## File: static\src\project_sharing\components\chatter\chatter_attachments_viewer.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.ChatterAttachmentsViewer" owl="1">
        <div class="o_portal_chatter_attachments mt-3">
            <div t-if="props.attachments.length" class="row">
                <div t-foreach="props.attachments" t-as="attachment" t-key="attachment.id" class="col-lg-3 col-md-4 col-sm-6">
                    <div class="o_portal_chatter_attachment mb-2 position-relative text-center">
                        <button
                            t-if="props.canDelete and attachment.state == 'pending'"
                            class="btn btn-sm btn-outline-danger"
                            title="Delete"
                            t-on-click="() => props.delete(attachment)"
                        >
                            <i class="fa fa-times"/>
                        </button>
                        <a t-attf-href="/web/content/#{attachment.id}?download=true&amp;access_token=#{attachment.access_token}" target="_blank">
                            <div class='oe_attachment_embedded o_image' t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype"/>
                            <div class='o_portal_chatter_attachment_name'>
                                <t t-out='attachment.filename'/>
                            </div>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\project_sharing\components\chatter\chatter_composer.js

```javascript
/** @odoo-module */

import { useService } from '@web/core/utils/hooks';
import { TextField } from '@web/views/fields/text/text_field';
import { PortalAttachDocument } from '../portal_attach_document/portal_attach_document';
import { ChatterAttachmentsViewer } from './chatter_attachments_viewer';

const { Component, useState, onWillUpdateProps } = owl;

export class ChatterComposer extends Component {
    setup() {
        this.rpc = useService('rpc');
        this.state = useState({
            displayError: false,
            attachments: this.props.attachments.map(file => file.state === 'done'),
            message: '',
            loading: false,
        });

        onWillUpdateProps(this.onWillUpdateProps);
    }

    onWillUpdateProps(nextProps) {
        this.clearErrors();
        this.state.message = '';
        this.state.attachments = nextProps.attachments.map(file => file.state === 'done');
    }

    get discussionUrl() {
        return `${window.location.href.split('#')[0]}#discussion`;
    }

    update(change) {
        this.clearErrors();
        this.state.message = change;
    }

    prepareMessageData() {
        const attachment_ids = [];
        const attachment_tokens = [];
        for (const attachment of this.state.attachments) {
            attachment_ids.push(attachment.id);
            attachment_tokens.push(attachment.access_token);
        }
        return {
            message: this.state.message,
            attachment_ids,
            attachment_tokens,
            res_model: this.props.resModel,
            res_id: this.props.resId,
            project_sharing_id: this.props.projectSharingId,
        };
    }

    async sendMessage() {
        this.clearErrors();
        if (!this.state.message && !this.state.attachments.length) {
            this.state.displayError = true;
            return;
        }

        await this.rpc(
            "/mail/chatter_post",
            this.prepareMessageData(),
        );
        this.props.postProcessMessageSent();
        this.state.message = "";
        this.state.attachments = [];
    }

    clearErrors() {
        this.state.displayError = false;
    }

    async beforeUploadFile() {
        this.state.loading = true;
        return true;
    }

    onFileUpload(files) {
        this.state.loading = false;
        this.clearErrors();
        for (const file of files) {
            file.state = 'pending';
            this.state.attachments.push(file);
        }
    }

    async deleteAttachment(attachment) {
        this.clearErrors();
        try {
            await this.rpc(
                '/portal/attachment/remove',
                {
                    attachment_id: attachment.id,
                    access_token: attachment.access_token,
                },
            );
        } catch (err) {
            console.error(err);
            this.state.displayError = true;
        }
        this.state.attachments = this.state.attachments.filter(a => a.id !== attachment.id);
    }
}

ChatterComposer.components = {
    ChatterAttachmentsViewer,
    PortalAttachDocument,
    TextField,
};

ChatterComposer.props = {
    resModel: String,
    projectSharingId: Number,
    resId: { type: Number, optional: true },
    allowComposer: { type: Boolean, optional: true },
    displayComposer: { type: Boolean, optional: true },
    token: { type: String, optional: true },
    messageCount: { type: Number, optional: true },
    isUserPublic: { type: Boolean, optional: true },
    partnerId: { type: Number, optional: true },
    postProcessMessageSent: { type: Function, optional: true },
    attachments: { type: Array, optional: true },
};
ChatterComposer.defaultProps = {
    allowComposer: true,
    displayComposer: false,
    isUserPublic: true,
    token: '',
    attachments: [],
};

ChatterComposer.template = 'project.ChatterComposer';

```

## File: static\src\project_sharing\components\chatter\chatter_composer.xml

```xml
<templates id="template" xml:space="preserve">

    <!-- Widget PortalComposer (standalone)

        required many options: token, res_model, res_id, ...
    -->
    <t t-name="project.ChatterComposer" owl="1">
        <div t-if="props.allowComposer" class="o_portal_chatter_composer">
            <t t-if="props.displayComposer">
                <div t-if="state.displayError" class="alert alert-danger mb8 o_portal_chatter_composer_error" role="alert">
                    Oops! Something went wrong. Try to reload the page and log in.
                </div>

                <div class="d-flex">
                    <img t-if="!props.isUserPublic or props.token"
                         alt="Avatar"
                         class="o_portal_chatter_avatar o_object_fit_cover align-self-start"
                         t-attf-src="/web/image/res.partner/{{ props.partnerId }}/avatar_128"
                    />
                    <div class="flex-grow-1">
                        <div class="o_portal_chatter_composer_input">
                            <div class="o_portal_chatter_composer_body mb32">
                                <TextField
                                    rowCount="4"
                                    placeholder="'Write a message...'"
                                    value="state.message"
                                    update.bind="update"
                                />
                                <ChatterAttachmentsViewer
                                    attachments="state.attachments"
                                    canDelete="true"
                                    delete.bind="deleteAttachment"
                                />
                                <div class="mt8">
                                    <button name="send_message" t-on-click="sendMessage" class="btn btn-primary me-1" type="submit" t-att-disabled="state.loading">
                                        Send
                                    </button>
                                    <PortalAttachDocument
                                        resModel="props.resModel"
                                        resId="props.resId"
                                        token="props.token"
                                        multiUpload="true"
                                        onUpload.bind="onFileUpload"
                                        beforeOpen.bind="beforeUploadFile"
                                    >
                                        <i class="fa fa-paperclip"/>
                                    </PortalAttachDocument>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
            <t t-else="">
                <h4>Leave a comment</h4>
                <p>You must be <a t-attf-href="/web/login?redirect={{ discussionUrl }}">logged in</a> to post a comment.</p>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\project_sharing\components\chatter\chatter_container.js

```javascript
/** @odoo-module */

import { formatDateTime, parseDateTime } from "@web/core/l10n/dates";
import { useService } from "@web/core/utils/hooks";
import { sprintf } from '@web/core/utils/strings';
import { ChatterComposer } from "./chatter_composer";
import { ChatterMessageCounter } from "./chatter_message_counter";
import { ChatterMessages } from "./chatter_messages";
import { ChatterPager } from "./chatter_pager";

const { Component, markup, onWillStart, useState, onWillUpdateProps } = owl;

export class ChatterContainer extends Component {
    setup() {
        this.rpc = useService('rpc');
        this.state = useState({
            currentPage: this.props.pagerStart,
            messages: [],
            options: this.defaultOptions,
        });

        onWillStart(this.onWillStart);
        onWillUpdateProps(this.onWillUpdateProps);
    }

    get defaultOptions() {
        return {
            message_count: 0,
            is_user_public: true,
            is_user_employee: false,
            is_user_published: false,
            display_composer: Boolean(this.props.resId),
            partner_id: null,
            pager_scope: 4,
            pager_step: 10,
        };
    }

    get options() {
        return this.state.options;
    }

    set options(options) {
        this.state.options = {
            ...this.defaultOptions,
            ...options,
            display_composer: !!options.display_composer,
            access_token: typeof options.display_composer === 'string' ? options.display_composer : '',
        };
    }

    get composerProps() {
        return {
            allowComposer: Boolean(this.props.resId),
            displayComposer: this.state.options.display_composer,
            partnerId: this.state.options.partner_id || undefined,
            token: this.state.options.access_token,
            resModel: this.props.resModel,
            resId: this.props.resId,
            projectSharingId: this.props.projectSharingId,
            postProcessMessageSent: async () => {
                this.state.currentPage = 1;
                await this.fetchMessages();
            },
            attachments: this.state.options.default_attachment_ids,
        };
    }

    onWillStart() {
        this.initChatter(this.messagesParams(this.props));
    }

    onWillUpdateProps(nextProps) {
        this.initChatter(this.messagesParams(nextProps));
    }

    async onChangePage(page) {
        this.state.currentPage = page;
        await this.fetchMessages();
    }

    async initChatter(params) {
        if (params.res_id && params.res_model) {
            const chatterData = await this.rpc(
                '/mail/chatter_init',
                params,
            );
            this.state.messages = this.preprocessMessages(chatterData.messages);
            this.options = chatterData.options;
        } else {
            this.state.messages = [];
            this.options = {};
        }
    }

    async fetchMessages() {
        const result = await this.rpc(
            '/mail/chatter_fetch',
            this.messagesParams(this.props),
        );
        this.state.messages = this.preprocessMessages(result.messages);
        this.state.options.message_count = result.message_count;
        return result;
    }

    messagesParams(props) {
        const params = {
            res_model: props.resModel,
            res_id: props.resId,
            limit: this.state.options.pager_step,
            offset: (this.state.currentPage - 1) * this.state.options.pager_step,
            allow_composer: Boolean(props.resId),
            project_sharing_id: props.projectSharingId,
        };
        if (props.token) {
            params.token = props.token;
        }
        if (props.domain) {
            params.domain = props.domain;
        }
        return params;
    }

    preprocessMessages(messages) {
        return messages.map(m => ({
            ...m,
            author_avatar_url: sprintf('/web/image/mail.message/%s/author_avatar/50x50', m.id),
            published_date_str: sprintf(
                this.env._t('Published on %s'),
                formatDateTime(
                    parseDateTime(
                        m.date,
                        { format: 'MM-dd-yyy HH:mm:ss' },
                    ),
                )
            ),
            body: markup(m.body),
        }));
    }

    updateMessage(message_id, changes) {
        Object.assign(
            this.state.messages.find(m => m.id === message_id),
            changes,
        );
    }
}

ChatterContainer.components = {
    ChatterComposer,
    ChatterMessageCounter,
    ChatterMessages,
    ChatterPager,
};

ChatterContainer.props = {
    token: { type: String, optional: true },
    resModel: String,
    resId: { type: Number, optional: true },
    pid: { type: String, optional: true },
    hash: { type: String, optional: true },
    pagerStart: { type: Number, optional: true },
    twoColumns: { type: Boolean, optional: true },
    projectSharingId: Number,
};
ChatterContainer.defaultProps = {
    token: '',
    pid: '',
    hash: '',
    pagerStart: 1,
};
ChatterContainer.template = 'project.ChatterContainer';

```

## File: static\src\project_sharing\components\chatter\chatter_container.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.ChatterContainer" owl="1">
        <div t-attf-class="o_portal_chatter p-0 container {{props.twoColumns ? 'row' : ''}}">
            <div t-attf-class="{{props.twoColumns ? 'col-lg-5' : props.resId ? 'border-bottom' : ''}}">
                <div class="o_portal_chatter_header">
                    <ChatterMessageCounter count="state.options.message_count"/>
                </div>
                <hr/>
                <ChatterComposer t-props="composerProps"/>
            </div>
            <div t-attf-class="{{props.twoColumns ? 'offset-lg-1 col-lg-6' : 'pt-4'}}">
                <ChatterMessages messages="props.resId ? state.messages : []" isUserEmployee="state.options.is_user_employee" update.bind="updateMessage" />
                <div class="o_portal_chatter_footer">
                    <ChatterPager
                        page="this.state.currentPage || 1"
                        messageCount="this.state.options.message_count"
                        pagerScope="this.state.options.pager_scope"
                        pagerStep="this.state.options.pager_step"
                        changePage.bind="onChangePage"
                    />
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\project_sharing\components\chatter\chatter_messages.js

```javascript
/** @odoo-module */

import { useService } from "@web/core/utils/hooks";

import { ChatterAttachmentsViewer } from "./chatter_attachments_viewer";

const { Component } = owl;

export class ChatterMessages extends Component {
    setup() {
        this.rpc = useService('rpc');
    }

    /**
     * Toggle the visibility of the message.
     *
     * @param {Object} message message to change the visibility
     */
    async toggleMessageVisibility(message) {
        const result = await this.rpc(
            '/mail/update_is_internal',
            { message_id: message.id, is_internal: !message.is_internal },
        );
        this.props.update(message.id, { is_internal: result });
    }
}

ChatterMessages.template = 'project.ChatterMessages';
ChatterMessages.props = {
    messages: Array,
    isUserEmployee: { type: Boolean, optional: true },
    update: { type: Function, optional: true },
};
ChatterMessages.defaultProps = {
    update: (message_id, changes) => {},
};
ChatterMessages.components = { ChatterAttachmentsViewer };

```

## File: static\src\project_sharing\components\chatter\chatter_messages.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.ChatterMessages" owl="1">
        <div class="o_portal_chatter_messages">
            <t t-foreach="props.messages" t-as="message" t-key="message.id">
                <div class="d-flex o_portal_chatter_message">
                    <img class="o_portal_chatter_avatar" t-att-src="message.author_avatar_url" alt="avatar"/>
                    <div class="flex-grow-1">
                        <t t-if="props.isUserEmployee">
                            <div t-if="message.is_message_subtype_note" class="float-end">
                                <button class="btn btn-secondary" title="Internal notes are only displayed to internal users." disabled="true">Internal Note</button>
                            </div>
                            <div t-else=""
                                 t-attf-class="float-end {{message.is_internal ? 'o_portal_message_internal_on' : 'o_portal_message_internal_off'}}"
                                 t-on-click="() => this.toggleMessageVisibility(message)"
                            >
                                <button class="btn btn-danger"
                                       title="Currently restricted to internal employees, click to make it available to everyone viewing this document."
                                >
                                    Employees Only
                                </button>
                                <button class="btn btn-success"
                                       title="Currently available to everyone viewing this document, click to restrict to internal employees."
                                >
                                    Visible
                                </button>
                            </div>
                        </t>
                        <div class="o_portal_chatter_message_title">
                            <h5 class='mb-1'><t t-out="message.author_id[1]"/></h5>
                            <p class="o_portal_chatter_puslished_date"><t t-out="message.published_date_str"/></p>
                        </div>
                        <t t-out="message.body"/>
                        <div class="o_portal_chatter_attachments">
                            <ChatterAttachmentsViewer attachments="message.attachment_ids"/>
                        </div>
                    </div>
                </div>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\project_sharing\components\chatter\chatter_message_counter.js

```javascript
/** @odoo-module */

const { Component } = owl;

export class ChatterMessageCounter extends Component { }

ChatterMessageCounter.props = {
    count: Number,
};
ChatterMessageCounter.template = 'project.ChatterMessageCounter';

```

## File: static\src\project_sharing\components\chatter\chatter_message_counter.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.ChatterMessageCounter" owl="1">
        <div class="o_message_counter">
            <t t-if="props.count">
                <span class="fa fa-comments" />
                <span class="o_message_count"> <t t-esc="props.count"/> </span>
                comments
            </t>
            <t t-else="">
                There are no comments for now.
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\project_sharing\components\chatter\chatter_pager.js

```javascript
/** @odoo-module */

const { Component, useState, onWillUpdateProps } = owl;

export class ChatterPager extends Component {
    setup() {
        this.state = useState({
            disabledButtons: false,
            pageCount: 1,
            pageStart: 1,
            pageEnd: 1,
            pagePrevious: 1,
            pageNext: 1,
            pages: [1],
            offset: 0,
        });
        this.computePagerState(this.props);

        onWillUpdateProps(this.onWillUpdateProps);
    }

    computePagerState(props) {
        let page = props.page || 1;
        let scope = props.pagerScope;

        const step = props.pagerStep;

        // Compute Pager
        this.state.messageCount = Math.ceil(parseFloat(props.messageCount) / step);

        page = Math.max(1, Math.min(page, this.state.messageCount));

        const pageStart = Math.max(page - parseInt(Math.floor(scope / 2)), 1);
        this.state.pageEnd = Math.min(pageStart + scope, this.state.messageCount);
        this.state.pageStart = Math.max(this.state.pageEnd - scope, 1);

        this.state.pages = Array.from(
            {length: this.state.pageEnd - this.state.pageStart + 1},
            (_, i) => i + this.state.pageStart,
        );
        this.state.pagePrevious = Math.max(this.state.pageStart, page - 1);
        this.state.pageNext = Math.min(this.state.pageEnd, page + 1);
    }

    onWillUpdateProps(nextProps) {
        this.computePagerState(nextProps);
    }

    async onPageChanged(page) {
        this.state.disabledButtons = true;
        await this.props.changePage(page);
        this.state.disabledButtons = false;
    }
}

ChatterPager.props = {
    pagerScope: Number,
    pagerStep: Number,
    page: Number,
    messageCount: Number,
    changePage: Function,
};

ChatterPager.template = 'project.ChatterPager';

```

## File: static\src\project_sharing\components\chatter\chatter_pager.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.ChatterPager" owl="1">
        <div class="d-flex justify-content-center">
            <ul class="pagination mb-0 pb-4" t-if="state.pages.length &gt; 1">
                <li t-if="props.page != props.page_previous" t-att-data-page="state.pagePrevious" class="page-item o_portal_chatter_pager_btn">
                    <a t-on-click="() => this.onPageChanged(state.pagePrevious)" class="page-link"><i class="fa fa-chevron-left" role="img" aria-label="Previous" title="Previous"/></a>
                </li>
                <t t-foreach="state.pages" t-as="page" t-key="page_index">
                    <li t-att-data-page="page" t-attf-class="page-item #{page == props.page ? 'o_portal_chatter_pager_btn active' : 'o_portal_chatter_pager_btn'}">
                        <a t-on-click="() => this.onPageChanged(page)" t-att-disabled="page == props.page" class="page-link"><t t-esc="page"/></a>
                    </li>
                </t>
                <li t-if="props.page != state.pageNext" t-att-data-page="state.pageNext" class="page-item o_portal_chatter_pager_btn">
                    <a t-on-click="() => this.onPageChanged(state.pageNext)" class="page-link"><i class="fa fa-chevron-right" role="img" aria-label="Next" title="Next"/></a>
                </li>
            </ul>
        </div>
    </t>

</templates>

```

## File: static\src\project_sharing\components\portal_attach_document\portal_attach_document.js

```javascript
/** @odoo-module */

import { PortalFileInput } from '../portal_file_input/portal_file_input';

const { Component } = owl;

export class PortalAttachDocument extends Component {}

PortalAttachDocument.template = 'project.PortalAttachDocument';
PortalAttachDocument.components = { PortalFileInput };
PortalAttachDocument.props = {
    highlight: { type: Boolean, optional: true },
    onUpload: { type: Function, optional: true },
    beforeOpen: { type: Function, optional: true },
    slots: {
        type: Object,
        shape: {
            default: Object,
        },
    },
    resId: { type: Number, optional: true },
    resModel: { type: String, optional: true },
    multiUpload: { type: Boolean, optional: true },
    hidden: { type: Boolean, optional: true },
    acceptedFileExtensions: { type: String, optional: true },
    token: { type: String, optional: true },
};
PortalAttachDocument.defaultProps = {
    acceptedFileExtensions: "*",
    onUpload: () => {},
    route: "/portal/attachment/add",
    beforeOpen: async () => true,
};

```

## File: static\src\project_sharing\components\portal_attach_document\portal_attach_document.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.PortalAttachDocument" owl="1">
        <button t-attf-class="btn o_attachment_button #{props.highlight ? 'btn-primary' : 'btn-secondary'}">
            <PortalFileInput
                onUpload="props.onUpload"
                beforeOpen="props.beforeOpen"
                multiUpload="props.multiUpload"
                resModel="props.resModel"
                resId="props.resId"
                route="props.route"
                accessToken="props.token"
            >
                <t t-set-slot="default">
                    <i class="fa fa-paperclip"/>
                </t>
            </PortalFileInput>
        </button>
    </t>

</templates>

```

## File: static\src\project_sharing\components\portal_file_input\portal_file_input.js

```javascript
/** @odoo-module */

import { FileInput } from '@web/core/file_input/file_input';

export class PortalFileInput extends FileInput {
    /**
     * @override
     */
    get httpParams() {
        const {
            model: res_model,
            id: res_id,
            ...otherParams
        } = super.httpParams;
        return {
            res_model,
            res_id,
            access_token: this.props.accessToken,
            ...otherParams,
        }
    }

    async uploadFiles(params) {
        const { ufile: files, ...otherParams } = params;
        const filesData = await Promise.all(
            files.map(
                (file) =>
                    super.uploadFiles({
                        file,
                        name: file.name,
                        ...otherParams,
                    })
            )
        );
        return filesData;
    }
}

PortalFileInput.props = {
    ...FileInput.props,
    accessToken: { type: String, optional: true },
};
PortalFileInput.defaultProps = {
    ...FileInput.defaultProps,
    accessToken: '',
};

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

## File: static\src\project_sharing\views\form\project_sharing_form_compiler.js

```javascript
/** @odoo-module */

import { append, createElement, setAttributes } from "@web/core/utils/xml";
import { registry } from "@web/core/registry";
import { SIZES } from "@web/core/ui/ui_service";
import { getModifier, ViewCompiler } from "@web/views/view_compiler";
import { patch } from "@web/core/utils/patch";
import { FormCompiler } from "@web/views/form/form_compiler";

/**
 * Compiler the portal chatter in project sharing.
 *
 * @param {HTMLElement} node
 * @param {Object} params
 * @returns
 */
function compileChatter(node, params) {
    const chatterContainerXml = createElement('ChatterContainer');
    const parentURLQuery = new URLSearchParams(window.parent.location.search);
    setAttributes(chatterContainerXml, {
        token: `'${parentURLQuery.get('access_token')}'` || '',
        resModel: params.resModel,
        resId: params.resId,
        projectSharingId: params.projectSharingId,
    });
    const chatterContainerHookXml = createElement('div');
    chatterContainerHookXml.classList.add('o_FormRenderer_chatterContainer');
    append(chatterContainerHookXml, chatterContainerXml);
    return chatterContainerHookXml;
}

export class ProjectSharingChatterCompiler extends ViewCompiler {
    setup() {
        this.compilers.push({ selector: "t", fn: this.compileT });
        this.compilers.push({ selector: 'div.oe_chatter', fn: this.compileChatter });
    }

    compile(node, params) {
        const res = super.compile(node, params).children[0];
        const chatterContainerHookXml = res.querySelector(".o_FormRenderer_chatterContainer");
        if (chatterContainerHookXml) {
            setAttributes(chatterContainerHookXml, {
                "t-if": `uiService.size >= ${SIZES.XXL}`,
            });
            chatterContainerHookXml.classList.add('overflow-x-hidden', 'overflow-y-auto', 'o-aside', 'h-100');
        }
        return res;
    }

    compileT(node, params) {
        const compiledRoot = createElement("t");
        for (const child of node.childNodes) {
            const invisible = getModifier(child, "invisible");
            let compiledChild = this.compileNode(child, params, false);
            compiledChild = this.applyInvisible(invisible, compiledChild, {
                ...params,
                recordExpr: "model.root",
            });
            append(compiledRoot, compiledChild);
        }
        return compiledRoot;
    }

    compileChatter(node) {
        return compileChatter(node, {
            resId: 'model.root.resId or undefined',
            resModel: 'model.root.resModel',
            projectSharingId: 'model.root.context.active_id_chatter',
        });
    }
}

registry.category("form_compilers").add("portal_chatter_compiler", {
    selector: "div.oe_chatter",
    fn: (node) =>
        compileChatter(node, {
            resId: "props.record.resId or undefined",
            resModel: "props.record.resModel",
            projectSharingId: "props.record.context.active_id_chatter",
        }),
});

patch(FormCompiler.prototype, 'project_sharing_chatter', {
    compile(node, params) {
        const res = this._super(node, params);
        const chatterContainerHookXml = res.querySelector('.o_FormRenderer_chatterContainer');
        if (!chatterContainerHookXml) {
            return res; // no chatter, keep the result as it is
        }
        if (chatterContainerHookXml.parentNode.classList.contains('o_form_sheet')) {
            return res; // if chatter is inside sheet, keep it there
        }
        const formSheetBgXml = res.querySelector('.o_form_sheet_bg');
        const parentXml = formSheetBgXml && formSheetBgXml.parentNode;
        if (!parentXml) {
            return res; // miss-config: a sheet-bg is required for the rest
        }
        // after sheet bg (standard position, below form)
        setAttributes(chatterContainerHookXml, {
            't-if': `uiService.size < ${SIZES.XXL}`,
        });
        append(parentXml, chatterContainerHookXml);
        return res;
    }
});

```

## File: static\src\project_sharing\views\form\project_sharing_form_controller.js

```javascript
/** @odoo-module */

import { useService } from '@web/core/utils/hooks';
import { createElement } from "@web/core/utils/xml";
import { FormController } from '@web/views/form/form_controller';
import { useViewCompiler } from '@web/views/view_compiler';
import { ProjectSharingChatterCompiler } from './project_sharing_form_compiler';
import { ChatterContainer } from '../../components/chatter/chatter_container';

export class ProjectSharingFormController extends FormController {
    setup() {
        super.setup();
        this.uiService = useService('ui');
        const { arch, xmlDoc } = this.archInfo;
        const template = createElement('t');
        const xmlDocChatter = xmlDoc.querySelector("div.oe_chatter");
        if (xmlDocChatter && xmlDocChatter.parentNode.nodeName === "form") {
            template.appendChild(xmlDocChatter.cloneNode(true));
        }
        const mailTemplates = useViewCompiler(ProjectSharingChatterCompiler, arch, { Mail: template }, {});
        this.mailTemplate = mailTemplates.Mail;
    }

    getActionMenuItems() {
        return {};
    }

    get translateAlert() {
        return null;
    }
}

ProjectSharingFormController.components = {
    ...FormController.components,
    ChatterContainer,
}

```

## File: static\src\project_sharing\views\form\project_sharing_form_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="web.FormView" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o_form_view_container')]" position="after">
            <t t-if="mailTemplate">
                <t t-call="{{ mailTemplate }}" />
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\project_sharing\views\form\project_sharing_form_renderer.js

```javascript
/** @odoo-module */

import { ChatterContainer } from '../../components/chatter/chatter_container';
import { FormRenderer } from '@web/views/form/form_renderer';

export class ProjectSharingFormRenderer extends FormRenderer { }
ProjectSharingFormRenderer.components = {
    ...FormRenderer.components,
    ChatterContainer,
};

```

## File: static\src\project_sharing\views\form\project_sharing_form_view.js

```javascript
/** @odoo-module */

import { formView } from '@web/views/form/form_view';
import { ProjectSharingFormController } from './project_sharing_form_controller';
import { ProjectSharingFormRenderer } from './project_sharing_form_renderer';

formView.Controller = ProjectSharingFormController;
formView.Renderer = ProjectSharingFormRenderer;

```

## File: static\src\project_sharing\views\kanban\kanban_view.js

```javascript
/** @odoo-module */

import { kanbanView } from "@web/views/kanban/kanban_view";
import { KanbanDynamicGroupList, KanbanModel } from "@web/views/kanban/kanban_model";

export class ProjectSharingTaskKanbanDynamicGroupList extends KanbanDynamicGroupList {
    get context() {
        return {
            ...super.context,
            project_kanban: true,
        };
    }
}

export class ProjectSharingTaskKanbanModel extends KanbanModel {}

ProjectSharingTaskKanbanModel.DynamicGroupList = ProjectSharingTaskKanbanDynamicGroupList;

kanbanView.Model = ProjectSharingTaskKanbanModel;

```

## File: static\src\project_sharing\views\list\list_renderer.js

```javascript
/** @odoo-module */

import { ListRenderer } from "@web/views/list/list_renderer";
import { evalDomain } from "@web/views/utils";

const { onWillUpdateProps } = owl;

export class ProjectSharingListRenderer extends ListRenderer {
    setup() {
        super.setup(...arguments);
        this.setColumns(this.allColumns);
        onWillUpdateProps((nextProps) => {
            this.setColumns(nextProps.archInfo.columns);
        });
    }

    setColumns(columns) {
        if (this.props.list.records.length) {
            const allColumns = [];
            const firstRecord = this.props.list.records[0];
            for (const column of columns) {
                if (
                    column.modifiers.column_invisible &&
                    column.modifiers.column_invisible instanceof Array
                ) {
                    const result = evalDomain(column.modifiers.column_invisible, firstRecord.evalContext);
                    if (result) {
                        continue;
                    }
                }
                allColumns.push(column);
            }
            this.allColumns = allColumns;
        } else {
            this.allColumns = columns;
        }
        this.state.columns = this.allColumns.filter(
            (col) => !col.optional || this.optionalActiveFields[col.name]
        );
    }
}

```

## File: static\src\project_sharing\views\list\list_view.js

```javascript
/** @odoo-module */

import { listView } from "@web/views/list/list_view";

import { ProjectSharingListRenderer } from "./list_renderer";

const props = listView.props;
listView.props = function (genericProps, view) {
    const result = props(genericProps, view);
    return {
        ...result,
        allowSelectors: false,
    };
};
listView.Renderer = ProjectSharingListRenderer;

```

## File: static\src\services\project_task_recurrence.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";

import { ProjectStopRecurrenceConfirmationDialog } from '../components/project_stop_recurrence_confirmation_dialog/project_stop_recurrence_confirmation_dialog';

class ProjectTaskRecurrence {
    constructor(env, dialog, orm) {
        this.env = env;
        this.dialog = dialog;
        this.orm = orm;
        this.resModel = 'project.task';
    }

    async stopRecurrence(tasks, callback = () => {}) {
        const taskIdsWithRecurrence = [];
        const tasksPerRecurrence = {};
        const recurrenceIds = [];
        for (const task of tasks) {
            if (task.data.recurrence_id) {
                taskIdsWithRecurrence.push(task.resId);
                const recurrenceId = task.data.recurrence_id && task.data.recurrence_id[0];
                if (!(recurrenceId in tasksPerRecurrence)) {
                    tasksPerRecurrence[recurrenceId] = [task.resId];
                    recurrenceIds.push(recurrenceId);
                } else {
                    tasksPerRecurrence[recurrenceId].push(task.resId);
                }
            }
        }

        if (!recurrenceIds.length) {
            callback();
            return;
        }

        let allowContinue = false;
        if (recurrenceIds.length === 1) {
            const count = await this.orm.searchCount(
                this.resModel,
                [['recurrence_id', '=', recurrenceIds[0]]],
            );
            allowContinue = count != 1;
        } else {
            const taskReadGroup = await this.orm.readGroup(
                this.resModel,
                [['recurrence_id', 'in', recurrenceIds]],
                ['recurrence_id'],
                ['recurrence_id'],
            );
            allowContinue = true;
            for (const res of taskReadGroup) {
                const taskCount = tasksPerRecurrence[res.recurrence_id[0]].length;
                if (taskCount === res.recurrence_id_count) {
                    allowContinue = false;
                    break;
                }
            }
        }
        let dialogBody;
        if (tasks.length > 1) {
            dialogBody = allowContinue
                    ? this.env._t('It seems that some tasks are part of a recurrence.')
                    : this.env._t('It seems that some tasks are part of a recurrence. At least one of them must be kept as a model to create the next occurences.');
        } else {
            dialogBody = allowContinue
                    ? this.env._t('It seems that this task is part of a recurrence.')
                    : this.env._t('It seems that this task is recurrent. Would you like to stop its recurrence?');
        }

        const dialogProps = {
            body: dialogBody,
            confirm: async () => {
                await this.orm.call(
                    this.resModel,
                    'action_stop_recurrence',
                    [taskIdsWithRecurrence],
                );
                callback();
            },
            cancel: () => {},
        };
        if (allowContinue) {
            dialogProps.continueRecurrence = async () => {
                await this.orm.call(
                    this.resModel,
                    'action_continue_recurrence',
                    [taskIdsWithRecurrence],
                );
                callback();
            };
        }
        this.dialog.add(ProjectStopRecurrenceConfirmationDialog, dialogProps);
    }
}

export const taskRecurrenceService = {
    dependencies: ['dialog', 'orm'],
    async: [
        'stopRecurrence',
    ],
    start(env, { dialog, orm }) {
        return new ProjectTaskRecurrence(env, dialog, orm);
    }
};

registry.category('services').add('project_task_recurrence', taskRecurrenceService);

```

## File: static\src\utils\project_utils.js

```javascript
/** @odoo-module */

/**
 * List of colors according to the selection value, see `project_update.py`
 */
export const STATUS_COLORS = {
    'on_track': 10,
    'at_risk': 2,
    'off_track': 1,
    'on_hold': 4,
};

export const STATUS_COLOR_PREFIX = 'o_status_bubble mx-0 o_color_bubble_';

```

## File: static\src\views\burndown_chart\burndown_chart_model.js

```javascript
/** @odoo-module **/

import { GraphModel } from "@web/views/graph/graph_model";

export class BurndownChartModel extends GraphModel {
    /**
     * @protected
     * @override
     */
    async _loadDataPoints(metaData) {
        metaData.measures.__count.string = this.env._t('# of Tasks');
        return super._loadDataPoints(metaData);
    }
}

```

## File: static\src\views\burndown_chart\burndown_chart_search_model.js

```javascript
/** @odoo-module */

import { useService } from "@web/core/utils/hooks";
import { SearchModel } from "@web/search/search_model";


export class BurndownChartSearchModel extends SearchModel {

    /**
     * @override
     */
    setup(services) {
        this.notificationService = useService("notification");
        super.setup(...arguments);
    }

    /**
     * @override
     */
    async load(config) {
        await super.load(...arguments);
        // Store date and stage_id searchItemId in the SearchModel for reuse in other functions.
        for (const searchItem of Object.values(this.searchItems)) {
            if (['dateGroupBy', 'groupBy'].includes(searchItem.type)) {
                if (this.stageIdSearchItemId && this.dateSearchItemId) {
                    return;
                }
                switch (searchItem.fieldName) {
                    case 'date':
                        this.dateSearchItemId = searchItem.id;
                        break;
                    case 'stage_id':
                        this.stageIdSearchItemId = searchItem.id;
                        break;
                }
            }
        }
    }

    /**
     * @override
     */
    deactivateGroup(groupId) {
        // Prevent removing Date & Stage group by from the search
        if (this.searchItems[this.stageIdSearchItemId].groupId == groupId && this.searchItems[this.dateSearchItemId].groupId) {
            this._addGroupByNotification(this.env._t("Date and Stage"));
            return;
        }
        super.deactivateGroup(groupId);
    }

    /**
     * @override
     */
    toggleDateGroupBy(searchItemId, intervalId) {
        // Ensure that there is always one and only one date group by selected.
        if (searchItemId === this.dateSearchItemId) {
            let filtered_query = [];
            let triggerNotification = false;
            for (const queryElem of this.query) {
                if (queryElem.searchItemId !== searchItemId) {
                    filtered_query.push(queryElem);
                } else if (queryElem.intervalId === intervalId) {
                    triggerNotification = true;
                }
            }
            if (filtered_query.length !== this.query.length) {
                this.query = filtered_query;
                if (triggerNotification) {
                    this._addGroupByNotification(this.env._t("Date"));
                }
            }
        }
        super.toggleDateGroupBy(...arguments);
    }

    /**
     * @override
     */
    toggleSearchItem(searchItemId) {
        // Ensure that stage_id is always selected.
        if (searchItemId === this.stageIdSearchItemId
            && this.query.some(queryElem => queryElem.searchItemId === searchItemId)) {
            this._addGroupByNotification(this.env._t("Stage"));
            return;
        }
        super.toggleSearchItem(...arguments);
    }

    /**
     * Adds a notification relative to the group by constraint of the Burndown Chart.
     * @param fieldName The field name(s) the notification has to be related to.
     * @private
     */
    _addGroupByNotification(fieldName) {
        const notif = this.env._t("The Burndown Chart must be grouped by");
        this.notificationService.add(
            `${notif} ${fieldName}`,
            { type: "danger" }
        );
    }

    /**
     * @override
     */
    async _notify() {
        // Ensure that we always group by date firstly and by stage_id secondly
        let stageIdIndex = -1;
        let dateIndex = -1;
        for (const [index, queryElem] of this.query.entries()) {
            if (stageIdIndex !== -1 && dateIndex !== -1) {
                break;
            }
            switch (queryElem.searchItemId) {
                case this.dateSearchItemId:
                    dateIndex = index;
                    break;
                case this.stageIdSearchItemId:
                    stageIdIndex = index;
                    break;
            }
        }
        if (stageIdIndex > 0) {
            if (stageIdIndex > dateIndex) {
                dateIndex += 1;
            }
            this.query.splice(0, 0, this.query.splice(stageIdIndex, 1)[0]);
        }
        if (dateIndex > 0) {
            this.query.splice(0, 0, this.query.splice(dateIndex, 1)[0]);
        }
        await super._notify(...arguments);
    }

}

```

## File: static\src\views\burndown_chart\burndown_chart_view.js

```javascript
/** @odoo-module **/

import { BurndownChartModel } from "./burndown_chart_model";
import { graphView } from "@web/views/graph/graph_view";
import { registry } from "@web/core/registry";
import { BurndownChartSearchModel } from "./burndown_chart_search_model";

const viewRegistry = registry.category("views");

const burndownChartGraphView = {
  ...graphView,
  buttonTemplate: "project.BurndownChartView.Buttons",
  hideCustomGroupBy: true,
  Model: BurndownChartModel,
  searchMenuTypes: graphView.searchMenuTypes.filter(menuType => menuType !== "comparison"),
  SearchModel: BurndownChartSearchModel,
};

viewRegistry.add("burndown_chart", burndownChartGraphView);

```

## File: static\src\views\burndown_chart\burndown_chart_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.BurndownChartView.Buttons" t-inherit="web.GraphView.Buttons" t-inherit-mode="primary" owl="1">
        <xpath expr="//button[@data-mode='pie']" position="replace">
        </xpath>
        <xpath expr="//div[@role='toolbar'][@name='toggleOrderToolbar']" position="replace">
        </xpath>
    </t>

</templates>

```

## File: static\src\views\form_with_html_expander\form_renderer_with_html_expander.js

```javascript
/** @odoo-module */

import { useService } from '@web/core/utils/hooks';
import { FormRenderer } from '@web/views/form/form_renderer';

const { useRef, useEffect } = owl;

export class FormRendererWithHtmlExpander extends FormRenderer {
    setup() {
        super.setup();
        this.ui = useService('ui');
        const ref = useRef('compiled_view_root');
        useEffect(
            (el, size) => {
                if (el && size === 6) {
                    const descriptionField = el.querySelector(this.htmlFieldQuerySelector);
                    if (descriptionField) {
                        const editor = descriptionField.querySelector('.note-editable');
                        const elementToResize = editor || descriptionField;
                        const { bottom, height } = elementToResize.getBoundingClientRect();
                        const minHeight = document.documentElement.clientHeight - bottom - height;
                        elementToResize.style.minHeight = `${minHeight}px`;
                    }
                }
            },
            () => [ref.el, this.ui.size, this.props.record.mode],
        );
    }

    get htmlFieldQuerySelector() {
        return '.oe_form_field.oe_form_field_html';
    }
}

```

## File: static\src\views\form_with_html_expander\form_view_with_html_expander.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { formView } from '@web/views/form/form_view';
import { FormRendererWithHtmlExpander } from './form_renderer_with_html_expander';

export const formViewWithHtmlExpander = {
    ...formView,
    Renderer: FormRendererWithHtmlExpander,
};

registry.category('views').add('form_description_expander', formViewWithHtmlExpander);

```

## File: static\src\views\project_calendar\project_calendar_controller.js

```javascript
/** @odoo-module **/

import { CalendarController } from "@web/views/calendar/calendar_controller";

export class ProjectCalendarController extends CalendarController {
    setup() {
        super.setup(...arguments);
        this.displayName += this.env._t(" - Tasks by Deadline");
    }
}

```

## File: static\src\views\project_calendar\project_calendar_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { calendarView } from "@web/views/calendar/calendar_view";
import { ProjectCalendarController } from "@project/views/project_calendar/project_calendar_controller";
import { ProjectControlPanel } from "@project/components/project_control_panel/project_control_panel";

export const projectCalendarView = {
    ...calendarView,
    Controller: ProjectCalendarController,
    ControlPanel: ProjectControlPanel,
};
registry.category("views").add("project_calendar", projectCalendarView);

```

## File: static\src\views\project_form\project_form_renderer.js

```javascript
/** @odoo-module */

import { FormRendererWithHtmlExpander } from "../form_with_html_expander/form_renderer_with_html_expander";

export class ProjectFormRenderer extends FormRendererWithHtmlExpander {
    get htmlFieldQuerySelector() {
        return '.o_field_html[name="description"]';
    }
}

```

## File: static\src\views\project_form\project_form_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { formViewWithHtmlExpander } from '../form_with_html_expander/form_view_with_html_expander';
import { ProjectFormRenderer } from "./project_form_renderer";

export const projectFormView = {
    ...formViewWithHtmlExpander,
    Renderer: ProjectFormRenderer,
};

registry.category("views").add("project_form", projectFormView);

```

## File: static\src\views\project_task_form\project_task_form_controller.js

```javascript
/** @odoo-module */

import { useService } from '@web/core/utils/hooks';
import { FormController } from '@web/views/form/form_controller';

export class ProjectTaskFormController extends FormController {
    setup() {
        super.setup();
        this.taskRecurrence = useService('project_task_recurrence');
    }

    getActionMenuItems() {
        if (!(this.archiveEnabled && this.model.root.isActive) || !this.model.root.data.recurrence_id) {
            return super.getActionMenuItems();
        }
        this.archiveEnabled = false;
        const actionMenuItems = super.getActionMenuItems();
        this.archiveEnabled = true;
        if (actionMenuItems) {
            actionMenuItems.other.unshift({
                description: this.env._t('Archive'),
                callback: () => this.taskRecurrence.stopRecurrence(
                    [this.model.root],
                    () => this.model.root.archive(),
                ),
            });
        }
        return actionMenuItems;
    }

    deleteRecord() {
        if (!this.model.root.data.recurrence_id) {
            return super.deleteRecord();
        }
        this.taskRecurrence.stopRecurrence(
            [this.model.root],
            () => {
                this.model.root.delete();
                if (!this.model.root.resId) {
                    this.env.config.historyBack();
                }
            }
        );
    }
}

```

## File: static\src\views\project_task_form\project_task_form_renderer.js

```javascript
/** @odoo-module */

import { FormRendererWithHtmlExpander } from "../form_with_html_expander/form_renderer_with_html_expander";

export class ProjectTaskFormRenderer extends FormRendererWithHtmlExpander {
    get htmlFieldQuerySelector() {
        return '.o_field_html[name="description"]';
    }
}

```

## File: static\src\views\project_task_form\project_task_form_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { formViewWithHtmlExpander } from '../form_with_html_expander/form_view_with_html_expander';
import { ProjectTaskFormController } from './project_task_form_controller';
import { ProjectTaskFormRenderer } from "./project_task_form_renderer";

export const projectTaskFormView = {
    ...formViewWithHtmlExpander,
    Controller: ProjectTaskFormController,
    Renderer: ProjectTaskFormRenderer,
};

registry.category("views").add("project_task_form", projectTaskFormView);

```

## File: static\src\views\project_task_kanban\project_task_kanban_dynamic_group_list.js

```javascript
/** @odoo-module */

import { KanbanDynamicGroupList } from "@web/views/kanban/kanban_model";
import { Domain } from '@web/core/domain';
import { session } from '@web/session';

export class ProjectTaskKanbanDynamicGroupList extends KanbanDynamicGroupList {
    get context() {
        const context = super.context;
        context.project_kanban = true;
        if (context.createPersonalStageGroup) {
            context.default_user_id = context.uid;
            delete context.createPersonalStageGroup;
            delete context.default_project_id;
        }
        return context;
    }

    get isGroupedByStage() {
        return !!this.groupByField && this.groupByField.name === 'stage_id';
    }

    get isGroupedByPersonalStages() {
        return !!this.groupByField && this.groupByField.name === 'personal_stage_type_ids';
    }

    async _loadGroups() {
        if (!this.isGroupedByPersonalStages) {
            return super._loadGroups(...arguments);
        }
        const previousDomain = this.domain;
        this.domain = Domain.and([[['user_ids', 'in', session.uid]], previousDomain]).toList({});
        const result = await super._loadGroups(...arguments);
        this.domain = previousDomain;
        return result;
    }

    async createGroup() {
        if (this.isGroupedByPersonalStages) {
            this.defaultContext = Object.assign({}, this.defaultContext || {}, {
                createPersonalStageGroup: true,
            });
        }
        const result = await super.createGroup(...arguments);
        if (this.isGroupedByPersonalStages) {
            delete this.defaultContext.createPersonalStageGroup;
        }
        return result;
    }
}

```

## File: static\src\views\project_task_kanban\project_task_kanban_model.js

```javascript
/** @odoo-module */

import { KanbanModel } from "@web/views/kanban/kanban_model";

import { ProjectTaskKanbanDynamicGroupList } from "./project_task_kanban_dynamic_group_list";
import { ProjectTaskRecord } from './project_task_kanban_record';

export class ProjectTaskKanbanGroup extends KanbanModel.Group {
    get isPersonalStageGroup() {
        return !!this.groupByField && this.groupByField.name === 'personal_stage_type_ids';
    }

    async delete() {
        if (this.isPersonalStageGroup) {
            this.deleted = true;
            return await this.model.orm.call(this.resModel, 'remove_personal_stage', [this.resId]);
        } else {
            return await super.delete();
        }
    }
}

export class ProjectTaskKanbanModel extends KanbanModel { }

ProjectTaskKanbanModel.DynamicGroupList = ProjectTaskKanbanDynamicGroupList;
ProjectTaskKanbanModel.Group = ProjectTaskKanbanGroup;
ProjectTaskKanbanModel.Record = ProjectTaskRecord;

```

## File: static\src\views\project_task_kanban\project_task_kanban_record.js

```javascript
/* @odoo-module */

import { Record } from '@web/views/relational_model';

export class ProjectTaskRecord extends Record {
    async _applyChanges(changes) {
        const value = changes.personal_stage_type_ids;
        if (value && Array.isArray(value)) {
            delete changes.personal_stage_type_ids;
            changes.personal_stage_type_id = value;
        }
        await super._applyChanges(changes);
    }
}

```

## File: static\src\views\project_task_kanban\project_task_kanban_renderer.js

```javascript
/** @odoo-module */

import { useService } from '@web/core/utils/hooks';
import { KanbanRenderer } from '@web/views/kanban/kanban_renderer';
import { FormViewDialog } from "@web/views/view_dialogs/form_view_dialog";

const { onWillStart } = owl;

export class ProjectTaskKanbanRenderer extends KanbanRenderer {
    setup() {
        super.setup();
        this.userService = useService('user');
        this.action = useService('action');

        this.isProjectManager = false;
        onWillStart(this.onWillStart);
    }

    get canMoveRecords() {
        let canMoveRecords = super.canMoveRecords;
        if (!canMoveRecords && this.canResequenceRecords && this.props.list.isGroupedByPersonalStages) {
            const { groupByField } = this.props.list;
            const { modifiers } = groupByField;
            canMoveRecords = !(modifiers && modifiers.readonly);
        }
        return canMoveRecords;
    }

    get canResequenceGroups() {
        let canResequenceGroups = super.canResequenceGroups;
        if (!canResequenceGroups && this.props.list.isGroupedByPersonalStages) {
            const { modifiers } = this.props.list.groupByField;
            const { groupsDraggable } = this.props.archInfo;
            canResequenceGroups = groupsDraggable && !(modifiers && modifiers.readonly);
        }
        return canResequenceGroups;
    }

    async onWillStart() {
        if (!this.props.list.isGroupedByPersonalStages) { // no need to check it if the group by is personal stages
            this.isProjectManager = await this.userService.hasGroup('project.group_project_manager');
        }
    }

    canCreateGroup() {
        return super.canCreateGroup() && (!this.props.list.isGroupedByStage || this.isProjectManager) || this.props.list.isGroupedByPersonalStages;
    }

    canDeleteGroup(group) {
        return super.canDeleteGroup(group) && (!this.props.list.isGroupedByStage || this.isProjectManager) || this.props.list.isGroupedByPersonalStages;
    }

    canEditGroup(group) {
        return super.canEditGroup(group) && (!this.props.list.isGroupedByStage || this.isProjectManager) || this.props.list.isGroupedByPersonalStages;
    }

    async deleteGroup(group) {
        if (group && group.groupByField.name === 'stage_id') {
            const action = await group.model.orm.call(
                group.resModel,
                'unlink_wizard',
                [group.resId],
                { context: group.context },
            );
            this.action.doAction(action);
            return;
        }
        super.deleteGroup(group);
    }

    editGroup(group) {
        const groupBy = this.props.list.groupBy;
        if (groupBy.length !== 1 || groupBy[0] !== 'personal_stage_type_ids') {
            super.editGroup(group);
            return;
        }
        const context = Object.assign({}, group.context, {
            form_view_ref: 'project.personal_task_type_edit',
        });
        this.dialog.add(FormViewDialog, {
            context,
            resId: group.value,
            resModel: group.resModel,
            title: this.env._t('Edit Personal Stage'),
            onRecordSaved: async () => {
                await this.props.list.load();
                this.props.list.model.notify();
            },
        });
    }
}

```

## File: static\src\views\project_task_kanban\project_task_kanban_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { kanbanView } from '@web/views/kanban/kanban_view';
import { ProjectTaskKanbanModel } from "./project_task_kanban_model";
import { ProjectTaskKanbanRenderer } from './project_task_kanban_renderer';
import { ProjectControlPanel } from "../../components/project_control_panel/project_control_panel";

export const projectTaskKanbanView = {
    ...kanbanView,
    Model: ProjectTaskKanbanModel,
    Renderer: ProjectTaskKanbanRenderer,
    ControlPanel: ProjectControlPanel,
};

registry.category('views').add('project_task_kanban', projectTaskKanbanView);

```

## File: static\src\views\project_task_list\project_task_list_controller.js

```javascript
/** @odoo-module */

import { useService } from '@web/core/utils/hooks';
import { ListController } from '@web/views/list/list_controller';

export class ProjectTaskListController extends ListController {
    setup() {
        super.setup();
        this.taskRecurrence = useService('project_task_recurrence');
    }

    getActionMenuItems() {
        if (!this.archiveEnabled || this.model.root.isM2MGrouped) {
            return super.getActionMenuItems();
        }
        const hasAnyRecurrences = this._anySelectedTasksWithRecurrence();
        this.archiveEnabled = !hasAnyRecurrences;
        const actionMenuItems = super.getActionMenuItems();
        this.archiveEnabled = true;
        if (actionMenuItems && hasAnyRecurrences) {
            actionMenuItems.other.splice(
                this.isExportEnable ? 1 : 0,
                0,
                {
                    description: this.env._t('Archive'),
                    callback: () => this.taskRecurrence.stopRecurrence(
                        this.model.root.selection,
                        () => this.toggleArchiveState(true),
                    ),
                },
                {
                    description: this.env._t('Unarchive'),
                    callback: () => this.toggleArchiveState(false),
                },
            );
        }
        return actionMenuItems;
    }

    onDeleteSelectedRecords() {
        if (this._anySelectedTasksWithRecurrence()) {
            return this.taskRecurrence.stopRecurrence(
                this.model.root.selection,
                () => this.model.root.deleteRecords(),
            );
        }
        return super.onDeleteSelectedRecords();
    }

    _anySelectedTasksWithRecurrence() {
        for (const selectedTask of this.model.root.selection) {
            if (selectedTask.data.recurrence_id) {
                return true;
            }
        }
        return false;
    }
}

```

## File: static\src\views\project_task_list\project_task_list_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { listView } from '@web/views/list/list_view';
import { ProjectControlPanel } from "../../components/project_control_panel/project_control_panel";
import { ProjectTaskListController } from './project_task_list_controller';

export const projectTaskListView = {
    ...listView,
    Controller: ProjectTaskListController,
    ControlPanel: ProjectControlPanel,
};

registry.category("views").add("project_task_list", projectTaskListView);

```

## File: static\src\views\project_update_kanban\project_update_kanban_controller.js

```javascript
/** @odoo-module */

import { KanbanController } from '@web/views/kanban/kanban_controller';
import { ProjectRightSidePanel } from '../../components/project_right_side_panel/project_right_side_panel';

export class ProjectUpdateKanbanController extends KanbanController {
    get className() {
        return super.className + ' o_controller_with_rightpanel';
    }
}

ProjectUpdateKanbanController.components = {
    ...KanbanController.components,
    ProjectRightSidePanel,
};
ProjectUpdateKanbanController.template = 'project.ProjectUpdateKanbanView';

```

## File: static\src\views\project_update_kanban\project_update_kanban_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectUpdateKanbanView" t-inherit="web.KanbanView" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-component='props.Renderer']" position="before">
            <ProjectRightSidePanel
                context="props.context"
                domain="props.domain"
            />
        </xpath>
    </t>

</templates>

```

## File: static\src\views\project_update_kanban\project_update_kanban_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { kanbanView } from "@web/views/kanban/kanban_view";
import { ProjectUpdateKanbanController } from './project_update_kanban_controller';

export const projectUpdateKanbanView = {
    ...kanbanView,
    Controller: ProjectUpdateKanbanController,
};

registry.category('views').add('project_update_kanban', projectUpdateKanbanView);

```

## File: static\src\views\project_update_list\project_update_list_controller.js

```javascript
/** @odoo-module */

import { ListController } from '@web/views/list/list_controller';
import { ProjectRightSidePanel } from '../../components/project_right_side_panel/project_right_side_panel';

export class ProjectUpdateListController extends ListController {
    get className() {
        return super.className + ' o_controller_with_rightpanel';
    }
}

ProjectUpdateListController.components = {
    ...ListController.components,
    ProjectRightSidePanel,
};
ProjectUpdateListController.template = 'project.ProjectUpdateListView';

```

## File: static\src\views\project_update_list\project_update_list_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectUpdateListView" t-inherit="web.ListView" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-component='props.Renderer']" position="before">
            <ProjectRightSidePanel
                context="props.context"
                domain="props.domain"
            />
        </xpath>
    </t>

</templates>

```

## File: static\src\views\project_update_list\project_update_list_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";
import { ProjectUpdateListController } from './project_update_list_controller';

export const projectUpdateListView = {
    ...listView,
    Controller: ProjectUpdateListController,
};

registry.category('views').add('project_update_list', projectUpdateListView);

```

## File: static\src\xml\project_task_kanban_examples.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.example.generic" owl="1">
      Prioritize Tasks by using the <a style="color: gold;" class="fa fa-star"></a> icon.
      <br/>
      Use the <span class="o_status d-inline-block o_status_green"></span> button to inform your colleagues that a task is ready for the next stage.
      <br/>
      Use the <span class="o_status d-inline-block o_status_red"></span> to indicate a problem or a need for discussion on a task.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.agilescrum" owl="1">
      Use <span class="o_status d-inline-block o_status_green"></span> and <span class="o_status d-inline-block o_status_red"></span> bullets to indicate the status of a task.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.digitalmarketing" owl="1">
      Everyone can propose ideas, and the Editor marks the best ones as <span class="o_status d-inline-block o_status_green"></span>.
      Attach all documents or links to the task directly, to have all research information centralized. 
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>
  
    <t t-name="project.example.customerfeedback" owl="1">
      Customers propose feedbacks by email; Odoo creates tasks automatically, and you can
      communicate on the task directly. Your managers decide which feedback is accepted
      <span class="o_status d-inline-block o_status_green"></span> and which feedback is
      moved to the "Refused" column.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.consulting" owl="1">
      Manage the lifecycle of your project using the kanban view. Add newly acquired projects,
      assign them and use the <span class="o_status d-inline-block o_status_green"></span> and
      <span class="o_status d-inline-block o_status_red"></span> to define if the project is
      ready for the next step.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.researchproject" owl="1">
      Handle your idea gathering within Tasks of your new Project and discuss them in the chatter of the tasks. Use the
      <span class="o_status d-inline-block o_status_green"></span> and <span class="o_status d-inline-block o_status_red"></span>
      to signalize what is the current status of your Idea.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.tshirtprinting" owl="1">
      Communicate with customers on the task using the email gateway. Attach logo designs to the task, so that information flows from
      designers to the workers who print the t-shirt. Organize priorities amongst orders using the
      <a style="color: gold;" class="fa fa-star"></a> icon.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

</templates>

```

## File: static\src\xml\project_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <span t-name="project.statusWithColor" t-att-class="'o_status_bubble me-2 o_color_bubble_' + color"></span>

    <div t-name="project.ControlPanel" t-inherit="web.Legacy.ControlPanel" t-inherit-mode="extension" owl="1">
        <xpath expr="//ol[hasclass('breadcrumb')]" position="inside">
            <t t-call="project.ProjectControlPanelContent">
                <t t-set="showProjectUpdate" t-value="show_project_update"/>
                <t t-set="isProjectUser" t-value="is_project_user"/>
            </t>
        </xpath>
    </div>

    <t t-name="project.task.PrivateProjectName">
        <span class="fst-italic text-muted"><i class="fa fa-lock"></i> Private</span>
    </t>

</templates>

```

## File: upgrades\1.3\pre-migrate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


def migrate(cr, version):
    cr.execute("""
        UPDATE ir_model_access a
           SET perm_read = true
          FROM ir_model_data d
         WHERE d.res_id = a.id
           AND d.model = 'ir.model.access'
           AND d.module = 'project'
           AND d.name = 'access_project_milestone_portal'
        """)

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
        <field name="priority">40</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpis']/group[last()]" position="before">
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
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No activity types found. Let's create one!
            </p><p>
                Those represent the different categories of things you have to do (e.g. "Call" or "Send email").
            </p>
        </field>
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

    <record id="project_sharing_access_view_tree" model="ir.ui.view">
        <field name="name">project.collaborator.view.tree</field>
        <field name="model">project.collaborator</field>
        <field name="arch" type="xml">
            <tree string="Project Collaborators" create="0">
                <field name="partner_id" options="{'no_create': True}"/>
                <field name="partner_email"/>
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
        <field name="view_mode">tree</field>
        <field name="domain">[('project_id', '=', active_id)]</field>
        <field name="search_view_id" ref="project_collaborator_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No collaborators found
            </p>
            <p>
                Collaborate efficiently with key stakeholders by sharing with them the Kanban view of your tasks. Collaborators will be able to edit parts of tasks and send messages.
            </p>
        </field>
    </record>

</odoo>

```

## File: views\project_milestone_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_milestone_view_form" model="ir.ui.view">
        <field name="name">project.milestone.view.form</field>
        <field name="model">project.milestone</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="%(project.action_view_task_from_milestone)d"
                                type="action"
                                class="oe_stat_button"
                                icon="fa-tasks"
                                attrs="{'invisible': [('task_count', '=', 0)]}"
                                context="{'default_project_id': project_id}"
                                groups="project.group_project_milestone"
                                close="1"
                        >
                            <!-- TODO: Remove me in master -->
                            <field name="task_count" string="Tasks" widget="statinfo" invisible="1"/>
                            <div class="o_form_field o_stat_info">
                                <span class="o_stat_value">
                                    <field name="task_count" widget="statinfo" nolabel="1"/>
                                </span>
                                <span class="o_stat_text">Tasks</span>
                            </div>
                        </button>
                    </div>
                    <group>
                        <group name="main_details">
                            <field name="project_id" invisible="1"/>
                            <field name="name" placeholder="e.g: Product Launch"/>
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
            <tree decoration-success="can_be_marked_as_done" decoration-danger="is_deadline_exceeded and not can_be_marked_as_done" decoration-muted="is_reached" editable="bottom" sample="1">
                <field name="name"/>
                <field name="deadline" optional="show"/>
                <field name="is_reached" optional="show"/>
                <field name="is_deadline_exceeded" invisible="1"/>
                <field name="task_count" invisible="1" />
                <field name="can_be_marked_as_done" invisible="1"/>
                <button name="action_view_tasks"
                        type="object"
                        title="View Tasks"
                        string="View Tasks"
                        class="btn btn-link float-end"
                        attrs="{'invisible': [('task_count', '=', 0)]}"
                        groups="project.group_project_milestone"
                />
            </tree>
        </field>
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
            <li t-if="page_name in ['project_task', 'project_subtasks'] and project" class="breadcrumb-item active">
                <a t-if="project" t-attf-href="/my/projects/{{ project.id }}?{{ keep_query() }}"><t t-esc="project.name"/></a>
            </li>
            <li t-elif="project" t-attf-class="breadcrumb-item #{'active ' if not project else ''} text-truncate col-8 col-lg-10">
                <t t-esc="project.name"/>
            </li>
            <li t-if="page_name == 'task' or (task and not project)" t-attf-class="breadcrumb-item #{'active ' if not task else ''}">
                <a t-if="task" t-attf-href="/my/tasks?{{ keep_query() }}">Tasks</a>
                <t t-else="">Tasks</t>
            </li>
            <li t-if="page_name == 'project_subtasks' and task and project" class="breadcrumb-item active">
                <a t-attf-href="/my/projects/{{ project.id }}/task/{{ task.id }}?{{ keep_query() }}"><t t-esc="task.name"/></a>
            </li>
            <li t-elif="task" class="breadcrumb-item active text-truncate">
                <span t-field="task.name"/>
            </li>
            <li t-if="page_name == 'project_subtasks' or (task and subtask and project)" t-attf-class="breadcrumb-item text-truncate #{'active ' if not subtask else ''}">
                <a t-if="subtask" t-attf-href="/my/tasks/{{ task.id }}/subtasks?{{ keep_query() }}">Sub-tasks</a>
                <t t-else="">Sub-tasks</t>
            </li>
            <li t-if="subtask" class="breadcrumb-item active text-truncate">
                <span t-field="subtask.name"/>
            </li>
        </xpath>
    </template>

    <template id="portal_my_tasks_priority_widget_template" name="Priority Widget Template">
        <span t-attf-class="o_priority_star fa fa-star#{'' if task.priority == '1' else '-o'}" t-attf-title="Priority: {{'Important' if task.priority == '1' else 'Normal'}}"/>
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
                            <a t-attf-href="/my/projects/#{project.id}?{{ keep_query() }}"><span t-field="project.name"/></a>
                        </td>
                        <td class="text-end">
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
                <thead>
                    <tr>
                        <!-- Allows overrides in modules -->
                        <t t-set="group_by_in_header_list" t-value="['priority', 'status', 'project', 'stage', 'milestone']"></t>
                        <t t-set="number_of_header" t-value="8"></t>
                        <!-- Computes the right colspan once and use it everywhere -->
                        <t t-set="grouped_tasks_colspan" t-value="number_of_header - 1 if groupby in group_by_in_header_list else number_of_header"></t>
                        <t t-set="grouped_tasks_colspan" t-value="grouped_tasks_colspan if allow_milestone else grouped_tasks_colspan - 1"></t>
                        <th t-attf-colspan="{{2 if groupby != 'priority' else 1}}"/>
                        <th>Name</th>
                        <th>Assignees</th>
                        <th t-if="groupby != 'milestone' and allow_milestone" name="project_portal_milestones">Milestone</th>
                        <th t-if="groupby != 'status'"/>
                        <th t-if="groupby != 'project'">Project</th>
                        <th t-if="groupby != 'stage'" class="text-end">Stage</th>
                    </tr>
                </thead>
                <t t-foreach="grouped_tasks" t-as="tasks">
                    <tbody t-if="tasks">
                        <tr t-if="not groupby == 'none'" class="table-light">
                            <th t-if="groupby == 'project'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <!-- This div is necessary for documents_project_sale -->
                                <div name="project_name" class="d-flex w-100 align-items-center">
                                    <span t-field="tasks[0].sudo().project_id.name"/>
                                </div>
                            </th>
                            <th t-if="groupby == 'milestone'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <span t-if="tasks[0].sudo().milestone_id and tasks[0].sudo().allow_milestones"
                                      class="text-truncate"
                                      t-field="tasks[0].sudo().milestone_id.name"/>
                                <span t-else="">No Milestone</span>
                            </th>
                            <th t-if="groupby == 'stage'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <span class="text-truncate" t-field="tasks[0].sudo().stage_id.name"/></th>
                            <th t-if="groupby == 'priority'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <span class="text-truncate" t-field="tasks[0].sudo().priority"/></th>
                            <th t-if="groupby == 'status'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <span class="text-truncate" t-field="tasks[0].sudo().kanban_state"/></th>
                            <th t-if="groupby == 'customer'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <span t-if="tasks[0].sudo().partner_id"
                                      class="text-truncate"
                                      t-field="tasks[0].sudo().partner_id.name"/>
                                <span t-else="">No Customer</span>
                            </th>
                        </tr>
                    </tbody>
                    <tbody t-if="tasks">
                        <t t-foreach="tasks" t-as="task">
                            <tr>
                                <td class="text-start">
                                    #<span t-esc="task.id"/>
                                </td>
                                <td t-if="groupby != 'priority'" class="text-end">
                                    <t t-call="project.portal_my_tasks_priority_widget_template"/>
                                </td>
                                <td>
                                    <a t-attf-href="/my/#{task_url}/#{task.id}?{{ keep_query() }}"><span t-field="task.name"/></a>
                                </td>
                                <td>
                                    <t t-set="assignees" t-value="task.sudo().user_ids"/>
                                    <div t-if="assignees" class="row flex-nowrap ps-3">
                                        <img class="rounded-circle o_portal_contact_img me-2 px-0" t-attf-src="#{image_data_uri(assignees[:1].avatar_128)}" alt="User" style="width: 20px; height: 20px;"/>
                                        <span t-out="'%s%s' % (assignees[:1].name, ' + %s others' % len(assignees[1:]) if len(assignees.user_ids) > 1 else '')" t-att-title="'\n'.join(assignees.mapped('name'))"/>
                                    </div>
                                </td>
                                <td t-if="groupby != 'milestone' and allow_milestone" name="project_portal_milestones">
                                    <t t-if="task.milestone_id and task.allow_milestones">
                                        <span t-esc="task.milestone_id.name" />
                                    </t>
                                </td>
                                <td t-if="groupby != 'status'" align="right">
                                    <t t-call="project.portal_my_tasks_state_widget_template">
                                        <t t-set="path" t-value="'tasks'"/>
                                    </t>
                                </td>
                                <td t-if="groupby != 'project'">
                                    <span class="badge rounded-pill text-bg-info mw-100 text-truncate" title="Current project of the task" t-esc="task.project_id.name" />
                                </td>
                                <td t-if="groupby != 'stage'" class="text-end">
                                    <span t-attf-class="badge #{'text-bg-primary' if task.stage_id.fold else 'text-bg-light'}" title="Current stage of the task" t-esc="task.stage_id.name"/>
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

    <template id="portal_my_task" name="My Task" inherit_id="portal.portal_sidebar" primary="True">
        <xpath expr="//div[hasclass('o_portal_sidebar')]" position="inside">
            <t t-set="o_portal_fullwidth_alert" groups="project.group_project_user">
                <t t-call="portal.portal_back_in_edit_mode">
                    <t t-set="backend_url" t-value="'/web#model=project.task&amp;id=%s&amp;action=%s&amp;view_type=form' % (task.id, task.env.ref('project.action_view_all_task').id)"/>
                </t>
            </t>

            <div class="row mt16 o_project_portal_sidebar">
                <t t-call="portal.portal_record_sidebar">
                    <t t-set="classes" t-value="'col-lg-3 d-print-none'"/>

                    <t t-set="entries">
                        <ul class="list-group list-group-flush flex-wrap flex-row flex-lg-column">
                            <li id="task-nav" class="list-group-item ps-0 flex-grow-1 d-flex align-items-center" t-ignore="true" role="complementary">
                                <ul class="nav flex-column">
                                    <li class="nav-item" id="nav-header">
                                        <a class="nav-link ps-3" href="#card_header" style="max-width: 200px;">
                                            Task
                                        </a>
                                    </li>
                                    <li class="nav-item" id="nav-chat">
                                        <a class="nav-link ps-3" href="#task_chat">
                                            History
                                        </a>
                                    </li>
                                </ul>
                            </li>
                            <li id="task-links" t-if="task_link_section" class="list-group-item ps-0 flex-grow-1 d-flex align-items-center" t-ignore="true" role="complementary">
                                <ul class="nav flex-column">
                                    <t t-foreach="task_link_section" t-as="task_link">
                                        <li class="nav-item">
                                            <a class="nav-link ps-3" t-att-href="task_link['access_url']">
                                                <t t-out="task_link['title']"/>
                                            </a>
                                        </li>
                                    </t>
                                </ul>
                            </li>

                            <li t-if="task.user_ids or task.partner_id" class="list-group-item flex-grow-1">
                                <div class="col-12 col-md-12 pb-2" t-if="task.user_ids">
                                    <strong>Assignees</strong>
                                    <t t-foreach="task.user_ids" t-as="user">
                                        <div class="d-flex mb-3 flex-nowrap mt-1">
                                            <img class="rounded-circle o_portal_contact_img" t-att-src="image_data_uri(user.avatar_128)" alt="Contact"/>
                                            <div class="ms-2">
                                                <div t-esc="user" t-options='{"widget": "contact", "fields": ["name"]}'/>
                                                <a t-attf-href="tel:{{user.phone}}" t-if="user.phone"><div t-esc="user" t-options='{"widget": "contact", "fields": ["phone"]}'/></a>
                                                <a t-if="user.email" class="text-break" t-attf-href="mailto:{{user.email}}">
                                                    <div t-out="user" t-options='{"widget": "contact", "fields": ["email"]}'/>
                                                </a>
                                            </div>
                                        </div>
                                    </t>
                                </div>
                                <div class="col-12 col-md-12 pb-2" t-if="task.partner_id">
                                    <strong>Customer</strong>
                                    <div class="d-flex flex-nowrap mt-1">
                                        <img class="rounded-circle o_portal_contact_img" t-att-src="image_data_uri(task.partner_id.avatar_128)" alt="Contact"/>
                                        <div class="ms-2">
                                            <div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["name"]}'/>
                                            <a t-attf-href="tel:{{task.partner_id.phone}}" t-if="task.partner_id.phone"><div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["phone"]}'/></a>
                                            <a t-if="task.partner_id.email" class="text-break" t-attf-href="mailto:{{task.partner_id.email}}">
                                                <div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["email"]}'/>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </li>
                        </ul>
                    </t>
                </t>
                <div id="task_content" class="col-lg-9 justify-content-end">
                    <div id="card" class="card">
                        <div id="card_header" class="card-header bg-white" data-anchor="true">
                            <div class="row g-0">
                                <div class="col-12">
                                    <h5 class="d-flex mb-1 mb-md-0 row">
                                        <div class="col-9">
                                            <t t-call="project.portal_my_tasks_priority_widget_template"/>
                                            <span t-field="task.name" class="text-truncate"/>
                                            <small class="text-muted d-none d-md-inline"> (#<span t-field="task.id"/>)</small>
                                        </div>
                                        <div class="col-3 text-end">
                                            <small class="text-end">Stage:</small>
                                            <span t-field="task.stage_id.name" class=" badge rounded-pill text-bg-info" title="Current stage of this task"/>
                                        </div>
                                    </h5>
                                </div>
                            </div>
                        </div>
                        <div id="card_body" class="card-body">
                            <div class="float-end">
                                <t t-call="project.portal_my_tasks_state_widget_template">
                                    <t t-set="path" t-value="'task'"/>
                                </t>
                            </div>
                            <div class="row mb-4 container">
                                <div class="col-12 col-md-6">
                                    <div t-if="project_accessible"><strong>Project:</strong> <a t-attf-href="/my/projects/#{task.project_id.id}" t-field="task.project_id"/></div>
                                    <div t-else=""><strong>Project:</strong> <a t-field="task.project_id"/></div>
                                    <div t-if="task.date_deadline"><strong>Deadline:</strong> <span t-field="task.date_deadline" t-options='{"widget": "date"}'/></div>
                                    <div t-if="task.milestone_id and task.allow_milestones"><strong>Milestone:</strong> <span t-field="task.milestone_id"/></div>
                                    <div name="portal_my_task_planned_hours">
                                        <t t-call="project.portal_my_task_planned_hours_template"/>
                                    </div>
                                </div>
                                <div class="col-12 col-md-6" name="portal_my_task_second_column"></div>
                            </div>

                            <div class="row" t-if="task.description or task.attachment_ids">
                                <div t-if="not is_html_empty(task.description)" t-attf-class="col-12 col-lg-7 mb-4 mb-md-0 {{'col-lg-7' if task.attachment_ids else 'col-lg-12'}}">
                                    <hr class="mb-1"/>
                                    <div class="d-flex my-2">
                                        <strong>Description</strong>
                                    </div>
                                    <div class="py-1 px-2 bg-100 small table-responsive" t-field="task.description"/>
                                </div>
                                <div t-if="task.attachment_ids" t-attf-class="col-12 col-lg-5 o_project_portal_attachments {{'col-lg-5' if task.description else 'col-lg-12'}}">
                                    <hr class="mb-1 d-none d-lg-block"/>
                                    <strong class="d-block mb-2">Attachments</strong>
                                    <div class="row">
                                        <div t-attf-class="col {{'col-lg-6' if not task.description else 'col-lg-12'}}">
                                            <ul class="list-group">
                                                <a class="list-group-item list-group-item-action d-flex align-items-center oe_attachments py-1 px-2" t-foreach='task.attachment_ids' t-as='attachment' t-attf-href="/web/content/#{attachment.id}?download=true&amp;access_token=#{attachment.access_token}" target="_blank" data-no-post-process="">
                                                    <div class='oe_attachment_embedded o_image o_image_small me-2 me-lg-3' t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype" t-attf-data-src="/web/image/#{attachment.id}/50x40?access_token=#{attachment.access_token}"/>
                                                    <div class='oe_attachment_name text-truncate'><t t-esc='attachment.name'/></div>
                                                </a>
                                            </ul>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="mt32" id="task_chat" data-anchor="true">
                        <h4><strong>Message and communication history</strong></h4>
                        <t t-call="portal.message_thread">
                            <t t-set="token" t-value="task.access_token"/>
                        </t>
                    </div>
                </div>
            </div>
        </xpath>
    </template>

    <template id="portal_my_task_planned_hours_template">
        <strong>Allocated Hours:</strong> <span t-esc="task.planned_hours" t-options='{"widget": "float_time"}'/>
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
            <tree editable="bottom" sample="1">
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
                            <strong><field name="name"/></strong>
                            <br/>
                            <span class="text-muted"><field name="mail_template_id"/></span>
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
              Define the steps your projects move through from creation to completion.
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
            <!-- To add the class on div#wrapwrap to remove the overflow -->
            <t t-set="pageName" t-value="'o_project_sharing_container'"/>
            <t t-set="no_footer" t-value="true"/>
            <t t-call="project.project_sharing"/>
        </t>
    </template>

    <template id="project_sharing" name="Project Sharing View">
        <!--    We need to forward the request lang to ensure that the lang set on the portal match the lang delivered -->
        <iframe class="flex-grow-1" frameborder="0" t-attf-src="/{{ request.context['lang'] }}/my/projects/{{ str(project_id) }}/project_sharing{{ '?task_id=' + task_id if task_id else '' }}"/>
    </template>

    <template id="project_sharing_embed" name="Project Sharing View Embed">
        <t t-call="web.layout">
            <t t-set="head_project_sharing">
                <script type="text/javascript">
                    odoo.__session_info__ = <t t-out="json.dumps(session_info)"/>;
                    // Prevent the menu_service to load anything. In an ideal world, Project Sharing assets would only contain
                    // what is genuinely necessary, and not the whole backend.
                    odoo.loadMenusPromise = Promise.resolve();
                </script>
                <base target="_parent"/>
                <t t-call-assets="web.assets_common" t-js="false"/>
                <t t-call-assets="project.webclient" t-js="false"/>
                <t t-call-assets="web.assets_common" t-css="false"/>
                <t t-call-assets="project.webclient" t-css="false"/>
                <t t-call="web.conditional_assets_tests"/>
            </t>
            <t t-set="head" t-value="head_project_sharing + (head or '')"/>
            <t t-set="body_classname" t-value="'o_web_client o_project_sharing'"/>
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
                    <field name="name" string="Task Title" placeholder="e.g. Send Invitations"/>
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
                <field name="portal_user_names"/>
                <field name="partner_id"/>
                <field name="sequence"/>
                <field name="is_closed" force_save="0"/>
                <field name="partner_is_company"/>
                <field name="displayed_image_id"/>
                <field name="active"/>
                <field name="allow_subtasks"/>
                <field name="child_text"/>
                <field name="legend_blocked" invisible="1" force_save="0"/>
                <field name="legend_normal" invisible="1" force_save="0"/>
                <field name="legend_done" invisible="1" force_save="0"/>
                <field name="allow_milestones" />
                <field name="has_late_and_unreached_milestone"/>
                <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger", "normal": "200"}'/>
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
                                    <span t-if="record.allow_milestones.raw_value and record.milestone_id.raw_value" t-attf-class="{{record.has_late_and_unreached_milestone.raw_value ? 'text-danger' : ''}}">
                                        <br/>
                                        <field name="milestone_id" />
                                    </span>
                                    <br />
                                    <t t-if="record.partner_id.value">
                                        <span t-if="!record.partner_is_company.raw_value" t-attf-title="#{record.commercial_partner_id.value}">
                                            <field name="commercial_partner_id" class="text-truncate d-block"/>
                                        </span>
                                        <span t-else="" t-attf-title="#{record.partner_id.value}">
                                            <field name="partner_id" class="text-truncate d-block"/>
                                        </span>
                                    </t>
                                    <t t-else="record.email_from.raw_value"><span><field name="email_from"/></span></t>
                                </div>
                                <div class="o_dropdown_kanban dropdown" t-if="!selection_mode">
                                    <a role="button" class="dropdown-toggle o-no-caret btn" data-bs-toggle="dropdown" data-bs-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                        <span class="fa fa-ellipsis-v"/>
                                    </a>
                                    <div class="dropdown-menu" role="menu">
                                        <a t-if="widget.editable" role="menuitem" type="edit" class="dropdown-item">Edit</a>
                                        <div invisible="1" role="separator" class="dropdown-divider"></div>
                                        <ul invisible="1" class="oe_kanban_colorpicker" data-field="color"/>
                                    </div>
                                </div>
                            </div>
                            <div class="o_kanban_record_body">
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" context="{'project_id': project_id}"/>
                                <div t-if="record.date_deadline.raw_value" name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}">
                                    <field name="date_deadline" widget="remaining_days"/>
                                </div>
                                <div t-if="record.displayed_image_id.value" groups="base.group_user">
                                    <field name="displayed_image_id" widget="attachment_image"/>
                                </div>
                            </div>
                            <div class="o_kanban_record_bottom" t-if="!selection_mode">
                                <div class="oe_kanban_bottom_left">
                                    <field name="priority" widget="priority"/>
                                </div>
                                <div class="oe_kanban_bottom_right" t-if="!selection_mode">
                                    <span t-if="record.portal_user_names.raw_value.length > 0" class="pe-2" t-att-title="record.portal_user_names.raw_value">
                                        <t t-set="user_count" t-value="record.portal_user_names.raw_value.split(',').length"/>
                                        <t t-out="user_count"/>
                                        <t t-if="user_count > 1"> assignees</t>
                                        <t t-else=""> assignee</t>
                                    </span>
                                    <field name="kanban_state" widget="state_selection"/>
                                </div>
                            </div>
                        </div>
                        <div class="clearfix"></div>
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
                <field name="sequence" invisible="1" readonly="1"/>
                <field name="allow_milestones" invisible="1"/>
                <field name="priority" widget="priority" optional="show" nolabel="1"/>
                <field name="child_text" invisible="1"/>
                <field name="allow_subtasks" invisible="1" />
                <field name="name" widget="name_with_subtask_count"/>
                <field name="company_id" invisible="1"/>
                <field name="milestone_id" attrs="{'column_invisible': [('allow_milestones', '=', False)]}"/>
                <field name="partner_id" optional="hide"/>
                <field name="portal_user_names" string="Assignees" optional="show"/>
                <field name="date_deadline" optional="hide" widget="remaining_days" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                <field name="kanban_state" widget="state_selection" options="{'hide_label': True}" nolabel="1" optional="show"/>
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
                        <field name="display_parent_task_button" invisible="1"/>
                        <button name="action_project_sharing_view_parent_task" type="object" class="oe_stat_button" icon="fa-tasks" string="Parent Task" attrs="{'invisible': [('display_parent_task_button', '=', False)]}"/>
                        <button name="action_project_sharing_open_subtasks" type="object" class="oe_stat_button" icon="fa-tasks"
                            attrs="{'invisible' : ['|', '|', ('allow_subtasks', '=', False), ('id', '=', False), ('subtask_count', '=', 0)]}" context="{'default_user_ids': [(6, 0, [uid])]}">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="subtask_count" widget="statinfo" nolabel="1"/>
                                </span>
                                <span class="o_stat_text">Sub-tasks</span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title pe-0">
                        <h1 class="d-flex flex-row justify-content-between">
                            <field name="priority" widget="priority" class="me-3"/>
                            <field name="name" class="o_task_name text-truncate" placeholder="Task Title..."/>
                            <field name="kanban_state" widget="state_selection" class="ms-auto"/>
                            <field name="legend_blocked" invisible="1"/>
                            <field name="legend_normal" invisible="1"/>
                            <field name="legend_done" invisible="1"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="project_id" invisible="1"/>
                            <field name="display_project_id" string="Project" invisible="1"/>
                            <field name="allow_milestones" invisible="1"/>
                            <field name="milestone_id"
                                placeholder="e.g. Product Launch"
                                context="{'default_project_id': project_id if not parent_id or not display_project_id else display_project_id}"
                                attrs="{'invisible': [('allow_milestones', '=', False)]}"
                                options="{'no_open': True, 'no_create': True, 'no_edit': True}"
                            />
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
                            <field name="tag_ids" context="{'project_id': project_id}" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True, 'no_edit_color': True}"/>
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
                                    <field name="priority" widget="priority" optional="show" nolabel="1"/>
                                    <field name="name"/>
                                    <field name="display_project_id" string="Project" optional="hide" invisible="1"/>
                                    <field name="allow_milestones" invisible="1"/>
                                    <field name="milestone_id"
                                        optional="hide"
                                        context="{'default_project_id': display_project_id or project_id}"
                                        attrs="{'invisible': [('allow_milestones', '=', False)], 'column_invisible': [('parent.allow_milestones', '=', False)]}"
                                        options="{'no_open': True, 'no_create': True, 'no_edit': True}"
                                    />
                                    <field name="company_id" invisible="1"/>
                                    <field name="partner_id" options="{'no_open': True, 'no_create': True, 'no_edit': True}" optional="hide"/>
                                    <field name="user_ids" invisible="1" />
                                    <field name="portal_user_names" string="Assignees" optional="show"/>
                                    <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}" optional="show"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="kanban_state" widget="state_selection" options="{'hide_label': True}" nolabel="1" optional="show"/>
                                    <field name="stage_id" domain="[('user_id', '=', False), ('project_ids', 'in', [project_id])]"/>
                                    <button name="action_open_task" type="object" title="View Task" string="View Task" class="btn btn-link float-end"
                                            context="{'form_view_ref': 'project.project_sharing_project_task_view_form'}"
                                            attrs="{'invisible': &quot;[('project_id', '!=', False), ('project_id', '!=', active_id)]&quot;}"/>
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
                <field name="portal_user_names" string="Assignees"/>
                <field string="Project" name="display_project_id"/>
                <field name="milestone_id" groups="project.group_project_milestone"/>
                <field name="stage_id"/>
                <field name="partner_id" operator="child_of"/>
                <filter string="Unassigned" name="unassigned" domain="[('user_ids', '=', False)]"/>
                <separator/>
                <filter string="High Priority" name="high_priority" domain="[('priority', '=', 1)]"/>
                <filter string="Low Priority" name="low_priority" domain="[('priority', '=', 0)]"/>
                <separator/>
                <filter string="Late Tasks" name="late" domain="[('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d')), ('is_closed', '=', False)]"/>
                <filter string="Tasks Due Today" name="tasks_due_today" domain="[('date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter string="Late Milestones" name="late_milestone" domain="[('is_closed', '=', False), ('has_late_and_unreached_milestone', '=', True)]" groups="project.group_project_milestone"/>
                <separator/>
                <filter string="Open Tasks" name="open_tasks" domain="[('is_closed', '=', False)]"/>
                <filter string="Closed Tasks" name="closed_tasks" domain="[('is_closed', '=', True)]"/>
                <filter string="Closed Last 7 Days" name="closed_last_7_days" domain="[('is_closed', '=', True), ('date_last_stage_update', '&gt;', datetime.datetime.now() - relativedelta(days=7))]"/>
                <filter string="Closed Last 30 Days" name="closed_last_30_days" domain="[('is_closed', '=', True), ('date_last_stage_update', '&gt;', datetime.datetime.now() - relativedelta(days=30))]"/>
                <separator/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}"/>
                    <filter string="Milestone" name="milestone" context="{'group_by': 'milestone_id'}" groups="project.group_project_milestone"/>
                    <filter string="Customer" name="customer" context="{'group_by': 'partner_id'}"/>
                    <filter string="Kanban State" name="kanban_state" context="{'group_by': 'kanban_state'}"/>
                    <filter string="Deadline" name="date_deadline" context="{'group_by': 'date_deadline'}"/>
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
            'active_id_chatter': active_id,
        }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No tasks found. Let's create one!
            </p>
            <p>
                Keep track of the progress of your tasks from creation to completion.<br/>
                Collaborate efficiently by chatting in real-time or via email.
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

    <record id="project_sharing_project_task_action_sub_task" model="ir.actions.act_window">
        <field name="name">Sub-tasks</field>
        <field name="res_model">project.task</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="search_view_id" ref="project.project_sharing_project_task_view_search"/>
        <field name="domain">[('id', 'child_of', active_id), ('id', '!=', active_id)]</field>
        <field name="context">{'default_parent_id': active_id}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No tasks found. Let's create one!
            </p><p>
                To get things done, use activities and status on tasks.<br/>
                Chat in real time or by email to collaborate efficiently.
            </p>
        </field>
    </record>

    <record id="project_sharing_subtasks_tree_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action_sub_task"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_tree"/>
    </record>

    <record id="project_sharing_subtasks_kanban_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action_sub_task"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_kanban"/>
    </record>

    <record id="project_sharing_subtasks_form_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action_sub_task"/>
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
(due <t t-esc="milestone['deadline']" t-options='{"widget": "date"}'/><t t-if="not milestone['is_reached'] or not milestone['reached_date']">
<t t-if="milestone['can_be_marked_as_done']"> - <font t-att-style="'color: rgb(0, ' + str(color_level) + ', 0)'">ready to be marked as reached</font></t>)</t><t t-else=""> - reached on<t t-if="milestone['reached_date'] &gt; milestone['deadline']">
<font t-att-style="'color: rgb(' + str(color_level) + ', 0, 0)'"><b><t t-esc="milestone['reached_date']" t-options='{"widget": "date"}'/></b></font>)</t><t t-else="">
<font t-att-style="'color: rgb(0, ' + str(color_level) + ', 0)'"><b><t t-esc="milestone['reached_date']" t-options='{"widget": "date"}'/></b></font>)</t></t>
</t>
<t t-elif="milestone['can_be_marked_as_done']">
(<font t-att-style="'color: rgb(0, ' + str(color_level) + ', 0)'">ready to be marked as reached</font>)
</t>
    </template>

    <template id="project_update_default_description" name="Project Update Description">
<!--As this template is rendered in an html field, the spaces may be interpreted as nbsp while editing. -->
<div name="summary">
<br/><h1 style="font-weight: bolder;">Summary</h1>
<br/><p>How’s this project going?</p><br/><br/>
</div>

<div name="activities" t-if="show_activities">
<h1 style="font-weight: bolder;">Activities</h1>
</div>

<div name="milestone" t-if="milestones['show_section']">
<br/>
<h3 style="font-weight: bolder"><u>Milestones</u></h3>

<ul class="o_checklist" t-if="milestones['list']">
<t t-foreach="milestones['list']" t-as="milestone">
<li t-attf-class="{{milestone['is_reached'] and 'o_checked' or ''}}">
<t t-esc="milestone['name']"/>
<span t-if="milestone['is_deadline_future'] and not milestone['is_reached'] and not milestone['can_be_marked_as_done']"><font style="color: rgb(190, 190, 190);"><t t-set="color_level" t-value="64"/><t t-call="project.milestone_deadline"/></font></span>
<span t-elif="milestone['is_deadline_exceeded']"><font style="color: rgb(255, 0, 0);"><t t-call="project.milestone_deadline"/></font></span>
<span t-else=""><t t-set="color_level" t-value="128"/><t t-call="project.milestone_deadline"/></span>
</li>
</t>
</ul>

<t t-if="milestones['updated']">
<t t-if="milestones['last_update_date']">Since <t t-esc="milestones['last_update_date']" t-options='{"widget": "date"}'/> (last project update), </t>
<t t-if="len(milestones['updated']) > 1">the deadline for the following milestones has been updated:</t>
<t t-else="">the deadline for the following milestone has been updated:</t>
<ul>
<t t-foreach="milestones['updated']" t-as="milestone">
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
<t t-foreach="milestones['created']" t-as="milestone">
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
                <field name="user_id"/>
                <field name="description"/>
                <field name="status"/>
                <filter string="My Updates" name="my_updates" domain="[('user_id', '=', uid)]"/>
                <filter string="Followed Updates" name="followed_updates" domain="[('message_is_follower', '=', True)]"/>
                <separator/>
                <filter string="On Track" name="on_track" domain="[('status', '=', 'on_track')]"/>
                <filter string="At Risk" name="at_risk" domain="[('status', '=', 'at_risk')]"/>
                <filter string="Off Track" name="off_track" domain="[('status', '=', 'off_track')]"/>
                <filter string="On Hold" name="on_hold" domain="[('status', '=', 'on_hold')]"/>
                <separator/>
                <filter name="date" string="Date" date="date"/>
            </search>
        </field>
    </record>

    <record id="project_update_view_form" model="ir.ui.view">
        <field name="name">project.update.view.form</field>
        <field name="model">project.update</field>
        <field name="arch" type="xml">
            <form string="Project Update" class="o_form_project_update" js_class="form_description_expander">
                <sheet>
                    <div class="oe_title">
                        <h1>
                            <field name="name" class="o_text_overflow" placeholder="e.g. Monthly review"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="project_id" invisible="1"/>
                            <field name="color" invisible="1"/>
                            <field name="status" widget="status_with_color" options="{'color_field': 'color'}"/>
                            <field name="progress" widget="progressbar" options="{'editable': true}"/>
                        </group>
                        <group>
                            <field name="user_id" widget="many2one_avatar_user" readonly="1"/>
                            <field name="date"/>
                        </group>
                    </group>
                    <separator/>
                    <notebook>
                        <page string="Description" name="description">
                            <field name="description" nolabel="1" class="o_project_update_description" options="{'resizable': false, 'collaborative': true}"/>
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
                                <div class="col-sm-4 col-6 o_pupdate_name">
                                    <b><field name="name_cropped"/></b>
                                    <div>
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                        <t t-esc="record.user_id.value"/>
                                    </div>
                                </div>
                                <div class="col-sm-2 text-sm-start col-6 align-end">
                                    <field name="color" invisible="1"/>
                                    <b><field name="status" widget="status_with_color" options="{'color_field': 'color'}"/></b>
                                </div>
                                <div class="col-sm-2 col-6 pb-0">
                                    <b><field name="progress_percentage" widget="percentage"/></b>
                                    <div>Progress</div>
                                </div>
                                <div class="col-sm-2 col-6 pb-0">
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
                <field name="user_id" widget="many2one_avatar_user" class="fw-bold" optional="show"/>
                <field name="date" optional="show"/>
                <field name="progress_percentage" string="Progress" widget="percentage" optional="show"/>
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
            web_icon="project,static/description/icon.svg"
            sequence="70"/>

        <menuitem id="menu_project_config" name="Configuration" parent="menu_main_pm"
            sequence="100" groups="project.group_project_manager"/>

        <record id="view_task_search_form" model="ir.ui.view">
            <field name="name">project.task.search.form</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
               <search string="Tasks">
                    <field name="name" string="Task" filter_domain="['|', ('name', 'ilike', self), ('id', 'ilike', self)]"/>
                    <field name="tag_ids"/>
                    <field name="user_ids" filter_domain="[('user_ids.name', 'ilike', self), ('user_ids.active', 'in', [True, False])]"/>
                    <field name="milestone_id" groups="project.group_project_milestone"/>
                    <field name="ancestor_id" groups="project.group_subtask_project"/>
                    <field name="stage_id"/>
                    <field name="partner_id" operator="child_of"/>
                    <field name="description"/>
                    <field name="rating_last_text"/>
                    <filter string="My Tasks" name="my_tasks" domain="[('user_ids', 'in', uid)]"/>
                    <filter string="Followed Tasks" name="followed_by_me" domain="[('message_is_follower', '=', True)]"/>
                    <filter string="Unassigned" name="unassigned" domain="[('user_ids', '=', False)]"/>
                    <separator/>
                    <filter string="High Priority" name="high_priority" domain="[('priority', '=', 1)]"/>
                    <filter string="Low Priority" name="low_priority" domain="[('priority', '=', 0)]"/>
                    <separator/>
                    <filter string="Blocked" name="blocked" domain="[('is_blocked', '=', True)]" groups="project.group_project_task_dependencies"/>
                    <filter string="Not Blocked" name="not_blocked" domain="[('is_blocked', '=', False), ('is_private', '=', False)]" groups="project.group_project_task_dependencies"/>
                    <separator groups="project.group_project_task_dependencies"/>
                    <filter string="Blocking" name="blocking" domain="[('is_closed', '=', False), ('dependent_ids', '!=', False)]" groups="project.group_project_task_dependencies"/>
                    <filter string="Not Blocking" name="not_blocking" domain="['|', ('is_closed', '=', True), ('dependent_ids', '=', False), ('is_private', '=', False)]" groups="project.group_project_task_dependencies"/>
                    <separator groups="project.group_project_task_dependencies"/>
                    <filter string="Late Milestones" name="late_milestone" domain="[('is_closed', '=', False), ('has_late_and_unreached_milestone', '=', True)]" groups="project.group_project_milestone"/>
                    <filter string="Late Tasks" name="late" domain="[('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d')), ('is_closed', '=', False)]"/>
                    <filter string="Tasks Due Today" name="tasks_due_today" domain="[('date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter string="Stalling for 30 Days+" name="stall_last_30_days" domain="[('is_closed', '=', False), ('date_last_stage_update', '&lt;=', datetime.datetime.now() - relativedelta(days=30))]"/>
                    <separator/>
                    <filter string="Open Tasks" name="open_tasks" domain="[('is_closed', '=', False)]"/>
                    <filter string="Closed Tasks" name="closed_tasks" domain="[('is_closed', '=', True)]"/>
                    <filter string="Closed Last 7 Days" name="closed_last_7_days" domain="[('is_closed', '=', True), ('date_last_stage_update', '&gt;', datetime.datetime.now() - relativedelta(days=7))]"/>
                    <filter string="Closed Last 30 Days" name="closed_last_30_days" domain="[('is_closed', '=', True), ('date_last_stage_update', '&gt;', datetime.datetime.now() - relativedelta(days=30))]"/>
                    <separator/>
                    <filter name="rating_satisfied" string="Satisfied" domain="[('rating_avg', '&gt;=', 3.66)]" groups="project.group_project_rating"/>
                    <filter name="rating_okay" string="Okay" domain="[('rating_avg', '&lt;', 3.66), ('rating_avg', '&gt;=', 2.33)]" groups="project.group_project_rating"/>
                    <filter name="dissatisfied" string="Dissatisfied" domain="[('rating_avg', '&lt;', 2.33), ('rating_last_value', '!=', 0)]" groups="project.group_project_rating"/>
                    <filter name="no_rating" string="No Rating" domain="[('rating_last_value', '=', 0)]" groups="project.group_project_rating"/>
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
                        <filter string="Ancestor Task" name="groupby_ancestor_task" context="{'group_by': 'ancestor_id'}" groups="project.group_subtask_project"/>
                        <filter string="Milestone" name="milestone" context="{'group_by': 'milestone_id'}" groups="project.group_project_milestone"/>
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
                <xpath expr="//field[@name='user_ids']" position='after'>
                    <field string="Project" name="display_project_id"/>
                </xpath>
                <xpath expr="//filter[@name='my_tasks']" position='after'>
                    <filter string="My Private Tasks" name="my_private_task" domain="[('project_id', '=', False), ('user_ids', 'in', uid)]"/>
                </xpath>
                <xpath expr="//filter[@name='unassigned']" position="after">
                    <separator/>
                    <filter string="My Projects" name="my_projects" domain="[('project_id.user_id', '=', uid)]"/>
                    <filter string="My Favorite Projects" name="my_favorite_projects" domain="[('project_id.favorite_user_ids', 'in', [uid])]"/>
                </xpath>
                <xpath expr="//filter[@name='user']" position='after'>
                    <filter string="Project" name="project" context="{'group_by': 'project_id'}"/>
                </xpath>
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
                    <field name="sequence" invisible="1"/>
                    <field name="planned_hours" widget="float_time"/>
                    <field name="working_hours_close" widget="float_time"/>
                    <field name="working_hours_open" widget="float_time"/>
                </pivot>
            </field>
        </record>

        <record id="view_project_task_pivot_inherit" model="ir.ui.view">
            <field name="name">project.task.pivot.inherit</field>
            <field name="model">project.task</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="project.view_project_task_pivot"/>
            <field name="arch" type="xml">
                <xpath expr="/pivot" position="inside">
                    <field name="user_ids" type="row"/>
                </xpath>
            </field>
        </record>

        <record id="act_project_project_2_project_task_all" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form,calendar,pivot,graph,activity</field>
            <field name="domain">[('display_project_id', '=', active_id)]</field>
            <field name="context">{
                'default_project_id': active_id,
                'show_project_update': True,
            }</field>
            <field name="search_view_id" ref="view_task_search_form"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p>
                <p>
                    Keep track of the progress of your tasks from creation to completion.<br/>
                    Collaborate efficiently by chatting in real-time or via email.
                </p>
            </field>
        </record>

    <!-- Set pivot view and arrange in order -->
    <record id="project_task_kanban_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="project.act_project_project_2_project_task_all"/>
    </record>

    <record id="project_task_tree_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="20"/>
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="project.act_project_project_2_project_task_all"/>
    </record>

    <record id="project_task_form_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="30"/>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="project.act_project_project_2_project_task_all"/>
    </record>

    <record id="project_all_task_calendar_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">calendar</field>
        <field name="act_window_id" ref="project.act_project_project_2_project_task_all"/>
    </record>

    <record id="project_all_task_pivot_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="70"/>
        <field name="view_mode">pivot</field>
        <field name="view_id" ref="view_project_task_pivot_inherit"/>
        <field name="act_window_id" ref="act_project_project_2_project_task_all"/>
    </record>

    <record id="project_all_task_graph_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="80"/>
        <field name="view_mode">graph</field>
        <field name="act_window_id" ref="project.act_project_project_2_project_task_all"/>
    </record>

    <record id="project_all_task_activity_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="90"/>
        <field name="view_mode">activity</field>
        <field name="act_window_id" ref="project.act_project_project_2_project_task_all"/>
    </record>

        <record id="project_task_action_sub_task" model="ir.actions.act_window">
            <field name="name">Sub-tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,kanban,form,calendar,pivot,graph,activity</field>
            <field name="search_view_id" ref="project.view_task_search_form"/>
            <field name="domain">[('id', 'child_of', active_id), ('id', '!=', active_id)]</field>
            <field name="context">{'show_project_update': False, 'default_parent_id': active_id}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p>
                <p>
                    Keep track of the progress of your tasks from creation to completion.<br/>
                    Collaborate efficiently by chatting in real-time or via email.
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
                   <field name="project_ids" string="Project"/>
                   <field name="mail_template_id"/>
                   <field name="rating_template_id"/>
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
                                <div class="alert alert-warning" role="alert" colspan='2' attrs="{'invisible': ['|', ('rating_template_id','=', False), ('disabled_rating_warning', '=', False)]}" groups="project.group_project_rating">
                                    <i class="fa fa-warning" title="Customer disabled on projects"/><b> Customer Ratings</b> are disabled on the following project(s) : <br/>
                                    <field name="disabled_rating_warning" class="mb-0" />
                                </div>
                                <field name="auto_validation_kanban_state" attrs="{'invisible': [('rating_template_id','=', False)]}" groups="project.group_project_rating"/>
                                <field name="sequence" groups="base.group_no_one"/>
                            </group>
                            <group>
                                <field name="fold"/>
                                <field name="project_ids" widget="many2many_tags" options="{'color_field': 'color'}" required="1"/>
                            </group>
                        </group>
                        <group string="Stage Description and Tooltips">
                            <group>
                                <p class="text-muted" colspan="2">
                                    At each stage, employees can block tasks or mark them as ready for the next step.
                                    You can customize here the labels for each state.
                                </p>
                                <div class="row g-0 ms-1" colspan="2">
                                    <label for="legend_normal" string=" " class="o_status mt4"
                                        title="Task in progress. Click to block or set as done."
                                        aria-label="Task in progress. Click to block or set as done." role="img"/>
                                    <div class="col-11 ps-2">
                                        <field name="legend_normal"/>
                                    </div>
                                </div>
                                <div class="row g-0 ms-1" colspan="2">
                                    <label for="legend_blocked" string=" " class="o_status o_status_red mt4"
                                        title="Task is blocked. Click to unblock or set as done."
                                        aria-label="Task is blocked. Click to unblock or set as done." role="img"/>
                                    <div class="col-11 ps-2">
                                        <field name="legend_blocked"/>
                                    </div>
                                </div>
                                <div class="row g-0 ms-1" colspan="2">
                                    <label for="legend_done" string=" " class="o_status o_status_green mt4"
                                        title="This step is done. Click to block or set in progress."
                                        aria-label="This step is done. Click to block or set in progress." role="img"/>
                                    <div class="col-11 ps-2">
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
                    <field name="fold" optional="show"/>
                </tree>
            </field>
        </record>

        <record id="task_type_tree_inherited" model="ir.ui.view">
            <field name="name">project.task.type.tree.inherited</field>
            <field name="model">project.task.type</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="task_type_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='name']" position="after">
                    <field name="mail_template_id" optional="hide"/>
                    <field name="rating_template_id" optional="hide" groups="project.group_project_rating"/>
                    <field name="project_ids" optional="show" widget="many2many_tags" options="{'color_field': 'color'}"/>
                </xpath>
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
                    <field name="sequence" widget="handle"/>
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
            <field name="view_id" ref="task_type_tree_inherited"/>
            <field name="domain">[('user_id', '=', False)]</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No stages found. Let's create one!
              </p><p>
                Define the steps your tasks move through from creation to completion.
              </p>
            </field>
        </record>

        <record id="open_task_type_form_domain" model="ir.actions.act_window">
            <field name="name">Task Stages</field>
            <field name="res_model">project.task.type</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="domain">[('project_ids','=', project_id)]</field>
            <field name="view_id" ref="task_type_tree_inherited"/>
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

        <record id="action_send_mail_project_task" model="ir.actions.act_window">
            <field name="name">Send Email</field>
            <field name="res_model">mail.compose.message</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="context" eval="{
                'default_composition_mode': 'mass_mail',
                'default_use_template': False,
            }"/>
            <field name="binding_model_id" ref="project.model_project_task"/>
            <field name="binding_view_types">list</field>
        </record>

        <record id="action_send_mail_project_project" model="ir.actions.act_window">
            <field name="name">Send Email</field>
            <field name="res_model">mail.compose.message</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="context" eval="{
                'default_composition_mode': 'mass_mail',
                'default_use_template': False,
            }"/>
            <field name="binding_model_id" ref="project.model_project_project"/>
            <field name="binding_view_types">list</field>
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
                <form string="Project" class="o_form_project_project" js_class="project_form">
                    <field name="company_id" invisible="1"/>
                    <field name="analytic_account_id" invisible="1"/>
                    <header>
                        <button name="%(project.project_share_wizard_action)d" string="Share Readonly" type="action" class="oe_highlight" groups="project.group_project_manager"
                        attrs="{'invisible': [('privacy_visibility', '!=', 'portal')]}" context="{'default_access_mode': 'read'}" data-hotkey="r"/>
                        <button name="%(project.project_share_wizard_action)d" string="Share Editable" type="action" class="oe_highlight" groups="project.group_project_manager"
                        attrs="{'invisible': [('privacy_visibility', '!=', 'portal')]}" context="{'default_access_mode': 'edit'}" data-hotkey="e"/>
                        <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" groups="project.group_project_stages"/>
                    </header>
                <sheet string="Project">
                    <div class="oe_button_box" name="button_box" groups="base.group_user">
                        <button class="oe_stat_button ps-2" name="project_update_all_action" type="object" groups="project.group_project_user">
                            <div class="w-100">
                                <field name="last_update_color" invisible="1"/>
                                <field name="last_update_status" readonly="1" widget="status_with_color" options="{'color_field': 'last_update_color'}"/>
                            </div>
                        </button>
                        <!-- To Do: remove me in master -->
                        <button class="oe_stat_button o_project_not_clickable ps-2" disabled="disabled" groups="!project.group_project_manager" invisible="1">
                            <div class="w-100">
                                <field name="last_update_color" invisible="1"/>
                                <field name="last_update_status" readonly="1" widget="status_with_color" options="{'color_field': 'last_update_color'}"/>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="%(project.project_collaborator_action)d" type="action" icon="fa-users" groups="project.group_project_manager" attrs="{'invisible':[('privacy_visibility', '!=', 'portal')]}">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="collaborator_count" nolabel="1"/>
                                </span>
                                <span class="o_stat_text">
                                    Collaborators
                                </span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <h1 class="d-flex flex-row">
                            <field name="is_favorite" nolabel="1" widget="boolean_favorite" class="me-2"/>
                            <field name="name" class="o_text_overflow" placeholder="e.g. Office Party"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="label_tasks" string="Name of the tasks"/>
                            <field name="partner_id" widget="res_partner_many2one"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="user_id" string="Project Manager" widget="many2one_avatar_user" attrs="{'readonly':[('active','=',False)]}" domain="[('share', '=', False)]"/>
                            <label for="date_start" string="Planned Date"/>
                            <div name="dates" class="o_row">
                                <field name="date_start" widget="daterange" options='{"related_end_date": "date"}'/>
                                <i class="fa fa-long-arrow-right mx-2 oe_edit_only" aria-label="Arrow icon" title="Arrow"/>
                                <i class="fa fa-long-arrow-right mx-2 oe_read_only" aria-label="Arrow icon" title="Arrow" attrs="{'invisible': [('date_start', '=', False), ('date', '=', False)]}"/>
                                <field name="date" widget="daterange" options='{"related_start_date": "date_start"}'/>
                            </div>
                        </group>
                    </group>
                    <notebook>
                        <page name="description" string="Description">
                            <field name="description" options="{'resizable': false}" placeholder="Project description..."/>
                        </page>
                        <page name="settings" string="Settings">
                            <group>
                                <group>
                                    <field name="analytic_account_id" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]" context="{'default_partner_id': partner_id}" groups="analytic.group_analytic_accounting"/>
                                    <field name="privacy_visibility" widget="radio"/>
                                    <span colspan="2" class="text-muted" attrs="{'invisible':[('access_instruction_message', '=', '')]}">
                                        <i class="fa fa-lightbulb-o"/>&amp;nbsp;<field class="d-inline" name="access_instruction_message" nolabel="1"/>
                                    </span>
                                    <span colspan="2" class="text-muted" attrs="{'invisible':[('privacy_visibility_warning', '=', '')]}">
                                        <i class="fa fa-warning"/>&amp;nbsp;<field class="d-inline" name="privacy_visibility_warning" nolabel="1"/>
                                    </span>
                                </group>
                                <group>
                                    <div name="alias_def" colspan="2" class="pb-2" attrs="{'invisible': [('alias_domain', '=', False)]}">
                                        <!-- Always display the whole alias in edit mode. It depends in read only -->
                                        <field name="alias_enabled" invisible="1"/>
                                        <label for="alias_name" class="fw-bold o_form_label" string="Create tasks by sending an email to"/>
                                        <field name="alias_value" class="oe_read_only d-inline" readonly="1" widget="email" attrs="{'invisible':  [('alias_name', '=', False)]}" />
                                        <span class="oe_edit_only" dir="ltr">
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
                            <group>
                                <group name="group_tasks_managment" string="Tasks Management" col="1" class="row mt16 o_settings_container" groups="project.group_subtask_project,project.group_project_task_dependencies,project.group_project_milestone,project.group_project_recurring_tasks">
                                    <div>
                                        <div class="o_setting_box" id="subtask_settings" groups="project.group_subtask_project">
                                            <div class="o_setting_left_pane">
                                                <field name="allow_subtasks"/>
                                            </div>
                                            <div class="o_setting_right_pane">
                                                <label for="allow_subtasks"/>
                                                <div class="text-muted">
                                                    Split your tasks to organize your work into sub-milestones
                                                </div>
                                            </div>
                                        </div>
                                        <div class="o_setting_box mt-4" id="recurring_tasks_setting" groups="project.group_project_recurring_tasks">
                                            <div class="o_setting_left_pane">
                                                <field name="allow_recurring_tasks"/>
                                            </div>
                                            <div class="o_setting_right_pane">
                                                <label for="allow_recurring_tasks"/>
                                                <div class="text-muted">
                                                    Auto-generate tasks for regular activities
                                                </div>
                                            </div>
                                        </div>
                                        <div class="o_setting_box mt-4" id="task_dependencies_setting" groups="project.group_project_task_dependencies">
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
                                        <div class="o_setting_box mt-4" id="project_milestone_setting" groups="project.group_project_milestone">
                                            <div class="o_setting_left_pane">
                                                <field name="allow_milestones"/>
                                            </div>
                                            <div class="o_setting_right_pane">
                                                <label for="allow_milestones"/>
                                                <div class="text-muted">
                                                    Track major progress points that must be reached to achieve success
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </group>
                                <group name="group_time_managment" string="Time Management" invisible="1" col="1" class="row mt16 o_settings_container"/>
                                <group name="group_documents_analytics" string="Analytics" col="1" class="row mt16 o_settings_container" attrs="{'invisible': [('allow_rating', '=', False)]}">
                                    <div>
                                        <field name="allow_rating" invisible="1"/>
                                        <div class="o_setting_box" name="analytic_div" groups="project.group_project_rating">
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
                                    </div>
                                </group>
                            </group>
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
                    <field name="tag_ids"/>
                    <field name="user_id" string="Project Manager"/>
                    <field name="partner_id" string="Customer" filter_domain="[('partner_id', 'child_of', self)]"/>
                    <field name="analytic_account_id"/>
                    <field name="stage_id" groups="project.group_project_stages"/>
                    <filter string="My Projects" name="own_projects" domain="[('user_id', '=', uid)]"/>
                    <filter string="My Favorites" name="my_projects" domain="[('favorite_user_ids', 'in', uid)]"/>
                    <filter string="Followed" name="followed_by_me" domain="[('message_is_follower', '=', True)]"/>
                    <filter string="Unassigned" name="unassigned_projects" domain="[('user_id', '=', False)]"/>
                    <separator/>
                    <filter string="Late Milestones" name="late_milestones" domain="[('is_milestone_exceeded', '=', True)]" groups="project.group_project_milestone"/>
                    <separator/>
                    <filter string="Open" name="open_project" domain="[('stage_id.fold', '=', False)]" groups="project.group_project_stages"/>
                    <filter string="Closed" name="closed_project" domain="[('stage_id.fold', '=', True)]" groups="project.group_project_stages"/>
                    <separator/>
                    <filter string="Start Date" name="start_date" date="date_start"/>
                    <filter string="End Date" name="end_date" date="date"/>
                    <separator/>
                    <filter name="rating_satisfied" string="Satisfied" domain="[('rating_active', '=', True), ('rating_avg', '&gt;=', 3.66)]" groups="project.group_project_rating"/>
                    <filter name="rating_okay" string="Okay" domain="[('rating_active', '=', True), ('rating_avg', '&lt;', 3.66), ('rating_avg', '&gt;=', 2.33)]" groups="project.group_project_rating"/>
                    <filter name="dissatisfied" string="Dissatisfied" domain="[('rating_active', '=', True), ('rating_avg', '&lt;', 2.33), ('rating_avg', '&gt;', 0)]" groups="project.group_project_rating"/>
                    <filter name="no_rating" string="No Rating" domain="['|', ('rating_active', '=', False), ('rating_avg', '=', 0)]" groups="project.group_project_rating"/>
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
                <tree decoration-muted="active == False" string="Projects" multi_edit="1" sample="1" default_order="sequence, name, id">
                    <field name="sequence" optional="show" widget="handle"/>
                    <field name="name" invisible="1"/>
                    <field name="message_needaction" invisible="1"/>
                    <field name="name" invisible="1"/>
                    <field name="active" invisible="1"/>
                    <field name="is_favorite" nolabel="1" width="1" widget="boolean_favorite"/>
                    <field name="display_name" string="Name" class="fw-bold"/>
                    <field name="partner_id" optional="show" string="Customer"/>
                    <field name="privacy_visibility" optional="hide"/>
                    <field name="company_id" optional="show"  groups="base.group_multi_company" options="{'no_create': True, 'no_create': True}"/>
                    <field name="company_id" invisible="1"/>
                    <field name="analytic_account_id" optional="hide" groups="analytic.group_analytic_accounting"/>
                    <field name="date_start" string="Start Date" widget="daterange" options="{'related_end_date': 'date'}"/>
                    <field name="date" string="End Date" widget="daterange" options="{'related_start_date': 'date_start'}"/>
                    <field name="user_id" optional="show" string="Project Manager" widget="many2one_avatar_user" options="{'no_open':True, 'no_create': True, 'no_create_edit': True}"/>
                    <field name="last_update_color" invisible="1"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                    <field name="last_update_status" string="Status" nolabel="1" optional="show" widget="status_with_color" options="{'color_field': 'last_update_color', 'hide_label': True}"/>
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
                                        <div class="oe_kanban_bottom_right float-end">
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
                    <div class="oe_title">
                        <label for="name" string="Name"/>
                        <h1>
                            <field name="name" class="o_project_name oe_inline" placeholder="e.g. Office Party"/>
                        </h1>
                    </div>
                    <field name="user_id" invisible="1"/>
                    <div class="row o_settings_container"/>
                    <div name="alias_def" colspan="2" attrs="{'invisible': [('alias_domain', '=', False)]}" dir="ltr">
                        <label for="alias_name" class="oe_inline mt-4" string="Create tasks by sending an email to"/>
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
                <xpath expr="//div[@name='alias_def']" position="after">
                    <footer>
                        <button string="Create project" name="action_view_tasks" type="object" class="btn-primary o_open_tasks" data-hotkey="q"/>
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
                <kanban
                    class="oe_background_grey o_kanban_dashboard o_project_kanban o_emphasize_colors"
                    on_create="project.open_create_project"
                    action="action_view_tasks" type="object"
                    sample="1"
                    default_order="sequence, name, id"
                >
                    <field name="display_name"/>
                    <field name="partner_id"/>
                    <field name="commercial_partner_id"/>
                    <field name="color"/>
                    <field name="task_count"/>
                    <field name="milestone_count_reached"/>
                    <field name="milestone_count"/>
                    <field name="allow_milestones"/>
                    <field name="label_tasks"/>
                    <field name="alias_id"/>
                    <field name="alias_name"/>
                    <field name="alias_domain"/>
                    <field name="is_favorite"/>
                    <field name="rating_count" />
                    <field name="rating_avg"/>
                    <field name="rating_status"/>
                    <field name="rating_active" />
                    <field name="analytic_account_id"/>
                    <field name="date"/>
                    <field name="privacy_visibility"/>
                    <field name="last_update_color"/>
                    <field name="last_update_status"/>
                    <field name="tag_ids"/>
                    <progressbar field="last_update_status" colors='{"on_track": "success", "at_risk": "warning", "off_track": "danger", "on_hold": "info"}'/>
                    <field name="sequence" widget="handle"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{kanban_color(record.color.raw_value)} oe_kanban_global_click o_has_icon oe_kanban_content oe_kanban_card">
                                <div class="o_project_kanban_main ">
                                    <div class="o_kanban_card_content mw-100">
                                        <div class="o_kanban_primary_left">
                                            <div class="o_primary">
                                                <span class="o_text_overflow" t-att-title="record.display_name.value"><t t-esc="record.display_name.value"/></span>
                                                <span class="o_text_overflow text-muted" t-if="record.partner_id.value">
                                                    <span class="fa fa-user me-2" aria-label="Partner" title="Partner"></span><t t-esc="record.partner_id.value"/>
                                                </span>
                                                <div t-if="record.date.raw_value or record.date_start.raw_value" class="text-muted o_row">
                                                    <span class="fa fa-clock-o me-2" title="Dates"></span><field name="date_start"/>
                                                    <i t-if="record.date.raw_value and record.date_start.raw_value" class="fa fa-long-arrow-right mx-2 oe_read_only" aria-label="Arrow icon" title="Arrow"/>
                                                    <field name="date"/>
                                                </div>
                                                <div t-if="record.alias_name.value and record.alias_domain.value" class="text-muted text-truncate" t-att-title="record.alias_id.value">
                                                    <span class="fa fa-envelope-o me-2" aria-label="Domain Alias" title="Domain Alias"></span><t t-esc="record.alias_id.value"/>
                                                </div>
                                                <div t-if="record.rating_active.raw_value and record.rating_count.raw_value &gt; 0" class="text-muted" groups="project.group_project_rating">
                                                    <b class="me-1">
                                                        <span style="font-weight:bold;" class="fa mt4 fa-smile-o text-success" t-if="record.rating_avg.raw_value &gt;= 3.66" title="Average Rating: Satisfied" role="img" aria-label="Happy face"/>
                                                        <span style="font-weight:bold;" class="fa mt4 fa-meh-o text-warning" t-elif="record.rating_avg.raw_value &gt;= 2.33" title="Average Rating: Okay" role="img" aria-label="Neutral face"/>
                                                        <span style="font-weight:bold;" class="fa mt4 fa-frown-o text-danger" t-else="" title="Average Rating: Dissatisfied" role="img" aria-label="Sad face"/>
                                                    </b>
                                                    <field name="rating_avg_percentage" widget="percentage"/>
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
                                                <div role="menuitem" groups="project.group_project_milestone" t-if="record.allow_milestones.raw_value">
                                                    <a name="action_get_list_view" type="object">Milestones</a>
                                                </div>
                                            </div>
                                            <div class="col-6 o_kanban_card_manage_section o_kanban_manage_reporting">
                                                <div role="menuitem" class="o_kanban_card_manage_title" groups="project.group_project_user">
                                                    <span>Reporting</span>
                                                </div>
                                                <div role="menuitem" groups="project.group_project_user">
                                                    <a name="action_view_tasks_analysis" type="object">Tasks Analysis</a>
                                                </div>
                                                <div role="menuitem" name="project_burndown_menu" groups="project.group_project_user">
                                                    <a name="action_project_task_burndown_chart_report" type="object">Burndown Chart</a>
                                                </div>
                                            </div>
                                        </div>
                                        <div class="o_kanban_card_manage_settings row">
                                            <div role="menuitem" aria-haspopup="true" class="col-6" groups="project.group_project_manager">
                                                <ul class="oe_kanban_colorpicker" data-field="color" role="popup"/>
                                            </div>
                                            <div role="menuitem" class="col-6" groups="project.group_project_manager">
                                                <a t-if="record.privacy_visibility.raw_value == 'portal'" class="dropdown-item" role="menuitem" name="%(project.project_share_wizard_action)d" type="action">Share</a>
                                                <a class="dropdown-item" role="menuitem" type="edit">Settings</a>
                                            </div>
                                            <div class="o_kanban_card_manage_section o_kanban_manage_view col-12 row ps-0" groups="!project.group_project_manager">
                                                <div role="menuitem" class="w-100">
                                                    <a class="dropdown-item mx-0" role="menuitem" type="open">View</a>
                                                </div>
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
                                        <div class="o_project_kanban_boxes d-flex align-items-baseline">
                                            <a class="o_project_kanban_box" name="action_view_tasks" type="object">
                                                <div>
                                                    <span class="o_value"><t t-esc="record.task_count.value"/></span>
                                                    <span class="o_label ms-1"><t t-esc="record.label_tasks.value"/></span>
                                                </div>
                                            </a>
                                            <a groups='project.group_project_milestone' t-if="record.allow_milestones and record.allow_milestones.raw_value and record.milestone_count.value &gt; 0"
                                                class="o_kanban_inline_block text-muted small"
                                                name="action_get_list_view"
                                                type="object"
                                                t-attf-title="#{record.milestone_count_reached.value} Milestones reached out of #{record.milestone_count.value}"
                                            >
                                                <span class="fa fa-check-square-o me-1"/>
                                                <t t-out="record.milestone_count_reached.value"/>/<t t-out="record.milestone_count.value"/>
                                            </a>
                                        </div>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field t-if="record.last_update_status.value &amp;&amp; widget.editable" name="last_update_status" widget="project_state_selection" options="{'color_field': 'last_update_color', 'hide_label': 1}"/>
                                        <span t-if="record.last_update_status.value &amp;&amp; !widget.editable" t-att-class="'o_status_bubble mx-0 o_color_bubble_' + record.last_update_color.value" t-att-title="record.last_update_status.value"></span>
                                        <field name="user_id" widget="many2one_avatar_user" t-if="record.user_id.raw_value"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_project_config_kanban" model="ir.ui.view">
            <field name="name">project.kanban.inherit.config.project</field>
            <field name="model">project.project</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="view_project_kanban"/>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="action"></attribute>
                </xpath>
            </field>
        </record>

        <record id="view_project_calendar" model="ir.ui.view">
            <field name="name">project.project.calendar</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <calendar
                    date_start="date_start"
                    date_stop="date"
                    string="Projects"
                    mode="month"
                    scales="month,year"
                    event_open_popup="true"
                    quick_add="false"
                    color="color">
                    <field name="partner_id" attrs="{'invisible': [('partner_id', '=', False)]}"/>
                    <field name="user_id" widget="many2one_avatar_user" attrs="{'invisible': [('user_id', '=', False)]}"/>
                    <field name="is_favorite" widget="boolean_favorite" nolabel="1" string="Favorite"/>
                    <field name="stage_id" groups="project.group_project_stages"/>
                    <field name="last_update_color" invisible="1"/>
                    <field name="last_update_status" string="Status" widget="status_with_color" options="{'color_field': 'last_update_color'}"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" attrs="{'invisible': [('tag_ids', '=', [])]}"/>
                </calendar>
            </field>
        </record>

        <!-- Please update both act_window when modifying one (open_view_project_all or open_view_project_all_group_stage) as one or the other is used in the menu menu_project -->
        <record id="open_view_project_all" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="domain">[]</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="view_id" ref="view_project_kanban"/>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="target">main</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No projects found. Let's create one!
                </p>
                <p>
                    Create projects to organize your tasks. Define a different workflow for each project.
                </p>
            </field>
        </record>

        <!-- Please update both act_window when modifying one (open_view_project_all or open_view_project_all_group_stage) as one or the other is used in the menu menu_project -->
        <record id="open_view_project_all_group_stage" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="context">{'search_default_groupby_stage': 1}</field>
            <field name="domain">[]</field>
            <field name="view_mode">kanban,tree,form,calendar,activity</field>
            <field name="view_id" ref="view_project_kanban"/>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="target">main</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No projects found. Let's create one!
                </p>
                <p>
                    Projects contain tasks on the same topic, and each has its own dashboard.
                </p>
            </field>
        </record>

        <!-- Please update both act_window when modifying one (open_view_project_all_config or open_view_project_all_config_group_stage) as one or the other is used in the menu menu_project_config -->
        <record id="open_view_project_all_config" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="domain">[]</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('view_project')}),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('view_project_config_kanban')})]"/>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="context">{}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                   No projects found. Let's create one!
                </p>
                <p>
                    Create projects to organize your tasks and define a different workflow for each project.
                </p>
            </field>
        </record>

        <!-- Please update both act_window when modifying one (open_view_project_all_config or open_view_project_all_config_group_stage) as one or the other is used in the menu menu_project_config -->
        <record id="open_view_project_all_config_group_stage" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="domain">[]</field>
            <field name="view_mode">tree,kanban,form,calendar,activity</field>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="context">{}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                   No projects found. Let's create one!
                </p>
                <p>
                    Projects contain tasks on the same topic, and each has its own dashboard.
                </p>
            </field>
        </record>

        <record id="open_view_project_all_config_group_stage_tree_action_view" model="ir.actions.act_window.view">
            <field name="sequence" eval="10"/>
            <field name="view_mode">tree</field>
            <field name="act_window_id" ref="project.open_view_project_all_config_group_stage"/>
            <field name="view_id" ref="view_project"/>
        </record>
        <record id="open_view_project_all_config_group_stage_kanban_action_view" model="ir.actions.act_window.view">
            <field name="sequence" eval="20"/>
            <field name="view_mode">kanban</field>
            <field name="act_window_id" ref="project.open_view_project_all_config_group_stage"/>
            <field name="view_id" ref="view_project_config_kanban"/>
        </record>

        <!-- Task -->
        <record id="view_task_form2" model="ir.ui.view">
            <field name="name">project.task.form</field>
            <field name="model">project.task</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <form string="Task" class="o_form_project_tasks" js_class="project_task_form">
                    <field name="allow_subtasks" invisible="1" />
                    <field name="is_closed" invisible="1" />
                    <field name="allow_recurring_tasks" invisible="1" />
                    <field name="repeat_show_dow" invisible="1" />
                    <field name="repeat_show_day" invisible="1" />
                    <field name="repeat_show_week" invisible="1" />
                    <field name="repeat_show_month" invisible="1" />
                    <field name="recurrence_id" invisible="1" />
                    <field name="allow_task_dependencies" invisible="1" />
                    <field name="rating_last_value" invisible="1"/>
                    <field name="rating_count" invisible="1"/>
                    <field name="allow_milestones" invisible="1" />
                    <field name="parent_id" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <header>
                        <button name="action_assign_to_me" string="Assign to Me" type="object" attrs="{'invisible': &quot;['|', ('user_ids', 'in', uid), ('user_ids', '=', [])]&quot;}" data-hotkey="q"/>
                        <button name="action_assign_to_me" string="Assign to Me" type="object" class="oe_highlight" attrs="{'invisible' : &quot;['|', ('user_ids', 'in', uid), ('user_ids', '!=', [])]&quot;}" data-hotkey="q"/>
                        <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" attrs="{'invisible': [('project_id', '=', False), ('stage_id', '=', False)]}"/>
                        <field name="personal_stage_type_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" attrs="{'invisible': [('project_id', '!=', False)]}" domain="[('user_id', '=', uid)]"/>
                    </header>
                    <div class="alert alert-info oe_edit_only mb-0" role="status" attrs="{'invisible': ['|', ('recurring_task', '=', False), ('recurrence_id', '=', False)]}" groups="project.group_project_recurring_tasks">
                        <p>Edit recurring task</p>
                        <field name="recurrence_update" widget="radio"/>
                    </div>
                    <sheet string="Task">
                    <div class="oe_button_box" name="button_box">
                        <!-- Dummy tag for organizing buttons, using position='replace' when inheriting -->
                        <span id="button_products" invisible="1"/>
                        <span id="button_worksheet" invisible="1"/>
                        <!-- Dummy tag used to organize buttons, englobing the 3 buttons modifies the width of the button -->
                        <span id="start_rating_buttons" invisible="1"/>
                        <field name="rating_avg" invisible="1"/>
                        <field name="rating_active" invisible="1"/>
                        <button name="action_open_ratings" type="object" attrs="{'invisible': ['|', ('rating_count', '=', 0), ('rating_active', '=', False)]}" class="oe_stat_button" groups="project.group_project_rating">
                            <i class="fa fa-fw o_button_icon fa-smile-o text-success" attrs="{'invisible': [('rating_avg', '&lt;', 3.66)]}" title="Satisfied"/>
                            <i class="fa fa-fw o_button_icon fa-meh-o text-warning" attrs="{'invisible': ['|', ('rating_avg', '&lt;', 2.33), ('rating_avg', '&gt;=', 3.66)]}" title="Okay"/>
                            <i class="fa fa-fw o_button_icon fa-frown-o text-danger" attrs="{'invisible': [('rating_avg', '&gt;=', 2.33)]}" title="Dissatisfied"/>
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="rating_avg_text" nolabel="1"/></span>
                                <span class="o_stat_text">Last Rating</span>
                            </div>
                        </button>
                        <!-- Dummy tag used to organize buttons -->
                        <span id="end_rating_buttons" invisible="1"/>
                        <button name="action_open_parent_task" type="object" class="oe_stat_button" icon="fa-tasks" string="Parent Task" attrs="{'invisible': ['|', ('allow_subtasks', '=', False), ('parent_id', '=', False)]}" groups="project.group_subtask_project"/>
                        <button name="action_recurring_tasks" type="object" attrs="{'invisible': [('recurrence_id', '=', False)]}" class="oe_stat_button" icon="fa-repeat" groups="project.group_project_recurring_tasks">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="recurring_count" widget="statinfo" nolabel="1" />
                                    Tasks
                                </span>
                                <span class="o_stat_text">in Recurrence</span>
                            </div>
                        </button>
                        <button name="%(project_task_action_sub_task)d" type="action" class="oe_stat_button" icon="fa-tasks"
                            attrs="{'invisible' : ['|', '|', ('allow_subtasks', '=', False), ('id', '=', False), ('subtask_count', '=', 0)]}" context="{'default_user_ids': user_ids}">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="subtask_count" widget="statinfo" nolabel="1"/>
                                </span>
                                <span class="o_stat_text">Sub-tasks</span>
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
                        <!-- Dummy tag used to organize buttons -->
                        <span id="end_button_box" invisible="1"/>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title pe-0">
                        <h1 class="d-flex justify-content-between align-items-center">
                            <div class="d-flex w-100">
                                <field name="priority" widget="priority" class="me-3"/>
                                <field name="name" class="o_task_name text-truncate w-100 w-md-75 pe-2" placeholder="Task Title..."/>
                            </div>
                            <field name="kanban_state" widget="state_selection" class=""/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="project_id"
                                   placeholder="Private"
                                   domain="[('active', '=', True), ('company_id', '=', company_id)]"
                                   attrs="{'invisible': [('parent_id', '!=', False)]}"
                                   widget="project_private_task"
                            />
                            <field name="display_project_id" string="Project" attrs="{'invisible': [('parent_id', '=', False)]}" domain="[('active', '=', True), ('company_id', '=', company_id)]"/>
                            <field name="milestone_id"
                                placeholder="e.g. Product Launch"
                                context="{'default_project_id': project_id if not parent_id or not display_project_id else display_project_id}"
                                attrs="{'invisible': ['|', ('project_id', '=', False), ('allow_milestones', '=', False)]}"
                            />
                            <field name="user_ids"
                                class="o_task_user_field"
                                options="{'no_open': True, 'no_quick_create': True}"
                                widget="many2many_avatar_user"
                                domain="[('share', '=', False), ('active', '=', True)]"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="partner_id" widget="res_partner_many2one" class="o_task_customer_field"/>
                            <field name="partner_phone" widget="phone" attrs="{'invisible': True}"/>
                            <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}" context="{'project_id': project_id}"/>
                            <field name="legend_blocked" invisible="1"/>
                            <field name="legend_normal" invisible="1"/>
                            <field name="legend_done" invisible="1"/>
                        </group>
                    </group>
                    <field attrs="{'invisible': [('project_id', '=', False)]}"
                        name="task_properties" nolabel="1" columns="2" hideKanbanOption="1"/>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" type="html" options="{'collaborative': true, 'resizable': false}" placeholder="Task description..."/>
                        </page>
                        <page name="sub_tasks_page" string="Sub-tasks" attrs="{'invisible': [('allow_subtasks', '=', False)]}">
                            <field name="child_ids"
                                   context="{'search_view_ref' : 'project.view_task_search_form_extended', 'default_project_id': project_id if not parent_id or not display_project_id else display_project_id, 'default_user_ids': user_ids, 'default_parent_id': id,
                                    'default_partner_id': partner_id, 'default_milestone_id': allow_milestones and milestone_id, 'search_default_display_project_id': project_id }"
                                   widget="many2many"
                                   domain="['!', ('id', 'parent_of', id)]">
                                <tree editable="bottom" decoration-muted="is_closed == True">
                                    <field name="legend_normal" invisible="1"/>
                                    <field name="legend_done" invisible="1"/>
                                    <field name="legend_blocked" invisible="1"/>
                                    <field name="project_id" invisible="1"/>
                                    <field name="is_closed" invisible="1"/>
                                    <field name="allow_milestones" invisible="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="priority" widget="priority" optional="show" nolabel="1" options="{'autosave': False}"/>
                                    <field name="id" optional="hide"/>
                                    <field name="child_text" invisible="1"/>
                                    <field name="allow_subtasks" invisible="1"/>
                                    <field name="name" widget="name_with_subtask_count"/>
                                    <field name="display_project_id" string="Project" optional="hide" options="{'no_open': 1}"/>
                                    <field name="milestone_id"
                                        optional="hide"
                                        context="{'default_project_id': display_project_id or project_id}"
                                        attrs="{'invisible': [('allow_milestones', '=', False)], 'column_invisible': [('parent.allow_milestones', '=', False)]}"
                                    />
                                    <field name="partner_id" optional="hide"/>
                                    <field name="user_ids" widget="many2many_avatar_user" optional="show" domain="[('share', '=', False), ('active', '=', True)]"/>
                                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                                    <field name="company_id" invisible="1"/>
                                    <field name="activity_ids" string="Next Activity" widget="list_activity" optional="hide"/>
                                    <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}" optional="show"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="rating_last_text" string="Rating" decoration-danger="rating_last_text == 'ko'"
                                        decoration-warning="rating_last_text == 'ok'" decoration-success="rating_last_text == 'top'"
                                        attrs="{'invisible': [('rating_last_text', '=', 'none')]}"
                                        class="fw-bold" widget="badge" optional="hide"/>
                                    <field name="kanban_state" widget="state_selection" optional="show" options="{'hide_label': True, 'autosave': False}" nolabel="1"/>
                                    <field name="stage_id" optional="show" context="{'default_project_id': project_id}"/>
                                    <button name="action_open_task" type="object" title="View Task" string="View Task" class="btn btn-link float-end"/>
                                </tree>
                            </field>
                        </page>
                        <page name="task_dependencies" string="Blocked By" attrs="{'invisible': [('allow_task_dependencies', '=', False)]}" groups="project.group_project_task_dependencies">
                            <field name="depend_on_ids" nolabel="1" context="{'default_project_id' : project_id}">
                                <tree editable="bottom" decoration-muted="is_closed == True">
                                    <field name="allow_milestones" invisible="1"/>
                                    <field name="parent_id" invisible="1" />
                                    <field name="display_project_id" invisible="1" />
                                    <field name="is_closed" invisible="1" />
                                    <field name="priority" widget="priority" optional="show" nolabel="1" options="{'autosave': False}"/>
                                    <field name="child_text" invisible="1"/>
                                    <field name="allow_subtasks" invisible="1"/>
                                    <field name="name" widget="name_with_subtask_count"/>
                                    <field name="id" optional="hide"/>
                                    <field name="project_id" optional="hide" options="{'no_open': 1}" />
                                    <field name="milestone_id"
                                        optional="hide"
                                        context="{'default_project_id': project_id if not parent_id or not display_project_id else display_project_id}"
                                        attrs="{'invisible': [('allow_milestones', '=', False)], 'column_invisible': [('parent.allow_milestones', '=', False)]}"
                                    />
                                    <field name="partner_id" optional="hide"/>
                                    <field name="parent_id" optional="hide" attrs="{'invisible': [('allow_subtasks', '=', False)]}" groups="base.group_no_one"/>
                                    <field name="user_ids" widget="many2many_avatar_user" optional="show" domain="[('share', '=', False), ('active', '=', True)]"/>
                                    <field name="company_id" optional="hide" groups="base.group_multi_company" />
                                    <field name="company_id" invisible="1"/>
                                    <field name="activity_ids" string="Next Activity" widget="list_activity" optional="hide"/>
                                    <field name="date_deadline" attrs="{'invisible': [('is_closed', '=', True)]}" optional="show" />
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="rating_last_text" string="Rating" decoration-danger="rating_last_text == 'ko'"
                                        decoration-warning="rating_last_text == 'ok'" decoration-success="rating_last_text == 'top'"
                                        attrs="{'invisible': [('rating_last_text', '=', 'none')]}"
                                        class="fw-bold" widget="badge" optional="hide"/>
                                    <field name="legend_normal" invisible="1"/>
                                    <field name="legend_done" invisible="1"/>
                                    <field name="legend_blocked" invisible="1"/>
                                    <field name="kanban_state" widget="state_selection" optional="show" options="{'hide_label': True, 'autosave': False}" nolabel="1"/>
                                    <field name="stage_id" optional="show" />
                                    <button class="oe_link float-end" string="View Task" name="action_open_task" type="object"/>
                                </tree>
                            </field>
                        </page>
                        <page name="recurrence" string="Recurrent" groups="project.group_project_recurring_tasks">
                            <label for="recurring_task" />
                            <field name="recurring_task" class="ms-5" attrs="{'invisible': ['|', ('allow_recurring_tasks', '=', False), ('active', '=', False)]}"/>
                            <group attrs="{'invisible': [('recurring_task', '=', False)]}">
                                <group>
                                    <label for="repeat_interval" />
                                    <div class="o_col">
                                        <div class="o_row">
                                            <field name="repeat_interval" attrs="{'required': [('recurring_task', '=', True)]}" />
                                            <field name="repeat_unit" attrs="{'required': [('recurring_task', '=', True)]}" />
                                        </div>
                                        <widget name="week_days" attrs="{'invisible': [('repeat_show_dow', '=', False)]}" groups="project.group_project_user"/>
                                    </div>

                                    <label for="repeat_on_month" string="Repeat On" attrs="{'invisible': [('repeat_unit', 'not in', ('month', 'year'))]}" />
                                    <div class="o_row" attrs="{'invisible': [('repeat_unit', 'not in', ('month', 'year'))]}">
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
                            <group attrs="{'invisible': ['|', ('recurring_task', '=', False), ('recurrence_message', '=', False)]}" groups="project.group_project_user">
                                <div class="alert alert-success o_form_project_recurrence_message" role="status" colspan="2">
                                    <field name="recurrence_message" widget="html" class="mb-0" />
                                </div>
                            </group>
                        </page>
                        <page name="extra_info" string="Extra Info" groups="base.group_no_one">
                            <group>
                                <group>
                                    <field name="is_analytic_account_id_changed" invisible="1"/>
                                    <field name="parent_id" attrs="{'invisible': [('allow_subtasks', '=', False)]}" groups="base.group_no_one"/>
                                    <field name="analytic_account_id" groups="analytic.group_analytic_accounting" context="{'default_partner_id': partner_id}"/>
                                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                    <field name="sequence" groups="base.group_no_one"/>
                                    <field name="email_from" invisible="1"/>
                                    <field name="email_cc" groups="base.group_no_one"/>
                                    <field name="displayed_image_id" groups="base.group_no_one" options="{'no_create': True}"/>
                                </group>
                                <group>
                                    <field name="date_assign" groups="base.group_no_one"/>
                                    <field name="date_last_stage_update" groups="base.group_no_one"/>
                                </group>
                                <group string="Working Time to Assign" attrs="{'invisible': [('working_hours_open', '=', 0.0)]}">
                                    <field name="working_hours_open" widget="float_time" string="Hours"/>
                                    <field name="working_days_open" string="Days"/>
                                </group>
                                <group string="Working Time to Close" attrs="{'invisible': [('working_hours_close', '=', 0.0)]}">
                                    <field name="working_hours_close" widget="float_time" string="Hours"/>
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

        <record id="quick_create_task_form" model="ir.ui.view">
            <field name="name">project.task.form.quick_create</field>
            <field name="model">project.task</field>
            <field name="priority">1000</field>
            <field name="arch" type="xml">
                <form class="o_form_project_tasks">
                    <group>
                        <field name="name" string = "Task Title" placeholder="e.g. Send Invitations"/>
                        <field name="project_id" widget="project_private_task" invisible="context.get('default_project_id', False)" placeholder="Private" class="o_project_task_project_field"/>
                        <field name="user_ids" options="{'no_open': True, 'no_quick_create': True}" domain="[('share', '=', False), ('active', '=', True)]"
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
                <kanban
                    default_group_by="stage_id"
                    class="o_kanban_small_column o_kanban_project_tasks"
                    on_create="quick_create"
                    quick_create_view="project.quick_create_task_form"
                    examples="project"
                    js_class="project_task_kanban" sample="1"
                >
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
                    <field name="rating_count"/>
                    <field name="rating_avg"/>
                    <field name="allow_subtasks"/>
                    <field name="child_text"/>
                    <field name="is_private"/>
                    <field name="rating_active"/>
                    <field name="has_late_and_unreached_milestone" />
                    <field name="allow_milestones" />
                    <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger", "normal": "200"}'/>
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
                                        <span invisible="context.get('default_project_id', False)"><br/><field name="project_id" widget="project_private_task" options="{'no_open': True}"/></span>
                                        <span t-if="record.allow_milestones.raw_value and record.milestone_id.raw_value" t-attf-class="{{record.has_late_and_unreached_milestone.raw_value ? 'text-danger' : ''}}">
                                            <br/>
                                            <field name="milestone_id" options="{'no_open': True}" />
                                        </span>
                                        <br />
                                        <t t-if="record.partner_id.value">
                                            <span t-if="!record.partner_is_company.raw_value" t-attf-title="#{record.commercial_partner_id.value}">
                                                <field name="commercial_partner_id" class="text-truncate d-block"/>
                                            </span>
                                            <span t-else="" t-attf-title="#{record.partner_id.value}">
                                                <field name="partner_id" class="text-truncate d-block"/>
                                            </span>
                                        </t>
                                        <t t-else="record.email_from.raw_value"><span><field name="email_from"/></span></t>
                                    </div>
                                    <div class="o_dropdown_kanban dropdown" t-if="!selection_mode" groups="base.group_user">
                                        <a role="button" class="dropdown-toggle o-no-caret btn" data-bs-toggle="dropdown" data-bs-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
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
                                        <b t-if="record.rating_active.raw_value and record.rating_count.raw_value &gt; 0" groups="project.group_project_rating">
                                            <span style="font-weight:bold;" class="fa fa-fw mt4 fa-smile-o text-success" t-if="record.rating_avg.raw_value &gt;= 3.66" title="Average Rating: Satisfied" role="img" aria-label="Happy face"/>
                                            <span style="font-weight:bold;" class="fa fa-fw mt4 fa-meh-o text-warning" t-elif="record.rating_avg.raw_value &gt;= 2.33" title="Average Rating: Okay" role="img" aria-label="Neutral face"/>
                                            <span style="font-weight:bold;" class="fa fa-fw mt4 fa-frown-o text-danger" t-else="" title="Average Rating: Dissatisfied" role="img" aria-label="Sad face"/>
                                        </b>
                                    </div>
                                    <div class="oe_kanban_bottom_right" t-if="!selection_mode">
                                        <field name="kanban_state" widget="state_selection" groups="base.group_user"/>
                                        <t t-if="record.user_ids.raw_value"><field name="user_ids" widget="many2many_avatar_user"/></t>
                                    </div>
                                </div>
                            </div>
                            <div class="clearfix"></div>
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
                <tree string="Tasks" multi_edit="1" sample="1" js_class="project_task_list">
                    <field name="message_needaction" invisible="1" readonly="1"/>
                    <field name="is_closed" invisible="1" />
                    <field name="sequence" invisible="1" readonly="1"/>
                    <field name="allow_milestones" invisible="1"/>
                    <field name="priority" widget="priority" optional="show" nolabel="1"/>
                    <field name="id" optional="hide"/>
                    <field name="child_text" invisible="1"/>
                    <field name="allow_subtasks" invisible="1"/>
                    <field name="name" widget="name_with_subtask_count"/>
                    <field name="project_id" widget="project_private_task" optional="show" readonly="1" options="{'no_open': 1}"/>
                    <field name="milestone_id" attrs="{'invisible': [('allow_milestones', '=', False)]}" context="{'default_project_id': project_id}" groups="project.group_project_milestone"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="parent_id" optional="hide" attrs="{'invisible': [('allow_subtasks', '=', False)]}" groups="base.group_no_one"/>
                    <field name="user_ids" optional="show" widget="many2many_avatar_user" domain="[('share', '=', False), ('active', '=', True)]" options='{"no_quick_create": True}'/>
                    <field name="company_id" groups="base.group_multi_company" optional="show" readonly="True"/>
                    <field name="company_id" invisible="1"/>
                    <field name="activity_ids" string="Next Activity" widget="list_activity" optional="show"/>
                    <field name="date_deadline" optional="hide" widget="remaining_days" attrs="{'invisible': [('is_closed', '=', True)]}"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show" context="{'project_id': project_id}"/>
                    <field name="rating_active" invisible="1"/>
                    <field name="rating_last_text" string="Rating" decoration-danger="rating_last_text == 'ko'"
                        decoration-warning="rating_last_text == 'ok'" decoration-success="rating_last_text == 'top'"
                        attrs="{'invisible': ['|', ('rating_active', '=', False), ('rating_last_text', '=', 'none')]}"
                        class="fw-bold" widget="badge" optional="hide" groups="project.group_project_rating"/>
                    <field name="legend_normal" invisible="1"/>
                    <field name="legend_done" invisible="1"/>
                    <field name="legend_blocked" invisible="1"/>
                    <field name="kanban_state" widget="state_selection" optional="show" options="{'hide_label': True}" nolabel="1" required="0"/>
                    <field name="stage_id" invisible="context.get('set_visible',False)" optional="show" readonly="not context.get('default_project_id')"/>
                    <field name="recurrence_id" invisible="1" />
                </tree>
            </field>
        </record>

        <!--
            The below view is invalid since `multi_edit="1"` has been set on the tree
            https://github.com/odoo/odoo/commit/972e097dbe7cdc7afdae722391692f9b3bf063b8#diff-db3b2f2e90f34ffef22194c3eac5a34b17af9b9a0a644d77da9e80738963931eL859
            because `activity_type_id` uses in his domain a field `res_model`
            which is not available in the view and not even in the model.
            It has not been detected because:
            - this view is unused / impossible to reach since Odoo 12.0,
              and therefore nobody tried to edit the many2one field `activity_type_id` which would trigger the issue
            - the field domains were not validated because `multi_edit="1"` tree views were not considered as editable,
              and therefore their field domains were not validated in the views validation, as they should have been.
            This view has been removed in master as it was no longer used,
            through https://github.com/odoo/odoo/pull/103702.
            We cannot remove it in 16.0 because it would require an upgrade script.
            Hence we just remove the multi_edit="1" so the view is not validated
            and the domain is not used in the web client.
        -->
        <record id="project_task_view_tree_activity" model="ir.ui.view">
            <field name="name">project.task.tree.activity</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <tree string="Next Activities" decoration-danger="not is_closed and activity_date_deadline &lt; current_date" default_order="activity_date_deadline">
                    <field name="company_id" invisible="1"/>
                    <field name="is_closed"/>
                    <field name="name"/>
                    <field name="project_id" options="{'no_open': 1}"/>
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
                <calendar date_start="date_deadline" string="Tasks" mode="month"
                          color="color" event_limit="5" hide_time="true"
                          event_open_popup="true" quick_add="false" show_unusual_days="True"
                          js_class="project_calendar"
                          scales="month,year">
                    <field name="allow_milestones" invisible="1" />
                    <field name="project_id" widget="project_private_task"/>
                    <field name="milestone_id" attrs="{'invisible': [('allow_milestones', '=', False)]}"/>
                    <field name="user_ids" widget="many2many_avatar_user"/>
                    <field name="partner_id" attrs="{'invisible': [('partner_id', '=', False)]}"/>
                    <field name="priority" widget="priority"/>
                    <field name="date_deadline"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" attrs="{'invisible': [('tag_ids', '=', [])]}"/>
                    <field name="stage_id"/>
                    <field name="kanban_state"/>
                </calendar>
            </field>
        </record>

        <record id="view_task_all_calendar" model="ir.ui.view">
            <field name="name">project.task.all.calendar</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_calendar"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//calendar" position="attributes">
                    <attribute name="color">project_color</attribute>
                </xpath>
                <xpath expr="//field[@name='project_id']" position="attributes">
                    <attribute name="filters">1</attribute>
                    <attribute name="color">color</attribute>
                </xpath>
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
                    <field name="rating_last_value" string="Rating (/5)"/>
                </graph>
            </field>
        </record>

        <record id="project_task_view_activity" model="ir.ui.view">
            <field name="name">project.task.activity</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <activity string="Project Tasks" js_class="project_activity">
                    <field name="user_ids"/>
                    <field name="project_id"/>
                    <templates>
                        <div class="justify-content-between" t-name="activity-box">
                            <field name="user_ids" widget="many2many_avatar_user"/>
                            <div class="text-end">
                                <span t-att-title="record.name.value">
                                    <field name="name" display="full"/>
                                </span>
                                <span t-att-title="record.project_id.value">
                                    <field t-if="record.project_id.value" name="project_id" muted="1" display="full"/>
                                    <span t-else="" class="fst-italic text-muted"><i class="fa fa-lock"></i> Private</span>
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
                </p>
                <p>
                    Keep track of the progress of your tasks from creation to completion.<br/>
                    Collaborate efficiently by chatting in real-time or via email.
                </p>
            </field>
        </record>

        <record id="view_task_kanban_inherit_my_task" model="ir.ui.view">
            <field name="name">project.task.kanban.inherit.my.task</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="default_group_by">personal_stage_type_ids</attribute>
                </xpath>
            </field>
        </record>

        <record id="action_view_all_task" model="ir.actions.act_window">
            <field name="name">My Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form,calendar,pivot,graph,activity</field>
            <field name="context">{'search_default_my_tasks': 1, 'search_default_open_tasks': 1, 'all_task': 0, 'default_user_ids': [(4, uid)]}</field>
            <field name="search_view_id" ref="view_task_search_form_extended"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p>
                <p>
                    Organize your tasks by dispatching them across the pipeline.<br/>
                    Collaborate efficiently by chatting in real-time or via email.
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

        <record id="open_view_all_task_list_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="10"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="view_task_kanban_inherit_my_task"/>
            <field name="act_window_id" ref="action_view_all_task"/>
        </record>
        <record id="open_view_all_task_list_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="20"/>
            <field name="view_mode">tree</field>
            <field name="act_window_id" ref="action_view_all_task"/>
        </record>
        <record id="open_view_all_task_list_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="40"/>
            <field name="view_mode">calendar</field>
            <field name="act_window_id" ref="action_view_all_task"/>
            <field name="view_id" ref="view_task_all_calendar"/>
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

        <record id="action_view_task_from_milestone" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,calendar,pivot,graph,activity,form</field>
            <field name="context">{'default_milestone_id': active_id}</field>
            <field name="domain">[('milestone_id', '=', active_id)]</field>
            <field name="search_view_id" ref="view_task_search_form"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No tasks found. Let's create one!
                </p>
                <p>
                    Keep track of the progress of your tasks from creation to completion.<br/>
                    Collaborate efficiently by chatting in real-time or via email.
                </p>
            </field>
        </record>

        <!-- Menu item for project -->
        <menuitem id="menu_tasks_config" name="GTD" parent="menu_project_config" sequence="2"/>

        <menuitem action="project_project_stage_configure" id="menu_project_config_project_stage" name="Project Stages" parent="menu_project_config" sequence="9" groups="project.group_project_stages"/>

        <menuitem action="open_task_type_form" id="menu_project_config_project" name="Task Stages" parent="menu_project_config" sequence="10" groups="base.group_no_one"/>

        <menuitem action="open_view_project_all" id="menu_projects" name="Projects" parent="menu_main_pm" sequence="1"/>
        <menuitem action="open_view_project_all_group_stage" id="menu_projects_group_stage" name="Projects" parent="menu_main_pm" sequence="1" groups="project.group_project_stages"/>
        <menuitem action="open_view_project_all_config" id="menu_projects_config" name="Projects" parent="menu_project_config" sequence="5"/>
        <menuitem action="open_view_project_all_config_group_stage" id="menu_projects_config_group_stage" name="Projects" parent="menu_project_config" sequence="5" groups="project.group_project_stages"/>

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
                <tree string="Tags" editable="top" sample="1" multi_edit="1" default_order="name">
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
            sequence="51"/>

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
                    <div role="menuitem" groups="project.group_project_user">
                        <a name="project_update_all_action" type="object" t-attf-context="{'active_id': #{record.id.raw_value} }">Project Updates</a>
                    </div>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: views\rating_rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="rating_rating_view_tree_project" model="ir.ui.view">
        <field name="name">rating.rating.tree.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_tree"/>
        <field name="mode">primary</field>
        <field name="priority">64</field>
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
        <field name="inherit_id" ref="rating.rating_rating_view_form_text"/>
        <field name="mode">primary</field>
        <field name="priority">64</field>
        <field name="arch" type="xml">
            <xpath expr="//form" position="attributes">
                <attribute name="edit">0</attribute>
            </xpath>
            <field name="resource_ref" position="before">
                <field name="rated_partner_id" position="move"/>
                <field name="parent_ref" position="move"/>
            </field>
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
            <field name="create_date" position="replace">
                <field name="write_date" readonly="1" string="Submitted On"/>
            </field>
            <field name="feedback" position="attributes">
                <attribute name="readonly">1</attribute>
            </field>
            <field name="write_date" position="after">
                <field name="partner_id" position="move"/>
            </field>
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
        <field name="priority">64</field>
        <field name="arch" type="xml">
            <xpath expr="//pivot" position="attributes">
                <attribute name="js_class">project_rating_pivot</attribute>
            </xpath>
            <xpath expr="//field[@name='create_date']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_view_graph" model="ir.ui.view">
        <field name="name">rating.rating.view.graph.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_graph"/>
        <field name="mode">primary</field>
        <field name="priority">64</field>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="attributes">
                <attribute name="js_class">project_rating_graph</attribute>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_project_view_kanban" model="ir.ui.view">
        <field name="name">rating.rating.kanban.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_kanban"/>
        <field name="mode">primary</field>
        <field name="priority">64</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='create_date']" position="replace">
                <field name="write_date"/>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_view_search_project" model="ir.ui.view">
        <field name="name">rating.rating.search.project</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_search"/>
        <field name="mode">primary</field>
        <field name="priority">64</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='rated_partner_id']" position="after">
                <field name="parent_res_name" position="move"/>
                <field name="res_name" position="move"/>
            </xpath>
            <xpath expr="//field[@name='rated_partner_id']" position="attributes">
                <attribute name="string">Assigned to</attribute>
            </xpath>
            <xpath expr="//field[@name='parent_res_name']" position="attributes">
                <attribute name="string">Project</attribute>
            </xpath>
            <xpath expr="//field[@name='res_name']" position="attributes">
                <attribute name="string">Task</attribute>
            </xpath>
            <xpath expr="//filter[@name='responsible']" position="after">
                <filter name="rating_text" position="move"/>
                <filter string="Project" name="groupby_project" context="{'group_by': 'parent_res_name'}"/>
                <filter name="resource" position="move"/>
                <filter name="customer" position="move"/>
            </xpath>
            <xpath expr="//filter[@name='resource']" position="attributes">
                <attribute name="string">Task</attribute>
            </xpath>
            <xpath expr="//filter[@name='responsible']" position="attributes">
                <attribute name="string">Assigned to</attribute>
            </xpath>
            <xpath expr="//filter[@name='filter_create_date']" position="replace">
                <filter name="filter_write_date" string="Submitted On" date="write_date"/>
            </xpath>
            <xpath expr="//filter[@name='month']" position="attributes">
                <attribute name="context">{'group_by':'write_date:month'}</attribute>
            </xpath>
            <xpath expr="//filter[@name='today']" position="attributes">
                <attribute name="domain">[('write_date', '&gt;', (context_today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d'))]</attribute>
            </xpath>
            <xpath expr="//filter[@name='last_7days']" position="attributes">
                <attribute name="domain">[('write_date','&gt;', (context_today() - datetime.timedelta(days=7)).strftime('%Y-%m-%d'))]</attribute>
            </xpath>
            <xpath expr="//filter[@name='last_month']" position="attributes">
                <attribute name="domain">[('write_date','&gt;', (context_today() - relativedelta(months=1)).strftime('%Y-%m-%d'))]</attribute>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_action_view_project_rating" model="ir.actions.act_window">
        <field name="name">Ratings</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,graph,pivot,form</field>
        <field name="domain">[('consumed','=',True), ('parent_res_model','=','project.project'), ('parent_res_id', '=', active_id)]</field>
        <field name="search_view_id" ref="rating_rating_view_search_project"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There are no ratings for this project at the moment
            </p>
        </field>
    </record>

    <record id="rating_rating_action_view_project_rating_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="rating_rating_action_view_project_rating"/>
        <field name="view_id" ref="rating_rating_project_view_kanban"/>
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
        <field name="name">Ratings</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,pivot,graph,form</field>
        <field name="domain">[('res_model', '=', 'project.task'), ('res_id', '=', active_id), ('consumed', '=', True)]</field>
        <field name="search_view_id" ref="rating_rating_view_search_project"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No customer ratings yet
            </p>
            <p>
                Let's wait for your customers to manifest themselves.
            </p>
        </field>
    </record>

    <record id="rating_rating_action_task_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="rating_rating_action_task"/>
        <field name="view_id" ref="rating_rating_project_view_kanban"/>
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
            </p>
            <p>
                Measure your customer satisfaction by sending rating requests when your tasks reach a certain stage.
            </p>
        </field>
        <field name="context">{
            'search_default_last_month': 1,
            'graph_groupbys': ['rated_partner_id'],
        }</field>
    </record>

    <record id="rating_rating_action_project_report_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="rating_rating_action_project_report"/>
        <field name="view_id" ref="rating_rating_project_view_kanban"/>
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
                            <div class="col-12 col-lg-6 o_setting_box" id="project_milestone">
                                <div class="o_setting_left_pane">
                                    <field name="group_project_milestone"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_project_milestone"/>
                                    <div class="text-muted">
                                        Track major progress points that must be reached to achieve success
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Time Management</h2>
                        <div class="row mt16 o_settings_container" name="project_time">
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
                            <div class="col-12 col-lg-6 o_setting_box" name="project_time_management">
                                <div class="o_setting_left_pane">
                                    <field name="module_project_forecast" widget="upgrade_boolean"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_project_forecast"/>
                                    <div class="text-muted" name="project_forecast_msg">
                                        Plan resource allocation across projects and estimate deadlines more accurately
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2 name="section_analytics">Analytics</h2>
                        <div class="row mt16 o_settings_container" name="analytic">
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
                                        <div class="mt16">
                                            <button name="%(project.open_task_type_form)d" context="{'project_id':id}" icon="fa-arrow-right" type="action" string="Set a Rating Email Template on Stages" class="btn-link"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box"
                                 id="default_plan_setting"
                                 groups="analytic.group_analytic_accounting"
                                 title="Track the profitability of your projects. Any project, its tasks and timesheets are linked to an analytic account and any analytic account belongs to a plan.">
                                <div class="o_setting_left_pane"/>
                                <div class="o_setting_right_pane">
                                    <label for="analytic_plan_id"/>
                                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                    <div class="text-muted">
                                        Assign each new project to this plan
                                    </div>
                                    <div class="content-group">
                                        <div class="mt16">
                                            <field name="analytic_plan_id"/>
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
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" type="action" name="%(project_task_action_from_partner)d"
                        groups="project.group_project_user"
                        context="{'search_default_partner_id': active_id, 'default_partner_id': active_id}" attrs="{'invisible': [('task_count', '=', 0)]}"
                        icon="fa-tasks">
                        <field  string="Tasks" name="task_count" widget="statinfo"/>
                    </button>
                </div>
            </field>
       </record>

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
                    <field name="partner_ids" widget="many2many_tags_email" placeholder="Add contacts to share the project..." nolabel="1" context="{'show_email': True}"/>
                </group>
                <group>
                    <field name="note" placeholder="Add a note" nolabel="1" colspan="2"/>
                </group>
                <footer>
                    <button string="Send" name="action_send_mail" attrs="{'invisible': [('access_warning', '!=', '')]}" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z" />
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
from ast import literal_eval


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

    def action_unarchive_task(self):
        inactive_tasks = self.env['project.task'].with_context(active_test=False).search(
            [('active', '=', False), ('stage_id', 'in', self.stage_ids.ids)])
        inactive_tasks.action_unarchive()

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
            action['domain'] = [('display_project_id', '=', project_id)]
            action['context'] = str({
                'pivot_row_groupby': ['user_ids'],
                'default_project_id': project_id,
            })
        elif self.env.context.get('stage_view'):
            action = self.env["ir.actions.actions"]._for_xml_id("project.open_task_type_form")
        else:
            action = self.env["ir.actions.actions"]._for_xml_id("project.action_view_all_task")

        context = action.get('context', '{}')
        context = context.replace('uid', str(self.env.uid))
        context = dict(literal_eval(context), active_test=True)
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

    <record id="view_project_task_type_unarchive_wizard" model="ir.ui.view">
        <field name="name">project.task.type.delete.wizard.form</field>
        <field name="model">project.task.type.delete.wizard</field>
        <field name="arch" type="xml">
            <form string="Delete Stage">
                <div>
                    <p>Would you like to unarchive all of the tasks contained in these stages as well?</p>
                </div>
                <footer>
                    <button string="Confirm" type="object" name="action_unarchive_task" class="btn btn-primary"/>
                    <button string="Discard" special="cancel"/>
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

from . import project_task_type_delete
from . import project_share_wizard

```

