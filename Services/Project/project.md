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

from odoo.tools.sql import create_index


def _check_exists_collaborators_for_project_sharing(env):
    """ Check if it exists at least a collaborator in a shared project

        If it is the case we need to active the portal rules added only for this feature.
    """
    collaborator = env['project.collaborator'].search([], limit=1)
    if collaborator:
        # Then we need to enable the access rights linked to project sharing for the portal user
        env['project.collaborator']._toggle_project_sharing_portal_rules(True)


def _project_post_init(env):
    _check_exists_collaborators_for_project_sharing(env)

    # Index to improve the performance of burndown chart.
    project_task_stage_field_id = env['ir.model.fields']._get_ids('project.task').get('stage_id')
    create_index(
        env.cr,
        'mail_tracking_value_mail_message_id_old_value_integer_task_stage',
        env['mail.tracking.value']._table,
        ['mail_message_id', 'old_value_integer'],
        where=f'field_id={project_task_stage_field_id}'
    )

def _project_uninstall_hook(env):
    """Since the m2m table for the project share wizard's `partner_ids` field is not dropped at uninstall, it is
    necessary to ensure it is emptied, else re-installing the module will fail due to foreign keys constraints."""
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
        'report/project_task_burndown_chart_report_views.xml',
        'views/account_analytic_account_views.xml',
        'views/digest_digest_views.xml',
        'views/rating_rating_views.xml',
        'views/project_update_views.xml',
        'views/project_update_templates.xml',
        'views/project_project_stage_views.xml',
        'wizard/project_share_wizard_views.xml',
        'views/project_collaborator_views.xml',
        'views/project_task_type_views.xml',
        'views/project_project_views.xml',
        'views/project_task_views.xml',
        'views/project_tags_views.xml',
        'views/project_milestone_views.xml',
        'views/res_partner_views.xml',
        'views/res_config_settings_views.xml',
        'views/mail_activity_plan_views.xml',
        'views/mail_activity_type_views.xml',
        'views/project_sharing_project_task_views.xml',
        'views/project_portal_project_project_templates.xml',
        'views/project_portal_project_task_templates.xml',
        'views/project_task_templates.xml',
        'views/project_sharing_project_task_templates.xml',
        'report/project_report_views.xml',
        'data/ir_cron_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_template_data.xml',
        'data/project_data.xml',
        'wizard/project_task_type_delete_views.xml',
        'wizard/project_project_stage_delete_views.xml',
        'views/project_menus.xml',
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
            'project/static/src/components/**/*',
            'project/static/src/views/**/*',
            'project/static/src/js/tours/project.js',
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

            'web/static/src/libs/fontawesome/css/font-awesome.css',
            'web/static/lib/odoo_ui_icons/*',
            'web/static/lib/select2/select2.css',
            'web/static/lib/select2-bootstrap-css/select2-bootstrap.css',
            'web/static/src/webclient/navbar/navbar.scss',
            'web/static/src/scss/animation.scss',
            'web/static/src/core/colorpicker/colorpicker.scss',
            'web/static/src/scss/mimetypes.scss',
            'web/static/src/scss/ui.scss',
            'web/static/src/legacy/scss/ui.scss',
            'web/static/src/views/fields/translation_dialog.scss',
            'web/static/src/scss/fontawesome_overridden.scss',

            'web/static/src/module_loader.js',
            'web/static/src/session.js',

            'web/static/lib/luxon/luxon.js',
            'web/static/lib/owl/owl.js',
            'web/static/lib/owl/odoo_module.js',
            'web/static/lib/jquery/jquery.js',
            'web/static/lib/popper/popper.js',
            'web/static/lib/bootstrap/js/dist/dom/data.js',
            'web/static/lib/bootstrap/js/dist/dom/event-handler.js',
            'web/static/lib/bootstrap/js/dist/dom/manipulator.js',
            'web/static/lib/bootstrap/js/dist/dom/selector-engine.js',
            'web/static/lib/bootstrap/js/dist/base-component.js',
            'web/static/lib/bootstrap/js/dist/alert.js',
            'web/static/lib/bootstrap/js/dist/button.js',
            'web/static/lib/bootstrap/js/dist/carousel.js',
            'web/static/lib/bootstrap/js/dist/collapse.js',
            'web/static/lib/bootstrap/js/dist/dropdown.js',
            'web/static/lib/bootstrap/js/dist/modal.js',
            'web/static/lib/bootstrap/js/dist/offcanvas.js',
            'web/static/lib/bootstrap/js/dist/tooltip.js',
            'web/static/lib/bootstrap/js/dist/popover.js',
            'web/static/lib/bootstrap/js/dist/scrollspy.js',
            'web/static/lib/bootstrap/js/dist/tab.js',
            'web/static/lib/bootstrap/js/dist/toast.js',
            'web/static/lib/select2/select2.js',
            'web/static/src/legacy/js/libs/bootstrap.js',
            'web/static/src/legacy/js/libs/jquery.js',
            ('include', 'web._assets_bootstrap_backend'),

            'base/static/src/css/modules.css',

            'web/static/src/core/utils/transitions.scss',
            'web/static/src/core/**/*',
            'web/static/src/model/**/*',
            'web/static/src/search/**/*',
            'web/static/src/webclient/icons.scss', # variables required in list_controller.scss
            'web/static/src/views/**/*.js',
            'web/static/src/views/*.xml',
            'web/static/src/views/*.scss',
            'web/static/src/views/fields/**/*',
            'web/static/src/views/form/**/*',
            'web/static/src/views/kanban/**/*',
            'web/static/src/views/list/**/*',
            'web/static/src/views/view_button/**/*',
            'web/static/src/views/view_components/**/*',
            'web/static/src/views/view_dialogs/**/*',
            'web/static/src/views/widgets/**/*',
            'web/static/src/webclient/**/*',
            ('remove', 'web/static/src/webclient/clickbot/clickbot.js'), # lazy loaded
            ('remove', 'web/static/src/views/form/button_box/*.scss'),
            ('remove', 'web/static/src/core/emoji_picker/emoji_data.js'),

            # remove the report code and whitelist only what's needed
            ('remove', 'web/static/src/webclient/actions/reports/**/*'),
            'web/static/src/webclient/actions/reports/*.js',
            'web/static/src/webclient/actions/reports/*.xml',

            'web/static/src/env.js',

            ('include', 'web_editor.assets_wysiwyg'),

            'web/static/src/legacy/scss/fields.scss',

            'base/static/src/scss/res_partner.scss',

            # Form style should be computed before
            'web/static/src/views/form/button_box/*.scss',

            'web_editor/static/src/js/editor/odoo-editor/src/base_style.scss',
            'web_editor/static/lib/vkbeautify/**/*',
            'web_editor/static/src/js/common/**/*',
            'web_editor/static/src/js/editor/odoo-editor/src/utils/utils.js',
            'web_editor/static/src/js/wysiwyg/fonts.js',

            'web_editor/static/src/components/**/*',
            'web_editor/static/src/scss/web_editor.common.scss',
            'web_editor/static/src/scss/web_editor.backend.scss',

            'web_editor/static/src/js/backend/**/*',
            'web_editor/static/src/xml/backend.xml',

            'mail/static/src/scss/variables/*.scss',
            'mail/static/src/views/web/form/form_renderer.scss',

            'project/static/src/components/project_task_name_with_subtask_count_char_field/*',
            'project/static/src/components/project_task_state_selection/*',
            'project/static/src/components/project_many2one_field/*',
            'project/static/src/views/project_task_form/*.scss',

            'project/static/src/project_sharing/search/favorite_menu/custom_favorite_item.xml',
            'project/static/src/project_sharing/**/*',
            'web/static/src/start.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from collections import OrderedDict
from operator import itemgetter
from markupsafe import Markup

from odoo import conf, http, _
from odoo.exceptions import AccessError, MissingError, UserError
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
        values = self._prepare_tasks_values(page, date_begin, date_end, sortby, search, search_in, groupby, url, domain, su=bool(access_token), project=project)
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
            preview_object=project,
        )

        if not groupby:
            values['groupby'] = 'project' if self._display_project_groupby(project) else 'none'

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
        if not groupby:
            groupby = 'stage'
        values = self._project_get_page_view_values(project_sudo, access_token, page, date_begin, date_end, sortby, search, search_in, groupby, **kw)
        return request.render("project.portal_my_project", values)

    def _get_project_sharing_company(self, project):
        return project.company_id or request.env.user.company_id

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

        project_company = self._get_project_sharing_company(project)

        session_info.update(
            cache_hashes=cache_hashes,
            action_name=project.action_project_sharing(),
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
            'preview_object': task,
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

    def _task_get_searchbar_sortings(self, milestones_allowed, project=False):
        values = {
            'date': {'label': _('Newest'), 'order': 'create_date desc', 'sequence': 1},
            'name': {'label': _('Title'), 'order': 'name', 'sequence': 2},
            'stage': {'label': _('Stage'), 'order': 'stage_id, project_id', 'sequence': 5},
            'status': {'label': _('Status'), 'order': 'state', 'sequence': 6},
            'priority': {'label': _('Priority'), 'order': 'priority desc', 'sequence': 8},
            'date_deadline': {'label': _('Deadline'), 'order': 'date_deadline asc', 'sequence': 9},
            'update': {'label': _('Last Stage Update'), 'order': 'date_last_stage_update desc', 'sequence': 11},
        }
        if not project:
            values['project'] = {'label': _('Project'), 'order': 'project_id, stage_id', 'sequence': 3}
        if milestones_allowed:
            values['milestone'] = {'label': _('Milestone'), 'order': 'milestone_id', 'sequence': 7}
        return values

    # Meant to be overridden in documents_project
    def _display_project_groupby(self, project):
        return not project

    def _task_get_searchbar_groupby(self, milestones_allowed, project=False):
        values = {
            'none': {'input': 'none', 'label': _('None'), 'order': 1},
            'stage': {'input': 'stage', 'label': _('Stage'), 'order': 4},
            'status': {'input': 'status', 'label': _('Status'), 'order': 5},
            'priority': {'input': 'priority', 'label': _('Priority'), 'order': 7},
            'customer': {'input': 'customer', 'label': _('Customer'), 'order': 10},
        }
        if self._display_project_groupby(project):
            values['project'] = {'input': 'project', 'label': _('Project'), 'order': 2}
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
            'status': 'state',
        }

    def _task_get_order(self, order, groupby):
        groupby_mapping = self._task_get_groupby_mapping()
        field_name = groupby_mapping.get(groupby, '')
        if not field_name:
            return order
        return '%s, %s' % (field_name, order)

    def _task_get_searchbar_inputs(self, milestones_allowed, project=False):
        values = {
            'all': {'input': 'all', 'label': _('Search in All'), 'order': 1},
            'content': {'input': 'content', 'label': Markup(_('Search <span class="nolabel"> (in Content)</span>')), 'order': 1},
            'ref': {'input': 'ref', 'label': _('Search in Ref'), 'order': 1},
            'users': {'input': 'users', 'label': _('Search in Assignees'), 'order': 3},
            'stage': {'input': 'stage', 'label': _('Search in Stages'), 'order': 4},
            'status': {'input': 'status', 'label': _('Search in Status'), 'order': 5},
            'priority': {'input': 'priority', 'label': _('Search in Priority'), 'order': 7},
            'customer': {'input': 'customer', 'label': _('Search in Customer'), 'order': 10},
            'message': {'input': 'message', 'label': _('Search in Messages'), 'order': 11},
        }
        if not project:
            values['project'] = {'input': 'project', 'label': _('Search in Project'), 'order': 2}
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
            state_dict = dict(map(reversed, request.env['project.task']._fields['state']._description_selection(request.env)))
            search_domain.append([('state', 'ilike', state_dict.get(search, search))])
        return OR(search_domain)

    def _prepare_tasks_values(self, page, date_begin, date_end, sortby, search, search_in, groupby, url="/my/tasks", domain=None, su=False, project=False):
        values = self._prepare_portal_layout_values()

        Task = request.env['project.task']
        milestone_domain = AND([domain, [('allow_milestones', '=', 'True')]])
        milestones_allowed = Task.sudo().search_count(milestone_domain, limit=1) == 1
        searchbar_sortings = dict(sorted(self._task_get_searchbar_sortings(milestones_allowed, project).items(),
                                         key=lambda item: item[1]["sequence"]))
        searchbar_inputs = self._task_get_searchbar_inputs(milestones_allowed, project)
        searchbar_groupby = self._task_get_searchbar_groupby(milestones_allowed, project)

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


            task_states = dict(Task_sudo._fields['state']._description_selection(request.env))
            if sortby == 'status':
                if groupby == 'none' and grouped_tasks:
                    grouped_tasks[0] = grouped_tasks[0].sorted(lambda tasks: task_states.get(tasks.state))
                else:
                    grouped_tasks.sort(key=lambda tasks: task_states.get(tasks[0].state))
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
        project_groups = request.env['project.task']._read_group(AND([[('project_id', 'not in', projects.ids)], task_domain or []]),
                                                                ['project_id'])
        for [project] in project_groups:
            proj_name = project.sudo().display_name if project else _('Others')
            searchbar_filters.update({
                str(project.id): {'label': proj_name, 'domain': [('project_id', '=', project.id)]}
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

    @http.route('/project_sharing/attachment/add_image', type='http', auth='user', methods=['POST'], website=True)
    def add_image(self, name, data, res_id, access_token=None, **kwargs):
        try:
            task_sudo = self._document_check_access('project.task', int(res_id), access_token=access_token)
            if not task_sudo.with_user(request.env.uid).project_id._check_project_sharing_access():
                return request.not_found()
        except (AccessError, MissingError):
            raise UserError(_("The document does not exist or you do not have the rights to access it."))

        IrAttachment = request.env['ir.attachment']

        # Avoid using sudo when not necessary: internal users can create attachments,
        # as opposed to public and portal users.
        if not request.env.user._is_internal():
            IrAttachment = IrAttachment.sudo()

        values = IrAttachment._check_contents({
            'name': name,
            'datas': data,
            'res_model': 'project.task',
            'res_id': res_id,
            'access_token': IrAttachment._generate_access_token(),
        })

        valid_image_mime_types = ['image/jpeg', 'image/png', 'image/bmp', 'image/tiff']

        if values.get('mimetype', False) not in valid_image_mime_types:
            return request.make_response(
                data=json.dumps({'error': _('Only jpeg, png, bmp and tiff images are allowed as attachments.')}),
                headers=[('Content-Type', 'application/json')],
                status=400
            )

        attachment = IrAttachment.create(values)
        return request.make_response(
            data=json.dumps(attachment.read(['id', 'name', 'mimetype', 'file_size', 'access_token'])[0]),
            headers=[('Content-Type', 'application/json')]
        )

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
            task = request.env['project.task'].sudo().with_context(active_test=False).search([('id', '=', res_id), ('project_id', '=', project_sudo.id)])
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
            <field name="name">Tip: Use task states to keep track of your tasks' progression</field>
            <field name="sequence">1200</field>
            <field name="group_id" ref="project.group_project_user"/>
            <field name="tip_description" type="html">
            <div>
                <p class="tip_title">Tip: Use task states to keep track of your tasks' progression</p>
                <p class="tip_content">
                Quickly check the status of tasks for approvals or change requests and identify those on hold until dependencies are resolved with the hourglass icon.
                </p>
                <img src="https://download.odoocdn.com/digests/project/static/src/img/task-state-img.png" width="720" class="illustration_border" />
            </div>
            </field>
        </record>

        <record id="digest_tip_project_1" model="digest.tip">
            <field name="name">Tip: Create tasks from incoming emails</field>
            <field name="sequence">1300</field>
            <field name="group_id" ref="project.group_project_user"/>
            <field name="tip_description" type="html">
<div>
    <t t-set="project_record" t-value="object.env['project.project'].search([('alias_name', '!=', False), ('alias_domain_id', '!=', False)], limit=1, order='sequence asc')"/>
    <p class="tip_title">Tip: Create tasks from incoming emails</p>
    <t t-if="project_record.alias_email">
        <p class="tip_content">Emails sent to <a t-attf-href="mailto:{{project_record.alias_email}}" target="_blank" style="color: #714B67; text-decoration: none;"><t t-out="project_record.alias_email" /></a> will generate tasks in your <t t-out="project_record.name"></t> project.</p>
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
    <!-- new state subtypes-->
    <record id="mt_task_in_progress" model="mail.message.subtype">
        <field name="name">Task In Progress</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="sequence" eval="101"/>
        <field name="description">Task In Progress</field>
    </record>
    <record id="mt_task_changes_requested" model="mail.message.subtype">
        <field name="name">Changes Requested</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="sequence" eval="102"/>
        <field name="description">Changes Requested</field>
    </record>
    <record id="mt_task_approved" model="mail.message.subtype">
        <field name="name">Task Approved</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="sequence" eval="103"/>
        <field name="description">Task approved</field>
    </record>
    <record id="mt_task_canceled" model="mail.message.subtype">
        <field name="name">Task Canceled</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="sequence" eval="104"/>
        <field name="description">Task canceled</field>
    </record>
    <record id="mt_task_done" model="mail.message.subtype">
        <field name="name">Task Done</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="sequence" eval="105"/>
        <field name="description">Task done</field>
    </record>
    <record id="mt_task_waiting" model="mail.message.subtype">
        <field name="name">Task Waiting</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="sequence" eval="106"/>
        <field name="description">Task Waiting</field>
        <field name="hidden" eval="True"/>
    </record>
    <record id="mt_task_rating" model="mail.message.subtype">
        <field name="name">Task Rating</field>
        <field name="res_model">project.task</field>
        <field name="sequence" eval="108"/>
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
    <record id="mt_project_task_stage" model="mail.message.subtype">
        <field name="name">Task Stage Changed</field>
        <field name="sequence">16</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_stage"/>
        <field name="relation_field">project_id</field>
    </record>
    <record id="mt_project_task_rating" model="mail.message.subtype">
        <field name="name">Task Rating</field>
        <field name="sequence">27</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_rating"/>
        <field name="relation_field">project_id</field>
        <field name="hidden" eval="True"/>
    </record>
    <record id="mt_project_update_create" model="mail.message.subtype">
        <field name="name">Update Created</field>
        <field name="sequence">19</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_update_create"/>
        <field name="relation_field">project_id</field>
        <field name="hidden" eval="True"/>
    </record>
    <record id="mt_project_task_in_progress" model="mail.message.subtype">
        <field name="name">Task In Progress</field>
        <field name="sequence">20</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_in_progress"/>
        <field name="relation_field">project_id</field>
    </record>
    <record id="mt_project_task_changes_requested" model="mail.message.subtype">
        <field name="name">Changes Requested</field>
        <field name="sequence">21</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_changes_requested"/>
        <field name="relation_field">project_id</field>
    </record>
    <record id="mt_project_task_approved" model="mail.message.subtype">
        <field name="name">Task Approved</field>
        <field name="sequence">22</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_approved"/>
        <field name="relation_field">project_id</field>
    </record>
    <record id="mt_project_task_canceled" model="mail.message.subtype">
        <field name="name">Task Canceled</field>
        <field name="sequence">23</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_canceled"/>
        <field name="relation_field">project_id</field>
    </record>
    <record id="mt_project_task_done" model="mail.message.subtype">
        <field name="name">Task Done</field>
        <field name="sequence">24</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_done"/>
        <field name="relation_field">project_id</field>
    </record>
    <record id="mt_project_task_waiting" model="mail.message.subtype">
        <field name="name">Task Waiting</field>
        <field name="sequence">25</field>
        <field name="res_model">project.project</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_task_waiting"/>
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
            <field name="active" eval="False"/>
            <field name="subject">{{ object.project_id.company_id.name or user.env.company.name }}: Satisfaction Survey</field>
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
            Please take a moment to rate our services related to the <strong t-out="object.name or ''">Planning and budget</strong> task
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
                    <strong>Tell us how you feel about our services</strong><br/>
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
            We appreciate your feedback. It helps us improve continuously.
            <t t-if="object.project_id.rating_status == 'stage'">
                <br/><span style="margin: 0; font-size: 12px; opacity: 0.5; color: #454748;">This satisfaction survey has been sent because your task has been moved to the <b t-out="object.stage_id.name or ''">In progress</b> stage</span>
            </t>
            <t t-if="object.project_id.rating_status == 'periodic'">
                <br/><span style="margin: 0; font-size: 12px; opacity: 0.5; color: #454748;">This satisfaction survey is sent <b t-out="object.project_id.rating_status_period or ''">weekly</b> as long as the task is in the <b t-out="object.stage_id.name or ''">In progress</b> stage.</span>
            </t>
        </td></tr>
        <tr><td><br/>Best regards,</td></tr>
        <tr><td>
           <t t-out="object.project_id.company_id.name or ''">YourCompany</t>
        </td></tr>
        <tr><td style="opacity: 0.5;">
            <t t-out="object.project_id.company_id.phone or ''">1 650-123-4567</t>
            <t t-if="object.project_id.company_id.email">
                | <a t-attf-href="mailto:{{ object.project_id.company_id.email }}" style="text-decoration:none; color: #454748;" t-out="object.project_id.company_id.email or ''">info@yourcompany.com</a>
            </t>
            <t t-if="object.project_id.company_id.website">
                | <a t-attf-href="{{ object.project_id.company_id.website }}" style="text-decoration:none; color: #454748;" t-out="object.project_id.company_id.website or ''">http://www.example.com</a>
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
<div>
    Dear <t t-esc="assignee_name"/>,
    <br/><br/>
    <span style="margin-top: 8px;">You have been assigned to the <t t-esc="model_description or 'document'"/> <t t-esc="object.display_name"/>.</span>
</div>
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
            <field name="groups_id" eval="[(3, ref('project.group_project_manager'))]"/>
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
            <field name="company_id" eval="False"/>
        </record>

        <record id="analytic_research_development" model="account.analytic.account">
            <field name="name">Research &amp; Development</field>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
            <field name="company_id" eval="False"/>
        </record>

        <record id="analytic_renovations" model="account.analytic.account">
            <field name="name">Renovations</field>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
            <field name="company_id" eval="False"/>
        </record>

        <!-- Stage templates -->
        <record id="project.project_project_stage_2" model="project.project.stage">
            <field name="mail_template_id" ref="project.project_done_email_template"/>
        </record>

        <!-- Task Stages -->
        <record id="project_stage_0" model="project.task.type">
            <field name="sequence">1</field>
            <field name="name">New</field>
            <field name="mail_template_id" ref="project.mail_template_data_project_task"/>
        </record>
        <record id="project_stage_1" model="project.task.type">
            <field name="sequence">10</field>
            <field name="name">In Progress</field>
        </record>
        <record id="project_stage_2" model="project.task.type">
            <field name="sequence">20</field>
            <field name="name">Done</field>
            <field name="fold" eval="True"/>
        </record>
        <record id="project_stage_3" model="project.task.type">
            <field name="sequence">30</field>
            <field name="name">Canceled</field>
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

        <record id="project_project_4" model="project.project">
            <field name="date_start" eval="(DateTime.today() + relativedelta(months=-1)).strftime('%Y-%m-%d 10:00:00')"/>
            <field name="date" eval="(DateTime.today() + relativedelta(days=-5)).strftime('%Y-%m-%d 17:00:00')"/>
            <field name="name">Home Make Over</field>
            <field name="color">4</field>
            <field name="active">False</field>
            <field name="description">Interior designing and refurnishing.</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="type_ids" eval="[
                Command.link(ref('project_stage_0')),
                Command.link(ref('project_stage_1')),
                Command.link(ref('project_stage_2')),
                Command.link(ref('project_stage_3')),
            ]"/>
            <field name="favorite_user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="tag_ids" eval="[Command.link(ref('project_tags_04')), Command.link(ref('project_tags_02'))]"/>
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
            <field name="allocated_hours">20.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Office planning</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">7</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="milestone_id" ref="project.project_1_milestone_1" />
        </record>

        <record id="project_1_task_2" model="project.task">
            <field name="allocated_hours" eval="32.0"/>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_2_mail_message_1"/>
        </record>
        <record id="project_1_task_2_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_1_task_2_mail_message_2"/>
        </record>

        <record id="project_1_task_3" model="project.task">
            <field name="allocated_hours" eval="10.0"/>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_3_mail_message_1"/>
        </record>
        <record id="project_1_task_3_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_1_task_3_mail_message_2"/>
        </record>

        <record id="project_1_task_4" model="project.task">
            <field name="sequence">17</field>
            <field name="allocated_hours">8.0</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_4_mail_message_1"/>
        </record>
        <record id="project_1_task_4_mail_message_2_track_1" model="mail.tracking.value">
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_1_task_4_mail_message_2"/>
        </record>

        <record id="project_1_task_5" model="project.task">
            <field name="allocated_hours" eval="15.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Energy Certificate</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="state">01_in_progress</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
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
            <field name="allocated_hours" eval="76.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Room 1: Decoration</field>
            <field name="state">03_approved</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
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
            <field name="allocated_hours" eval="24.0"/>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_1_task_7_mail_message_1"/>
        </record>

        <record id="project_1_task_8" model="project.task">
            <field name="allocated_hours" eval="60.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Black Chairs for managers</field>
            <field name="description">Use the account_budget module</field>
            <field name="date_deadline" eval="time.strftime('%Y-%m-19')"/>
            <field name="color">5</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="state">02_changes_requested</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
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
            <field name="allocated_hours" eval="40.0"/>
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

        <!-- Archive Tasks -->
        <record id="project_1_task_9_archive_1" model="project.task">
            <field name="name">Kitchen Assembly</field>
            <field name="active">False</field>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id" ref="project_project_4"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
        <record id="project_task_furniture" model="project.task">
            <field name="name">Furniture Delivery</field>
            <field name="active">False</field>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id" ref="project_project_4"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>
         <record id="project_task_ceiling" model="project.task">
            <field name="name">Ceiling fan</field>
            <field name="active">False</field>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id" ref="project_project_4"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
        </record>

        <!-- Project 1 Recurring tasks and subtasks -->
        <record id="project_task_recurrence_1" model="project.task.recurrence">
            <field name="repeat_unit">month</field>
            <field name="repeat_type">until</field>
            <field name="repeat_until" eval="DateTime.now() + relativedelta(months=4)"/>
            <field name="create_date" eval="DateTime.now() + relativedelta(weeks=-2)"/>
        </record>
        <record id="project_task_recurrence_2" model="project.task.recurrence">
            <field name="repeat_unit">week</field>
            <field name="repeat_type">forever</field>
        </record>
        <record id="project_1_task_10" model="project.task">
            <field name="sequence">20</field>
            <field name="allocated_hours">20.0</field>
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
            <field name="allocated_hours">0.25</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Daily stand-up meeting - Send minutes</field>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="recurring_task" eval="True"/>
            <field name="recurrence_id" ref="project_task_recurrence_2"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(weeks=-1)"/>
        </record>
        <record id="project_1_task_12" model="project.task">
            <field name="sequence">20</field>
            <field name="allocated_hours">8.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_10"/>
            <field name="name">Customer Meeting</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_1_task_13" model="project.task">
            <field name="sequence">10</field>
            <field name="allocated_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_12"/>
            <field name="name">Daily Meetings summary</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_1_task_14" model="project.task">
            <field name="sequence">20</field>
            <field name="allocated_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_12"/>
            <field name="name">Preparation</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_1_task_15" model="project.task">
            <field name="sequence">30</field>
            <field name="allocated_hours">2.0</field>
            <field name="user_ids" eval="False"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_12"/>
            <field name="name">Minutes</field>
            <field name="stage_id" ref="project_stage_1"/>
        </record>
        <record id="project_1_task_16" model="project.task">
            <field name="allocated_hours" eval="24.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="name">Chair Cabinet</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_1_task_2'))]"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=6)"/>
            <field name="color">9</field>
        </record>
        <record id="project_1_task_17" model="project.task">
            <field name="name">Plywood requirement</field>
            <field name="sequence">40</field>
            <field name="allocated_hours">2.0</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="parent_id" ref="project.project_1_task_16"/>
            <field name="stage_id" ref="project_stage_2"/>
        </record>

        <!-- Project 2 Tasks-->
        <record id="project_2_task_1" model="project.task">
            <field name="allocated_hours">12.0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Customer analysis + Architecture</field>
            <field name="state">1_done</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_2_task_1_mail_message_2"/>
        </record>

        <record id="project_2_task_2" model="project.task">
            <field name="allocated_hours">24.0</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Basic outline</field>
            <field name="state">1_done</field>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_1'))]"/>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_02')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_2_task_2_mail_message_2"/>
        </record>

        <record id="project_2_task_3" model="project.task">
            <field name="allocated_hours" eval="40.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Planning and budget</field>
            <field name="state">1_done</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">6</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
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
            <field name="allocated_hours" eval="16.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">User interface improvements</field>
            <field name="tag_ids" eval="[Command.set([
                    ref('project.project_tags_01'),
                    ref('project.project_tags_03')])]"/>
            <field name="stage_id" ref="project_stage_1"/>
            <field name="state">03_approved</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_4_mail_message_1"/>
        </record>

        <record id="project_2_task_5" model="project.task">
            <field name="allocated_hours" eval="38.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Social network integration</field>
            <field name="description">Facebook and Twitter integration</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_5_mail_message_1"/>
        </record>

        <record id="project_2_task_6" model="project.task">
            <field name="allocated_hours">42.0</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_6_mail_message_1"/>
        </record>

        <record id="project_2_task_7" model="project.task">
            <field name="allocated_hours" eval="22.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">New portal system</field>
            <field name="priority">0</field>
            <field name="state">1_done</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="tag_ids" eval="[Command.set([ref('project.project_tags_02')])]"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_3')), Command.link(ref('project_2_task_2'))]"/>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">In Progress</field>
            <field name="new_value_char">Done</field>
            <field name="old_value_integer">2</field>
            <field name="new_value_integer">3</field>
            <field name="mail_message_id" ref="project_2_task_7_mail_message_2"/>
        </record>

        <record id="project_2_task_8" model="project.task">
            <field name="allocated_hours">14.0</field>
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
            <field name="field_id" model="ir.model.fields" eval="obj().search([('model', '=', 'project.task'), ('name', '=', 'stage_id')])"/>
            <field name="old_value_char">New</field>
            <field name="new_value_char">In Progress</field>
            <field name="old_value_integer">1</field>
            <field name="new_value_integer">2</field>
            <field name="mail_message_id" ref="project_2_task_8_mail_message_1"/>
        </record>

        <record id="project_2_task_9" model="project.task">
            <field name="allocated_hours" eval="18.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_2"/>
            <field name="name">Document management</field>
            <field name="stage_id" ref="project_stage_0"/>
            <field name="state">1_canceled</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
            <field name="depend_on_ids" eval="[Command.link(ref('project.project_2_task_8'))]"/>
            <field name="date_deadline" eval="DateTime.now() + relativedelta(days=15)"/>
            <field name="color">4</field>
        </record>

        <record id="project_2_task_10" model="project.task">
            <field name="sequence">20</field>
            <field name="allocated_hours">35.0</field>
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
            <field name="allocated_hours" eval="20.0"/>
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
            <field name="allocated_hours" eval="40.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Entry Hall</field>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="color">3</field>
        </record>

        <record id="project_3_task_2" model="project.task">
            <field name="allocated_hours" eval="10.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">0</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Check Lift</field>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
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
            <field name="allocated_hours" eval="24.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Room 1: Paint</field>
            <field name="description">Repaint the walls with the hex color #0FF1CE</field>
            <field name="state">1_done</field>
            <field name="priority">0</field>
            <field name="date_deadline" eval="DateTime.today() - relativedelta(days=5)"/>
            <field name="stage_id" ref="project_stage_2"/>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_01')])]"/>
            <field name="color">9</field>
        </record>

        <record id="project_3_task_4" model="project.task">
            <field name="allocated_hours" eval="76.0"/>
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="priority">1</field>
            <field name="project_id" ref="project.project_project_3"/>
            <field name="name">Bathroom</field>
            <field name="stage_id" ref="project_stage_2"/>
        </record>

        <record id="project_3_task_5" model="project.task">
            <field name="allocated_hours" eval="40.0"/>
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

        <!-- Change task creation notifications date -->
        <function model="mail.message" name="write">
            <value model="mail.message"
                eval="obj().env['mail.message'].search([
                        ('subtype_id', '=', ref('project.mt_task_new')),
                        ('res_id', 'in', [
                            ref('project.project_1_task_1'),
                            ref('project.project_1_task_2'),
                            ref('project.project_1_task_3'),
                            ref('project.project_1_task_4'),
                            ref('project.project_1_task_5'),
                            ref('project.project_1_task_6'),
                            ref('project.project_1_task_7'),
                            ref('project.project_1_task_8'),
                            ref('project.project_1_task_9'),
                            ref('project.project_2_task_1'),
                            ref('project.project_2_task_2'),
                            ref('project.project_2_task_3'),
                            ref('project.project_2_task_4'),
                            ref('project.project_2_task_5'),
                            ref('project.project_2_task_6'),
                            ref('project.project_2_task_7'),
                            ref('project.project_2_task_8'),
                            ref('project.project_3_task_2'),
                        ]),
                    ]).ids"
            />
            <value eval="{'date': DateTime.now() - relativedelta(months=5)}"/>
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
        <record id="project_update_3" model="project.update" context="{'default_project_id': ref('project.project_project_3')}">
            <field name="name">Status</field>
            <field name="user_id" eval="ref('base.user_admin')"/>
            <field name="progress" eval="100"/>
            <field name="status">done</field>
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

## File: models\account_analytic_account.py

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
        project_data = self.env['project.project']._read_group([('analytic_account_id', 'in', self.ids)], ['analytic_account_id'], ['__count'])
        mapping = {analytic_account.id: count for analytic_account, count in project_data}
        for account in self:
            account.project_count = mapping.get(account.id, 0)

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

## File: models\digest_digest.py

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

        self._calculate_company_based_kpi(
            'project.task',
            'kpi_project_task_opened_value',
            additional_domain=[('stage_id.fold', '=', False), ('project_id', '!=', False)],
        )

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

    @api.depends('project_id', 'partner_id')
    def _compute_display_name(self):
        for collaborator in self:
            collaborator.display_name = f'{collaborator.project_id.display_name} - {collaborator.partner_id.display_name}'

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

from .project_task import CLOSED_STATES

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
    done_task_count = fields.Integer('# of Done Tasks', compute='_compute_task_count', groups='project.group_project_milestone')
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
        all_and_done_task_count_per_milestone = {
            milestone.id: (count, sum(state in CLOSED_STATES for state in state_list))
            for milestone, count, state_list in self.env['project.task']._read_group(
                [('milestone_id', 'in', self.ids), ('allow_milestones', '=', True)],
                ['milestone_id'], ['__count', 'state:array_agg'],
            )
        }
        for milestone in self:
            milestone.task_count, milestone.done_task_count = all_and_done_task_count_per_milestone.get(milestone.id, (0, 0))

    def _compute_can_be_marked_as_done(self):
        if not any(self._ids):
            for milestone in self:
                milestone.can_be_marked_as_done = not milestone.is_reached and all(milestone.task_ids.mapped(lambda t: t.state in CLOSED_STATES))
            return

        unreached_milestones = self.filtered(lambda milestone: not milestone.is_reached)
        (self - unreached_milestones).can_be_marked_as_done = False
        task_read_group = self.env['project.task']._read_group(
            [('milestone_id', 'in', unreached_milestones.ids)],
            ['milestone_id', 'state'],
            ['__count'],
        )
        task_count_per_milestones = defaultdict(lambda: (0, 0))
        for milestone, state, count in task_read_group:
            opened_task_count, closed_task_count = task_count_per_milestones[milestone.id]
            if state in CLOSED_STATES:
                closed_task_count += count
            else:
                opened_task_count += count
            task_count_per_milestones[milestone.id] = opened_task_count, closed_task_count
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

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import json
from collections import defaultdict
from datetime import timedelta

from odoo import api, Command, fields, models, _, _lt
from odoo.addons.rating.models import rating_data
from odoo.exceptions import UserError
from odoo.tools import get_lang, SQL
from .project_update import STATUS_COLOR
from .project_task import CLOSED_STATES


class Project(models.Model):
    _name = "project.project"
    _description = "Project"
    _inherit = ['portal.mixin', 'mail.alias.mixin', 'rating.parent.mixin', 'mail.thread', 'mail.activity.mixin']
    _order = "sequence, name, id"
    _rating_satisfaction_days = 30  # takes 30 days by default
    _systray_view = 'activity'

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
        project_and_state_counts = self.env['project.task'].with_context(
            active_test=any(project.active for project in self)
        )._read_group(
            [('project_id', 'in', self.ids), ('display_in_project', '=', True)],
            ['project_id', 'state'],
            ['__count'],
        )
        task_counts_per_project_id = defaultdict(lambda: {
            'open_task_count': 0,
            'closed_task_count': 0,
        })
        for project, state, count in project_and_state_counts:
            task_counts_per_project_id[project.id]['closed_task_count' if state in CLOSED_STATES else 'open_task_count'] += count
        for project in self:
            open_task_count, closed_task_count = task_counts_per_project_id[project.id].values()
            project.open_task_count = open_task_count
            project.closed_task_count = closed_task_count
            project.task_count = open_task_count + closed_task_count

    def _default_stage_id(self):
        # Since project stages are order by sequence first, this should fetch the one with the lowest sequence number.
        return self.env['project.project.stage'].search([], limit=1)

    @api.model
    def _search_is_favorite(self, operator, value):
        if operator not in ['=', '!='] or not isinstance(value, bool):
            raise NotImplementedError(_('Operation not supported'))
        return [('favorite_user_ids', 'in' if (operator == '=') == value else 'not in', self.env.uid)]

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

    name = fields.Char("Name", index='trigram', required=True, tracking=True, translate=True, default_export_compatible=True)
    description = fields.Html(help="Description to provide more information and context about this project")
    active = fields.Boolean(default=True,
        help="If the active field is set to False, it will allow you to hide the project without removing it.")
    sequence = fields.Integer(default=10)
    partner_id = fields.Many2one('res.partner', string='Customer', auto_join=True, tracking=True, domain="['|', ('company_id', '=?', company_id), ('company_id', '=', False)]")
    company_id = fields.Many2one('res.company', string='Company', compute="_compute_company_id", inverse="_inverse_company_id", store=True, readonly=False)
    currency_id = fields.Many2one('res.currency', compute="_compute_currency_id", string="Currency", readonly=True)
    analytic_account_id = fields.Many2one('account.analytic.account', string="Analytic Account", copy=False, ondelete='set null',
        domain="['|', ('company_id', '=', False), ('company_id', '=?', company_id)]", check_company=True,
        help="Analytic account to which this project, its tasks and its timesheets are linked. \n"
            "Track the costs and revenues of your project by setting this analytic account on your related documents (e.g. sales orders, invoices, purchase orders, vendor bills, expenses etc.).\n"
            "This analytic account can be changed on each task individually if necessary.\n"
            "An analytic account is required in order to use timesheets.")
    analytic_account_balance = fields.Monetary(related="analytic_account_id.balance")

    favorite_user_ids = fields.Many2many(
        'res.users', 'project_favorite_user_rel', 'project_id', 'user_id',
        default=_get_default_favorite_user_ids,
        string='Members')
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite', search='_search_is_favorite',
        compute_sudo=True, string='Show Project on Dashboard')
    label_tasks = fields.Char(string='Use Tasks as', default=lambda s: _('Tasks'), translate=True,
        help="Name used to refer to the tasks of your project e.g. tasks, tickets, sprints, etc...")
    tasks = fields.One2many('project.task', 'project_id', string="Task Activities")
    resource_calendar_id = fields.Many2one(
        'resource.calendar', string='Working Time', compute='_compute_resource_calendar_id')
    type_ids = fields.Many2many('project.task.type', 'project_task_type_rel', 'project_id', 'type_id', string='Tasks Stages')
    task_count = fields.Integer(compute='_compute_task_count', string="Task Count")
    open_task_count = fields.Integer(compute='_compute_task_count', string="Open Task Count")
    # [XBO] TODO: remove me in master
    closed_task_count = fields.Integer(compute='_compute_task_count', string="Closed Task Count")
    task_ids = fields.One2many('project.task', 'project_id', string='Tasks',
                               domain=lambda self: [('state', 'in', self.env['project.task'].OPEN_STATES)])
    color = fields.Integer(string='Color Index')
    user_id = fields.Many2one('res.users', string='Project Manager', default=lambda self: self.env.user, tracking=True)
    alias_id = fields.Many2one(help="Internal email associated with this project. Incoming emails are automatically synchronized "
                                    "with Tasks (or optionally Issues if the Issue Tracker module is installed).")
    privacy_visibility = fields.Selection([
            ('followers', 'Invited internal users (private)'),
            ('employees', 'All internal users'),
            ('portal', 'Invited portal users and all internal users (public)'),
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
    allow_task_dependencies = fields.Boolean('Task Dependencies', default=lambda self: self.env.user.has_group('project.group_project_task_dependencies'), inverse='_inverse_allow_task_dependencies')
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
        [('stage', 'when reaching a given stage'),
         ('periodic', 'on a periodic basis')
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
        ('done', 'Done'),
    ], default='to_define', compute='_compute_last_update_status', store=True, readonly=False, required=True)
    last_update_color = fields.Integer(compute='_compute_last_update_color')
    milestone_ids = fields.One2many('project.milestone', 'project_id')
    milestone_count = fields.Integer(compute='_compute_milestone_count', groups='project.group_project_milestone')
    milestone_count_reached = fields.Integer(compute='_compute_milestone_reached_count', groups='project.group_project_milestone')
    is_milestone_exceeded = fields.Boolean(compute="_compute_is_milestone_exceeded", search='_search_is_milestone_exceeded')

    _sql_constraints = [
        ('project_date_greater', 'check(date >= date_start)', "The project's start date must be before its end date.")
    ]

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if (self.env.user.has_group('project.group_project_stages') and self.stage_id.company_id
                and self.stage_id.company_id != self.company_id):
            self.stage_id = self.env['project.project.stage'].search(
                [('company_id', 'in', [self.company_id.id, False])],
                order=f"sequence asc, {self.env['project.project.stage']._order}",
                limit=1,
            ).id

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

    @api.depends('analytic_account_id.company_id', 'partner_id.company_id')
    def _compute_company_id(self):
        for project in self:
            # if a new restriction is put on the account or the customer, the restriction on the project is updated.
            if project.analytic_account_id.company_id:
                project.company_id = project.analytic_account_id.company_id
            if not project.company_id and project.partner_id.company_id:
                project.company_id = project.partner_id.company_id

    @api.depends_context('company')
    @api.depends('company_id', 'company_id.resource_calendar_id')
    def _compute_resource_calendar_id(self):
        for project in self:
            project.resource_calendar_id = project.company_id.resource_calendar_id or self.env.company.resource_calendar_id

    def _inverse_company_id(self):
        """
        Ensures that the new company of the project is valid for the account. If not set back the previous company, and raise a user Error.
        Ensures that the new company of the project is valid for the partner
        """
        for project in self:
            account = project.analytic_account_id
            if project.partner_id and project.partner_id.company_id and project.company_id != project.partner_id.company_id:
                raise UserError(_('The project and the associated partner must be linked to the same company.'))
            if not account or not account.company_id:
                continue
            # if the account of the project has more than one company linked to it, or if it has aal, do not update the account, and set back the old company on the project.
            if (account.project_count > 1 or account.line_ids) and project.company_id != account.company_id:
                raise UserError(
                    _("The project's company cannot be changed if its analytic account has analytic lines or if more than one project is linked to it."))
            account.company_id = project.company_id

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
        read_group = self.env['project.milestone']._read_group([('project_id', 'in', self.ids)], ['project_id'], ['__count'])
        mapped_count = {project.id: count for project, count in read_group}
        for project in self:
            project.milestone_count = mapped_count.get(project.id, 0)

    @api.depends('milestone_ids.is_reached')
    def _compute_milestone_reached_count(self):
        read_group = self.env['project.milestone']._read_group(
            [('project_id', 'in', self.ids), ('is_reached', '=', True)],
            ['project_id'],
            ['__count'],
        )
        mapped_count = {project.id: count for project, count in read_group}
        for project in self:
            project.milestone_count_reached = mapped_count.get(project.id, 0)

    @api.depends('milestone_ids', 'milestone_ids.is_reached', 'milestone_ids.deadline', 'allow_milestones')
    def _compute_is_milestone_exceeded(self):
        today = fields.Date.context_today(self)
        read_group = self.env['project.milestone']._read_group([
            ('project_id', 'in', self.filtered('allow_milestones').ids),
            ('is_reached', '=', False),
            ('deadline', '<=', today)], ['project_id'], ['__count'])
        mapped_count = {project.id: count for project, count in read_group}
        for project in self:
            project.is_milestone_exceeded = bool(mapped_count.get(project.id, 0))

    @api.depends_context('company')
    @api.depends('company_id')
    def _compute_currency_id(self):
        default_currency_id = self.env.company.currency_id
        for project in self:
            project.currency_id = project.company_id.currency_id or default_currency_id

    @api.model
    def _search_is_milestone_exceeded(self, operator, value):
        if not isinstance(value, bool):
            raise ValueError(_('Invalid value: %s', value))
        if operator not in ['=', '!=']:
            raise ValueError(_('Invalid operator: %s', operator))

        query = """
            SELECT P.id
              FROM project_project P
         LEFT JOIN project_milestone M ON P.id = M.project_id
             WHERE M.is_reached IS false
               AND P.allow_milestones IS true
               AND M.deadline <= CAST(now() AS date)
        """
        if (operator == '=' and value is True) or (operator == '!=' and value is False):
            operator_new = 'inselect'
        else:
            operator_new = 'not inselect'
        return [('id', operator_new, (query, ()))]

    @api.depends('collaborator_ids', 'privacy_visibility')
    def _compute_collaborator_count(self):
        project_sharings = self.filtered(lambda project: project.privacy_visibility == 'portal')
        collaborator_read_group = self.env['project.collaborator']._read_group(
            [('project_id', 'in', project_sharings.ids)],
            ['project_id'],
            ['__count'],
        )
        collaborator_count_by_project = {project.id: count for project, count in collaborator_read_group}
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
                project.access_instruction_message = _('Grant portal users access to your project or tasks by adding them as followers. Customers automatically get access to their tasks in their portal.')
            elif project.privacy_visibility == 'followers':
                project.access_instruction_message = _('Grant employees access to your project or tasks by adding them as followers. Employees automatically get access to the tasks they are assigned to.')
            else:
                project.access_instruction_message = ''

    def _inverse_allow_task_dependencies(self):
        """ Reset state for waiting tasks in the project if the feature is disabled
            or recompute the tasks with dependencies if the project has the feature enabled again
        """
        project_with_task_dependencies_feature = self.filtered('allow_task_dependencies')
        projects_without_task_dependencies_feature = self - project_with_task_dependencies_feature
        ProjectTask = self.env['project.task']
        if (
            project_with_task_dependencies_feature
            and (
                open_tasks_with_dependencies := ProjectTask.search([
                    ('project_id', 'in', project_with_task_dependencies_feature.ids),
                    ('depend_on_ids.state', 'in', ProjectTask.OPEN_STATES),
                    ('state', 'in', ProjectTask.OPEN_STATES),
                ])
            )
        ):
            open_tasks_with_dependencies.state = '04_waiting_normal'
        if (
            projects_without_task_dependencies_feature
            and (
                waiting_tasks := ProjectTask.search([
                    ('project_id', 'in', projects_without_task_dependencies_feature.ids),
                    ('state', '=', '04_waiting_normal'),
                ])
            )
        ):
            waiting_tasks.state = '01_in_progress'

    # TODO: Remove in master
    @api.onchange('date_start', 'date')
    def _onchange_planned_date(self):
        return

    @api.model
    def _map_tasks_default_valeus(self, task, project):
        """ get the default value for the copied task on project duplication """
        return {
            'stage_id': task.stage_id.id,
            'name': task.name,
            'state': task.state,
            'company_id': project.company_id.id,
            'project_id': project.id,
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
        all_subtasks = new_tasks._get_all_subtasks()
        subtasks_not_displayed = all_subtasks.filtered(
            lambda task: not task.display_in_project
        )
        all_subtasks.filtered(
            lambda child: child.project_id == self
        ).write({
            'project_id': project.id
        })
        project.write({'tasks': [Command.set(new_tasks.ids)]})
        subtasks_not_displayed.write({
            'display_in_project': False
        })
        return True

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default is None:
            default = {}
        if not default.get('name'):
            default['name'] = _("%s (copy)", self.name)
        self_with_mail_context = self.with_context(mail_auto_subscribe_no_notify=True, mail_create_nosubscribe=True)
        project = super(Project, self_with_mail_context).copy(default)
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
        if any('label_tasks' in vals and not vals['label_tasks'] for vals in vals_list):
            task_label = _("Tasks")
            for vals in vals_list:
                if 'label_tasks' in vals and not vals['label_tasks']:
                    vals['label_tasks'] = task_label
        if self.env.user.has_group('project.group_project_stages'):
            if 'default_stage_id' in self._context:
                stage = self.env['project.project.stage'].browse(self._context['default_stage_id'])
                # The project's company_id must be the same as the stage's company_id
                if stage.company_id:
                    for vals in vals_list:
                        if vals.get('stage_id'):
                            continue
                        vals['company_id'] = stage.company_id.id
            else:
                companies_ids = [vals.get('company_id', False) for vals in vals_list] + [False]
                stages = self.env['project.project.stage'].search([('company_id', 'in', companies_ids)])
                for vals in vals_list:
                    if vals.get('stage_id'):
                        continue
                    # Pick the stage with the lowest sequence with no company or project's company
                    stage_domain = [False] if 'company_id' not in vals else [False, vals.get('company_id')]
                    stage = stages.filtered(lambda s: s.company_id.id in stage_domain)[:1]
                    vals['stage_id'] = stage.id

        projects = super().create(vals_list)
        return projects

    def write(self, vals):
        if vals.get('access_token'):
            self.ensure_one()  # We are not supposed to add a single access token to multiple project
            if self.privacy_visibility != 'portal':
                vals['access_token'] = ''

        # Here we modify the project's stage according to the selected company (selecting the first
        # stage in sequence that is linked to the company).
        company_id = vals.get('company_id')
        if self.env.user.has_group('project.group_project_stages') and company_id:
            projects_already_with_company = self.filtered(lambda p: p.company_id.id == company_id)
            if projects_already_with_company:
                projects_already_with_company.write({key: value for key, value in vals.items() if key != 'company_id'})
                self -= projects_already_with_company
            if company_id not in (None, *self.company_id.ids) and self.stage_id.company_id:
                ProjectStage = self.env['project.project.stage']
                vals["stage_id"] = ProjectStage.search(
                    [('company_id', 'in', (company_id, False))],
                    order=f"sequence asc, {ProjectStage._order}",
                    limit=1,
                ).id

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

        date_start = vals.get('date_start', True)
        date_end = vals.get('date', True)
        if not date_start or not date_end:
            vals['date_start'] = False
            vals['date'] = False
        else:
            no_current_date_begin = not all(project.date_start for project in self)
            no_current_date_end = not all(project.date for project in self)
            date_start_update = 'date_start' in vals
            date_end_update = 'date' in vals
            if (date_start_update and no_current_date_end and not date_end_update):
                del vals['date_start']
            elif (date_end_update and no_current_date_begin and not date_start_update):
                del vals['date']

        res = super(Project, self).write(vals) if vals else True

        if 'allow_task_dependencies' in vals and not vals.get('allow_task_dependencies'):
            self.env['project.task'].search([('project_id', 'in', self.ids), ('state', '=', '04_waiting_normal')]).write({'state': '01_in_progress'})

        if 'active' in vals:
            # archiving/unarchiving a project does it on its tasks, too
            self.with_context(active_test=False).mapped('tasks').write({'active': vals['active']})
        if 'name' in vals and self.analytic_account_id:
            projects_read_group = self.env['project.project']._read_group(
                [('analytic_account_id', 'in', self.analytic_account_id.ids)],
                ['analytic_account_id'],
                having=[('__count', '=', 1)],
            )
            analytic_account_to_update = self.env['account.analytic.account'].browse([
                analytic_account.id for [analytic_account] in projects_read_group
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

    def _order_field_to_sql(self, alias, field_name, direction, nulls, query):
        if field_name == 'is_favorite':
            sql_field = SQL(
                "%s IN (SELECT project_id FROM project_favorite_user_rel WHERE user_id = %s)",
                SQL.identifier(alias, 'id'), self.env.uid,
            )
            return SQL("%s %s %s", sql_field, direction, nulls)

        return super()._order_field_to_sql(alias, field_name, direction, nulls, query)

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

    @api.constrains('stage_id')
    def _ensure_stage_has_same_company(self):
        for project in self:
            if project.stage_id.company_id and project.stage_id.company_id != project.company_id:
                raise UserError(
                    _('This project is associated with %s, whereas the selected stage belongs to %s. '
                    'There are a couple of options to consider: either remove the company designation '
                    'from the project or from the stage. Alternatively, you can update the company '
                    'information for these records to align them under the same company.', project.company_id.name, project.stage_id.company_id.name)
                    if project.company_id else
                    _('This project is not associated to any company, while the stage is associated to %s. '
                    'There are a couple of options to consider: either change the project\'s company '
                    'to align with the stage\'s company or remove the company designation from the stage', project.stage_id.company_id.name)
                )

    # ---------------------------------------------------
    # Mail gateway
    # ---------------------------------------------------

    def _track_template(self, changes):
        res = super()._track_template(changes)
        project = self[0]
        if self.user_has_groups('project.group_project_stages') and 'stage_id' in changes and project.stage_id.mail_template_id:
            res['stage_id'] = (project.stage_id.mail_template_id, {
                'auto_delete_keep_log': False,
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
        if not self.rating_active:
            res -= self.env.ref('project.mt_project_task_rating')
        if len(self) == 1:
            waiting_subtype = self.env.ref('project.mt_project_task_waiting')
            if not self.allow_task_dependencies and waiting_subtype in res:
                res -= waiting_subtype
        return res

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Give access to the portal user/customer if the project visibility is portal. """
        groups = super()._notify_get_recipients_groups(message, model_description, msg_vals=msg_vals)
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
        context = action['context'].replace('active_id', str(self.id))
        context = ast.literal_eval(context)
        context.update({
            'stage_name_and_sequence_per_id': {
                stage.id: {
                    'sequence': stage.sequence,
                    'name': stage.name
                } for stage in self.type_ids
            }
        })
        action['context'] = context
        return action

    # TODO to remove in master
    def action_project_timesheets(self):
        pass

    def project_update_all_action(self):
        action = self.env['ir.actions.act_window']._for_xml_id('project.project_update_all_action')
        action['display_name'] = _("%(name)s's Updates", name=self.name)
        return action

    def action_project_sharing(self):
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id('project.project_sharing_project_task_action')
        action['context'] = {
            'default_project_id': self.id,
            'delete': False,
            'search_default_open_tasks': True,
            'active_id_chatter': self.id,
            'allow_milestones': self.allow_milestones,
        }
        action['display_name'] = self.name
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
        show_profitability = self._show_profitability()
        panel_data = {
            'user': self._get_user_values(),
            'buttons': sorted(self._get_stat_buttons(), key=lambda k: k['sequence']),
            'currency_id': self.currency_id.id,
            'show_project_profitability_helper': show_profitability and self._show_profitability_helper(),
        }
        if self.allow_milestones:
            panel_data['milestones'] = self._get_milestones()
        if show_profitability:
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

    def _show_profitability_helper(self):
        return self.user_has_groups('analytic.group_analytic_accounting')

    def _get_profitability_aal_domain(self):
        return [('account_id', 'in', self.analytic_account_id.ids)]

    def _get_profitability_items(self, with_action=True):
        return self._get_items_from_aal(with_action)

    def _get_items_from_aal(self, with_action=True):
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
        if self.task_count:
            number = _lt(
                "%(closed_task_count)s / %(task_count)s (%(closed_rate)s%%)",
                closed_task_count=self.closed_task_count,
                task_count=self.task_count,
                closed_rate=round(100 * self.closed_task_count / self.task_count),
            )
        else:
            number = _lt(
                "%(closed_task_count)s / %(task_count)s",
                closed_task_count=self.closed_task_count,
                task_count=self.task_count,
            )
        buttons = [{
            'icon': 'check',
            'text': _lt('Tasks'),
            'number': number,
            'action_type': 'object',
            'action': 'action_view_tasks',
            'show': True,
            'sequence': 1,
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
                'text': _lt('Average Rating'),
                'number': f'{int(self.rating_avg) if self.rating_avg.is_integer() else round(self.rating_avg, 1)} / 5',
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
                    'stage_name_and_sequence_per_id': {
                        stage.id: {
                            'sequence': stage.sequence,
                            'name': stage.name
                        } for stage in self.type_ids
                    },
                }),
                'show': True,
                'sequence': 60,
            })
        return buttons

    # ---------------------------------------------------
    #  Business Methods
    # ---------------------------------------------------

    def _get_hide_partner(self):
        return False

    @api.model
    def _create_analytic_account_from_values(self, values):
        company = self.env['res.company'].browse(values.get('company_id', False))
        project_plan_id = int(self.env['ir.config_parameter'].sudo().get_param('analytic.analytic_plan_projects'))

        if not project_plan_id:
            project_plan, _other_plans = self.env['account.analytic.plan']._get_all_plans()
            project_plan_id = project_plan.id

        analytic_account = self.env['account.analytic.account'].create({
            'name': values.get('name', _('Unknown Analytic Account')),
            'company_id': company.id,
            'partner_id': values.get('partner_id'),
            'plan_id': project_plan_id,
        })
        return analytic_account

    def _create_analytic_account(self):
        for project in self:
            project.analytic_account_id = self._create_analytic_account_from_values({
                'company_id': project.company_id.id,
                'name': project.name,
                'partner_id': project.partner_id.id
            })

    def _get_projects_to_make_billable_domain(self):
        return [('partner_id', '!=', False)]

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

```

## File: models\project_project_stage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import UserError

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
    company_id = fields.Many2one('res.company', string="Company")

    def copy(self, default=None):
        default = dict(default or {})
        if not default.get('name'):
            default['name'] = _("%s (copy)", self.name)
        return super().copy(default)

    def unlink_wizard(self, stage_view=False):
        wizard = self.with_context(active_test=False).env['project.project.stage.delete.wizard'].create({
            'stage_ids': self.ids
        })

        context = dict(self.env.context)
        context['stage_view'] = stage_view
        return {
            'name': _('Delete Project Stage'),
            'view_mode': 'form',
            'res_model': 'project.project.stage.delete.wizard',
            'views': [(self.env.ref('project.view_project_project_stage_delete_wizard').id, 'form')],
            'type': 'ir.actions.act_window',
            'res_id': wizard.id,
            'target': 'new',
            'context': context,
        }

    def write(self, vals):
        if vals.get('company_id'):
            # Checking if there is a project with a different company_id than the target one. If so raise an error since this is not allowed
            project = self.env['project.project'].search(['&', ('stage_id', 'in', self.ids), ('company_id', '!=', vals['company_id'])], limit=1)
            if project:
                company = self.env['res.company'].browse(vals['company_id'])
                raise UserError(
                    _("You are not able to switch the company of this stage to %(company_name)s since it currently "
                    "includes projects associated with %(project_company_name)s. Please ensure that this stage exclusively "
                    "consists of projects linked to %(company_name)s.",
                        company_name=company.name,
                        project_company_name=project.company_id.name or "no company"
                    )
                )

        if 'active' in vals and not vals['active']:
            self.env['project.project'].search([('stage_id', 'in', self.ids)]).write({'active': False})
        return super().write(vals)

    def toggle_active(self):
        res = super().toggle_active()
        stage_active = self.filtered('active')
        inactive_projects = self.env['project.project'].with_context(active_test=False).search(
            [('active', '=', False), ('stage_id', 'in', stage_active.ids)], limit=1)
        if stage_active and inactive_projects:
            wizard = self.env['project.project.stage.delete.wizard'].create({
                'stage_ids': stage_active.ids,
            })

            return {
                'name': _('Unarchive Projects'),
                'view_mode': 'form',
                'res_model': 'project.project.stage.delete.wizard',
                'views': [(self.env.ref('project.view_project_project_stage_unarchive_wizard').id, 'form')],
                'type': 'ir.actions.act_window',
                'res_id': wizard.id,
                'target': 'new',
            }
        return res

```

## File: models\project_tags.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import api, fields, models, SUPERUSER_ID
from odoo.osv import expression


class ProjectTags(models.Model):
    """ Tags of project's tasks """
    _name = "project.tags"
    _description = "Project Tags"
    _order = "name"

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
            tag_ids = self._name_search('')
            domain = expression.AND([domain, [('id', 'in', tag_ids)]])
        return super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    @api.model
    def search_read(self, domain=None, fields=None, offset=0, limit=None, order=None):
        if 'project_id' in self.env.context:
            tag_ids = self._name_search('')
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
    def _name_search(self, name, domain=None, operator='ilike', limit=None, order=None):
        ids = []
        if not (name == '' and operator in ('like', 'ilike')):
            if domain is None:
                domain = []
            domain += [('name', operator, name)]
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
            # we apply the domain and limit to the ids we've already found
            ids += self.env['project.tags'].search(expression.AND([domain, project_tasks_tags_domain]), limit=limit, order=order).ids
        if not limit or len(ids) < limit:
            limit = limit and limit - len(ids)
            ids += self.env['project.tags'].search(expression.AND([domain, [('id', 'not in', ids)]]), limit=limit, order=order).ids
        return ids

    @api.model
    def name_create(self, name):
        existing_tag = self.search([('name', '=ilike', name.strip())], limit=1)
        if existing_tag:
            return existing_tag.id, existing_tag.display_name
        return super().name_create(name)

```

## File: models\project_task.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from pytz import UTC
from collections import defaultdict
from datetime import timedelta, datetime, time

from odoo import api, Command, fields, models, tools, SUPERUSER_ID, _, _lt
from odoo.addons.rating.models import rating_data
from odoo.addons.web_editor.tools import handle_history_divergence
from odoo.exceptions import UserError, ValidationError, AccessError
from odoo.osv import expression
from odoo.tools.misc import get_lang
from odoo.addons.resource.models.utils import filter_domain_leaf


PROJECT_TASK_READABLE_FIELDS = {
    'id',
    'active',
    'priority',
    'project_id',
    'display_in_project',
    'color',
    'subtask_count',
    'email_from',
    'create_date',
    'write_date',
    'company_id',
    'displayed_image_id',
    'display_name',
    'portal_user_names',
    'user_ids',
    'display_parent_task_button',
    'allow_milestones',
    'milestone_id',
    'has_late_and_unreached_milestone',
    'date_assign',
    'dependent_ids',
    'message_is_follower',
    'recurring_task',
    'closed_subtask_count',
}

PROJECT_TASK_WRITABLE_FIELDS = {
    'name',
    'description',
    'partner_id',
    'date_deadline',
    'date_last_stage_update',
    'tag_ids',
    'sequence',
    'stage_id',
    'child_ids',
    'parent_id',
    'priority',
    'state',
}

CLOSED_STATES = {
    '1_done': 'Done',
    '1_canceled': 'Canceled',
}


class Task(models.Model):
    _name = "project.task"
    _description = "Task"
    _date_name = "date_assign"
    _inherit = [
        'portal.mixin',
        'mail.thread.cc',
        'mail.activity.mixin',
        'rating.mixin',
        'mail.tracking.duration.mixin'
    ]
    _mail_post_access = 'read'
    _order = "priority desc, sequence, date_deadline asc, id desc"
    _primary_email = 'email_from'
    _systray_view = 'activity'
    _track_duration_field = 'stage_id'

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
        return self.stage_find(project_id, order="fold, sequence, id")

    @api.model
    def _default_personal_stage_type_id(self):
        default_id = self.env.context.get('default_personal_stage_type_ids')
        return (default_id or self.env['project.task.type'].search([('user_id', '=', self.env.user.id)], limit=1).ids or [False])[0]

    @api.model
    def _default_user_ids(self):
        return self.env.context.keys() & {'default_personal_stage_type_ids', 'default_personal_stage_type_id'} and self.env.user

    @api.model
    def _default_company_id(self):
        if self._context.get('default_project_id'):
            return self.env['project.project'].browse(self._context['default_project_id']).company_id
        return False

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        search_domain = [('id', 'in', stages.ids)]
        if 'default_project_id' in self.env.context and not self._context.get('subtask_action') and 'project_kanban' in self.env.context:
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
        domain="[('project_ids', '=', project_id)]")
    tag_ids = fields.Many2many('project.tags', string='Tags')

    state = fields.Selection([
        ('01_in_progress', 'In Progress'),
        ('02_changes_requested', 'Changes Requested'),
        ('03_approved', 'Approved'),
        *CLOSED_STATES.items(),
        ('04_waiting_normal', 'Waiting'),
    ], string='State', copy=False, default='01_in_progress', required=True, compute='_compute_state', inverse='_inverse_state', readonly=False, store=True, index=True, recursive=True, tracking=True)

    create_date = fields.Datetime("Created On", readonly=True, index=True)
    write_date = fields.Datetime("Last Updated On", readonly=True)
    date_end = fields.Datetime(string='Ending Date', index=True, copy=False)
    date_assign = fields.Datetime(string='Assigning Date', copy=False, readonly=True,
        help="Date on which this task was last assigned (or unassigned). Based on this, you can get statistics on the time it usually takes to assign tasks.")
    date_deadline = fields.Datetime(string='Deadline', index=True, tracking=True)

    date_last_stage_update = fields.Datetime(string='Last Stage Update',
        index=True,
        copy=False,
        readonly=True,
        help="Date on which the state of your task has last been modified.\n"
            "Based on this information you can identify tasks that are stalling and get statistics on the time it usually takes to move tasks from one stage/state to another.")

    project_id = fields.Many2one('project.project', string='Project', domain="['|', ('company_id', '=', False), ('company_id', '=?',  company_id)]", index=True, tracking=True, change_default=True)
    display_in_project = fields.Boolean(default=True, readonly=True)
    task_properties = fields.Properties('Properties', definition='project_id.task_properties_definition', copy=True)
    allocated_hours = fields.Float("Allocated Time", tracking=True)
    subtask_allocated_hours = fields.Float("Sub-tasks Allocated Time", compute='_compute_subtask_allocated_hours',
        help="Sum of the hours allocated for all the sub-tasks (and their own sub-tasks) linked to this task. Usually less than or equal to the allocated hours of this task.")
    # Tracking of this field is done in the write function
    user_ids = fields.Many2many('res.users', relation='project_task_user_rel', column1='task_id', column2='user_id',
        string='Assignees', context={'active_test': False}, tracking=True, default=_default_user_ids, domain="[('share', '=', False), ('active', '=', True)]")
    # User names displayed in project sharing views
    portal_user_names = fields.Char(compute='_compute_portal_user_names', compute_sudo=True, search='_search_portal_user_names')
    # Second Many2many containing the actual personal stage for the current user
    # See project_task_stage_personal.py for the model defininition
    personal_stage_type_ids = fields.Many2many('project.task.type', 'project_task_user_rel', column1='task_id', column2='stage_id',
        ondelete='restrict', group_expand='_read_group_personal_stage_type_ids', copy=False,
        domain="[('user_id', '=', user.id)]", depends=['user_ids'], string='Personal Stages')
    # Personal Stage computed from the user
    personal_stage_id = fields.Many2one('project.task.stage.personal', string='Personal Stage State', compute_sudo=False,
        compute='_compute_personal_stage_id', help="The current user's personal stage.")
    # This field is actually a related field on personal_stage_id.stage_id
    # However due to the fact that personal_stage_id is computed, the orm throws out errors
    # saying the field cannot be searched.
    personal_stage_type_id = fields.Many2one('project.task.type', string='Personal Stage',
        compute='_compute_personal_stage_type_id', inverse='_inverse_personal_stage_type_id', store=False,
        search='_search_personal_stage_type_id', default=_default_personal_stage_type_id,
        help="The current user's personal task stage.", domain="[('user_id', '=', uid)]")
    partner_id = fields.Many2one('res.partner',
        string='Customer', recursive=True, tracking=True, compute='_compute_partner_id', store=True, readonly=False,
        domain="['|', ('company_id', '=?', company_id), ('company_id', '=', False)]", )
    email_cc = fields.Char(help='Email addresses that were in the CC of the incoming emails from this task and that are not currently linked to an existing customer.')
    company_id = fields.Many2one('res.company', string='Company', compute='_compute_company_id', store=True, readonly=False, recursive=True, copy=True, default=_default_company_id)
    color = fields.Integer(string='Color Index')
    rating_active = fields.Boolean(string='Project Rating Status', related="project_id.rating_active")
    attachment_ids = fields.One2many('ir.attachment', compute='_compute_attachment_ids', string="Main Attachments",
        help="Attachments that don't come from a message.")
    # In the domain of displayed_image_id, we couln't use attachment_ids because a one2many is represented as a list of commands so we used res_model & res_id
    displayed_image_id = fields.Many2one('ir.attachment', domain="[('res_model', '=', 'project.task'), ('res_id', '=', id), ('mimetype', 'ilike', 'image')]", string='Cover Image')

    parent_id = fields.Many2one('project.task', string='Parent Task', index=True, domain="['!', ('id', 'child_of', id)]", tracking=True)
    child_ids = fields.One2many('project.task', 'parent_id', string="Sub-tasks", domain="[('recurring_task', '=', False)]")
    subtask_count = fields.Integer("Sub-task Count", compute='_compute_subtask_count')
    closed_subtask_count = fields.Integer("Closed Sub-tasks Count", compute='_compute_subtask_count')
    project_privacy_visibility = fields.Selection(related='project_id.privacy_visibility', string="Project Visibility")
    # Computed field about working time elapsed between record creation and assignation/closing.
    working_hours_open = fields.Float(compute='_compute_elapsed', string='Working Hours to Assign', digits=(16, 2), store=True, group_operator="avg")
    working_hours_close = fields.Float(compute='_compute_elapsed', string='Working Hours to Close', digits=(16, 2), store=True, group_operator="avg")
    working_days_open = fields.Float(compute='_compute_elapsed', string='Working Days to Assign', store=True, group_operator="avg")
    working_days_close = fields.Float(compute='_compute_elapsed', string='Working Days to Close', store=True, group_operator="avg")
    # customer portal: include comment and (incoming/outgoing) emails in communication history
    website_message_ids = fields.One2many(domain=lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment', 'email_outgoing'])])
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

    # Project sharing fields
    display_parent_task_button = fields.Boolean(compute='_compute_display_parent_task_button', compute_sudo=True)

    # recurrence fields
    recurring_task = fields.Boolean(string="Recurrent")
    recurring_count = fields.Integer(string="Tasks in Recurrence", compute='_compute_recurring_count')
    recurrence_id = fields.Many2one('project.task.recurrence', copy=False)
    repeat_interval = fields.Integer(string='Repeat Every', default=1, compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_unit = fields.Selection([
        ('day', 'Days'),
        ('week', 'Weeks'),
        ('month', 'Months'),
        ('year', 'Years'),
    ], default='week', compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_type = fields.Selection([
        ('forever', 'Forever'),
        ('until', 'Until'),
    ], default="forever", string="Until", compute='_compute_repeat', readonly=False, groups="project.group_project_user")
    repeat_until = fields.Date(string="End Date", compute='_compute_repeat', readonly=False, groups="project.group_project_user")

    # Account analytic
    analytic_account_id = fields.Many2one('account.analytic.account', ondelete='set null', compute='_compute_analytic_account_id', store=True, readonly=False,
        domain="[('company_id', '=?', company_id)]",
        help="Analytic account to which this task and its timesheets are linked.\n"
            "Track the costs and revenues of your task by setting its analytic account on your related documents (e.g. sales orders, invoices, purchase orders, vendor bills, expenses etc.).\n"
            "By default, the analytic account of the project is set. However, it can be changed on each task individually if necessary.")

    # Quick creation shortcuts
    display_name = fields.Char(compute='_compute_display_name', inverse='_inverse_display_name',
        help="""Use these keywords in the title to set new tasks:\n
            #tags Set tags on the task
            @user Assign the task to a user
            ! Set the task a high priority\n
            Make sure to use the right format and order e.g. Improve the configuration screen #feature #v16 @Mitchell !""",
    )

    _sql_constraints = [
        ('recurring_task_has_no_parent', 'CHECK (NOT (recurring_task IS TRUE AND parent_id IS NOT NULL))', "A subtask cannot be recurrent."),
        ('private_task_has_no_parent', 'CHECK (NOT (project_id IS NULL AND parent_id IS NOT NULL))', "A private task cannot have a parent."),
    ]

    @api.constrains('company_id', 'partner_id')
    def _ensure_company_consistency_with_partner(self):
        """ Ensures that the company of the task is valid for the partner. """
        for task in self:
            if task.partner_id and task.partner_id.company_id and task.company_id and task.company_id != task.partner_id.company_id:
                raise ValidationError(_('The task and the associated partner must be linked to the same company.'))

    @property
    def SELF_READABLE_FIELDS(self):
        return PROJECT_TASK_READABLE_FIELDS | self.SELF_WRITABLE_FIELDS

    @property
    def SELF_WRITABLE_FIELDS(self):
        return PROJECT_TASK_WRITABLE_FIELDS

    @api.depends('project_id.analytic_account_id')
    def _compute_analytic_account_id(self):
        for task in self:
            task.analytic_account_id = task.project_id.analytic_account_id

    @api.depends('stage_id', 'depend_on_ids.state')
    def _compute_state(self):
        for task in self:
            dependent_open_tasks = []
            if task.allow_task_dependencies:
                dependent_open_tasks = [dependent_task for dependent_task in task.depend_on_ids if dependent_task.state not in CLOSED_STATES]
            # if one of the blocking task is in a blocking state
            if dependent_open_tasks:
                # here we check that the blocked task is not already in a closed state (if the task is already done we don't put it in waiting state)
                if task.state not in CLOSED_STATES:
                    task.state = '04_waiting_normal'
            # if the task as no blocking dependencies and is in waiting_normal, the task goes back to in progress
            elif task.state not in CLOSED_STATES:
                task.state = '01_in_progress'

    @property
    def OPEN_STATES(self):
        """ Return a list of the technical names complementing the CLOSED_STATES, a.k.a the open states """
        return list(set(self._fields['state'].get_values(self.env)) - set(CLOSED_STATES))

    @api.onchange('project_id')
    def _onchange_project_id(self):
        if self.state != '04_waiting_normal':
            self.state = '01_in_progress'

    def is_blocked_by_dependences(self):
        return any(blocking_task.state not in CLOSED_STATES for blocking_task in self.depend_on_ids)

    def _inverse_state(self):
        last_task_id_per_recurrence_id = self.recurrence_id._get_last_task_id_per_recurrence_id()
        for task in self:
            if task.state in CLOSED_STATES and task.id == last_task_id_per_recurrence_id.get(task.recurrence_id.id):
                task.recurrence_id._create_next_occurrence(task)

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
        return [
            'repeat_interval',
            'repeat_unit',
            'repeat_type',
            'repeat_until',
        ]

    @api.depends('recurring_task')
    def _compute_repeat(self):
        rec_fields = self._get_recurrence_fields()
        defaults = self.default_get(rec_fields)
        for task in self:
            for f in rec_fields:
                if task.recurrence_id:
                    task[f] = task.recurrence_id.sudo()[f]
                else:
                    if task.recurring_task:
                        task[f] = defaults.get(f)
                    else:
                        task[f] = False

    def _is_recurrence_valid(self):
        self.ensure_one()
        return self.repeat_interval > 0 and\
                (self.repeat_type != 'until' or self.repeat_until and self.repeat_until > fields.Date.today())

    @api.depends('recurrence_id')
    def _compute_recurring_count(self):
        self.recurring_count = 0
        recurring_tasks = self.filtered(lambda l: l.recurrence_id)
        count = self.env['project.task']._read_group([('recurrence_id', 'in', recurring_tasks.recurrence_id.ids)], ['recurrence_id'], ['__count'])
        tasks_count = {recurrence.id: count for recurrence, count in count}
        for task in recurring_tasks:
            task.recurring_count = tasks_count.get(task.recurrence_id.id, 0)

    @api.depends('dependent_ids')
    def _compute_dependent_tasks_count(self):
        tasks_with_dependency = self.filtered('allow_task_dependencies')
        (self - tasks_with_dependency).dependent_tasks_count = 0
        if tasks_with_dependency:
            group_dependent = self.env['project.task']._read_group([
                ('depend_on_ids', 'in', tasks_with_dependency.ids),
            ], ['depend_on_ids'], ['__count'])
            dependent_tasks_count_dict = {
                depend_on.id: count
                for depend_on, count in group_dependent
            }
            for task in tasks_with_dependency:
                task.dependent_tasks_count = dependent_tasks_count_dict.get(task.id, 0)

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

    def _compute_access_url(self):
        super(Task, self)._compute_access_url()
        for task in self:
            task.access_url = f'/my/tasks/{task.id}'

    def _compute_access_warning(self):
        super(Task, self)._compute_access_warning()
        for task in self.filtered(lambda x: x.project_id.privacy_visibility != 'portal'):
            task.access_warning = _(
                "The task cannot be shared with the recipient(s) because the privacy of the project is too restricted. Set the privacy of the project to 'Visible by following customers' in order to make it accessible by the recipient(s).")

    @api.depends('child_ids.allocated_hours')
    def _compute_subtask_allocated_hours(self):
        for task in self:
            task.subtask_allocated_hours = sum(task.child_ids.mapped('allocated_hours'))

    @api.depends('child_ids')
    def _compute_subtask_count(self):
        total_and_closed_subtask_count_per_parent_id = {
            parent.id: (count, sum(s in CLOSED_STATES for s in states))
            for parent, states, count in self.env['project.task']._read_group(
                [('parent_id', 'in', self.ids)],
                ['parent_id'],
                ['state:array_agg', '__count'],
            )
        }
        for task in self:
            task.subtask_count, task.closed_subtask_count = total_and_closed_subtask_count_per_parent_id.get(task.id, (0, 0))

    @api.onchange('company_id')
    def _onchange_task_company(self):
        if self.project_id.company_id and self.project_id.company_id != self.company_id:
            self.project_id = False

    @api.depends('project_id.company_id', 'parent_id.company_id')
    def _compute_company_id(self):
        for task in self:
            if not task.parent_id and not task.project_id:
                continue
            task.company_id = task.project_id.company_id or task.parent_id.company_id

    @api.depends('project_id')
    def _compute_stage_id(self):
        for task in self:
            project = task.project_id or task.parent_id.project_id
            if project:
                if project not in task.stage_id.project_ids:
                    task.stage_id = task.stage_find(project.id, [('fold', '=', False)])
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
        if self._origin:
            # fetch 'user_ids' in superuser mode (and override value in cache
            # browse is useful to avoid miscache because of the newIds contained in self
            self.invalidate_recordset(fnames=['user_ids'])
            self._origin.fetch(['user_ids'])
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

    def _get_group_pattern(self):
        return {
            'tags_and_users': r'\s([#@]%s[^\s]+)',
            'priority': r'\s(!)',
        }

    def _prepare_pattern_groups(self):
        group = self._get_group_pattern()
        return [
            group['tags_and_users'] % '',
            group['priority'],
        ]

    def _get_groups_patterns(self):
        return [
            r'(?:%s)*' % ('|').join(self._prepare_pattern_groups()),
        ]

    def _get_cannot_start_with_patterns(self):
        return [r'(?![#!@\s])']

    def _extract_tags_and_users(self):
        tags = []
        users = []
        tags_and_users_group = self._get_group_pattern()['tags_and_users']
        for word in re.findall(tags_and_users_group % '', self.display_name):
            (tags if word.startswith('#') else users).append(word[1:])
        users_to_keep = []
        user_ids = []
        for user in users:
            matched_users = self.env['res.users'].name_search(user)
            if len(matched_users) == 1:
                user_ids.append(Command.link(matched_users[0][0]))
            else:
                users_to_keep.append(r'%s\b' % user)
        self.user_ids = user_ids
        if tags:
            domain = expression.OR([[('name', '=ilike', tag)] for tag in tags])
            existing_tags = self.env['project.tags'].search(domain)
            existing_tags_names = {tag.name.lower() for tag in existing_tags}
            new_tags_names = {tag for tag in tags if tag.lower() not in existing_tags_names}
            self.tag_ids = [Command.set(existing_tags.ids)] + [Command.create({'name': name}) for name in new_tags_names]
        pattern = tags_and_users_group % ('(?!%s)' % ('|').join(users_to_keep) if users_to_keep else '')
        self.display_name, dummy = re.subn(pattern, '', self.display_name)

    def _extract_priority(self):
        self.priority = "1"
        priority_group = self._get_group_pattern()['priority']
        self.display_name, dummy = re.subn(priority_group, '', self.display_name)

    def _get_groups(self):
        return [
            lambda task: task._extract_tags_and_users(),
            lambda task: task._extract_priority(),
        ]

    def _inverse_display_name(self):
        for task in self:
            pattern = re.compile(r'^%s.+?%s$' % (
                ('').join(task._get_cannot_start_with_patterns()),
                ('').join(task._get_groups_patterns()))
            )
            match = pattern.match(task.display_name)
            if match:
                for group, extract_data in enumerate(task._get_groups(), start=1):
                    if match.group(group):
                        extract_data(task)
                task.name = task.display_name.strip()

    def _portal_get_parent_hash_token(self, pid):
        return self.project_id._sign_token(pid)

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
        default['child_ids'] = [child.copy({'name': child.name}).id for child in self.child_ids]
        self_with_mail_context = self.with_context(mail_auto_subscribe_no_notify=True, mail_create_nosubscribe=True)
        task_copy = super(Task, self_with_mail_context).copy(default)
        original_task_state = self.state
        if self.allow_task_dependencies:
            task_mapping = self.env.context.get('task_mapping')
            task_mapping[self.id] = task_copy.id
            new_tasks = task_mapping.values()
            self.write({'depend_on_ids': [Command.unlink(t.id) for t in self.depend_on_ids if t.id in new_tasks]})
            self.write({'dependent_ids': [Command.unlink(t.id) for t in self.dependent_ids if t.id in new_tasks]})
            task_copy.write({'depend_on_ids': [Command.link(task_mapping.get(t.id, t.id)) for t in self.depend_on_ids]})
            task_copy.write({'dependent_ids': [Command.link(task_mapping.get(t.id, t.id)) for t in self.dependent_ids]})
        if self.state != original_task_state:
            self.write({'state': original_task_state})
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
        makes the view cache dependent on the fact the user has the group portal or not"""
        key = super()._get_view_cache_key(view_id, view_type, **options)
        return key + (self.env.user.has_group('base.group_portal'),)

    @api.model
    def default_get(self, default_fields):
        vals = super(Task, self).default_get(default_fields)

        if 'repeat_until' in default_fields:
            vals['repeat_until'] = fields.Date.today() + timedelta(days=7)

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
            if 'company_id' in default_fields and 'default_project_id' not in self.env.context:
                vals['company_id'] = project.sudo().company_id.id
        elif 'default_user_ids' not in self.env.context and 'user_ids' in default_fields:
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
            or key == 'default_user_ids' and value is False
            or not key.startswith('default_')
            or key[8:] in (field for field in self.SELF_WRITABLE_FIELDS if self._fields[field].type not in ('one2many', 'many2many'))
        }

    def read(self, fields=None, load='_classic_read'):
        self._ensure_fields_are_accessible(fields)
        return super(Task, self).read(fields=fields, load=load)

    @api.model
    def _read_group_check_field_access_rights(self, field_names):
        super()._read_group_check_field_access_rights(field_names)
        self._ensure_fields_are_accessible(field_names)

    @api.model
    def _search(self, domain, offset=0, limit=None, order=None, access_rights_uid=None):
        fields_list = {term[0] for term in domain if isinstance(term, (tuple, list)) and term not in [expression.TRUE_LEAF, expression.FALSE_LEAF]}
        self._ensure_fields_are_accessible(fields_list)
        return super()._search(domain, offset, limit, order, access_rights_uid)

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
                raise AccessError(_('You have not write access of %s field.', field))

    def _set_stage_on_project_from_task(self):
        stage_ids_per_project = defaultdict(list)
        for task in self:
            if task.stage_id and task.stage_id not in task.project_id.type_ids and task.stage_id.id not in stage_ids_per_project[task.project_id]:
                stage_ids_per_project[task.project_id].append(task.stage_id.id)

        for project, stage_ids in stage_ids_per_project.items():
            project.write({'type_ids': [Command.link(stage_id) for stage_id in stage_ids]})

    def _load_records_create(self, vals_list):
        for vals in vals_list:
            if vals.get('recurring_task'):
                if not vals.get('recurrence_id'):
                    default_val = self.default_get(self._get_recurrence_fields())
                    vals.update(**default_val)
            project_id = vals.get('project_id')
            if project_id:
                self = self.with_context(default_project_id=project_id)
        tasks = super()._load_records_create(vals_list)

        return tasks

    @api.model_create_multi
    def create(self, vals_list):
        new_context = dict(self.env.context)
        default_personal_stage = new_context.pop('default_personal_stage_type_ids', False)
        self = self.with_context(new_context)

        is_portal_user = self.env.user.has_group('base.group_portal')
        if is_portal_user:
            self.check_access_rights('create')
        default_stage = dict()
        for vals in vals_list:
            project_id = vals.get('project_id')
            if vals.get('user_ids'):
                vals['date_assign'] = fields.Datetime.now()
                if not (vals.get('parent_id') or project_id or self._context.get('default_project_id')):
                    user_ids = self._fields['user_ids'].convert_to_cache(vals.get('user_ids', []), self)
                    if self.env.user.id not in list(user_ids) + [SUPERUSER_ID]:
                        vals['user_ids'] = [Command.set(list(user_ids) + [self.env.user.id])]

            if default_personal_stage and 'personal_stage_type_id' not in vals:
                vals['personal_stage_type_id'] = default_personal_stage[0]
            if not vals.get('name') and vals.get('display_name'):
                vals['name'] = vals['display_name']
            if is_portal_user:
                self._ensure_fields_are_accessible(vals.keys(), operation='write', check_group_user=False)

            if project_id:
                # set the project => "I want to display the task in the project"
                #                 => => set `display_in_project` to True
                vals['display_in_project'] = vals.get('display_in_project', True)
            elif vals.get('parent_id'):
                # unset the project => 2 cases:
                # 1) the task has no parent => "I want it to be private" => nothing to do
                # 2) the task has a parent  => "I don't want to display the task in the project"
                #                           => set `project_id` to the one of its parent and `display_in_project` to False
                project_id = self.browse(vals['parent_id']).project_id.id
                vals.update({
                    'project_id': project_id,
                    'display_in_project': False,
                })

            project_id = project_id or self.env.context.get('default_project_id')
            if project_id and not "company_id" in vals:
                vals["company_id"] = self.env["project.project"].browse(
                    project_id
                ).company_id.id
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
            # Stage change: Update date_end if folded stage and date_last_stage_update
            if vals.get('stage_id'):
                vals.update(self.update_date_end(vals['stage_id']))
                vals['date_last_stage_update'] = fields.Datetime.now()
            # recurrence
            rec_fields = vals.keys() & self._get_recurrence_fields()
            if rec_fields and vals.get('recurring_task') is True:
                rec_values = {rec_field: vals[rec_field] for rec_field in rec_fields}
                recurrence = self.env['project.task.recurrence'].create(rec_values)
                vals['recurrence_id'] = recurrence.id
        # The sudo is required for a portal user as the record creation
        # requires the read access on other models, as mail.template
        # in order to compute the field tracking
        was_in_sudo = self.env.su
        if is_portal_user:
            vals_list_no_sudo, vals_list = zip(*(self._get_portal_sudo_vals(vals, defaults=True) for vals in vals_list))
            self_no_sudo, self = self, self.sudo().with_context(self._get_portal_sudo_context())
        tasks = super(Task, self.with_context(mail_create_nosubscribe=True)).create(vals_list)
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
        if tasks.project_id:
            tasks._set_stage_on_project_from_task()
        for task in tasks:
            if task.project_id.privacy_visibility == 'portal':
                task._portal_ensure_token()
            for follower in task.parent_id.message_follower_ids:
                task.message_subscribe(follower.partner_id.ids, follower.subtype_ids.ids)
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

        if 'project_id' in vals:
            project_id = vals['project_id']
            if project_id:
                # set the project => "I want to display the task in the project"
                #                 => set `display_in_project` to True
                if 'display_in_project' not in vals:
                    vals['display_in_project'] = True
                    no_display_subtasks = self.child_ids.filtered(lambda t: not t.display_in_project)
                    if no_display_subtasks:
                        no_display_subtasks.write({'project_id': project_id})
            else:
                # unset the project => 2 cases:
                # 1) the task has no parent => "I want it to be private" => nothing to do
                # 2) the task has a parent  => "I don't want to display the task in the project"
                #                           => set `project_id` back and `display_in_project` to False
                if 'parent_id' in vals:
                    if vals['parent_id']:
                        vals.update({
                            'project_id': self.browse(vals['parent_id']).project_id.id,
                            'display_in_project': False,
                        })
                else:
                    task_ids_per_parent_project_id = defaultdict(list)
                    for task in self:
                        task_ids_per_parent_project_id[task.parent_id.project_id.id].append(task.id)
                    self = self.browse(task_ids_per_parent_project_id.pop(False, False))
                    for parent_project_id, task_ids in task_ids_per_parent_project_id.items():
                        self.browse(task_ids).write({
                            **vals,
                            'project_id': parent_project_id,
                            'display_in_project': False,
                        })

        if 'parent_id' in vals:
            parent_id = vals['parent_id']
            if parent_id in self.ids:
                raise UserError(_("Sorry. You can't set a task as its parent task."))
            elif not parent_id:
                # unset the parent => "I want to display the task back in the project"
                #                    => set `display_in_project` to True
                vals['display_in_project'] = True

        if 'milestone_id' in vals:
            # WARNING: has to be done after 'project_id' vals is written on subtasks
            milestone = self.env['project.milestone'].browse(vals['milestone_id'])

            # 1. Task for which the milestone is unvalid -> milestone_id is reset
            if 'project_id' not in vals:
                unvalid_milestone_tasks = self.filtered(lambda task: task.project_id != milestone.project_id) if vals['milestone_id'] else self.env['project.task']
            else:
                unvalid_milestone_tasks = self if not vals['milestone_id'] or milestone.project_id.id != vals['project_id'] else self.env['project.task']
            valid_milestone_tasks = self - unvalid_milestone_tasks
            if unvalid_milestone_tasks:
                unvalid_milestone_tasks.write({'milestone_id': False})
                if valid_milestone_tasks:
                    valid_milestone_tasks.write({'milestone_id': vals['milestone_id']})
                del vals['milestone_id']

            # 2. Parent's milestone is set to subtask with no milestone recursively
            subtasks_to_update = valid_milestone_tasks.child_ids.filtered(
                lambda task: (task not in self and \
                              not task.milestone_id and \
                              task.project_id == milestone.project_id and \
                              task.state not in CLOSED_STATES))

            # 3. If parent and child task share the same milestone, child task's milestone is updated when the parent one is changed
            # No need to check if state is changed in vals as it won't affect the subtasks selected for update
            if 'project_id' not in vals:
                subtasks_to_update |= valid_milestone_tasks.child_ids.filtered(
                    lambda task: (task not in self and \
                                  task.milestone_id == task.parent_id.milestone_id and \
                                  task.state not in CLOSED_STATES))
            else:
                subtasks_to_update |= valid_milestone_tasks.child_ids.filtered(
                    lambda task: (task not in self and \
                                  (not task.display_in_project or task.project_id.id == vals['project_id']) and \
                                  task.milestone_id == task.parent_id.milestone_id  and \
                                  task.state not in CLOSED_STATES))
            if subtasks_to_update:
                subtasks_to_update.write({'milestone_id': vals['milestone_id']})

        # stage change: update date_last_stage_update
        now = fields.Datetime.now()
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
                    recurrence = self.env['project.task.recurrence'].create(rec_values)
                    task.recurrence_id = recurrence.id

        if not vals.get('recurring_task', True) and self.recurrence_id:
            tasks_in_recurrence = self.recurrence_id.task_ids
            self.recurrence_id.unlink()
            tasks_in_recurrence.write({'recurring_task': False})

        # The sudo is required for a portal user as the record update
        # requires the write access on others models, as rating.rating
        # in order to keep the same name than the task.
        if portal_can_write:
            self_no_sudo, self = self, self.sudo().with_context(self._get_portal_sudo_context())
            vals_no_sudo, vals = self._get_portal_sudo_vals(vals)

        # Track user_ids to send assignment notifications
        old_user_ids = {t: t.user_ids for t in self.sudo()}

        if "personal_stage_type_id" in vals and not vals['personal_stage_type_id']:
            del vals['personal_stage_type_id']

        result = super().write(vals)
        if portal_can_write:
            super(Task, self_no_sudo).write(vals_no_sudo)

        if 'user_ids' in vals:
            self._populate_missing_personal_stages()

        # user_ids change: update date_assign
        if 'user_ids' in vals:
            for task in self:
                if not task.user_ids and task.date_assign:
                    task.date_assign = False
                elif 'date_assign' not in vals and task.id in task_ids_without_user_set:
                    task.date_assign = now

        # rating on stage
        if 'stage_id' in vals and vals.get('stage_id'):
            self.filtered(lambda x: x.project_id.rating_active and x.project_id.rating_status == 'stage')._send_task_rating_mail(force_send=True)

        if 'state' in vals:
            # specific use case: when the blocked task goes from 'forced' done state to a not closed state, we fix the state back to waiting
            for task in self:
                if task.allow_task_dependencies:
                    if task.is_blocked_by_dependences() and vals['state'] not in CLOSED_STATES and vals['state'] != '04_waiting_normal':
                        task.state = '04_waiting_normal'
                task.date_last_stage_update = now
        elif 'project_id' in vals:
            self.filtered(lambda t: t.state != '04_waiting_normal').state = '01_in_progress'

        self._task_message_auto_subscribe_notify({task: task.user_ids - old_user_ids[task] - self.env.user for task in self})
        return result

    def unlink(self):
        # Add subtasks to batch of tasks to delete
        self |= self._get_all_subtasks()
        last_task_id_per_recurrence_id = self.recurrence_id._get_last_task_id_per_recurrence_id()
        for task in self:
            if task.id == last_task_id_per_recurrence_id.get(task.recurrence_id.id):
                task.recurrence_id.unlink()
        return super().unlink()

    def update_date_end(self, stage_id):
        project_task_type = self.env['project.task.type'].browse(stage_id)
        if project_task_type.fold:
            return {'date_end': fields.Datetime.now()}
        return {'date_end': False}

    def _search_on_comodel(self, domain, field, comodel, order=None, additional_domain=None):
        """ This method is called by `group_expand` methods, whose purpose is to add empty groups to the `read_group`
            (which otherwise returns groups containing records that match the domain).
            When specifically filtering on a comodel's field, the result of the `read_group` should contain all matching groups.
            However, if the search isn't filtered on any comodel's field, the result shouldn't be affected,
            which explains why we return `False` if `filtered_domain` is empty.
        """
        def _change_operator(domain):
            new_domain = []
            for dom in domain:
                if len(dom) == 3:
                    _, op, value = dom
                    op = "ilike" if op == "child_of" else op
                    if isinstance(value, list) and all(isinstance(val, int) for val in value):
                        new_domain.append(("id", op, value))
                    if isinstance(value, str) or (isinstance(value, list) and not all(isinstance(val, str) for val in value)):
                        new_domain.append(("name", op, value))
                    if isinstance(value, int):
                        if op == "=":
                            op = "in"
                        if op == "!=":
                            op = "not in"
                        new_domain.append(("id", op, [value]))
                else:
                    new_domain.append(dom)
            return new_domain

        filtered_domain = filter_domain_leaf(domain, lambda field_to_check: field_to_check in [
            field,
            f"{field}.id",
            f"{field}.name",
        ], {
            field: "name",
            f"{field}.id": "id",
            f"{field}.name": "name",
        })
        filtered_domain = _change_operator(filtered_domain)
        if not filtered_domain:
            return self.env[comodel]
        if additional_domain:
            filtered_domain = expression.AND([filtered_domain, additional_domain])
        return self.env[comodel].search(filtered_domain, order=order)

    # ---------------------------------------------------
    # Subtasks
    # ---------------------------------------------------

    @api.depends('parent_id.partner_id', 'project_id')
    def _compute_partner_id(self):
        """ Compute the partner_id when the tasks have no partner_id.

            Use the project partner_id if any, or else the parent task partner_id.
        """
        for task in self:
            if task.partner_id and not (task.project_id or task.parent_id):
                task.partner_id = False
                continue
            if not task.partner_id:
                task.partner_id = self._get_default_partner_id(task.project_id, task.parent_id)

    @api.depends('project_id')
    def _compute_milestone_id(self):
        for task in self:
            if task.project_id != task.milestone_id.project_id:
                task.milestone_id = task.parent_id.project_id == task.project_id and task.parent_id.milestone_id

    def _compute_has_late_and_unreached_milestone(self):
        if all(not task.allow_milestones for task in self):
            self.has_late_and_unreached_milestone = False
            return
        late_milestones = self.env['project.milestone'].sudo()._search([  # sudo is needed for the portal user in Project Sharing.
            ('id', 'in', self.milestone_id.ids),
            ('is_reached', '=', False),
            ('deadline', '<=', fields.Date.today()),
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
        if self.stage_id:
            render_context['subtitles'].append(_('Stage: %s', self.stage_id.name))
        return render_context

    @api.model
    def _task_message_auto_subscribe_notify(self, users_per_task):
        if self.env.context.get('mail_auto_subscribe_no_notify'):
            return
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

    def _track_template(self, changes):
        res = super(Task, self)._track_template(changes)
        test_task = self[0]
        if 'stage_id' in changes and test_task.stage_id.mail_template_id:
            res['stage_id'] = (test_task.stage_id.mail_template_id, {
                'auto_delete_keep_log': False,
                'subtype_id': self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note'),
                'email_layout_xmlid': 'mail.mail_notification_light'
            })
        return res

    def _creation_subtype(self):
        return self.env.ref('project.mt_task_new')

    def _track_subtype(self, init_values):
        self.ensure_one()
        mail_message_subtype_per_state = {
            '1_done': 'project.mt_task_done',
            '1_canceled': 'project.mt_task_canceled',
            '01_in_progress': 'project.mt_task_in_progress',
            '03_approved': 'project.mt_task_approved',
            '02_changes_requested': 'project.mt_task_changes_requested',
            '04_waiting_normal': 'project.mt_task_waiting',
        }

        if 'stage_id' in init_values:
            return self.env.ref('project.mt_task_stage')
        elif 'state' in init_values and self.state in mail_message_subtype_per_state:
            return self.env.ref(mail_message_subtype_per_state[self.state])
        return super(Task, self)._track_subtype(init_values)

    def _mail_get_message_subtypes(self):
        res = super()._mail_get_message_subtypes()
        if not self.project_id.rating_active:
            res -= self.env.ref('project.mt_task_rating')
        if len(self) == 1:
            waiting_subtype = self.env.ref('project.mt_task_waiting')
            if ((self.project_id and not self.project_id.allow_task_dependencies)\
                or (not self.project_id and not self.user_has_groups('project.group_project_task_dependencies')))\
                and waiting_subtype in res:
                res -= waiting_subtype
        return res

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Handle project users and managers recipients that can assign
        tasks and create new one directly from notification emails. Also give
        access button to portal users and portal customers. If they are notified
        they should probably have access to the document. """
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        self.ensure_one()

        project_user_group_id = self.env.ref('project.group_project_user').id
        new_group = ('group_project_user', lambda pdata: pdata['type'] == 'user' and project_user_group_id in pdata['groups'], {})
        groups = [new_group] + groups

        if self.project_privacy_visibility == 'portal':
            groups.insert(0, (
                'allowed_portal_users',
                lambda pdata: pdata['type'] == 'portal',
                {
                    'active': True,
                    'has_button_access': True,
                }
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

    def _ensure_personal_stages(self):
        user = self.env.user
        ProjectTaskTypeSudo = self.env['project.task.type'].sudo()
        # In the case no stages have been found, we create the default stages for the user
        if not ProjectTaskTypeSudo.search_count([('user_id', '=', user.id)], limit=1):
            ProjectTaskTypeSudo.with_context(lang=user.lang, default_project_id=False).create(
                self.with_context(lang=user.lang)._get_default_personal_stage_create_vals(user.id)
            )

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
        create_context['mail_notify_author'] = True  # Allows sending stage updates to the author
        if custom_values is None:
            custom_values = {}
        # Auto create partner if not existant when the task is created from email
        if not msg.get('author_id') and msg.get('email_from'):
            msg['author_id'] = self.env['res.partner'].create({
                'email': msg['email_from'],
                'name': msg['email_from'],
            }).id

        defaults = {
            'name': msg.get('subject') or _("No Subject"),
            'allocated_hours': 0.0,
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
        return recipients

    def _notify_by_email_get_headers(self, headers=None):
        headers = super(Task, self)._notify_by_email_get_headers(headers=headers)
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

        # use the sanitized body of the email from the message thread to populate the task's description
        if (
           not self.description
           and message.subtype_id == self._creation_subtype()
           and self.partner_id == message.author_id
           and msg_vals['message_type'] == 'email'
        ):
            self.description = message.body
        return super(Task, self)._message_post_after_hook(message, msg_vals)

    def _get_projects_to_make_billable_domain(self, additional_domain=None):
        return expression.AND([
            [('partner_id', '!=', False)],
            additional_domain or [],
        ])

    def _get_all_subtasks(self):
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
                          WHERE t.parent_id IS NOT NULL
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
        action = self.with_context({
            'search_view_ref': 'project.project_sharing_project_task_view_search',
        }).action_open_parent_task()
        action['views'] = [(self.env.ref('project.project_sharing_project_task_view_form').id, 'form')]
        action['search_view_id'] = self.env.ref("project.project_sharing_project_task_view_search").id
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

    def action_unlink_recurrence(self):
        self.recurrence_id.task_ids.recurring_task = False
        self.recurrence_id.unlink()

    def action_convert_to_subtask(self):
        self.ensure_one()
        if self.project_id:
            return {
                'name': _('Convert to Task/Sub-Task'),
                'type': 'ir.actions.act_window',
                'res_model': 'project.task',
                'res_id': self.id,
                'views': [(self.env.ref('project.project_task_convert_to_subtask_view_form', False).id, 'form')],
                'target': 'new',
            }
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'danger',
                'message': _('Private tasks cannot be converted into sub-tasks. Please set a project for the task to gain access to this feature.'),
            }
        }

    # ---------------------------------------------------
    # Rating business
    # ---------------------------------------------------

    def _send_task_rating_mail(self, force_send=False):
        for task in self:
            rating_template = task.stage_id.rating_template_id
            partner = task.partner_id
            if rating_template and partner and partner != self.env.user.partner_id:
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
        if self.stage_id and self.stage_id.auto_validation_state:
            state = '03_approved' if rating.rating >= rating_data.RATING_LIMIT_SATISFIED else '02_changes_requested'
            self.write({'state': state})
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
        return self.analytic_account_id or self.project_id.analytic_account_id

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        calendar = self.env.company.resource_calendar_id
        return calendar._get_unusual_days(
            datetime.combine(fields.Date.from_string(date_from), time.min).replace(tzinfo=UTC),
            datetime.combine(fields.Date.from_string(date_to), time.max).replace(tzinfo=UTC)
        )

    def action_redirect_to_project_task_form(self):
        return {
            'type': 'ir.actions.act_url',
            'url': '/web#model=project.task&id=%s&action=%s&view_type=form' % (self.id, self.env.ref('project.action_view_my_task').id),
            'target': 'new',
        }

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        # A read_group can not be performed if records are grouped by personal_stage_type_id as it is a computed field.
        # personal_stage_type_ids behaves like a M2O from the point of view of the user, we therefore use this field instead.
        if 'personal_stage_type_id' in groupby and (not lazy or groupby[0] == 'personal_stage_type_id'):
            groupby = ["personal_stage_type_ids" if field == "personal_stage_type_id" else field for field in groupby] # limitation: problem when both personal_stage_type_id and personal_stage_type_ids appear in read_group, but this has no functional utility
            result = super().read_group(domain, fields, groupby, offset, limit, orderby, lazy)
            for group in result:
                group['personal_stage_type_id'] = group.pop('personal_stage_type_ids', False)
                group['personal_stage_type_id_count'] = group.pop('personal_stage_type_ids_count', 0)
            return result
        return super().read_group(domain, fields, groupby, offset, limit, orderby, lazy)

```

## File: models\project_task_recurrence.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models, Command
from odoo.exceptions import ValidationError

from dateutil.relativedelta import relativedelta

class ProjectTaskRecurrence(models.Model):
    _name = 'project.task.recurrence'
    _description = 'Task Recurrence'

    task_ids = fields.One2many('project.task', 'recurrence_id', copy=False)

    repeat_interval = fields.Integer(string='Repeat Every', default=1)
    repeat_unit = fields.Selection([
        ('day', 'Days'),
        ('week', 'Weeks'),
        ('month', 'Months'),
        ('year', 'Years'),
    ], default='week')
    repeat_type = fields.Selection([
        ('forever', 'Forever'),
        ('until', 'Until'),
    ], default="forever", string="Until")
    repeat_until = fields.Date(string="End Date")

    @api.constrains('repeat_interval')
    def _check_repeat_interval(self):
        if self.filtered(lambda t: t.repeat_interval <= 0):
            raise ValidationError(_('The interval should be greater than 0'))

    @api.constrains('repeat_type', 'repeat_until')
    def _check_repeat_until_date(self):
        today = fields.Date.today()
        if self.filtered(lambda t: t.repeat_type == 'until' and t.repeat_until < today):
            raise ValidationError(_('The end date should be in the future'))

    @api.model
    def _get_recurring_fields_to_copy(self):
        return [
            'analytic_account_id',
            'company_id',
            'description',
            'displayed_image_id',
            'email_cc',
            'message_partner_ids',
            'name',
            'parent_id',
            'partner_id',
            'allocated_hours',
            'project_id',
            'project_privacy_visibility',
            'recurrence_id',
            'recurring_task',
            'sequence',
            'tag_ids',
            'user_ids',
        ]

    @api.model
    def _get_recurring_fields_to_postpone(self):
        return [
            'date_deadline',
        ]

    def _get_last_task_id_per_recurrence_id(self):
        return {} if not self else {
            recurrence.id: max_task_id
            for recurrence, max_task_id in self.env['project.task'].sudo()._read_group(
                [('recurrence_id', 'in', self.ids)],
                ['recurrence_id'],
                ['id:max'],
            )
        }

    def _get_recurrence_delta(self):
        return relativedelta(**{
            f"{self.repeat_unit}s": self.repeat_interval
        })

    def _create_next_occurrence(self, occurrence_from):
        self.ensure_one()
        # Prevent double mail_followers creation
        self = self.with_context(mail_create_nosubscribe=True)
        create_values = self._create_next_occurrence_values(occurrence_from)
        date_deadline = create_values['date_deadline']
        if not (self.repeat_type == 'until' and date_deadline and date_deadline.date() > self.repeat_until):
            self.env['project.task'].sudo().create(create_values)

    def _create_next_occurrence_values(self, occurrence_from):
        self.ensure_one()
        fields_to_copy = occurrence_from.read(self._get_recurring_fields_to_copy()).pop()
        create_values = {
            field: value[0] if isinstance(value, tuple) else value
            for field, value in fields_to_copy.items()
        }

        fields_to_postpone = occurrence_from.read(self._get_recurring_fields_to_postpone()).pop()
        fields_to_postpone.pop('id', None)
        create_values.update({
            field: value and value + self._get_recurrence_delta()
            for field, value in fields_to_postpone.items()
        })

        create_values['stage_id'] = occurrence_from.project_id.type_ids[0].id if occurrence_from.project_id.type_ids else occurrence_from.stage_id.id
        create_values['child_ids'] = [
            Command.create(self._create_next_occurrence_values(child)) for child in occurrence_from.with_context(active_test=False).child_ids
        ]
        return create_values

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
    stage_id = fields.Many2one('project.task.type', domain="[('user_id', '=', user_id)]", ondelete='set null')

    _sql_constraints = [
        ('project_personal_stage_unique', 'UNIQUE (task_id, user_id)', 'A task can only have a single personal stage per user.'),
    ]

```

## File: models\project_task_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ProjectTaskType(models.Model):
    _name = 'project.task.type'
    _description = 'Task Stage'
    _order = 'sequence, id'

    def _get_default_project_ids(self):
        default_project_id = self.env.context.get('default_project_id')
        return [default_project_id] if default_project_id else None

    def _default_user_id(self):
        return not self.env.context.get('default_project_id', False) and self.env.uid

    active = fields.Boolean('Active', default=True)
    name = fields.Char(string='Name', required=True, translate=True)
    description = fields.Text(translate=True)
    sequence = fields.Integer(default=1)
    project_ids = fields.Many2many('project.project', 'project_task_type_rel', 'type_id', 'project_id', string='Projects',
        default=lambda self: self._get_default_project_ids(),
        help="Projects in which this stage is present. If you follow a similar workflow in several projects,"
            " you can share this stage among them and get consolidated information this way.")
    mail_template_id = fields.Many2one(
        'mail.template',
        string='Email Template',
        domain=[('model', '=', 'project.task')],
        help="If set, an email will be automatically sent to the customer when the task reaches this stage.")
    fold = fields.Boolean(string='Folded in Kanban')
    rating_template_id = fields.Many2one(
        'mail.template',
        string='Rating Email Template',
        domain=[('model', '=', 'project.task')],
        help="If set, a rating request will automatically be sent by email to the customer when the task reaches this stage. \n"
             "Alternatively, it will be sent at a regular interval as long as the task remains in this stage, depending on the configuration of your project. \n"
             "To use this feature make sure that the 'Customer Ratings' option is enabled on your project.")
    auto_validation_state = fields.Boolean('Automatic Kanban Status', default=False,
        help="Automatically modify the state when the customer replies to the feedback for this stage.\n"
            " * Good feedback from the customer will update the state to 'Approved' (green bullet).\n"
            " * Neutral or bad feedback will set the kanban state to 'Changes Requested' (orange bullet).\n")
    disabled_rating_warning = fields.Text(compute='_compute_disabled_rating_warning')

    user_id = fields.Many2one('res.users', 'Stage Owner', default=_default_user_id, compute='_compute_user_id', store=True, index=True)

    def unlink_wizard(self, stage_view=False):
        self = self.with_context(active_test=False)
        # retrieves all the projects with a least 1 task in that stage
        # a task can be in a stage even if the project is not assigned to the stage
        readgroup = self.with_context(active_test=False).env['project.task']._read_group([('stage_id', 'in', self.ids)], ['project_id'])
        project_ids = list(set([project.id for [project] in readgroup] + self.project_ids.ids))

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

    def copy(self, default=None):
        default = dict(default or {})
        if not default.get('name'):
            default['name'] = _("%s (copy)", self.name)
        return super().copy(default)

    @api.ondelete(at_uninstall=False)
    def _unlink_if_remaining_personal_stages(self):
        """ Prepare personal stages for deletion (i.e. move task to other personal stages) and
            avoid unlink if no remaining personal stages for an active internal user.
        """
        # Personal stages are processed if the user still has at least one personal stage after unlink
        personal_stages = self.filtered('user_id')
        if not personal_stages:
            return
        remaining_personal_stages_all = self.env['project.task.type']._read_group(
            [('user_id', 'in', personal_stages.user_id.ids), ('id', 'not in', personal_stages.ids)],
            groupby=['user_id', 'sequence', 'id'],
            order="user_id,sequence DESC",
        )
        remaining_personal_stages_by_user = defaultdict(list)
        for user, sequence, stage in remaining_personal_stages_all:
            remaining_personal_stages_by_user[user].append({'id': stage.id, 'seq': sequence})

        # For performance issue, project.task.stage.personal records that need to be modified are listed before calling _prepare_personal_stages_deletion
        personal_stages_to_update = self.env['project.task.stage.personal']._read_group([('stage_id', 'in', personal_stages.ids)], ['stage_id'], ['id:recordset'])
        for user in personal_stages.user_id:
            if not user.active or user.share:
                continue
            user_stages_to_unlink = personal_stages.filtered(lambda stage: stage.user_id == user)
            user_remaining_stages = remaining_personal_stages_by_user[user]
            if not user_remaining_stages:
                raise UserError(_("Each user should have at least one personal stage. Create a new stage to which the tasks can be transferred after the selected ones are deleted."))
            user_stages_to_unlink._prepare_personal_stages_deletion(user_remaining_stages, personal_stages_to_update)

    def _prepare_personal_stages_deletion(self, remaining_stages_dict, personal_stages_to_update):
        """ _prepare_personal_stages_deletion prepare the deletion of personal stages of a single user.
            Tasks using that stage will be moved to the first stage with a lower sequence if it exists
            higher if not.
        :param self: project.task.type recordset containing the personal stage of a user
                     that need to be deleted
        :param remaining_stages_dict: list of dict representation of the personal stages of a user that
                                      can be used to replace the deleted ones. Can not be empty.
                                      e.g: [{'id': stage1_id, 'seq': stage1_sequence}, ...]
        :param personal_stages_to_update: project.task.stage.personal recordset containing the records
                                          that need to be updated after stage modification. Is passed to
                                          this method as an argument to avoid to reload it for each users
                                          when this method is called multiple times.
        """
        stages_to_delete_dict = sorted([{'id': stage.id, 'seq': stage.sequence} for stage in self],
                                       key=lambda stage: stage['seq'])
        replacement_stage_id = remaining_stages_dict.pop()['id']
        next_replacement_stage = remaining_stages_dict and remaining_stages_dict.pop()

        personal_stages_by_stage = {
            stage.id: personal_stages
            for stage, personal_stages in personal_stages_to_update
        }
        for stage in stages_to_delete_dict:
            while next_replacement_stage and next_replacement_stage['seq'] < stage['seq']:
                replacement_stage_id = next_replacement_stage['id']
                next_replacement_stage = remaining_stages_dict and remaining_stages_dict.pop()
            if stage['id'] in personal_stages_by_stage:
                personal_stages_by_stage[stage['id']].stage_id = replacement_stage_id

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

    @api.depends('project_ids')
    def _compute_user_id(self):
        """ Fields project_ids and user_id cannot be set together for a stage. It can happen that
            project_ids is set after stage creation (e.g. when setting demo data). In such case, the
            default user_id has to be removed.
        """
        self.sudo().filtered('project_ids').user_id = False

    @api.constrains('user_id', 'project_ids')
    def _check_personal_stage_not_linked_to_projects(self):
        if any(stage.user_id and stage.project_ids for stage in self):
            raise UserError(_('A personal stage cannot be linked to a project because it is only visible to its corresponding user.'))

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
    'at_risk': 22,  # orange
    'off_track': 23,  # red / danger
    'on_hold': 21,  # light blue
    'done': 24,  # purple
    False: 0,  # default grey -- for studio
    # Only used in project.task
    'to_define': 0,
}

class ProjectUpdate(models.Model):
    _name = 'project.update'
    _description = 'Project Update'
    _order = 'id desc'
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
        ('on_hold', 'On Hold'),
        ('done', 'Done'),
    ], required=True, tracking=True)
    color = fields.Integer(compute='_compute_color')
    progress = fields.Integer(tracking=True)
    progress_percentage = fields.Float(compute='_compute_progress_percentage')
    user_id = fields.Many2one('res.users', string='Author', required=True, default=lambda self: self.env.user)
    description = fields.Html()
    date = fields.Date(default=fields.Date.context_today, tracking=True)
    project_id = fields.Many2one('project.project', required=True)
    name_cropped = fields.Char(compute="_compute_name_cropped")
    task_count = fields.Integer("Task Count", readonly=True)
    closed_task_count = fields.Integer("Closed Task Count", readonly=True)
    closed_task_percentage = fields.Integer("Closed Task Percentage", compute="_compute_closed_task_percentage")

    @api.depends('status')
    def _compute_color(self):
        for update in self:
            update.color = STATUS_COLOR[update.status]

    @api.depends('progress')
    def _compute_progress_percentage(self):
        for update in self:
            update.progress_percentage = update.progress / 100

    @api.depends('name')
    def _compute_name_cropped(self):
        for update in self:
            update.name_cropped = (update.name[:57] + '...') if len(update.name) > 60 else update.name

    def _compute_closed_task_percentage(self):
        for update in self:
            update.closed_task_percentage = update.task_count and round(update.closed_task_count * 100 / update.task_count)

    # ---------------------------------
    # ORM Override
    # ---------------------------------
    @api.model_create_multi
    def create(self, vals_list):
        updates = super().create(vals_list)
        for update in updates:
            project = update.project_id
            project.sudo().last_update_id = update
            update.write({
                "task_count": project.task_count,
                "closed_task_count": project.closed_task_count,
            })
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
                         ON mtv.field_id = imf.id
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

    module_hr_timesheet = fields.Boolean(string="Task Logs")
    group_project_rating = fields.Boolean("Customer Ratings", implied_group='project.group_project_rating')
    group_project_stages = fields.Boolean("Project Stages", implied_group="project.group_project_stages")
    group_project_recurring_tasks = fields.Boolean("Recurring Tasks", implied_group="project.group_project_recurring_tasks")
    group_project_task_dependencies = fields.Boolean("Task Dependencies", implied_group="project.group_project_task_dependencies")
    group_project_milestone = fields.Boolean('Milestones', implied_group='project.group_project_milestone', group='base.group_portal,base.group_user')

    # Analytic Accounting
    analytic_plan_id = fields.Many2one(
        comodel_name='account.analytic.plan',
        string="Analytic Plan",
        config_parameter="analytic.analytic_plan_projects",
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

        task_waiting_subtype_id = self.env.ref('project.mt_task_waiting')
        project_task_waiting_subtype_id = self.env.ref('project.mt_project_task_waiting')
        if task_waiting_subtype_id.hidden != (not self['group_project_task_dependencies']):
            task_waiting_subtype_id.hidden = not self['group_project_task_dependencies']
            project_task_waiting_subtype_id.hidden = not self['group_project_task_dependencies']
        # Hide Project Stage Changed mail subtype according to the settings
        project_stage_change_mail_type = self.env.ref('project.mt_project_stage_change')
        if project_stage_change_mail_type.hidden == self['group_project_stages']:
            project_stage_change_mail_type.hidden = not self['group_project_stages']
        # Hide task rating tempalate when customer rating is disbled
        task_rating_subtype_id = self.env.ref('project.mt_project_task_rating')
        task_rating_subtype_id.hidden = not self['group_project_rating']
        self.env.ref('project.mt_task_rating').hidden = not self['group_project_rating']
        task_rating_subtype_id.default = self['group_project_rating']
        rating_project_request_email_template = self.env.ref('project.rating_project_request_email_template')
        if rating_project_request_email_template.active != self['group_project_rating']:
            rating_project_request_email_template.active = self['group_project_rating']
        if not self['group_project_recurring_tasks']:
            self.env['project.task'].sudo().search([('recurring_task', '=', True)]).write({'recurring_task': False})

        super().set_values()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import email_normalize


class ResPartner(models.Model):
    """ Inherits partner and adds Tasks information in the partner form """
    _inherit = 'res.partner'

    project_ids = fields.One2many('project.project', 'partner_id', string='Projects')
    task_ids = fields.One2many('project.task', 'partner_id', string='Tasks')
    task_count = fields.Integer(compute='_compute_task_count', string='# Tasks')

    @api.constrains('company_id', 'project_ids')
    def _ensure_same_company_than_projects(self):
        for partner in self:
            if partner.company_id and partner.project_ids.company_id and partner.project_ids.company_id != partner.company_id:
                raise UserError(_("Partner company cannot be different from its assigned projects' company"))

    @api.constrains('company_id', 'task_ids')
    def _ensure_same_company_than_tasks(self):
        for partner in self:
            if partner.company_id and partner.task_ids.company_id and partner.task_ids.company_id != partner.company_id:
                raise UserError(_("Partner company cannot be different from its assigned tasks' company"))

    def _compute_task_count(self):
        # retrieve all children partners and prefetch 'parent_id' on them
        all_partners = self.with_context(active_test=False).search_fetch(
            [('id', 'child_of', self.ids)],
            ['parent_id'],
        )
        task_data = self.env['project.task']._read_group(
            domain=[('partner_id', 'in', all_partners.ids)],
            groupby=['partner_id'], aggregates=['__count']
        )
        self_ids = set(self._ids)

        self.task_count = 0
        for partner, count in task_data:
            while partner:
                if partner.id in self_ids:
                    partner.task_count += count
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

from . import account_analytic_account
from . import mail_message
from . import project_project_stage
from . import project_task_recurrence
# `project_task_stage_personal` has to be loaded before `project_project` and `project_milestone`
from . import project_task_stage_personal
from . import project_milestone
from . import project_project
from . import project_task
from . import project_task_type
from . import project_tags
from . import project_collaborator
from . import project_update
from . import res_config_settings
from . import res_partner
from . import digest_digest
from . import ir_ui_menu

```

## File: populate\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import collections

from odoo import models, Command
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
    _populate_dependencies = ["project.project", "res.partner", "res.users"]

    def _populate_factories(self):
        project_ids = self.env.registry.populated_models["project.project"]
        stage_ids = self.env.registry.populated_models["project.task.type"]
        user_ids = self.env.registry.populated_models['res.users']
        partner_ids_per_company_id = {
            company.id: ids
            for company, ids in self.env['res.partner']._read_group(
                [('company_id', '!=', False), ('id', 'in', self.env.registry.populated_models["res.partner"])],
                ['company_id'],
                ['id:array_agg'],
            )
        }
        company_id_per_project_id = {
            record['id']: record['company_id']
            for record in self.env['project.project'].search_read(
                [('company_id', '!=', False), ('id', 'in', self.env.registry.populated_models["project.project"])],
                ['company_id'],
                load=False
            )
        }
        partner_ids_per_project_id = {
            project_id: partner_ids_per_company_id[company_id]
            for project_id, company_id in company_id_per_project_id.items()
        }
        def get_project_id(random, **kwargs):
            return random.choice([False, False, False] + project_ids)
        def get_stage_id(random, **kwargs):
            return random.choice([False, False] + stage_ids)
        def get_partner_id(random, **kwargs):
            project_id = kwargs['values'].get('project_id')
            partner_ids = partner_ids_per_project_id.get(project_id, False)
            return partner_ids and random.choice(partner_ids + [False] * len(partner_ids))
        def get_user_ids(values, counter, random):
            return [Command.set([random.choice(user_ids) for i in range(random.randint(0, 3))])]

        return [
            ("name", populate.constant('project_task_{counter}')),
            ("sequence", populate.randomize([False] + [i for i in range(1, 101)])),
            ("active", populate.randomize([True, False], [0.8, 0.2])),
            ("color", populate.randomize([False] + [i for i in range(1, 7)])),
            ("state", populate.randomize(['01_in_progress', '03_approved', '02_changes_requested', '1_done', '1_canceled'])),
            ("project_id", populate.compute(get_project_id)),
            ("stage_id", populate.compute(get_stage_id)),
            ('partner_id', populate.compute(get_partner_id)),
            ('user_ids', populate.compute(get_user_ids)),
        ]

    def _populate(self, size):
        records = super()._populate(size)
        # set parent_ids
        self._populate_set_children_tasks(records)
        return records

    def _populate_set_children_tasks(self, tasks):
        _logger.info('Setting parent tasks')
        rand = populate.Random('project.task+children_generator')
        task_ids_per_company = collections.defaultdict(set)
        for task in tasks:
            if task.project_id:
                task_ids_per_company[task.company_id].add(task.id)

        for task_ids in task_ids_per_company.values():
            parent_ids = set()
            for task_id in task_ids:
                if not rand.getrandbits(4):
                    parent_ids.add(task_id)
            child_ids = task_ids - parent_ids
            parent_ids = list(parent_ids)

            child_ids_per_parent_id = collections.defaultdict(set)
            for child_id in child_ids:
                if not rand.getrandbits(4):
                    child_ids_per_parent_id[rand.choice(parent_ids)].add(child_id)

            for count, (parent_id, child_ids) in enumerate(child_ids_per_parent_id.items()):
                if (count + 1) % 100 == 0:
                    _logger.info('Setting parent: %s/%s', count + 1, len(child_ids_per_parent_id))
                self.browse(child_ids).write({'parent_id': parent_id})

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
    date_deadline = fields.Datetime(string='Deadline', readonly=True)
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
    rating_last_value = fields.Float('Rating (/5)', group_operator="avg", readonly=True, groups="project.group_project_rating")
    rating_avg = fields.Float('Average Rating', readonly=True, group_operator='avg', groups="project.group_project_rating")
    priority = fields.Selection([
        ('0', 'Low'),
        ('1', 'High')
        ], readonly=True, string="Priority")

    state = fields.Selection([
        ('01_in_progress', 'In Progress'),
        ('1_done', 'Done'),
        ('04_waiting_normal', 'Waiting'),
        ('03_approved', 'Approved'),
        ('1_canceled', 'Canceled'),
        ('02_changes_requested', 'Changes Requested'),
    ], string='State', readonly=True)
    company_id = fields.Many2one('res.company', string='Company', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    stage_id = fields.Many2one('project.task.type', string='Stage', readonly=True)
    task_id = fields.Many2one('project.task', string='Tasks', readonly=True)
    active = fields.Boolean(readonly=True)
    tag_ids = fields.Many2many('project.tags', relation='project_tags_project_task_rel',
        column1='project_task_id', column2='project_tags_id',
        string='Tags', readonly=True)
    parent_id = fields.Many2one('project.task', string='Parent Task', readonly=True)
    personal_stage_type_ids = fields.Many2many('project.task.type', relation='project_task_user_rel',
        column1='task_id', column2='stage_id',
        string="Personal Stage", readonly=True)
    milestone_id = fields.Many2one('project.milestone', readonly=True)
    message_is_follower = fields.Boolean(related='task_id.message_is_follower')
    dependent_ids = fields.Many2many('project.task', relation='task_dependencies_rel', column1='depends_on_id',
        column2='task_id', string='Block', readonly=True,
        domain="[('allow_task_dependencies', '=', True), ('id', '!=', id)]")
    description = fields.Text(readonly=True)

    def _select(self):
        return """
                (select 1) AS nbr,
                t.id as id,
                t.id as task_id,
                t.active,
                t.create_date,
                t.date_assign,
                t.date_end,
                t.date_last_stage_update,
                t.date_deadline,
                t.project_id,
                t.priority,
                t.name as name,
                t.company_id,
                t.partner_id,
                t.parent_id,
                t.stage_id,
                t.state,
                t.milestone_id,
                CASE WHEN pm.id IS NOT NULL THEN true ELSE false END as has_late_and_unreached_milestone,
                t.description,
                NULLIF(t.rating_last_value, 0) as rating_last_value,
                AVG(rt.rating) as rating_avg,
                t.working_days_close,
                t.working_days_open,
                t.working_hours_open,
                t.working_hours_close,
                (extract('epoch' from (t.date_deadline-(now() at time zone 'UTC'))))/(3600*24) as delay_endings_days,
                COUNT(td.task_id) as dependent_ids_count
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
                t.priority,
                t.name,
                t.company_id,
                t.partner_id,
                t.parent_id,
                t.stage_id,
                t.state,
                t.rating_last_value,
                t.working_days_close,
                t.working_days_open,
                t.working_hours_open,
                t.working_hours_close,
                t.milestone_id,
                pm.id,
                td.depends_on_id
        """

    def _from(self):
        return f"""
                project_task t
                    LEFT JOIN rating_rating rt ON rt.res_id = t.id
                          AND rt.res_model = 'project.task'
                          AND rt.consumed = True
                          AND rt.rating >= {RATING_LIMIT_MIN}
                    LEFT JOIN project_milestone pm ON pm.id = t.milestone_id
                          AND pm.is_reached = False
                          AND pm.deadline <= CAST(now() AS DATE)
                    LEFT JOIN task_dependencies_rel td ON td.depends_on_id = t.id
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
                <pivot string="Tasks Analysis" display_quantity="1" sample="1" disable_linking="1">
                    <field name="project_id" type="row"/>
                    <field name="working_hours_open" widget="timesheet_uom"/>
                    <field name="working_hours_close" widget="timesheet_uom"/>
                    <field name="nbr" invisible="1"/>
                    <field name="rating_avg" invisible="1"/>
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
                     <field name="working_hours_open" widget="float_time"/>
                     <field name="working_hours_close" widget="float_time"/>
                     <field name="rating_avg" invisible="1"/>
                 </graph>
             </field>
        </record>

        <record id="view_task_project_user_search" model="ir.ui.view">
            <field name="name">report.project.task.user.search</field>
            <field name="model">report.project.task.user</field>
            <field name="inherit_id" ref="project.view_task_search_form_project_fsm_base"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <search position="attributes">
                    <attribute name="string">Tasks Analysis</attribute>
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
from odoo.exceptions import UserError
from odoo.tools import SQL
from odoo.addons.resource.models.utils import filter_domain_leaf


class ReportProjectTaskBurndownChart(models.AbstractModel):
    _name = 'project.task.burndown.chart.report'
    _description = 'Burndown Chart'
    _auto = False
    _order = 'date'

    allocated_hours = fields.Float(string='Allocated Time', readonly=True)
    date = fields.Datetime('Date', readonly=True)
    date_assign = fields.Datetime(string='Assignment Date', readonly=True)
    date_deadline = fields.Date(string='Deadline', readonly=True)
    date_last_stage_update = fields.Date(string='Last Stage Update', readonly=True)
    state = fields.Selection([
        ('01_in_progress', 'In Progress'),
        ('1_done', 'Done'),
        ('04_waiting_normal', 'Waiting'),
        ('03_approved', 'Approved'),
        ('1_canceled', 'Canceled'),
        ('02_changes_requested', 'Changes Requested'),
    ], string='State', readonly=True)
    milestone_id = fields.Many2one('project.milestone', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    project_id = fields.Many2one('project.project', readonly=True)
    stage_id = fields.Many2one('project.task.type', readonly=True)
    tag_ids = fields.Many2many('project.tags', relation='project_tags_project_task_rel',
                               column1='project_task_id', column2='project_tags_id',
                               string='Tags', readonly=True)
    user_ids = fields.Many2many('res.users', relation='project_task_user_rel', column1='task_id', column2='user_id',
                                string='Assignees', readonly=True)

    # This variable is used in order to distinguish conditions that can be set on `project.task` and thus being used
    # at a lower level than the "usual" query made by the `read_group_raw`. Indeed, the domain applied on those fields
    # will be performed on a `CTE` that will be later use in the `SQL` in order to limit the subset of data that is used
    # in the successive `GROUP BY` statements.
    @property
    def task_specific_fields(self):
        return [
            'date_assign',
            'date_deadline',
            'date_last_stage_update',
            'state',
            'milestone_id',
            'partner_id',
            'project_id',
            'stage_id',
            'tag_ids',
            'user_ids',
        ]

    def _where_calc(self, domain, active_test=True):
        burndown_specific_domain, task_specific_domain = self._determine_domains(domain)

        main_query = super()._where_calc(burndown_specific_domain, active_test)

        # Build the query on `project.task` with the domain fields that are linked to that model. This is done in order
        # to be able to reduce the number of treated records in the query by limiting them to the one corresponding to
        # the ids that are returned from this sub query.
        self.env['project.task']._flush_search(task_specific_domain, fields=self.task_specific_fields)
        project_task_query = self.env['project.task']._where_calc(task_specific_domain)
        project_task_from_clause, project_task_where_clause, project_task_where_clause_params = project_task_query.get_sql()

        # Get the stage_id `ir.model.fields`'s id in order to inject it directly in the query and avoid having to join
        # on `ir_model_fields` table.
        IrModelFieldsSudo = self.env['ir.model.fields'].sudo()
        field_id = IrModelFieldsSudo.search([('name', '=', 'stage_id'), ('model', '=', 'project.task')]).id

        groupby = self.env.context.get('project_task_burndown_chart_report_groupby', ['date:month', 'stage_id'])
        date_groupby = [g for g in groupby if g.startswith('date')][0]

        # Computes the interval which needs to be used in the `SQL` depending on the date group by interval.
        interval = date_groupby.split(':')[1]
        sql_interval = '1 %s' % interval if interval != 'quarter' else '3 month'

        simple_date_groupby_sql, __ = self._read_group_groupby(f"date:{interval}", main_query)
        # Removing unexistant table name from the expression
        simple_date_groupby_sql = self.env.cr.mogrify(simple_date_groupby_sql).decode()
        simple_date_groupby_sql = simple_date_groupby_sql.replace('"project_task_burndown_chart_report".', '')

        burndown_chart_query = """
            (
              WITH task_ids AS (
                 SELECT id
                 FROM %(task_query_from)s
                 %(task_query_where)s
              ),
              all_stage_task_moves AS (
                 SELECT count(*) as __count,
                        sum(allocated_hours) as allocated_hours,
                        project_id,
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
                                   allocated_hours,
                                   project_id,
                                   %(date_begin)s as date_begin,
                                   %(date_end)s as date_end,
                                   first_value(stage_id) OVER task_date_begin_window AS stage_id
                              FROM (
                                     SELECT pt.id as task_id,
                                            pt.allocated_hours,
                                            pt.project_id,
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
                                                                                     AND mtv.field_id = %(field_id)s
                                                                                     AND mm.model='project.task'
                                                                                     AND mm.message_type = 'notification'
                                                        JOIN project_task_type ptt ON ptt.id = mtv.old_value_integer
                                                ) ON mm.res_id = pt.id
                                      WHERE pt.active=true AND pt.id IN (SELECT id from task_ids)
                                   ) task_stage_id_history
                          GROUP BY task_id,
                                   allocated_hours,
                                   project_id,
                                   %(date_begin)s,
                                   %(date_end)s,
                                   stage_id
                            WINDOW task_date_begin_window AS (PARTITION BY task_id, %(date_begin)s)
                          UNION ALL
                            -- Gathers the current stage_ids per task_id for those which values changed at least
                            -- once (=those for which we have at least a mail message and a mail tracking value
                            -- on project.task stage_id).
                            SELECT pt.id as task_id,
                                   pt.allocated_hours,
                                   pt.project_id,
                                   last_stage_id_change_mail_message.date as date_begin,
                                   (now() at time zone 'utc')::date + INTERVAL '%(interval)s' as date_end,
                                   pt.stage_id as old_value_integer
                              FROM project_task pt
                                   JOIN project_task_type ptt ON ptt.id = pt.stage_id
                                   JOIN LATERAL (
                                       SELECT mm.date
                                       FROM mail_message mm
                                       JOIN mail_tracking_value mtv ON mm.id = mtv.mail_message_id
                                       AND mtv.field_id = %(field_id)s
                                       AND mm.model='project.task'
                                       AND mm.message_type = 'notification'
                                       AND mm.res_id = pt.id
                                       ORDER BY mm.id DESC
                                       FETCH FIRST ROW ONLY
                                   ) AS last_stage_id_change_mail_message ON TRUE
                             WHERE pt.active=true AND pt.id IN (SELECT id from task_ids)
                        ) AS project_task_burndown_chart
               GROUP BY allocated_hours,
                        project_id,
                        %(date_begin)s,
                        %(date_end)s,
                        stage_id
              )
              SELECT (project_id*10^13 + stage_id*10^7 + to_char(date, 'YYMMDD')::integer)::bigint as id,
                     allocated_hours,
                     project_id,
                     stage_id,
                     date,
                     __count
                FROM all_stage_task_moves t
                         JOIN LATERAL generate_series(t.date_begin, t.date_end-INTERVAL '1 day', '%(interval)s')
                            AS date ON TRUE
            )
        """ % {
            'task_query_from': project_task_from_clause,
            'task_query_where': f'WHERE {project_task_where_clause}' if project_task_where_clause else '',
            'date_begin': simple_date_groupby_sql.replace('"date"', '"date_begin"'),
            'date_end': simple_date_groupby_sql.replace('"date"', '"date_end"'),
            'interval': sql_interval,
            'field_id': field_id,
        }

        # hardcode 'project_task_burndown_chart_report' as the query above
        # (with its own parameters)
        burndown_chart_sql = SQL(burndown_chart_query, *project_task_where_clause_params)
        main_query._tables['project_task_burndown_chart_report'] = burndown_chart_sql

        return main_query

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

        See `filter_domain_leaf` for more details on the new domains.

        :param domain: The domain that has been passed to the read_group.
        :return: A tuple containing the non `project.task` specific domain and the `project.task` specific domain.
        """
        burndown_chart_specific_fields = list(set(self._fields) - set(self.task_specific_fields))
        task_specific_domain = filter_domain_leaf(domain, lambda field: field not in burndown_chart_specific_fields)
        non_task_specific_domain = filter_domain_leaf(domain, lambda field: field not in self.task_specific_fields)
        return non_task_specific_domain, task_specific_domain

    def _read_group_select(self, aggregate_spec, query):
        if aggregate_spec == '__count':
            return SQL("SUM(%s)", SQL.identifier(self._table, '__count')), []
        return super()._read_group_select(aggregate_spec, query)

    def _read_group(self, domain, groupby=(), aggregates=(), having=(), offset=0, limit=None, order=None):
        self._validate_group_by(groupby)
        self = self.with_context(project_task_burndown_chart_report_groupby=groupby)

        return super()._read_group(
            domain=domain, groupby=groupby, aggregates=aggregates,
            having=having, offset=offset, limit=limit, order=order,
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
                <field name="tag_ids"/>
                <field name="user_ids"/>
                <field name="stage_id" />
                <field name="project_id" />
                <field name="milestone_id" groups="project.group_project_milestone"/>
                <field name="partner_id" filter_domain="[('partner_id', 'child_of', self)]"/>
                <separator/>
                <filter string="My Tasks" name="my_tasks" domain="[('user_ids', 'in', uid)]"/>
                <filter string="Unassigned" name="unassigned" domain="[('user_ids', '=', False)]"/>
                <separator/>
                <filter name="filter_date" date="date" string="Date" default_period="this_year,last_year" />
                <filter name="filter_last_stage_update" date="date_last_stage_update"/>
                <filter name="filter_date_deadline" date="date_deadline"/>
                <filter string="Last Month" invisible="1" name="last_month" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=30)).strftime('%Y-%m-%d'))]"/>
                <separator/>
                <filter string="Open Tasks" name="open_tasks" domain="[('state', 'in', ['01_in_progress', '02_changes_requested', '03_approved', '04_waiting_normal'])]"/>
                <filter string="Closed Tasks" name="closed_tasks" domain="[('state', 'in', ['1_done', '1_canceled'])]"/>
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
        <field name="context">{'search_default_project_id': active_id, 'search_default_date': 1, 'search_default_stage': 1, 'search_default_filter_date': 1}</field>
        <field name="domain">[('project_id', '!=', False)]</field>
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
access_report_project_task_user,report.project.task.user,model_report_project_task_user,project.group_project_manager,1,0,0,0
access_report_project_task_user_project_user,report.project.task.user.project.user,model_report_project_task_user,project.group_project_user,1,0,0,0
access_partner_task_user,base.res.partner user,base.model_res_partner,project.group_project_user,1,0,0,0
access_task_on_partner,project.task on partners,model_project_task,base.group_user,1,0,0,0
access_task_portal,task_portal,project.model_project_task,base.group_portal,1,0,0,0
access_project_user,project.project on partners,model_project_project,base.group_user,1,0,0,0
access_project_portal,project_portal,project.model_project_project,base.group_portal,1,0,0,0
access_resource_calendar,project.resource_calendar user,resource.model_resource_calendar,project.group_project_user,1,0,0,0
access_resource_calendar_attendance,project.resource_calendar_attendance user,resource.model_resource_calendar_attendance,project.group_project_user,1,0,0,0
access_resource_calendar_leaves_user,resource.calendar.leaves user,resource.model_resource_calendar_leaves,project.group_project_user,1,1,1,1
access_project_tags_all,project.project_tags_all,model_project_tags,base.group_user,1,0,0,0
access_project_tags_manager,project.project_tags_manager,model_project_tags,project.group_project_manager,1,1,1,1
access_project_tags_portal,project_tags_portal,project.model_project_tags,base.group_portal,1,0,0,0
access_mail_activity_type_project_manager,mail.activity.type.project.manager,mail.model_mail_activity_type,project.group_project_manager,1,1,1,1
access_account_analytic_account_user,account.analytic.account,analytic.model_account_analytic_account,project.group_project_user,1,0,0,0
access_account_analytic_account_manager,account.analytic.account,analytic.model_account_analytic_account,project.group_project_manager,1,1,1,1
access_account_analytic_line_project,account.analytic.line project,analytic.model_account_analytic_line,project.group_project_manager,1,1,1,1
access_project_task_type_delete_wizard,project.task.type.delete.wizard,model_project_task_type_delete_wizard,project.group_project_manager,1,1,1,1
access_project_project_stage_delete_wizard,project.project.stage.delete.wizard,model_project_project_stage_delete_wizard,project.group_project_manager,1,1,1,1
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
access_mail_activity_plan_project_manager,mail.activity.plan.project.manager,mail.model_mail_activity_plan,project.group_project_manager,1,1,1,1
access_mail_activity_plan_template_project_manager,mail.activity.plan.template.project.manager,mail.model_mail_activity_plan_template,project.group_project_manager,1,1,1,1

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
        <field name="implied_ids" eval="[(4, ref('group_project_user')), (4, ref('mail.group_mail_template_editor'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
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
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="project_project_stage_rule">
        <field name="name">Project Stage: multi-company</field>
        <field name="model_id" ref="model_project_project_stage"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
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
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
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
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
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
        <field name="domain_force">['|', ('project_id.company_id', 'in', company_ids), ('project_id.company_id', '=', False)]</field>
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
        <field name="domain_force">['|', ('project_id.company_id', 'in', company_ids), ('project_id.company_id', '=', False)]</field>
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

    <record id="mail_plan_rule_group_project_manager_task" model="ir.rule">
        <field name="name">Manager can manage project/task plans</field>
        <field name="groups" eval="[(4, ref('group_project_manager'))]"/>
        <field name="model_id" ref="mail.model_mail_activity_plan"/>
        <field name="domain_force">[('res_model', 'in', ('project.project', 'project.task'))]</field>
        <field name="perm_read" eval="False"/>
    </record>

    <record id="mail_plan_templates_rule_group_project_manager_task" model="ir.rule">
        <field name="name">Manager can manage project/task plan templates</field>
        <field name="groups" eval="[(4, ref('group_project_manager'))]"/>
        <field name="model_id" ref="mail.model_mail_activity_plan_template"/>
        <field name="domain_force">[('plan_id.res_model', 'in', ('project.project', 'project.task'))]</field>
        <field name="perm_read" eval="False"/>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="m30.842 34.612-8.105 10.387L5.452 31.37a3.862 3.862 0 0 1-.616-5.417l5.748-7.243 20.258 15.903Z" fill="#985184"/><path d="m22.623 44.909-10.455-8.335 8.128-10.242 10.547 8.28L22.738 45l-.115-.091Z" fill="#005E7A"/><path d="m22.593 44.886.144.114 22.447-28.767a3.862 3.862 0 0 0-.636-5.393L37.223 5 20.296 26.332l10.547 8.28-8.105 10.387-.144-.113Z" fill="#1AD3BB"/></svg>

```

## File: static\src\components\delete_subtasks_confirmation_dialog\delete_subtasks_confirmation_dialog.js

```javascript
/** @odoo-module **/

import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";

export class DeleteSubtasksConfirmationDialog extends ConfirmationDialog {}

DeleteSubtasksConfirmationDialog.props = {
    ...ConfirmationDialog.props,
    body: { String, optional: true },
}

DeleteSubtasksConfirmationDialog.defaultProps = {
    ...ConfirmationDialog.defaultProps,
    body: _t("Deleting a task will also delete its associated sub-tasks. If you wish to preserve the sub-tasks, make sure to unlink them from their parent task beforehand. Are you sure you want to proceed?"),
    confirmLabel: _t("Delete"),
    cancel: () => {},
};

```

## File: static\src\components\project_control_panel\project_control_panel.js

```javascript
/** @odoo-module **/

import { ControlPanel } from "@web/search/control_panel/control_panel";
import { useService } from "@web/core/utils/hooks";
import { onWillStart } from "@odoo/owl";

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

    <t t-name="project.ProjectControlPanelContentBadge">
        <t t-tag="isProjectUser ? 'button' : 'span'" class="badge border d-flex p-2 ms-2 bg-view" data-hotkey="y">
            <span t-attf-class="o_status_bubble o_color_bubble_{{data.color}}"/>
            <span t-att-class="'fw-normal ms-1' + (data.color === 0 ? ' text-muted' : '')" t-esc="data.status"/>
        </t>
    </t>

    <t t-name="project.ProjectControlPanelContent">
        <t t-if="showProjectUpdate">
            <div t-if="isProjectUser" class="o_project_updates_breadcrumb z-index-1" t-on-click="onStatusClick">
                <t t-call="project.ProjectControlPanelContentBadge"></t>
            </div>
            <div t-else="" class="o_project_updates_breadcrumb z-index-1">
                <t t-call="project.ProjectControlPanelContentBadge"></t>
            </div>
        </t>
    </t>

    <t t-name="project.ProjectControlPanel" t-inherit="web.ControlPanel" t-inherit-mode="primary">
        <xpath expr="//t[@t-call='web.Breadcrumbs']" position="after">
            <t t-call="project.ProjectControlPanelContent"/>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\project_many2one_field\project_many2one_field.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from '@web/core/registry';
import { Many2OneField, many2OneField } from '@web/views/fields/many2one/many2one_field';

export class ProjectMany2OneField extends Many2OneField {
    get Many2XAutocompleteProps() {
        const props = super.Many2XAutocompleteProps;
        const { project_id, parent_id } = this.props.record.data;
        if (!project_id && !parent_id) {
            props.placeholder = _t("Private");
        }
        return props;
    }

    get displayName() {
        const { project_id, display_in_project } = this.props.record.data;
        return project_id && !display_in_project ? "" : super.displayName;
    }

    updateRecord(value) {
        const { display_in_project } = this.props.record.data;
        if (!display_in_project && value) {
            this.props.record.update({ "display_in_project": true });
        }
        super.updateRecord(value);
    }
}
ProjectMany2OneField.template = 'project.ProjectMany2OneField';

export const projectMany2OneField = {
    ...many2OneField,
    component: ProjectMany2OneField,
    fieldDependencies: [
        ...(many2OneField.fieldDependencies || []),
        { name: "display_in_project", type: "boolean" },
    ],
};

registry.category("fields").add("project", projectMany2OneField);

```

## File: static\src\components\project_many2one_field\project_many2one_field.xml

```xml
<templates>

    <t t-name="project.ProjectMany2OneField" t-inherit="web.Many2OneField" t-inherit-mode="primary">
        <xpath expr="//t[@t-if='!props.canOpen']/span" position="attributes">
            <attribute name="t-if">props.record.data[props.name]</attribute>
        </xpath>
        <xpath expr="//t[@t-if='!props.canOpen']/span" position="after">
            <span t-elif="!props.record.data.parent_id &amp;&amp; !props.record.data.project_id" class="text-danger fst-italic text-muted"><i class="fa fa-lock"></i> Private</span>
        </xpath>
        <xpath expr="//t[@t-else='']/a" position="attributes">
            <attribute name="t-if">displayName</attribute>
        </xpath>
        <xpath expr="//t[@t-else='']/a" position="after">
            <span t-elif="!props.record.data.parent_id &amp;&amp; !props.record.data.project_id" class="text-danger fst-italic text-muted"><i class="fa fa-lock"></i> Private</span>
        </xpath>
        <xpath expr="//div[hasclass('o_field_many2one_selection')]" position="attributes">
            <attribute name="t-att-class">{
                private_placeholder: !props.record.data.parent_id &amp;&amp; !props.record.data.project_id,
            }</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\project_right_side_panel\project_right_side_panel.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useService } from '@web/core/utils/hooks';
import { formatFloat } from "@web/views/fields/formatters";
import { ViewButton } from '@web/views/view_button/view_button';
import { FormViewDialog } from '@web/views/view_dialogs/form_view_dialog';

import { ProjectRightSidePanelSection } from './components/project_right_side_panel_section';
import { ProjectMilestone } from './components/project_milestone';
import { ProjectProfitability } from './components/project_profitability';
import { getCurrency } from '@web/core/currency';
import { Component, onWillStart, useState } from "@odoo/owl";

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
            'milestones': _t('Milestones'),
            'profitability': _t('Profitability'),
        };
    }

    get showProjectProfitability() {
        const { costs, revenues } = this.state.data.profitability_items;
        return costs.data.length || revenues.data.length;
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
        const currency = getCurrency(this.currencyId);
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
            title: _t('New Milestone'),
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

    <t t-name="project.ProjectRightSidePanel">
        <div t-if="projectId" class="o_rightpanel pt-0 bg-view border-start border-bottom overflow-auto">
            <ProjectRightSidePanelSection
                name="'stat_buttons'"
                header="false"
                show="!!state.data.buttons"
            >
                <div class="o_form_view">
                    <div class="oe_button_box o-form-buttonbox d-flex flex-wrap">
                        <t t-foreach="state.data.buttons" t-as="button" t-key="button.sequence">
                            <ViewButton
                                t-if="button.show"
                                defaultRank="'oe_stat_button'"
                                className="'flex-grow-0 h-auto py-2 border border-start-0 border-top-0 text-start rounded-0'"
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
                show="showProfitability or state.data.show_project_profitability_helper"
                dataClassName="'py-3'"
            >
                <t t-set-slot="title">
                    Profitability
                </t>
                <ProjectProfitability
                    t-if="showProjectProfitability"
                    data="state.data.profitability_items"
                    labels="state.data.profitability_labels"
                    formatMonetary="formatMonetary.bind(this)"
                    onClick="(params) => this.onProjectActionClick(params)"
                />
                <span t-elif="state.data.show_project_profitability_helper" class="text-muted fst-italic">
                    Track project costs, revenues, and margin by setting the analytic account associated with the project on relevant documents.
                </span>
            </ProjectRightSidePanelSection>
            <ProjectRightSidePanelSection
                name="'milestones'"
                show="!!state.data.milestones &amp;&amp; !!state.data.milestones.data"
                dataClassName="'my-3'"
                headerClassName="'border-bottom'"
            >
                <t t-set-slot="header">
                    <span class="btn btn-secondary">
                        <div class="o_add_milestone">
                            <a t-on-click="addMilestone">Add Milestone</a>
                        </div>
                    </span>
                </t>
                <t t-set-slot="title">
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
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";
import { Component, useState, onWillUpdateProps, status } from "@odoo/owl";

const { DateTime } = luxon;

export class ProjectMilestone extends Component {
    setup() {
        this.orm = useService('orm');
        this.dialog = useService("dialog");
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
        this.dialog.add(ConfirmationDialog, {
            body: _t("Are you sure you want to delete this record?"),
            confirm: async () => {
                await this.orm.call('project.milestone', 'unlink', [this.milestone.id]);
                await this.props.load();
            },
            cancel: () => {},
        });
    }

    async onOpenMilestone() {
        if (!this.write_mutex) {
            this.write_mutex = true;
            this.props.open({
                resModel: this.resModel,
                resId: this.milestone.id,
                title: _t("Milestone"),
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

    <t t-name="project.ProjectMilestone">
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

import { Component } from "@odoo/owl";

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
    <t t-name="project.ProjectProfitability">
        <div class="o_rightpanel_subsection pb-3 border-bottom" t-if="revenues.data.length">
            <table class="table table-sm table-striped table-hover mb-0">
                <thead class="bg-100 align-middle">
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
        <div class="o_rightpanel_subsection pb-3 border-bottom" t-if="costs.data.length">
            <table class="table table-sm table-striped table-hover mb-0">
                <thead class="bg-100">
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
        <div class="o_rightpanel_subsection" t-if="revenues.data.length &amp;&amp; costs.data.length">
            <table class="table table-sm table-borderless w-100 mb-0">
                <thead>
                    <tr class="align-top">
                        <th>Margin</th>
                        <th class="text-end" t-att-class="margin.invoiced_billed &lt; 0 ? 'text-danger' : 'text-success'">
                            <t t-esc="props.formatMonetary(margin.invoiced_billed)"/><br/>
                            <t t-if="costs.total.billed != 0">
                                <t t-esc="margin.invoiced_billed > 0 ? '+' : ''"/><t t-esc="(margin.invoiced_billed / (-costs.total.billed) * 100).toFixed(0)"/>%
                            </t>
                        </th>
                        <th class="text-end" t-att-class="margin.to_invoice_to_bill &lt; 0 ? 'text-danger' : 'text-success'">
                            <t t-esc="props.formatMonetary(margin.to_invoice_to_bill)"/><br/>
                            <t t-if="costs.total.to_bill != 0">
                                <t t-esc="margin.to_invoice_to_bill > 0 ? '+' : ''"/><t t-esc="(margin.to_invoice_to_bill / (-costs.total.to_bill) * 100).toFixed(0)"/>%
                            </t>
                        </th>
                        <th class="text-end" t-att-class="margin.total &lt; 0 ? 'text-danger' : 'text-success'">
                            <t t-esc="props.formatMonetary(margin.total)"/><br/>
                            <t t-if="(costs.total.billed + costs.total.to_bill) != 0">
                                <t t-esc="margin.total > 0 ? '+' : ''"/><t t-esc="(margin.total / (-costs.total.billed - costs.total.to_bill) * 100).toFixed(0)"/>%
                            </t>
                        </th>
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

import { Component } from "@odoo/owl";

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
    dataClassName: { type: String, optional: true },
    headerClassName: { type: String, optional: true },
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

    <t t-name="project.ProjectRightSidePanelSection">
        <div class="o_rightpanel_section py-0" t-att-name="props.name" t-if="props.show">
            <div class="o_rightpanel_header d-flex align-items-center justify-content-between py-2 bg-100 border-bottom" t-att-class="props.headerClassName" t-if="props.header">
                <div class="o_rightpanel_title d-flex flex-row-reverse align-items-center" t-if="props.slots.title">
                    <h3 class="m-0 lh-lg"><t t-slot="title"/></h3>
                </div>
                <t t-slot="header"/>
            </div>
            <div class="o_rightpanel_data fs-6" t-if="props.showData" t-att-class="props.dataClassName">
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
import {
    StateSelectionField,
    stateSelectionField,
} from "@web/views/fields/state_selection/state_selection_field";

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
    get options() {
        return super.options.filter(o => o[0] !== 'to_define');
    }
}

export const projectStateSelectionField = {
    ...stateSelectionField,
    component: ProjectStateSelectionField,
};

registry.category("fields").add("project_state_selection", projectStateSelectionField);

```

## File: static\src\components\project_status_with_color_selection\project_status_with_color_selection_field.js

```javascript
/** @odoo-module */

import { SelectionField, selectionField } from '@web/views/fields/selection/selection_field';
import { registry } from '@web/core/registry';

import { STATUS_COLORS, STATUS_COLOR_PREFIX } from '../../utils/project_utils';

export class ProjectStatusWithColorSelectionField extends SelectionField {
    setup() {
        super.setup();
        this.colorPrefix = STATUS_COLOR_PREFIX;
        this.colors = STATUS_COLORS;
    }

    get currentValue() {
        return this.props.record.data[this.props.name] || this.options[0][0];
    }

    statusColor(value) {
        return this.colors[value] ? this.colorPrefix + this.colors[value] : "";
    }
}

ProjectStatusWithColorSelectionField.props = {
    ...SelectionField.props,
    statusLabel: { type: String, optional: true },
};

ProjectStatusWithColorSelectionField.template = 'project.ProjectStatusWithColorSelectionField';

export const projectStatusWithColorSelectionField = {
    ...selectionField,
    component: ProjectStatusWithColorSelectionField,
    extractProps: (fieldInfo, dynamicInfo) => {
        const props = selectionField.extractProps(fieldInfo, dynamicInfo);
        props.statusLabel = fieldInfo.attrs.status_label;
        return props;
    },
};

registry.category("fields").add("status_with_color", projectStatusWithColorSelectionField);

```

## File: static\src\components\project_status_with_color_selection\project_status_with_color_selection_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectStatusWithColorSelectionField" t-inherit="web.SelectionField" t-inherit-mode="primary">
        <xpath expr="//t[@t-if='props.readonly']/span" position="replace">
            <div class="d-flex align-items-center">
                <div>
                    <span t-attf-class="o_status {{ statusColor(currentValue) }} d-inline-block"/>
                </div>
                <div class="ps-2">
                    <div class="o_stat_text" t-if="this.props.statusLabel" t-out="this.props.statusLabel"/>
                    <div class="o_stat_value" t-out="string" t-att-raw-value="value"/>
                </div>
            </div>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\project_task_name_with_subtask_count_char_field\project_task_name_with_subtask_count_char_field.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';
import { CharField, charField } from '@web/views/fields/char/char_field';

export class ProjectTaskNameWithSubtaskCountCharField extends CharField {
    static template = "project.ProjectTaskNameWithSubtaskCountCharField";
}

export const projectTaskNameWithSubtaskCountCharField = {
    ...charField,
    component: ProjectTaskNameWithSubtaskCountCharField,
    fieldsDependencies: [
        { name: "subtask_count", type: "integer" },
        { name: "closed_subtask_count", type: "integer" },
    ],
}
registry.category("fields").add("name_with_subtask_count", projectTaskNameWithSubtaskCountCharField);

```

## File: static\src\components\project_task_name_with_subtask_count_char_field\project_task_name_with_subtask_count_char_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectTaskNameWithSubtaskCountCharField" t-inherit="web.CharField" t-inherit-mode="primary">
        <xpath expr="//span[@t-esc='formattedValue']" position="after">
            <span class="text-muted ms-2 fw-normal">
                <t t-if="props.record.data.subtask_count">
                    (<t t-out="props.record.data.closed_subtask_count"/>/<t t-out="props.record.data.subtask_count"/> sub-tasks)
                </t>
            </span>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\project_task_priority_switch_field\project_task_priority_switch_field.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { PriorityField, priorityField } from "@web/views/fields/priority/priority_field";

export class PrioritySwitchField extends PriorityField {
    get commands() {
        return this.options.map(([id, name]) => [
            _t("Set priority as %s", name),
            () => this.updateRecord(id),
            {
                category: "smart_action",
                hotkey: "alt+r",
                isAvailable: () => this.props.record.data[this.props.name] !== id,
            },
        ]);
    }
}

export const prioritySwitchField = {
    ...priorityField,
    component: PrioritySwitchField,
    extractProps({ viewType }) {
        const props = priorityField.extractProps(...arguments);
        props.withCommand = viewType === "form";
        return props;
    },
};

registry.category("fields").add("priority_switch", prioritySwitchField);

```

## File: static\src\components\project_task_state_selection\project_task_state_selection.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import {
    StateSelectionField,
    stateSelectionField,
} from "@web/views/fields/state_selection/state_selection_field";
import { useCommand } from "@web/core/commands/command_hook";
import { formatSelection } from "@web/views/fields/formatters";

import { registry } from "@web/core/registry";
import { useState } from "@odoo/owl";

export class ProjectTaskStateSelection extends StateSelectionField {
    setup() {
        this.state = useState({
            isStateButtonHighlighted: false,
        });
        this.icons = {
            "01_in_progress": "o_status",
            "03_approved": "o_status o_status_green",
            "02_changes_requested": "fa fa-lg fa-exclamation-circle",
            "1_done": "fa fa-lg fa-check-circle",
            "1_canceled": "fa fa-lg fa-times-circle",
            "04_waiting_normal": "fa fa-lg fa-hourglass-o",
        };
        this.colorIcons = {
            "01_in_progress": "",
            "03_approved": "text-success",
            "02_changes_requested": "o_status_changes_requested",
            "1_done": "text-success",
            "1_canceled": "text-danger",
            "04_waiting_normal": "",
        };
        this.colorButton = {
            "01_in_progress": "btn-outline-secondary",
            "03_approved": "btn-outline-success",
            "02_changes_requested": "btn-outline-warning",
            "1_done": "btn-outline-success",
            "1_canceled": "btn-outline-danger",
            "04_waiting_normal": "btn-outline-secondary",
        };
        if (this.props.viewType != 'form') {
            super.setup();
        } else {
            const commandName = _t("Set state as...");
            useCommand(
                commandName,
                () => {
                    return {
                        placeholder: commandName,
                        providers: [
                            {
                                provide: () =>
                                    this.options.map(subarr => ({
                                        name: subarr[1],
                                        action: () => {
                                            this.updateRecord(subarr[0]);
                                        },
                                    })),
                            },
                        ],
                    };
                },
                {
                    category: "smart_action",
                    hotkey: "alt+f",
                    isAvailable: () => !this.props.readonly && !this.props.isDisabled,
                }
            );
        }
    }

    get options() {
        const labels = new Map(super.options);
        const states = ["1_canceled", "1_done"];
        const currentState = this.props.record.data[this.props.name];
        if (currentState != "04_waiting_normal") {
            states.unshift("01_in_progress", "02_changes_requested", "03_approved");
        }
        return states.map((state) => [state, labels.get(state)]);
    }

    get availableOptions() {
        // overrided because we need the currentOption in the dropdown as well
        return this.options;
    }

    get label() {
        const waitOption = super.options.findLast(([state, _]) => state === "04_waiting_normal");
        const fullSelection = [...this.options, waitOption];
        return formatSelection(this.currentValue, {
            selection: fullSelection,
        });
    }

    stateIcon(value) {
        return this.icons[value] || "";
    }

    /**
     * @override
     */
    statusColor(value) {
        return this.colorIcons[value] || "";
    }

    /**
     * determine if a single click will trigger the toggleState() method
     * which will switch the state from in progress to done.
     * Either the isToggleMode is active on the record OR the task is_private
     */
    get isToggleMode() {
        return this.props.isToggleMode || !this.props.record.data.project_id;
    }

    isView(viewNames) {
        return viewNames.includes(this.props.viewType);
    }

    async toggleState() {
        const toggleVal = this.currentValue == "1_done" ? "01_in_progress" : "1_done";
        await this.updateRecord(toggleVal);
    }

    getDropdownPosition() {
        if (this.isView(['activity', 'kanban', 'list', 'calendar']) || this.env.isSmall) {
            return '';
        }
        return 'bottom-end';
    }

    getTogglerClass(currentValue) {
        if (this.isView(['activity', 'kanban', 'list', 'calendar']) || this.env.isSmall) {
            return 'btn btn-link d-flex p-0';
        }
        return 'o_state_button btn rounded-pill ' + this.colorButton[currentValue];
    }

    async updateRecord(value) {
        const result = await super.updateRecord(value);
        this.state.isStateButtonHighlighted = false;
        if (result) {
            return result;
        }
    }

    /**
     * @param {MouseEvent} ev
     */
    onMouseEnterStateButton(ev) {
        if (!this.env.isSmall) {
            this.state.isStateButtonHighlighted = true;
        }
    }

    /**
     * @param {MouseEvent} ev
     */
    onMouseLeaveStateButton(ev) {
        this.state.isStateButtonHighlighted = false;
    }
}

ProjectTaskStateSelection.template = "project.ProjectTaskStateSelection";

ProjectTaskStateSelection.props = {
    ...stateSelectionField.component.props,
    isToggleMode: { type: Boolean, optional: true },
    viewType: { type: String },
}

export const projectTaskStateSelection = {
    ...stateSelectionField,
    component: ProjectTaskStateSelection,
    fieldDependencies: [{ name: "project_id", type: "many2one" }],
    supportedOptions: [
        ...stateSelectionField.supportedOptions, {
            label: _t("Is toggle mode"),
            name: "is_toggle_mode",
            type: "boolean"
        }
    ],
    extractProps({ options, viewType }) {
        const props = stateSelectionField.extractProps(...arguments);
        props.isToggleMode = Boolean(options.is_toggle_mode);
        props.viewType = viewType;
        return props;
    },
}

registry.category("fields").add("project_task_state_selection", projectTaskStateSelection);

```

## File: static\src\components\project_task_state_selection\project_task_state_selection.xml

```xml
<templates>
    <t t-name="project.ProjectTaskStateSelection" t-inherit="web.StateSelectionField" t-inherit-mode="primary">
        <!-- Readonly button -->
        <xpath expr="//t[@t-if='props.readonly']/button" position="attributes">
            <attribute name="tabindex">-1</attribute>
        </xpath>
        <xpath expr="//t[@t-if='props.readonly']/button/span[1]" position="attributes">
            <attribute name="t-attf-class">{{ stateIcon(currentValue) }} {{ statusColor(currentValue) }}</attribute>
        </xpath>
        <xpath expr="//t[@t-if='props.readonly']/button" position="attributes">
            <attribute name="style">cursor: default;</attribute>
            <attribute name="t-att-title">label</attribute>
        </xpath>
        <!-- Waiting state button -->
        <xpath expr="//t[@t-if='props.readonly']" position="after">
            <t t-elif="currentValue == '04_waiting_normal' and isView(['activity', 'kanban', 'list', 'calendar'])">
                <button class="d-flex align-items-center btn fw-normal p-0 justify-content-center " title="This task is blocked by another unfinished task" t-att-class="{'o_task_state_list_view': isView(['list'])}">
                    <i class="fa fa-lg fa-hourglass-o"></i>
                </button>
            </t>
        </xpath>
        <!-- The toggle mark as done button -->
        <xpath expr="//t[@t-if='props.readonly']" position="after">
            <t t-elif="isToggleMode and currentValue == '01_in_progress'">
                <button t-if="isView(['activity', 'kanban', 'list', 'calendar']) or this.env.isSmall"
                        class="d-flex align-items-center btn fw-normal p-0"
                        tabindex="-1"
                        t-att-class="{'o_task_state_list_view': isView(['list'])}"
                        t-on-click.stop="toggleState"
                >
                    <i t-attf-class="{{ stateIcon(currentValue) }} {{ statusColor(currentValue) }} {{ ['1_done', '1_canceled'].includes(currentValue) and isView(['activity', 'kanban']) ? 'opacity-50' : '' }}"></i>
                </button>
                <button t-else="" class="o_state_button btn oe_highlight rounded-pill" style="white-space: nowrap;" tabindex="-1"
                    t-attf-class="#{currentValue == '1_done' ? 'btn-success' : 'btn-outline-secondary'}"
                    t-att-class="{'bg-view border' : state.isStateButtonHighlighted}"
                    t-on-click="toggleState" t-on-mouseenter="onMouseEnterStateButton"
                    t-on-mouseleave="onMouseLeaveStateButton">
                    <div class="d-flex align-items-center">
                        <span class="o_status_label">
                            <t t-if="state.isStateButtonHighlighted">
                                <span class="text-success oe_highlight">
                                    <i class="fa fa-fw fa-check"/>
                                    Mark as done
                                </span>
                            </t>
                            <t t-else="currentValue == '01_in_progress'">
                                In Progress
                            </t>
                        </span>
                    </div>
                </button>
            </t>
        </xpath>

        <!-- Normal dropdown mode toggle button (displayed on the card/record by default) -->
        <xpath expr="//t[@t-set-slot='toggler']/div" position="replace">
            <div t-if="isView(['activity', 'kanban', 'list', 'calendar']) or this.env.isSmall" class="d-flex align-items-center" t-att-class="{'o_task_state_list_view': isView(['list'])}" t-att-title="label">
                <i t-if="currentValue == '04_waiting_normal'" t-attf-class="{{ stateIcon(currentValue) }} {{ statusColor(currentValue) }}" style="color: #4A4F59;"/>
                <i t-else="" t-attf-class="{{ stateIcon(currentValue) }} {{ statusColor(currentValue) }} {{ ['1_done', '1_canceled'].includes(currentValue) and isView(['activity', 'kanban']) ? 'opacity-50' : '' }}"/>
            </div>
            <div t-else="" class="d-flex align-items-center">
                <t t-if="currentValue == '04_waiting_normal'">
                    <i class="fa fa-fw fa-hourglass-o"/>
                    <span class="o_status_label" title="This task is blocked by another unfinished task">
                        Waiting
                    </span>
                </t>
                <t t-elif="currentValue != '1_done' and currentValue != '1_canceled'">
                    <span t-attf-class="o_status_label" t-out="label"/>
                </t>
                <t t-else="">
                    <span class="o_status_label" t-out="label"/>
                </t>
            </div>
        </xpath>
        <!-- Tooltip for the dropdown toggler -->
        <xpath expr="//Dropdown" position="attributes">
            <attribute name="tooltip">''</attribute>
            <attribute name="class">toggle_dropdown</attribute>
            <attribute name="position">`${ getDropdownPosition() }`</attribute>
            <attribute name="togglerClass">getTogglerClass(currentValue)</attribute>
        </xpath>
        <!-- Dropdown divider -->
        <xpath expr="//DropdownItem" position="before">
            <div t-if="option[0] == '1_canceled' and (currentValue != '04_waiting_normal' or this.env.isSmall)" role="separator" class="dropdown-divider"/>
        </xpath>
        <!-- Approval mode dropdown button (class)-->
        <xpath expr="//DropdownItem/span[1]" position="attributes">
            <attribute name="t-attf-class" separator=" " add="{{ stateIcon(option[0]) }}" remove="o_status"></attribute>
        </xpath>
        <xpath expr="//DropdownItem/span[2]" position="attributes">
            <attribute name="t-attf-class">{{ statusColor(option[0]) }}</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\subtask_kanban_list\subtask_kanban_list.js

```javascript
/** @odoo-module */

import { Component, onWillStart } from "@odoo/owl";

import { useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";

import { Record } from "@web/model/record";
import { KanbanMany2ManyTagsAvatarUserField } from "@mail/views/web/fields/many2many_avatar_user_field/many2many_avatar_user_field";
import { Field, getFieldFromRegistry } from "@web/views/fields/field";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";

export class SubtaskKanbanList extends Component {
    setup() {
        this.actionService = useService("action");
        this.orm = useService("orm");
        this.subTasksRead = [];
        this.subTaskClosed = new Set();

        onWillStart(this.onWillStart);
    }

    async onWillStart() {
        this.subTasksRead = await this.orm.searchRead(
            this.props.record.resModel, [
                ["parent_id", "=", this.props.record.resId],
                ["state", "not in", ["1_done", "1_canceled"]],
            ],
            this.fieldNames
        );
    }

    async goToSubtask(subtask_id) {
        return this.actionService.doAction({
            type: "ir.actions.act_window",
            res_model: this.props.record.resModel,
            res_id: subtask_id,
            views: [[false, "form"]],
            target: "current",
            context: {
                active_id: subtask_id,
            },
        });
    }

    get fieldNames() {
        return Object.keys(this.fields);
    }

    get fields() {
        const { display_name, state, user_ids, project_id } = this.props.record.fields;
        return {
            display_name,
            state,
            user_ids,
            project_id,
        };
    }

    get activeFields() {
        return {
            display_name: {},
            state: {
                name: "state",
                viewType: "kanban",
                field: getFieldFromRegistry(this.fields.state.type, "project_task_state_selection", "kanban"),
            },
            user_ids: {
                name: "user_ids",
                field: getFieldFromRegistry(this.fields.user_ids.type, "many2many_avatar_user", "kanban"),
            },
            project_id: {
                field: getFieldFromRegistry(this.fields.project_id.type, "project", "kanban")
            },
        };
    }

    async onSubTaskSaved(subTask) {
        const ids = this.subTasksRead.map((t) => t.id);
        this.subTasksRead = await this.orm.searchRead(
            this.props.record.resModel,
            [["id", "in", ids]],
            this.fieldNames
        );
        const isKnownAsClosed = this.subTaskClosed.has(subTask.resId);
        const isClosed = subTask.data.state.startsWith("1_");
        if (isKnownAsClosed && !isClosed) {
            this.subTaskClosed.delete(subTask.resId);
        } else if (!isKnownAsClosed && isClosed) {
            this.subTaskClosed.add(subTask.resId);
        } else { // nothing to do
            return;
        }
        await this.props.record.load();
    }
}

SubtaskKanbanList.components = {
    Record,
    Field,
    KanbanMany2ManyTagsAvatarUserField,
};
SubtaskKanbanList.props = {
    ...standardWidgetProps,
};
SubtaskKanbanList.template = "project.SubtaskKanbanList";
const subtaskKanbanList = {
    component: SubtaskKanbanList,
    fieldDependencies: [{ name: "child_ids", type: "one2many" }],
};

registry.category("view_widgets").add("subtask_kanban_list", subtaskKanbanList);

```

## File: static\src\components\subtask_kanban_list\subtask_kanban_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<templates>

    <t t-name="project.SubtaskKanbanList">
        <div class="subtask_list">
            <t t-foreach="subTasksRead" t-as="subTask" t-key="subTask.id">
                <Record resModel="'project.task'"
                        resId="subTask.id"
                        fields="fields"
                        activeFields="activeFields"
                        values="subTask"
                        t-slot-scope="data"
                        onRecordSaved.bind="onSubTaskSaved"
                >
                    <div class="subtask_list_row">
                        <a t-attf-class="subtask_name_col {{['1_done', '1_canceled'].includes(data.record.data.state) ? 'text-muted opacity-50' : ''}}"
                           t-att-title="data.record.data.display_name"
                           style="color: inherit;"
                           t-esc="data.record.data.display_name"
                           t-on-click.prevent="() => this.goToSubtask(data.record.resId)"/>
                        <Field
                            class="`subtask_user_widget_col d-inline-flex justify-content-end align-items-center me-1 ${['1_done', '1_canceled'].includes(data.record.data.state) ? 'opacity-50' : ''}`"
                            name="'user_ids'"
                            record="data.record"
                            fieldInfo="data.record.activeFields.user_ids"
                            readonly="false"
                            type="'many2many_avatar_user'"/>
                        <Field name="'state'"
                            class="`subtask_state_widget_col d-flex justify-content-center align-items-center`"
                            record="data.record"
                            fieldInfo="data.record.activeFields.state"
                            type="'project_task_state_selection'"/>
                    </div>
                </Record>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\components\subtask_one2many_field\subtask_list_renderer.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { ListRenderer } from '@web/views/list/list_renderer';

import { useEffect } from "@odoo/owl";

export class SubtaskListRenderer extends ListRenderer {
    setup() {
        super.setup();
        this.dialog = useService("dialog");
        useEffect(
            (editedRecord) => this.focusName(editedRecord),
            () => [this.editedRecord]
        );
    }

    focusName(editedRecord) {
        if (editedRecord?.isNew && !editedRecord.dirty) {
            const col = this.state.columns.find((c) => c.name === "name");
            this.focusCell(col);
        }
    }

    async onDeleteRecord(record) {
        this.dialog.add(ConfirmationDialog, {
            body: _t("Are you sure you want to delete this record?"),
            confirm: () => super.onDeleteRecord(record),
            cancel: () => {},
        });
    }
}

```

## File: static\src\components\subtask_one2many_field\subtask_one2many_field.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { X2ManyField, x2ManyField } from '@web/views/fields/x2many/x2many_field';

import { SubtaskListRenderer } from './subtask_list_renderer';

export class SubtaskOne2ManyField extends X2ManyField {}

SubtaskOne2ManyField.components = {
    ...X2ManyField.components,
    ListRenderer: SubtaskListRenderer,
}

export const subtaskOne2ManyField = {
    ...x2ManyField,
    component: SubtaskOne2ManyField,
    additionalClasses: ["o_field_one2many"],
}

registry.category("fields").add("subtasks_one2many", subtaskOne2ManyField);

```

## File: static\src\img\folder.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M32.5125 11.4113L27.7457 8.71763C27.1893 8.28739 26.4241 8.24848 25.827 8.62007L5.13932 20.5438C4.48071 20.9576 4.19072 21.7682 4.43744 22.5058L18.4411 60.5643C18.5986 61.0023 19.1694 61.9233 19.6074 61.7658C19.6564 61.7482 19.7037 61.7261 19.7486 61.6997L59.5116 38.6814C60.1816 38.2923 60.4578 37.4658 60.1566 36.752L47.0757 5.88032C46.7147 5.02549 45.7291 4.62519 44.8742 4.9862C44.835 5.00275 44.7965 5.02078 44.7587 5.04025L34.3227 11.5712C33.7399 11.8763 33.0328 11.8138 32.5125 11.4113Z" fill="#FBDBD0"/>
<path d="M3.40499 22.1807L18.0382 60.8153L17.959 59.2562L4.43747 22.514C4.34574 22.2361 4.32707 21.9394 4.38327 21.6522L3.35349 21.2999C3.29203 21.5929 3.30981 21.8969 3.40499 22.1807Z" fill="#C1DBF6"/>
<path d="M26.9895 8.37342C26.4303 7.9467 25.6642 7.91425 25.0709 8.29212L4.10676 20.2159C3.71826 20.4593 3.44612 20.8509 3.35339 21.2998L4.38318 21.644C4.47273 21.1887 4.74631 20.7906 5.13924 20.5438L25.8269 8.62003C26.1857 8.3971 26.6154 8.31774 27.0302 8.39781L26.9895 8.37342Z" fill="white"/>
<path d="M43.7642 4.99963L33.5667 11.2325C33.2289 11.4099 32.8403 11.4654 32.4664 11.3897L32.5125 11.4168C33.0328 11.8193 33.7399 11.8818 34.3227 11.5767L44.7587 5.04029C45.1904 4.81487 45.6987 4.78818 46.1516 4.96713C46.1516 4.96713 44.7885 4.44409 43.7642 4.99963Z" fill="white"/>
<path d="M46.4362 10.7474L45.3902 16.7391C45.3128 17.0273 45.1287 17.2753 44.8753 17.4328L18.8599 32.5109C18.5202 32.7272 18.3155 33.1028 18.3179 33.5055L18.8599 61.3989C18.8647 61.6608 19.081 61.8691 19.3428 61.8643C19.4237 61.8628 19.5027 61.8406 19.5726 61.8L60.1404 38.3129C60.4772 38.1185 60.6839 37.7587 60.6824 37.3699V3.82893C60.6826 3.18089 60.1574 2.65536 59.5094 2.65515C59.2888 2.65508 59.0726 2.71721 58.8857 2.83437L46.962 10.0537C46.7037 10.2083 46.5153 10.4569 46.4362 10.7474Z" fill="white"/>
<path d="M43.9565 16.9044L17.9085 31.931C17.7906 32.0026 17.686 32.0944 17.5996 32.202L18.5725 32.7657L18.6375 32.6871C18.7061 32.6153 18.7835 32.5525 18.8679 32.5001L44.8833 17.422C45.0062 17.3459 45.1137 17.2476 45.2004 17.132L44.2736 16.6117C44.1867 16.7279 44.0793 16.8271 43.9565 16.9044Z" fill="white"/>
<path d="M45.5148 10.0672L44.4688 16.2106C44.4283 16.3597 44.3592 16.4996 44.2655 16.6225L45.1923 17.1428C45.2798 17.0236 45.345 16.8896 45.3847 16.7471L46.4307 10.7555C46.4505 10.6836 46.4768 10.6138 46.5093 10.5468C46.5643 10.435 46.6365 10.3326 46.7234 10.2433L45.7207 9.64709C45.6253 9.77233 45.5554 9.91505 45.5148 10.0672Z" fill="#C1DBF6"/>
<path d="M57.0781 2.38721C57.0613 2.39425 57.045 2.40239 57.0293 2.4116L46.0296 9.37615C45.9114 9.44737 45.8067 9.53914 45.7207 9.64714L46.7315 10.2244C46.7987 10.1558 46.8732 10.0949 46.9537 10.0428L54.9074 5.22993L58.8829 2.82351C59.4283 2.48309 60.1461 2.64504 60.4926 3.18664C59.975 2.40076 58.4222 1.76392 57.0781 2.38721Z" fill="white"/>
<path d="M17.3559 32.9283L18.0095 60.8153C18.0095 61.0132 19.1119 62.0576 19.5265 61.8163C19.5265 61.8163 19.5237 61.7906 19.4181 61.8054C19.0089 61.838 18.8409 61.3854 18.8328 60.9925L18.3152 33.4947C18.3161 33.2954 18.3674 33.0997 18.4642 32.9256C18.4953 32.869 18.5316 32.8155 18.5726 32.7657L17.5998 32.202C17.4401 32.4103 17.3542 32.6658 17.3559 32.9283Z" fill="#C1DBF6"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M51.8461 19.7264C52.0683 19.8463 52.1512 20.1236 52.0312 20.3458L39.1085 44.282C39.0509 44.3887 38.9533 44.4682 38.8371 44.5029C38.7209 44.5376 38.5957 44.5247 38.489 44.4671L33.1549 41.5857C32.9327 41.4657 32.8499 41.1883 32.9699 40.9662C33.0899 40.7441 33.3673 40.6613 33.5894 40.7813L38.5212 43.4453L51.2267 19.9114C51.3466 19.6893 51.624 19.6064 51.8461 19.7264Z" fill="#374874"/>
<path d="M58.2392 2.13562C59.2166 2.13567 60.1207 2.6225 60.4923 3.18609C60.4896 3.18209 60.4862 3.17901 60.4834 3.17501C60.609 3.36189 60.6824 3.5868 60.6823 3.82888V37.3699C60.6839 37.7586 60.4771 38.1185 60.1404 38.3129C60.1404 38.3129 19.4236 61.8628 19.3428 61.8643H19.3423C19.2174 61.8643 18.0094 60.9114 18.0094 60.8153L3.40501 22.1806C3.30981 21.8969 3.29205 21.5929 3.35352 21.2999C3.44625 20.851 3.71839 20.4593 4.10689 20.2159L25.071 8.29219C25.3467 8.11661 25.6597 8.02961 25.972 8.02963C26.3316 8.02963 26.6903 8.14509 26.9896 8.3735L27.0285 8.39686C27.2842 8.44511 27.5307 8.55135 27.7457 8.71768L32.4761 11.3908C32.5824 11.4116 32.6899 11.422 32.797 11.422C33.0635 11.422 33.3278 11.3579 33.5667 11.2325L43.7642 4.99965C44.0909 4.82248 44.5835 4.68003 45.1017 4.68006C45.8459 4.68008 46.6428 4.97389 47.0757 5.88035L48.0224 8.11448C48.0224 8.11448 56.3587 2.84095 57.0781 2.38721C57.4579 2.21112 57.8542 2.13562 58.2392 2.13562ZM58.2392 1.22134C57.695 1.22131 57.175 1.3345 56.6935 1.55777L56.6401 1.5825L56.5904 1.61388C56.0181 1.97482 50.6262 5.38545 48.4439 6.76593L47.9176 5.52367L47.9096 5.50481L47.9007 5.48634C47.3785 4.39299 46.3584 3.76586 45.1018 3.76577C44.4788 3.76572 43.8324 3.92252 43.3283 4.19594L43.3075 4.20721L43.2873 4.21955L33.1204 10.4337C33.0203 10.4822 32.9089 10.5077 32.797 10.5077C32.7883 10.5077 32.7796 10.5076 32.7709 10.5073L28.2464 7.95046C27.9935 7.76636 27.7056 7.62854 27.4016 7.5454C26.9798 7.26705 26.4785 7.11535 25.972 7.11532C25.4865 7.11532 25.0131 7.25085 24.6013 7.50751L3.65487 19.4212L3.63793 19.4308L3.62143 19.4411C3.0246 19.8151 2.60058 20.4252 2.45813 21.115C2.36396 21.5639 2.39144 22.0339 2.5382 22.4714L2.54369 22.4877L2.54979 22.5039L17.1263 61.0648C17.2203 61.4152 17.528 61.6723 18.0632 62.094C18.8065 62.6796 19.0345 62.7785 19.3423 62.7785H19.3508L19.3598 62.7784C19.6004 62.7739 19.6254 62.7596 20.2671 62.3938C20.5704 62.2209 21.0128 61.9673 21.5744 61.6446C22.6958 61.0004 24.2934 60.0801 26.2082 58.9757C30.037 56.7676 35.1344 53.8236 40.2293 50.8797C50.4188 44.9919 60.5981 39.1044 60.5981 39.1044C61.2166 38.7473 61.5995 38.0811 61.5966 37.3662V3.82888C61.5967 3.42636 61.4821 3.03656 61.2647 2.69907L61.2657 2.69841L61.2567 2.68422L61.251 2.6759C60.6729 1.80561 59.4638 1.22143 58.2392 1.22134Z" fill="#374874"/>
</svg>

```

## File: static\src\img\tasks.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M19.4073 61.7256L17.3805 60.5477C17.1171 60.3946 16.9537 60.0693 16.9524 59.6071L18.9792 60.7849C18.9805 61.2472 19.1439 61.5725 19.4073 61.7256Z" fill="#FBDBD0"/>
<path d="M46.3291 1.72739L48.3559 2.90524C48.0893 2.75029 47.7202 2.77217 47.3134 3.00702L45.2866 1.82917C45.6934 1.59432 46.0625 1.57246 46.3291 1.72739Z" fill="#FBDBD0"/>
<path d="M18.9792 60.7849L16.9524 59.6071L16.8399 19.9392L18.8666 21.1171L18.9792 60.7849Z" fill="#FBDBD0"/>
<path d="M20.3289 18.5865L18.3021 17.4087L45.2866 1.82917L47.3134 3.00702L20.3289 18.5865Z" fill="#FBDBD0"/>
<path d="M47.3134 3.00702C48.1213 2.5406 48.7807 2.91429 48.7833 3.84469L48.8958 43.5126C48.8984 44.443 48.2433 45.5777 47.4354 46.0442L20.4509 61.6237C19.6412 62.0912 18.9818 61.7153 18.9792 60.7849L18.8666 21.1171C18.864 20.1867 19.5192 19.054 20.3289 18.5865L47.3134 3.00702Z" fill="white"/>
<path d="M18.8666 21.1171L16.8399 19.9392C16.8372 19.0088 17.4924 17.8762 18.3021 17.4087L20.3289 18.5865C19.5192 19.054 18.864 20.1867 18.8666 21.1171Z" fill="#FBDBD0"/>
<path d="M36.3632 5.83742L34.3364 4.65957C34.7715 4.40832 35.1659 4.38505 35.4507 4.55056L37.4775 5.72841C37.1927 5.5629 36.7983 5.58617 36.3632 5.83742Z" fill="#C1DBF6"/>
<path d="M29.687 13.4999L27.6602 12.3221L27.6545 10.3199L29.6814 11.4977L29.687 13.4999Z" fill="#C1DBF6"/>
<path d="M31.2424 8.79392L29.2156 7.61607L34.3364 4.65957L36.3632 5.83742L31.2424 8.79392Z" fill="#C1DBF6"/>
<path d="M29.6814 11.4977L27.6545 10.3199C27.6517 9.32618 28.3508 8.11533 29.2156 7.61607L31.2424 8.79392C30.3776 9.29321 29.6785 10.5041 29.6814 11.4977Z" fill="#C1DBF6"/>
<path d="M36.3631 5.83743C37.2279 5.33814 37.9315 5.73914 37.9343 6.73284L37.94 8.73501L38.9774 8.13608C39.8385 7.63892 40.5403 8.03885 40.5431 9.02838L40.5474 10.5528C40.5479 10.7191 40.4594 10.8729 40.3154 10.9561L27.7929 18.1859C27.484 18.3642 27.0978 18.142 27.0968 17.7853L27.094 16.7932C27.0912 15.8037 27.7884 14.596 28.6496 14.0988L29.6869 13.4999L29.6813 11.4977C29.6785 10.504 30.3775 9.29318 31.2423 8.79391L36.3631 5.83743Z" fill="#C1DBF6"/>
<path d="M45.951 1.62559C46.0851 1.62559 46.2039 1.65594 46.3011 1.71332C46.3011 1.71332 48.3529 2.90383 48.353 2.90383C48.6175 3.05594 48.782 3.38106 48.7833 3.84456L48.8433 25.0004L48.8958 24.6125L52.895 18.7457C52.895 18.7457 52.9862 18.7477 53.1284 18.7611L53.1295 18.7594C53.1295 18.7594 53.2675 18.3456 54.0125 18.2489C54.0682 18.2418 54.1199 18.2384 54.1679 18.2384C54.3157 18.2384 54.4288 18.2699 54.5156 18.317L55.0473 17.4625C55.0473 17.4625 55.2359 17.4086 55.483 17.4086C55.7037 17.4086 55.9711 17.4516 56.1924 17.6143C56.7571 18.0295 56.6615 18.4974 56.6615 18.4974L55.4805 20.1077C55.6295 20.3841 55.6267 20.5668 55.6267 20.5668L48.8575 30.0141L48.8958 43.5125C48.8985 44.4429 48.2433 45.5777 47.4354 46.0441L20.4509 61.6236C20.215 61.7599 19.9921 61.8241 19.7943 61.8241C19.6486 61.8241 19.5166 61.7892 19.4033 61.7222C19.403 61.7222 17.3804 60.5477 17.3804 60.5477C17.117 60.3946 16.9536 60.0692 16.9523 59.6071L16.8399 19.9391C16.8372 19.0087 17.4924 17.8761 18.3021 17.4086L27.6593 12.0063L27.6545 10.3199C27.6517 9.32614 28.3508 8.11533 29.2155 7.6161L34.3363 4.65962C34.5887 4.51386 34.8273 4.44479 35.0388 4.44479C35.192 4.44479 35.331 4.48106 35.4506 4.5506L37.4774 5.72838C37.4767 5.72792 37.4758 5.72772 37.475 5.72726C37.6248 5.81363 37.7434 5.95338 37.823 6.13821L45.2866 1.82916C45.5278 1.68998 45.7556 1.62548 45.951 1.62559ZM45.9511 0.711304C45.5869 0.71119 45.1991 0.824035 44.8295 1.03732L38.0028 4.97862C37.9904 4.97069 37.978 4.96278 37.9653 4.95519C37.9559 4.94927 37.9465 4.94346 37.9368 4.93789L35.91 3.7601C35.6517 3.61 35.3504 3.53064 35.0389 3.53053C34.6569 3.53053 34.2666 3.64404 33.879 3.86792L28.7584 6.82429C27.6041 7.49069 26.7364 8.9946 26.7402 10.3225L26.7435 11.4793L17.8449 16.6168C16.7471 17.2507 15.922 18.6801 15.9255 19.9417L16.038 59.6097C16.0402 60.3832 16.362 61.0132 16.9209 61.3381L17.9326 61.9256C18.4528 62.2278 18.752 62.4015 18.943 62.5014L18.9382 62.5094C19.1918 62.6593 19.4879 62.7385 19.7943 62.7385C20.1621 62.7385 20.5368 62.6298 20.9081 62.4154L47.8925 46.836C48.9893 46.2028 49.8136 44.7728 49.8101 43.51L49.7726 30.3067L56.3699 21.0993C56.4784 20.9479 56.538 20.7668 56.5408 20.5806C56.5418 20.519 56.5373 20.4065 56.5051 20.2567L57.3987 19.0381C57.4767 18.9317 57.5308 18.8097 57.5572 18.6804C57.6553 18.2002 57.4986 17.4399 56.7339 16.8778C56.3928 16.627 55.9602 16.4944 55.483 16.4944C55.1259 16.4944 54.8482 16.5686 54.7961 16.5835C54.578 16.6457 54.3908 16.7869 54.2709 16.9796L54.0545 17.3274C54.0021 17.3303 53.9488 17.3354 53.8952 17.3423C53.3139 17.4176 52.9317 17.6321 52.6837 17.8561C52.4646 17.9082 52.2694 18.0401 52.1394 18.2308L49.7483 21.7386L49.6975 3.84205C49.6954 3.0666 49.372 2.43624 48.8105 2.11236C48.7637 2.08522 46.7598 0.922504 46.7598 0.922504C46.5281 0.785567 46.2463 0.711304 45.9511 0.711304Z" fill="#374874"/>
<path d="M55.0474 17.4626C55.0474 17.4626 55.7234 17.2694 56.1925 17.6144C56.7572 18.0296 56.6616 18.4974 56.6616 18.4974L55.2957 20.3599L53.8884 19.3252L55.0474 17.4626Z" fill="white"/>
<path d="M37.8288 40.8481L52.8949 18.7457C52.8949 18.7457 53.3762 18.7524 53.9082 18.9064C54.1838 18.9861 54.4731 19.1054 54.7161 19.2838C55.6391 19.9614 55.6267 20.5669 55.6267 20.5669L40.4778 42.5038L37.8288 43.9938L37.3735 43.6489L37.8288 40.8481Z" fill="#C1DBF6"/>
<path d="M50.2599 24.9606L54.799 18.7457C54.799 18.7457 54.7576 18.1525 54.0126 18.249C53.2676 18.3456 53.1296 18.7595 53.1296 18.7595L49.0733 24.4851C49.0733 24.4851 48.5353 25.2567 50.2599 24.9606Z" fill="white"/>
<path d="M37.4151 43.4211L37.2359 44.2192C37.2184 44.2974 37.3051 44.3571 37.3719 44.3127L38.029 43.8764C38.029 43.8764 37.8873 43.636 37.8152 43.5798C37.7303 43.5137 37.4151 43.4211 37.4151 43.4211Z" fill="#ECECEC"/>
<path d="M33.5678 20.2363C33.7408 20.1365 33.9139 20.1156 34.042 20.1983C34.2881 20.3569 34.2763 20.8357 34.0159 21.2682L26.7734 33.302C26.6488 33.5097 26.4894 33.669 26.3298 33.7611C26.1635 33.8571 25.9971 33.88 25.87 33.8084L23.4514 32.4301C23.198 32.284 23.1967 31.8129 23.4484 31.3754C23.5749 31.157 23.7407 30.9886 23.9067 30.8928C24.0727 30.7969 24.2389 30.7736 24.3663 30.8454L24.5389 30.9439L26.3385 31.9709L31.7319 23.0094L33.1255 20.6938C33.2496 20.4871 33.4087 20.3282 33.5678 20.2363Z" fill="#374874"/>
<path d="M34.9714 16.5324C35.1509 16.4288 35.2973 16.5106 35.2979 16.7168L35.3387 31.0812C35.3392 31.2857 35.1938 31.5377 35.0143 31.6413L22.5149 38.8578C22.3354 38.9614 22.1905 38.8772 22.1899 38.6727L22.1492 24.3082C22.1486 24.102 22.2926 23.8525 22.4721 23.7489L34.9714 16.5324ZM35.0132 31.2691L34.9725 16.9047L22.4731 24.1212L22.5139 38.4856L35.0132 31.2691Z" fill="#374874"/>
<path d="M35.3679 33.4351C35.5474 33.3314 35.6923 33.4141 35.6929 33.6203L35.7337 47.9846C35.7342 48.1893 35.5903 48.4403 35.4108 48.544L22.91 55.7613C22.7319 55.8641 22.5855 55.7807 22.5849 55.5761L22.5442 41.2118C22.5436 41.0055 22.6891 40.7552 22.8671 40.6525L35.3679 33.4351ZM35.4097 48.1717L35.369 33.8074L22.8682 41.0247L22.9089 55.389L35.4097 48.1717Z" fill="#374874"/>
<path d="M33.9633 37.1396C34.1361 37.0398 34.3088 37.0192 34.4369 37.1018C34.683 37.2604 34.6712 37.7392 34.4108 38.1717L27.1683 50.2055C27.0437 50.4131 26.8843 50.5724 26.7247 50.6645C26.5584 50.7605 26.392 50.7835 26.2649 50.712L23.8477 49.3327C23.5929 49.1876 23.5916 48.7163 23.8447 48.278C23.9705 48.0601 24.1363 47.8917 24.3023 47.7959C24.4684 47.7 24.6345 47.6767 24.7612 47.7488L24.9338 47.8474L26.7334 48.8745L32.1268 39.9128L33.5204 37.5974C33.6452 37.3901 33.8043 37.2314 33.9633 37.1396Z" fill="#374874"/>
</svg>

```

## File: static\src\js\portal_rating.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { parseDate } from '@web/core/l10n/dates';

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
                var baseDate = parseDate(ratingDate);
                var duration = baseDate.toRelative();
                var $rating = $('#rating_' + id);
                $rating.find('.rating_timeduration').text(duration);
                return $rating.html();
            },
        });
        return this._super.apply(this, arguments);
    },
});

```

## File: static\src\js\tours\project.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { stepUtils } from "@web_tour/tour_service/tour_utils";

import { markup } from "@odoo/owl";

registry.category("web_tour.tours").add('project_tour', {
    sequence: 110,
    url: "/web",
    rainbowManMessage: _t("Congratulations, you are now a master of project management."),
    steps: () => [stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="project.menu_main_pm"]',
    content: markup(_t('Want a better way to <b>manage your projects</b>? <i>It starts here.</i>')),
    position: 'right',
    edition: 'community',
}, {
    trigger: '.o_app[data-menu-xmlid="project.menu_main_pm"]',
    content: markup(_t('Want a better way to <b>manage your projects</b>? <i>It starts here.</i>')),
    position: 'bottom',
    edition: 'enterprise',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_project_kanban',
    content: markup(_t('Let\'s create your first <b>project</b>.')),
    position: 'bottom',
    width: 200,
}, {
    trigger: '.o_project_name input',
    content: markup(_t('Choose a <b>name</b> for your project. <i>It can be anything you want: the name of a customer, of a product, of a team, of a construction site, etc.</i>')),
    position: 'right',
}, {
    trigger: '.o_open_tasks',
    content: markup(_t('Let\'s create your first <b>project</b>.')),
    position: 'top',
    run: function (actions) {
        actions.auto('.modal:visible .btn.btn-primary');
    },
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_header input",
    content: markup(_t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>.")),
    position: 'bottom',
    run: "text Test",
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    content: markup(_t('Let\'s create your first <b>stage</b>.')),
    position: 'right',
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_header input",
    extra_trigger: '.o_kanban_group',
    content: markup(_t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>.")),
    position: 'bottom',
    run: "text Test",
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    content: markup(_t('Let\'s create your second <b>stage</b>.')),
    position: 'right',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_kanban_group:eq(1)',
    content: markup(_t("Let's create your first <b>task</b>.")),
    position: 'bottom',
    width: 200,
}, {
    trigger: '.o_kanban_quick_create div.o_field_char[name=display_name] input',
    extra_trigger: '.o_kanban_project_tasks',
    content: markup(_t('Choose a task <b>name</b> <i>(e.g. Website Design, Purchase Goods...)</i>')),
    position: 'right',
}, {
    trigger: '.o_kanban_quick_create .o_kanban_add',
    extra_trigger: '.o_kanban_project_tasks',
    content: _t("Add your task once it is ready."),
    position: "bottom",
}, {
    trigger: ".o_kanban_record .oe_kanban_content",
    extra_trigger: '.o_kanban_project_tasks',
    content: markup(_t("<b>Drag &amp; drop</b> the card to change your task from stage.")),
    position: "bottom",
    run: "drag_and_drop_native .o_kanban_group:eq(1) ",
}, {
    trigger: ".o_kanban_record:first",
    extra_trigger: '.o_kanban_project_tasks',
    content: _t("Let's start working on your task."),
    position: "bottom",
}, {
    trigger: ".o-mail-Chatter-topbar button.o-mail-Chatter-sendMessage",
    extra_trigger: '.o_form_project_tasks',
    content: markup(_t("Use the chatter to <b>send emails</b> and communicate efficiently with your customers. Add new people to the followers' list to make them aware of the main changes about this task.")),
    width: 350,
    position: "bottom",
}, {
    trigger: "button.o-mail-Chatter-logNote",
    extra_trigger: '.o_form_project_tasks',
    content: markup(_t("<b>Log notes</b> for internal communications <i>(the people following this task won't be notified of the note you are logging unless you specifically tag them)</i>. Use @ <b>mentions</b> to ping a colleague or # <b>mentions</b> to reach an entire team.")),
    width: 350,
    position: "bottom"
}, {
    trigger: ".o-mail-Chatter-topbar button.o-mail-Chatter-activity",
    extra_trigger: '.o_form_project_tasks',
    content: markup(_t("Create <b>activities</b> to set yourself to-dos or to schedule meetings.")),
    position: "bottom",
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
    trigger: ".o_breadcrumb .o_back_button",
    extra_trigger: '.o_form_project_tasks',
    content: markup(_t("Let's go back to the <b>kanban view</b> to have an overview of your next tasks.")),
    position: "right",
    run: 'click',
}, {
    trigger: '.o_kanban_renderer',
    // last step to confirm we've come back before considering the tour successful
    auto: true
}]});

```

## File: static\src\project_sharing\main.js

```javascript
/** @odoo-module **/
import { startWebClient } from '@web/start';
import { ProjectSharingWebClient } from './project_sharing';

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
import { Component, markup, useEffect, useExternalListener, useState } from "@odoo/owl";

export class ProjectSharingWebClient extends Component {
    setup() {
        window.parent.document.body.style.margin = "0"; // remove the margin in the parent body
        this.actionService = useService('action');
        this.user = useService("user");
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
        if (action_name.help) {
            action_name.help = markup(action_name.help);
        }
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

ProjectSharingWebClient.props = {};
ProjectSharingWebClient.components = { ActionContainer, MainComponentsContainer };
ProjectSharingWebClient.template = 'project.ProjectSharingWebClient';

```

## File: static\src\project_sharing\project_sharing.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectSharingWebClient">
        <ActionContainer />
        <MainComponentsContainer/>
    </t>

</templates>

```

## File: static\src\project_sharing\components\chatter\chatter_attachments_viewer.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";

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

    <t t-name="project.ChatterAttachmentsViewer">
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
import { Component, useState, onWillUpdateProps, useRef } from "@odoo/owl";

export class ChatterComposer extends Component {
    setup() {
        this.rpc = useService('rpc');
        this.state = useState({
            displayError: false,
            attachments: this.props.attachments.map(file => file.state === 'done'),
            message: '',
            loading: false,
        });
        this.inputRef = useRef("textarea");

        onWillUpdateProps(this.onWillUpdateProps);
    }

    onWillUpdateProps(nextProps) {
        this.clearErrors();
        this.state.message = '';
        if (this.inputRef.el) {
            this.inputRef.el.value = "";
        }
        this.state.attachments = nextProps.attachments.map(file => file.state === 'done');
    }

    get discussionUrl() {
        return `${window.location.href.split('#')[0]}#discussion`;
    }

    update() {
        this.clearErrors();
        this.state.message = this.inputRef.el.value;
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
    <t t-name="project.ChatterComposer">
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
                                <textarea
                                    class="o_input"
                                    placeholder="Write a message..."
                                    rows="4"
                                    value="state.message"
                                    t-on-input="update"
                                    t-ref="textarea"
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

import { useService } from "@web/core/utils/hooks";
import { ChatterComposer } from "./chatter_composer";
import { ChatterMessageCounter } from "./chatter_message_counter";
import { ChatterMessages } from "./chatter_messages";
import { ChatterPager } from "./chatter_pager";
import { Component, markup, onWillStart, useState, onWillUpdateProps } from "@odoo/owl";

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

    <t t-name="project.ChatterContainer">
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
import { Component } from "@odoo/owl";

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

    <t t-name="project.ChatterMessages">
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
                            <p class="o_portal_chatter_puslished_date">Published on <t t-out="message.published_date_str"/></p>
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

import { Component } from "@odoo/owl";

export class ChatterMessageCounter extends Component { }

ChatterMessageCounter.props = {
    count: Number,
};
ChatterMessageCounter.template = 'project.ChatterMessageCounter';

```

## File: static\src\project_sharing\components\chatter\chatter_message_counter.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.ChatterMessageCounter">
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

import { Component, useState, onWillUpdateProps } from "@odoo/owl";

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

    <t t-name="project.ChatterPager">
        <div class="d-flex justify-content-center">
            <ul class="pagination mb-0 pb-4" t-if="state.pages.length &gt; 1">
                <li t-if="props.page != props.page_previous" t-att-data-page="state.pagePrevious" class="page-item o_portal_chatter_pager_btn">
                    <a t-on-click="() => this.onPageChanged(state.pagePrevious)" class="page-link"><i class="oi oi-chevron-left" role="img" aria-label="Previous" title="Previous"/></a>
                </li>
                <t t-foreach="state.pages" t-as="page" t-key="page_index">
                    <li t-att-data-page="page" t-attf-class="page-item #{page == props.page ? 'o_portal_chatter_pager_btn active' : 'o_portal_chatter_pager_btn'}">
                        <a t-on-click="() => this.onPageChanged(page)" t-att-disabled="page == props.page" class="page-link"><t t-esc="page"/></a>
                    </li>
                </t>
                <li t-if="props.page != state.pageNext" t-att-data-page="state.pageNext" class="page-item o_portal_chatter_pager_btn">
                    <a t-on-click="() => this.onPageChanged(state.pageNext)" class="page-link"><i class="oi oi-chevron-right" role="img" aria-label="Next" title="Next"/></a>
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
import { Component } from "@odoo/owl";

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

    <t t-name="project.PortalAttachDocument">
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

## File: static\src\project_sharing\editor\odoo_editor.js

```javascript
/** @odoo-module **/

import { OdooEditor } from "@web_editor/js/editor/odoo-editor/src/OdooEditor";
import { patch } from "@web/core/utils/patch";

/**
 * The goal of this patch is to remove the crop and replace buttons
 * from the image editor toolbar as the portal user doesn't have
 * access to save modified attachments.
 */
patch(OdooEditor.prototype, {
    /**
     * @override
     */
    _updateToolbar(show) {
        super._updateToolbar(show);
        const isInMedia = this.toolbar.classList.contains('oe-media');
        const cropButton = this.toolbar.querySelector('#image-crop');
        const replaceButton = this.toolbar.querySelector('#media-replace');
        cropButton?.classList.toggle('d-none', isInMedia);
        replaceButton?.classList.toggle('d-none', isInMedia);
    },
});

```

## File: static\src\project_sharing\editor\wysiwyg.js

```javascript
/** @odoo-module **/

import { Wysiwyg } from "@web_editor/js/wysiwyg/wysiwyg";
import { useService } from '@web/core/utils/hooks';
import { patch } from "@web/core/utils/patch";

/**
 * The goal of this patch is to allow portal user to add images in html fields
 */
patch(Wysiwyg.prototype, {
    /**
     * @override
     */
    setup() {
        super.setup();
        this.http = useService('http');
    },
    /**
     * @overwrite
     */
    async _saveB64Image(el, resModel, resId) {
        if (resId) {
            el.classList.remove('o_b64_image_to_save');
            const params = {
                name: el.dataset.fileName || '',
                data: el.getAttribute('src').split('base64,')[1],
                res_id: resId,
                access_token: '',
                csrf_token: odoo.csrf_token,
            };

            const response = JSON.parse(await this.http.post('/project_sharing/attachment/add_image', params, "text"));
            if (response.error) {
                this.notification.add(response.error, { type: 'danger' });
                el.remove();
            }
            else {
                const attachment = response;
                let src = "/web/image/" + attachment.id + "-" + attachment.name;
                if (!attachment.public) {
                    let accessToken = attachment.access_token;
                    src += `?access_token=${encodeURIComponent(accessToken)}`;
                }
                el.setAttribute('src', src);
            }
        }
    },
});

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
    chatterContainerHookXml.classList.add("o-mail-ChatterContainer", 'o-mail-Form-chatter');
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
        const chatterContainerHookXml = res.querySelector(".o-mail-Form-chatter");
        if (chatterContainerHookXml) {
            setAttributes(chatterContainerHookXml, {
                "t-if": `__comp__.uiService.size >= ${SIZES.XXL}`,
            });
            chatterContainerHookXml.classList.add('overflow-x-hidden', 'overflow-y-auto', 'o-aside', 'h-100', 'd-none');
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
                recordExpr: "__comp__.model.root",
            });
            append(compiledRoot, compiledChild);
        }
        return compiledRoot;
    }

    compileChatter(node) {
        return compileChatter(node, {
            resId: '__comp__.model.root.resId or undefined',
            resModel: '__comp__.model.root.resModel',
            projectSharingId: '__comp__.model.root.context.active_id_chatter',
        });
    }
}

registry.category("form_compilers").add("portal_chatter_compiler", {
    selector: "div.oe_chatter",
    fn: (node) =>
        compileChatter(node, {
            resId: "__comp__.props.record.resId or undefined",
            resModel: "__comp__.props.record.resModel",
            projectSharingId: "__comp__.props.record.context.active_id_chatter",
        }),
});

patch(FormCompiler.prototype, {
    compile(node, params) {
        const res = super.compile(node, params);
        const chatterContainerHookXml = res.querySelector('.o-mail-Form-chatter');
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
            't-att-class': `{
                'overflow-x-hidden overflow-y-auto o-aside h-100': __comp__.uiService.size >= ${SIZES.XXL},
                'px-3 py-0': __comp__.uiService.size < ${SIZES.XXL},
            }`,
        });
        append(parentXml, chatterContainerHookXml);
        return res;
    }
});

```

## File: static\src\project_sharing\views\form\project_sharing_form_controller.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { useService } from '@web/core/utils/hooks';
import { createElement } from "@web/core/utils/xml";
import { FormController } from '@web/views/form/form_controller';
import { useViewCompiler } from '@web/views/view_compiler';
import { ProjectSharingChatterCompiler } from './project_sharing_form_compiler';
import { ChatterContainer } from '../../components/chatter/chatter_container';
import { useExternalListener } from "@odoo/owl";

export class ProjectSharingFormController extends FormController {
    setup() {
        super.setup();
        this.uiService = useService('ui');
        this.notification = useService('notification');
        const { xmlDoc } = this.archInfo;
        const template = createElement('t');
        const xmlDocChatter = xmlDoc.querySelector("div.oe_chatter");
        if (xmlDocChatter && xmlDocChatter.parentNode.nodeName === "form") {
            template.appendChild(xmlDocChatter.cloneNode(true));
        }
        const mailTemplates = useViewCompiler(ProjectSharingChatterCompiler, { Mail: template });
        this.mailTemplate = mailTemplates.Mail;
        useExternalListener(window, "paste", this.onGlobalPaste, { capture: true });
        useExternalListener(window, "drop", this.onGlobalDrop, { capture: true });
    }

    get actionMenuItems() {
        return {};
    }

    get translateAlert() {
        return null;
    }

    onGlobalPaste(ev) {
        if (ev.target.closest('.o_field_widget[name="description"]')) {
            ev.preventDefault();
            const items = ev.clipboardData.items;
            for (let i = 0; i < items.length; i++) {
                if (items[i].type.indexOf('image') !== -1 && !this.model.root.resId) {
                    this.notification.add(
                        _t("Save the task to be able to paste images in description"),
                        { type: 'warning' },
                    )
                    ev.stopImmediatePropagation();
                    return;
                }
            }
        }
    }

    onGlobalDrop(ev) {
        if (ev.target.closest('.o_field_widget[name="description"]')) {
            ev.preventDefault();
            if(ev.dataTransfer.files.length > 0 && !this.model.root.resId){
                this.notification.add(
                    _t("Save the task to be able to drag images in description"),
                    { type: 'warning' },
                )
                ev.stopImmediatePropagation();
            }
        }
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
                <t t-call="{{ mailTemplate }}" t-call-context="{ __comp__: Object.assign(Object.create(this), { this: this }) }"/>
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
import { RelationalModel } from "@web/model/relational_model/relational_model";

export class ProjectSharingTaskKanbanModel extends RelationalModel {
    async _webReadGroup(config, firstGroupByName, orderBy) {
        config.context = {
            ...config.context,
            project_kanban: true,
        };
        return super._webReadGroup(...arguments);
    }
}

kanbanView.Model = ProjectSharingTaskKanbanModel;

```

## File: static\src\project_sharing\views\list\list_renderer.js

```javascript
/** @odoo-module */

import { ListRenderer } from "@web/views/list/list_renderer";

export class ProjectSharingListRenderer extends ListRenderer {
    /* TODO: Remove me in master */
    setColumns(columns) {}
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

## File: static\src\utils\project_utils.js

```javascript
/** @odoo-module */

/**
 * List of colors according to the selection value, see `project_update.py`
 */
export const STATUS_COLORS = {
    'on_track': 20,
    'at_risk': 22,
    'off_track': 23,
    'on_hold': 21,
    'done': 24,
};

export const STATUS_COLOR_PREFIX = 'o_status_bubble mx-0 o_color_bubble_';

```

## File: static\src\views\burndown_chart\burndown_chart_model.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { GraphModel } from "@web/views/graph/graph_model";
import { sortBy } from "@web/core/utils/arrays";

export class BurndownChartModel extends GraphModel {
    /**
     * @override
     */
    setup(params) {
        super.setup(params);
        this.stageSeqAndNamePerId = {};
    }

    /**
     * Fetch the sequence of each stage in the project. This function alters this.stageSeqAndNamePerId
     * @protected
     * @param {Object} context
     */
    async _fetchStageInfo(context) {
        const searchDomain =
            !context.active_id || !context.default_project_id
                ? []
                : [["project_ids", "in", context.active_id]];
        const data = await this.orm.webSearchRead("project.task.type", searchDomain, {
            specification: {
                name: {},
                sequence: {},
            },
        });
        const stageSeqAndNamePerId = {};
        for (const { id, name, sequence } of data.records) {
            stageSeqAndNamePerId[id] = { name, sequence };
        }
        return stageSeqAndNamePerId;
    }

    /**
     * @param {SearchParams} searchParams
     */
    async load(searchParams) {
        const { context, groupBy } = searchParams;

        if (groupBy.includes("stage_id")) {
            if (context.stage_name_and_sequence_per_id && context.default_project_id) {
                this.stageSeqAndNamePerId = context.stage_name_and_sequence_per_id;
            } else {
                // if the stage_name_and_sequence_per_id wasn't given by the action (for example if the page is simply reloaded)
                this.stageSeqAndNamePerId = await this._fetchStageInfo(context);
            }
        } else {
            this.stageSeqAndNamePerId = {};
        }
        await super.load(searchParams);
    }

    /**
     * @override
     */
    _prepareData() {
        super._prepareData();
        const { groupBy } = this.searchParams;
        const { mode } = this.metaData;
        if (mode === "line" && groupBy.includes("stage_id")) {
            this.data.datasets = sortBy(this.data.datasets, (dataSet) => {
                const firstIdentifier = [...dataSet.identifiers][0];
                const group = Object.assign(...JSON.parse(firstIdentifier));
                const val = group.stage_id;
                if (Array.isArray(val)) {
                    return this.stageSeqAndNamePerId[val[0]]?.sequence || -1;
                }
                return -1;
            });
        }
    }

    /**
     * @protected
     * @override
     */
    async _loadDataPoints(metaData) {
        metaData.measures.__count.string = _t("# of Tasks");
        return super._loadDataPoints(metaData);
    }
}

```

## File: static\src\views\burndown_chart\burndown_chart_search_model.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
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
        if (this.stageIdSearchItemId && this.searchItems[this.stageIdSearchItemId].groupId == groupId && this.searchItems[this.dateSearchItemId].groupId) {
            this._addGroupByNotification(_t("Date and Stage"));
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
                    this._addGroupByNotification(_t("Date"));
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
            this._addGroupByNotification(_t("Stage"));
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
        const notif = _t("The Burndown Chart must be grouped by");
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

    <t t-name="project.BurndownChartView.Buttons" t-inherit="web.GraphView.Buttons" t-inherit-mode="primary">
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
import { useRef, useEffect } from "@odoo/owl";

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
                        const containerEL = descriptionField.closest(
                            this.getHTMLFieldContainerQuerySelector
                        );
                        const editor = descriptionField.querySelector('.note-editable');
                        const elementToResize = editor || descriptionField;
                        const { top, bottom } = elementToResize.getBoundingClientRect();
                        const { bottom: containerBottom } = containerEL.getBoundingClientRect();
                        const { paddingTop, paddingBottom } = window.getComputedStyle(containerEL);
                        const nonEditableHeight =
                            containerBottom -
                            bottom +
                            parseInt(paddingTop) +
                            parseInt(paddingBottom);
                        const minHeight =
                            document.documentElement.clientHeight - top - nonEditableHeight;
                        elementToResize.style.minHeight = `${minHeight}px`;
                    }
                }
            },
            () => [ref.el, this.ui.size, this.props.record.resId]
        );
    }

    get htmlFieldQuerySelector() {
        return '.oe_form_field.oe_form_field_html';
    }

    get getHTMLFieldContainerQuerySelector() {
        return ".o_form_sheet";
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

## File: static\src\views\project_activity\project_activity_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { activityView } from "@mail/views/web/activity/activity_view";
import { ProjectControlPanel } from "@project/components/project_control_panel/project_control_panel";

export const projectActivityView = {
    ...activityView,
    ControlPanel: ProjectControlPanel,
};
registry.category("views").add("project_activity", projectActivityView);

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

## File: static\src\views\project_project_calendar\project_project_calendar_controller.js

```javascript
/** @odoo-module **/

import { CalendarController } from "@web/views/calendar/calendar_controller";
import { _t } from "@web/core/l10n/translation";

export class ProjectProjectCalendarController extends CalendarController {
    get editRecordDefaultDisplayText() {
        return _t("New Project");
    }
}

```

## File: static\src\views\project_project_calendar\project_project_calendar_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { calendarView } from "@web/views/calendar/calendar_view";
import { ProjectProjectCalendarController } from "./project_project_calendar_controller";

const viewRegistry = registry.category("views");

const projectProjectCalendarView = {
    ...calendarView,
    Controller: ProjectProjectCalendarController,
};

viewRegistry.add("project_project_calendar", projectProjectCalendarView);

```

## File: static\src\views\project_project_kanban\project_project_kanban_header.js

```javascript
/** @odoo-module */

import { KanbanHeader } from "@web/views/kanban/kanban_header";
import { useService } from "@web/core/utils/hooks";

export class ProjectProjectKanbanHeader extends KanbanHeader {
    setup() {
        super.setup();
        this.action = useService("action");
    }

    async deleteGroup() {
        if (this.group.groupByField.name === 'stage_id') {
            const action = await this.group.model.orm.call(
                this.group.groupByField.relation,
                'unlink_wizard',
                [this.group.value],
                { context: this.group.context },
            );
            this.action.doAction(action);
            return;
        }
        super.deleteGroup();
    }
}


```

## File: static\src\views\project_project_kanban\project_project_kanban_renderer.js

```javascript
/** @odoo-module */

import { KanbanRenderer } from "@web/views/kanban/kanban_renderer";
import { ProjectProjectKanbanHeader } from "./project_project_kanban_header";


export class ProjectProjectKanbanRenderer extends KanbanRenderer {}

ProjectProjectKanbanRenderer.components = {
    ...KanbanRenderer.components,
    KanbanHeader: ProjectProjectKanbanHeader,
};

```

## File: static\src\views\project_project_kanban\project_task_kanban_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { kanbanView } from "@web/views/kanban/kanban_view";
import { ProjectProjectKanbanRenderer } from "./project_project_kanban_renderer";

export const projectProjectKanbanView = {
    ...kanbanView,
    Renderer: ProjectProjectKanbanRenderer,
};

registry.category("views").add("project_project_kanban", projectProjectKanbanView);

```

## File: static\src\views\project_project_list\project_project_list_renderer.js

```javascript
/** @odoo-module */

import { ListRenderer } from "@web/views/list/list_renderer";
import { getRawValue } from "@web/views/kanban/kanban_record";

export class ProjectProjectListRenderer extends ListRenderer {
    isCellReadonly(column, record) {
        let readonly = super.isCellReadonly(column, record);
        const { selection } = this.props.list;
        if (column.name === "stage_id" && selection.length) {
            const companyId = getRawValue(selection[0], "company_id");
            readonly = selection.some(
                (task) => getRawValue(task, "company_id") !== companyId
            );
        }
        return readonly;
    }
}

```

## File: static\src\views\project_project_list\project_project_list_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { listView } from '@web/views/list/list_view';
import { ProjectProjectListRenderer } from "./project_project_list_renderer";

export const projectProjectListView = {
    ...listView,
    Renderer: ProjectProjectListRenderer,
};

registry.category("views").add("project_project_list", projectProjectListView);

```

## File: static\src\views\project_rating_graph\project_rating_graph_view.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { GraphArchParser } from "@web/views/graph/graph_arch_parser";
import { graphView } from "@web/views/graph/graph_view";

const viewRegistry = registry.category("views");

const MEASURE_STRINGS = {
    parent_res_id: _t("Project"),
    rating: _t("Rating Value (/5)"),
    res_id: _t("Task"),
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

## File: static\src\views\project_rating_pivot\project_rating_pivot_view.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { PivotArchParser } from "@web/views/pivot/pivot_arch_parser";
import { pivotView } from "@web/views/pivot/pivot_view";

const viewRegistry = registry.category("views");

const MEASURE_STRINGS = {
    parent_res_id: _t("Project"),
    rating: _t("Rating Value (/5)"),
    res_id: _t("Task"),
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

## File: static\src\views\project_task_calendar\project_task_calendar_controller.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { CalendarController } from "@web/views/calendar/calendar_controller";
import { ProjectTaskCalendarFilterPanel } from "./project_task_calendar_filter_panel/project_task_calendar_filter_panel";
import { DeleteSubtasksConfirmationDialog } from "@project/components/delete_subtasks_confirmation_dialog/delete_subtasks_confirmation_dialog";

export class ProjectTaskCalendarController extends CalendarController {
    static components = {
        ...ProjectTaskCalendarController.components,
        FilterPanel: ProjectTaskCalendarFilterPanel,
    };
    setup() {
        super.setup(...arguments);
        this.env.config.setDisplayName(this.env.config.getDisplayName() + _t(" - Tasks by Deadline"));
    }

    get editRecordDefaultDisplayText() {
        return _t("New Task");
    }

    deleteRecord(record) {
        if  (!record.rawRecord.subtask_count) {
            return super.deleteRecord(record);
        }
        this.displayDialog(DeleteSubtasksConfirmationDialog, {
            confirm: () => {
                this.model.unlinkRecord(record.id);
            },
        });
    }
}

```

## File: static\src\views\project_task_calendar\project_task_calendar_model.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { CalendarModel } from '@web/views/calendar/calendar_model';

export class ProjectTaskCalendarModel extends CalendarModel {
    /**
     * @override
     */
    get defaultFilterLabel() {
        this.isCheckProject = 'project_id' in this.meta.filtersInfo;
        if (this.isCheckProject) {
            return _t("Private");
        }
        return super.defaultFilterLabel;
    }
}

```

## File: static\src\views\project_task_calendar\project_task_calendar_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { calendarView } from "@web/views/calendar/calendar_view";
import { ProjectControlPanel } from "@project/components/project_control_panel/project_control_panel";
import { ProjectTaskCalendarController } from "./project_task_calendar_controller";
import { ProjectTaskCalendarModel } from "./project_task_calendar_model";

export const projectTaskCalendarView = {
    ...calendarView,
    Controller: ProjectTaskCalendarController,
    ControlPanel: ProjectControlPanel,
    Model: ProjectTaskCalendarModel,
};
registry.category("views").add("project_task_calendar", projectTaskCalendarView);

```

## File: static\src\views\project_task_calendar\project_task_calendar_filter_panel\project_task_calendar_filter_panel.js

```javascript
/** @odoo-module **/

import { CalendarFilterPanel } from "@web/views/calendar/filter_panel/calendar_filter_panel";

export class ProjectTaskCalendarFilterPanel extends CalendarFilterPanel { }

ProjectTaskCalendarFilterPanel.subTemplates = {
    filter: "project.ProjectTaskCalendarFilterPanel.filter",
};

```

## File: static\src\views\project_task_calendar\project_task_calendar_filter_panel\project_task_calendar_filter_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project.ProjectTaskCalendarFilterPanel.filter" t-inherit="web.CalendarFilterPanel.filter" t-inherit-mode="primary">
        <xpath expr="//span[@t-esc='filter.label']" position="before">
            <span t-if="props.model.isCheckProject and !filter.value" class="text-danger pe-1">
                <i class="fa fa-lock"/>
            </span>
        </xpath>
    </t>

</templates>

```

## File: static\src\views\project_task_form\project_task_form_controller.js

```javascript
/** @odoo-module */

import { FormController } from '@web/views/form/form_controller';
import { DeleteSubtasksConfirmationDialog } from "@project/components/delete_subtasks_confirmation_dialog/delete_subtasks_confirmation_dialog";

export class ProjectTaskFormController extends FormController {
    deleteRecord() {
        if (!this.model.root.data.subtask_count) {
            return super.deleteRecord();
        }
        this.dialogService.add(DeleteSubtasksConfirmationDialog, {
            confirm: async () => {
                await this.model.root.delete();
                if (!this.model.root.resId) {
                    this.env.config.historyBack();
                }
            },
        });
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
import { ProjectTaskFormController } from "./project_task_form_controller";
import { ProjectTaskFormRenderer } from "./project_task_form_renderer";

export const projectTaskFormView = {
    ...formViewWithHtmlExpander,
    Controller: ProjectTaskFormController,
    Renderer: ProjectTaskFormRenderer,
};

registry.category("views").add("project_task_form", projectTaskFormView);

```

## File: static\src\views\project_task_graph\project_task_graph_model.js

```javascript
/** @odoo-module **/

import { GraphModel } from "@web/views/graph/graph_model";
import { _t } from "@web/core/l10n/translation";

export class ProjectTaskGraphModel extends GraphModel {
    _getDefaultFilterLabel(field) {
        if (field.fieldName === "project_id") {
            return _t("🔒 Private");
        }
        return super._getDefaultFilterLabel(field);
    }
}

```

## File: static\src\views\project_task_graph\project_task_graph_view.js

```javascript
/** @odoo-module **/

import { ProjectControlPanel } from "@project/components/project_control_panel/project_control_panel";
import { registry } from "@web/core/registry";
import { graphView } from "@web/views/graph/graph_view";
import { ProjectTaskGraphModel } from "./project_task_graph_model";

const viewRegistry = registry.category("views");

export const projectTaskGraphView = {
    ...graphView,
    ControlPanel: ProjectControlPanel,
    Model: ProjectTaskGraphModel,
};

viewRegistry.add("project_task_graph", projectTaskGraphView);

```

## File: static\src\views\project_task_kanban\project_task_kanban_compiler.js

```javascript
/** @odoo-module **/

import { KanbanCompiler } from "@web/views/kanban/kanban_compiler";
import { append, createElement } from "@web/core/utils/xml";

export class ProjectTaskKanbanCompiler extends KanbanCompiler {
    setup() {
        super.setup();
        this.subtaskListComponentCompiled = {
            button: false,
            component: false,
        };
        this.compilers.push(
            { selector: ".subtask_list_button", fn: this.compileSubtaskListButton },
            { selector: "div.kanban_bottom_subtasks_section", fn: this.compileSubtaskListComponent },
        );
    }

    /**
     * @param {Element} el
     * @returns {Element}
     */
    compileSubtaskListButton(el) {
        this.subtaskListComponentCompiled.button = true;
        el.setAttribute("t-on-click", `() => __comp__.state.folded = !__comp__.state.folded`);
        const compiled = createElement(el.nodeName);
        for (const { name, value } of el.attributes) {
            compiled.setAttribute(name, value);
        }

        for (const child of el.childNodes) {
            append(compiled, this.compileNode(child));
        }

        return compiled;
    }

    /**
     * @param {Element} el
     * @returns {Element}
     */
    compileSubtaskListComponent(el) {
        this.subtaskListComponentCompiled.component = true;
        el.setAttribute("t-if", `!__comp__.state.folded and !selection_mode`);
        const compiled = createElement(el.nodeName);
        for (const { name, value } of el.attributes) {
            compiled.setAttribute(name, value);
        }
        const listContainer = createElement('widget');
        const listElemenent = createElement('SubtaskKanbanList');
        listElemenent.setAttribute("record", '__comp__.props.record');

        append(listContainer, listElemenent);
        append(compiled, listContainer);

        return compiled;
    }

    /**
     * @override
     */
    compile(key, params = {}) {
        const newRoot = super.compile(key, params);
        if (this.subtaskListComponentCompiled.component !== this.subtaskListComponentCompiled.button) {
            // Error since one of them is not compiled
            throw new Error("The subtask list component cannot be rendered if the button and the component are not in the view definition.");
        }
        return newRoot;
    }
}

```

## File: static\src\views\project_task_kanban\project_task_kanban_examples.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { renderToMarkup } from '@web/core/utils/render';
import { markup } from "@odoo/owl";

const greenBullet = markup(`<span class="o_status d-inline-block o_status_green"></span>`);
const orangeBullet = markup(`<span class="o_status d-inline-block text-warning"></span>`);
const star = markup(`<a style="color: gold;" class="fa fa-star"></a>`);
const clock = markup(`<a class="fa fa-clock-o"></a>`);

const exampleData = {
    ghostColumns: [_t('New'), _t('Assigned'), _t('In Progress'), _t('Done')],
    applyExamplesText: _t("Use This For My Project"),
    allowedGroupBys: ['stage_id'],
    foldField: "fold",
    examples:[{
        name: _t('Software Development'),
        columns: [_t('Backlog'), _t('Specifications'), _t('Development'), _t('Tests')],
        foldedColumns: [_t('Delivered')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, orangeBullet, star],
    }, {
        name: _t('Agile Scrum'),
        columns: [_t('Backlog'), _t('Sprint Backlog'), _t('Sprint in Progress')],
        foldedColumns: [_t('Sprint Complete'), _t('Old Completed Sprint')],
        get description() {
            return renderToMarkup("project.example.agilescrum");
        },
        bullets: [greenBullet, orangeBullet],
    }, {
        name: _t('Digital Marketing'),
        columns: [_t('Ideas'), _t('Researching'), _t('Writing'), _t('Editing')],
        foldedColumns: [_t('Done')],
        get description() {
            return renderToMarkup("project.example.digitalmarketing");
        },
        bullets: [greenBullet, orangeBullet],
    }, {
        name: _t('Customer Feedback'),
        columns: [_t('New'), _t('In development')],
        foldedColumns: [_t('Done'), _t('Refused')],
        get description() {
            return renderToMarkup("project.example.customerfeedback");
        },
        bullets: [greenBullet, orangeBullet],
    }, {
        name: _t('Consulting'),
        columns: [_t('New Projects'), _t('Resources Allocation'), _t('In Progress')],
        foldedColumns: [_t('Done')],
        get description() {
            return renderToMarkup("project.example.consulting");
        },
        bullets: [greenBullet, orangeBullet],
    }, {
        name: _t('Research Project'),
        columns: [_t('Brainstorm'), _t('Research'), _t('Draft')],
        foldedColumns: [_t('Final Document')],
        get description() {
            return renderToMarkup("project.example.researchproject");
        },
        bullets: [greenBullet, orangeBullet],
    }, {
        name: _t('Website Redesign'),
        columns: [_t('Page Ideas'), _t('Copywriting'), _t('Design')],
        foldedColumns: [_t('Live')],
        get description() {
            return renderToMarkup("project.example.researchproject");
        },
    }, {
        name: _t('T-shirt Printing'),
        columns: [_t('New Orders'), _t('Logo Design'), _t('To Print')],
        foldedColumns: [_t('Done')],
        get description() {
            return renderToMarkup("project.example.tshirtprinting");
        },
        bullets: [star],
    }, {
        name: _t('Design'),
        columns: [_t('New Request'), _t('Design'), _t('Client Review')],
        foldedColumns: [_t('Handoff')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, orangeBullet, star, clock],
    }, {
        name: _t('Publishing'),
        columns: [_t('Ideas'), _t('Writing'), _t('Editing')],
        foldedColumns: [_t('Published')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, orangeBullet, star, clock],
    }, {
        name: _t('Manufacturing'),
        columns: [_t('New Orders'), _t('Material Sourcing'), _t('Manufacturing'), _t('Assembling')],
        foldedColumns: [_t('Delivered')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, orangeBullet, star, clock],
    }, {
        name: _t('Podcast and Video Production'),
        columns: [_t('Research'), _t('Script'), _t('Recording'), _t('Mixing')],
        foldedColumns: [_t('Published')],
        get description() {
            return renderToMarkup("project.example.generic");
        },
        bullets: [greenBullet, orangeBullet, star, clock],
    }],
};

registry.category("kanban_examples").add('project', exampleData);

```

## File: static\src\views\project_task_kanban\project_task_kanban_header.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useService } from '@web/core/utils/hooks';
import { KanbanHeader } from "@web/views/kanban/kanban_header";
import { onWillStart } from "@odoo/owl";

export class ProjectTaskKanbanHeader extends KanbanHeader {
    setup() {
        super.setup();
        this.action = useService('action');
        this.userService = useService('user');

        this.isProjectManager = false;
        onWillStart(this.onWillStart);
    }

    async onWillStart() {
        if (this.props.list.isGroupedByStage) { // no need to check it if not grouped by stage
            this.isProjectManager = await this.userService.hasGroup('project.group_project_manager');
        }
    }

    async deleteGroup() {
        if (this.group.groupByField.name === 'stage_id') {
            const action = await this.group.model.orm.call(
                this.group.groupByField.relation,
                'unlink_wizard',
                [this.group.value],
                { context: this.group.context },
            );
            this.action.doAction(action);
            return;
        }
        super.deleteGroup();
    }

    canEditGroup(group) {
        return super.canEditGroup(group) && (!this.props.list.isGroupedByStage || this.isProjectManager);
    }

    canDeleteGroup(group) {
        return super.canDeleteGroup(group) && (!this.props.list.isGroupedByStage || this.isProjectManager);
    }

    /**
     * @override
     */
    _getEmptyGroupLabel(fieldName) {
        if (fieldName === "project_id") {
            return _t("🔒 Private");
        } else if (fieldName === "user_ids") {
            return _t("👤 Unassigned");
        } else {
            return super._getEmptyGroupLabel(fieldName);
        }
    }
}

```

## File: static\src\views\project_task_kanban\project_task_kanban_model.js

```javascript
/** @odoo-module */

import { RelationalModel } from "@web/model/relational_model/relational_model";

export class ProjectTaskKanbanDynamicGroupList extends RelationalModel.DynamicGroupList {
    get isGroupedByStage() {
        return !!this.groupByField && this.groupByField.name === "stage_id";
    }
}

export class ProjectTaskKanbanModel extends RelationalModel {
    async _webReadGroup(config, firstGroupByName, orderBy) {
        config.context = {
            ...config.context,
            project_kanban: true,
        };
        return super._webReadGroup(...arguments);
    }
}

ProjectTaskKanbanModel.DynamicGroupList = ProjectTaskKanbanDynamicGroupList;

```

## File: static\src\views\project_task_kanban\project_task_kanban_record.js

```javascript
/* @odoo-module */

import { KanbanRecord } from "@web/views/kanban/kanban_record";
import { useState } from "@odoo/owl";
import { ProjectTaskKanbanCompiler } from "./project_task_kanban_compiler";
import { SubtaskKanbanList } from "@project/components/subtask_kanban_list/subtask_kanban_list"

export class ProjectTaskKanbanRecord extends KanbanRecord {
    setup() {
        super.setup();
        this.state = useState({folded: true});
    }

    /**
     * @override
     */
    get renderingContext() {
        const context = super.renderingContext;
        context["state"] = this.state;
        return context;
    }
}

ProjectTaskKanbanRecord.Compiler = ProjectTaskKanbanCompiler;
ProjectTaskKanbanRecord.components = {
    ...KanbanRecord.components,
    SubtaskKanbanList,
};

```

## File: static\src\views\project_task_kanban\project_task_kanban_renderer.js

```javascript
/** @odoo-module */

import { KanbanRenderer } from '@web/views/kanban/kanban_renderer';
import { ProjectTaskKanbanRecord } from './project_task_kanban_record';
import { ProjectTaskKanbanHeader } from './project_task_kanban_header';
import { useService } from '@web/core/utils/hooks';
import { onWillStart } from "@odoo/owl";

export class ProjectTaskKanbanRenderer extends KanbanRenderer {
    setup() {
        super.setup();
        this.action = useService('action');
        const user = useService("user");

        onWillStart(async () => {
            this.isProjectManager = await user.hasGroup('project.group_project_manager');
        });
    }

    canCreateGroup() {
        // This restrict the creation of project stages to the kanban view of a given project
        return super.canCreateGroup() && (this.isProjectTasksContext() == this.props.list.isGroupedByStage
            && this.isProjectManager || this.props.list.groupByField.name === 'personal_stage_type_id');
    }

    isProjectTasksContext() {
        return (
            ["project.project", "project.task.type.delete.wizard"].includes(
                this.props.list.context.active_model
            ) && !!this.props.list.context.default_project_id
        );
    }
}

ProjectTaskKanbanRenderer.components = {
    ...KanbanRenderer.components,
    KanbanRecord: ProjectTaskKanbanRecord,
    KanbanHeader: ProjectTaskKanbanHeader,
};

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

import { ListController } from "@web/views/list/list_controller";
import { DeleteSubtasksConfirmationDialog } from "@project/components/delete_subtasks_confirmation_dialog/delete_subtasks_confirmation_dialog";

export class ProjectTaskListController extends ListController {
    async onDeleteSelectedRecords() {
        if (!Math.max(...this.model.root.selection.map((record) => record.data.subtask_count ))) {
            return super.onDeleteSelectedRecords();
        }
        this.dialogService.add(DeleteSubtasksConfirmationDialog, {
            confirm: async () => {
                await this.model.root.deleteRecords();
                // A re-load is needed to remove deleted sub-tasks from the view
                await this.model.load();
            },
        });
    }
}

```

## File: static\src\views\project_task_list\project_task_list_renderer.js

```javascript
/** @odoo-module */

import { ListRenderer } from "@web/views/list/list_renderer";
import { getRawValue } from "@web/views/kanban/kanban_record";
import { _t } from "@web/core/l10n/translation";

export class ProjectTaskListRenderer extends ListRenderer {
    /**
     * This method prevents from computing the selection once for each cell when
     * rendering the list. Indeed, `selection` is a getter which browses all
     * records, so computing it for each cell slows down the rendering a lot on
     * large tables. Moreover, it also prevents from iterating over the selection
     * to compare tasks' projects.
     *
     * It returns true iff the selected tasks are all in the same project.
     */
    areSelectedTasksInSameProject() {
        if (this._areSelectedTasksInSameProject === undefined) {
            const selection = this.props.list.selection;
            const projectId = selection.length && getRawValue(selection[0], "project_id");
            this._areSelectedTasksInSameProject = selection.every(
                (task) => getRawValue(task, "project_id") === projectId
            );
            Promise.resolve().then(() => {
                delete this._areSelectedTasksInSameProject;
            });
        }
        return this._areSelectedTasksInSameProject;
    }
    isCellReadonly(column, record) {
        let readonly = false;
        if (column.name === "stage_id") {
            readonly = !this.areSelectedTasksInSameProject();
        }
        return readonly || super.isCellReadonly(column, record);
    }

    getGroupDisplayName(group) {
        if (group.groupByField.name === "project_id" && !group.value) {
            return _t("🔒 Private");
        } else if (group.groupByField.name === "user_ids" && !group.value) {
            return _t("👤 Unassigned");
        } else {
            return super.getGroupDisplayName(group);
        }
    }
}

```

## File: static\src\views\project_task_list\project_task_list_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { listView } from '@web/views/list/list_view';
import { ProjectControlPanel } from "../../components/project_control_panel/project_control_panel";
import { ProjectTaskListController } from "./project_task_list_controller";
import { ProjectTaskListRenderer } from "./project_task_list_renderer";

export const projectTaskListView = {
    ...listView,
    ControlPanel: ProjectControlPanel,
    Controller: ProjectTaskListController,
    Renderer: ProjectTaskListRenderer,
};

registry.category("views").add("project_task_list", projectTaskListView);

```

## File: static\src\views\project_task_pivot\project_pivot_model.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { PivotModel } from "@web/views/pivot/pivot_model";

export class ProjectTaskPivotModel extends PivotModel {
    /**
     * @override
     */
    _getEmptyGroupLabel(fieldName) {
        if (fieldName === "project_id") {
            return _t("Private");
        } else if (fieldName === "user_ids") {
            return _t("Unassigned");
        } else {
            return super._getEmptyGroupLabel(fieldName);
        }
    }
}

```

## File: static\src\views\project_task_pivot\project_pivot_view.js

```javascript
/** @odoo-module **/

import { ProjectControlPanel } from "@project/components/project_control_panel/project_control_panel";
import { registry } from "@web/core/registry";
import { pivotView } from "@web/views/pivot/pivot_view";
import { ProjectTaskPivotModel } from "./project_pivot_model";

const projectPivotView = {
    ...pivotView,
    ControlPanel: ProjectControlPanel,
    Model: ProjectTaskPivotModel,
};

registry.category("views").add("project_pivot", projectPivotView);

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

    <t t-name="project.ProjectUpdateKanbanView" t-inherit="web.KanbanView" t-inherit-mode="primary">
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

    <t t-name="project.ProjectUpdateListView" t-inherit="web.ListView" t-inherit-mode="primary">
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

    <t t-name="project.example.generic">
      Prioritize your tasks by marking important ones using the <a style="color: gold;" class="fa fa-star"></a> button.
      <br/><br/>
      Use the <span class="o_status d-inline-block o_status_green"></span> state to inform your colleagues that a task is approved for the next stage. 
      <br/>
      Use the <span class="o_status d-inline-block bg-warning"></span> state to indicate a request for changes or a need for discussion on a task.
      <br/><br/>
      Use the <span class="fa fa-check-circle text-success d-inline-block"></span> state to mark the task as complete.
      <br/>
      Use the <span class="fa fa-times-circle text-danger d-inline-block"></span> state to mark the task as canceled.
      <br/><br/>
      Look for the <span class="fa fa-hourglass-o d-inline-block"></span> icon to see tasks waiting on other ones. Once a task is marked as complete or canceled, all of its dependencies will be unblocked.
      <br/><br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.agilescrum">
      Use the <span class="o_status d-inline-block o_status_green"></span> state to inform your colleagues that a task is approved for the next stage. 
      <br/>
      Use the <span class="o_status d-inline-block bg-warning"></span> state to indicate a request for changes or a need for discussion on a task.
      <br/><br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.digitalmarketing">
      Everyone can propose ideas, and the Editor marks the best ones as <span class="o_status d-inline-block o_status_green"></span>.<br/>
      Attach all documents or links to the task directly, to have all research information centralized. 
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>
  
    <t t-name="project.example.customerfeedback">
      Customers propose feedbacks by email; Odoo creates tasks automatically, and you can
      communicate on the task directly. <br/>Your managers decide which feedback is accepted
      <span class="o_status d-inline-block o_status_green"></span> and which feedback is
      moved to the "Refused" column.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.consulting">
      Manage the lifecycle of your project using the kanban view. Add newly acquired projects,
      assign them and use the <span class="o_status d-inline-block o_status_green"></span> and
      <span class="o_status d-inline-block bg-warning"></span> to define if the project is
      ready for the next step.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.researchproject">
      Handle your idea gathering within Tasks of your new Project and discuss them in the chatter of the tasks. <br/>Use the
      <span class="o_status d-inline-block o_status_green"></span> and <span class="o_status d-inline-block bg-warning"></span>
      to signalize what is the current status of your Idea.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.tshirtprinting">
      Communicate with customers on the task using the email gateway. Attach logo designs to the task, so that information flows from
      designers to the workers who print the t-shirt. <br/>Organize priorities amongst orders using the
      <a style="color: gold;" class="fa fa-star"></a> icon.
      <br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
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

## File: views\account_analytic_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_analytic_account_view_form_inherit" model="ir.ui.view">
        <field name="name">account.analytic.account.form.inherit</field>
        <field name="model">account.analytic.account</field>
        <field name="inherit_id" ref="analytic.view_account_analytic_account_form"/>
        <field eval="40" name="priority"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(analytic.account_analytic_line_action)d']" position="before">
                <button class="oe_stat_button" type="object" name="action_view_projects"
                    icon="fa-puzzle-piece" invisible="project_count == 0">
                    <field string="Projects" name="project_count" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\digest_digest_views.xml

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

## File: views\mail_activity_plan_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mail_activity_plan_view_form_project_and_task" model="ir.ui.view">
            <field name="name">mail.activity.plan.view.form.project.and.task</field>
            <field name="model">mail.activity.plan</field>
            <field name="mode">primary</field>
            <field name="priority">32</field>
            <field name="inherit_id" ref="mail.mail_activity_plan_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='res_model']" position="attributes">
                    <attribute name="widget">filterable_selection</attribute>
                    <attribute name="options">{'whitelisted_values': ['project.project', 'project.task']}</attribute>
                </xpath>
                <xpath expr="//field[@name='template_ids']/tree" position="attributes">
                    <attribute name="editable">bottom</attribute>
                </xpath>
            </field>
        </record>

        <record id="mail_activity_plan_action_config_project_task_plan" model="ir.actions.act_window">
            <field name="name">Activity Plans</field>
            <field name="res_model">mail.activity.plan</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="mail.mail_activity_plan_view_search"/>
            <field name="context">{'default_res_model': 'project.task'}</field>
            <field name="domain">[('res_model', 'in', ('project.project', 'project.task'))]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new plan
                </p>
            </field>
        </record>

        <record id="mail_activity_plan_action_project_task_view_tree" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="mail.mail_activity_plan_view_tree"/>
            <field name="act_window_id" ref="project.mail_activity_plan_action_config_project_task_plan"/>
        </record>

        <!-- Force the project view that allows to modify the target models of the plan to project or task. -->
        <record id="mail_activity_plan_action_project_task_view_form" model="ir.actions.act_window.view">
            <field name="sequence">2</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="project.mail_activity_plan_view_form_project_and_task"/>
            <field name="act_window_id" ref="project.mail_activity_plan_action_config_project_task_plan"/>
        </record>

        <record id="mail_activity_plan_action_config_task_plan" model="ir.actions.act_window">
            <field name="name">Task Plans</field>
            <field name="res_model">mail.activity.plan</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="mail.mail_activity_plan_view_search"/>
            <field name="context">{'default_res_model': 'project.task'}</field>
            <field name="domain">[('res_model', '=', 'project.task')]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new plan
                </p>
            </field>
        </record>
    </data>
</odoo>
```

## File: views\mail_activity_type_views.xml

```xml
<?xml version="1.0"?>
<odoo>
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
                <header>
                    <button name="%(project_share_wizard_action)d" class="btn-primary" type="action" string="Invite Collaborators"
                            context="{'default_access_mode': 'edit', 'default_project_id': context.get('active_id')}" display="always"/>
                </header>
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

## File: views\project_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem
        name="Project"
        id="menu_main_pm"
        groups="group_project_manager,group_project_user"
        web_icon="project,static/description/icon.png"
        sequence="70"
    >
        <menuitem
            name="Projects"
            id="menu_projects"
            action="open_view_project_all"
            sequence="1"
        />
        <menuitem
            name="Projects"
            id="menu_projects_group_stage"
            action="open_view_project_all_group_stage"
            groups="project.group_project_stages"
            sequence="1"
        />
        <menuitem
            name="Tasks"
            id="menu_project_management"
            sequence="2"
        >
            <menuitem
                name="My Tasks"
                id="menu_project_management_my_tasks"
                action="action_server_view_my_task"
                sequence="1"
            />
            <menuitem
                name="All Tasks"
                id="menu_project_management_all_tasks"
                action="action_view_all_task"
                sequence="2"
            />
        </menuitem>
        <menuitem
            name="Reporting"
            id="menu_project_report"
            sequence="99"
        >
            <menuitem
                name="Tasks Analysis"
                id="menu_project_report_task_analysis"
                action="project.action_project_task_user_tree"
                sequence="10"
            />
            <menuitem
                name="Customer Ratings"
                id="rating_rating_menu_project"
                action="rating_rating_action_project_report"
                groups="project.group_project_rating"
                sequence="51"
            />

        </menuitem>
        <menuitem
            name="Configuration"
            id="menu_project_config"
            groups="project.group_project_manager"
            sequence="100"
        >
            <menuitem
                name="Settings"
                id="project_config_settings_menu_action"
                action="project_config_settings_action"
                groups="base.group_system"
                sequence="0"
            />
            <menuitem
                name="Projects"
                id="menu_projects_config_group_stage"
                action="open_view_project_all_config_group_stage"
                groups="project.group_project_stages"
                sequence="5"
            />
            <menuitem
                name="Projects"
                id="menu_projects_config"
                action="open_view_project_all_config"
                sequence="5"
            />
            <menuitem
                name="Project Stages"
                id="menu_project_config_project_stage"
                action="project_project_stage_configure"
                groups="project.group_project_stages"
                sequence="9"
            />
            <menuitem
                name="Task Stages"
                id="menu_project_config_project"
                action="open_task_type_form"
                groups="base.group_no_one"
                sequence="10"
            />
            <menuitem
                name="Tags"
                id="menu_project_tags_act"
                action="project_tags_action"
            />
            <menuitem
                name="Activity Types"
                id="project_menu_config_activity_type"
                action="mail_activity_type_action_config_project_types"
            />
            <menuitem
                name="Activity Plans"
                id="mail_activity_plan_menu_config_project"
                action="mail_activity_plan_action_config_project_task_plan"
            />
        </menuitem>
    </menuitem>
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
                                context="{'default_project_id': project_id}"
                                groups="project.group_project_milestone"
                                close="1"
                        >
                            <div class="o_form_field o_stat_info">
                                <span class="o_stat_value">
                                    <field name="task_count" nolabel="1"/>
                                    <span class="fw-normal"> Tasks</span>
                                </span>
                                <span class="o_stat_value" invisible="done_task_count == 0">
                                    <field name="done_task_count" nolabel="1"/>
                                    <span class="fw-normal"> Done</span>
                                </span>
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
                <field name="is_deadline_exceeded" column_invisible="True"/>
                <field name="task_count" column_invisible="True" />
                <field name="can_be_marked_as_done" column_invisible="True"/>
                <button name="action_view_tasks"
                        type="object"
                        title="View Tasks"
                        string="View Tasks"
                        class="btn btn-link float-end"
                        invisible="task_count == 0"
                        groups="project.group_project_milestone"/>
            </tree>
        </field>
    </record>
</odoo>

```

## File: views\project_portal_project_project_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="portal_layout" name="Portal layout: project menu entry" inherit_id="portal.portal_breadcrumbs" priority="40">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <li t-if="page_name == 'project' or project" class="col-lg-2" t-attf-class="breadcrumb-item #{'active ' if not project else ''}">
                <a t-if="project" t-attf-href="/my/projects">Projects</a>
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
            <li t-elif="task" class="breadcrumb-item active text-break">
                <span t-field="task.name"/>
            </li>
            <li t-if="page_name == 'project_subtasks' or (task and subtask and project)" t-attf-class="breadcrumb-item text-truncate #{'active ' if not subtask else ''}">
                <a t-if="subtask" t-attf-href="/my/tasks/{{ task.id }}/subtasks?{{ keep_query() }}">Sub-tasks</a>
                <t t-else="">Sub-tasks</t>
            </li>
            <li t-if="subtask" class="breadcrumb-item active text-break">
                <span t-field="subtask.name"/>
            </li>
        </xpath>
    </template>

    <template id="portal_my_home" name="Show Projects / Tasks" customize_show="True" inherit_id="portal.portal_my_home" priority="40">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="before">
            <t t-set="portal_service_category_enable" t-value="True"/>
        </xpath>
        <div id="portal_service_category" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="icon" t-value="'/project/static/src/img/folder.svg'"/>
                <t t-set="title">Projects</t>
                <t t-set="url" t-value="'/my/projects'"/>
                <t t-set="text">Follow the evolution of your projects</t>
                <t t-set="placeholder_count" t-value="'project_count'"/>
            </t>
            <t t-call="portal.portal_docs_entry">
                <t t-set="icon" t-value="'/project/static/src/img/tasks.svg'"/>
                <t t-set="title">Tasks</t>
                <t t-set="url" t-value="'/my/tasks'"/>
                <t t-set="text">Follow and comments tasks of your projects</t>
                <t t-set="placeholder_count" t-value="'task_count'"/>
            </t>
        </div>
    </template>

    <template id="portal_my_projects" name="My Projects">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Projects</t>
            </t>
            <t t-if="not projects">
                <div class="alert alert-warning" role="alert">
                    There are no projects.
                </div>
            </t>
            <t t-if="projects" t-call="portal.portal_table">
                <tbody>
                    <tr t-foreach="projects" t-as="project">
                        <td>
                            <a t-attf-href="/my/projects/#{project.id}"><span t-field="project.name"/></a>
                        </td>
                        <td class="text-end">
                            <t t-out="project.task_count" />
                            <t t-out="project.label_tasks" />
                        </td>
                    </tr>
                </tbody>
            </t>
        </t>
    </template>

    <template id="portal_my_project" name="My Project">
        <t t-call="portal.portal_layout">
            <t t-set="title" t-value="project.name"/>
            <t t-set="o_portal_fullwidth_alert" groups="project.group_project_user">
                <t t-call="portal.portal_back_in_edit_mode">
                    <t t-set="backend_url" t-value="'/web#model=project.project&amp;id=%s&amp;view_type=kanban' % (project.id)"/>
                </t>
            </t>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Tasks</t>
            </t>
            <t t-if="not grouped_tasks">
                <div class="alert alert-warning" role="alert">
                    There are no tasks.
                </div>
            </t>

            <t t-call="project.portal_tasks_list"/>
        </t>
    </template>
</odoo>

```

## File: views\project_portal_project_task_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="portal_my_tasks_priority_widget_template" name="Priority Widget Template">
        <span t-attf-class="o_priority_star fa fa-star#{'' if task.priority == '1' else '-o'} #{classes if classes else ''}" t-attf-title="Priority: {{'Important' if task.priority == '1' else 'Normal'}}"/>
    </template>

    <template id="portal_my_tasks_state_widget_template" name="Status Widget Template">
        <span
            t-att-title="dict(task.fields_get(allfields=['state'])['state']['selection'])[task.state]"
            t-attf-class="#{'fa' if task.state in ['1_done','1_canceled','04_waiting_normal'] else 'o_status rounded-circle' } #{'fa-check-circle text-success' if task.state == '1_done' else 'fa-times-circle text-danger' if task.state == '1_canceled' else 'bg-warning' if task.state == '02_changes_requested' else 'bg-success' if task.state == '03_approved' else 'fa-hourglass-o' if task.state == '04_waiting_normal' else ''}"
        />
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
                                    <span t-if="tasks[0].sudo().project_id" t-field="tasks[0].sudo().project_id.name"/>
                                    <span t-else="">No Project</span>
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
                                <span class="text-truncate" t-field="tasks[0].sudo().state"/></th>
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
                                    <a t-attf-href="/my/#{task_url}/#{task.id}?{{ keep_query() }}"><span t-att-title="task.name" t-field="task.name"/></a>
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
                                <td t-if="groupby != 'status'" align="right" class="align-middle">
                                    <t t-call="project.portal_my_tasks_state_widget_template">
                                        <t t-set="path" t-value="'tasks'"/>
                                    </t>
                                </td>
                                <td t-if="groupby != 'project'">
                                    <span title="Current project of the task" t-esc="task.project_id.name" />
                                </td>
                                <td t-if="groupby != 'stage'" class="text-end lh-1">
                                    <span t-attf-class="badge #{'text-bg-success' if task.stage_id.fold else 'text-bg-primary'} fw-normal o_text_overflow" t-attf-title="#{task.stage_id.name}" t-esc="task.stage_id.name"/>
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
                <div class="alert alert-warning" role="alert">
                    There are no tasks.
                </div>
            </t>
            <t t-call="project.portal_tasks_list"/>
        </t>
    </template>

    <template id="task_link_preview_front_end" inherit_id="portal.frontend_layout" primary="True">
        <xpath expr="//t[@t-if='not_uses_default_logo'][1]" position="before">
            <t t-if="preview_object.displayed_image_id">
                <meta property="og:image" t-attf-content="/web/image/{{ preview_object.displayed_image_id.id }}/300x200?access_token={{ preview_object.displayed_image_id.generate_access_token()[0] }}"/>
            </t>
        </xpath>
        <xpath expr="//t[@t-if='not_uses_default_logo'][2]" position="before">
            <t t-if="preview_object.displayed_image_id">
                <meta property="twitter:image" t-attf-content="/web/image/{{ preview_object.displayed_image_id.id }}/300x200?access_token={{ preview_object.displayed_image_id.generate_access_token()[0] }}"/>
            </t>
        </xpath>
    </template>

    <template id="task_link_preview_portal_layout" inherit_id="portal.portal_layout" primary="True">
        <xpath expr="//t[@t-call='portal.frontend_layout']" position="attributes">
            <attribute name="t-call">project.task_link_preview_front_end</attribute>
        </xpath>
    </template>

    <template id="portal_my_task" name="My Task" inherit_id="portal.portal_sidebar" primary="True">
        <xpath expr="//t[@t-call='portal.portal_layout']" position="attributes">
            <attribute name="t-call">project.task_link_preview_portal_layout</attribute>
        </xpath>
        <xpath expr="//div[hasclass('o_portal_sidebar')]" position="inside">
            <t t-set="title" t-value="task.name"/>
            <t t-set="o_portal_fullwidth_alert" groups="project.group_project_user">
                <t t-call="portal.portal_back_in_edit_mode">
                    <t t-set="backend_url" t-value="'/web#model=project.task&amp;id=%s&amp;action=%s&amp;view_type=form' % (task.id, task.env.ref('project.action_view_my_task').id)"/>
                </t>
            </t>

            <div class="row o_project_portal_sidebar">
                <t t-call="portal.portal_record_sidebar">
                    <t t-set="classes" t-value="'col-lg-3 col-xl-4 d-print-none'"/>

                    <t t-set="entries">
                        <div class="d-flex flex-wrap flex-column gap-4">
                            <div id="task-nav" class="d-flex align-items-center flex-grow-1 p-0" t-ignore="true" role="complementary">
                                <ul class="nav flex-column">
                                    <li class="nav-item" id="nav-header">
                                        <a class="nav-link p-0" href="#card_header">
                                            Task
                                        </a>
                                    </li>
                                    <li class="nav-item" id="nav-chat">
                                        <a class="nav-link p-0" href="#task_chat">
                                            History
                                        </a>
                                    </li>
                                </ul>
                            </div>
                            <div id="task-links" t-if="task_link_section" class="d-flex align-items-center flex-grow-1 ps-0" t-ignore="true" role="complementary">
                                <ul class="nav flex-column">
                                    <t t-foreach="task_link_section" t-as="task_link">
                                        <li class="nav-item">
                                            <a class="nav-link p-0" t-att-href="task_link['access_url']">
                                                <t t-out="task_link['title']"/>
                                            </a>
                                        </li>
                                    </t>
                                </ul>
                            </div>

                            <div t-if="task.user_ids or task.partner_id" class="d-flex flex-column gap-4">
                                <div class="col-12" t-if="task.user_ids">
                                    <h6 class="flex-basis-100"><small class="text-muted">Assignees</small></h6>
                                    <t t-foreach="task.user_ids" t-as="user">
                                        <div t-attf-class="o_portal_contact_details d-flex flex-column gap-2 {{ 'mb-3' if len(task.user_ids) > 1 else '' }}">
                                            <div class="d-flex justify-content-start align-items-center gap-2">
                                                <img class="o_avatar o_portal_contact_img rounded" t-att-src="image_data_uri(user.avatar_128)"/>
                                                <h6 class="mb-0" t-field="user.name"></h6>
                                            </div>
                                            <div t-out="user" t-options='{"widget": "contact", "fields": ["email", "phone"]}'/>
                                        </div>
                                    </t>
                                </div>
                                <div class="col-12 d-flex flex-column" t-if="task.partner_id">
                                    <h6><small class="text-muted">Customer</small></h6>
                                    <t t-if="task.partner_id">
                                        <div class="o_portal_contact_details d-flex flex-column gap-2">
                                            <div class="d-flex justify-content-start align-items-center gap-2">
                                                <img class="o_avatar o_portal_contact_img rounded" t-att-src="image_data_uri(task.partner_id.avatar_512)"/>
                                                <h6 class="mb-0" t-out="task.partner_id.name"></h6>
                                            </div>
                                            <div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["email", "phone"]}'/>
                                        </div>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </t>
                </t>
                <div id="task_content" class="o_portal_content col-12 col-lg-9 col-xl-8">
                    <div id="card">
                        <div id="card_header" data-anchor="true">
                            <div class="row justify-content-between align-items-end mb-3">
                                <div class="col-12 col-md-9">
                                    <div class="d-flex align-items-center gap-2">
                                        <t t-call="project.portal_my_tasks_priority_widget_template">
                                            <t t-set="classes" t-translation="off">fs-4</t>
                                        </t>
                                        <h3 t-field="task.name" class="text-truncate my-0"/>
                                        <small class="text-muted d-none d-md-inline align-self-end">(#<span t-field="task.id"/>)</small>
                                    </div>
                                </div>
                                <div class="col-auto">
                                    <small class="text-end">Stage:</small>
                                    <span t-field="task.stage_id.name" class=" badge rounded-pill text-bg-info" title="Current stage of this task"/>
                                </div>
                            </div>
                        </div>
                        <div id="card_body">
                            <div class="float-end">
                                <t t-call="project.portal_my_tasks_state_widget_template">
                                    <t t-set="path" t-value="'task'"/>
                                </t>
                            </div>
                            <div class="row mb-4 container">
                                <div class="col-12 col-md-6 flex-grow-1">
                                    <div t-if="project_accessible"><strong>Project:</strong> <a t-attf-href="/my/projects/#{task.project_id.id}" t-field="task.project_id"/></div>
                                    <div t-else=""><strong>Project:</strong> <a t-field="task.project_id"/></div>
                                    <div t-if="task.date_deadline"><strong>Deadline:</strong> <span t-field="task.date_deadline" t-options='{"widget": "datetime"}'/></div>
                                    <div t-if="task.milestone_id and task.allow_milestones"><strong>Milestone:</strong> <span t-field="task.milestone_id"/></div>
                                    <div name="portal_my_task_allocated_hours">
                                        <strong t-if="task.allocated_hours > 0">Allocated Time:</strong>
                                        <t t-call="project.portal_my_task_allocated_hours_template"/>
                                    </div>
                                </div>
                                <div class="col-12 col-md-6 d-empty-none" name="portal_my_task_second_column"></div>
                            </div>

                            <div class="row" t-if="task.description or task.attachment_ids">
                                <div t-if="not is_html_empty(task.description)" t-attf-class="col-12 mb-4 mb-md-0 {{'col-lg-6' if task.attachment_ids else 'col-lg-12'}}">
                                    <hr class="mb-1"/>
                                    <div class="d-flex my-2">
                                        <h5>Description</h5>
                                    </div>
                                    <div class="py-1 px-2 bg-100 small table-responsive" t-field="task.description"/>
                                </div>
                                <div t-if="task.attachment_ids" t-attf-class="col-12 o_project_portal_attachments {{'col-lg-6' if task.description else 'col-lg-12'}}">
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

                    <hr/>
                    <div id="task_chat" data-anchor="true">
                        <h3>Communication history</h3>
                        <t t-call="portal.message_thread">
                            <t t-set="token" t-value="task.access_token"/>
                        </t>
                    </div>
                </div>
            </div>
        </xpath>
    </template>

    <template id="portal_my_task_allocated_hours_template">
        <strong t-if="task.allocated_hours > 0" class="d-none">Allocated Time:</strong>
        <span t-out="task.allocated_hours" t-options='{"widget": "float_time"}'/>
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
            <tree editable="bottom" sample="1" delete="0">
                <field name="sequence" widget="handle"/>
                <field name="name" placeholder="e.g. To Do"/>
                <field name="mail_template_id" optional="hide" context="{'default_model': 'project.project'}"/>
                <field name="company_id" optional="hide" groups="base.group_multi_company"/>
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
                    <field name="name" placeholder="e.g. To Do"/>
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
            <form delete="0">
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <h1><field name="name" placeholder="e.g. To Do"/></h1>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="mail_template_id" context="{'default_model': 'project.project'}"/>
                            <field name="sequence" groups="base.group_no_one"/>
                        </group>
                        <group>
                            <field name="fold"/>
                            <field name="company_id" groups="base.group_multi_company"/>
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
                            <span class="text-muted" invisible="not mail_template_id">
                                <field name="mail_template_id"/>
                                <br/>
                            </span>
                            <span groups="base.group_multi_company" invisible="not company_id">
                                <field name="company_id"/>
                                <br/>
                            </span>
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
                <field name="company_id" groups="base.group_multi_company"/>
                <filter string="Archived" name="archived" domain="[('active', '=', False)]"/>
                <group>
                    <filter string="Company" name="company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                </group>
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

    <record id="unlink_project_stage_action" model="ir.actions.server">
        <field name="name">Delete</field>
        <field name="model_id" ref="project.model_project_project_stage"/>
        <field name="binding_model_id" ref="project.model_project_project_stage"/>
        <field name="binding_view_types">form,list</field>
        <field name="state">code</field>
        <field name="code">action = records.unlink_wizard(stage_view=True)</field>
    </record>
</odoo>

```

## File: views\project_project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="project_project_view_activity" model="ir.ui.view">
            <field name="name">project.project.view.activity</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <activity string="Project">
                    <templates>
                        <div t-name="activity-box" class="d-flex">
                            <field name="user_id" widget="many2one_avatar_user"/>
                            <field name="name" string="Project Name" class="flex-grow-1 o_text_block"/>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="action_send_mail_project_project" model="ir.actions.act_window">
            <field name="name">Send Email</field>
            <field name="res_model">mail.compose.message</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="context" eval="{
                'default_composition_mode': 'mass_mail',
            }"/>
            <field name="binding_model_id" ref="project.model_project_project"/>
            <field name="binding_view_types">list</field>
        </record>

        <record id="edit_project" model="ir.ui.view">
            <field name="name">project.project.form</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <form string="Project" class="o_form_project_project" js_class="project_form">
                    <field name="company_id" invisible="1"/>
                    <field name="analytic_account_id" invisible="1"/>
                    <header>
                        <button name="%(project.project_share_wizard_action)d" string="Share Read-only" type="action" class="oe_highlight" groups="project.group_project_manager"
                        invisible="privacy_visibility != 'portal'" context="{'default_access_mode': 'read', 'dialog_size': 'medium'}" data-hotkey="r"/>
                        <button name="%(project.project_share_wizard_action)d" string="Share Editable" type="action" class="oe_highlight" groups="project.group_project_manager"
                        invisible="privacy_visibility != 'portal'" context="{'default_access_mode': 'edit', 'dialog_size': 'medium'}" data-hotkey="e"/>
                        <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" groups="project.group_project_stages" domain="[('company_id', 'in', (company_id, False))]"/>
                    </header>
                <sheet string="Project">
                    <div class="oe_button_box" name="button_box" groups="base.group_user">
                        <button class="oe_stat_button" type="object" name="action_view_tasks" icon="fa-tasks">
                            <field string="Tasks" name="open_task_count" widget="statinfo"/>
                        </button>
                        <button class="oe_stat_button" name="project_update_all_action" type="object" groups="project.group_project_user">
                            <field name="last_update_color" invisible="1"/>
                            <div class="o_stat_info">
                                <field name="last_update_status" readonly="1" widget="status_with_color" status_label="Project Status"/>
                            </div>
                        </button>
                        <!-- To Do: remove me in master -->
                        <button class="oe_stat_button o_project_not_clickable" disabled="disabled" groups="!project.group_project_manager" invisible="1">
                            <div>
                                <field name="last_update_color" invisible="1"/>
                                <field name="last_update_status" readonly="1" widget="status_with_color" status_label="Project Status"/>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="%(project.project_collaborator_action)d" type="action" icon="fa-users" groups="project.group_project_manager" invisible="privacy_visibility != 'portal'">
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
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title">
                        <h1 class="d-flex flex-row">
                            <field name="is_favorite" nolabel="1" widget="boolean_favorite" class="me-2"/>
                            <field name="name" options="{'line_breaks': False}" widget="text" class="o_text_overflow" placeholder="e.g. Office Party"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="label_tasks" string="Name of the Tasks" placeholder="e.g. Tasks"/>
                            <field name="partner_id" widget="res_partner_many2one"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="user_id" string="Project Manager" widget="many2one_avatar_user" readonly="not active" domain="[('share', '=', False)]" options="{'no_quick_create': True}"/>
                            <field name="date_start" string="Planned Date" widget="daterange" options='{"end_date_field": "date", "always_range": "1"}' required="date_start or date" />
                            <field name="date" invisible="1" required="date_start"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="description" string="Description">
                            <field name="description" options="{'resizable': false}" placeholder="Project description..."/>
                        </page>
                        <page name="settings" string="Settings">
                            <group>
                                <group>
                                    <field name="analytic_account_id" domain="['|', ('company_id', '=?', company_id), ('company_id', '=', False)]" context="{'default_partner_id': partner_id, 'default_company_id': company_id} " groups="analytic.group_analytic_accounting"/>
                                    <field name="privacy_visibility" widget="radio"/>
                                    <span colspan="2" class="text-muted o_row ps-1" invisible="access_instruction_message == ''">
                                        <i class="fa fa-lightbulb-o"/>&amp;nbsp;<field class="d-inline" name="access_instruction_message" nolabel="1"/>
                                    </span>
                                    <span colspan="2" class="text-muted o_row ps-1" invisible="privacy_visibility_warning == ''">
                                        <i class="fa fa-warning"/>&amp;nbsp;<field class="d-inline" name="privacy_visibility_warning" nolabel="1"/>
                                    </span>
                                </group>
                                <group>
                                    <div name="alias_def" colspan="2" class="pb-2">
                                        <!-- Always display the whole alias in edit mode. It depends in read only -->
                                        <!-- Need to add alias_id in view for getting alias_domain_id by default -->
                                        <field name="alias_id" invisible="1"/>
                                        <label for="alias_name" class="fw-bold o_form_label" string="Create tasks by sending an email to"/>
                                        <field name="alias_email" class="oe_read_only d-inline" widget="email" readonly="1" invisible="not alias_name" />
                                        <span class="oe_edit_only o_row">
                                            <field name="alias_name"/>@
                                            <field name="alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                                                   options="{'no_create': True, 'no_open': True}"/>
                                        </span>
                                    </div>
                                    <!-- the alias contact must appear when the user start typing and it must disappear
                                        when the string is deleted. -->
                                    <field name="alias_contact" class="oe_inline" string="Accept Emails From"
                                           invisible="not alias_email"/>
                                </group>
                                <group name="extra_settings">
                                </group>
                            </group>
                            <group>
                                <group name="group_tasks_managment" string="Tasks Management" col="1" class="row mt16 o_settings_container" groups="project.group_project_task_dependencies,project.group_project_milestone">
                                    <div>
                                        <setting class="col-lg-12" id="task_dependencies_setting" help="Determine the order in which to perform tasks" groups="project.group_project_task_dependencies">
                                            <field name="allow_task_dependencies"/>
                                        </setting>
                                        <setting class="col-lg-12" id="project_milestone_setting" help="Track major progress points that must be reached to achieve success" groups="project.group_project_milestone">
                                            <field name="allow_milestones"/>
                                        </setting>
                                    </div>
                                </group>
                                <group name="group_time_managment" string="Time Management" invisible="1" col="1" class="row mt16 o_settings_container"/>
                                <group name="group_documents_analytics" string="Analytics" col="1" class="row mt16 o_settings_container" invisible="not allow_rating">
                                    <div>
                                        <field name="allow_rating" invisible="1"/>
                                        <setting class="col-lg-12" name="analytic_div" help="Get customer feedback and evaluate the performance of your employees" groups="project.group_project_rating">
                                            <field name="rating_active"/>
                                            <div class="mt16" invisible="not rating_active">
                                                <span class="text-muted o_row ps-1 pb-3">Send a rating request:</span>
                                                <field name="rating_status" widget="radio" class="o_row"/>
                                                <div  invisible="rating_status != 'periodic'"  required="rating_status == 'periodic'">
                                                    <label for="rating_status_period" string="Frequency"/>
                                                    <field class="mx-3 w-auto" name="rating_status_period"/>
                                                </div>
                                                <span colspan="2" class="text-muted o_row ps-1">
                                                    <i class="fa fa-lightbulb-o pe-2"/>
                                                    <span invisible="rating_status == 'periodic'">A rating request will be sent as soon as the task reaches a stage on which a Rating Email Template is defined.</span>
                                                    <span invisible="rating_status == 'stage'">Rating requests will be sent as long as the task remains in a stage on which a Rating Email Template is defined.</span>
                                                </span>
                                                <div class="content-group">
                                                    <div class="mt8">
                                                        <button name="%(project.open_task_type_form_domain)d" context="{'project_id':id}" icon="oi-arrow-right" type="action" string="Set a Rating Email Template on Stages" class="btn-link"/>
                                                    </div>
                                                </div>
                                            </div>
                                        </setting>
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
                    <field name="stage_id" groups="project.group_project_stages"/>
                    <field name="partner_id" string="Customer" filter_domain="[('partner_id', 'child_of', self)]"/>
                    <filter string="My Projects" name="own_projects" domain="[('user_id', '=', uid)]"/>
                    <filter string="My Favorites" name="my_projects" domain="[('favorite_user_ids', 'in', uid)]"/>
                    <filter string="Unassigned" name="unassigned_projects" domain="[('user_id', '=', False)]"/>
                    <separator/>
                    <filter string="Late Milestones" name="late_milestones" domain="[('is_milestone_exceeded', '=', True)]" groups="project.group_project_milestone"/>
                    <separator/>
                    <filter string="Start Date" name="start_date" date="date_start"/>
                    <filter string="End Date" name="end_date" date="date"/>
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
                        <filter string="Stage" name="groupby_stage" context="{'group_by': 'stage_id'}" groups="project.group_project_stages"/>
                        <filter string="Status" name="status" context="{'group_by': 'last_update_status'}"/>
                        <filter string="Tags" name="tags" context="{'group_by': 'tag_ids'}"/>
                        <filter string="Company" name="company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="view_project" model="ir.ui.view">
            <field name="name">project.project.tree</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <tree decoration-muted="active == False" string="Projects" multi_edit="1" sample="1" default_order="is_favorite desc, sequence, name, id" js_class="project_project_list">
                    <field name="sequence" column_invisible="True"/>
                    <field name="name" column_invisible="1"/>
                    <field name="message_needaction" column_invisible="True"/>
                    <field name="active" column_invisible="True"/>
                    <field name="is_favorite" string="Favorite" nolabel="1" widget="boolean_favorite" optional="hide"/>
                    <field name="display_name" string="Name" class="fw-bold"/>
                    <field name="partner_id" optional="show" string="Customer"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company" options="{'no_create': True, 'no_open': True}"/>
                    <field name="company_id" column_invisible="True"/>
                    <field name="date_start" string="Planned Date" widget="daterange" options="{'end_date_field': 'date', 'always_range': '1'}" optional="hide"/>
                    <field name="date" column_invisible="True" />
                    <field name="user_id" optional="show" string="Project Manager" widget="many2one_avatar_user" options="{'no_open':True, 'no_create': True, 'no_create_edit': True}"/>
                    <field name="last_update_color" column_invisible="True"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                    <field name="last_update_status" string="Status" nolabel="1" optional="show" widget="project_state_selection"/>
                    <field name="stage_id" options="{'no_open': True}" domain="[('company_id', 'in', (company_id, False))]" optional="show"/>
                    <button string="View Tasks" name="action_view_tasks" type="object"/>
                </tree>
            </field>
        </record>

        <record id="project_list_view_group_stage" model="ir.ui.view">
            <field name="name">project.project.tree.group.stage</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="view_project"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <tree position="attributes">
                    <attribute name="default_group_by">stage_id</attribute>
                </tree>
            </field>
        </record>

        <record id="view_project_config" model="ir.ui.view">
            <field name="name">project.project.tree.config</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="default_order">sequence, name, id</attribute>
                </xpath>
                <field name="sequence" position="attributes">
                    <attribute name="column_invisible">0</attribute>
                    <attribute name="widget">handle</attribute>
                </field>
            </field>
        </record>

        <record id="view_project_config_group_stage" model="ir.ui.view">
            <field name="name">project.project.tree.config.group.stage</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="view_project_config"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <tree position="attributes">
                    <attribute name="default_group_by">stage_id</attribute>
                </tree>
            </field>
        </record>

        <record id="quick_create_project_form" model="ir.ui.view">
            <field name="name">project.form.quick_create</field>
            <field name="model">project.project</field>
            <field name="priority">1000</field>
            <field name="arch" type="xml">
                <form class="o_form_project_project">
                    <group>
                        <field name="name" string="Project Title" placeholder="e.g. Office Party"/>
                    </group>
                </form>
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
                    <div class="oe_title mb-lg-3 mb-md-2">
                        <label for="name" string="Name"/>
                        <h1>
                            <field name="name" class="o_project_name" placeholder="e.g. Office Party"/>
                        </h1>
                    </div>
                    <field name="user_id" invisible="1"/>
                    <div class="row o_settings_container"/>
                    <div name="alias_def" class="mt-2" colspan="2">
                        <label for="alias_name" string="Create tasks by sending an email to"/>
                        <span>
                            <!-- Need to add alias_id in view for getting alias_domain_id by default -->
                            <field name="alias_id" invisible="1"/>
                            <field name="alias_name" placeholder="e.g. office-party"/>@
                            <field name="alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                                   options="{'no_create': True, 'no_open': True}"/>
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
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="x"/>
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
            <field name="context">{"default_allow_billable": 0}</field>
        </record>

        <record model="ir.ui.view" id="view_project_kanban">
            <field name="name">project.project.kanban</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <kanban
                    class="o_kanban_dashboard o_project_kanban o_emphasize_colors"
                    js_class="project_project_kanban"
                    on_create="project.open_create_project"
                    action="action_view_tasks" type="object"
		    quick_create_view="project.quick_create_project_form"
                    sample="1"
                    default_order="is_favorite desc, sequence, name, id"
                >
                    <field name="display_name"/>
                    <field name="partner_id"/>
                    <field name="color"/>
                    <field name="task_count"/>
                    <field name="closed_task_count"/>
                    <field name="open_task_count"/>
                    <field name="milestone_count_reached"/>
                    <field name="milestone_count"/>
                    <field name="allow_milestones"/>
                    <field name="label_tasks"/>
                    <field name="alias_email"/>
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
                    <progressbar field="last_update_status" colors='{"on_track": "success", "at_risk": "warning", "off_track": "danger", "on_hold": "info", "done": "purple"}'/>
                    <field name="sequence" widget="handle"/>
                    <templates>
                        <t t-name="kanban-menu" groups="base.group_user">
                            <div class="container">
                                <div class="row">
                                    <div class="col-6 o_kanban_card_manage_section o_kanban_manage_view">
                                        <h5 role="menuitem" class="o_kanban_card_manage_title">
                                            <span>View</span>
                                        </h5>
                                        <div role="menuitem">
                                            <a name="action_view_tasks" type="object">Tasks</a>
                                        </div>
                                        <div role="menuitem" groups="project.group_project_milestone" t-if="record.allow_milestones.raw_value">
                                            <a name="action_get_list_view" type="object">Milestones</a>
                                        </div>
                                    </div>
                                    <div class="col-6 o_kanban_card_manage_section o_kanban_manage_reporting">
                                        <h5 role="menuitem" class="o_kanban_card_manage_title" groups="project.group_project_user">
                                            <span>Reporting</span>
                                        </h5>
                                        <div role="menuitem" groups="project.group_project_user" class="o_kanban_task_analysis">
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
                                        <a t-if="record.privacy_visibility.raw_value == 'portal'" class="dropdown-item" role="menuitem" name="%(project.project_share_wizard_action)d" type="action" context="{'dialog_size': 'medium'}">Share</a>
                                        <a class="dropdown-item" role="menuitem" type="edit">Settings</a>
                                    </div>
                                    <div class="o_kanban_card_manage_section o_kanban_manage_view col-12 ps-0" groups="!project.group_project_manager">
                                        <div role="menuitem" class="w-100">
                                            <a class="dropdown-item mx-0" role="menuitem" type="open">View</a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{kanban_color(record.color.raw_value)} oe_kanban_global_click o_has_icon oe_kanban_content oe_kanban_card">
                                <div class="o_project_kanban_main d-flex align-items-baseline gap-1">
                                    <field name="is_favorite" widget="boolean_favorite" nolabel="1" force_save="1"/>
                                    <div class="o_kanban_card_content mw-100">
                                        <div class="o_kanban_primary_left">
                                            <div class="o_primary me-5">
                                                <span class="o_text_overflow" t-att-title="record.display_name.value"><t t-esc="record.display_name.value"/></span>
                                                <span class="o_text_overflow text-muted" t-if="record.partner_id.value">
                                                    <span class="fa fa-user me-2" aria-label="Partner" title="Partner"></span><t t-esc="record.partner_id.value"/>
                                                </span>
                                                <div t-if="record.date.raw_value or record.date_start.raw_value" class="text-muted o_row">
                                                    <span class="fa fa-clock-o me-2" title="Dates"></span><field name="date_start"/>
                                                    <i t-if="record.date.raw_value and record.date_start.raw_value" class="fa fa-long-arrow-right mx-2 oe_read_only" aria-label="Arrow icon" title="Arrow"/>
                                                    <field name="date"/>
                                                </div>
                                                <div t-if="record.alias_email.value" class="text-muted text-truncate" t-att-title="record.alias_email.value">
                                                    <span class="fa fa-envelope-o me-2" aria-label="Domain Alias" title="Domain Alias"></span><t t-esc="record.alias_email.value"/>
                                                </div>
                                                <div t-if="record.rating_active.raw_value and record.rating_count.raw_value &gt; 0" class="text-muted" groups="project.group_project_rating">
                                                    <b class="me-1">
                                                        <span style="font-weight:bold;" class="fa mt4 fa-smile-o text-success" t-if="record.rating_avg.raw_value &gt;= 3.66" title="Average Rating: Satisfied" role="img" aria-label="Happy face"/>
                                                        <span style="font-weight:bold;" class="fa mt4 fa-meh-o text-warning" t-elif="record.rating_avg.raw_value &gt;= 2.33" title="Average Rating: Okay" role="img" aria-label="Neutral face"/>
                                                        <span style="font-weight:bold;" class="fa mt4 fa-frown-o text-danger" t-else="" title="Average Rating: Dissatisfied" role="img" aria-label="Sad face"/>
                                                    </b>
                                                    <t t-if="record.rating_avg.raw_value % 1 == 0">
                                                        <field name="rating_avg" nolabel="1" widget="float" digits="[1, 0]"/>
                                                    </t>
                                                    <t t-else="">
                                                        <field name="rating_avg" nolabel="1" widget="float" digits="[1, 1]"/>
                                                    </t> / 5
                                                </div>
                                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div class="o_kanban_record_bottom mt-3">
                                    <div class="oe_kanban_bottom_left">
                                        <div class="o_project_kanban_boxes d-flex align-items-baseline">
                                            <a class="o_project_kanban_box" name="action_view_tasks" type="object">
                                                <div>
                                                    <span class="o_value"><t t-esc="record.open_task_count.value"/></span>
                                                    <span class="o_label ms-1"><t t-esc="record.label_tasks.value"/></span>
                                                </div>
                                            </a>
                                            <a groups='project.group_project_milestone' t-if="record.allow_milestones and record.allow_milestones.raw_value and record.milestone_count.raw_value &gt; 0"
                                                class="o_kanban_inline_block btn-link text-dark small"
                                                role="button"
                                                name="action_get_list_view"
                                                type="object"
                                                t-attf-title="#{record.milestone_count_reached.value} Milestones reached out of #{record.milestone_count.value}"
                                            >
                                                <span class="fa fa-flag me-1"/>
                                                <t t-out="record.milestone_count_reached.value"/>/<t t-out="record.milestone_count.value"/>
                                            </a>
                                        </div>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                                        <field t-if="record.last_update_status.value &amp;&amp; widget.editable" name="last_update_status" widget="project_state_selection"/>
                                        <span t-if="record.last_update_status.value &amp;&amp; !widget.editable" t-att-class="'o_status_bubble mx-0 o_color_bubble_' + record.last_update_color.value" t-att-title="record.last_update_status.value"></span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="project_kanban_view_group_stage" model="ir.ui.view">
            <field name="name">project.project.kanban.group.stage</field>
            <field name="model">project.project</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="view_project_kanban"/>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="default_group_by">stage_id</attribute>
                </xpath>
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

        <record id="view_project_config_kanban_group_stage" model="ir.ui.view">
            <field name="name">project.kanban.inherit.config.project.group.stage</field>
            <field name="model">project.project</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="view_project_config_kanban"/>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="default_group_by">stage_id</attribute>
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
                    quick_create="0"
                    color="color"
                    js_class="project_project_calendar">
                    <field name="partner_id" invisible="not partner_id"/>
                    <field name="user_id" widget="many2one_avatar_user" invisible="not user_id"/>
                    <field name="is_favorite" widget="boolean_favorite" nolabel="1" string="Favorite"/>
                    <field name="stage_id" groups="project.group_project_stages" invisible="not stage_id"/>
                    <field name="last_update_color" invisible="1"/>
                    <field name="last_update_status" string="Status" widget="status_with_color" invisible="last_update_status == 'to_define'"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" invisible="not tag_ids"/>
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
            <field name="context">{}</field>
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

        <record id="open_view_project_all_group_stage_kanban_view" model="ir.actions.act_window.view">
            <field name="sequence" eval="10"/>
            <field name="view_mode">kanban</field>
            <field name="act_window_id" ref="open_view_project_all_group_stage"/>
            <field name="view_id" ref="project_kanban_view_group_stage"/>
        </record>
        <record id="open_view_project_all_group_stage_tree_view" model="ir.actions.act_window.view">
            <field name="sequence" eval="20"/>
            <field name="view_mode">tree</field>
            <field name="act_window_id" ref="open_view_project_all_group_stage"/>
            <field name="view_id" ref="project_list_view_group_stage"/>
        </record>

        <!-- Please update both act_window when modifying one (open_view_project_all_config or open_view_project_all_config_group_stage) as one or the other is used in the menu menu_project_config -->
        <record id="open_view_project_all_config" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="domain">[]</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('view_project_config')}),
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
            <field name="view_id" ref="view_project_config_group_stage"/>
        </record>
        <record id="open_view_project_all_config_group_stage_kanban_action_view" model="ir.actions.act_window.view">
            <field name="sequence" eval="20"/>
            <field name="view_mode">kanban</field>
            <field name="act_window_id" ref="project.open_view_project_all_config_group_stage"/>
            <field name="view_id" ref="view_project_config_kanban_group_stage"/>
        </record>

        <record id="project_view_kanban_inherit_project" model="ir.ui.view">
            <field name="name">project.kanban.inherit.project</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project_kanban"/>
            <field name="priority">200</field>
            <field name="arch" type="xml">
                <xpath expr="/kanban" position="inside">
                    <field name="id"/>
                </xpath>
                <xpath expr="//div[hasclass('o_kanban_task_analysis')]" position="before">
                    <div role="menuitem" groups="project.group_project_user">
                        <a name="project_update_all_action" type="object" t-attf-context="{'active_id': #{record.id.raw_value} }">Project Updates</a>
                    </div>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: views\project_sharing_project_task_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="project_sharing_portal" name="My Project">
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
                <t t-call-assets="project.webclient"/>
            </t>
            <t t-set="head" t-value="head_project_sharing + (head or '')"/>
            <t t-set="body_classname" t-value="'o_web_client o_project_sharing'"/>
        </t>
    </template>
</odoo>

```

## File: views\project_sharing_project_task_views.xml

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
                groups_draggable="0"
            >
                <field name="color"/>
                <field name="priority"/>
                <field name="stage_id" options='{"group_by_tooltip": {"description": "Description"}}'/>
                <field name="portal_user_names"/>
                <field name="partner_id"/>
                <field name="sequence"/>
                <field name="state"/>
                <field name="displayed_image_id"/>
                <field name="active"/>
                <field name="closed_subtask_count"/>
                <field name="subtask_count"/>
                <field name="allow_milestones" />
                <field name="has_late_and_unreached_milestone"/>
                <progressbar field="state" colors='{"1_done": "success", "03_approved": "success", "02_changes_requested": "warning", "1_canceled": "danger", "04_waiting_normal": "200", "01_in_progress": "200"}'/>
                <templates>
                <t t-name="kanban-menu" t-if="!selection_mode">
                    <div invisible="1" role="separator" class="dropdown-divider"></div>
                    <ul invisible="1" class="oe_kanban_colorpicker" data-field="color"/>
                </t>
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
                                    <span t-if="record.allow_milestones.raw_value and record.milestone_id.raw_value" t-attf-class="{{record.has_late_and_unreached_milestone.raw_value and !record.state.raw_value.startsWith('1_') ? 'text-danger' : ''}}">
                                        <br/>
                                        <field name="milestone_id" />
                                    </span>
                                    <br />
                                    <t t-if="record.partner_id.value">
                                        <field name="partner_id"/>
                                    </t>
                                </div>
                            </div>
                            <div class="o_kanban_record_body">
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" context="{'project_id': project_id}"/>
                                <div t-if="record.date_deadline.raw_value" name="date_deadline" invisible="state in ['1_done', '1_canceled']">
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
                                        <t t-if="user_count > 1"><t t-out="user_count"/> assignees</t>
                                        <t t-else="" t-out="record.portal_user_names.raw_value"/>
                                    </span>
                                    <field name="state" widget="project_task_state_selection" options="{'is_toggle_mode': false}"/>
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
        <field name="inherit_id" ref="project_task_view_tree_main_base"/>
        <field name="mode">primary</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <tree position="attributes">
                <attribute name="delete">0</attribute>
                <attribute name="import">0</attribute>
            </tree>
            <xpath expr="//field[@widget='res_partner_many2one']" position="attributes">
                <attribute name="widget">many2one</attribute>
            </xpath>
            <field name="user_ids" position="replace">
                <field name="portal_user_names" string="Assignees"/>
            </field>
            <xpath expr="//field[@name='milestone_id']" position="attributes">
                <attribute name="column_invisible">not context.get('allow_milestones', True)</attribute>
            </xpath>
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
                    <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="not project_id and not stage_id" />
                </header>
                <div groups="base.group_user" role="status" class="alert alert-info alert-dismissible rounded-0 fade show d-print-none css_editable_mode_hidden">
                    <div class="text-center">This is a preview of how the project will look when it's shared with customers and they have editing access.
                        <a name="action_redirect_to_project_task_form" type="object"><i class="oi oi-arrow-right me-1"/>Back to edit mode</a>
                    </div>
                </div>
                <sheet string="Task">
                    <div class="oe_button_box" name="button_box">
                        <field name="display_parent_task_button" invisible="1"/>
                        <button name="action_project_sharing_view_parent_task" type="object" class="oe_stat_button" icon="fa-tasks" invisible="not display_parent_task_button">
                            <div class="o_stat_info">
                                <span class="o_stat_text">Parent Task</span>
                            </div>
                        </button>
                        <button name="action_project_sharing_open_subtasks" type="object" class="oe_stat_button" icon="fa-tasks"
                            invisible="not id or subtask_count == 0" context="{'default_user_ids': [(6, 0, [uid])], 'default_project_id': project_id }">
                            <field name="subtask_count" widget="statinfo" string="Sub-tasks"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title pe-0">
                        <h1 class="d-flex flex-row justify-content-between">
                            <field name="priority" widget="priority" class="me-3"/>
                            <field name="name" class="o_task_name text-truncate" placeholder="Task Title..."/>
                            <div class="d-flex justify-content-end o_state_container" invisible="not active">
                                <field name="state" widget="project_task_state_selection" class="o_task_state_widget"/>
                            </div>
                            <div class="d-flex justify-content-start o_state_container w-100 w-md-50 w-lg-25" invisible="active">
                                <field name="state" widget="project_task_state_selection" class="o_task_state_widget"/>
                            </div>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="project_id" invisible="1"/>
                            <field name="allow_milestones" invisible="1"/>
                            <field name="milestone_id"
                                placeholder="e.g. Product Launch"
                                invisible="not allow_milestones"
                                readonly="1"
                                options="{'no_open': True}"/>
                            <field name="user_ids" invisible="1" />
                            <field name="portal_user_names"
                                string="Assignees"
                                class="o_task_user_field"/>
                            <field name="tag_ids" context="{'project_id': project_id}" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True, 'no_edit_color': True}"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="parent_id" invisible="1" />
                            <field name="company_id" invisible="1" />
                            <field name="state" invisible="1" />
                            <field name="partner_id" options="{'no_open': True, 'no_create': True, 'no_edit': True}" invisible="not project_id"/>
                            <field name="date_deadline" invisible="state in ['1_done', '1_canceled']" decoration-danger="date_deadline &lt; current_date"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" type="html" options="{'collaborative': true, 'allowCommandImage': false, 'allowCommandVideo': false, 'allowCommandFile': false}"/>
                        </page>
                        <page name="sub_tasks_page" string="Sub-tasks">
                            <field name="child_ids" context="{
                                'default_project_id': project_id,
                                'default_display_in_project': False,
                                'default_parent_id': id,
                                'default_partner_id': partner_id,
                                'form_view_ref' : 'project.project_sharing_project_task_view_form',
                            }">
                                <tree editable="bottom">
                                    <field name="project_id" column_invisible="True"/>
                                    <field name="state" column_invisible="True"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="priority" widget="priority" optional="show" nolabel="1" width="40px"/>
                                    <field name="state" widget="project_task_state_selection" nolabel="1" width="40px"/>
                                    <field name="name"/>
                                    <field name="allow_milestones" column_invisible="True"/>
                                    <field name="milestone_id"
                                        optional="hide"
                                        options="{'no_open': True}"
                                        readonly="1"
                                        column_invisible="not parent.allow_milestones"
                                        invisible="not allow_milestones"/>
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="partner_id" options="{'no_open': True, 'no_create': True, 'no_edit': True}" optional="hide"/>
                                    <field name="user_ids" column_invisible="True" />
                                    <field name="portal_user_names" string="Assignees" optional="show"/>
                                    <field name="date_deadline" invisible="state in ['1_done', '1_canceled']" decoration-danger="date_deadline &lt; current_date" optional="show"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="stage_id" domain="[('user_id', '=', False), ('project_ids', 'in', [project_id])]"/>
                                    <button name="action_open_task" type="object" title="View Task" string="View Task" class="btn btn-link float-end"
                                            context="{'form_view_ref': 'project.project_sharing_project_task_view_form', 'search_view_ref': 'project.project_sharing_project_task_view_search'}"
                                            invisible="project_id != context.get('active_id')"/>
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
        <field name="inherit_id" ref="project.view_task_search_form_project_base"/>
        <field name="mode">primary</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <search position="inside"/>
            <xpath expr="//search/filter[@name='inactive']" position='attributes'>
                <attribute name='invisible'>1</attribute>
            </xpath>
        </field>
    </record>

    <record id="project_sharing_project_task_action" model="ir.actions.act_window">
        <field name="name">Project Sharing</field>
        <field name="res_model">project.task</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="search_view_id" ref="project.project_sharing_project_task_view_search"/>
        <field name="domain">[('project_id', '=', active_id), ('display_in_project', '=', True)]</field>
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

## File: views\project_tags_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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

## File: views\project_task_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active" />
                        <group>
                            <group>
                                <field name="name" placeholder="e.g. To Do"/>
                                <field name="user_id" invisible="True"/>
                                <field name="mail_template_id" context="{'default_model': 'project.task'}" invisible="user_id"/>
                                <field name="rating_template_id"
                                       placeholder="Task: Rating Request"
                                       groups="project.group_project_rating"
                                       context="{'default_model': 'project.task'}"
                                       invisible="user_id"/>
                                <div class="alert alert-warning" role="alert" colspan='2'
                                     invisible="not rating_template_id or not disabled_rating_warning or user_id"
                                     groups="project.group_project_rating">
                                    <i class="fa fa-warning" title="Customer disabled on projects"/><b> Customer Ratings</b> are disabled on the following project(s) : <br/>
                                    <field name="disabled_rating_warning" class="mb-0" />
                                </div>
                                <field name="auto_validation_state" invisible="not rating_template_id" groups="project.group_project_rating"/>
                                <field name="sequence" groups="base.group_no_one"/>
                            </group>
                            <group>
                                <field name="fold"/>
                                <field name="project_ids" widget="many2many_tags" options="{'color_field': 'color'}"
                                       invisible="user_id"
                                       required="not user_id"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <!-- TODO: remove in master -->
        <record id="personal_task_type_edit" model="ir.ui.view">
            <field name="name">project.task.type.form</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <form string="Task Stage" delete="0">
                    <field name="active" invisible="1" />
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active" />
                        <group>
                            <group>
                                <field name="name" placeholder="e.g. To Do"/>
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

        <record id="task_type_tree" model="ir.ui.view">
            <field name="name">project.task.type.tree</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <tree string="Task Stage" delete="0" sample="1" multi_edit="1" editable="bottom" open_form_view="True">
                    <field name="sequence" widget="handle" optional="show"/>
                    <field name="name" placeholder="e.g. To Do"/>
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
                <xpath expr="//tree" position="attributes">
                    <attribute name="default_group_by">project_ids</attribute>
                </xpath>
                <xpath expr="//field[@name='name']" position="after">
                    <field name="mail_template_id" optional="hide"/>
                    <field name="rating_template_id" optional="hide" groups="project.group_project_rating"/>
                    <field name="project_ids" required="1" optional="show" widget="many2many_tags" options="{'color_field': 'color'}"/>
                </xpath>
            </field>
        </record>

        <record id="view_project_task_type_kanban" model="ir.ui.view">
            <field name="name">project.task.type.kanban</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1" default_group_by="project_ids">
                    <field name="name"/>
                    <field name="fold"/>
                    <field name="description"/>
                    <field name="sequence" widget="handle"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-12">
                                        <strong class="o_text_overflow"><t t-esc="record.name.value"/></strong>
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
            <field name="context">{'default_project_id': False}</field>
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

        <record id="unlink_task_type_action" model="ir.actions.server">
            <field name="name">Delete</field>
            <field name="model_id" ref="project.model_project_task_type"/>
            <field name="binding_model_id" ref="project.model_project_task_type"/>
            <field name="binding_view_types">form,list</field>
            <field name="state">code</field>
            <field name="code">action = records.unlink_wizard(stage_view=True)</field>
        </record>
</odoo>

```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Base search view, contains the fields and filters common to project, project sharing and fsm -->
        <record id="view_task_search_form_base" model="ir.ui.view">
            <field name="name">project.task.search.form</field>
            <field name="model">project.task</field>
            <field name="priority">999</field>
            <field name="arch" type="xml">
                <search string="Tasks">
                    <field name="name" string="Tasks" filter_domain="['|', ('name', 'ilike', self), ('id', 'ilike', self)]"/>
                    <field name="tag_ids"/>
                    <field name="user_ids" filter_domain="[('user_ids.name', 'ilike', self), ('user_ids.active', 'in', [True, False])]"/>
                    <field name="stage_id"/>
                    <field name="milestone_id" groups="project.group_project_milestone"/>
                    <field name="partner_id" operator="child_of"/>
                    <filter string="Followed" name="followed_by_me" domain="[('message_is_follower', '=', True)]"/>
                    <filter string="Unassigned" name="unassigned" domain="[('user_ids', '=', False)]"/>
                    <separator invisible="context.get('default_project_id')"/>
                    <filter string="Favorite Projects" name="favorite_projects" domain="[('project_id.is_favorite', '=', True)]" invisible="context.get('default_project_id')"/>
                    <separator/>
                    <filter string="Starred Tasks" name="starred_tasks" domain="[('priority', '=', '1')]"/>
                    <separator groups="project.group_project_task_dependencies"/>
                    <filter string="Blocked" name="blocked" domain="[('state', '=', '04_waiting_normal')]" groups="project.group_project_task_dependencies"/>
                    <filter string="Blocking" name="blocking" domain="[('state', 'in', ['01_in_progress', '02_changes_requested', '03_approved', '04_waiting_normal']), ('dependent_ids', '!=', False)]" groups="project.group_project_task_dependencies"/>
                    <separator/>
                    <filter string="Last Stage Update" name="date_last_stage_update" date="date_last_stage_update"/>
                    <separator/>
                    <filter string="Open Tasks" name="open_tasks" domain="[('state', 'in', ['01_in_progress', '02_changes_requested', '03_approved', '04_waiting_normal'])]"/>
                    <filter string="Closed Tasks" name="closed_tasks" domain="[('state', 'in', ['1_done','1_canceled'])]"/>
                    <filter string="Closed On" name="closed_on" domain="[('state', 'in', ['1_done','1_canceled'])]" date="date_last_stage_update" invisible="1"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Assignees" name="user" context="{'group_by': 'user_ids'}"/>
                        <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}"/>
                        <filter string="Milestone" name="milestone" context="{'group_by': 'milestone_id'}" groups="project.group_project_milestone"/>
                        <filter string="Tags" name="tags" context="{'group_by': 'tag_ids'}"/>
                        <filter string="Customer" name="customer" context="{'group_by': 'partner_id'}"/>
                        <filter string="Company" name="company_id" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Creation Date" name="create_date" context="{'group_by': 'create_date'}"/>
                        <filter string="Assignement Date" name="date_assign" context="{'group_by': 'date_assign'}"/>
                        <filter string="Last Stage Update" name="last_stage_update" context="{'group_by': 'date_last_stage_update'}"/>
                    </group>
                </search>
            </field>
        </record>

        <!-- Contains the fields and filters common to projet and fsm -->
        <record id="view_task_search_form_project_fsm_base" model="ir.ui.view">
            <field name="name">project.task.search.form.project.base</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_search_form_base"/>
            <field name="mode">primary</field>
            <field name="priority">999</field>
            <field name="arch" type="xml">
                <field name="stage_id" position="after">
                    <field name="project_id" string="Project" groups="base.group_user"/>
                </field>
                <field name="partner_id" position="after">
                    <field name="company_id" groups="base.group_multi_company"/>
                    <filter string="My Tasks" name="my_tasks" domain="[('user_ids', 'in', uid)]" groups="base.group_user"/>
                </field>
                <filter name="stage" position="after">
                    <filter string="Project" name="project" context="{'group_by': 'project_id'}" groups="base.group_user"/>
                </filter>
            </field>
        </record>

        <!-- Base search view for project only -->
        <record id="view_task_search_form_project_base" model="ir.ui.view">
            <field name="name">project.task.search.form.project.base</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_search_form_project_fsm_base"/>
            <field name="mode">primary</field>
            <field name="priority">999</field>
            <field name="arch" type="xml">
                <filter name="date_last_stage_update" position="after">
                    <filter string="Deadline" name="date_deadline" date="date_deadline"/>
                </filter>
                <filter name="last_stage_update" position="after">
                    <filter string="Deadline" name="deadline" context="{'group_by': 'date_deadline'}"/>
                    <separator/>
                    <filter string="Properties" name="group_by_properties" context="{'group_by': 'task_properties'}" groups="base.group_user"/>
                </filter>
            </field>
        </record>

        <record id="view_task_search_form" model="ir.ui.view">
            <field name="name">project.task.search.form</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_search_form_project_base"></field>
            <field name="mode">primary</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <filter name="my_tasks" position="before">
                    <field string="Properties" name="task_properties"/>
                </filter>
                <filter name="inactive" position="before">
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]" groups="mail.group_mail_notification_type_inbox"/>
                    <separator/>
                </filter>
                <filter name="inactive" position="after">
                    <separator invisible="1"/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                </filter>
                <filter name="unassigned" position="after">
                    <filter string="Private Tasks" name="private_tasks" domain="[('project_id', '=', False)]" invisible="context.get('default_project_id')"/>
                </filter>
            </field>
        </record>

        <record id="view_project_task_pivot" model="ir.ui.view">
            <field name="name">project.task.pivot</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <pivot string="Tasks" sample="1" js_class="project_pivot">
                    <field name="project_id" type="row"/>
                    <field name="color" invisible="1"/>
                    <field name="sequence" invisible="1"/>
                    <field name="allocated_hours" widget="float_time"/>
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
                <xpath expr="//field[@name='project_id']" position="after">
                    <field name="stage_id" type="col"/>
                </xpath>
                <xpath expr="//field[@name='project_id']" position="attributes">
                    <attribute name="invisible">True</attribute>
                </xpath>
            </field>
        </record>

        <record id="act_project_project_2_project_task_all" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form,calendar,pivot,graph,activity</field>
            <field name="domain">[('project_id', '=', active_id), ('display_in_project', '=', True)]</field>
            <field name="context">{
                'active_model': 'project.project',
                'default_project_id': active_id,
                'show_project_update': True,
                'search_default_open_tasks': 1,
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

        <record id="action_send_mail_project_task" model="ir.actions.act_window">
            <field name="name">Send Email</field>
            <field name="res_model">mail.compose.message</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="context" eval="{
                'default_composition_mode': 'mass_mail',
            }"/>
            <field name="binding_model_id" ref="project.model_project_task"/>
            <field name="binding_view_types">list</field>
        </record>

        <record id="view_task_form2" model="ir.ui.view">
            <field name="name">project.task.form</field>
            <field name="model">project.task</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <form string="Task" class="o_form_project_tasks" js_class="project_task_form">
                    <field name="recurrence_id" invisible="1" />
                    <field name="allow_task_dependencies" invisible="1" />
                    <field name="rating_last_value" invisible="1"/>
                    <field name="rating_count" invisible="1"/>
                    <field name="allow_milestones" invisible="1" />
                    <field name="parent_id" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <field name="project_id" invisible="1"/>
                    <header>
                        <field name="stage_id" widget="statusbar_duration" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="not project_id and not stage_id"/>
                        <field name="state" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="1"/>
                        <field name="personal_stage_type_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="project_id" domain="[('user_id', '=', uid)]" string="Personal Stage"/>
                    </header>
                    <sheet string="Task">
                    <div class="oe_button_box" name="button_box">
                        <!-- Dummy tag for organizing buttons, using position='replace' when inheriting -->
                        <span id="button_products" invisible="1"/>
                        <span id="button_worksheet" invisible="1"/>
                        <!-- Dummy tag used to organize buttons, englobing the 3 buttons modifies the width of the button -->
                        <span id="start_rating_buttons" invisible="1"/>
                        <field name="rating_avg" invisible="1"/>
                        <field name="rating_active" invisible="1"/>
                        <button name="action_open_ratings" type="object" invisible="rating_count == 0 or not rating_active" class="oe_stat_button" groups="project.group_project_rating">
                            <i class="fa fa-fw o_button_icon fa-smile-o text-success" invisible="rating_avg &lt; 3.66" title="Satisfied"/>
                            <i class="fa fa-fw o_button_icon fa-meh-o text-warning" invisible="rating_avg &lt; 2.33 or rating_avg &gt;= 3.66" title="Okay"/>
                            <i class="fa fa-fw o_button_icon fa-frown-o text-danger" invisible="rating_avg &gt;= 2.33" title="Dissatisfied"/>
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="rating_avg_text" nolabel="1"/></span>
                                <span class="o_stat_text">Last Rating</span>
                            </div>
                        </button>
                        <!-- Dummy tag used to organize buttons -->
                        <span id="end_rating_buttons" invisible="1"/>
                        <button name="action_open_parent_task" type="object" class="oe_stat_button" icon="fa-tasks" invisible="not parent_id">
                            <div class="o_stat_info">
                                <span class="o_stat_text">Parent Task</span>
                            </div>
                        </button>
                        <button name="action_recurring_tasks" type="object" invisible="not active or not recurrence_id" class="oe_stat_button" icon="fa-repeat" groups="project.group_project_recurring_tasks">
                            <field name="recurring_count" widget="statinfo" string="Recurring Tasks"/>
                        </button>
                        <button name="%(project_task_action_sub_task)d" type="action" class="oe_stat_button" icon="fa-tasks"
                            invisible="not id or subtask_count == 0"
                            context="{'default_user_ids': user_ids, 'default_milestone_id': milestone_id, 'subtask_action': True}">
                            <div class="o_field_widget o_stat_info">
                                <div class="d-flex align-items-baseline gap-1">
                                    <span class="o_stat_value order-1">
                                        <field name="subtask_count" widget="statinfo" nolabel="1"/>
                                    </span>
                                    <span class="o_stat_text order-2">Sub-tasks</span>
                                </div>
                                <div class="d-flex align-items-baseline gap-1">
                                    <span class="o_stat_value">
                                        <field name="closed_subtask_count" widget="statinfo" nolabel="1"/>
                                    </span>
                                    <span class="o_stat_text order-2">Closed</span>
                                </div>
                            </div>
                        </button>
                        <button name="action_dependent_tasks" type="object" invisible="dependent_tasks_count == 0" class="oe_stat_button" icon="fa-tasks" groups="project.group_project_task_dependencies">
                            <field name="dependent_tasks_count" widget="statinfo" string="Blocking Tasks" />
                        </button>
                        <!-- Dummy tag used to organize buttons -->
                        <span id="end_button_box" invisible="1"/>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title pe-0">
                        <h1 class="d-flex justify-content-between align-items-center">
                            <div class="d-flex w-100">
                                <field name="priority" widget="priority_switch" class="me-3"/>
                                <field name="name" options="{'line_breaks': False}" widget="text" class="o_task_name text-truncate w-md-75 w-100 pe-2" placeholder="Task Title..."/>
                            </div>
                            <div class="d-flex justify-content-end o_state_container" invisible="not active">
                                <field name="state" widget="project_task_state_selection" class="o_task_state_widget" />
                            </div>
                            <div class="d-flex justify-content-start o_state_container w-100 w-md-50 w-lg-25" invisible="active">
                                <field name="state" widget="project_task_state_selection" class="o_task_state_widget" />
                            </div>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="display_in_project" invisible="1" force_save="1"/>
                            <field name="project_id"
                                   domain="[('active', '=', True), '|', ('company_id', '=', False), ('company_id', '=?', company_id)]"
                                   widget="project"
                            />
                            <field name="display_in_project" invisible="True" force_save="1"/>
                            <field name="milestone_id"
                                placeholder="e.g. Product Launch"
                                context="{'default_project_id': project_id}"
                                invisible="not project_id or not allow_milestones"/>
                            <field name="user_ids"
                                class="o_task_user_field"
                                options="{'no_open': True, 'no_quick_create': True}"
                                widget="many2many_avatar_user"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}" context="{'project_id': project_id}"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="partner_id" nolabel="0" widget="res_partner_many2one" class="o_task_customer_field" invisible="not project_id"/>
                            <label for="date_deadline"/>
                            <div id="date_deadline_and_recurring_task" class="d-inline-flex w-100">
                                <field name="date_deadline" nolabel="1" decoration-danger="date_deadline and date_deadline &lt; current_date and state not in ['1_done', '1_canceled']"/>
                                <field name="recurring_task" nolabel="1" class="ms-0" style="width: fit-content;"
                                       widget="boolean_icon" options="{'icon': 'fa-repeat'}"
                                       invisible="not active or parent_id"
                                       groups="project.group_project_recurring_tasks"/>
                            </div>
                            <label for="repeat_interval" groups="project.group_project_recurring_tasks" invisible="not recurring_task" />
                            <div invisible="not recurring_task" class="d-flex" groups="project.group_project_recurring_tasks">
                                <field name="repeat_interval" required="recurring_task"
                                       class="me-2" style="max-width: 2rem !important;" />
                                <field name="repeat_unit" required="recurring_task"
                                       class="me-2" style="max-width: 4rem !important;" />
                                <field name="repeat_type" required="recurring_task"
                                       class="me-2" style="max-width: 15rem !important;" />
                                <field name="repeat_until" invisible="repeat_type != 'until'" required="repeat_type == 'until'"
                                       class="me-2" />
                            </div>
                        </group>
                    </group>
                        <field name="task_properties" columns="2"/>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" type="html" options="{'collaborative': true, 'resizable': false}" placeholder="Add details about this task..."/>
                        </page>
                        <page name="sub_tasks_page" string="Sub-tasks" invisible="not project_id">
                            <field name="child_ids"
                                   context="{'default_project_id': project_id, 'default_display_in_project': False, 'default_user_ids': user_ids, 'default_parent_id': id,
                                    'default_partner_id': partner_id, 'default_milestone_id': allow_milestones and milestone_id}"
                                   widget="subtasks_one2many">
                                <tree editable="bottom" decoration-muted="state in ['1_done','1_canceled']" open_form_view="True">
                                    <field name="allow_milestones" column_invisible="True"/>
                                    <field name="display_in_project" column_invisible="True" force_save="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="id" optional="hide"/>
                                    <field name="parent_id" column_invisible="True"/>
                                    <field name="priority" widget="priority" nolabel="1" options="{'autosave': False}" width="40px"/>
                                    <field name="state" widget="project_task_state_selection" nolabel="1" options="{'autosave':  False}" width="40px"/>
                                    <field name="name" widget="name_with_subtask_count"/>
                                    <field name="subtask_count" column_invisible="True"/>
                                    <field name="closed_subtask_count" column_invisible="True"/>
                                    <field name="project_id" string="Project" optional="hide" options="{'no_open': 1}" widget="project"/>
                                    <field name="milestone_id"
                                        optional="hide"
                                        context="{'default_project_id': project_id}"
                                        column_invisible="not parent.allow_milestones"
                                        invisible="not allow_milestones"/>
                                    <field name="partner_id" optional="hide" widget="res_partner_many2one" invisible="not project_id"/>
                                    <field name="user_ids" widget="many2many_avatar_user" optional="show"/>
                                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="date_deadline" invisible="state in ['1_done', '1_canceled']" optional="hide" decoration-danger="date_deadline and date_deadline &lt; current_date"/>
                                    <field name="activity_ids" string="Next Activity" widget="list_activity" optional="hide"/>
                                     <field name="my_activity_date_deadline" string="My Deadline" widget="remaining_days" options="{'allow_order': '1'}" optional="hide"/>
                                    <field name="rating_last_text" string="Rating" decoration-danger="rating_last_text == 'ko'"
                                        decoration-warning="rating_last_text == 'ok'" decoration-success="rating_last_text == 'top'"
                                        class="fw-bold" widget="badge" optional="hide" invisible="rating_last_text == 'none'" column_invisible="True" groups="project.group_project_rating"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="stage_id" optional="hide" context="{'default_project_id': project_id}"/>
                                </tree>
                            </field>
                        </page>
                        <page name="task_dependencies" string="Blocked By" invisible="not allow_task_dependencies" groups="project.group_project_task_dependencies">
                            <field name="depend_on_ids" nolabel="1"
                                   context="{'default_project_id': project_id, 'search_view_ref' : 'project.view_task_search_form', 'search_default_project_id': project_id, 'tree_view_ref': 'project.open_view_all_tasks_list_view', 'search_default_open_tasks': 1}">
                                <tree editable="bottom" decoration-muted="state in ['1_done','1_canceled']" open_form_view="True">
                                    <field name="allow_milestones" column_invisible="True"/>
                                    <field name="parent_id" column_invisible="True" />
                                    <field name="subtask_count" column_invisible="True"/>
                                    <field name="closed_subtask_count" column_invisible="True"/>
                                    <field name="id" optional="hide"/>
                                    <field name="priority" widget="priority" nolabel="1" options="{'autosave': False}" width="40px"/>
                                    <field name="state" widget="project_task_state_selection" nolabel="1" options="{'autosave': False}" width="40px"/>
                                    <field name="name" widget="name_with_subtask_count"/>
                                    <field name="project_id" optional="hide" options="{'no_open': 1}" />
                                    <field name="milestone_id"
                                        optional="hide"
                                        context="{'default_project_id': project_id}"
                                        column_invisible="not parent.allow_milestones"
                                        invisible="not allow_milestones"/>
                                    <field name="partner_id" optional="hide" widget="res_partner_many2one" invisible="not project_id"/>
                                    <field name="parent_id" optional="hide" groups="base.group_no_one"/>
                                    <field name="user_ids" widget="many2many_avatar_user" optional="show"/>
                                    <field name="company_id" optional="hide" groups="base.group_multi_company" />
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="date_deadline" invisible="state in ['1_done', '1_canceled']" optional="hide" decoration-danger="date_deadline and date_deadline &lt; current_date"/>
                                    <field name="activity_ids" string="Next Activity" widget="list_activity" optional="hide"/>
                                    <field name="my_activity_date_deadline" string="My Deadline" widget="remaining_days" options="{'allow_order': '1'}" optional="hide"/>
                                    <field name="rating_last_text" string="Rating" decoration-danger="rating_last_text == 'ko'"
                                        decoration-warning="rating_last_text == 'ok'" decoration-success="rating_last_text == 'top'"
                                        class="fw-bold" widget="badge" optional="hide" invisible="rating_last_text == 'none'" column_invisible="True" groups="project.group_project_rating"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="stage_id" optional="hide"/>
                                </tree>
                            </field>
                        </page>
                        <page name="extra_info" string="Extra Info" groups="base.group_no_one">
                            <group>
                                <group>
                                    <field name="parent_id" groups="base.group_no_one" context="{'search_view_ref' : 'project.view_task_search_form','search_default_project_id': project_id}"/>
                                    <field name="analytic_account_id" groups="analytic.group_analytic_accounting" context="{'default_partner_id': partner_id, 'default_company_id': company_id}"/>
                                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                    <field name="sequence" groups="base.group_no_one"/>
                                    <field name="email_cc" groups="base.group_no_one"/>
                                    <field name="displayed_image_id" groups="base.group_no_one" options="{'no_create': True}"/>
                                </group>
                                <group>
                                    <field name="date_assign" groups="base.group_no_one"/>
                                    <field name="date_last_stage_update" groups="base.group_no_one"/>
                                </group>
                                <group string="Working Time to Assign" invisible="working_hours_open == 0.0">
                                    <field name="working_hours_open" widget="float_time" string="Hours"/>
                                    <field name="working_days_open" string="Days"/>
                                </group>
                                <group string="Working Time to Close" invisible="working_hours_close == 0.0">
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
            <field name="context">{'dialog_size': 'medium'}</field>
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
                        <field name="display_name" string= "Task Title" placeholder="e.g. Send Invitations" required="1"/>
                        <field name="project_id"
                               widget="project"
                               invisible="context.get('default_project_id', False)"
                               placeholder="Private"
                               class="o_project_task_project_field"
                               domain="[('type_ids', 'in', context['default_stage_id'])] if context.get('default_stage_id') else []"
                        />
                        <field name="user_ids" options="{'no_open': True, 'no_quick_create': True}"
                            widget="many2many_avatar_user"/>
                        <field name="company_id" invisible="1"/>
                        <field name="parent_id" invisible="1" groups="base.group_no_one"/>
                        <field name="description" invisible="1"/>
                    </group>
                </form>
            </field>
        </record>

        <record id="project_task_convert_to_subtask_view_form" model="ir.ui.view">
            <field name="name">project.task.convert.to.subtask.form</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <form>
                    <div class="text-muted o_row mb-3">
                        To transform a task into a sub-task, select a parent task. Alternatively, leave the parent task field blank to convert a sub-task into a standalone task.
                    </div>
                    <group>
                        <field name="project_id" invisible="1" />
                        <field name="company_id" invisible="1" />
                        <field name="parent_id" domain="[('id', '!=', id), '!', ('id', 'child_of', id)]" context="{'search_default_project_id': project_id, 'search_default_open_tasks': 1}" />
                    </group>
                    <footer>
                        <button string="Convert Task" class="btn-primary" special="save" data-hotkey="q"/>
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z" />
                    </footer>
                </form>
            </field>
        </record>

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
                    default_order="priority desc, sequence, state, date_deadline asc, id desc"
                >
                    <field name="color"/>
                    <field name="priority"/>
                    <field name="stage_id" options='{"group_by_tooltip": {"description": "Description"}}'/>
                    <field name="user_ids"/>
                    <field name="partner_id"/>
                    <field name="sequence"/>
                    <field name="displayed_image_id"/>
                    <field name="active"/>
                    <field name="activity_ids"/>
                    <field name="activity_state"/>
                    <field name="rating_count"/>
                    <field name="rating_avg"/>
                    <field name="rating_active"/>
                    <field name="has_late_and_unreached_milestone" />
                    <field name="allow_milestones" />
                    <field name="state" />
                    <field name="company_id"/>
                    <field name="recurrence_id" />
                    <field name="subtask_count"/>
                    <field name="closed_subtask_count"/>
                    <field name="subtask_count"/>
                    <progressbar field="state" colors='{"1_done": "success-done", "1_canceled": "danger", "03_approved": "success", "02_changes_requested": "warning", "04_waiting_normal": "info", "01_in_progress": "200" }'/>
                    <templates>
                    <t t-name="kanban-menu" t-if="!selection_mode" groups="base.group_user">
                        <a t-if="widget.editable" role="menuitem" type="set_cover" class="dropdown-item" data-field="displayed_image_id">Set Cover Image</a>
                        <a name="%(portal.portal_share_action)d" role="menuitem" type="action" class="dropdown-item" context="{'dialog_size': 'medium'}">Share</a>
                        <div role="separator" class="dropdown-divider"></div>
                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click">
                            <div class="oe_kanban_content">
                                <div class="o_kanban_record_top" t-att-class="['1_done', '1_canceled'].includes(record.state.raw_value) ? 'opacity-50' : ''">
                                    <div class="o_kanban_record_headings text-muted">
                                        <strong class="o_kanban_record_title">
                                            <s t-if="!record.active.raw_value">
                                                <field name="name"/>
                                            </s>
                                            <t t-else="">
                                                <field name="name"/>
                                            </t>
                                        </strong>
                                        <span t-if="record.parent_id.raw_value" invisible="context.get('default_parent_id', False)" style="display: block; margin-top: 4px;">
                                            <field name="parent_id"/>
                                        </span>
                                        <span invisible="context.get('default_project_id', False)" style="display: block; margin-top: 4px;">
                                            <field name="project_id" options="{'no_open': True}"/>
                                        </span>
                                        <span t-if="record.partner_id.value" style="display: block; margin-top: 4px;">
                                            <field name="partner_id"/>
                                        </span>
                                        <span t-if="record.allow_milestones.raw_value and record.milestone_id.raw_value" t-att-class="record.has_late_and_unreached_milestone.raw_value and !record.state.raw_value.startsWith('1_') ? 'text-danger' : ''" style="display: block; margin-top: 4px;">
                                            <field name="milestone_id" options="{'no_open': True}" />
                                        </span>
                                    </div>
                                </div>
                                <div class="o_kanban_record_body" t-att-class="['1_done', '1_canceled'].includes(record.state.raw_value) ? 'opacity-50' : 'text-muted'">
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                    <div t-if="record.date_deadline.raw_value" name="date_deadline" invisible="state in ['1_done', '1_canceled']">
                                        <field name="date_deadline" widget="remaining_days"/>
                                    </div>
                                    <field name="task_properties" widget="properties"/>
                                    <div t-if="record.displayed_image_id.value">
                                        <field name="displayed_image_id" widget="attachment_image"/>
                                    </div>
                                </div>
                                <div class="o_kanban_record_bottom" t-if="!selection_mode">
                                    <div class="oe_kanban_bottom_left" t-att-class="['1_done', '1_canceled'].includes(record.state.raw_value) ? 'opacity-50' : ''">
                                        <field name="priority" widget="priority" style="margin-right: 5px;"/>
                                        <field name="activity_ids" widget="kanban_activity" style="padding-top: 1.5px; margin-right: 2px"/>
                                        <b t-if="record.rating_active.raw_value and record.rating_count.raw_value &gt; 0" groups="project.group_project_rating">
                                            <span class="fa fa-fw fa-smile-o text-success rating_face" t-if="record.rating_avg.raw_value &gt;= 3.66" title="Average Rating: Satisfied" role="img" aria-label="Happy face"/>
                                            <span class="fa fa-fw fa-meh-o text-warning rating_face" t-elif="record.rating_avg.raw_value &gt;= 2.33" title="Average Rating: Okay" role="img" aria-label="Neutral face"/>
                                            <span class="fa fa-fw fa-frown-o text-danger rating_face" t-else="" title="Average Rating: Dissatisfied" role="img" aria-label="Sad face"/>
                                        </b>
                                        <a t-if="!record.project_id.raw_value" class="text-muted" style="font-size: 17px; padding-top: 1.5px; margin-left: 1.5px">
                                            <i title="Private Task" class="fa fa-lock"/>
                                        </a>
                                        <t t-if="record.project_id.raw_value and record.subtask_count.raw_value">
                                            <a t-if="record.project_id.raw_value and record.subtask_count.raw_value and record.subtask_count.value &gt; record.closed_subtask_count.value"
                                               t-attf-title="{{ record.closed_subtask_count.value }} sub-tasks closed out of {{ record.subtask_count.value }}"
                                               class="subtask_list_button btn-link text-dark"
                                               role="button">
                                                <span class="fa fa-check-square-o me-1"/>
                                                <t t-out="record.closed_subtask_count.value + '/' + record.subtask_count.value"/>
                                            </a>
                                            <div t-else="" t-attf-title="{{ record.closed_subtask_count.value }} sub-tasks closed out of {{ record.subtask_count.value }}" class="text-muted">
                                                <span class="fa fa-check-square-o me-1"/>
                                                <t t-out="record.closed_subtask_count.value + '/' + record.subtask_count.value"/>
                                            </div>
                                        </t>
                                    </div>
                                    <div class="oe_kanban_bottom_right" t-if="!selection_mode">
                                        <div  t-att-class="['1_done', '1_canceled'].includes(record.state.raw_value) ? 'opacity-50' : ''">
                                            <field name="user_ids" widget="many2many_avatar_user"/>
                                        </div>
                                        <field name="state" widget="project_task_state_selection" options="{'is_toggle_mode': false}"/>
                                    </div>
                                </div>
                                <t t-if="record.project_id.raw_value and record.subtask_count.raw_value and record.subtask_count.raw_value &gt; record.closed_subtask_count.raw_value">
                                    <div class="kanban_bottom_subtasks_section"/>
                                </t>
                            </div>
                            <div class="clearfix"></div>
                        </div>
                    </t>
                    </templates>
                </kanban>
            </field>
         </record>

        <!-- The main base of the project.task tree view -->
        <record id="project_task_view_tree_main_base" model="ir.ui.view">
            <field name="name">project.task.view.tree.main.base</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <tree string="Tasks" sample="1" default_order="priority desc, sequence, state, date_deadline asc, id desc">
                    <field name="sequence" readonly="1" column_invisible="True"/>
                    <field name="allow_milestones" column_invisible="True"/>
                    <field name="subtask_count" column_invisible="True"/>
                    <field name="closed_subtask_count" column_invisible="True"/>
                    <field name="id" optional="hide"/>
                    <field name="priority" widget="priority" nolabel="1"/>
                    <field name="state" widget="project_task_state_selection" nolabel="1" options="{'is_toggle_mode': false}"/>
                    <field name="name" string="Title" widget="name_with_subtask_count"/>
                    <field name="project_id" widget="project" optional="show" options="{'no_open': 1}" readonly="1" column_invisible="context.get('default_project_id')"/>
                    <field name="milestone_id" invisible="not allow_milestones" context="{'default_project_id': project_id}" groups="project.group_project_milestone" optional="hide"/>
                    <field name="partner_id" optional="hide" widget="res_partner_many2one" invisible="not project_id" options="{'no_open': True}"/>
                    <field name="user_ids" optional="show" widget="many2many_avatar_user"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show" column_invisible="context.get('default_project_id')"/>
                    <field name="company_id" column_invisible="True"/>
                    <field name="date_deadline" optional="hide" widget="remaining_days" invisible="state in ['1_done', '1_canceled']"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show" context="{'project_id': project_id}"/>
                    <field name="date_last_stage_update" optional="hide"/>
                    <field name="stage_id" column_invisible="context.get('set_visible', False)" optional="show"/>
                </tree>
            </field>
        </record>

        <!-- common base of the tree view in project app -->
        <record id="project_task_view_tree_base" model="ir.ui.view">
            <field name="name">project.task.view.tree.base</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project_task_view_tree_main_base"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <tree position="attributes">
                    <attribute name="multi_edit">1</attribute>
                </tree>
                <field name="date_deadline" position="after">
                    <field name="activity_ids" string="Next Activity" widget="list_activity" optional="show"/>
                    <field name="my_activity_date_deadline" string="My Deadline" widget="remaining_days" options="{'allow_order': '1'}" optional="hide"/>
                    <field name="rating_active" column_invisible="True"/>
                    <field name="rating_last_text" string="Rating" decoration-danger="rating_last_text == 'ko'"
                        decoration-warning="rating_last_text == 'ok'" decoration-success="rating_last_text == 'top'"
                        invisible="not rating_active or rating_last_text == 'none'"
                        class="fw-bold" widget="badge" optional="hide" groups="project.group_project_rating"/>
                </field>
                <tree position="inside">
                    <field name="task_properties"/>
                    <field name="recurrence_id" column_invisible="True" />
                </tree>
                <xpath expr="//field[@name='partner_id']" position="after">
                    <field name="parent_id" optional="hide" context="{'search_view_ref': 'project.view_task_search_form', 'search_default_project_id': project_id}" groups="base.group_no_one"/>
                </xpath>
                <xpath expr="//field[@name='stage_id']" position="after">
                    <field name="personal_stage_type_id" string="Personal Stage" optional="hide"/>
                </xpath>
            </field>
        </record>

        <!-- main tree view -->
        <record id="view_task_tree2" model="ir.ui.view">
            <field name="name">project.task.tree</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project_task_view_tree_base"/>
            <field name="mode">primary</field>
            <field name="priority">2</field>
            <field name="arch" type="xml">
                <tree position="attributes">
                    <attribute name="js_class">project_task_list</attribute>
                    <attribute name="default_group_by">stage_id</attribute>
                </tree>
            </field>
        </record>

        <record id="view_task_calendar" model="ir.ui.view">
            <field name="name">project.task.calendar</field>
            <field name="model">project.task</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <calendar date_start="date_deadline" string="Tasks" mode="month"
                          color="stage_id" event_limit="5" hide_time="true"
                          event_open_popup="true" quick_create="0" show_unusual_days="True"
                          js_class="project_task_calendar"
                          scales="month,year">
                    <field name="allow_milestones" invisible="1" />
                    <field name="project_id" widget="project" invisible="context.get('default_project_id', False)"/>
                    <field name="display_in_project" invisible="1"/>
                    <field name="subtask_count" invisible="1"/>
                    <field name="milestone_id" invisible="not allow_milestones or not milestone_id"/>
                    <field name="user_ids" widget="many2many_avatar_user" invisible="not user_ids"/>
                    <field name="partner_id" invisible="not partner_id"/>
                    <field name="priority" widget="priority"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" invisible="not tag_ids"/>
                    <field name="stage_id" invisible="not project_id or not stage_id"/>
                    <field name="state" widget="project_task_state_selection" readonly="1"/>
                    <field name="personal_stage_id" string="Personal Stage" options="{'no_open': True}" invisible="project_id or not personal_stage_id"/>
                    <field name="task_properties"/>
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
                    <attribute name="color">project_id</attribute>
                </xpath>
                <xpath expr="//field[@name='project_id']" position="attributes">
                    <attribute name="filters">1</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_project_task_graph" model="ir.ui.view">
            <field name="name">project.task.graph</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <graph string="Tasks" sample="1" js_class="project_task_graph">
                    <field name="project_id" invisible="context.get('default_project_id', False)"/>
                    <field name="stage_id"/>
                    <field name="working_hours_open" widget="float_time"/>
                    <field name="working_hours_close" widget="float_time"/>
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
                        <div t-name="activity-box" class="justify-content-between gap-2">
                            <div class="flex-shrink-1">
                                <field name="name" display="full" class="o_text_block"/>
                                <div t-att-title="record.project_id.value" invisible="context.get('default_project_id', False)">
                                    <field t-if="record.project_id.value" name="project_id" muted="1" class="o_text_block"/>
                                    <div t-else="" class="fst-italic text-muted"><i class="fa fa-lock"/> Private</div>
                                </div>
                            </div>
                            <div class="d-flex justify-content-end gap-2 flex-grow-1">
                                <field name="state" widget="project_task_state_selection" class="align-self-center"
                                       options="{'is_toggle_mode': false}"/>
                                <field name="user_ids" widget="many2many_avatar_user" class="o_many2many_avatar_user_no_wrap"/>
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
            <field name="domain">[('project_id', '!=', False), ('display_in_project', '=', True)]</field>
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
                    <attribute name="default_group_by">personal_stage_type_id</attribute>
                    <attribute name="examples"/>
                </xpath>
            </field>
        </record>

        <record id="view_task_kanban_inherit_all_task" model="ir.ui.view">
            <field name="name">project.task.kanban.inherit.all.task</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="default_group_by">project_id</attribute>
                </xpath>
            </field>
        </record>

        <record id="open_view_my_tasks_list_view" model="ir.ui.view">
            <field name="name">open.view.my.tasks.list.view</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_tree2"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <tree position="attributes">
                    <attribute name="default_group_by">personal_stage_type_id</attribute>
                </tree>
            </field>
        </record>

        <record id="open_view_all_tasks_list_view" model="ir.ui.view">
            <field name="name">open.view.all.tasks.list.view</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_tree2"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <tree position="attributes">
                    <attribute name="default_group_by"/>
                </tree>
                <field name="project_id" position="attributes">
                    <attribute name="column_invisible">0</attribute>
                </field>
            </field>
        </record>

        <record id="view_task_kanban_inherit_view_default_project" model="ir.ui.view" >
            <field name="name">project.task.kanban.sale.order</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <kanban position="attributes">
                    <attribute name="quick_create_view">project.quick_create_task_form_inherit_view_default_project</attribute>
                </kanban>
            </field>
        </record>

        <record id="quick_create_task_form_inherit_view_default_project" model="ir.ui.view">
            <field name="name">project.task.form.quick.create.sale.order.inherited</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="quick_create_task_form"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <field name="project_id" position="attributes">
                    <attribute name="invisible">0</attribute>
                </field>
            </field>
        </record>

        <record id="action_view_my_task" model="ir.actions.act_window">
            <field name="name">My Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,tree,form,calendar,pivot,graph,activity</field>
            <field name="context">{
                'search_default_open_tasks': 1,
                'all_task': 0,
                'default_user_ids': [(4, uid)],
            }</field>
            <field name="search_view_id" ref="view_task_search_form"/>
            <field name="domain">[('user_ids', 'in', uid)]</field>
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

        <record id="open_view_my_task_list_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="10"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="view_task_kanban_inherit_my_task"/>
            <field name="act_window_id" ref="action_view_my_task"/>
        </record>

        <record id="open_view_my_task_list_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="20"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="open_view_my_tasks_list_view"/>
            <field name="act_window_id" ref="action_view_my_task"/>
        </record>

        <record id="open_view_my_task_list_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="40"/>
            <field name="view_mode">calendar</field>
            <field name="act_window_id" ref="action_view_my_task"/>
            <field name="view_id" ref="view_task_all_calendar"/>
        </record>

        <record id="action_view_all_task" model="ir.actions.act_window">
            <field name="name">All Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,kanban,form,calendar,pivot,graph,activity</field>
            <field name="domain">[('display_in_project', '=', True)]</field>
            <field name="context">{'search_default_open_tasks': 1, 'default_user_ids': [(4, uid)]}</field>
            <field name="search_view_id" ref="view_task_search_form"/>
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

        <record id="open_view_all_task_list_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="10"/>
            <field name="view_mode">tree</field>
            <field name="act_window_id" ref="action_view_all_task"/>
            <field name="view_id" ref="open_view_all_tasks_list_view"/>
        </record>

        <record id="open_view_all_task_list_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="20"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="view_task_kanban_inherit_all_task"/>
            <field name="act_window_id" ref="action_view_all_task"/>
        </record>

        <record id="open_view_all_task_list_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="40"/>
            <field name="view_mode">calendar</field>
            <field name="act_window_id" ref="action_view_all_task"/>
            <field name="view_id" ref="view_task_all_calendar"/>
        </record>

        <record id="action_server_view_my_task" model="ir.actions.server">
            <field name="name">menu view My Tasks</field>
            <field name="model_id" ref="project.model_project_task"/>
            <field name="state">code</field>
            <field name="code">
                model._ensure_personal_stages(); action = env["ir.actions.actions"]._for_xml_id("project.action_view_my_task")
            </field>
        </record>

        <record id="action_server_convert_to_subtask" model="ir.actions.server">
            <field name="name">Convert to Task/Sub-Task</field>
            <field name="model_id" ref="project.model_project_task"/>
            <field name="binding_model_id" ref="project.model_project_task"/>
            <field name="binding_view_types">form</field>
            <field name="state">code</field>
            <field name="code">
                action = record.action_convert_to_subtask()
            </field>
        </record>

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
            <field name="domain">[('state', 'in', ['01_in_progress', '02_changes_requested', '03_approved', '04_waiting_normal']), ('date_deadline','&lt;',time.strftime('%Y-%m-%d')), ('project_id', '!=', False), ('display_in_project', '=', True)]</field>
            <field name="filter" eval="True"/>
            <field name="search_view_id" ref="view_task_search_form"/>
        </record>

        <!-- Opening task when double clicking on project -->
        <record id="dblc_proj" model="ir.actions.act_window">
            <field name="res_model">project.task</field>
            <field name="name">Project's tasks</field>
            <field name="view_mode">tree,form,calendar,graph,kanban</field>
            <field name="domain">[('project_id', '=', active_id), ('display_in_project', '=', True)]</field>
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

        <!-- User Form -->
        <record id="act_res_users_2_project_task_opened" model="ir.actions.act_window">
            <field name="name">Assigned Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">tree,form,calendar,graph</field>
            <field name="context">{'default_user_ids': [(6, 0, [active_id])]}</field>
            <field name="domain">[('project_id', '!=', False), ('display_in_project', '=', True), ('user_ids', 'in', [active_id])]</field>
            <field name="binding_model_id" ref="base.model_res_users"/>
            <field name="binding_view_types">form</field>
        </record>
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
<li t-att-class="milestone['is_reached'] and 'o_checked'" t-attf-id="checkId-{{milestone_index}}">
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
            <form string="Project Update" js_class="form_description_expander">
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
                            <field name="status" widget="status_with_color"/>
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
            <kanban sample="1" js_class="project_update_kanban">
                <field name="color"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + record.color.raw_value : ''}} oe_kanban_global_click o_pupdate_kanban_card">
                            <!-- Project Update Kanban View is always ungrouped - see js_class -->
                            <div class="o_kanban_detail_ungrouped row">
                                <div class="col-sm-4 col-6 o_pupdate_name">
                                    <b><field name="name_cropped"/></b>
                                    <div class="d-flex gap-1">
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                        <t t-esc="record.user_id.value"/>
                                    </div>
                                </div>
                                <div class="col-sm-2 text-sm-start col-6 align-end">
                                    <field name="color" invisible="1"/>
                                    <b><field name="status" readonly="1" widget="status_with_color"/></b>
                                </div>
                                <div class="col-sm-2 col-6 pb-0">
                                    <b><field name="progress_percentage" widget="percentage"/></b>
                                    <div>Progress</div>
                                </div>
                                <div class="col-sm-2 col-6 pb-0">
                                    <b id="tasks_stats">
                                        <field name="closed_task_count"/> / <field name="task_count"/> Tasks<span invisible="not task_count"> (<field name="closed_task_percentage"/>%)</span>
                                    </b>
                                </div>
                                <div class="col-sm-2 col-6 pb-0 pt-2">
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
                <field name="progress" string="Progress" widget="progressbar" optional="show"/>
                <field name="color" column_invisible="True"/>
                <field name="status" widget="status_with_color"/>
            </tree>
        </field>
    </record>

    <record id="project_update_all_action" model="ir.actions.act_window">
        <field name="name">Project Updates</field>
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
                <xpath expr="//form" position="inside">
                    <app data-string="Project" string="Project" name="project" groups="project.group_project_manager">
                        <block title="Tasks Management" id="tasks_management">
                            <setting id="recurring_tasks_setting" help="Auto-generate tasks for regular activities">
                                <field name="group_project_recurring_tasks"/>
                            </setting>
                            <setting id="task_dependencies_setting" help="Determine the order in which to perform tasks">
                                <field name="group_project_task_dependencies"/>
                            </setting>
                            <setting id="project_stages" help="Track the progress of your projects">
                                <field name="group_project_stages"/>
                                <div class="content-group" invisible="not group_project_stages">
                                    <div class="mt8">
                                        <button name="%(project.project_project_stage_configure)d" icon="oi-arrow-right" type="action" string="Configure Stages" class="btn-link"/>
                                    </div>
                                </div>
                            </setting>
                            <setting id="project_milestone" help="Track major progress points that must be reached to achieve success">
                                <field name="group_project_milestone"/>
                            </setting>
                        </block>
                        <block title="Time Management" name="project_time">
                            <setting id="log_time_tasks_setting" help="Track time spent on projects and tasks">
                                <field name="module_hr_timesheet"/>
                            </setting>
                        </block>
                        <block title="Analytics" name="analytic">
                            <setting id="track_customer_satisfaction_setting" help="Track customer satisfaction on tasks">
                                <field name="group_project_rating"/>
                                <div class="content-group" invisible="not group_project_rating">
                                    <div class="mt16">
                                        <button name="%(project.open_task_type_form)d" context="{'project_id':id}" icon="oi-arrow-right" type="action" string="Set a Rating Email Template on Stages" class="btn-link"/>
                                    </div>
                                </div>
                            </setting>
                            <setting id="default_plan_setting" groups="analytic.group_analytic_accounting" help="Track the profitability of your project with analytic accounting. Select the analytic plan for new projects:" title="Track the profitability of your projects. Any project, its tasks and timesheets are linked to an analytic account and any analytic account belongs to a plan.">
                                <field name="analytic_plan_id"/>
                            </setting>
                        </block>
                    </app>
                </xpath>
            </field>
        </record>

        <record id="project_config_settings_action" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'project', 'bin_size': False}</field>
        </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_task_partner_info_form" model="ir.ui.view">
            <field name="name">res.partner.task.buttons</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="priority" eval="7"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" type="action" name="%(project_task_action_from_partner)d"
                        groups="project.group_project_user"
                        context="{'search_default_partner_id': id, 'default_partner_id': id}" invisible="task_count == 0"
                        icon="fa-tasks">
                        <field  string="Tasks" name="task_count" widget="statinfo"/>
                    </button>
                </div>
            </field>
       </record>
</odoo>

```

## File: wizard\project_project_stage_delete.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval
from odoo import api, fields, models

class ProjectStageDelete(models.TransientModel):
    _name = 'project.project.stage.delete.wizard'
    _description = 'Project Stage Delete Wizard'

    stage_ids = fields.Many2many('project.project.stage', string='Stages To Delete', ondelete='cascade', context={'active_test': False})
    projects_count = fields.Integer('Number of Projects', compute='_compute_projects_count')
    stages_active = fields.Boolean(compute='_compute_stages_active')

    def _compute_projects_count(self):
        for wizard in self:
            wizard.projects_count = self.with_context(active_test=False).env['project.project'].search_count([('stage_id', 'in', wizard.stage_ids.ids)])

    @api.depends('stage_ids')
    def _compute_stages_active(self):
        for wizard in self:
            wizard.stages_active = all(wizard.stage_ids.mapped('active'))

    def action_archive(self):
        projects = self.with_context(active_test=False).env['project.project'].search([('stage_id', 'in', self.stage_ids.ids)])
        projects.write({'active': False})
        self.stage_ids.write({'active': False})
        return self._get_action()

    def action_unarchive_project(self):
        inactive_projects = self.env['project.project'].with_context(active_test=False).search(
            [('active', '=', False), ('stage_id', 'in', self.stage_ids.ids)])
        inactive_projects.action_unarchive()

    def action_unlink(self):
        self.stage_ids.unlink()
        return self._get_action()

    def _get_action(self):
        action = self.env["ir.actions.actions"]._for_xml_id("project.project_project_stage_configure")\
              if self.env.context.get('stage_view')\
            else self.env["ir.actions.actions"]._for_xml_id("project.open_view_project_all_group_stage")

        context = action.get('context', '{}')
        context = context.replace('uid', str(self.env.uid))
        context = dict(literal_eval(context), active_test=True)
        action['context'] = context
        action['target'] = 'main'
        return action

```

## File: wizard\project_project_stage_delete_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_project_project_stage_delete_wizard" model="ir.ui.view">
        <field name="name">project.project.stage.delete.wizard.form</field>
        <field name="model">project.project.stage.delete.wizard</field>
        <field name="arch" type="xml">
            <form string="Delete Project Stage">
                <field name="projects_count" invisible="1" />
                <field name="stages_active" invisible="1" />
                <div invisible="projects_count &gt; 0">
                    <p>Are you sure you want to delete these stages?</p>
                </div>
                <div invisible="not stages_active or projects_count == 0">
                    <p>You cannot delete stages containing projects. You can either archive them or first delete all of their projects.</p>
                </div>
                <div invisible="stages_active or projects_count == 0">
                    <p>You cannot delete stages containing projects. You should first delete all of their projects.</p>
                </div>
                <footer>
                    <button string="Archive Stages" type="object" name="action_archive" class="btn btn-primary" invisible="not stages_active or projects_count == 0" data-hotkey="q"/>
                    <button string="Delete" type="object" name="action_unlink" class="btn btn-primary" invisible="projects_count &gt; 0" data-hotkey="w"/>
                    <button string="Discard" special="cancel" data-hotkey="x" class="btn btn-primary" invisible="not (stages_active or projects_count)" />
                    <button string="Discard" special="cancel" data-hotkey="x" invisible="stages_active or projects_count" />
                </footer>
            </form>
        </field>
    </record>

    <record id="view_project_project_stage_unarchive_wizard" model="ir.ui.view">
        <field name="name">project.project.stage.delete.wizard.form</field>
        <field name="model">project.project.stage.delete.wizard</field>
        <field name="arch" type="xml">
            <form string="Delete Stage">
                <div>
                    <p>Would you like to unarchive all of the projects contained in these stages as well?</p>
                </div>
                <footer>
                    <button string="Confirm" type="object" name="action_unarchive_project" class="btn btn-primary"/>
                    <button string="Discard" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\project_share_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ProjectShareWizard(models.TransientModel):
    _name = 'project.share.wizard'
    _inherit = 'portal.share'
    _description = 'Project Sharing'

    @api.model
    def default_get(self, fields):
        # The project share action could be called in `project.collaborator`
        # and so we have to check the active_model and active_id to use
        # the right project.
        active_model = self._context.get('active_model', '')
        active_id = self._context.get('active_id', False)
        if active_model == 'project.collaborator':
            active_model = 'project.project'
            active_id = self._context.get('default_project_id', False)
        result = super(ProjectShareWizard, self.with_context(active_model=active_model, active_id=active_id)).default_get(fields)
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

    def action_share_record(self):
        # Confirmation dialog is only opened if new portal user(s) need to be created in a 'on invitation' website
        self.ensure_one()
        on_invite = self.env['res.users']._get_signup_invitation_scope() == 'b2b'
        new_portal_user = self.partner_ids.filtered(lambda p: not p.user_ids) and on_invite
        if not new_portal_user:
            return self.action_send_mail()
        return {
            'name': _('Confirmation'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'views': [(self.env.ref('project.project_share_wizard_confirm_form').id, 'form')],
            'res_model': 'project.share.wizard',
            'res_id': self.id,
            'target': 'new',
            'context': self.env.context,
        }

    def action_send_mail(self):
        if self.access_mode == 'edit':
            portal_partners = self.partner_ids.filtered('user_ids')
            self.resource_ref._add_collaborators(self.partner_ids)
            self._send_public_link(portal_partners)
            self._send_signup_link(partners=self.with_context({'signup_valid': True}).partner_ids - portal_partners)
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
                <p class="text-muted" invisible="access_mode != 'edit'">
                    When sharing a project in edit mode, you are giving people access to the kanban and list views of your tasks.
                    Collaborators will be able to view and modify part of the tasks' information.
                </p>
                <p class="text-muted" invisible="access_mode == 'edit'">
                    When sharing a project in read-only mode, you are giving people access to the tasks in their portal.
                    These people will be able to view the tasks, but not edit them.
                </p>
                <field name="res_model" invisible="1"/>
                <field name="res_id" invisible="1"/>
                <field name="display_access_mode" invisible="1" />
                <p class="alert alert-warning" invisible="access_warning == ''" role="alert"><field name="access_warning"/></p>
                <group>
                    <field options="{'horizontal': true}" name="access_mode" widget="radio" invisible="not display_access_mode" class="mb-4"/>
                    <field name="share_link" widget="CopyClipboardChar" options="{'string': 'Copy Link'}" invisible="access_mode == 'edit'" string="Link" class="mb-4"/>
                    <div class="o_td_label mb-4">
                        <label for="partner_ids" string="Invite People" invisible="access_mode == 'read'"/>
                        <label for="partner_ids" invisible="access_mode == 'edit'"/>
                    </div>
                    <field name="partner_ids" widget="many2many_tags_email" options="{'no_quick_create': True}" placeholder="Add contacts to share the project..." nolabel="1" context="{'form_view_ref': 'base.view_partner_simple_form'}" class="mb-4"/>
                </group>
                <field name="note" placeholder="Add a note" nolabel="1"/>
                <footer>
                    <button string="Send" name="action_share_record" invisible="access_warning != ''" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="x" />
                </footer>
            </form>
        </field>
    </record>

    <record id="project_share_wizard_confirm_form" model="ir.ui.view">
        <field name="name">project.share.wizard.view.form</field>
        <field name="model">project.share.wizard</field>
        <field name="arch" type="xml">
            <form string="Confirmation">
                <p>People invited to collaborate on the project will have portal access rights.</p>
                <p>They can edit shared project tasks and view specific documents in read-only mode on your website. This includes leads/opportunities, quotations/sales orders, purchase orders, invoices and bills, timesheets, and tickets.</p>
                <p>You have full control and can revoke portal access anytime. Are you ready to proceed?</p>
                <footer>
                    <button string="Grant Portal Access" name="action_send_mail" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="x" />
                </footer>
            </form>
        </field>
    </record>

    <record id="project_share_wizard_action" model="ir.actions.act_window">
        <field name="name">Share Project</field>
        <field name="res_model">project.share.wizard</field>
        <field name="binding_model_id" ref="model_project_project"/>
        <field name="view_mode">form</field>
        <field name="context">{'dialog_size': 'medium'}</field>
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
    _description = 'Project Task Stage Delete Wizard'

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
            action['domain'] = [('project_id', '=', project_id), ('display_in_project', '=', 'True')]
            action['context'] = str({
                'pivot_row_groupby': ['user_ids'],
                'default_project_id': project_id,
            })
        elif self.env.context.get('stage_view'):
            action = self.env["ir.actions.actions"]._for_xml_id("project.open_task_type_form")
        else:
            action = self.env["ir.actions.actions"]._for_xml_id("project.action_view_my_task")

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
                <div invisible="tasks_count &gt; 0">
                    <p>Are you sure you want to delete these stages?</p>
                </div>
                <div invisible="not stages_active or tasks_count == 0">
                    <p>You cannot delete stages containing tasks. You can either archive them or first delete all of their tasks.</p>
                </div>
                <div invisible="stages_active or tasks_count == 0">
                    <p>You cannot delete stages containing tasks. You should first delete all of their tasks.</p>
                </div>
                <footer>
                    <button string="Archive Stages" type="object" name="action_archive" class="btn btn-primary" invisible="not stages_active or tasks_count == 0" data-hotkey="q"/>
                    <button string="Delete" type="object" name="action_unlink" class="btn btn-primary" invisible="tasks_count &gt; 0" data-hotkey="w"/>
                    <button string="Discard" special="cancel" data-hotkey="x" />
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
                    <button string="Discard" special="cancel" data-hotkey="x" />
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

from . import project_project_stage_delete
from . import project_task_type_delete
from . import project_share_wizard

```

