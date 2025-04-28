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

    # Create analytic plan fields on project model for existing plans
    env['account.analytic.plan'].search([])._sync_plan_column('project.project')

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
        'data/project_tour.xml',
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
            'project/static/src/core/web/**/*',
            'project/static/src/utils/**/*',
            'project/static/src/components/**/*',
            'project/static/src/views/**/*',
            'project/static/src/js/tours/project.js',
            'project/static/src/scss/project_dashboard.scss',
            'project/static/src/scss/project_form.scss',
            'project/static/src/scss/project_widgets.scss',
            'project/static/src/xml/**/*',
            ('remove', 'project/static/src/views/project_task_graph/**'),
            ('remove', 'project/static/src/views/project_task_pivot/**'),
            ('remove', 'project/static/src/views/burndown_chart/**'),
        ],
        'web.assets_backend_lazy': [
            'project/static/src/views/project_task_graph/**',
            'project/static/src/views/project_task_pivot/**',
            'project/static/src/views/burndown_chart/**',
        ],
        'web.assets_frontend': [
            'project/static/src/scss/portal_rating.scss',
            'project/static/src/scss/project_sharing_frontend.scss',
            'project/static/src/js/portal_rating.js',
        ],
        'web.assets_unit_tests': [
            'project/static/src/project_sharing/components/portal_file_input/portal_file_input.js',
            'project/static/tests/mock_server/**/*',
            'project/static/tests/project_models.js',
            'project/static/tests/**/*.test.js',
        ],
        'web.qunit_suite_tests': [
            'project/static/src/project_sharing/components/portal_file_input/portal_file_input.js',
            'project/static/tests/legacy/**/*.js',
            'project/static/tests/tours/**/*',
        ],
        'web.assets_tests': [
            'project/static/tests/tours/**/*',
        ],
        'project.webclient': [
            ('include', 'web._assets_helpers'),
            ('include', 'web._assets_backend_helpers'),

            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',
            'web/static/lib/bootstrap/scss/_variables-dark.scss',
            'web/static/lib/bootstrap/scss/_maps.scss',
            ('include', 'web._assets_bootstrap_backend'),

            'web/static/src/libs/fontawesome/css/font-awesome.css',
            'web/static/lib/odoo_ui_icons/*',
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
            'web/static/lib/bootstrap/js/dist/util/index.js',
            'web/static/lib/bootstrap/js/dist/dom/data.js',
            'web/static/lib/bootstrap/js/dist/dom/event-handler.js',
            'web/static/lib/bootstrap/js/dist/dom/manipulator.js',
            'web/static/lib/bootstrap/js/dist/dom/selector-engine.js',
            'web/static/lib/bootstrap/js/dist/util/config.js',
            'web/static/lib/bootstrap/js/dist/util/component-functions.js',
            'web/static/lib/bootstrap/js/dist/util/backdrop.js',
            'web/static/lib/bootstrap/js/dist/util/focustrap.js',
            'web/static/lib/bootstrap/js/dist/util/sanitizer.js',
            'web/static/lib/bootstrap/js/dist/util/scrollbar.js',
            'web/static/lib/bootstrap/js/dist/util/swipe.js',
            'web/static/lib/bootstrap/js/dist/util/template-factory.js',
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
            'web/static/lib/dompurify/DOMpurify.js',
            'web/static/src/libs/bootstrap.js',
            'web/static/src/libs/pdfjs.js',
            'web/static/src/legacy/js/libs/jquery.js',

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

            'web/static/src/legacy/scss/fields.scss',

            'base/static/src/scss/res_partner.scss',

            # Form style should be computed before
            'web/static/src/views/form/button_box/*.scss',

            'bus/static/src/**/*.js',

            'html_editor/static/src/**/*',

            'mail/static/src/scss/variables/*.scss',
            'mail/static/src/chatter/web/form_renderer.scss',
            'mail/static/src/views/fields/**/*',

            'project/static/src/components/project_task_name_with_subtask_count_char_field/*',
            'project/static/src/components/project_task_state_selection/*',
            'project/static/src/components/project_many2one_field/*',
            'project/static/src/views/project_task_form/*.scss',

            ('include', 'portal.assets_chatter_helpers'),
            'portal/static/src/chatter/core/**/*',
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
from odoo.osv.expression import AND, FALSE_DOMAIN
from odoo.tools import groupby as groupbyelem

from odoo.addons.portal.controllers.portal import CustomerPortal, pager as portal_pager


class ProjectCustomerPortal(CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        if 'project_count' in counters:
            values['project_count'] = request.env['project.project'].search_count([]) \
                if request.env['project.project'].has_access('read') else 0
        if 'task_count' in counters:
            values['task_count'] = request.env['project.task'].search_count([('project_id', '!=', False)])\
                if request.env['project.task'].has_access('read') else 0
        return values

    # ------------------------------------------------------------
    # My Project
    # ------------------------------------------------------------
    def _project_get_page_view_values(self, project, access_token, page=1, date_begin=None, date_end=None, sortby=None, search=None, search_in='content', groupby=None, **kwargs):
        # default filter by value
        domain = [('project_id', '=', project.id)]
        # pager
        url = "/my/projects/%s" % project.id
        values = self._prepare_tasks_values(page, date_begin, date_end, sortby, search, search_in, groupby, url, domain, su=bool(access_token) and request.env.user.has_group('base.group_public'), project=project)
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
            multiple_projects=False,
            task_url=f'projects/{project.id}/task',
            preview_object=project,
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
        if not sortby:
            sortby = 'name'
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
            groupby = 'stage_id'
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
            action_name="project.project_sharing_project_task_action",
            action_context={
                'allow_milestones': project.allow_milestones,
            },
            project_id=project.id,
            project_name=project.name,
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
                'show_project': True,
                'task': task_sudo,
                'grouped_tasks': values['grouped_tasks'](pager['offset']),
                'pager': pager,
                'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
                'filterby': filterby,
            })
            return request.render("project.portal_my_tasks", values)
        except (AccessError, MissingError):
            return request.not_found()

    @http.route('/my/projects/<int:project_id>/task/<int:task_id>/recurrent_tasks', type='http', auth='user', methods=['GET'], website=True)
    def portal_my_project_recurrent_tasks(self, project_id, task_id, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, search=None, search_in='content', groupby=None, **kw):
        try:
            project_sudo = self._document_check_access('project.project', project_id)
            task_sudo = request.env['project.task'].search([('project_id', '=', project_id), ('id', '=', task_id)], limit=1).sudo()
            task_domain = [('id', 'in', task_sudo.recurrence_id.task_ids.ids)]
            searchbar_filters = self._get_my_tasks_searchbar_filters([('id', '=', project_id)], task_domain)

            if not filterby:
                filterby = 'all'
            domain = searchbar_filters.get(filterby, searchbar_filters.get('all'))['domain']

            values = self._prepare_tasks_values(
                page, date_begin, date_end, sortby, search, search_in, groupby,
                url=f'/my/projects/{project_id}/task/{task_id}/recurrent_tasks',
                domain=AND([task_domain, domain])
            )
            values['page_name'] = 'project_recurrent_tasks'
            pager = portal_pager(**values['pager'])

            values.update({
                'project': project_sudo,
                'task': task_sudo,
                'grouped_tasks': values['grouped_tasks'](pager['offset']),
                'pager': pager,
                'searchbar_filters': dict(sorted(searchbar_filters.items())),
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
            'create_date desc': {'label': _('Newest'), 'order': 'create_date desc', 'sequence': 10},
            'name': {'label': _('Title'), 'order': 'name', 'sequence': 20},
            'stage_id, project_id': {'label': _('Stage'), 'order': 'stage_id, project_id', 'sequence': 50},
            'state': {'label': _('Status'), 'order': 'state', 'sequence': 60},
            'priority desc': {'label': _('Priority'), 'order': 'priority desc', 'sequence': 80},
            'date_deadline asc': {'label': _('Deadline'), 'order': 'date_deadline asc', 'sequence': 90},
            'date_last_stage_update desc': {'label': _('Last Stage Update'), 'order': 'date_last_stage_update desc', 'sequence': 110},
        }
        if not project:
            values['project_id, stage_id'] = {'label': _('Project'), 'order': 'project_id, stage_id', 'sequence': 30}
        if milestones_allowed:
            values['milestone_id'] = {'label': _('Milestone'), 'order': 'milestone_id', 'sequence': 70}
        return values

    def _task_get_searchbar_groupby(self, milestones_allowed, project=False):
        values = {
            'none': {'label': _('None'), 'sequence': 10},
            'stage_id': {'label': _('Stage'), 'sequence': 20},
            'state': {'label': _('Status'), 'sequence': 40},
            'priority': {'label': _('Priority'), 'sequence': 60},
            'partner_id': {'label': _('Customer'), 'sequence': 70},
        }
        if not project:
            values['project_id'] = {'label': _('Project'), 'sequence': 30}
        if milestones_allowed:
            values['milestone_id'] = {'label': _('Milestone'), 'sequence': 50}
        return values

    def _task_get_searchbar_inputs(self, milestones_allowed, project=False):
        values = {
            'name': {'input': 'name', 'label': _(
                'Search%(left)s Tasks%(right)s',
                left=Markup('<span class="nolabel">'),
                right=Markup('</span>'),
            ), 'sequence': 10},
            'users': {'input': 'user_ids', 'label': _('Search in Assignees'), 'sequence': 20},
            'stage_id': {'input': 'stage_id', 'label': _('Search in Stages'), 'sequence': 30},
            'status': {'input': 'status', 'label': _('Search in Status'), 'sequence': 40},
            'priority': {'input': 'priority', 'label': _('Search in Priority'), 'sequence': 60},
            'partner_id': {'input': 'partner_id', 'label': _('Search in Customer'), 'sequence': 80},
        }
        if not project:
            values['project_id'] = {'input': 'project_id', 'label': _('Search in Project'), 'sequence': 50}
        if milestones_allowed:
            values['milestone_id'] = {'input': 'milestone_id', 'label': _('Search in Milestone'), 'sequence': 70}

        return values

    def _task_get_search_domain(self, search_in, search, milestones_allowed, project):
        if not search_in or search_in == 'name':
            return ['|', ('name', 'ilike', search), ('id', 'ilike', search)]
        elif search_in == 'users':
            user_ids = request.env['res.users'].sudo().search([('name', 'ilike', search)])
            return [('user_ids', 'in', user_ids.ids)]
        elif search_in == 'priority':
            return [('priority', 'ilike', '0' if search == 'normal' else '1')]
        elif search_in == 'status':
            state_dict = dict(map(reversed, request.env['project.task']._fields['state']._description_selection(request.env)))
            return [('state', 'ilike', state_dict.get(search, search))]
        elif search_in in self._task_get_searchbar_inputs(milestones_allowed, project):
            return [(search_in, 'ilike', search)]
        else:
            return ['|', ('name', 'ilike', search), ('id', 'ilike', search)]

    def _prepare_tasks_values(self, page, date_begin, date_end, sortby, search, search_in, groupby, url="/my/tasks", domain=None, su=False, project=False):
        values = self._prepare_portal_layout_values()

        Task = request.env['project.task']

        if not domain:
            domain = []
        if not su and Task.has_access('read'):
            domain = AND([domain, request.env['ir.rule']._compute_domain(Task._name, 'read')])
        Task_sudo = Task.sudo()
        milestone_domain = AND([domain, [('allow_milestones', '=', True)], [('milestone_id', '!=', False)]])
        milestones_allowed = Task_sudo.search_count(milestone_domain, limit=1) == 1
        searchbar_sortings = dict(sorted(self._task_get_searchbar_sortings(milestones_allowed, project).items(),
                                         key=lambda item: item[1]["sequence"]))
        searchbar_inputs = dict(sorted(self._task_get_searchbar_inputs(milestones_allowed, project).items(), key=lambda item: item[1]['sequence']))
        searchbar_groupby = dict(sorted(self._task_get_searchbar_groupby(milestones_allowed, project).items(), key=lambda item: item[1]['sequence']))

        # default sort by value
        if not sortby or (sortby == 'milestone_id' and not milestones_allowed):
            sortby = 'create_date desc'

        # default group by value
        if not groupby or (groupby == 'milestone_id' and not milestones_allowed):
            groupby = 'project_id'

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        # search reset if needed
        if not milestones_allowed and search_in == 'milestone_id':
            search_in = 'all'
        # search
        if search and search_in:
            domain = AND([domain, self._task_get_search_domain(search_in, search, milestones_allowed, project)])

        # content according to pager and archive selected
        if groupby == 'none':
            group_field = None
        elif groupby == 'priority':
            group_field = 'priority desc'
        else:
            group_field = groupby
        order = '%s, %s' % (group_field, sortby) if group_field else sortby

        def get_grouped_tasks(pager_offset):
            tasks = Task_sudo.search(domain, order=order, limit=self._items_per_page, offset=pager_offset)
            request.session['my_project_tasks_history' if url.startswith('/my/projects') else 'my_tasks_history'] = tasks.ids[:100]

            tasks_project_allow_milestone = tasks.filtered(lambda t: t.allow_milestones)
            tasks_no_milestone = tasks - tasks_project_allow_milestone

            if groupby != 'none':
                if groupby == 'milestone_id':
                    grouped_tasks = [Task_sudo.concat(*g) for k, g in groupbyelem(tasks_project_allow_milestone, itemgetter(groupby))]

                    if not grouped_tasks:
                        if tasks_no_milestone:
                            grouped_tasks = [tasks_no_milestone]
                    else:
                        if grouped_tasks[len(grouped_tasks) - 1][0].milestone_id and tasks_no_milestone:
                            grouped_tasks.append(tasks_no_milestone)
                        else:
                            grouped_tasks[len(grouped_tasks) - 1] |= tasks_no_milestone

                else:
                    grouped_tasks = [Task_sudo.concat(*g) for k, g in groupbyelem(tasks, itemgetter(groupby))]
            else:
                grouped_tasks = [tasks] if tasks else []


            task_states = dict(Task_sudo._fields['state']._description_selection(request.env))
            if sortby == 'state':
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
            'multiple_projects': True,
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
        projects = request.env['project.project'].search(project_domain or [], order="id")
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
    def portal_my_tasks(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, search=None, search_in='name', groupby=None, **kw):
        searchbar_filters = self._get_my_tasks_searchbar_filters()

        if not filterby:
            filterby = 'all'
        domain = searchbar_filters.get(filterby, searchbar_filters.get('all'))['domain']

        values = self._prepare_tasks_values(page, date_begin, date_end, sortby, search, search_in, groupby, domain=domain)

        # pager
        pager_vals = values['pager']
        pager_vals['url_args'].update(filterby=filterby)
        pager = portal_pager(**pager_vals)

        grouped_tasks = values['grouped_tasks'](pager['offset'])
        values.update({
            'grouped_tasks': grouped_tasks,
            'show_project': True,
            'pager': pager,
            'searchbar_filters': searchbar_filters,
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import Forbidden

from odoo.http import request

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

```

## File: controllers\thread.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request
from odoo.osv import expression
from odoo.addons.mail.controllers import thread


class ThreadController(thread.ThreadController):
    def _filter_message_post_partners(self, thread, partners):
        if thread._name == "project.task":
            domain = [
                ("res_model", "=", "project.task"),
                ("res_id", "=", thread.id),
                ("partner_id", "in", partners.ids),
            ]
            if thread.project_id:
                project_domain = [
                    ("res_model", "=", "project.project"),
                    ("res_id", "=", thread.project_id.id),
                    ("partner_id", "in", partners.ids),
                ]
                domain = expression.OR([domain, project_domain])
            # sudo: mail.followers - filtering partners that are followers is acceptable
            return request.env["mail.followers"].sudo().search(domain).partner_id
        return super()._filter_message_post_partners(thread, partners)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal
from . import project_sharing_chatter
from . import thread

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

        <record id="digest_tip_project_2" model="digest.tip">
            <field name="name">Tip: Your Own Personal Kanban</field>
            <field name="sequence">3200</field>
            <field name="group_id" ref="project.group_project_user"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Your Own Personal Kanban</p>
    <p class="tip_content">Use personal stages to organize your tasks and create your own workflow.</p>
    <img src="https://download.odoocdn.com/digests/project/static/src/img/18-project-personal-stages.gif" width="540" class="illustration_border"/>
</div>
            </field>
        </record>

        <record id="digest_tip_project_3" model="digest.tip">
            <field name="name">Tip: Project-Specific Fields</field>
            <field name="sequence">3900</field>
            <field name="group_id" ref="project.group_project_manager"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Project-Specific Fields</p>
    <p class="tip_content">Add project-specific property fields on tasks to customize your project management process.</p>
    <img src="https://download.odoocdn.com/digests/project/static/src/img/18-project-property-fields.gif" width="540" class="illustration_border"/>
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
        <field name="name">Task Cancelled</field>
        <field name="res_model">project.task</field>
        <field name="default" eval="False"/>
        <field name="sequence" eval="104"/>
        <field name="description">Task cancelled</field>
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
            <field name="description">Automatically send an email to customers when a task reaches a specific stage in a project by setting this template on that stage</field>
            <field name="body_html" type="html">
<div>
    Dear <t t-out="object.partner_id.name or 'customer'">Brandon Freeman</t>,<br/><br/>
    Thank you for contacting us. We appreciate your interest in our products/services.<br/>
    Our team is currently reviewing your inquiry and will respond to your email as soon as possible.<br/>
    If you have any further questions or concerns in the meantime, please do not hesitate to let us know. We are here to help.<br/><br/>
    Thank you for your patience.<br/>
    Best regards,
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
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="width:100%; margin: 32px 0px 32px 0px; display: inline-table;">
                <tr><td style="font-size: 13px;text-align:center;">
                    <strong>Tell us how you feel about our services</strong><br/>
                    <span style="font-size: 12px; opacity: 0.5; color: #454748;">(click on one of these smileys)</span>
                </td></tr>
                <tr><td style="font-size: 13px;">
                    <table style="width:100%;text-align:center;margin-top:2rem;">
                        <tr>
                            <td>
                                <a t-attf-href="/rate/{{ access_token }}/5" t-att-class="'pe-none' if object._rating_get_operator() else ''">
                                    <img alt="Satisfied" src="/rating/static/src/img/rating_5.png" title="Satisfied"/>
                                </a>
                            </td>
                            <td>
                                <a t-attf-href="/rate/{{ access_token }}/3" t-att-class="'pe-none' if object._rating_get_operator() else ''">
                                    <img alt="Okay" src="/rating/static/src/img/rating_3.png" title="Okay"/>
                                </a>
                            </td>
                            <td>
                                <a t-attf-href="/rate/{{ access_token }}/1" t-att-class="'pe-none' if object._rating_get_operator() else ''">
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

        <!-- You have been invited to follow the task -->
        <template id="task_invitation_follower">
<div>
    Hello <t t-out="partner_name"/>,
    <br/><br/>
    <span style="margin-top: 8px;">You have been invited to follow Task Document : <t t-out="object.display_name"/>.</span>
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
        <field name="name">Cancelled</field>
        <field name="fold" eval="True"/>
    </record>

    <!-- Project Task export template -->
    <record id="project_task_export_template" model="ir.exports">
        <field name="name">Tasks</field>
        <field name="resource">project.task</field>
    </record>

    <record id="project_task_export_template_line_id" model="ir.exports.line">
        <field name="export_id" ref="project_task_export_template"/>
        <field name="name">id</field>
    </record>

    <record id="project_task_export_template_line_project_id" model="ir.exports.line">
        <field name="export_id" ref="project_task_export_template"/>
        <field name="name">project_id</field>
    </record>

    <record id="project_task_export_template_line_name" model="ir.exports.line">
        <field name="export_id" ref="project_task_export_template"/>
        <field name="name">name</field>
    </record>

    <record id="project_task_export_template_line_user_ids" model="ir.exports.line">
        <field name="export_id" ref="project_task_export_template"/>
        <field name="name">user_ids</field>
    </record>

    <record id="project_task_export_template_line_stage_id" model="ir.exports.line">
        <field name="export_id" ref="project_task_export_template"/>
        <field name="name">stage_id</field>
    </record>

    <record id="project_task_export_template_line_state" model="ir.exports.line">
        <field name="export_id" ref="project_task_export_template"/>
        <field name="name">state</field>
    </record>

    <record id="project_task_export_template_line_tag_ids" model="ir.exports.line">
        <field name="export_id" ref="project_task_export_template"/>
        <field name="name">tag_ids</field>
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
        <record id="project_tags_06" model="project.tags">
            <field name="name">Construction</field>
        </record>
        <record id="project_tags_07" model="project.tags">
            <field name="name">Architecture</field>
        </record>
        <record id="project_tags_08" model="project.tags">
            <field name="name">Design</field>
        </record>
        <record id="project_tags_09" model="project.tags">
            <field name="name">Interior</field>
        </record>
        <record id="project_tags_10" model="project.tags">
            <field name="name">Office</field>
        </record>
        <record id="project_tags_11" model="project.tags">
            <field name="name">Finance</field>
        </record>
        <record id="project_tags_12" model="project.tags">
            <field name="name">Social</field>
        </record>
        <record id="project_tags_13" model="project.tags">
            <field name="name">Home</field>
        </record>
        <record id="project_tags_14" model="project.tags">
            <field name="name">Work</field>
        </record>
        <record id="project_tags_15" model="project.tags">
            <field name="name">Meeting</field>
        </record>
        <record id="project_tags_16" model="project.tags">
            <field name="name">Priority</field>
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

        <record id="analytic_construction" model="account.analytic.account">
            <field name="name">Home Construction</field>
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
            <field name="fold" eval="False"/>
        </record>
        <record id="project_stage_3" model="project.task.type">
            <field name="sequence">30</field>
            <field name="name">Cancelled</field>
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
            <field name="account_id" ref="project.analytic_office_design"/>
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
            <field name="account_id" ref="project.analytic_research_development"/>
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
            <field name="account_id" ref="project.analytic_renovations"/>
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

        <record id="project_home_construction" model="project.project">
            <field name="date_start" eval="(DateTime.today() + relativedelta(months=-2)).strftime('%Y-%m-%d 10:00:00')"/>
            <field name="date" eval="(DateTime.today() + relativedelta(days=-5)).strftime('%Y-%m-%d 17:00:00')"/>
            <field name="name">Home Construction</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="description">Designing and construction of an appealing, comfortable, safe and well-built home that fits the needs and wants of you and your family.</field>
            <field name="color">5</field>
            <field name="user_id" ref="base.user_demo"/>
            <field name="type_ids" eval="[Command.link(ref('project_stage_0')), Command.link(ref('project_stage_1')), Command.link(ref('project_stage_2')), Command.link(ref('project_stage_3'))]"/>
            <field name="favorite_user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="tag_ids" eval="[Command.link(ref('project_tags_06')), Command.link(ref('project_tags_07')), Command.link(ref('project_tags_08')), Command.link(ref('project_tags_09'))]"/>
            <field name="stage_id" ref="project.project_project_stage_2"/>
            <field name="account_id" ref="project.analytic_construction"/>
            <field name="privacy_visibility">portal</field>
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
            <field name="name">Cancelled</field>
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
            <field name="name">Cancelled</field>
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

        <!-- Project Home Construction -->
        <record id="project_home_construction_milestone_1" model="project.milestone">
            <field name="is_reached" eval="True"/>
            <field name="deadline" eval="time.strftime('%Y-%m-2')"/>
            <field name="name">Piping Installation</field>
            <field name="reached_date" eval="time.strftime('%Y-%m-2')"/>
            <field name="project_id" ref="project.project_home_construction"/>
        </record>
        <record id="project_home_construction_milestone_2" model="project.milestone">
            <field name="is_reached" eval="False"/>
            <field name="deadline" eval="time.strftime('%Y-%m-5')"/>
            <field name="name">Door and Window Framing</field>
            <field name="project_id" ref="project.project_home_construction"/>
        </record>
        <record id="project_home_construction_milestone_3" model="project.milestone">
            <field name="is_reached" eval="False"/>
            <field name="deadline" eval="time.strftime('%Y-%m-10')"/>
            <field name="name">Wiring</field>
            <field name="project_id" ref="project.project_home_construction"/>
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
            <field name="description">Facebook and X integration</field>
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
            <field name="description">Select a thoughtful gift for Marc Demo's birthday .</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_16')])]"/>
        </record>
        <record id="project_private_task_2" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Change left screen cable</field>
            <field name="description">Perform a replacement of the left screen cable to address any issues with connectivity or display quality.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_14')])]"/>
        </record>
        <record id="project_private_task_3" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="project_id"/>
            <field name="name">Clean kitchen fridge</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_13')])]"/>
        </record>
        <record id="project_private_task_4" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_demo'))]"/>
            <field name="project_id"/>
            <field name="name">Check employees lunch accounts</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_10')])]"/>
        </record>

        <!-- demo data for Today-->
        <record id="project_private_task_5" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="project_id"/>
            <field name="name">Attend a project status meeting</field>
            <field name="description">Join the scheduled project status meeting to provide updates, discuss progress, and align with team members.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_15')])]"/>
        </record>
        <record id="project_private_task_6" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Review and reply to emails</field>
            <field name="description">Spend time going through inbox, reviewing and replying to important emails to stay organized and maintain communication.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_10')])]"/>
        </record>

        <!-- demo data  for This week-->
        <record id="project_private_task_7" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="project_id"/>
            <field name="name">Empty Trash Bins</field>
            <field name="description">Take a few minutes to empty and replace the trash bins in the office. Keeping living space clean and odor-free.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_14')])]"/>
        </record>

        <!-- demo data  for This month-->
        <record id="project_private_task_8" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="project_id"/>
            <field name="name">Conduct a training session</field>
            <field name="description">Conduct a training session to enhance team members proficiency and productivity.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_10')])]"/>
        </record>
        <record id="project_private_task_9" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Review Finances</field>
            <field name="description">Take a comprehensive look at finances, including budgeting, savings, and investments.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_11')])]"/>
        </record>

        <!-- demo data  for Later-->
        <record id="project_private_task_10" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Research Investment Options</field>
            <field name="description">Conduct thorough research on various investment options to make informed decisions about growing your financial portfolio.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_11')])]"/>
        </record>
        <record id="project_private_task_11" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin')), Command.link(ref('base.user_demo'))]"/>
            <field name="project_id"/>
            <field name="name">Explore Charitable Opportunities</field>
            <field name="description">Explore various charitable organizations and opportunities to contribute to meaningful causes and make a positive impact on the community.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_12')])]"/>
        </record>

        <!-- demo data  for Cancel-->
        <record id="project_private_task_12" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Attend a Local Event</field>
            <field name="description">Participate in a local event to connect with the community</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_12')])]"/>
        </record>
        <record id="project_private_task_13" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Review Document</field>
            <field name="description">Thoroughly review and provide feedback on an important document.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_10')])]"/>
        </record>

        <!-- Demo data Archived tasks  -->
        <record id="project_private_task_14" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Lead a Team Discussion</field>
            <field name="description">Facilitate a team discussion to encourage idea sharing and decision-making.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_16'), ref('project_tags_15')])]"/>
            <field name="active" eval="False"/>
        </record>
        <record id="project_private_task_15" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Attend a Training or Workshop</field>
            <field name="description">Participate in a training session or workshop to develop new skills or gain knowledge relevant to your work.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_14')])]"/>
            <field name="active" eval="False"/>
        </record>
        <record id="project_private_task_16" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Review and Provide Feedback</field>
            <field name="description">Attend a meeting to review documents, proposals, or reports and provide constructive feedback.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_10')])]"/>
            <field name="active" eval="False"/>
        </record>

        <!-- Demo data for Done which are Closed/Mark as Done tasks -->
        <record id="project_private_task_17" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Participate in a Design Review</field>
            <field name="description">Join a meeting where the team reviews and discusses the design or user experience aspects of a project.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_15')])]"/>
            <field name="state">1_done</field>
        </record>
        <record id="project_private_task_18" model="project.task">
            <field name="user_ids" eval="[Command.link(ref('base.user_admin'))]"/>
            <field name="project_id"/>
            <field name="name">Share Research Findings</field>
            <field name="description">Present findings from research you conducted to inform a project or decision.</field>
            <field name="tag_ids" eval="[Command.set([ref('project_tags_14')])]"/>
            <field name="state">1_done</field>
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

        <!-- demo data functions for Demo  -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_5')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_1')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_7')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_2')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_8')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_3')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_11')), ('user_id', '=', ref('base.user_demo'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_demo_4')}"/>
        </function>

        <!-- demo data fuctions for today -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_5')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_1')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_6')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_1')}"/>
        </function>

        <!-- demo data functions for this week -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_7')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_2')}"/>
        </function>

        <!-- demo data functions for this month -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_8')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_3')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_9')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_3')}"/>
        </function>

        <!-- demo data functions for Later -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_10')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_4')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_11')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_4')}"/>
        </function>

        <!-- demo data functions for Cancel -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_12')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_6')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_13')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_6')}"/>
        </function>

        <!-- demo data function for Archived tasks  -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_14')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_1')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_15')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_2')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_16')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_3')}"/>
        </function>

        <!-- Demo data function for Done which are Closed/Mark as Done tasks -->
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_17')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
        </function>
        <function model="project.task.stage.personal" name="write">
            <value model="project.task.stage.personal" search="[('task_id', '=', ref('project_private_task_18')), ('user_id', '=', ref('base.user_admin'))]"/>
            <value eval="{'stage_id': ref('project_personal_stage_admin_5')}"/>
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
        <!-- Home Construction Project Update -->
        <record id="project_update_construction_1" model="project.update" context="{'default_project_id': ref('project.project_home_construction')}">
            <field name="name">Design Approval</field>
            <field name="user_id" eval="ref('base.user_admin')"/>
            <field name="progress" eval="65"/>
            <field name="status">on_track</field>
            <field name="date" eval="time.strftime('%Y-%m-5')"/>
        </record>
        <record id="project_update_construction_2" model="project.update" context="{'default_project_id': ref('project.project_home_construction')}">
            <field name="name">Construction</field>
            <field name="user_id" eval="ref('base.user_admin')"/>
            <field name="progress" eval="30"/>
            <field name="status">on_track</field>
            <field name="date" eval="time.strftime('%Y-%m-1')"/>
        </record>
        <!-- Scheduled activity for Todo Task   -->
        <record id="project_private_task_9_mail" model="mail.activity">
            <field name="res_id" ref="project_private_task_9"/>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=20)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Collect data regarding finances</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="project_private_task_6_mail" model="mail.activity">
            <field name="res_id" ref="project_private_task_6"/>
            <field name="res_model_id" ref="project.model_project_task"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(hours=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Review Mark as Important Emails First</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
    </data>
</odoo>

```

## File: data\project_tour.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_tour" model="web_tour.tour">
        <field name="name">project_tour</field>
        <field name="sequence">110</field>
        <field name="rainbow_man_message">Congratulations, you are now a master of project management.</field>
    </record>
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

    project_ids = fields.One2many('project.project', 'account_id', string='Projects', export_string_translation=False)
    project_count = fields.Integer("Project Count", compute='_compute_project_count', export_string_translation=False)

    @api.depends('project_ids')
    def _compute_project_count(self):
        project_data = self.env['project.project']._read_group([('account_id', 'in', self.ids)], ['account_id'], ['__count'])
        mapping = {analytic_account.id: count for analytic_account, count in project_data}
        for account in self:
            account.project_count = mapping.get(account.id, 0)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_existing_tasks(self):
        projects = self.env['project.project'].search([('account_id', 'in', self.ids)])
        has_tasks = self.env['project.task'].search_count([('project_id', 'in', projects.ids)])
        if has_tasks:
            raise UserError(_('Please remove existing tasks in the project linked to the accounts you want to delete.'))

    def action_view_projects(self):
        kanban_view_id = self.env.ref('project.view_project_kanban').id
        result = {
            "type": "ir.actions.act_window",
            "res_model": "project.project",
            "views": [[kanban_view_id, "kanban"], [False, "form"]],
            "domain": [['account_id', '=', self.id]],
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
    kpi_project_task_opened_value = fields.Integer(compute='_compute_project_task_opened_value', export_string_translation=False)

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
        res['kpi_project_task_opened'] = 'project.open_view_project_all?menu_id=%s' % self.env.ref('project.menu_main_pm').id
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProjectCollaborator(models.Model):
    _name = 'project.collaborator'
    _description = 'Collaborators in project shared'

    project_id = fields.Many2one('project.project', 'Project Shared', domain=[('privacy_visibility', '=', 'portal')], required=True, readonly=True, export_string_translation=False)
    partner_id = fields.Many2one('res.partner', 'Collaborator', required=True, readonly=True, export_string_translation=False)
    partner_email = fields.Char(related='partner_id.email', export_string_translation=False)
    limited_access = fields.Boolean('Limited Access', default=False, export_string_translation=False)

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
from odoo.tools import format_date

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
    reached_date = fields.Date(compute='_compute_reached_date', store=True, export_string_translation=False)
    task_ids = fields.One2many('project.task', 'milestone_id', 'Tasks', export_string_translation=False)

    # computed non-stored fields
    is_deadline_exceeded = fields.Boolean(compute="_compute_is_deadline_exceeded", export_string_translation=False)
    is_deadline_future = fields.Boolean(compute="_compute_is_deadline_future", export_string_translation=False)
    task_count = fields.Integer('# of Tasks', compute='_compute_task_count', groups='project.group_project_milestone, export_string_translation=False')
    done_task_count = fields.Integer('# of Done Tasks', compute='_compute_task_count', groups='project.group_project_milestone', export_string_translation=False)
    can_be_marked_as_done = fields.Boolean(compute='_compute_can_be_marked_as_done', export_string_translation=False)

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
                milestone.can_be_marked_as_done = not milestone.is_reached and all(milestone.task_ids.mapped(lambda t: t.is_closed))
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

    def copy(self, default=None):
        default = dict(default or {})
        new_milestones = super().copy(default)
        milestone_mapping = self.env.context.get('milestone_mapping', {})
        for old_milestone, new_milestone in zip(self, new_milestones):
            if old_milestone.project_id.allow_milestones:
                milestone_mapping[old_milestone.id] = new_milestone.id
        return new_milestones

    def _compute_display_name(self):
        super()._compute_display_name()
        if not self._context.get('display_milestone_deadline'):
            return
        for milestone in self:
            if milestone.deadline:
                milestone.display_name = f'{milestone.display_name} - {format_date(self.env, milestone.deadline)}'

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import json
from datetime import timedelta

from odoo import api, Command, fields, models
from odoo.addons.mail.tools.discuss import Store
from odoo.addons.rating.models import rating_data
from odoo.exceptions import UserError
from odoo.osv.expression import AND
from odoo.tools import get_lang, SQL
from odoo.tools.misc import unquote
from odoo.tools.translate import _
from .project_update import STATUS_COLOR
from .project_task import CLOSED_STATES


class Project(models.Model):
    _name = "project.project"
    _description = "Project"
    _inherit = [
        'portal.mixin',
        'mail.alias.mixin',
        'rating.parent.mixin',
        'mail.thread',
        'mail.activity.mixin',
        'mail.tracking.duration.mixin',
        'analytic.plan.fields.mixin',
    ]
    _order = "sequence, name, id"
    _rating_satisfaction_days = 30  # takes 30 days by default
    _systray_view = 'activity'
    _track_duration_field = 'stage_id'

    def __compute_task_count(self, count_field='task_count', additional_domain=None):
        count_fields = {fname for fname in self._fields if 'count' in fname}
        if count_field not in count_fields:
            raise ValueError(f"Parameter 'count_field' can only be one of {count_fields}, got {count_field} instead.")
        domain = [('project_id', 'in', self.ids), ('display_in_project', '=', True)]
        if additional_domain:
            domain = AND([domain, additional_domain])
        tasks_count_by_project = dict(self.env['project.task'].with_context(
            active_test=any(project.active for project in self)
        )._read_group(domain, ['project_id'], ['__count']))
        for project in self:
            project.update({count_field: tasks_count_by_project.get(project, 0)})

    def _compute_task_count(self):
        self.__compute_task_count()

    def _compute_open_task_count(self):
        self.__compute_task_count(
            count_field='open_task_count',
            additional_domain=[('state', 'in', self.env['project.task'].OPEN_STATES)],
        )

    def _compute_closed_task_count(self):
        self.__compute_task_count(
            count_field='closed_task_count',
            additional_domain=[('state', 'in', [*CLOSED_STATES])],
        )

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

    def _set_favorite_user_ids(self, is_favorite):
        self_sudo = self.sudo() # To allow project users to set projects as favorite
        if is_favorite:
            self_sudo.favorite_user_ids = [Command.link(self.env.uid)]
        else:
            self_sudo.favorite_user_ids = [Command.unlink(self.env.uid)]

    name = fields.Char("Name", index='trigram', required=True, tracking=True, translate=True, default_export_compatible=True)
    description = fields.Html(help="Description to provide more information and context about this project")
    active = fields.Boolean(default=True, copy=False, export_string_translation=False)
    sequence = fields.Integer(default=10, export_string_translation=False)
    partner_id = fields.Many2one('res.partner', string='Customer', auto_join=True, tracking=True, domain="['|', ('company_id', '=?', company_id), ('company_id', '=', False)]")
    company_id = fields.Many2one('res.company', string='Company', compute="_compute_company_id", inverse="_inverse_company_id", store=True, readonly=False)
    currency_id = fields.Many2one('res.currency', compute="_compute_currency_id", string="Currency", readonly=True, export_string_translation=False)
    analytic_account_balance = fields.Monetary(related="account_id.balance")
    account_id = fields.Many2one('account.analytic.account', copy=False, domain="['|', ('company_id', '=', False), ('company_id', '=?', company_id)]", ondelete='set null')

    favorite_user_ids = fields.Many2many(
        'res.users', 'project_favorite_user_rel', 'project_id', 'user_id',
        string='Members', export_string_translation=False, copy=False)
    is_favorite = fields.Boolean(compute='_compute_is_favorite', readonly=False, search='_search_is_favorite',
        compute_sudo=True, string='Show Project on Dashboard', export_string_translation=False)
    label_tasks = fields.Char(string='Use Tasks as', default=lambda s: s.env._('Tasks'), translate=True,
        help="Name used to refer to the tasks of your project e.g. tasks, tickets, sprints, etc...")
    tasks = fields.One2many('project.task', 'project_id', string="Task Activities")
    resource_calendar_id = fields.Many2one(
        'resource.calendar', string='Working Time', compute='_compute_resource_calendar_id', export_string_translation=False)
    type_ids = fields.Many2many('project.task.type', 'project_task_type_rel', 'project_id', 'type_id', string='Tasks Stages', export_string_translation=False)
    task_count = fields.Integer(compute='_compute_task_count', string="Task Count", export_string_translation=False)
    open_task_count = fields.Integer(compute='_compute_open_task_count', string="Open Task Count", export_string_translation=False)
    task_ids = fields.One2many('project.task', 'project_id', string='Tasks', export_string_translation=False,
                               domain=lambda self: [('is_closed', '=', False)])
    color = fields.Integer(string='Color Index', export_string_translation=False)
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
        tracking=True,
        help="People to whom this project and its tasks will be visible.\n\n"
            "- Invited internal users: when following a project, internal users will get access to all of its tasks without distinction. "
            "Otherwise, they will only get access to the specific tasks they are following.\n "
            "A user with the project > administrator access right level can still access this project and its tasks, even if they are not explicitly part of the followers.\n\n"
            "- All internal users: all internal users can access the project and all of its tasks without distinction.\n\n"
            "- Invited portal users and all internal users: all internal users can access the project and all of its tasks without distinction.\n"
            "When following a project, portal users will only get access to the specific tasks they are following.\n\n"
            "When a project is shared in read-only, the portal user is redirected to their portal. They can view the tasks they are following, but not edit them.\n"
            "When a project is shared in edit, the portal user is redirected to the kanban and list views of the tasks. They can modify a selected number of fields on the tasks.\n\n"
            "In any case, an internal user with no project access rights can still access a task, "
            "provided that they are given the corresponding URL (and that they are part of the followers if the project is private).")
    privacy_visibility_warning = fields.Char('Privacy Visibility Warning', compute='_compute_privacy_visibility_warning', export_string_translation=False)
    access_instruction_message = fields.Char('Access Instruction Message', compute='_compute_access_instruction_message', export_string_translation=False)
    date_start = fields.Date(string='Start Date')
    date = fields.Date(string='Expiration Date', index=True, tracking=True,
        help="Date on which this project ends. The timeframe defined on the project is taken into account when viewing its planning.")
    allow_task_dependencies = fields.Boolean('Task Dependencies', default=lambda self: self.env.user.has_group('project.group_project_task_dependencies'), inverse='_inverse_allow_task_dependencies')
    allow_milestones = fields.Boolean('Milestones', default=lambda self: self.env.user.has_group('project.group_project_milestone'))
    tag_ids = fields.Many2many('project.tags', relation='project_project_project_tags_rel', string='Tags')
    task_properties_definition = fields.PropertiesDefinition('Task Properties')
    closed_task_count = fields.Integer(compute="_compute_closed_task_count", export_string_translation=False)
    task_completion_percentage = fields.Float(compute="_compute_task_completion_percentage", export_string_translation=False)

    # Project Sharing fields
    collaborator_ids = fields.One2many('project.collaborator', 'project_id', string='Collaborators', copy=False, export_string_translation=False)
    collaborator_count = fields.Integer('# Collaborators', compute='_compute_collaborator_count', compute_sudo=True, export_string_translation=False)

    # rating fields
    rating_request_deadline = fields.Datetime(compute='_compute_rating_request_deadline', store=True, export_string_translation=False)
    rating_active = fields.Boolean('Customer Ratings', default=lambda self: self.env.user.has_group('project.group_project_rating'))
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
        tracking=True, index=True, copy=False, default=_default_stage_id, group_expand='_read_group_expand_full')
    duration_tracking = fields.Json(groups="project.group_project_stages")

    update_ids = fields.One2many('project.update', 'project_id', export_string_translation=False)
    update_count = fields.Integer(compute='_compute_total_update_ids', export_string_translation=False)
    last_update_id = fields.Many2one('project.update', string='Last Update', copy=False, export_string_translation=False)
    last_update_status = fields.Selection(selection=[
        ('on_track', 'On Track'),
        ('at_risk', 'At Risk'),
        ('off_track', 'Off Track'),
        ('on_hold', 'On Hold'),
        ('to_define', 'Set Status'),
        ('done', 'Done'),
    ], default='to_define', compute='_compute_last_update_status', store=True, readonly=False, required=True, export_string_translation=False)
    last_update_color = fields.Integer(compute='_compute_last_update_color', export_string_translation=False)
    milestone_ids = fields.One2many('project.milestone', 'project_id', copy=True, export_string_translation=False)
    milestone_count = fields.Integer(compute='_compute_milestone_count', groups='project.group_project_milestone', export_string_translation=False)
    milestone_count_reached = fields.Integer(compute='_compute_milestone_reached_count', groups='project.group_project_milestone', export_string_translation=False)
    is_milestone_exceeded = fields.Boolean(compute="_compute_is_milestone_exceeded", search='_search_is_milestone_exceeded', export_string_translation=False)
    milestone_progress = fields.Integer("Milestones Reached", compute='_compute_milestone_reached_count', groups="project.group_project_milestone", export_string_translation=False)
    next_milestone_id = fields.Many2one('project.milestone', compute='_compute_next_milestone_id', groups="project.group_project_milestone", export_string_translation=False)
    can_mark_milestone_as_done = fields.Boolean(compute='_compute_next_milestone_id', groups="project.group_project_milestone", export_string_translation=False)
    is_milestone_deadline_exceeded = fields.Boolean(compute='_compute_next_milestone_id', groups="project.group_project_milestone", export_string_translation=False)

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

    @api.depends('milestone_ids', 'milestone_ids.is_reached', 'milestone_ids.deadline')
    def _compute_next_milestone_id(self):
        milestone_ids_per_project_id = {
            project.id: milestone_ids
            for project, milestone_ids in self.env['project.milestone']._read_group(
                [('project_id', 'in', self.ids), ('is_reached', '=', False)],
                ['project_id'],
                ['id:recordset'],
            )
        }
        for project in self:
            milestone = milestone_ids_per_project_id.get(project.id, self.env['project.milestone'])[:1]
            project.next_milestone_id = milestone
            project.can_mark_milestone_as_done = milestone.can_be_marked_as_done
            project.is_milestone_deadline_exceeded = milestone.is_deadline_exceeded

    def _compute_access_url(self):
        super(Project, self)._compute_access_url()
        for project in self:
            project.access_url = f'/my/projects/{project.id}'

    def _compute_access_warning(self):
        super(Project, self)._compute_access_warning()
        for project in self.filtered(lambda x: x.privacy_visibility != 'portal'):
            project.access_warning = _(
                "This project is currently restricted to \"Invited internal users\". The project's visibility will be changed to \"invited portal users and all internal users (public)\" in order to make it accessible to the recipients.")

    @api.depends('account_id.company_id', 'partner_id.company_id')
    def _compute_company_id(self):
        for project in self:
            # if a new restriction is put on the account or the customer, the restriction on the project is updated.
            if project.account_id.company_id:
                project.company_id = project.account_id.company_id
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
            account = project.account_id
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

    @api.depends('milestone_ids.is_reached', 'milestone_count')
    def _compute_milestone_reached_count(self):
        read_group = self.env['project.milestone']._read_group(
            [('project_id', 'in', self.ids), ('is_reached', '=', True)],
            ['project_id'],
            ['__count'],
        )
        mapped_count = {project.id: count for project, count in read_group}
        for project in self:
            project.milestone_count_reached = mapped_count.get(project.id, 0)
            project.milestone_progress = project.milestone_count and project.milestone_count_reached * 100 // project.milestone_count

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

        sql = SQL("""(
            SELECT P.id
              FROM project_project P
         LEFT JOIN project_milestone M ON P.id = M.project_id
             WHERE M.is_reached IS false
               AND P.allow_milestones IS true
               AND M.deadline <= CAST(now() AS date)
        )""")
        if (operator == '=' and value is True) or (operator == '!=' and value is False):
            operator_new = 'in'
        else:
            operator_new = 'not in'
        return [('id', operator_new, sql)]

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
                project.access_instruction_message = _('Grant portal users access to your project by adding them as followers (the tasks of the project are not included). To grant access to tasks to a portal user, add them as followers for these tasks.')
            elif project.privacy_visibility == 'followers':
                project.access_instruction_message = _('Grant employees access to your project or tasks by adding them as followers. Employees automatically get access to the tasks they are assigned to.')
            else:
                project.access_instruction_message = ''

    @api.depends('update_ids')
    def _compute_total_update_ids(self):
        update_count_per_project = dict(
            self.env['project.update']._read_group(
                [('project_id', 'in', self.ids)],
                ['project_id'],
                ['id:count'],
            )
        )
        for project in self:
            project.update_count = update_count_per_project.get(project, 0)

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

    @api.model
    def _map_tasks_default_values(self, project):
        """ get the default value for the copied task on project duplication.
        The stage_id, name field will be set for each task in the overwritten copy_data function in project.task """
        return {
            'state': '01_in_progress',
            'company_id': project.company_id.id,
            'project_id': project.id,
        }

    def map_tasks(self, new_project_id):
        """ copy and map tasks from old to new project """
        project = self.browse(new_project_id)
        new_tasks = self.env['project.task']
        # We want to copy archived task, but do not propagate an active_test context key
        tasks = self.env['project.task'].with_context(active_test=False).search([('project_id', '=', self.id), ('parent_id', '=', False)])
        if self.allow_task_dependencies and 'task_mapping' not in self.env.context:
            self = self.with_context(task_mapping=dict())
        # preserve task name and stage, normally altered during copy
        defaults = self._map_tasks_default_values(project)
        new_tasks = tasks.with_context(copy_project=True).copy(defaults)
        all_subtasks = new_tasks._get_all_subtasks()
        project.write({'tasks': [Command.set(new_tasks.ids)]})
        subtasks_not_displayed = all_subtasks.filtered(
            lambda task: not task.display_in_project
        )
        all_subtasks.filtered(
            lambda child: child.project_id == self
        ).write({
            'project_id': project.id
        })
        subtasks_not_displayed.write({
            'display_in_project': False
        })
        return True

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        if default and 'name' in default:
            return vals_list
        return [dict(vals, name=self.env._("%s (copy)", project.name)) for project, vals in zip(self, vals_list)]

    def copy(self, default=None):
        default = dict(default or {})
        # Since we dont want to copy the milestones if the original project has the feature disabled, we set the milestones to False by default.
        default['milestone_ids'] = False
        copy_context = dict(
             self.env.context,
             mail_auto_subscribe_no_notify=True,
             mail_create_nosubscribe=True,
         )
        copy_context.pop("default_stage_id", None)
        new_projects = super(Project, self.with_context(copy_context)).copy(default=default)
        if 'milestone_mapping' not in self.env.context:
            self = self.with_context(milestone_mapping={})
        actions_per_project = dict(self.env['ir.embedded.actions']._read_group(
            domain=[
                ('parent_res_id', 'in', self.ids),
                ('parent_res_model', '=', 'project.project'),
                ('user_id', '=', False),
            ],
            groupby=['parent_res_id'],
            aggregates=['id:recordset'],
        ))
        for old_project, new_project in zip(self, new_projects):
            for follower in old_project.message_follower_ids:
                new_project.message_subscribe(partner_ids=follower.partner_id.ids, subtype_ids=follower.subtype_ids.ids)
            if old_project.allow_milestones:
                new_project.milestone_ids = self.milestone_ids.copy().ids
            if 'tasks' not in default:
                old_project.map_tasks(new_project.id)
            if not old_project.active:
                new_project.with_context(active_test=False).tasks.active = True
            # Copy the shared embedded actions in the new project
            shared_embedded_actions = actions_per_project.get(old_project.id)
            if shared_embedded_actions:
                copy_shared_embedded_actions = shared_embedded_actions.copy({'parent_res_id': new_project.id})
                for original_action, copied_action in zip(shared_embedded_actions, copy_shared_embedded_actions):
                    copied_action.filter_ids = original_action.filter_ids.copy({'embedded_parent_res_id': new_project.id})
        return new_projects

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

        for vals in vals_list:
            if vals.pop('is_favorite', False):
                vals['favorite_user_ids'] = [self.env.uid]
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
            self._set_favorite_user_ids(vals.pop('is_favorite'))

        if 'last_update_status' in vals and vals['last_update_status'] != 'to_define':
            for project in self:
                # This does not benefit from multi create, this is to allow the default description from being built.
                # This does seem ok since last_update_status should only be updated on one record at once.
                self.env['project.update'].with_context(default_project_id=project.id).create({
                    'name': _('Status Update - %(date)s', date=fields.Date.today().strftime(get_lang(self.env).date_format)),
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
        if 'name' in vals and self.account_id:
            projects_read_group = self.env['project.project']._read_group(
                [('account_id', 'in', self.account_id.ids)],
                ['account_id'],
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
        for project in self:
            if project.account_id and not project.account_id.line_ids:
                analytic_accounts_to_delete |= project.account_id
        self.with_context(active_test=False).tasks.unlink()
        result = super(Project, self).unlink()
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

    def message_unsubscribe(self, partner_ids=None):
        super().message_unsubscribe(partner_ids=partner_ids)
        if partner_ids:
            self.env['project.collaborator'].search([('partner_id', 'in', partner_ids), ('project_id', 'in', self.ids)]).unlink()

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
                    _('This project is associated with %(project_company)s, whereas the selected stage belongs to %(stage_company)s. '
                    'There are a couple of options to consider: either remove the company designation '
                    'from the project or from the stage. Alternatively, you can update the company '
                    'information for these records to align them under the same company.', project_company=project.company_id.name, stage_company=project.stage_id.company_id.name)
                    if project.company_id else
                    _('This project is not associated with any company, while the stage is associated with %s. '
                    'There are a couple of options to consider: either change the project\'s company '
                    'to align with the stage\'s company or remove the company designation from the stage', project.stage_id.company_id.name)
                )

    # ---------------------------------------------------
    # Mail gateway
    # ---------------------------------------------------

    def _track_template(self, changes):
        res = super()._track_template(changes)
        project = self[0]
        if self.env.user.has_group('project.group_project_stages') and 'stage_id' in changes and project.stage_id.mail_template_id:
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

    def project_update_all_action(self):
        action = self.env['ir.actions.act_window']._for_xml_id('project.project_update_all_action')
        action['display_name'] = _("%(name)s Dashboard", name=self.name)
        return action

    def action_open_share_project_wizard(self):
        template = self.env.ref('project.mail_template_project_sharing', raise_if_not_found=False)

        local_context = self.env.context | {
            'default_template_id': template.id if template else False,
            'default_email_layout_xmlid': 'mail.mail_notification_light',
            'active_id': self.id,
            'active_model': 'project.project',
        }
        action = self.env["ir.actions.actions"]._for_xml_id("project.project_share_wizard_action")
        if self.env.context.get('default_access_mode'):
            action['name'] = _("Share Project")
        action['context'] = local_context
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
        action['display_name'] = self.name
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
        action_context['search_default_filter_write_date'] = 'custom_write_date_last_30_days'
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
            'views': [(self.env.ref('project.project_milestone_view_tree').id, 'list')],
            'view_mode': 'list',
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
        if not self.env.user.has_group('project.group_project_user'):
            return {}
        show_profitability = self._show_profitability()
        panel_data = {
            'user': self._get_user_values(),
            'buttons': sorted(self._get_stat_buttons(), key=lambda k: k['sequence']),
            'currency_id': self.currency_id.id,
            'show_project_profitability_helper': show_profitability and self._show_profitability_helper(),
            'show_milestones': self.allow_milestones,
        }
        if self.allow_milestones:
            panel_data['milestones'] = self._get_milestones()
        if show_profitability:
            profitability_items = self.with_context(active_test=False)._get_profitability_items()
            if self._get_profitability_sequence_per_invoice_type() and profitability_items and 'revenues' in profitability_items and 'costs' in profitability_items:  # sort the data values
                profitability_items['revenues']['data'] = sorted(profitability_items['revenues']['data'], key=lambda k: k['sequence'])
                profitability_items['costs']['data'] = sorted(profitability_items['costs']['data'], key=lambda k: k['sequence'])
            panel_data['profitability_items'] = profitability_items
            panel_data['profitability_labels'] = self._get_profitability_labels()
        return panel_data

    def get_milestones(self):
        if self.env.user.has_group('project.group_project_user'):
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
            'is_project_user': self.env.user.has_group('project.group_project_user'),
        }

    def _show_profitability(self):
        self.ensure_one()
        return True

    def _show_profitability_helper(self):
        return self.env.user.has_group('analytic.group_analytic_accounting')

    def _get_profitability_aal_domain(self):
        return [('account_id', 'in', self.account_id.ids)]

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
        closed_task_count = self.task_count - self.open_task_count
        if self.task_count:
            number = self.env._(
                "%(closed_task_count)s / %(task_count)s (%(closed_rate)s%%)",
                closed_task_count=closed_task_count,
                task_count=self.task_count,
                closed_rate=round(100 * closed_task_count / self.task_count),
            )
        else:
            number = self.env._(
                "%(closed_task_count)s / %(task_count)s",
                closed_task_count=closed_task_count,
                task_count=self.task_count,
            )
        buttons = [{
            'icon': 'check',
            'text': self.env._('Tasks'),
            'number': number,
            'action_type': 'object',
            'action': 'action_view_tasks',
            'show': True,
            'sequence': 1,
        }]
        if self.rating_count != 0 and self.env.user.has_group('project.group_project_rating'):
            if self.rating_avg >= rating_data.RATING_AVG_TOP:
                icon = 'smile-o text-success'
            elif self.rating_avg >= rating_data.RATING_AVG_OK:
                icon = 'meh-o text-warning'
            else:
                icon = 'frown-o text-danger'
            buttons.append({
                'icon': icon,
                'text': self.env._('Average Rating'),
                'number': f'{int(self.rating_avg) if self.rating_avg.is_integer() else round(self.rating_avg, 1)} / 5',
                'action_type': 'object',
                'action': 'action_view_all_rating',
                'show': self.rating_active,
                'sequence': 15,
            })
        if self.env.user.has_group('project.group_project_user'):
            buttons.append({
                'icon': 'area-chart',
                'text': self.env._('Burndown Chart'),
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
    def _get_values_analytic_account_batch(self, project_vals_list):
        project_plan, _other_plans = self.env['account.analytic.plan']._get_all_plans()
        return [{
            'name': project_vals.get('name', self.env._('Unknown Analytic Account')),
            'company_id': project_vals.get('company_id', False),
            'partner_id': project_vals.get('partner_id', False),
            'plan_id': project_plan.id,
        } for project_vals in project_vals_list]

    def _create_analytic_account(self):
        analytic_accounts_values = self._get_values_analytic_account_batch(self._read_format(['name', 'company_id', 'partner_id'], None))
        analytic_accounts = self.env['account.analytic.account'].create(analytic_accounts_values)
        for project, analytic_account in zip(self, analytic_accounts):
            project.account_id = analytic_account

    def _get_projects_to_make_billable_domain(self):
        return [('partner_id', '!=', False)]

    @api.constrains(lambda self: self._get_plan_fnames())
    def _check_account_id(self):
        # Overriden from 'analytic.plan.fields.mixin'
        pass

    def _get_plan_domain(self, plan):
        return AND([super()._get_plan_domain(plan), ['|', ('company_id', '=', False), ('company_id', '=?', unquote('company_id'))]])

    def _get_account_node_context(self, plan):
        return {
            **super()._get_account_node_context(plan),
            'default_company_id': unquote('company_id'),
        }

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
        if self.env.user._is_portal():
            return self.env['project.collaborator'].search([('project_id', '=', self.sudo().id), ('partner_id', '=', self.env.user.partner_id.id)])
        return self.env.user._is_internal()

    def _add_collaborators(self, partners, limited_access=False):
        self.ensure_one()
        new_collaborators = self._get_new_collaborators(partners)
        if not new_collaborators:
            # Then we have nothing to do
            return
        self.write({'collaborator_ids': [
            Command.create({
                'partner_id': collaborator.id,
                'limited_access': limited_access,
            }) for collaborator in new_collaborators],
        })

    def _get_new_collaborators(self, partners):
        self.ensure_one()
        return partners.filtered(
            lambda partner:
                partner not in self.collaborator_ids.partner_id
                and partner.partner_share
        )

    def _add_followers(self, partners):
        self.ensure_one()
        self.message_subscribe(partners.ids)

        dict_tasks_per_partner = {}
        dict_partner_ids_to_subscribe_per_partner = {}
        for task in self.task_ids:
            if task.partner_id in dict_tasks_per_partner:
                dict_tasks_per_partner[task.partner_id] |= task
            else:
                partner_ids_to_subscribe = [
                    partner.id for partner in partners
                    if partner == task.partner_id or partner in task.partner_id.child_ids
                ]
                if partner_ids_to_subscribe:
                    dict_tasks_per_partner[task.partner_id] = task
                    dict_partner_ids_to_subscribe_per_partner[task.partner_id] = partner_ids_to_subscribe
        for partner, tasks in dict_tasks_per_partner.items():
            tasks.message_subscribe(dict_partner_ids_to_subscribe_per_partner[partner])

    def _thread_to_store(self, store: Store, /, *, request_list=None, **kwargs):
        super()._thread_to_store(store, request_list=request_list, **kwargs)
        if request_list and "followers" in request_list:
            store.add(
                self,
                {"collaborator_ids": Store.many(self.collaborator_ids.partner_id, only_id=True)},
                as_thread=True,
            )

    @api.depends('task_count', 'open_task_count')
    def _compute_task_completion_percentage(self):
        for task in self:
            task.task_completion_percentage = task.task_count and 1 - task.open_task_count / task.task_count

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

    active = fields.Boolean(default=True, export_string_translation=False)
    sequence = fields.Integer(default=50, export_string_translation=False)
    name = fields.Char(required=True, translate=True)
    mail_template_id = fields.Many2one('mail.template', string='Email Template', domain=[('model', '=', 'project.project')],
        help="If set, an email will be automatically sent to the customer when the project reaches this stage.")
    fold = fields.Boolean('Folded in Kanban',
        help="If enabled, this stage will be displayed as folded in the Kanban view of your projects. Projects in a folded stage are considered as closed.")
    company_id = fields.Many2one('res.company', string="Company")

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, name=self.env._("%s (copy)", stage.name)) for stage, vals in zip(self, vals_list)]

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

from odoo import api, fields, models
from odoo.osv import expression
from odoo.tools import SQL


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
    project_ids = fields.Many2many('project.project', 'project_project_project_tags_rel', string='Projects', export_string_translation=False)
    task_ids = fields.Many2many('project.task', string='Tasks', export_string_translation=False)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "A tag with the same name already exists."),
    ]

    def _get_project_tags_domain(self, domain, project_id):
        # TODO: Remove in master
        return domain

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if 'project_id' in self.env.context:
            tag_ids = [id_ for id_, _label in self.name_search()]
            domain = expression.AND([domain, [('id', 'in', tag_ids)]])
        return super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    @api.model
    def search_read(self, domain=None, fields=None, offset=0, limit=None, order=None):
        if 'project_id' in self.env.context:
            tag_ids = [id_ for id_, _label in self.name_search()]
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
    def name_search(self, name='', args=None, operator='ilike', limit=100):
        if limit is None:
            return super().name_search(name, args, operator, limit)
        tags = self.browse()
        domain = expression.AND([self._search_display_name(operator, name), args or []])
        if self.env.context.get('project_id'):
            # optimisation for large projects, we look first for tags present on the last 1000 tasks of said project.
            # when not enough results are found, we complete them with a fallback on a regular search
            tag_sql = SQL("""
                (SELECT DISTINCT project_tasks_tags.id
                FROM (
                    SELECT rel.project_tags_id AS id
                    FROM project_tags_project_task_rel AS rel
                    JOIN project_task AS task
                        ON task.id=rel.project_task_id
                        AND task.project_id=%(project_id)s
                    ORDER BY task.id DESC
                    LIMIT 1000
                ) AS project_tasks_tags
            )""", project_id=self.env.context['project_id'])
            tags += self.search_fetch(expression.AND([[('id', 'in', tag_sql)], domain]), ['display_name'], limit=limit)
        if len(tags) < limit:
            tags += self.search_fetch(expression.AND([[('id', 'not in', tags.ids)], domain]), ['display_name'], limit=limit - len(tags))
        return [(tag.id, tag.display_name) for tag in tags.sudo()]

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

from odoo import api, Command, fields, models, tools, SUPERUSER_ID, _
from odoo.addons.rating.models import rating_data
from odoo.addons.web_editor.tools import handle_history_divergence
from odoo.exceptions import UserError, ValidationError, AccessError
from odoo.osv import expression
from odoo.tools import format_list, SQL
from odoo.addons.resource.models.utils import filter_domain_leaf
from odoo.addons.project.controllers.project_sharing_chatter import ProjectSharingChatter
from odoo.addons.mail.tools.discuss import Store


PROJECT_TASK_READABLE_FIELDS = {
    'id',
    'active',
    'priority',
    'project_id',
    'display_in_project',
    'color',
    'allow_task_dependencies',
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
    'current_user_same_company_partner',
    'allow_milestones',
    'milestone_id',
    'has_late_and_unreached_milestone',
    'date_assign',
    'dependent_ids',
    'message_is_follower',
    'recurring_task',
    'closed_subtask_count',
    'dependent_tasks_count',
    'depend_on_ids',
    'depend_on_count',
    'repeat_interval',
    'repeat_unit',
    'repeat_type',
    'repeat_until',
    'recurrence_id',
    'recurring_count',
    'duration_tracking',
    'display_follow_button',
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
    'is_closed',
}

CLOSED_STATES = {
    '1_done': 'Done',
    '1_canceled': 'Cancelled',
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
        'mail.tracking.duration.mixin',
        'html.field.history.mixin',
    ]
    _mail_post_access = 'read'
    _order = "priority desc, sequence, date_deadline asc, id desc"
    _primary_email = 'email_from'
    _systray_view = 'activity'
    _track_duration_field = 'stage_id'

    def _get_versioned_fields(self):
        return [Task.description.name]

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
    def _read_group_stage_ids(self, stages, domain):
        search_domain = [('id', 'in', stages.ids)]
        if 'default_project_id' in self.env.context and not self._context.get('subtask_action') and 'project_kanban' in self.env.context:
            search_domain = ['|', ('project_ids', '=', self.env.context['default_project_id'])] + search_domain

        stage_ids = stages.sudo()._search(search_domain, order=stages._order)
        return stages.browse(stage_ids)

    @api.model
    def _read_group_personal_stage_type_ids(self, stages, domain):
        return stages.search(['|', ('id', 'in', stages.ids), ('user_id', '=', self.env.user.id)])

    active = fields.Boolean(default=True, export_string_translation=False)
    name = fields.Char(string='Title', tracking=True, required=True, index='trigram')
    description = fields.Html(string='Description', sanitize_attributes=False)
    priority = fields.Selection([
        ('0', 'Low'),
        ('1', 'High'),
    ], default='0', index=True, string="Priority", tracking=True)
    sequence = fields.Integer(string='Sequence', default=10, export_string_translation=False)
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
    is_closed = fields.Boolean("Closed state", compute='_compute_is_closed', search='_search_is_closed')

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

    project_id = fields.Many2one('project.project', string='Project', domain="['|', ('company_id', '=', False), ('company_id', '=?',  company_id)]",
                                 compute="_compute_project_id", store=True, precompute=True, recursive=True, readonly=False, index=True, tracking=True, change_default=True)
    display_in_project = fields.Boolean(compute='_compute_display_in_project', store=True, readonly=False, export_string_translation=False)
    # Technical field to display the 'Display in Project' button in the form view, depending on the project
    show_display_in_project = fields.Boolean(compute='_compute_show_display_in_project')
    task_properties = fields.Properties('Properties', definition='project_id.task_properties_definition', copy=True)
    allocated_hours = fields.Float("Allocated Time", tracking=True)
    subtask_allocated_hours = fields.Float("Sub-tasks Allocated Time", compute='_compute_subtask_allocated_hours', export_string_translation=False,
        help="Sum of the hours allocated for all the sub-tasks (and their own sub-tasks) linked to this task. Usually less than or equal to the allocated hours of this task.")
    # Tracking of this field is done in the write function
    user_ids = fields.Many2many('res.users', relation='project_task_user_rel', column1='task_id', column2='user_id',
        string='Assignees', context={'active_test': False}, tracking=True, default=_default_user_ids, domain="[('share', '=', False), ('active', '=', True)]")
    # User names displayed in project sharing views
    portal_user_names = fields.Char(compute='_compute_portal_user_names', compute_sudo=True, search='_search_portal_user_names', export_string_translation=False)
    # Second Many2many containing the actual personal stage for the current user
    # See project_task_stage_personal.py for the model defininition
    personal_stage_type_ids = fields.Many2many('project.task.type', 'project_task_user_rel', column1='task_id', column2='stage_id',
        ondelete='restrict', group_expand='_read_group_personal_stage_type_ids', copy=False,
        domain="[('user_id', '=', uid)]", string='Personal Stages', export_string_translation=False)
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
    color = fields.Integer(string='Color Index', export_string_translation=False)
    rating_active = fields.Boolean(string='Project Rating Status', related="project_id.rating_active")
    attachment_ids = fields.One2many(
        'ir.attachment',
        compute='_compute_attachment_ids',
        string="Attachments",
        export_string_translation=False,
        help="Attachments that don't come from a message",
    )
    # In the domain of displayed_image_id, we couln't use attachment_ids because a one2many is represented as a list of commands so we used res_model & res_id
    displayed_image_id = fields.Many2one('ir.attachment', domain="[('res_model', '=', 'project.task'), ('res_id', '=', id), ('mimetype', 'ilike', 'image')]", string='Cover Image')

    parent_id = fields.Many2one('project.task', string='Parent Task', index=True, domain="['!', ('id', 'child_of', id)]", tracking=True)
    child_ids = fields.One2many('project.task', 'parent_id', string="Sub-tasks", domain="[('recurring_task', '=', False)]", export_string_translation=False)
    subtask_count = fields.Integer("Sub-task Count", compute='_compute_subtask_count', export_string_translation=False)
    closed_subtask_count = fields.Integer("Closed Sub-tasks Count", compute='_compute_subtask_count', export_string_translation=False)
    project_privacy_visibility = fields.Selection(related='project_id.privacy_visibility', string="Project Visibility", tracking=False)
    subtask_completion_percentage = fields.Float(compute="_compute_subtask_completion_percentage", export_string_translation=False)
    # Computed field about working time elapsed between record creation and assignation/closing.
    working_hours_open = fields.Float(compute='_compute_elapsed', string='Working Hours to Assign', digits=(16, 2), store=True, aggregator="avg")
    working_hours_close = fields.Float(compute='_compute_elapsed', string='Working Hours to Close', digits=(16, 2), store=True, aggregator="avg")
    working_days_open = fields.Float(compute='_compute_elapsed', string='Working Days to Assign', store=True, aggregator="avg")
    working_days_close = fields.Float(compute='_compute_elapsed', string='Working Days to Close', store=True, aggregator="avg")
    # customer portal: include comment and (incoming/outgoing) emails in communication history
    website_message_ids = fields.One2many(domain=lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment', 'email_outgoing'])], export_string_translation=False)
    allow_milestones = fields.Boolean(related='project_id.allow_milestones', export_string_translation=False)
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
        export_string_translation=False,
    )
    # Task Dependencies fields
    allow_task_dependencies = fields.Boolean(related='project_id.allow_task_dependencies', export_string_translation=False)
    # Tracking of this field is done in the write function
    depend_on_ids = fields.Many2many('project.task', relation="task_dependencies_rel", column1="task_id",
                                     column2="depends_on_id", string="Blocked By", tracking=True, copy=False,
                                     domain="[('project_id', '!=', False), ('id', '!=', id)]")
    depend_on_count = fields.Integer(string="Depending on Tasks", compute='_compute_depend_on_count', compute_sudo=True)
    closed_depend_on_count = fields.Integer(string="Closed Depending on Tasks", compute='_compute_depend_on_count', compute_sudo=True)
    dependent_ids = fields.Many2many('project.task', relation="task_dependencies_rel", column1="depends_on_id",
                                     column2="task_id", string="Block", copy=False,
                                     domain="[('project_id', '!=', False), ('id', '!=', id)]", export_string_translation=False)
    dependent_tasks_count = fields.Integer(string="Dependent Tasks", compute='_compute_dependent_tasks_count', export_string_translation=False)

    # Project sharing fields
    display_parent_task_button = fields.Boolean(compute='_compute_display_parent_task_button', compute_sudo=True, export_string_translation=False)
    current_user_same_company_partner = fields.Boolean(compute='_compute_current_user_same_company_partner', compute_sudo=True, export_string_translation=False)
    display_follow_button = fields.Boolean(compute='_compute_display_follow_button', compute_sudo=True, export_string_translation=False)

    # recurrence fields
    recurring_task = fields.Boolean(string="Recurrent")
    recurring_count = fields.Integer(string="Tasks in Recurrence", compute='_compute_recurring_count')
    recurrence_id = fields.Many2one('project.task.recurrence', copy=False)
    repeat_interval = fields.Integer(string='Repeat Every', default=1, compute='_compute_repeat', compute_sudo=True, readonly=False)
    repeat_unit = fields.Selection([
        ('day', 'Days'),
        ('week', 'Weeks'),
        ('month', 'Months'),
        ('year', 'Years'),
    ], default='week', compute='_compute_repeat', compute_sudo=True, readonly=False)
    repeat_type = fields.Selection([
        ('forever', 'Forever'),
        ('until', 'Until'),
    ], default="forever", string="Until", compute='_compute_repeat', compute_sudo=True, readonly=False)
    repeat_until = fields.Date(string="End Date", compute='_compute_repeat', compute_sudo=True, readonly=False)

    # Quick creation shortcuts
    display_name = fields.Char(
        compute='_compute_display_name',
        inverse='_inverse_display_name',
        search='_search_display_name',
        help="""Use these keywords in the title to set new tasks:\n
            #tags Set tags on the task
            @user Assign the task to a user
            ! Set the task a high priority\n
            Make sure to use the right format and order e.g. Improve the configuration screen #feature #v16 @Mitchell !""",
    )
    link_preview_name = fields.Char(compute='_compute_link_preview_name', export_string_translation=False)

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

    @api.constrains('child_ids', 'project_id')
    def _ensure_super_task_is_not_private(self):
        """ Ensures that the company of the task is valid for the partner. """
        for task in self:
            if not task.project_id and task.subtask_count:
                raise ValidationError(_('This task has sub-tasks, so it can\'t be private.'))

    @property
    def SELF_READABLE_FIELDS(self):
        return PROJECT_TASK_READABLE_FIELDS | self.SELF_WRITABLE_FIELDS

    @property
    def SELF_WRITABLE_FIELDS(self):
        return PROJECT_TASK_WRITABLE_FIELDS

    @api.depends('parent_id.project_id')
    def _compute_project_id(self):
        self.env.remove_to_compute(self._fields['display_in_project'], self)
        for task in self:
            if not task.display_in_project and task.parent_id and task.parent_id.project_id != task.project_id:
                task.project_id = task.parent_id.project_id

    @api.onchange('parent_id')
    def _onchange_parent_id(self):
        if self.display_in_project:
            return
        if not self.parent_id:
            self.display_in_project = True
        elif self.project_id != self.parent_id.project_id:
            self.project_id = self.parent_id.project_id

    @api.depends('project_id')
    def _compute_display_in_project(self):
        self.filtered(
            lambda t: not t.display_in_project and (
                not t.project_id or t.project_id != t.parent_id.project_id
            )
        ).display_in_project = True

    @api.depends('project_id', 'parent_id')
    def _compute_show_display_in_project(self):
        for task in self:
            task.show_display_in_project = bool(task.parent_id) and task.project_id == task.parent_id.project_id

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

    @api.depends('state')
    def _compute_is_closed(self):
        for task in self:
            task.is_closed = task.state in CLOSED_STATES

    def _search_is_closed(self, operator, value):
        if operator not in ('=', '!=') or not isinstance(value, bool):
            raise NotImplementedError(_(
                "The search does not support operator %(operator)s or value %(value)s.",
                operator=operator,
                value=value,
            ))
        if (operator == '!=' and value) or (operator == '=' and not value):
            searched_states = self.OPEN_STATES
        else:
            searched_states = list(CLOSED_STATES.keys())
        domain = [
            ('state', 'in', searched_states)
        ]
        return domain


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
            {'sequence': 7, 'name': _('Cancelled'), 'user_id': user_id, 'fold': True},
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
        if self._has_cycle('depend_on_ids'):
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

    @api.depends('depend_on_ids')
    def _compute_depend_on_count(self):
        tasks_with_dependency = self.filtered('allow_task_dependencies')
        tasks_without_dependency = self - tasks_with_dependency
        tasks_without_dependency.depend_on_count = 0
        tasks_without_dependency.closed_depend_on_count = 0
        if not any(self._ids):
            for task in self:
                task.depend_on_count = len(task.depend_on_ids)
                task.closed_depend_on_count = len(task.depend_on_ids.filtered(lambda r: r.state in CLOSED_STATES))
            return
        if tasks_with_dependency:
            # need the sudo for project sharing
            total_and_closed_depend_on_count = {
                dependent_on.id: (count, sum(s in CLOSED_STATES for s in states))
                for dependent_on, states, count in self.env['project.task']._read_group(
                    [('dependent_ids', 'in', tasks_with_dependency.ids)],
                    ['dependent_ids'],
                    ['state:array_agg', '__count'],
                )
            }
            for task in tasks_with_dependency:
                task.depend_on_count, task.closed_depend_on_count = total_and_closed_depend_on_count.get(task._origin.id or task.id, (0, 0))

    @api.depends('dependent_ids')
    def _compute_dependent_tasks_count(self):
        tasks_with_dependency = self.filtered('allow_task_dependencies')
        (self - tasks_with_dependency).dependent_tasks_count = 0
        if tasks_with_dependency:
            group_dependent = self.env['project.task']._read_group([
                ('depend_on_ids', 'in', tasks_with_dependency.ids),
                ('is_closed', '=', False),
            ], ['depend_on_ids'], ['__count'])
            dependent_tasks_count_dict = {
                depend_on.id: count
                for depend_on, count in group_dependent
            }
            for task in tasks_with_dependency:
                task.dependent_tasks_count = dependent_tasks_count_dict.get(task.id, 0)

    @api.constrains('parent_id')
    def _check_parent_id(self):
        if self._has_cycle():
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
            visibility_field = self.env['ir.model.fields'].search([('model', '=', 'project.project'), ('name', '=', 'privacy_visibility')], limit=1)
            visibility_public = self.env['ir.model.fields.selection'].search([('field_id', '=', visibility_field.id), ('value', '=', 'portal')])
            task.access_warning = _(
                "The task cannot be shared with the recipient(s) because the privacy of the project is too restricted. Set the privacy of the project to '%(visibility)s' in order to make it accessible by the recipient(s).",
                visibility=visibility_public.name,
            )

    @api.depends('child_ids.allocated_hours')
    def _compute_subtask_allocated_hours(self):
        for task in self:
            task.subtask_allocated_hours = sum(task.child_ids.mapped('allocated_hours'))

    @api.depends('child_ids')
    def _compute_subtask_count(self):
        if not any(self._ids):
            for task in self:
                task.subtask_count, task.closed_subtask_count = len(task.child_ids), len(task.child_ids.filtered(lambda r: r.state in CLOSED_STATES))
            return
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
            task.portal_user_names = format_list(self.env, task.user_ids.mapped('name'))

    def _search_portal_user_names(self, operator, value):
        if operator != 'ilike' and not isinstance(value, str):
            raise ValidationError(_('Not Implemented.'))

        sql = SQL("""(
            SELECT task_user.task_id
              FROM project_task_user_rel task_user
        INNER JOIN res_users users ON task_user.user_id = users.id
        INNER JOIN res_partner partners ON partners.id = users.partner_id
             WHERE partners.name ILIKE %s
        )""", f"%{value}%")
        return [('id', 'in', sql)]

    def _compute_display_parent_task_button(self):
        accessible_parent_tasks = self.parent_id.with_user(self.env.user)._filtered_access('read')
        for task in self:
            task.display_parent_task_button = task.parent_id in accessible_parent_tasks

    def _compute_current_user_same_company_partner(self):
        commercial_partner_id = self.env.user.partner_id.commercial_partner_id
        for task in self:
            task.current_user_same_company_partner = task.partner_id and commercial_partner_id == task.partner_id.commercial_partner_id

    def _compute_display_follow_button(self):
        if not self.env.user.share:
            self.display_follow_button = False
            return
        project_collaborator_read_group = self.env['project.collaborator']._read_group(
            [('project_id', 'in', self.project_id.ids), ('partner_id', '=', self.env.user.partner_id.id)],
            ['project_id'],
            ['limited_access:bool_and'],
        )
        limited_access_per_project_id = dict(project_collaborator_read_group)
        for task in self:
            task.display_follow_button = not limited_access_per_project_id.get(task.project_id, True)

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

    def _compute_link_preview_name(self):
        for task in self:
            link_preview_name = task.display_name
            if task.project_id:
                link_preview_name += f' | {task.project_id.sudo().name}'
            task.link_preview_name = link_preview_name

    def copy_data(self, default=None):
        default = dict(default or {})
        vals_list = super().copy_data(default=default)
        not_project_user = not self.env.user.has_group('project.group_project_user')
        if not_project_user:
            vals_list = [{k: v for k, v in vals.items() if k in self.SELF_READABLE_FIELDS} for vals in vals_list]

        milestone_mapping = self.env.context.get('milestone_mapping', {})
        for task, vals in zip(self, vals_list):

            if not default.get('stage_id'):
                vals['stage_id'] = task.stage_id.id
            if 'active' not in default and not task['active'] and not self.env.context.get('copy_project'):
                vals['active'] = True
            vals['name'] = task.name if self.env.context.get('copy_project') else _("%s (copy)", task.name)
            if task.recurrence_id and not default.get('recurrence_id'):
                vals['recurrence_id'] = task.recurrence_id.copy().id
            if task.allow_milestones:
                vals['milestone_id'] = milestone_mapping.get(vals['milestone_id'], vals['milestone_id'])
            if task.child_ids and not default.get('child_ids'):
                default = {
                    'depend_on_ids': False,
                    'dependent_ids': False,
                    'parent_id': False,
                }
                vals['child_ids'] = [Command.create(child_id.copy_data(default)[0]) for child_id in task.child_ids]
        return vals_list

    def _create_task_mapping(self, copied_tasks):
        """
        Thanks to the way create and command.create is handled, when a task with 2 children is copied, we have the guarantee that the children of the
        copied task will have the same index in the child_ids recordset. We can use this behavior to create a mapping containing all the original tasks and their copy.
        :return:
            task_mapping: a dict containing the mapping of the original task ids and their copied task (k: original_task.id, v: new_task)
            task_dependencies: a dict containing the ids of the dependencies of the original task when they have one.
            (k: original_task_id, v: [original_task.depend_on_ids.ids, original_task.dependent_ids.ids]
        """
        task_mapping, task_dependencies = {}, {}
        for original_task, copied_task in zip(self, copied_tasks):
            task_mapping[original_task.id] = copied_task
            if original_task.allow_task_dependencies and (original_task.depend_on_ids or original_task.dependent_ids):
                task_dependencies[original_task.id] = [original_task.depend_on_ids.ids, original_task.dependent_ids.ids]
            if original_task.child_ids:
                # If the task has children, we have to call the method create_task_mapping to get their ids and dependencies mapping too.
                children_mapping, children_dependencies = original_task.child_ids._create_task_mapping(copied_task.child_ids)
                task_mapping.update(children_mapping)
                task_dependencies.update(children_dependencies)
        return task_mapping, task_dependencies

    def _portal_get_parent_hash_token(self, pid):
        return self.project_id._sign_token(pid)

    def copy(self, default=None):
        default = default or {}
        default.update({
            'depend_on_ids': False,
            'dependent_ids': False,
        })
        copied_tasks = super(Task, self.with_context(
            mail_auto_subscribe_no_notify=True,
            mail_create_nosubscribe=True,
            mail_create_nolog=True,
        )).copy(default=default)

        task_mapping, task_dependencies = self._create_task_mapping(copied_tasks)

        for original_task_id, (depend_on_ids, dependant_ids) in task_dependencies.items():
            # If one of the task_id in the dependencies mapping is also a key of the task_mapping, it means that this task was copied too.
            # In this case, we should exchange this id with the id of the corresponding copied task
            task_mapping[original_task_id].depend_on_ids = [
                task_id if task_id not in task_mapping else task_mapping[task_id].id
                for task_id in depend_on_ids
            ]
            task_mapping[original_task_id].dependent_ids = [
                task_id if task_id not in task_mapping else task_mapping[task_id].id
                for task_id in dependant_ids
            ]

        return copied_tasks

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
        if not self.env.user._is_portal():
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
        return key + (self.env.user._is_portal(),)

    @api.model
    def default_get(self, default_fields):
        vals = super(Task, self).default_get(default_fields)

        # prevent creating new task in the waiting state
        if 'state' in default_fields and vals.get('state') == '04_waiting_normal':
            vals['state'] = '01_in_progress'

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
        if fields and (not check_group_user or self.env.user._is_portal()) and not self.env.su:
            unauthorized_fields = set(fields) - (self.SELF_READABLE_FIELDS if operation == 'read' else self.SELF_WRITABLE_FIELDS)
            if unauthorized_fields:
                unauthorized_field_list = format_list(self.env, list(unauthorized_fields))
                if operation == 'read':
                    error_message = _('You cannot read the following fields on tasks: %(field_list)s', field_list=unauthorized_field_list)
                else:
                    error_message = _('You cannot write on the following fields on tasks: %(field_list)s', field_list=unauthorized_field_list)
                raise AccessError(error_message)

    def _determine_fields_to_fetch(self, field_names, ignore_when_in_cache=False):
        if not self.env.su and self.env.user._is_portal():
            valid_names = self.SELF_READABLE_FIELDS
            field_names = [fname for fname in field_names if fname in valid_names]
        return super()._determine_fields_to_fetch(field_names, ignore_when_in_cache)

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

    @api.model
    def check_field_access_rights(self, operation, field_names):
        if field_names and operation in ('read', 'write'):
            self._ensure_fields_are_accessible(field_names, operation)
        elif not field_names and not self.env.su and self.env.user._is_portal():
            valid_names = self.SELF_READABLE_FIELDS
            return [
                fname for fname in super().check_field_access_rights(operation, field_names)
                if fname in valid_names
            ]
        return super().check_field_access_rights(operation, field_names)

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
        default_project_id = new_context.get("default_project_id", False)
        self = self.with_context(new_context)

        is_portal_user = self.env.user._is_portal()
        if is_portal_user:
            self.browse().check_access('create')
        default_stage = dict()
        for vals in vals_list:
            project_id = vals.get('project_id') or default_project_id

            if vals.get('user_ids'):
                vals['date_assign'] = fields.Datetime.now()
                if not (vals.get('parent_id') or project_id):
                    user_ids = self._fields['user_ids'].convert_to_cache(vals.get('user_ids', []), self.env['project.task'])
                    if self.env.user.id not in list(user_ids) + [SUPERUSER_ID]:
                        vals['user_ids'] = [Command.set(list(user_ids) + [self.env.user.id])]
            if default_personal_stage and 'personal_stage_type_id' not in vals:
                vals['personal_stage_type_id'] = default_personal_stage[0]
            if not vals.get('name') and vals.get('display_name'):
                vals['name'] = vals['display_name']
            if is_portal_user:
                self._ensure_fields_are_accessible(vals.keys(), operation='write', check_group_user=False)

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
            tasks.browse().with_user(self.env.user).check_access('create')
        current_partner = self.env.user.partner_id

        all_partner_emails = []
        for task in tasks:
            all_partner_emails += tools.email_split(task.email_cc)
        partners = self.env['res.partner'].search([('email', 'in', all_partner_emails)])
        partner_per_email = {
            partner.email: partner
            for partner in partners
            if not all(u.share for u in partner.user_ids)
        }
        if tasks.project_id:
            tasks.sudo()._set_stage_on_project_from_task()
        for task in tasks:
            if task.project_id.privacy_visibility == 'portal':
                task._portal_ensure_token()
            for follower in task.parent_id.message_follower_ids:
                task.message_subscribe(follower.partner_id.ids, follower.subtype_ids.ids)
            if current_partner not in task.message_partner_ids:
                task.message_subscribe(current_partner.ids)
            if task.email_cc:
                partners_with_internal_user = self.env['res.partner']
                for email in tools.email_split(task.email_cc):
                    new_partner = partner_per_email.get(email)
                    if new_partner:
                        partners_with_internal_user |= new_partner
                if not partners_with_internal_user:
                    continue
                task._send_email_notify_to_cc(partners_with_internal_user)
                task.message_subscribe(partners_with_internal_user.ids)
        return tasks

    def write(self, vals):
        if len(self) == 1:
            handle_history_divergence(self, 'description', vals)
        portal_can_write = False
        project_link_per_task_id = {}
        partner_ids = []
        if self.env.user._is_portal() and not self.env.su:
            # Check if all fields in vals are in SELF_WRITABLE_FIELDS
            self._ensure_fields_are_accessible(vals.keys(), operation='write', check_group_user=False)
            self.check_access('write')
            portal_can_write = True

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

        if vals.get('parent_id') in self.ids:
            raise UserError(_("Sorry. You can't set a task as its parent task."))

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

        # sends an email to the 'Task Creation' subtype subscribers
        # When project_id is changed
        if vals.get('project_id'):
            project = self.env['project.project'].browse(vals.get('project_id'))
            notification_subtype_id = self.env['ir.model.data']._xmlid_to_res_id('project.mt_project_task_new')
            partner_ids = project.message_follower_ids.filtered(lambda follower: notification_subtype_id in follower.subtype_ids.ids).partner_id.ids
            if partner_ids:
                link_per_project_id = {}
                for task in self:
                    if task.project_id:
                        project_link = link_per_project_id.get(task.project_id.id)
                        if not project_link:
                            project_link = link_per_project_id[task.project_id.id] = task.project_id._get_html_link(title=task.project_id.display_name)
                        project_link_per_task_id[task.id] = project_link
        if vals.get('parent_id') is False:
            vals['display_in_project'] = True
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

        if partner_ids:
            for task in self:
                project_link = project_link_per_task_id.get(task.id)
                if project_link:
                    body = _(
                        'Task Transferred from Project %(source_project)s to %(destination_project)s',
                        source_project=project_link,
                        destination_project=self.project_id._get_html_link(title=self.project_id.display_name),
                    )
                else:
                    body = _('Task Converted from To-Do')
                task.message_notify(
                    body=body,
                    partner_ids=partner_ids,
                    email_layout_xmlid='mail.mail_notification_layout',
                    record_name=task.display_name,
               )
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

    def _search_on_comodel(self, domain, field, comodel, additional_domain=None):
        """ This method is called by `group_expand` methods, whose purpose is to add empty groups to the `read_group`
            (which otherwise returns groups containing records that match the domain).
            When specifically filtering on a comodel's field, the result of the `read_group` should contain all matching groups.
            However, if the search isn't filtered on any comodel's field, the result shouldn't be affected,
            which explains why we return `False` if `filtered_domain` is empty.

            Returns:
                False or recordset of the comodel given in parameter.
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
        return self.env[comodel].search(filtered_domain)

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
            raise NotImplementedError(_(
                "The search does not support operator %(operator)s or value %(value)s.",
                operator=operator,
                value=value,
            ))
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

    def _send_email_notify_to_cc(self, partners_to_notify):
        self.ensure_one()
        template_id = self.env['ir.model.data']._xmlid_to_res_id('project.task_invitation_follower', raise_if_not_found=False)
        if not template_id:
            return
        task_model_description = self.env['ir.model']._get(self._name).display_name
        values = {
            'object': self,
        }
        for partner in partners_to_notify:
            values['partner_name'] = partner.name
            assignation_msg = self.env['ir.qweb']._render('project.task_invitation_follower', values, minimal_qcontext=True)
            self.message_notify(
                subject=_('You have been invited to follow %s', self.display_name),
                body=assignation_msg,
                partner_ids=partner.ids,
                record_name=self.display_name,
                email_layout_xmlid='mail.mail_notification_layout',
                model_description=task_model_description,
                mail_auto_delete=True,
            )

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

    def _creation_message(self):
        self.ensure_one()
        if self.project_id:
            return _('A new task has been created in the "%(project_name)s" project.',
                     project_name=self.project_id.display_name)
        return _('A new task has been created and is not part of any project.')

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
                or (not self.project_id and not self.env.user.has_group('project.group_project_task_dependencies')))\
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
        recipients = super()._message_get_suggested_recipients()
        if self.partner_id:
            reason = _('Customer Email') if self.partner_id.email else _('Customer')
            self._message_add_suggested_recipient(recipients, partner=self.partner_id, reason=reason)
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
        if self.parent_id.project_id != self.project_id and self.env.user._is_portal():
            project = self.parent_id.project_id._filtered_access('read')
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

    def action_project_sharing_open_blocking(self):
        self.ensure_one()
        blockings = self.dependent_ids
        action = self.env['ir.actions.act_window']._for_xml_id('project.project_sharing_project_task_action_blocking_tasks')
        if len(blockings) == 1:
            action['view_mode'] = 'form'
            action['views'] = [(view_id, view_type) for view_id, view_type in action['views'] if view_type == 'form']
            action['res_id'] = blockings.id
        return action

    def action_dependent_tasks(self):
        self.ensure_one()
        return {
            'res_model': 'project.task',
            'type': 'ir.actions.act_window',
            'context': {**self._context, 'default_depend_on_ids': [Command.link(self.id)], 'show_project_update': False, 'search_default_open_tasks': True},
            'domain': [('depend_on_ids', '=', self.id)],
            'name': _('Dependent Tasks'),
            'view_mode': 'list,form,kanban,calendar,pivot,graph,activity',
        }

    def action_recurring_tasks(self):
        return {
            'name': _('Tasks in Recurrence'),
            'type': 'ir.actions.act_window',
            'res_model': 'project.task',
            'view_mode': 'list,form,kanban,calendar,pivot,graph,activity',
            'context': {'create': False},
            'domain': [('recurrence_id', 'in', self.recurrence_id.ids)],
        }

    def action_project_sharing_recurring_tasks(self):
        self.ensure_one()
        recurrent_tasks = self.env['project.task'].search([('recurrence_id', 'in', self.recurrence_id.ids)])
        # If all the recurrent tasks are in the same project, open the list view in sharing mode.
        if recurrent_tasks.project_id == self.project_id:
            action = self.env['ir.actions.act_window']._for_xml_id('project.project_sharing_project_task_recurring_tasks_action')
            action.update({
                'context': {'default_project_id': self.project_id.id},
                'domain': [
                    ('project_id', '=', self.project_id.id),
                    ('recurrence_id', 'in', self.recurrence_id.ids)
                ]
            })
            return action
        # If at least one recurrent task belong to another project, open the portal page
        return {
            'name': 'Portal Recurrent Tasks',
            'type': 'ir.actions.act_url',
            'url':  f'/my/projects/{self.project_id.id}/task/{self.id}/recurrent_tasks',
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
                'message': _('Private tasks cannot be converted into sub-tasks. Please set a project on the task to gain access to this feature.'),
            }
        }

    def action_archive(self):
        child_tasks = self.child_ids.filtered(lambda child_task: not child_task.display_in_project)
        if child_tasks:
            child_tasks.action_archive()
        self.filtered(lambda t: not t.display_in_project and t.parent_id).display_in_project = True
        return super().action_archive()

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

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        calendar = self.env.company.resource_calendar_id
        return calendar._get_unusual_days(
            datetime.combine(fields.Date.from_string(date_from), time.min).replace(tzinfo=UTC),
            datetime.combine(fields.Date.from_string(date_to), time.max).replace(tzinfo=UTC)
        )

    def action_redirect_to_project_task_form(self):
        menu_id = self.env.ref('project.menu_project_management_all_tasks').id
        return {
            'type': 'ir.actions.act_url',
            'url': f"/odoo/1/action-project.act_project_project_2_project_task_all/{self.id}?menu_id={menu_id}",
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

    # ---------------------------------------------------
    # Project Sharing
    # ---------------------------------------------------

    def project_sharing_toggle_is_follower(self):
        self.ensure_one()
        self.check_access('write')
        is_follower = self.message_is_follower
        if is_follower:
            self.sudo().message_unsubscribe(self.env.user.partner_id.ids)
        else:
            self.sudo().message_subscribe(self.env.user.partner_id.ids)
        return not is_follower

    @api.depends('subtask_count', 'closed_subtask_count')
    def _compute_subtask_completion_percentage(self):
        for task in self:
            task.subtask_completion_percentage = task.subtask_count and task.closed_subtask_count / task.subtask_count

    @api.model
    def _get_thread_with_access(self, thread_id, mode="read", **kwargs):
        if project_sharing_id := kwargs.get("project_sharing_id"):
            if token := ProjectSharingChatter._check_project_access_and_get_token(
                self, project_sharing_id, self._name, thread_id, kwargs.get("token")
            ):
                kwargs["token"] = token
        return super()._get_thread_with_access(thread_id, mode, **kwargs)

    def get_mention_suggestions(self, search, limit=8):
        """Return the 'limit'-first followers of the given task or followers of its project matching
        a 'search' string as a list of partner data (returned by `_to_store()`).
        See similar method for all partners `get_mention_suggestions()`.
        """
        self.ensure_one()
        project = self.project_id
        if not (
            project
            and project._check_project_sharing_access()
            and project._get_thread_with_access(project.id)
        ):
            return {}
        # sudo: mail.followers - reading message_follower_ids on accessible task/project is allowed
        followers = project.sudo().message_follower_ids | self.sudo().message_follower_ids
        domain = expression.AND([
            self.env["res.partner"]._get_mention_suggestions_domain(search),
            [("id", "in", followers.partner_id.ids)],
        ])
        partners = self.env["res.partner"].sudo()._search_mention_suggestions(domain, limit)
        return Store(partners).get_result()

```

## File: models\project_task_recurrence.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
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
    ], default='week', export_string_translation=False)
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
            'recurrence_id',
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
        if (
            self.repeat_type != 'until' or not occurrence_from.date_deadline or
            self.repeat_until and (occurrence_from.date_deadline + self._get_recurrence_delta()).date() <= self.repeat_until
        ):
            occurrence_from.with_context(copy_project=True).sudo().copy(
                self._create_next_occurrence_values(occurrence_from)
            )

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

        create_values['priority'] = '0'
        create_values['stage_id'] = occurrence_from.project_id.type_ids[0].id if occurrence_from.project_id.type_ids else occurrence_from.stage_id.id
        create_values['child_ids'] = [
            child.with_context(copy_project=True).sudo().copy(self._create_next_occurrence_values(child)).id for child in occurrence_from.with_context(active_test=False).child_ids
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

    task_id = fields.Many2one('project.task', required=True, ondelete='cascade', index=True, export_string_translation=False)
    user_id = fields.Many2one('res.users', required=True, ondelete='cascade', index=True, export_string_translation=False)
    stage_id = fields.Many2one('project.task.type', domain="[('user_id', '=', user_id)]", ondelete='set null', export_string_translation=False)

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

    active = fields.Boolean('Active', default=True, export_string_translation=False)
    name = fields.Char(string='Name', required=True, translate=True)
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
    disabled_rating_warning = fields.Text(compute='_compute_disabled_rating_warning', export_string_translation=False)

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

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, name=self.env._("%s (copy)", task_type.name)) for task_type, vals in zip(self, vals_list)]

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
from odoo.tools import format_amount, formatLang

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
    ], required=True, tracking=True, export_string_translation=False)
    color = fields.Integer(compute='_compute_color', export_string_translation=False)
    progress = fields.Integer(tracking=True)
    progress_percentage = fields.Float(compute='_compute_progress_percentage', export_string_translation=False)
    user_id = fields.Many2one('res.users', string='Author', required=True, default=lambda self: self.env.user)
    description = fields.Html()
    date = fields.Date(default=fields.Date.context_today, tracking=True)
    project_id = fields.Many2one('project.project', required=True, export_string_translation=False)
    name_cropped = fields.Char(compute="_compute_name_cropped", export_string_translation=False)
    task_count = fields.Integer("Task Count", readonly=True, export_string_translation=False)
    closed_task_count = fields.Integer("Closed Task Count", readonly=True, export_string_translation=False)
    closed_task_percentage = fields.Integer("Closed Task Percentage", compute="_compute_closed_task_percentage", export_string_translation=False)

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
                "closed_task_count": project.task_count - project.open_task_count,
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
            'format_monetary': lambda value: format_amount(self.env, value, project.currency_id),
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
    group_project_recurring_tasks = fields.Boolean("Recurring Tasks", implied_group="project.group_project_recurring_tasks", group='base.group_portal,base.group_user')
    group_project_task_dependencies = fields.Boolean("Task Dependencies", implied_group="project.group_project_task_dependencies")
    group_project_milestone = fields.Boolean('Milestones', implied_group='project.group_project_milestone', group='base.group_portal,base.group_user')

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
            if self.env.user.has_group(config_flag_global) != config_feature_enabled:
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

    project_ids = fields.One2many('project.project', 'partner_id', string='Projects', export_string_translation=False)
    task_ids = fields.One2many('project.task', 'partner_id', string='Tasks', export_string_translation=False)
    task_count = fields.Integer(compute='_compute_task_count', string='# Tasks', export_string_translation=False)

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

    def action_view_tasks(self):
        self.ensure_one()
        action = {
            **self.env["ir.actions.actions"]._for_xml_id("project.project_task_action_from_partner"),
            'display_name': _("%(partner_name)s's Tasks", partner_name=self.name),
            'context': {
                'default_partner_id': self.id,
            },
        }
        all_child = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])
        search_domain = [('partner_id', 'in', (self | all_child).ids)]
        if self.task_count <= 1:
            task_id = self.env['project.task'].search(search_domain, limit=1)
            action['res_id'] = task_id.id
            action['views'] = [(view_id, view_type) for view_id, view_type in action['views'] if view_type == "form"]
        else:
            action['domain'] = search_domain
        return action

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
        digits=(16, 2), readonly=True, aggregator="avg")
    working_days_open = fields.Float(string='Working Days to Assign',
        digits=(16, 2), readonly=True, aggregator="avg")
    delay_endings_days = fields.Float(string='Days to Deadline', digits=(16, 2), aggregator="avg", readonly=True)
    nbr = fields.Integer('# of Tasks', readonly=True)  # TDE FIXME master: rename into nbr_tasks
    working_hours_open = fields.Float(string='Working Hours to Assign', digits=(16, 2), readonly=True, aggregator="avg")
    working_hours_close = fields.Float(string='Working Hours to Close', digits=(16, 2), readonly=True, aggregator="avg")
    rating_last_value = fields.Float('Last Rating (1-5)', aggregator="avg", readonly=True, groups="project.group_project_rating")
    rating_avg = fields.Float('Average Rating (1-5)', readonly=True, aggregator='avg', groups="project.group_project_rating")
    priority = fields.Selection([
        ('0', 'Low'),
        ('1', 'High')
        ], readonly=True, string="Priority")

    state = fields.Selection([
        ('01_in_progress', 'In Progress'),
        ('1_done', 'Done'),
        ('04_waiting_normal', 'Waiting'),
        ('03_approved', 'Approved'),
        ('1_canceled', 'Cancelled'),
        ('02_changes_requested', 'Changes Requested'),
    ], string='State', readonly=True)
    is_closed = fields.Boolean(string='Closed state', readonly=True)
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
                CASE WHEN t.state IN ('1_done', '1_canceled') THEN True ELSE False END AS is_closed,
                CASE WHEN pm.id IS NOT NULL THEN true ELSE false END as has_late_and_unreached_milestone,
                t.description,
                NULLIF(t.rating_last_value, 0) as rating_last_value,
                AVG(rt.rating) as rating_avg,
                NULLIF(t.working_days_close, 0) as Working_days_close,
                NULLIF(t.working_days_open, 0) as working_days_open,
                NULLIF(t.working_hours_open, 0) as working_hours_open,
                NULLIF(t.working_hours_close, 0) as working_hours_close,
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

        <!-- We need to create an fsm view here so that we can prevent the
        remaining_hours_so field from being automatically added to the fsm view
        when installing the sale_timesheet module -->
        <record id="view_task_project_user_fsm_pivot_base" model="ir.ui.view">
            <field name="name">report.project.task.user.pivot</field>
            <field name="model">report.project.task.user</field>
            <field name="inherit_id" ref="view_task_project_user_pivot"/>
            <field name="mode">primary</field>
            <field name="priority">999</field>
            <field name="arch" type="xml">
                <pivot position="inside"/>
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
    

        <!-- We need to create an fsm view here so that we can prevent the
        remaining_hours_so field from being automatically added to the fsm view
        when installing the sale_timesheet module -->
        <record id="view_task_project_user_fsm_graph_base" model="ir.ui.view">
            <field name="name">report.project.task.user.graph</field>
            <field name="model">report.project.task.user</field>
            <field name="inherit_id" ref="view_task_project_user_graph"/>
            <field name="mode">primary</field>
            <field name="priority">999</field>
            <field name="arch" type="xml">
                <graph position="inside"/>
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
            <field name="path">tasks-analysis</field>
            <field name="view_mode">graph,pivot</field>
            <field name="search_view_id" ref="view_task_project_user_search"/>
            <field name="context">{'group_by':[], 'graph_measure': '__count__'}</field>
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
        ('1_canceled', 'Cancelled'),
        ('02_changes_requested', 'Changes Requested'),
    ], string='State', readonly=True)
    is_closed = fields.Selection([('closed', 'Closed tasks'), ('open', 'Open tasks')], string="Closing Stage", readonly=True)
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
        project_task_query = self.env['project.task']._where_calc(task_specific_domain)
        self.env.flush_query(project_task_query.subselect())

        # Get the stage_id `ir.model.fields`'s id in order to inject it directly in the query and avoid having to join
        # on `ir_model_fields` table.
        IrModelFieldsSudo = self.env['ir.model.fields'].sudo()
        field_id = IrModelFieldsSudo.search([('name', '=', 'stage_id'), ('model', '=', 'project.task')]).id

        groupby = self.env.context.get('project_task_burndown_chart_report_groupby', ['date:month', 'stage_id'])
        date_groupby = [g for g in groupby if g.startswith('date')][0]

        # Computes the interval which needs to be used in the `SQL` depending on the date group by interval.
        interval = date_groupby.split(':')[1]
        sql_interval = '1 %s' % interval if interval != 'quarter' else '3 month'

        simple_date_groupby_sql = self._read_group_groupby(f"date:{interval}", main_query)
        # Removing unexistant table name from the expression
        simple_date_groupby_sql = self.env.cr.mogrify(simple_date_groupby_sql).decode()
        simple_date_groupby_sql = simple_date_groupby_sql.replace('"project_task_burndown_chart_report".', '')

        burndown_chart_sql = SQL("""
            (
              WITH task_ids AS %(task_query_subselect)s,
              all_stage_task_moves AS (
                 SELECT count(*) as __count,
                        sum(allocated_hours) as allocated_hours,
                        project_id,
                        %(date_begin)s as date_begin,
                        %(date_end)s as date_end,
                        stage_id,
                        is_closed
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
                                   first_value(stage_id) OVER task_date_begin_window AS stage_id,
                                   is_closed
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
                                            END as stage_id,
                                            CASE
                                                WHEN mtv.id IS NOT NULL AND mtv.old_value_char IN ('1_done', '1_canceled') THEN 'closed'
                                                WHEN mtv.id IS NOT NULL AND mtv.old_value_char NOT IN ('1_done', '1_canceled') THEN 'open'
                                                WHEN mtv.id IS NULL AND pt.state IN ('1_done', '1_canceled') THEN 'closed'
                                                ELSE 'open'
                                            END as is_closed
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
                                   stage_id,
                                   is_closed
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
                                   pt.stage_id as old_value_integer,
                                   CASE WHEN pt.state IN ('1_done', '1_canceled') THEN 'closed'
                                       ELSE 'open'
                                   END as is_closed
                              FROM project_task pt
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
                        stage_id,
                        is_closed
              )
              SELECT (project_id*10^13 + stage_id*10^7 + to_char(date, 'YYMMDD')::integer)::bigint as id,
                     allocated_hours,
                     project_id,
                     stage_id,
                     is_closed,
                     date,
                     __count
                FROM all_stage_task_moves t
                         JOIN LATERAL generate_series(t.date_begin, t.date_end-INTERVAL '1 day', '%(interval)s')
                            AS date ON TRUE
            )
            """,
            task_query_subselect=project_task_query.subselect(),
            date_begin=SQL(simple_date_groupby_sql.replace('"date"', '"date_begin"')),
            date_end=SQL(simple_date_groupby_sql.replace('"date"', '"date_end"')),
            interval=SQL(sql_interval),
            field_id=field_id,
        )

        # hardcode 'project_task_burndown_chart_report' as the query above
        # (with its own parameters)
        main_query._tables['project_task_burndown_chart_report'] = burndown_chart_sql

        return main_query

    @api.model
    def _validate_group_by(self, groupby):
        """ Check that the both `date` and `stage_id` are part of `group_by`, otherwise raise a `UserError`.

        :param groupby: List of group by fields.
        """

        is_closed_or_stage_in_groupby = False
        date_in_groupby = False
        for gb in groupby:
            if gb.startswith('date'):
                date_in_groupby = True
            elif gb in ['stage_id', 'is_closed']:
                is_closed_or_stage_in_groupby = True

        if not date_in_groupby or not is_closed_or_stage_in_groupby:
            raise UserError(_('The view must be grouped by date and by Stage - Burndown chart or Is Closed - Burnup chart'))

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
            return SQL("SUM(%s)", SQL.identifier(self._table, '__count'))
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
                <field name="is_closed"/>
                <field name="project_id" />
                <field name="milestone_id" groups="project.group_project_milestone"/>
                <field name="partner_id" filter_domain="[('partner_id', 'child_of', self)]"/>
                <separator/>
                <filter string="My Tasks" name="my_tasks" domain="[('user_ids', 'in', uid)]"/>
                <filter string="Unassigned" name="unassigned" domain="[('user_ids', '=', False)]"/>
                <separator/>
                <filter name="filter_date" date="date" string="Date" default_period="year,year-1" />
                <filter name="filter_last_stage_update" date="date_last_stage_update"/>
                <filter name="filter_date_deadline" date="date_deadline"/>
                <filter string="Last Month" invisible="1" name="last_month" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=30)).strftime('%Y-%m-%d'))]"/>
                <separator/>
                <filter string="Open Tasks" name="open_tasks" domain="[('is_closed', '=', 'open')]"/>
                <filter string="Closed Tasks" name="closed_tasks" domain="[('is_closed', '=', 'closed')]"/>
                <group expand="0" string="Group By">
                    <filter string="Date" name="date" context="{'group_by': 'date'}" />
                    <filter string="Stage (Burndown Chart)" name="stage" context="{'group_by': 'stage_id'}"/>
                    <filter string="Is Closed (Burn-up Chart)" name="is_closed" context="{'group_by': 'is_closed'}"/>
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
                <field name="is_closed"/>
            </graph>
        </field>
    </record>

    <record id="action_project_task_burndown_chart_report" model="ir.actions.act_window">
        <field name="name">Burndown Chart</field>
        <field name="res_model">project.task.burndown.chart.report</field>
        <field name="path">burndown-chart</field>
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
access_project_share_collaborator_manager,project.share.collaborator.wizard.manager,model_project_share_collaborator_wizard,project.group_project_manager,1,1,1,1
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
        <field name="description" />
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
        <field name="implied_ids" eval="[(4, ref('group_project_user')), (4, ref('mail.group_mail_canned_response_admin'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="group_project_rating" model="res.groups">
        <field name="name">Use Rating on Project</field>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
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
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
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

    <record model="ir.rule" id="task_visibility_rule_project_user">
        <field name="name">Project/Task: project users: follow required for follower-only projects</field>
        <field name="model_id" ref="model_project_task"/>
        <field name="perm_read" eval="False"/>
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
        <field name="groups" eval="[(4,ref('project.group_project_user'))]"/>
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
        <field name="name">Project/Task: portal users: can only see a task if he's a collaborator of the project and a follower of the task</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="domain_force">[
            ('project_id.privacy_visibility', '=', 'portal'),
            ('active', '=', True),
            '|',
                ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
                ('project_id.collaborator_ids', 'any', [
                    ('partner_id', '=', user.partner_id.id),
                    ('limited_access', '=', False),
                ]),
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
                ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),
                ('project_id.collaborator_ids', 'any', [
                    ('partner_id', '=', user.partner_id.id),
                    ('limited_access', '=', False),
                ]),
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

## File: static\src\components\task_list_renderer.js

```javascript
import { useService } from "@web/core/utils/hooks";
import { ListRenderer } from "@web/views/list/list_renderer";

import { useEffect } from "@odoo/owl";

export class TaskListRenderer extends ListRenderer {
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
            const col = this.columns.find((c) => c.name === "name");
            this.focusCell(col);
        }
    }
}

```

## File: static\src\components\notebook_task_one2many_field\notebook_task_list_renderer.js

```javascript
/** @odoo-module **/

import { useState } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";

import { TaskListRenderer } from "../task_list_renderer";

export class NotebookTaskListRenderer extends TaskListRenderer {
    static rowsTemplate = "project.NotebookTaskListRenderer.Rows";
    setup() {
        super.setup();
        this.hideState = useState({
            hide: localStorage.getItem(this._getStorageKey) === 'true',
        });
    }

    /**
     * @private
     * @returns {string}
     */
    get _getStorageKey() {
        return `hide_closed_${this.constructor.name}`;
    }

    get hideClosed() {
        return this.hideState.hide;
    }

    get closedX2MCount() {
        return this.props.list.context.closed_X2M_count;
    }

    get openLabel() {
        return typeof this.closedX2MCount === "undefined" ? _t("Show closed tasks") : _t("%s closed tasks", this.closedX2MCount);
    }

    get closeLabel() {
        return _t("Hide closed tasks");
    }

    get toggleListHideLabel() {
        return this.hideClosed ? this.openLabel : this.closeLabel;
    }

    get ShowX2MRecords() {
        // If there isn't a closed_X2M_count defined in the context of the x2m task in the view we are always displaying the Toggle button
        // In case there is no computed field to calculate the number of closed X2M tasks in the backend
        return this.closedX2MCount > 0 || typeof this.closedX2MCount === "undefined";
    }

    toggleHideClosed() {
        this.hideState.hide = !this.hideState.hide;
        localStorage.setItem(this._getStorageKey, this.hideState.hide);
        document.activeElement.blur();
    }
}

```

## File: static\src\components\notebook_task_one2many_field\notebook_task_list_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="project.NotebookTaskListRenderer.Rows" t-inherit="web.ListRenderer.Rows" t-inherit-mode="primary">
        <xpath expr="//t[@t-foreach='list.records']" position="replace">
            <t t-foreach="list.records" t-as="record" t-key="record.id">
                <t t-if="!['1_done','1_canceled'].includes(record.data.state) or !hideState.hide">
                    <t t-call="{{ constructor.recordRowTemplate }}"/>
                </t>
            </t>
        </xpath>

        <xpath expr="//t[@t-foreach='creates']" position="after">
            <a t-if="ShowX2MRecords" role="button" class="ml16 o_toggle_closed_task_button" href="#" t-on-click="() => this.toggleHideClosed()">
                <t t-esc="toggleListHideLabel"/>
            </a>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\notebook_task_one2many_field\notebook_task_one2many_field.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { X2ManyField, x2ManyField } from '@web/views/fields/x2many/x2many_field';

import { NotebookTaskListRenderer } from './notebook_task_list_renderer';

export class NotebookTaskOne2ManyField extends X2ManyField {
    static components = {
        ...X2ManyField.components,
        ListRenderer: NotebookTaskListRenderer,
    };
}

export const notebookTaskOne2ManyField = {
    ...x2ManyField,
    component: NotebookTaskOne2ManyField,
}

registry.category("fields").add("notebook_task_one2many", notebookTaskOne2ManyField);

```

## File: static\src\components\project_is_favorite\project_is_favorite_field.js

```javascript
import { registry } from "@web/core/registry";
import { booleanFavoriteField } from "@web/views/fields/boolean_favorite/boolean_favorite_field";

export const projectIsFavoriteField = {
    ...booleanFavoriteField,
    extractProps: (fieldsInfo, dynamicInfo) => {
        return {
            ...booleanFavoriteField.extractProps(fieldsInfo, dynamicInfo),
            readonly: Boolean(fieldsInfo.attrs.readonly),
        };
    },
};

registry.category("fields").add("project_is_favorite", projectIsFavoriteField);

```

## File: static\src\components\project_many2one_field\project_many2one_field.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from '@web/core/registry';
import { Many2OneField, many2OneField } from '@web/views/fields/many2one/many2one_field';

export class ProjectMany2OneField extends Many2OneField {
    static template = "project.ProjectMany2OneField";
    get Many2XAutocompleteProps() {
        const props = super.Many2XAutocompleteProps;
        const { record } = this.props;
        if (!record.data.project_id && !record._isRequired("project_id")) {
            props.placeholder = _t("Private");
        }
        return props;
    }
}

export const projectMany2OneField = {
    ...many2OneField,
    component: ProjectMany2OneField,
    fieldDependencies: [
        ...(many2OneField.fieldDependencies || []),
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
import { _t } from "@web/core/l10n/translation";
import { useBus, useService } from "@web/core/utils/hooks";
import { formatFloat } from "@web/views/fields/formatters";
import { ViewButton } from '@web/views/view_button/view_button';
import { FormViewDialog } from '@web/views/view_dialogs/form_view_dialog';

import { ProjectRightSidePanelSection } from './components/project_right_side_panel_section';
import { ProjectMilestone } from './components/project_milestone';
import { ProjectProfitability } from './components/project_profitability';
import { getCurrency } from '@web/core/currency';
import { Component, onWillStart, useState } from "@odoo/owl";
import { SIZES } from "@web/core/ui/ui_service";

export class ProjectRightSidePanel extends Component {
    static components = {
        ProjectRightSidePanelSection,
        ProjectMilestone,
        ViewButton,
        ProjectProfitability,
    };
    static template = "project.ProjectRightSidePanel";
    static props = {
        context: Object,
        domain: Array,
    };

    setup() {
        this.orm = useService('orm');
        this.actionService = useService('action');
        this.dialog = useService('dialog');
        this.uiService = useService("ui");
        useBus(this.uiService.bus, "resize", this.updateGridTemplateColumns)
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
            },
            gridTemplateColumns: this._getGridTemplateColumns(),
        });

        onWillStart(() => this.loadData());
    }

    _getGridTemplateColumns() {
        switch (this.uiService.size) {
            case SIZES.XS:
                return 2;
            case SIZES.VSM:
                return 3;
            case SIZES.XXL:
                return 6;
            default:
                return 4;
        }
    }

    updateGridTemplateColumns() {
        this.state.gridTemplateColumns = this._getGridTemplateColumns();
    }

    get panelVisible() {
        return this.state.data.show_milestones || this.state.data.show_project_profitability_helper;
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
            context: this.context,
            resModel: 'project.project',
        };
    }
}

```

## File: static\src\components\project_right_side_panel\project_right_side_panel.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="project.ProjectRightSidePanel">
        <div t-if="panelVisible" class="o_rightpanel ps-0 pt-0 bg-view border-start border-bottom overflow-auto">
            <ProjectRightSidePanelSection
                name="'stat_buttons'"
                header="false"
                show="!!state.data.buttons"
                dataClassName="'ps-3'"
            >
                <div class="o_form_view">
                    <div
                        class="oe_button_box o-form-buttonbox d-print-none d-grid"
                        t-attf-style="grid-template-columns: repeat({{ state.gridTemplateColumns }}, 1fr);"
                    >
                        <t t-foreach="state.data.buttons" t-as="button" t-key="button.sequence">
                            <ViewButton
                                t-if="button.show"
                                defaultRank="'oe_stat_button'"
                                className="'h-auto py-2 border border-start-0 border-top-0 text-start rounded-0'"
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
                <div t-if="state.data.milestones.data.length === 0" class="ps-3">
                    <span class="text-muted fst-italic">
                        Track major progress points that must be reached to achieve success.
                    </span>
                </div>
            </ProjectRightSidePanelSection>
            <ProjectRightSidePanelSection
                name="'profitability'"
                show="state.data.show_project_profitability_helper"
            >
                <t t-set-slot="title">
                    Profitability
                </t>
                <ProjectProfitability
                    t-if="showProjectProfitability"
                    data="state.data.profitability_items"
                    labels="state.data.profitability_labels"
                    formatMonetary="formatMonetary.bind(this)"
                    onProjectActionClick="onProjectActionClick.bind(this)"
                    onClick="(params) => this.onProjectActionClick(params)"
                />
                <div t-elif="state.data.show_project_profitability_helper" class="ps-3">
                    <span class="text-muted fst-italic">
                        Track project costs, revenues, and margin by setting the analytic account associated with the project on relevant documents.
                    </span>
                </div>
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
    static props = {
        context: Object,
        milestone: Object,
        open: Function,
        load: Function,
    };
    static template = "project.ProjectMilestone";

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

```

## File: static\src\components\project_right_side_panel\components\project_milestone.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="project.ProjectMilestone">
        <div class="list-group mb-2">
            <div class="o_rightpanel_milestone list-group-item list-group-item-action d-flex justify-content-evenly px-0 cursor-pointer" t-att-class="state.colorClass">
                <span t-on-click="toggleIsReached">
                    <t t-if="milestone.is_reached" t-set="title">Mark as incomplete</t>
                    <t t-else="" t-set="title">Mark as reached</t>
                    <i class="fa position-absolute pt-1" t-att-class="state.checkboxIcon" t-att-title="title"/>
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

    static props = {
        data: Object,
        labels: Object,
        formatMonetary: Function,
        onProjectActionClick: Function,
        onClick: Function,
    };
    static template = "project.ProjectProfitability";

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
                        <th class="text-end">Expected</th>
                        <th class="text-end">To Invoice</th>
                        <th class="text-end">Invoiced</th>
                    </tr>
                </thead>
                <tbody>
                    <t t-foreach="revenues.data" t-as="revenue" t-key="revenue.id" t-if="revenue.invoiced !== 0 || revenue.to_invoice !== 0">
                        <tr class="revenue_section">
                            <t t-set="revenue_label" t-value="props.labels[revenue.id] or revenue.id"/>
                            <td class="align-middle">
                                <a class="revenue_section" t-if="revenue.action" href="#"
                                    t-on-click="() => this.props.onClick(revenue.action)"
                                >
                                    <t t-esc="revenue_label"/>
                                </a>
                                <t t-esc="revenue_label" t-else=""/>
                            </td>
                            <td t-attf-class="text-end align-middle {{ revenue.invoiced + revenue.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.invoiced + revenue.to_invoice)"/></td>
                            <td t-attf-class="text-end align-middle {{ revenue.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.to_invoice)"/></td>
                            <td t-attf-class="text-end align-middle {{ revenue.invoiced === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.invoiced)"/></td>
                        </tr>
                    </t>
                </tbody>
                <tfoot>
                    <tr class="fw-bolder">
                        <td>Total Revenues</td>
                        <td t-attf-class="text-end {{ revenues.total.invoiced + revenues.total.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenues.total.invoiced + revenues.total.to_invoice)"/></td>
                        <td t-attf-class="text-end {{ revenues.total.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenues.total.to_invoice)"/></td>
                        <td t-attf-class="text-end {{ revenues.total.invoiced === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenues.total.invoiced)"/></td>
                    </tr>
                </tfoot>
            </table>
        </div>
        <div class="o_rightpanel_subsection pb-3 border-bottom" t-if="costs.data.length">
            <table class="table table-sm table-striped table-hover mb-0">
                <thead class="bg-100">
                    <tr>
                        <th>Costs</th>
                        <th class="text-end">Expected</th>
                        <th class="text-end">To Bill</th>
                        <th class="text-end">Billed</th>
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
                        <td t-attf-class="text-end align-middle {{ cost.billed + cost.to_bill === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(cost.billed + cost.to_bill)"/></td>
                        <td t-attf-class="text-end align-middle {{ cost.to_bill === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(cost.to_bill)"/></td>
                        <td t-attf-class="text-end align-middle {{ cost.billed === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(cost.billed)"/></td>
                    </tr>
                </tbody>
                <tfoot>
                    <tr class="fw-bolder">
                        <td>Total Costs</td>
                        <td t-attf-class="text-end {{ costs.total.billed + costs.total.to_bill  === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(costs.total.billed + costs.total.to_bill)"/></td>
                        <td t-attf-class="text-end {{ costs.total.to_bill === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(costs.total.to_bill)"/></td>
                        <td t-attf-class="text-end {{ costs.total.billed === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(costs.total.billed)"/></td>
                    </tr>
                </tfoot>
            </table>
        </div>
        <div class="o_rightpanel_subsection" t-if="revenues.data.length &amp;&amp; costs.data.length">
            <table class="table table-sm table-borderless w-100 mb-0">
                <thead>
                    <tr class="align-top">
                        <th>Total</th>
                        <th class="text-end" t-att-class="margin.total &lt; 0 ? 'text-danger' : 'text-success'">
                            <t t-set="total_revenue" t-value="revenues.total.to_invoice + revenues.total.invoiced"/>
                            <t t-out="props.formatMonetary(margin.total)"/><br/>
                            <t t-if="total_revenue != 0">
                                <t t-out="margin.total > 0 ? '+' : ''"/><t t-out="(margin.total / total_revenue * 100).toFixed()"/>%
                            </t>
                        </th>
                        <th class="text-end" t-att-class="margin.to_invoice_to_bill &lt; 0 ? 'text-danger' : 'text-success'">
                            <t t-out="props.formatMonetary(margin.to_invoice_to_bill)"/><br/>
                            <t t-if="revenues.total.to_invoice != 0">
                                <t t-out="margin.to_invoice_to_bill > 0 ? '+' : ''"/><t t-out="(margin.to_invoice_to_bill / revenues.total.to_invoice * 100).toFixed()"/>%
                            </t>
                        </th>
                        <th class="text-end" t-att-class="margin.invoiced_billed &lt; 0 ? 'text-danger' : 'text-success'">
                            <t t-out="props.formatMonetary(margin.invoiced_billed)"/><br/>
                            <t t-if="revenues.total.invoiced != 0">
                                <t t-out="margin.invoiced_billed > 0 ? '+' : ''"/><t t-out="(margin.invoiced_billed / revenues.total.invoiced * 100).toFixed()"/>%
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

export class ProjectRightSidePanelSection extends Component {
    static props = {
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
    static defaultProps = {
        header: true,
        showData: true,
    };

    static template = "project.ProjectRightSidePanelSection";
}

```

## File: static\src\components\project_right_side_panel\components\project_right_side_panel_section.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="project.ProjectRightSidePanelSection">
        <div class="o_rightpanel_section py-0" t-att-name="props.name" t-if="props.show">
            <div class="o_rightpanel_header d-flex align-items-center justify-content-between py-2 border-bottom" t-att-class="props.headerClassName" t-if="props.header">
                <div class="o_rightpanel_title d-flex flex-row-reverse align-items-center" t-if="props.slots.title">
                    <h3 class="m-0 lh-lg ps-1"><t t-slot="title"/></h3>
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
    static props = {
        ...SelectionField.props,
        statusLabel: { type: String, optional: true },
        hideStatusName: { type: Boolean, optional: true },
    };

    static template = "project.ProjectStatusWithColorSelectionField";

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

export const projectStatusWithColorSelectionField = {
    ...selectionField,
    component: ProjectStatusWithColorSelectionField,
    extractProps: (fieldInfo, dynamicInfo) => {
        const props = selectionField.extractProps(fieldInfo, dynamicInfo);
        props.statusLabel = fieldInfo.attrs.status_label;
        props.hideStatusName = Boolean(fieldInfo.attrs.hideStatusName);
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
                <span t-attf-class="o_status {{ statusColor(currentValue) }} d-inline-block"/>
                <div class="ps-2">
                    <div class="o_stat_text" t-if="this.props.statusLabel" t-out="this.props.statusLabel"/>
                    <div class="o_stat_value" t-out="string" t-if="!this.props.hideStatusName" t-att-raw-value="value"/>
                    <div class="o_stat_value" t-else="">
                        <span><t t-out="this.props.record.data['update_count']"/> Status</span>
                    </div>
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
    fieldDependencies: [
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
    static template = "project.ProjectTaskStateSelection";

    static props = {
        ...stateSelectionField.component.props,
        isToggleMode: { type: Boolean, optional: true },
        viewType: { type: String },
    };

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
            "04_waiting_normal": "btn-outline-info",
        };
        this.colorButton = {
            "01_in_progress": "btn-outline-secondary",
            "03_approved": "btn-outline-success",
            "02_changes_requested": "btn-outline-warning",
            "1_done": "btn-outline-success",
            "1_canceled": "btn-outline-danger",
            "04_waiting_normal": "btn-outline-info",
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
                    <i class="fa fa-lg fa-hourglass-o text-info"></i>
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
        <xpath expr="//Dropdown/button/div" position="replace">
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
            <!-- <attribute name="class">toggle_dropdown</attribute> -->
            <attribute name="position">`${ getDropdownPosition() }`</attribute>
            <attribute name="menuClass" separator=" + " add="' project_task_state_selection_menu'"></attribute>
        </xpath>
        <xpath expr="//Dropdown/button" position="attributes">
            <attribute name="tooltip">''</attribute>
            <attribute name="class" remove="p-0" add="shadow-none" separator=" "/>
            <attribute name="t-att-class">getTogglerClass(currentValue)</attribute>
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

## File: static\src\components\project_task_state_selection\project_task_stage_state_selection\project_task_stage_with_state_selection.js

```javascript
import { Component } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { omit } from "@web/core/utils/objects";
import { Many2OneField } from "@web/views/fields/many2one/many2one_field";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

import { ProjectTaskStateSelection } from "../project_task_state_selection";

export class TaskStageWithStateSelection extends Component {
    static template = "project.TaskStageWithStateSelection";

    static props = {
        ...standardFieldProps,
        stateReadonly: { type: Boolean, optional: true },
        viewType: { type: String },
    };

    static components = {
        ProjectTaskStateSelection,
        Many2OneField,
    };

    get stageProps() {
        return omit(this.props, "stateReadonly", "viewType");
    }

    get stateProps() {
        return {
            ...this.stageProps,
            name: "state",
            readonly: this.props.stateReadonly,
            viewType: this.props.viewType,
            showLabel: false,
        };
    }
}

export const taskStageWithStateSelection = {
    component: TaskStageWithStateSelection,
    supportedOptions: [
        {
            label: _t("State readonly"),
            name: "state_readonly",
            type: "boolean",
            default: true,
        },
    ],
    fieldDependencies: [{ name: "state", type: "selection" }],
    supportedTypes: ["many2one"],
    extractProps({ options, viewType }) {
        return {
            stateReadonly: "state_readonly" in options ? options.state_readonly : true,
            viewType: viewType,
        };
    },
};

registry.category("fields").add("task_stage_with_state_selection", taskStageWithStateSelection);

```

## File: static\src\components\project_task_state_selection\project_task_stage_state_selection\project_task_stage_with_state_selection.xml

```xml
<templates>

    <t t-name="project.TaskStageWithStateSelection">
        <div class="d-flex gap-1">
            <Many2OneField t-props="stageProps"/>
            <ProjectTaskStateSelection t-props="stateProps"/>
        </div>
    </t>

</templates>

```

## File: static\src\components\subtask_kanban_list\subtask_kanban_list.js

```javascript
/** @odoo-module */

import { Component, useState } from "@odoo/owl";

import { useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";

import { Field, getPropertyFieldInfo } from "@web/views/fields/field";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";
import { SubtaskCreate } from "./subtask_kanban_create/subtask_kanban_create";

export class SubtaskKanbanList extends Component {
    static components = {
        Field,
        SubtaskCreate,
    };
    static props = {
        ...standardWidgetProps,
        isReadonly: {
            type: Boolean,
            optional: true,
        },
    };
    static template = "project.SubtaskKanbanList";

    setup() {
        this.actionService = useService("action");
        this.orm = useService("orm");
        this.subtaskCreate = useState({
            open: false,
            name: "",
        });
    }

    get list() {
        return this.props.record.data.child_ids;
    }

    get closedList() {
        return this.list.records.filter((child) => {
            return !["1_done", "1_canceled"].includes(child.data.state);
        });
    }

    get fieldInfo() {
        return {
            state: {
                ...getPropertyFieldInfo({ name: "state", type: "project_task_state_selection" }),
                viewType: "kanban",
            },
        };
    }

    async goToSubtask(subtask_id) {
        return this.actionService.doAction({
            type: "ir.actions.act_window",
            res_model: this.list.resModel,
            res_id: subtask_id,
            views: [[false, "form"]],
            target: "current",
            context: {
                active_id: subtask_id,
            },
        });
    }

    async onSubTaskCreated(ev) {
        this.subtaskCreate.open = true;
    }

    async _onBlur() {
        this.subtaskCreate.open = false;
    }

    async _onSubtaskCreateNameChanged(name) {
        await this.orm.create("project.task", [{
            display_name: name,
            parent_id: this.props.record.resId,
            project_id: this.props.record.data.project_id[0],
            user_ids: this.props.record.data.user_ids.resIds,
        }]);
        this.subtaskCreate.open = false;
        this.subtaskCreate.name = "";
        this.props.record.load();
    }
}

const subtaskKanbanList = {
    component: SubtaskKanbanList,
};

registry.category("view_widgets").add("subtask_kanban_list", subtaskKanbanList);

```

## File: static\src\components\subtask_kanban_list\subtask_kanban_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<templates>

    <t t-name="project.SubtaskKanbanList">
        <t t-if="props.record.displaySubtasks">
            <div class="subtask_list">
                <div t-foreach="closedList" t-as="record" t-key="record.id"  class="subtask_list_row">
                    <a t-attf-class="subtask_name_col {{ ['1_done', '1_canceled'].includes(record.data.state) ? 'text-muted opacity-50' : '' }}"
                        t-att-title="record.data.display_name"
                        t-esc="record.data.display_name"
                        t-on-click.prevent="() => this.goToSubtask(record.resId)"/>
                    <Field
                        class="`o_field_many2many_avatar_user subtask_user_widget_col d-inline-flex align-items-center justify-content-end me-1 ${['1_done', '1_canceled'].includes(record.data.state) ? 'opacity-50' : ''}`"
                        name="'user_ids'"
                        record="record"
                        readonly="false"
                        isEditable="true"
                        type="'kanban.many2many_avatar_user'"/>
                    <Field name="'state'"
                        class="`subtask_state_widget_col d-flex justify-content-center align-items-center`"
                        record="record"
                        type="'project_task_state_selection'"
                        fieldInfo="fieldInfo.state"/>
                </div>
                <div class="subtask_create" t-on-click.stop="(ev) => this.onSubTaskCreated(ev)">
                    <t t-if="subtaskCreate.open">
                        <SubtaskCreate
                            name="subtaskCreate.name"
                            isReadonly="props.isReadonly" 
                            onSubtaskCreateNameChanged.bind="_onSubtaskCreateNameChanged"
                            onBlur.bind="_onBlur" />
                    </t>
                    <t t-else="">
                        <i class="fa fa-plus my-2"/> New
                    </t>
                </div>
            </div>
        </t>
    </t>

</templates>

```

## File: static\src\components\subtask_kanban_list\subtask_kanban_create\subtask_kanban_create.js

```javascript
/** @odoo-module **/

import { Component, useState, useRef } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";
import { useAutofocus } from "@web/core/utils/hooks";

export class SubtaskCreate extends Component {
    static template = "project.SubtaskCreate";
    static props = {
        name: String,
        isReadonly: { type: Boolean, optional: true },
        onSubtaskCreateNameChanged: { type: Function },
        onBlur: { type: Function },
    };
    setup() {
        this.placeholder = _t("Add Sub-tasks");
        this.state = useState({
            inputSize: 1,
            name: this.props.name,
        });
        this.input = useRef("subtaskCreateInput");
        useAutofocus({ refName: "subtaskCreateInput" });
    }

    /**
     * @private
     * @param {InputEvent} ev
     */
    _onFocus(ev) {
        ev.target.value = this.placeholder;
        ev.target.select();
    }

    /**
     * @private
     * @param {InputEvent} ev
     */
    _onInput(ev) {
        const value = ev.target.value;
        this.state.name = value;
    }

    _onClick() {
        this.input.el.focus();
    }

    async _onBlur() {
        this.props.onBlur();
    }

    /**
     * @private
     * @param {InputEvent} ev
     */
    _onNameChanged(ev) {
        const value = ev.target.value.trim();
        this.props.onSubtaskCreateNameChanged(value);
        ev.target.blur();
    }

    _onSaveClick() {
        if (this.input.el.value !== "") {
            this.props.onSubtaskCreateNameChanged(this.input.el.value);
        }
    }
}

```

## File: static\src\components\subtask_kanban_list\subtask_kanban_create\subtask_kanban_create.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <div t-name="project.SubtaskCreate" class="subtask_create_input d-flex" owl="1">
        <input type="text" title="Rename" 
            class='border-0 border-bottom py-1'
            t-ref='subtaskCreateInput'
            t-att-value="state.name"
            t-att-placeholder="placeholder"
            t-on-input="_onInput"
            t-on-click="_onClick"
            t-on-blur="_onBlur"
            t-on-change="_onNameChanged"
            t-att-disabled="props.isReadonly"/>
        <button t-on-click="_onSaveClick" class="ms-auto fw-bold btn btn-link py-0 px-2">
            SAVE
        </button>
    </div>
</templates>

```

## File: static\src\components\subtask_one2many_field\subtask_list_renderer.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { NotebookTaskListRenderer } from '../notebook_task_one2many_field/notebook_task_list_renderer';

export class SubtaskListRenderer extends NotebookTaskListRenderer {
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

export class SubtaskOne2ManyField extends X2ManyField {
    static components = {
        ...X2ManyField.components,
        ListRenderer: SubtaskListRenderer,
    };
}

export const subtaskOne2ManyField = {
    ...x2ManyField,
    component: SubtaskOne2ManyField,
    additionalClasses: ["o_field_one2many"],
}

registry.category("fields").add("subtasks_one2many", subtaskOne2ManyField);

```

## File: static\src\core\web\follower_list_patch.js

```javascript
import { FollowerList } from "@mail/core/web/follower_list";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";

import { patch } from "@web/core/utils/patch";

const followerListPatch = {
    setup() {
        super.setup();
        this.dialogService = useService("dialog");
    },
    /**
     * @param {MouseEvent} ev
     * @param {import("models").Follower} follower
     */
    async onClickRemove(ev, follower) {
        if (follower.partner.in(follower.thread.collaborator_ids)) {
            this.dialogService.add(ConfirmationDialog, {
                title: _t("Remove Collaborator"),
                body: _t(
                    "This follower is currently a project collaborator. Removing them will revoke their portal access to the project. Are you sure you want to proceed?"
                ),
                confirmLabel: _t("Remove Collaborator"),
                cancelLabel: _t("Discard"),
                confirm: () => super.onClickRemove(ev, follower),
                cancel: () => {},
            });
        } else {
            super.onClickRemove(ev, follower);
        }
    },
};
patch(FollowerList.prototype, followerListPatch);

```

## File: static\src\core\web\thread_model_patch.js

```javascript
import { Record } from "@mail/core/common/record";
import { Thread } from "@mail/core/common/thread_model";

import { patch } from "@web/core/utils/patch";

/** @type {import("models").Thread} */
const threadPatch = {
    setup() {
        super.setup();
        this.collaborator_ids = Record.many("Persona");
    },
};
patch(Thread.prototype, threadPatch);

```

## File: static\src\core\web\@types\models.d.ts

```ts
declare module "models" {
    export interface Thread {
        collaborator_ids: Persona[],
    }
}

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
    url: "/odoo",
    steps: () => [stepUtils.showAppsMenuItem(), {
    isActive: ["community"],
    trigger: '.o_app[data-menu-xmlid="project.menu_main_pm"]',
    content: markup(_t('Want a better way to <b>manage your projects</b>? <i>It starts here.</i>')),
    tooltipPosition: 'right',
    run: "click",
}, {
    isActive: ["enterprise"],
    trigger: '.o_app[data-menu-xmlid="project.menu_main_pm"]',
    content: markup(_t('Want a better way to <b>manage your projects</b>? <i>It starts here.</i>')),
    tooltipPosition: 'bottom',
    run: "click",
},
{
    trigger: ".o_project_kanban",
},
{
    trigger: '.o-kanban-button-new',
    content: markup(_t('Let\'s create your first <b>project</b>.')),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: '.o_project_name input',
    content: markup(_t('Choose a <b>name</b> for your project. <i>It can be anything you want: the name of a customer, of a product, of a team, of a construction site, etc.</i>')),
    tooltipPosition: 'right',
    run: "edit Test",
}, {
    trigger: '.o_open_tasks',
    content: markup(_t('Let\'s create your first <b>project</b>.')),
    tooltipPosition: 'top',
    run: "click .modal:visible .btn.btn-primary",
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_header input",
    content: markup(_t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>.")),
    tooltipPosition: 'bottom',
    run: "edit Test",
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    content: markup(_t('Let\'s create your first <b>stage</b>.')),
    tooltipPosition: 'right',
    run: "click",
},
{
    trigger: ".o_kanban_group",
},
{
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_header input",
    content: markup(_t("Add columns to organize your tasks into <b>stages</b> <i>e.g. New - In Progress - Done</i>.")),
    tooltipPosition: 'bottom',
    run: "edit Test",
}, {
    trigger: ".o_kanban_project_tasks .o_column_quick_create .o_kanban_add",
    content: markup(_t('Let\'s create your second <b>stage</b>.')),
    tooltipPosition: 'right',
    run: "click",
},
{
    trigger: ".o_kanban_group:eq(1)",
},
{
    trigger: '.o-kanban-button-new',
    content: markup(_t("Let's create your first <b>task</b>.")),
    tooltipPosition: 'bottom',
    run: "click",
},
{
    trigger: ".o_kanban_project_tasks",
},
{
    trigger: '.o_kanban_quick_create div.o_field_char[name=display_name] input',
    content: markup(_t('Choose a task <b>name</b> <i>(e.g. Website Design, Purchase Goods...)</i>')),
    tooltipPosition: 'right',
    run: "edit Test",
},
{
    trigger: ".o_kanban_project_tasks",
},
{
    trigger: '.o_kanban_quick_create .o_kanban_add',
    content: _t("Add your task once it is ready."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_kanban_project_tasks",
},
{
    trigger: ".o_kanban_record",
    content: markup(_t("<b>Drag &amp; drop</b> the card to change your task from stage.")),
    tooltipPosition: "bottom",
    run: "drag_and_drop(.o_kanban_group:eq(1))",
},
{
    trigger: ".o_kanban_project_tasks",
},
{
    trigger: ".o_kanban_record:first",
    content: _t("Let's start working on your task."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_form_project_tasks",
},
{
    trigger: ".o-mail-Chatter-topbar button.o-mail-Chatter-sendMessage",
    content: markup(_t("Use the chatter to <b>send emails</b> and communicate efficiently with your customers. Add new people to the followers' list to make them aware of the main changes about this task.")),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_form_project_tasks",
},
{
    trigger: "button.o-mail-Chatter-logNote",
    content: markup(_t("<b>Log notes</b> for internal communications <i>(the people following this task won't be notified of the note you are logging unless you specifically tag them)</i>. Use @ <b>mentions</b> to ping a colleague or # <b>mentions</b> to reach an entire team.")),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_form_project_tasks",
},
{
    trigger: ".o-mail-Chatter-topbar button.o-mail-Chatter-activity",
    content: markup(_t("Create <b>activities</b> to set yourself to-dos or to schedule meetings.")),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_form_project_tasks",
},
{
    trigger: ".modal-dialog .btn-primary",
    content: _t("Schedule your activity once it is ready."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_form_project_tasks",
},
{
    isActive: ["auto"],
    trigger: ".o_field_widget[name='user_ids'] input",
    content: _t("Assign a responsible to your task"),
    tooltipPosition: "right",
    run: "edit Admin",
},
{
    isActive: ["manual"],
    trigger: ".o_field_widget[name='user_ids']",
    content: _t("Assign a responsible to your task"),
    tooltipPosition: "right",
    run: "click",
},
{
    isActive: ["desktop", "auto"],
    trigger: "a.dropdown-item[id*='user_ids'] span",
    content: _t("Select an assignee from the menu"),
    run: "click",
},
{
    isActive: ["mobile"],
    trigger: "div.o_kanban_renderer > article.o_kanban_record",
    run: "click",
}, {
    isActive: ["auto"],
    trigger: 'a[name="sub_tasks_page"]',
    content: _t('Open sub-tasks notebook section'),
    run: 'click',
}, {
    isActive: ["auto"],
    trigger: '.o_field_subtasks_one2many .o_list_renderer a[role="button"]',
    content: _t('Add a sub-task'),
    run: 'click',
}, {
    isActive: ["auto"],
    trigger: '.o_field_subtasks_one2many div[name="name"] input',
    content: markup(_t('Give the sub-task a <b>name</b>')),
    run: "edit New Sub-task",
},
{
    trigger: ".o_form_project_tasks .o_form_dirty",
},
{
    isActive: ["auto"],
    trigger: ".o_form_button_save",
    content: markup(_t("You have unsaved changes - no worries! Odoo will automatically save it as you navigate.<br/> You can discard these changes from here or manually save your task.<br/>Let's save it manually.")),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_form_project_tasks",
},
{
    trigger: ".o_breadcrumb .o_back_button",
    content: markup(_t("Let's go back to the <b>kanban view</b> to have an overview of your next tasks.")),
    tooltipPosition: "right",
    run: 'click',
}, {
    isActive: ["auto"],
    trigger: ".o_kanban_record .o_widget_subtask_counter .subtask_list_button",
    content: _t("You can open sub-tasks from the kanban card!"),
    run: "click",
},
{
    trigger: ".o_widget_subtask_kanban_list .subtask_list",
},
{
    isActive: ["auto"],
    trigger: ".o_kanban_record .o_widget_subtask_kanban_list .subtask_create",
    content: _t("Create a new sub-task"),
    run: "click",
},
{
    trigger: ".subtask_create_input",
},
{
    isActive: ["auto"],
    trigger: ".o_kanban_record .o_widget_subtask_kanban_list .subtask_create_input input",
    content: markup(_t("Give the sub-task a <b>name</b>")),
    run: "edit Newer Sub-task && click body",
}, {
    isActive: ["auto"],
    trigger: ".o_kanban_record .o_widget_subtask_kanban_list .subtask_list_row:contains(newer sub-task) .o_field_project_task_state_selection button",
    content: _t("You can change the sub-task state here!"),
    run: "click",
},
{
    trigger: ".project_task_state_selection_menu.dropdown-menu",
},
{
    isActive: ["auto"],
    trigger: ".project_task_state_selection_menu.dropdown-menu span.text-danger",
    content: markup(_t("Mark the task as <b>Cancelled</b>")),
    run: "click",
}, {
    trigger: ".o-overlay-container:not(:visible):not(:has(.project_task_state_selection_menu))",
}, {
    isActive: ["auto"],
    trigger: ".o_kanban_record .o_widget_subtask_counter .subtask_list_button:contains('1/2')",
    content: _t("Close the sub-tasks list"),
    run: "click",
}, {
    isActive: ["auto"],
    trigger: '.o_kanban_renderer',
    // last step to confirm we've come back before considering the tour successful
    run: "click",
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
import { Component, useEffect, useExternalListener, useState } from "@odoo/owl";

export class ProjectSharingWebClient extends Component {
    static props = {};
    static components = { ActionContainer, MainComponentsContainer };
    static template = "project.ProjectSharingWebClient";

    setup() {
        window.parent.document.body.style.margin = "0"; // remove the margin in the parent body
        this.actionService = useService('action');
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
        const { action_name, action_context, project_id, project_name, open_task_action } = session;
        const action = await this.actionService.loadAction(action_name, {
            active_id: project_id,
        });
        action.display_name = project_name;
        await this.actionService.doAction(
            action,
            {
                clearBreadcrumbs: true,
                additionalContext: {
                    active_id: project_id,
                    ...action_context,
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

## File: static\src\project_sharing\chatter\chatter_patch.js

```javascript
import { Chatter } from "@mail/chatter/web_portal/chatter";

import { useSubEnv } from "@odoo/owl";
import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";

patch(Chatter.prototype, {
    setup() {
        super.setup(...arguments);
        Object.assign(this.state, {
            isFollower: this.props.isFollower,
        });
        this.orm = useService("orm");
        useSubEnv({
            projectSharingId: this.props.projectSharingId,
            inFrontendPortalChatter: true,
        });
    },

    async toggleIsFollower() {
        this.state.isFollower = await this.orm.call(
            this.props.threadModel,
            "project_sharing_toggle_is_follower",
            [this.props.threadId]
        );
    },
    onPostCallback() {
        super.onPostCallback();
        this.state.isFollower = true;
    },
});
Chatter.props = [
    ...Chatter.props,
    "token",
    "projectSharingId",
    "isFollower",
    "displayFollowButton",
];

```

## File: static\src\project_sharing\chatter\composer_patch.js

```javascript
import { Composer } from "@mail/core/common/composer";

import { patch } from "@web/core/utils/patch";

patch(Composer.prototype, {
    get extraData() {
        const extraData = super.extraData;
        if (this.env.projectSharingId) {
            extraData.project_sharing_id = this.env.projectSharingId;
        }
        return extraData;
    },

    get isSendButtonDisabled() {
        if (this.thread && !this.thread.id) {
            return true;
        }
        return super.isSendButtonDisabled;
    },

    get allowUpload() {
        if (this.thread && !this.thread.id) {
            return false;
        }
        return super.allowUpload;
    },
});

```

## File: static\src\project_sharing\chatter\portal_chatter_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="portal.Chatter" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o-mail-Chatter-top')]" position="before">
            <div class="d-flex justify-content-end">
                <button t-if="props.displayFollowButton" class="btn btn-link w-auto" t-on-click="toggleIsFollower">
                    <t t-if="state.isFollower">
                        Unfollow
                    </t>
                    <t t-else="">
                        Follow
                    </t>
                </button>
            </div>
        </xpath>
    </t>

</templates>

```

## File: static\src\project_sharing\chatter\suggestion_service_patch.js

```javascript
import { SuggestionService } from "@mail/core/common/suggestion_service";

import { patch } from "@web/core/utils/patch";

patch(SuggestionService.prototype, {
    async fetchPartners(term, thread, { abortSignal } = {}) {
        if (thread.model === "project.task") {
            const suggestedPartners = await this.makeOrmCall(
                "project.task",
                "get_mention_suggestions",
                [thread.id],
                { search: term },
                { abortSignal }
            );
            this.store.insert(suggestedPartners);
            const suggestedPartnersIds = suggestedPartners["res.partner"].map(
                (partner) => partner.id
            );
            thread.limitedMentions = Object.values(this.store.Persona.records).filter((persona) =>
                suggestedPartnersIds.includes(persona.id)
            );
        }
        return super.fetchPartners(...arguments);
    },

    getPartnerSuggestions(thread) {
        if (thread.model === "project.task") {
            return thread.limitedMentions;
        }
        return super.getPartnerSuggestions(...arguments);
    },
});

```

## File: static\src\project_sharing\chatter\thread_model_patch.js

```javascript
import { Record } from "@mail/core/common/record";
import { Thread } from "@mail/core/common/thread_model";

import { patch } from "@web/core/utils/patch";

patch(Thread.prototype, {
    setup() {
        super.setup();
        this.limitedMentions = Record.many("Persona");
    },
});

```

## File: static\src\project_sharing\components\chatter\chatter_attachments_viewer.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";

export class ChatterAttachmentsViewer extends Component {
    static template = "project.ChatterAttachmentsViewer";
    static props = {
        attachments: Array,
        canDelete: { type: Boolean, optional: true },
        delete: { type: Function, optional: true },
    };
    static defaultProps = {
        delete: async () => {},
    };
}

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

import { rpc } from "@web/core/network/rpc";
import { TextField } from '@web/views/fields/text/text_field';
import { PortalAttachDocument } from '../portal_attach_document/portal_attach_document';
import { ChatterAttachmentsViewer } from './chatter_attachments_viewer';
import { Component, useState, onWillUpdateProps, useRef } from "@odoo/owl";

export class ChatterComposer extends Component {
    static template = "project.ChatterComposer";
    static components = {
        ChatterAttachmentsViewer,
        PortalAttachDocument,
        TextField,
    };
    static props = {
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
    static defaultProps = {
        allowComposer: true,
        displayComposer: false,
        isUserPublic: true,
        token: "",
        attachments: [],
    };

    setup() {
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
            thread_model: this.props.resModel,
            thread_id: this.props.resId,
            post_data: {
                body: this.state.message,
                attachment_ids,
                message_type: "comment",
                subtype_xmlid: "mail.mt_comment",
            },
            attachment_tokens,
            project_sharing_id: this.props.projectSharingId,
        };
    }

    async sendMessage() {
        this.clearErrors();
        if (!this.state.message && !this.state.attachments.length) {
            this.state.displayError = true;
            return;
        }

        await rpc(
            "/mail/message/post",
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
        for (const fileData of files) {
            let file = fileData.data["ir.attachment"][0];
            file.state = 'pending';
            this.state.attachments.push(file);
        }
    }

    async deleteAttachment(attachment) {
        this.clearErrors();
        try {
            await rpc(
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

```

## File: static\src\project_sharing\components\chatter\chatter_composer.xml

```xml
<templates id="template" xml:space="preserve">

    <!-- Widget PortalComposer (standalone)
        required many options: token, res_model, res_id, ...
    -->
    <t t-name="project.ChatterComposer">
        <div t-if="props.allowComposer" class="o_portal_chatter_composer pt-1">
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
                            <div class="o_portal_chatter_composer_body pb-3">
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
                                <div class="pt-2">
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
import { useService } from "@web/core/utils/hooks";
import { rpc } from "@web/core/network/rpc";
import { ChatterComposer } from "./chatter_composer";
import { ChatterMessageCounter } from "./chatter_message_counter";
import { ChatterMessages } from "./chatter_messages";
import { ChatterPager } from "./chatter_pager";
import { Component, markup, onWillStart, useState, onWillUpdateProps } from "@odoo/owl";

export class ChatterContainer extends Component {
    static template = "project.ChatterContainer";
    static components = {
        ChatterComposer,
        ChatterMessageCounter,
        ChatterMessages,
        ChatterPager,
    };
    static props = {
        token: { type: String, optional: true },
        resModel: String,
        resId: { type: Number, optional: true },
        pid: { type: String, optional: true },
        hash: { type: String, optional: true },
        pagerStart: { type: Number, optional: true },
        twoColumns: { type: Boolean, optional: true },
        projectSharingId: Number,
        isFollower: Boolean,
        displayFollowButton: Boolean,
    };
    static defaultProps = {
        token: "",
        pid: "",
        hash: "",
        pagerStart: 1,
    };

    setup() {
        this.state = useState({
            currentPage: this.props.pagerStart,
            messages: [],
            options: this.defaultOptions,
        });
        this.ormService = useService("orm");

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
            is_follower: this.props.isFollower,
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
                const prom = [this.fetchMessages()];
                this.state.options.is_follower = true;
                await Promise.all(prom);
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
            const chatterData = await rpc(
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
        const result = await rpc(
            '/mail/chatter_fetch',
            this.messagesParams(this.props),
        );
        this.state.messages = this.preprocessMessages(result.messages);
        this.state.options.message_count = result.message_count;
        return result;
    }

    async toggleIsFollower() {
        const isFollower = await this.ormService.call(
            this.props.resModel,
            "project_sharing_toggle_is_follower",
            [this.props.resId]
        );
        this.state.options.is_follower = isFollower;
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

```

## File: static\src\project_sharing\components\chatter\chatter_container.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.ChatterContainer">
        <div t-attf-class="o_portal_chatter {{props.twoColumns ? 'row' : ''}}">
            <div t-attf-class="{{props.twoColumns ? 'col-lg-5' : props.resId ? 'border-bottom' : ''}}">
                <div class="o_portal_chatter_header d-flex pt-2">
                    <ChatterMessageCounter count="state.options.message_count"/>
                    <button t-if="props.displayFollowButton" class="btn btn-link w-auto" t-on-click="toggleIsFollower">
                        <t t-if="state.options.is_follower">
                            Unfollow
                        </t>
                        <t t-else="">
                            Follow
                        </t>
                    </button>
                </div>
                <ChatterComposer t-props="composerProps"/>
            </div>
            <div t-attf-class="{{props.twoColumns ? 'offset-lg-1 col-lg-6' : 'pt-3'}}">
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

import { rpc } from "@web/core/network/rpc";

import { ChatterAttachmentsViewer } from "./chatter_attachments_viewer";
import { Component } from "@odoo/owl";

export class ChatterMessages extends Component {
    static template = "project.ChatterMessages";
    static props = {
        messages: Array,
        isUserEmployee: { type: Boolean, optional: true },
        update: { type: Function, optional: true },
    };
    static defaultProps = {
        update: (message_id, changes) => {},
    };
    static components = { ChatterAttachmentsViewer };

    /**
     * Toggle the visibility of the message.
     *
     * @param {Object} message message to change the visibility
     */
    async toggleMessageVisibility(message) {
        const result = await rpc(
            '/mail/update_is_internal',
            { message_id: message.id, is_internal: !message.is_internal },
        );
        this.props.update(message.id, { is_internal: result });
    }
}

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

export class ChatterMessageCounter extends Component {
    static template = "project.ChatterMessageCounter";
    static props = {
        count: Number,
    };
}

```

## File: static\src\project_sharing\components\chatter\chatter_message_counter.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="project.ChatterMessageCounter">
        <div class="o_message_counter d-flex justify-content-center align-items-center">
            <t t-if="props.count">
                <span class="fa fa-comments" />
                <span class="o_message_count px-1"> <t t-esc="props.count"/> </span>
                comments
            </t>
            <span t-else="">
                There are no comments for now.
            </span>
        </div>
    </t>

</templates>

```

## File: static\src\project_sharing\components\chatter\chatter_pager.js

```javascript
/** @odoo-module */

import { Component, useState, onWillUpdateProps } from "@odoo/owl";

export class ChatterPager extends Component {
    static template = "project.ChatterPager";
    static props = {
        pagerScope: Number,
        pagerStep: Number,
        page: Number,
        messageCount: Number,
        changePage: Function,
    };

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

## File: static\src\project_sharing\components\depend_on_ids_one2many\depend_on_ids_list_renderer.js

```javascript
/** @odoo-module */

import { ListRenderer } from "@web/views/list/list_renderer";

export class DependOnIdsListRenderer extends ListRenderer {
    get nbHiddenRecords() {
        const { context, records } = this.props.list;
        return context.depend_on_count - records.length;
    }
}

DependOnIdsListRenderer.rowsTemplate = "project.DependOnIdsListRowsRenderer";

```

## File: static\src\project_sharing\components\depend_on_ids_one2many\depend_on_ids_list_renderer.xml

```xml
<templates>
    <t t-name="project.DependOnIdsListRowsRenderer" t-inherit="web.ListRenderer.Rows" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-foreach='list.records']" position="after">
        <tr class="o_data_row">
            <td t-if="nbHiddenRecords" t-att-colspan="nbCols" style="text-align:center;">
                <i class="text-muted">
                    This task is currently blocked by <t t-out="nbHiddenRecords"/> (other) tasks to which you do not have access.
                </i>
            </td>
        </tr>
        </xpath>
    </t>
</templates>

```

## File: static\src\project_sharing\components\depend_on_ids_one2many\depend_on_ids_one2many_field.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";

import { DependOnIdsListRenderer } from "./depend_on_ids_list_renderer";

export class DependOnIdsOne2ManyField extends X2ManyField {
    static components = {
        ...X2ManyField.components,
        ListRenderer: DependOnIdsListRenderer,
    };
}

export const dependOnIdsOne2ManyField = {
    ...x2ManyField,
    component: DependOnIdsOne2ManyField,
};

registry.category("fields").add("depend_on_ids_one2many", dependOnIdsOne2ManyField);

```

## File: static\src\project_sharing\components\portal_attach_document\portal_attach_document.js

```javascript
/** @odoo-module */

import { PortalFileInput } from '../portal_file_input/portal_file_input';
import { Component } from "@odoo/owl";

export class PortalAttachDocument extends Component {
    static template = "project.PortalAttachDocument";
    static components = { PortalFileInput };
    static props = {
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
    static defaultProps = {
        acceptedFileExtensions: "*",
        onUpload: () => {},
        route: "/mail/attachment/upload",
        beforeOpen: async () => true,
    };
}

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
    static props = {
        ...FileInput.props,
        accessToken: { type: String, optional: true },
    };
    static defaultProps = {
        ...FileInput.defaultProps,
        accessToken: "",
    };

    /**
     * @override
     */
    get httpParams() {
        const {
            model: thread_model,
            id: thread_id,
            ...otherParams
        } = super.httpParams;
        return {
            thread_model,
            thread_id,
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
                        ufile: [file],
                        token: otherParams.access_token,
                        is_pending: true,
                        ...otherParams,
                    })
            )
        );
        return filesData;
    }
}

```

## File: static\src\project_sharing\editor\project_sharing_media_plugin.js

```javascript
import { ImageCropPlugin } from "@html_editor/main/media/image_crop_plugin";
import { MediaPlugin } from "@html_editor/main/media/media_plugin";
import { MAIN_PLUGINS } from "@html_editor/plugin_sets";

export class ProjectSharingMediaPlugin extends MediaPlugin {
    resources = {
        ...this.resources,
        toolbar_items: this.resources.toolbar_items.filter(
            item => item.id !== "replace_image"
        ),
    }
    async createAttachment({ el, imageData, resId }) {
        const response = JSON.parse(
            await this.services.http.post(
                "/project_sharing/attachment/add_image",
                {
                    name: el.dataset.fileName || "",
                    data: imageData,
                    res_id: resId,
                    access_token: "",
                    csrf_token: odoo.csrf_token,
                },
                "text"
            )
        );
        if (response.error) {
            this.services.notification.add(response.error, { type: "danger" });
            el.remove();
        }
        const attachment = response;
        attachment.image_src = "/web/image/" + attachment.id + "-" + attachment.name;
        return attachment;
    }
}

MAIN_PLUGINS.splice(MAIN_PLUGINS.indexOf(MediaPlugin), 1);
MAIN_PLUGINS.push(ProjectSharingMediaPlugin);

MAIN_PLUGINS.splice(MAIN_PLUGINS.indexOf(ImageCropPlugin), 1);

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
import { append, createElement, setAttributes } from "@web/core/utils/xml";
import { registry } from "@web/core/registry";
import { SIZES } from "@web/core/ui/ui_service";
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
    const chatterContainerXml = createElement("Chatter");
    const parentURLQuery = new URLSearchParams(window.parent.location.search);
    setAttributes(chatterContainerXml, {
        token: `'${parentURLQuery.get("access_token")}'` || "",
        threadModel: params.resModel,
        threadId: params.resId,
        projectSharingId: params.projectSharingId,
        isFollower: params.isFollower,
        displayFollowButton: params.displayFollowButton,
    });
    const chatterContainerHookXml = createElement("div");
    chatterContainerHookXml.classList.add("o-mail-ChatterContainer", "o-mail-Form-chatter", "pt-2");
    append(chatterContainerHookXml, chatterContainerXml);
    return chatterContainerHookXml;
}

registry.category("form_compilers").add("portal_chatter_compiler", {
    selector: "chatter",
    fn: (node) =>
        compileChatter(node, {
            resId: "__comp__.props.record.resId or undefined",
            resModel: "__comp__.props.record.resModel",
            projectSharingId: "__comp__.props.record.context.active_id_chatter",
            isFollower: "__comp__.props.record.data.message_is_follower",
            displayFollowButton: "__comp__.props.record.data.display_follow_button",
        }),
});

patch(FormCompiler.prototype, {
    compile(node, params) {
        const res = super.compile(node, params);
        const chatterContainerHookXml = res.querySelector(".o-mail-Form-chatter");
        if (!chatterContainerHookXml) {
            return res; // no chatter, keep the result as it is
        }
        if (chatterContainerHookXml.parentNode.classList.contains("o_form_sheet")) {
            return res; // if chatter is inside sheet, keep it there
        }
        const formSheetBgXml = res.querySelector(".o_form_sheet_bg");
        const parentXml = formSheetBgXml && formSheetBgXml.parentNode;
        if (!parentXml) {
            return res; // miss-config: a sheet-bg is required for the rest
        }
        // after sheet bg (standard position, below form)
        setAttributes(chatterContainerHookXml, {
            "t-att-class": `{
                "overflow-x-hidden overflow-y-auto o-aside h-100": __comp__.uiService.size >= ${SIZES.XXL},
                "px-3 py-0": __comp__.uiService.size < ${SIZES.XXL},
            }`,
        });
        append(parentXml, chatterContainerHookXml);
        return res;
    },
});

```

## File: static\src\project_sharing\views\form\project_sharing_form_controller.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { FormController } from '@web/views/form/form_controller';
import { useService } from '@web/core/utils/hooks';
import { useExternalListener } from "@odoo/owl";

export class ProjectSharingFormController extends FormController {
    static components = {
        ...FormController.components,
    };

    setup() {
        super.setup();
        this.notification = useService('notification');
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

```

## File: static\src\project_sharing\views\form\project_sharing_form_renderer.js

```javascript
import { FormRenderer } from "@web/views/form/form_renderer";

import { Chatter } from "@mail/chatter/web_portal/chatter";

export class ProjectSharingFormRenderer extends FormRenderer {
    static components = {
        ...FormRenderer.components,
        Chatter,
    };
}

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

## File: static\src\project_sharing\views\list\list_view.js

```javascript
/** @odoo-module */

import { listView } from "@web/views/list/list_view";

const props = listView.props;
listView.props = function (genericProps, view) {
    const result = props(genericProps, view);
    return {
        ...result,
        allowSelectors: false,
    };
};

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

## File: static\src\views\analytic_account_form\analytic_account_form_controller.js

```javascript
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";
import { FormController } from "@web/views/form/form_controller";

export class AnalyticAccountFormController extends FormController {
    getStaticActionMenuItems() {
        const menuItems = super.getStaticActionMenuItems();
        if (this.model.root.data.project_count) {
            menuItems.archive.callback = async () => {
                const projects = await this.orm.call("project.project", "search_read", [
                    [["account_id", "=", this.props.resId]],
                    ["name"],
                ]);
                this.dialogService.add(ConfirmationDialog, {
                    ...this.archiveDialogProps,
                    body: _t(
                        "This analytic account is associated with the following projects:\n%(projectList)s\n\nArchiving the account will remove the option to log timesheets for these projects.\n\nAre you sure you want to proceed?",
                        {
                            projectList: projects
                                .map((project) => `\t- ${project.name}`)
                                .join(`\n`),
                        }
                    ),
                    confirmLabel: _t("Archive Account"),
                    cancelLabel: _t("Discard"),
                });
            };
        }
        return menuItems;
    }
}

```

## File: static\src\views\analytic_account_form\analytic_account_form_view.js

```javascript
import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";

import { AnalyticAccountFormController } from "./analytic_account_form_controller";

export const AnalyticAccountFormView = {
    ...formView,
    Controller: AnalyticAccountFormController,
};

registry.category("views").add("analytic_account_form_view", AnalyticAccountFormView);

```

## File: static\src\views\analytic_account_list\analytic_account_list_controller.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { ListController } from "@web/views/list/list_controller";

export class AnalyticAccountListController extends ListController {
    get archiveDialogProps() {
        let dialogProps = super.archiveDialogProps;
        const selectedRecords = this.model.root.selection;

        if (selectedRecords.length) {
            const analyticAccountWithProjects = selectedRecords
                .filter((record) => record.data.project_count)
                .map((record) => record.data.name);
            if (analyticAccountWithProjects.length) {
                dialogProps = {
                    ...dialogProps,
                    body: _t(
                        "Some of the selected analytic accounts are associated with a project:\n%(accountList)s\n\nArchiving these accounts will remove the option to log timesheets for their respective projects.\n\nAre you sure you want to proceed?",
                        {
                            accountList: analyticAccountWithProjects
                                .map((name) => `\t- ${name}`)
                                .join("\n"),
                        }
                    ),
                    confirmLabel: _t("Archive Accounts"),
                    cancelLabel: _t("Discard"),
                };
            }
        }
        return dialogProps;
    }
}

```

## File: static\src\views\analytic_account_list\analytic_account_list_view.js

```javascript
import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";

import { AnalyticAccountListController } from "./analytic_account_list_controller";

export const AnalyticAccountListView = {
    ...listView,
    Controller: AnalyticAccountListController,
};

registry.category("views").add("analytic_account_list_view", AnalyticAccountListView);

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
                if (this.stageIdSearchItemId && this.dateSearchItemId && this.isClosedSearchItemId) {
                    return;
                }
                switch (searchItem.fieldName) {
                    case 'date':
                        this.dateSearchItemId = searchItem.id;
                        break;
                    case 'stage_id':
                        this.stageIdSearchItemId = searchItem.id;
                        break;
                    case 'is_closed':
                        this.isClosedSearchItemId = searchItem.id;
                        break;
                }
            }
        }
    }

    /**
     * @override
     */
    deactivateGroup(groupId) {
        // Prevent removing 'Date & Stage' and 'Date & is closed' group by from the search
        if (this.searchItems[this.dateSearchItemId].groupId == groupId) {
            if (this.query.some(queryElem => [this.stageIdSearchItemId, this.isClosedSearchItemId].includes(queryElem.searchItemId))){
                this._addGroupByNotification(_t("The report should be grouped either by \"Stage\" to represent a Burndown Chart or by \"Is Closed\" to represent a Burn-up chart. Without one of these groupings applied, the report will not provide relevant information."));
            }
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
                    this._addGroupByNotification(_t("The Burndown Chart must be grouped by Date"));
                }
            }
        }
        super.toggleDateGroupBy(...arguments);
    }

    /**
     * @override
     * Ensure here that there is always either the 'stage' or the 'is_closed' searchItemId inside the query.
     */
    toggleSearchItem(searchItemId) {
        // if the current searchItem stage/is_closed, the counterpart is added before removing the current searchItem
        if (searchItemId === this.isClosedSearchItemId){
            super.toggleSearchItem(this.stageIdSearchItemId);
        } else if (searchItemId === this.stageIdSearchItemId){
            super.toggleSearchItem(this.isClosedSearchItemId);
        }
        super.toggleSearchItem(...arguments);
    }

    /**
     * Adds a notification related to the group by constraint of the Burndown Chart.
     * @param body The message to display in the notification.
     * @private
     */
    _addGroupByNotification(body) {
        this.notificationService.add(
            body,
            { type: "danger" }
        );
    }

    /**
     * @override
     */
    async _notify() {
        // Ensure that we always group by date first and by stage_id/is_closed second
        let stageIdIndex = -1;
        let dateIndex = -1;
        let isClosedIndex = -1;
        for (const [index, queryElem] of this.query.entries()) {
            if (dateIndex !== -1 && (stageIdIndex !== -1 || isClosedIndex !== -1)) {
                break;
            }
            switch (queryElem.searchItemId) {
                case this.dateSearchItemId:
                    dateIndex = index;
                    break;
                case this.stageIdSearchItemId:
                    stageIdIndex = index;
                    break;
                case this.isClosedSearchItemId:
                    isClosedIndex = index;
                    break;
            }
        }
        if (isClosedIndex > 0) {
            if (isClosedIndex > dateIndex) {
                dateIndex += 1;
            }
            this.query.splice(0, 0, this.query.splice(stageIdIndex, 1)[0]);
        } else if (stageIdIndex > 0) {
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


export class ProjectProjectKanbanRenderer extends KanbanRenderer {
    static components = {
        ...KanbanRenderer.components,
        KanbanHeader: ProjectProjectKanbanHeader,
    };
}

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

## File: static\src\views\project_task_calendar\project_task_calendar_controller.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { CalendarController } from "@web/views/calendar/calendar_controller";
import { subTaskDeleteConfirmationMessage } from "@project/views/project_task_form/project_task_form_controller";
import { ProjectTaskCalendarFilterPanel } from "./project_task_calendar_filter_panel/project_task_calendar_filter_panel";

export class ProjectTaskCalendarController extends CalendarController {
    static components = {
        ...ProjectTaskCalendarController.components,
        FilterPanel: ProjectTaskCalendarFilterPanel,
    };

    get editRecordDefaultDisplayText() {
        return _t("New Task");
    }

    deleteConfirmationDialogProps(record) {
        const deleteConfirmationDialogProps = super.deleteConfirmationDialogProps(record);
        if  (!record.rawRecord.subtask_count) {
            return deleteConfirmationDialogProps;
        }

        return {
            ...deleteConfirmationDialogProps,
            body: subTaskDeleteConfirmationMessage,
        }
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
import { ProjectTaskCalendarController } from "./project_task_calendar_controller";
import { ProjectTaskCalendarModel } from "./project_task_calendar_model";

export const projectTaskCalendarView = {
    ...calendarView,
    Controller: ProjectTaskCalendarController,
    Model: ProjectTaskCalendarModel,
};
registry.category("views").add("project_task_calendar", projectTaskCalendarView);

```

## File: static\src\views\project_task_calendar\project_task_calendar_filter_panel\project_task_calendar_filter_panel.js

```javascript
/** @odoo-module **/

import { CalendarFilterPanel } from "@web/views/calendar/filter_panel/calendar_filter_panel";

export class ProjectTaskCalendarFilterPanel extends CalendarFilterPanel { 
    static subTemplates = {
        filter: "project.ProjectTaskCalendarFilterPanel.filter",
    };
}

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

import { _t } from "@web/core/l10n/translation";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { HistoryDialog } from "@html_editor/components/history_dialog/history_dialog";
import { useService } from '@web/core/utils/hooks';
import { markup } from '@odoo/owl';
import { escape } from '@web/core/utils/strings';
import { FormControllerWithHTMLExpander } from '@resource/views/form_with_html_expander/form_controller_with_html_expander';

export const subTaskDeleteConfirmationMessage = _t(
    `Deleting a task will also delete its associated sub-tasks. \
If you wish to preserve the sub-tasks, make sure to unlink them from their parent task beforehand.

Are you sure you want to proceed?`
);

export class ProjectTaskFormController extends FormControllerWithHTMLExpander {
    setup() {
        super.setup();
        this.notifications = useService("notification");
    }

    /**
     * @override
     */
    getStaticActionMenuItems() {
        return {
            ...super.getStaticActionMenuItems(),
            openHistoryDialog: {
                sequence: 50,
                icon: "fa fa-history",
                description: _t("Version History"),
                callback: () => this.openHistoryDialog(),
            },
        };
    }

    get deleteConfirmationDialogProps() {
        const deleteConfirmationDialogProps = super.deleteConfirmationDialogProps;
        if (!this.model.root.data.subtask_count) {
            return deleteConfirmationDialogProps;
        }
        return {
            ...deleteConfirmationDialogProps,
            body: subTaskDeleteConfirmationMessage,
        }
    }

    async openHistoryDialog() {
        const record = this.model.root;
        const versionedFieldName = 'description';
        const historyMetadata = record.data["html_field_history_metadata"]?.[versionedFieldName];
        if (!historyMetadata) {
            this.notifications.add(
                escape(_t(
                    "The task description lacks any past content that could be restored at the moment."
                ))
            );
            return;
        }

        this.dialogService.add(
            HistoryDialog,
            {
                title: _t("Task Description History"),
                noContentHelper: markup(
                    `<span class='text-muted fst-italic'>${escape(
                        _t(
                            "The task description was empty at the time."
                        )
                    )}</span>`
                ),
                recordId: record.resId,
                recordModel: this.props.resModel,
                versionedFieldName,
                historyMetadata,
                restoreRequested: (html, close) => {
                    this.dialogService.add(ConfirmationDialog, {
                        title: _t("Are you sure you want to restore this version ?"),
                        body: _t("Restoring will replace the current content with the selected version. Any unsaved changes will be lost."),
                        confirm: () => {
                            const restoredData = {};
                            restoredData[versionedFieldName] = html;
                            record.update(restoredData);
                            close();
                        },
                        confirmLabel: _t("Restore"),
                    });
                },
            },
        );
    }
}

```

## File: static\src\views\project_task_form\project_task_form_view.js

```javascript
import { formViewWithHtmlExpander } from "@resource/views/form_with_html_expander/form_view_with_html_expander";
import { registry } from "@web/core/registry";
import { ProjectTaskFormController } from "./project_task_form_controller";

export const projectTaskFormView = {
    ...formViewWithHtmlExpander,
    Controller: ProjectTaskFormController,
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

import { registry } from "@web/core/registry";
import { graphView } from "@web/views/graph/graph_view";
import { ProjectTaskGraphModel } from "./project_task_graph_model";

const viewRegistry = registry.category("views");

export const projectTaskGraphView = {
    ...graphView,
    Model: ProjectTaskGraphModel,
};

viewRegistry.add("project_task_graph", projectTaskGraphView);

```

## File: static\src\views\project_task_kanban\project_task_kanban_compiler.js

```javascript
/** @odoo-module **/

import { KanbanCompiler } from "@web/views/kanban/kanban_compiler";

export class ProjectTaskKanbanCompiler extends KanbanCompiler {
    setup() {
        super.setup();
        this.subtaskListComponentCompiled = {
            button: false,
            component: false,
        };
        this.compilers.unshift(
            { selector: 'widget[name="subtask_counter"]', fn: this.compileSubtaskListButton },
            { selector: 'widget[name="subtask_kanban_list"]', fn: this.compileSubtaskListComponent },
        );
    }

    /**
     * @param {Element} el
     * @returns {Element}
     */
    compileSubtaskListButton(el) {
        this.subtaskListComponentCompiled.button = true;
        return this.compileWidget(el);
    }

    /**
     * @param {Element} el
     * @returns {Element}
     */
    compileSubtaskListComponent(el) {
        this.subtaskListComponentCompiled.component = true;
        return this.compileWidget(el);
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
import { user } from "@web/core/user";
import { useService } from '@web/core/utils/hooks';
import { KanbanHeader } from "@web/views/kanban/kanban_header";
import { onWillStart } from "@odoo/owl";

export class ProjectTaskKanbanHeader extends KanbanHeader {
    setup() {
        super.setup();
        this.action = useService('action');

        this.isProjectManager = false;
        onWillStart(this.onWillStart);
    }

    async onWillStart() {
        if (this.props.list.isGroupedByStage) { // no need to check it if not grouped by stage
            this.isProjectManager = await user.hasGroup('project.group_project_manager');
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
import { Record } from "@web/model/relational_model/record";
import { makeActiveField } from "@web/model/relational_model/utils";

export class ProjectTaskKanbanDynamicGroupList extends RelationalModel.DynamicGroupList {
    get isGroupedByStage() {
        return !!this.groupByField && this.groupByField.name === "stage_id";
    }
}

export class ProjectTaskRecord extends Record {
    setup() {
        super.setup(...arguments);
        this.displaySubtasks = false;
        this.canSaveOnUpdate = true;
    }

    async toggleSubtasksList() {
        const { display_name, project_id, state, user_ids } = this.config.fields;
        const activeField = makeActiveField({ onChange: true });
        activeField.related = {
            activeFields: {
                display_name: makeActiveField(),
                state: makeActiveField(),
                user_ids: makeActiveField(),
                project_id: makeActiveField(),
            },
            fields: {
                display_name,
                project_id,
                state,
                user_ids,
            },
        };
        await this._load({
            activeFields: { ...this.config.activeFields, child_ids: activeField },
        });
        this.displaySubtasks = !this.displaySubtasks;
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
ProjectTaskKanbanModel.Record = ProjectTaskRecord;

```

## File: static\src\views\project_task_kanban\project_task_kanban_record.js

```javascript
/* @odoo-module */

import { KanbanRecord } from "@web/views/kanban/kanban_record";
import { useState } from "@odoo/owl";
import { ProjectTaskKanbanCompiler } from "./project_task_kanban_compiler";
import { SubtaskKanbanList } from "@project/components/subtask_kanban_list/subtask_kanban_list"

export class ProjectTaskKanbanRecord extends KanbanRecord {
    static Compiler = ProjectTaskKanbanCompiler;
    static components = {
        ...KanbanRecord.components,
        SubtaskKanbanList,
    };

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

```

## File: static\src\views\project_task_kanban\project_task_kanban_renderer.js

```javascript
/** @odoo-module */

import { KanbanRenderer } from '@web/views/kanban/kanban_renderer';
import { ProjectTaskKanbanRecord } from './project_task_kanban_record';
import { ProjectTaskKanbanHeader } from './project_task_kanban_header';
import { useService } from '@web/core/utils/hooks';
import { onWillStart } from "@odoo/owl";
import { user } from "@web/core/user";

export class ProjectTaskKanbanRenderer extends KanbanRenderer {
    static components = {
        ...KanbanRenderer.components,
        KanbanRecord: ProjectTaskKanbanRecord,
        KanbanHeader: ProjectTaskKanbanHeader,
    };

    setup() {
        super.setup();
        this.action = useService('action');

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

```

## File: static\src\views\project_task_kanban\project_task_kanban_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { kanbanView } from '@web/views/kanban/kanban_view';
import { ProjectTaskKanbanModel } from "./project_task_kanban_model";
import { ProjectTaskKanbanRenderer } from './project_task_kanban_renderer';

export const projectTaskKanbanView = {
    ...kanbanView,
    Model: ProjectTaskKanbanModel,
    Renderer: ProjectTaskKanbanRenderer,
};

registry.category('views').add('project_task_kanban', projectTaskKanbanView);

```

## File: static\src\views\project_task_list\project_task_list_controller.js

```javascript
/** @odoo-module */

import { ListController } from "@web/views/list/list_controller";
import { subTaskDeleteConfirmationMessage } from "@project/views/project_task_form/project_task_form_controller";

export class ProjectTaskListController extends ListController {

    get deleteConfirmationDialogProps() {
        const deleteConfirmationDialogProps = super.deleteConfirmationDialogProps;
        const hasSubtasks = this.model.root.selection.some(task => task.data.subtask_count > 0)
        if (!hasSubtasks) {
            return deleteConfirmationDialogProps;
        }
        return {
            ...deleteConfirmationDialogProps,
            confirm: async () => {
                await this.model.root.deleteRecords();
                // A re-load is needed to remove deleted sub-tasks from the view
                await this.model.load();
            },
            body: subTaskDeleteConfirmationMessage,
        }
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
import { ProjectTaskListController } from "./project_task_list_controller";
import { ProjectTaskListRenderer } from "./project_task_list_renderer";

export const projectTaskListView = {
    ...listView,
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

import { registry } from "@web/core/registry";
import { pivotView } from "@web/views/pivot/pivot_view";
import { ProjectTaskPivotModel } from "./project_pivot_model";

export const projectPivotView = {
    ...pivotView,
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
    static template = "project.ProjectUpdateKanbanView";
    static components = {
        ...KanbanController.components,
        ProjectRightSidePanel,
    };
    get className() {
        return super.className + ' o_controller_with_rightpanel';
    }
}


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
    static template = "project.ProjectUpdateListView";
    static components = {
        ...ListController.components,
        ProjectRightSidePanel,
    };
    get className() {
        return super.className + ' o_controller_with_rightpanel';
    }
}


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

## File: static\src\views\widget\subtask_counter.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { Component } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";

export class SubtaskCounter extends Component {
    static template = "project.SubtaskCounter";
    static props = {
        ...standardWidgetProps,
    };

    onClick() {
        this.props.record.toggleSubtasksList();
    }

    get closedSubtaskCount() {
        return this.props.record.data.closed_subtask_count;
    }

    get subtaskCount() {
        return this.props.record.data.subtask_count;
    }

    get counterTitle() {
        return _t("%(closedCount)s sub-tasks closed out of %(totalCount)s", {
            closedCount: this.closedSubtaskCount,
            totalCount: this.subtaskCount,
        });
    }

    get counterDisplay() {
        return _t("%(count1)s/%(count2)s", {
            count1: this.closedSubtaskCount,
            count2: this.subtaskCount,
        });
    }
}

export const subtaskCounter = {
    component: SubtaskCounter,
    fieldDependencies: [
        { name: "closed_subtask_count", type: "integer" },
        { name: "subtask_count", type: "integer" },
    ],
};

registry.category("view_widgets").add("subtask_counter", subtaskCounter);

```

## File: static\src\views\widget\subtask_counter.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<templates>

    <t t-name="project.SubtaskCounter" owl="1">
        <a t-att-title="counterTitle"
           class="subtask_list_button btn-link text-dark"
           t-on-click.prevent="() => this.onClick()">
            <span class="fa fa-check-square-o me-1"/>
            <t t-esc="counterDisplay"/>
        </a>
    </t>

</templates>

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
      Use the <span class="fa fa-times-circle text-danger d-inline-block"></span> state to mark the task as cancelled.
      <br/><br/>
      Look for the <span class="fa fa-hourglass-o d-inline-block"></span> icon to see tasks waiting on other ones. Once a task is marked as complete or cancelled, all of its dependencies will be unblocked.
      <br/><br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
    </t>

    <t t-name="project.example.agilescrum">
      Use the <span class="o_status d-inline-block o_status_green"></span> state to inform your colleagues that a task is approved for the next stage. 
      <br/>
      Use the <span class="o_status d-inline-block bg-warning"></span> state to indicate a request for changes or a need for discussion on a task.
      <br/><br/>
      Use the <a class="fa fa-clock-o"></a> icon to organize your daily activities.
      <br/><br/>
      <a class="fa fa-lightbulb-o"></a> Sort your tasks by sprint using milestones, tags, or a dedicated property. At the end of each sprint, just pick the remaining tasks in your list and move them all at once to the next sprint by editing the milestone, tag, or property.
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
            <xpath expr="//form" position="attributes">
                <attribute name="js_class">analytic_account_form_view</attribute>
            </xpath>
        </field>
    </record>

    <record id="view_account_analytic_account_list_inherit" model="ir.ui.view">
        <field name="name">account.analytic.account.list.inherit</field>
        <field name="model">account.analytic.account</field>
        <field name="inherit_id" ref="analytic.view_account_analytic_account_list"/>
        <field name="priority">40</field>
        <field name="arch" type="xml">
            <xpath expr="//list" position="attributes">
                <attribute name="js_class">analytic_account_list_view</attribute>
            </xpath>
            <xpath expr="//list" position="inside">
                <field name="project_count" column_invisible="1"/>
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
                <xpath expr="//field[@name='template_ids']/list" position="attributes">
                    <attribute name="editable">bottom</attribute>
                </xpath>
            </field>
        </record>

        <record id="mail_activity_plan_action_config_project_task_plan" model="ir.actions.act_window">
            <field name="name">Activity Plans</field>
            <field name="res_model">mail.activity.plan</field>
            <field name="path">project-activity-plans</field>
            <field name="view_mode">list,kanban,form</field>
            <field name="search_view_id" ref="mail.mail_activity_plan_view_search"/>
            <field name="context">{'default_res_model': 'project.task'}</field>
            <field name="domain">[('res_model', 'in', ('project.project', 'project.task'))]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a Task Activity Plan
                </p>
                <p>
                    Activity plans are used to assign a list of activities in just a few clicks
                    (e.g. "Progress Report", "Stand-up Meeting", ...)
                </p>
            </field>
        </record>

        <record id="mail_activity_plan_action_project_task_view_tree" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">list</field>
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
        <field name="path">project-activity-types</field>
        <field name="view_mode">list,kanban,form</field>
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
        <field name="name">project.milestone.view.list</field>
        <field name="model">project.milestone</field>
        <field name="arch" type="xml">
            <list decoration-success="can_be_marked_as_done" decoration-danger="is_deadline_exceeded and not can_be_marked_as_done" decoration-muted="is_reached" editable="bottom" sample="1">
                <field name="name"/>
                <field name="deadline" optional="show"/>
                <field name="is_reached" optional="show"/>
                <button name="action_view_tasks"
                        type="object"
                        title="View Tasks"
                        string="View Tasks"
                        class="btn btn-link float-end"
                        invisible="task_count == 0"
                        groups="project.group_project_milestone"/>
            </list>
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
            <li t-if="page_name in ['project_task', 'project_subtasks', 'project_recurrent_tasks'] and project" class="breadcrumb-item active">
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
                <a t-attf-href="/my/projects/{{ project.id }}/task/{{ task.id }}"><t t-esc="task.name"/></a>
            </li>
            <li t-elif="page_name == 'project_recurrent_tasks' and task and project" class="breadcrumb-item active text-truncate">
                <a t-attf-href="/my/projects/{{ project.id }}/task/{{ task.id }}?{{ keep_query() }}"><t t-esc="task.name"/></a>
            </li>
            <li t-elif="task" class="breadcrumb-item active text-break">
                <span t-field="task.name"/>
            </li>
            <li t-if="page_name == 'project_subtasks' or (task and subtask and project)" t-attf-class="breadcrumb-item text-truncate #{'active ' if not subtask else ''}">
                <a t-if="subtask" t-attf-href="/my/tasks/{{ task.id }}/subtasks?{{ keep_query() }}">Sub-tasks</a>
                <t t-else="">Sub-tasks</t>
            </li>
            <li t-elif="page_name == 'project_recurrent_tasks' and task and project" t-attf-class="breadcrumb-item text-truncate">
                Recurrent tasks
            </li>
            <li t-if="subtask" class="breadcrumb-item active text-break">
                <span t-field="subtask.name"/>
            </li>
        </xpath>
    </template>

    <template id="portal_my_home" name="Projects / Tasks" customize_show="True" inherit_id="portal.portal_my_home" priority="40">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="before">
            <t t-set="portal_service_category_enable" t-value="True"/>
        </xpath>
        <div id="portal_service_category" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="icon" t-value="'/web/static/img/folder.svg'"/>
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
                    <t t-set="backend_url" t-value="'/odoo/project.project/%s' % (project.id)"/>
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
        <span t-attf-class="o_priority_star d-inline-block fa fa-star#{' text-warning' if task.priority == '1' else '-o opacity-75'} #{classes if classes else ''}" t-attf-title="Priority: {{'Important' if task.priority == '1' else 'Normal'}}"/>
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
                        <t t-set="group_by_in_header_list" t-value="['priority', 'state', 'project_id', 'stage_id', 'milestone_id']"></t>
                        <t t-set="number_of_header" t-value="8"></t>
                        <!-- Computes the right colspan once and use it everywhere -->
                        <t t-set="grouped_tasks_colspan" t-value="number_of_header - 1 if groupby in group_by_in_header_list else number_of_header"></t>
                        <t t-set="grouped_tasks_colspan" t-value="grouped_tasks_colspan if allow_milestone else grouped_tasks_colspan - 1"></t>
                        <th t-attf-colspan="{{2 if groupby != 'priority' else 1}}"/>
                        <th>Name</th>
                        <th>Assignees</th>
                        <th t-if="groupby != 'project_id' and multiple_projects">Project</th>
                        <th t-if="groupby != 'milestone_id' and allow_milestone" name="project_portal_milestones">Milestone</th>
                        <th t-if="groupby != 'state'"/>
                        <th t-if="groupby != 'stage_id'" class="text-end">Stage</th>
                    </tr>
                </thead>
                <t t-foreach="grouped_tasks" t-as="tasks">
                    <tbody t-if="tasks">
                        <tr t-if="not groupby == 'none'" class="table-light">
                            <th t-if="groupby == 'project_id' and multiple_projects" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <!-- This div is necessary for documents_project_sale -->
                                <div name="project_name" class="d-flex w-100 align-items-center">
                                    <span t-if="tasks[0].sudo().project_id" t-field="tasks[0].sudo().project_id.name"/>
                                    <span t-else="">No Project</span>
                                </div>
                            </th>
                            <th t-if="groupby == 'milestone_id'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <span t-if="tasks[0].sudo().milestone_id and tasks[0].sudo().allow_milestones"
                                      class="text-truncate"
                                      t-field="tasks[0].sudo().milestone_id.name"/>
                                <span t-else="">No Milestone</span>
                            </th>
                            <th t-if="groupby == 'stage_id'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <!-- This div is necessary for documents_project_sale -->
                                <div name="stage_name" class="d-flex w-100 align-items-center">
                                    <span class="text-truncate" t-field="tasks[0].sudo().stage_id.name"/>
                                </div>
                            </th>
                            <th t-if="groupby == 'priority'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <span class="text-truncate" t-field="tasks[0].sudo().priority"/></th>
                            <th t-if="groupby == 'state'" t-attf-colspan="{{grouped_tasks_colspan}}">
                                <span class="text-truncate" t-field="tasks[0].sudo().state"/></th>
                            <th t-if="groupby == 'partner_id'" t-attf-colspan="{{grouped_tasks_colspan}}">
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
                                    <div t-if="assignees" class="flex-nowrap ps-3">
                                        <img class="rounded o_portal_contact_img me-2" t-attf-src="#{image_data_uri(assignees[:1].avatar_128)}" alt="User" style="width: 20px; height: 20px;"/>
                                        <span t-att-title="'\n'.join(assignees.mapped('name'))">
                                            <span t-field="assignees[:1].name"/><span t-if="len(assignees) &gt; 1" class="badge ms-1 rounded-pill bg-light"> +<span t-out="len(assignees) - 1"/></span>
                                        </span>
                                    </div>
                                </td>
                                <td t-if="groupby != 'project_id' and multiple_projects">
                                    <span title="Current project of the task" t-out="task.project_id.name" />
                                </td>
                                <td t-if="groupby != 'milestone_id' and allow_milestone" name="project_portal_milestones">
                                    <t t-if="task.milestone_id and task.allow_milestones">
                                        <span t-esc="task.milestone_id.name" />
                                    </t>
                                </td>
                                <td t-if="groupby != 'state'" align="right" class="align-middle">
                                    <t t-call="project.portal_my_tasks_state_widget_template">
                                        <t t-set="path" t-value="'tasks'"/>
                                    </t>
                                </td>
                                <td t-if="groupby != 'stage_id'" class="text-end lh-1">
                                    <span class="fw-normal o_text_overflow" t-attf-title="#{task.stage_id.name}" t-out="task.stage_id.name"/>
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
                    <t t-set="backend_url" t-value="'/odoo/action-project.action_view_my_task/%s' % (task.id)"/>
                </t>
            </t>

            <div class="row o_project_portal_sidebar">
                <t t-call="portal.portal_record_sidebar">
                    <t t-set="classes" t-value="'col-lg-4 col-xxl-3 d-print-none'"/>

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
                                            <a class="nav-link p-0" t-att-href="task_link['access_url']" t-att-target="task_link.get('target', '_self')">
                                                <t t-out="task_link['title']"/>
                                            </a>
                                        </li>
                                    </t>
                                </ul>
                            </div>

                            <div t-if="task.user_ids or task.partner_id" class="d-flex flex-column gap-4 mt-3">
                                <div t-if="task.user_ids">
                                    <h6 class="flex-basis-100"><small class="text-muted">Assignees</small></h6>
                                    <t t-foreach="task.user_ids" t-as="user">
                                        <t t-call="portal.portal_my_contact">
                                            <t t-set="_spacingClass" t-value="'mb-3' if len(task.user_ids) > 1 else ''"/>
                                            <t t-set="_contactAvatar" t-value="image_data_uri(user.avatar_128)"/>
                                            <t t-set="_contactName" t-value="user.name"/>
                                            <div t-out="user" t-options='{"widget": "contact", "fields": ["email", "phone"]}'/>
                                        </t>
                                    </t>
                                </div>
                                <div class="col-12 d-flex flex-column" t-if="task.partner_id">
                                    <h6><small class="text-muted">Customer</small></h6>
                                    <t t-if="task.partner_id">
                                        <t t-call="portal.portal_my_contact">
                                            <t t-set="_contactAvatar" t-value="image_data_uri(task.partner_id.avatar_512)"/>
                                            <t t-set="_contactName" t-value="task.partner_id.display_name"/>
                                            <div t-field="task.partner_id" t-options='{"widget": "contact", "fields": ["email", "phone"]}'/>
                                        </t>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </t>
                </t>
                <div id="task_content" class="o_portal_content col-12 col-lg-8 col-xxl-9">
                    <div id="card">
                        <div id="card_header" data-anchor="true">
                            <div class="row justify-content-between mb-3">
                                <div class="col-12 col-md-9">
                                    <div class="d-flex gap-2">
                                        <h3 class="my-0">
                                            <t t-call="project.portal_my_tasks_priority_widget_template"/>
                                            <span t-field="task.name"/>
                                            <small class="text-muted d-none d-md-inline">(#<span t-field="task.id"/>)</small>
                                        </h3>
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
                                <div t-if="not is_html_empty(task.description)" class="mb-4 mb-md-0">
                                    <h5 class="mb-1">Description</h5>
                                    <hr class="mt-1 mb-2"/>
                                    <div class="py-1 px-2 bg-100 small overflow-auto table-responsive mb-4" t-field="task.description"/>
                                </div>
                                <div t-if="task.attachment_ids" class="o_project_portal_attachments">
                                    <h5 class="mb-1">Attachments</h5>
                                    <hr class="mt-1 mb-2"/>
                                    <div class="row">
                                        <t t-foreach="task.attachment_ids" t-as="attachment">
                                            <div class="col-md-6">
                                                <ul class="list-unstyled">
                                                    <li class="oe_attachments">
                                                        <a t-attf-href="/web/content/#{attachment.id}?download=true&amp;access_token=#{attachment.access_token}"
                                                        target="_blank" data-no-post-process="" class="d-flex align-items-center rounded bg-light p-2">
                                                            <div class='oe_attachment_embedded o_image o_image_small me-2 me-lg-3'
                                                                t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype"
                                                                t-attf-data-src="/web/image/#{attachment.id}/50x40?access_token=#{attachment.access_token}">
                                                            </div>
                                                            <div class='oe_attachment_name text-truncate'><t t-esc='attachment.name'/></div>
                                                        </a>
                                                    </li>
                                                </ul>
                                            </div>
                                        </t>
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
        <field name="name">project.project.stage.view.list</field>
        <field name="model">project.project.stage</field>
        <field name="arch" type="xml">
            <list editable="bottom" sample="1" delete="0">
                <field name="sequence" widget="handle"/>
                <field name="name" placeholder="e.g. To Do"/>
                <field name="mail_template_id" optional="hide" context="{'default_model': 'project.project'}"/>
                <field name="company_id" optional="hide" groups="base.group_multi_company" options="{'no_create': True}"/>
                <field name="fold" optional="show"/>
            </list>
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
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
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
                <templates>
                    <t t-name="card">
                        <field name="name" class="fw-bolder mb-4"/>
                        <field name="mail_template_id" class="text-muted mb-2" invisible="not mail_template_id"/>
                        <field name="company_id" groups="base.group_multi_company" invisible="not company_id" class="mb-2"/>
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
        <field name="path">project-stages</field>
        <field name="view_mode">list,kanban,form</field>
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
        <field name="view_mode">list</field>
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
            <field name="context">{
                'default_composition_mode': 'mass_mail',
            }</field>
            <field name="binding_model_id" ref="project.model_project_project"/>
            <field name="binding_view_types">list</field>
        </record>

        <record id="edit_project" model="ir.ui.view">
            <field name="name">project.project.form</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <form string="Project" class="o_form_project_project" js_class="form_description_expander">
                    <field name="company_id" invisible="1"/>
                    <header>
                        <button name="action_open_share_project_wizard" string="Share Project" type="object" class="oe_highlight" groups="project.group_project_manager"
                        invisible="privacy_visibility != 'portal'" context="{'default_access_mode': 'read'}" data-hotkey="r"/>
                        <field name="stage_id" widget="statusbar_duration" options="{'clickable': '1', 'fold_field': 'fold'}" groups="project.group_project_stages" domain="[('company_id', 'in', (company_id, False))]"/>
                    </header>
                <sheet string="Project">
                    <div class="oe_button_box" name="button_box" groups="base.group_user">
                        <button class="oe_stat_button" type="object" name="action_view_tasks" icon="fa-check">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Tasks</span>
                                <span class="o_stat_value">
                                    <field name="closed_task_count"/> / <field name="task_count"/>
                                    (<field name="task_completion_percentage" widget="percentage" options="{'digits': [1, 0]}"/>)
                                </span>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="project_update_all_action" type="object" groups="project.group_project_user" context="{'active_id': id}">
                            <field name="last_update_color" invisible="1"/>
                            <div class="o_stat_info">
                                <field name="update_count" invisible="1"/>
                                <field name="last_update_status" readonly="1" widget="status_with_color" status_label="Dashboard" hideStatusName="True"/>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title">
                        <h1 class="d-flex flex-row">
                            <field name="is_favorite" nolabel="1" widget="project_is_favorite" class="me-2" options="{'autosave': False}"/>
                            <field name="name" options="{'line_breaks': False}" widget="text" class="o_text_overflow" placeholder="e.g. Office Party"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="label_tasks" string="Name of the Tasks" placeholder="e.g. Tasks"/>
                            <field name="partner_id" widget="res_partner_many2one"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
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
                                            <field name="alias_name" placeholder="alias"/>@
                                            <field name="alias_domain_id" class="oe_inline" placeholder="e.g. mycompany.com"
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
                                <group name="group_documents_analytics" string="Analytics" col="1" class="row mt16 o_settings_container" groups="project.group_project_rating">
                                    <div>
                                        <field name="rating_active" invisible="1"/>
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
                        <page name="analytic" string="Analytic" groups="analytic.group_analytic_accounting">
                            <group>
                                 <group>
                                    <field name="account_id"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <chatter reload_on_follower="True"/>
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
                    <filter string="Start Date" name="start_date" date="date_start" end_month="1" end_year="1"/>
                    <filter string="End Date" name="end_date" date="date" end_month="1" end_year="1"/>
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
            <field name="name">project.project.list</field>
            <field name="model">project.project</field>
            <field name="arch" type="xml">
                <list decoration-muted="active == False" string="Projects" multi_edit="1" sample="1" default_order="is_favorite desc, sequence, name, id" js_class="project_project_list">
                    <field name="sequence" column_invisible="True"/>
                    <field name="name" column_invisible="1"/>
                    <field name="message_needaction" column_invisible="True"/>
                    <field name="active" column_invisible="True"/>
                    <field name="is_milestone_exceeded" column_invisible="True"/>
                    <field name="can_mark_milestone_as_done" column_invisible="True"/>
                    <field name="is_milestone_deadline_exceeded" column_invisible="1"/>
                    <field name="allow_milestones" column_invisible="True"/>
                    <field name="is_favorite" string="Favorite" nolabel="1" widget="project_is_favorite" optional="hide"/>
                    <field name="display_name" string="Name" class="fw-bold"/>
                    <field name="partner_id" optional="show" string="Customer"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company" options="{'no_create': True, 'no_open': True}"/>
                    <field name="company_id" column_invisible="True"/>
                    <field name="date_start" string="Planned Date" widget="daterange" options="{'end_date_field': 'date', 'always_range': '1'}" optional="hide"/>
                    <field name="date" column_invisible="True" />
                    <field name="milestone_progress" widget="progressbar"
                        invisible="milestone_progress == 0 or not allow_milestones"
                        optional="hide"
                    />
                    <field name="next_milestone_id"
                        decoration-danger="is_milestone_deadline_exceeded" decoration-success="can_mark_milestone_as_done"
                        optional="hide"
                        invisible="not allow_milestones"
                    />
                    <field name="user_id" optional="show" string="Project Manager" widget="many2one_avatar_user" options="{'no_open':True, 'no_create': True, 'no_create_edit': True}"/>
                    <field name="last_update_color" column_invisible="True"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                    <field name="last_update_status" string="Status" nolabel="1" width="20px" optional="show" widget="project_state_selection"/>
                    <field name="stage_id" options="{'no_open': True}" domain="[('company_id', 'in', (company_id, False))]" optional="show"/>
                    <button string="View Tasks" name="action_view_tasks" type="object"/>
                </list>
            </field>
        </record>

        <record id="project_list_view_group_stage" model="ir.ui.view">
            <field name="name">project.project.list.group.stage</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="view_project"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <list position="attributes">
                    <attribute name="default_group_by">stage_id</attribute>
                </list>
            </field>
        </record>

        <record id="view_project_config" model="ir.ui.view">
            <field name="name">project.project.list.config</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="project.view_project"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//list" position="attributes">
                    <attribute name="default_order">sequence, name, id</attribute>
                </xpath>
                <field name="sequence" position="attributes">
                    <attribute name="column_invisible">0</attribute>
                    <attribute name="widget">handle</attribute>
                </field>
            </field>
        </record>

        <record id="view_project_config_group_stage" model="ir.ui.view">
            <field name="name">project.project.list.config.group.stage</field>
            <field name="model">project.project</field>
            <field name="inherit_id" ref="view_project_config"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <list position="attributes">
                    <attribute name="default_group_by">stage_id</attribute>
                </list>
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
                    <templates>
                        <t t-name="card">
                            <field name="name" class="fw-bolder" string="Project Name"/>
                            <div class="d-flex">
                                <field name="partner_id" string="Contact"/>
                                <field name="user_id" class="ms-auto" widget="many2one_avatar_user"/>
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
                            <field name="alias_id" invisible="1"/>
                            <field name="alias_name" placeholder="e.g. office-party"/>@
                            <field name="alias_domain_id" class="oe_inline" placeholder="e.g. mycompany.com"
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
                <kanban highlight_color="color"
                    class="o_project_kanban"
                    js_class="project_project_kanban"
                    on_create="project.open_create_project"
                    action="action_view_tasks" type="object"
		    quick_create_view="project.quick_create_project_form"
                    sample="1"
                    default_order="is_favorite desc, sequence, name, id"
                >
                    <field name="allow_milestones"/>
                    <field name="rating_count" />
                    <field name="rating_active" />
                    <field name="privacy_visibility"/>
                    <field name="last_update_color"/>
                    <progressbar field="last_update_status" colors='{"on_track": "success", "at_risk": "warning", "off_track": "danger", "on_hold": "info", "done": "purple"}'/>
                    <field name="sequence" widget="handle"/>
                    <templates>
                        <t t-name="menu" groups="base.group_user">
                            <div class="container">
                                <div class="row">
                                    <div name="card_menu_view" class="col-6">
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
                                    <div class="col-6 o_kanban_manage_reporting">
                                        <h5 role="menuitem" class="o_kanban_card_manage_title" groups="project.group_project_user">
                                            <span>Reporting</span>
                                        </h5>
                                        <div role="menuitem" groups="project.group_project_user" name="task_analysis">
                                            <a name="action_view_tasks_analysis" type="object">Tasks Analysis</a>
                                        </div>
                                        <div role="menuitem" name="project_burndown_menu" groups="project.group_project_user">
                                            <a name="action_project_task_burndown_chart_report" type="object">Burndown Chart</a>
                                        </div>
                                    </div>
                                </div>
                                <div class="o_kanban_card_manage_settings row">
                                    <div role="menuitem" aria-haspopup="true" class="col-6" groups="project.group_project_manager">
                                        <field name="color" widget="kanban_color_picker"/>
                                    </div>
                                    <div role="menuitem" class="col-6" groups="project.group_project_manager">

                                        <a t-if="record.privacy_visibility.raw_value == 'portal'" class="dropdown-item" role="menuitem" name="action_open_share_project_wizard" type="object">Share Project</a>

                                        <a class="dropdown-item" role="menuitem" name="copy" type="object">Duplicate</a>
                                        <a class="dropdown-item" role="menuitem" type="open">Settings</a>
                                    </div>
                                    <div class="col-12 ps-0" groups="!project.group_project_manager">
                                        <div role="menuitem" class="w-100">
                                            <a class="dropdown-item mx-0" role="menuitem" type="open">View</a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                        <t t-name="card">
                            <div class="o_project_kanban_main d-flex align-items-baseline gap-1 ms-1">
                                <field name="is_favorite" widget="project_is_favorite" nolabel="1"/>
                                <div class="min-w-0 pb-4 me-2">
                                    <span class="text-truncate d-block fs-4 fw-bold" t-att-title="record.display_name.value"><field name="display_name"/></span>
                                    <span name="partner_name" class="text-muted d-flex align-items-baseline" t-if="record.partner_id.value">
                                        <span class="fa fa-user me-2" aria-label="Partner" title="Partner"></span><field class="text-truncate" name="partner_id"/>
                                    </span>
                                    <div t-if="record.date.raw_value or record.date_start.raw_value" class="text-muted">
                                        <span class="fa fa-clock-o me-2" title="Dates"></span><field name="date_start"/>
                                        <i t-if="record.date.raw_value and record.date_start.raw_value" class="fa fa-long-arrow-right mx-2 oe_read_only" aria-label="Arrow icon" title="Arrow"/>
                                        <field name="date"/>
                                    </div>
                                    <div t-if="record.alias_email.value" class="text-muted text-truncate" t-att-title="record.alias_email.value">
                                        <span class="fa fa-envelope-o me-2" aria-label="Domain Alias" title="Domain Alias"></span><field name="alias_email"/>
                                    </div>
                                    <div t-if="record.rating_active.raw_value and record.rating_count.raw_value &gt; 0" class="d-flex text-muted" groups="project.group_project_rating">
                                        <b class="me-1">
                                            <span class="fa mt4 fa-smile-o fw-bolder text-success" t-if="record.rating_avg.raw_value &gt;= 3.66" title="Average Rating: Satisfied" role="img" aria-label="Happy face"/>
                                            <span class="fa mt4 fa-meh-o fw-bolder text-warning" t-elif="record.rating_avg.raw_value &gt;= 2.33" title="Average Rating: Okay" role="img" aria-label="Neutral face"/>
                                            <span class="fa mt4 fa-frown-o fw-bolder text-danger" t-else="" title="Average Rating: Dissatisfied" role="img" aria-label="Sad face"/>
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
                            <footer class="mt-auto pt-0 ms-1">
                                <div class="d-flex align-items-center">
                                    <div class="o_project_kanban_boxes d-flex align-items-baseline">
                                        <a class="o_project_kanban_box me-1" name="action_view_tasks" type="object">
                                            <div>
                                                <field name="open_task_count" class="o_value"/>
                                                <field name="label_tasks" class="ms-1"/>
                                            </div>
                                        </a>
                                        <a groups='project.group_project_milestone' t-if="record.allow_milestones and record.allow_milestones.raw_value and record.milestone_count.raw_value &gt; 0"
                                            class="d-inline-block btn-link text-dark small me-1"
                                            role="button"
                                            name="action_get_list_view"
                                            type="object"
                                            t-attf-title="#{record.milestone_count_reached.value} Milestones reached out of #{record.milestone_count.value}"
                                        >
                                            <span class="fa fa-flag me-1"/>
                                            <field name="milestone_count_reached"/>/<field name="milestone_count"/>
                                        </a>
                                    </div>
                                    <field name="activity_ids" widget="kanban_activity" class="ms-2"/>
                                </div>
                                <div class="d-flex ms-auto align-items-center">
                                    <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]" class="me-1"/>
                                    <field t-if="record.last_update_status.value &amp;&amp; widget.editable" name="last_update_status" widget="project_state_selection"/>
                                    <span t-if="record.last_update_status.value &amp;&amp; !widget.editable" t-att-class="'o_status_bubble mx-0 o_color_bubble_' + record.last_update_color.value" t-att-title="record.last_update_status.value"></span>
                                </div>
                            </footer>
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
                    <field name="is_favorite" widget="project_is_favorite" nolabel="1" string="Favorite"/>
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
            <field name="path">project</field>
            <field name="res_model">project.project</field>
            <field name="domain">[]</field>
            <field name="context">{'display_milestone_deadline': True}</field>
            <field name="view_mode">kanban,list,form</field>
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
            <field name="context">{'display_milestone_deadline': True}</field>
            <field name="domain">[]</field>
            <field name="view_mode">kanban,list,form,calendar,activity</field>
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
            <field name="view_mode">list</field>
            <field name="act_window_id" ref="open_view_project_all_group_stage"/>
            <field name="view_id" ref="project_list_view_group_stage"/>
        </record>

        <!-- Please update both act_window when modifying one (open_view_project_all_config or open_view_project_all_config_group_stage) as one or the other is used in the menu menu_project_config -->
        <record id="open_view_project_all_config" model="ir.actions.act_window">
            <field name="name">Projects</field>
            <field name="res_model">project.project</field>
            <field name="path">project-configuration</field>
            <field name="domain">[]</field>
            <field name="view_mode">list,kanban,form</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'list', 'view_id': ref('view_project_config')}),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('view_project_config_kanban')})]"/>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="context">{'display_milestone_deadline': True}</field>
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
            <field name="view_mode">list,kanban,form,calendar,activity</field>
            <field name="search_view_id" ref="view_project_project_filter"/>
            <field name="context">{'display_milestone_deadline': True}</field>
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
            <field name="view_mode">list</field>
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
                <xpath expr="//div[@name='task_analysis']" position="before">
                    <div role="menuitem" groups="project.group_project_user">
                        <a name="project_update_all_action" type="object" t-attf-context="{'active_id': #{record.id.raw_value} }">Dashboard</a>
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
                highlight_color="color"
                class="o_kanban_small_column o_kanban_project_tasks"
                default_group_by="stage_id"
                on_create="quick_create"
                quick_create_view="project.project_sharing_quick_create_task_form"
                archivable="0"
                import="0"
                groups_draggable="0"
                default_order="priority desc, sequence, state, date_deadline asc, id desc"
            >
                <field name="state"/>
                <field name="allow_milestones" />
                <field name="has_late_and_unreached_milestone"/>
                <progressbar field="state" colors='{"1_done": "success", "03_approved": "success", "02_changes_requested": "warning", "1_canceled": "danger", "04_waiting_normal": "200", "01_in_progress": "200"}'/>
                <templates>
                <t t-name="menu" t-if="!selection_mode">
                    <div invisible="1" role="separator" class="dropdown-divider"></div>
                    <field invisible="1" name="color" widget="kanban_color_picker"/>
                </t>
                <t t-name="card">
                    <div t-att-class="{'opacity-50': ['1_done', '1_canceled'].includes(record.state.raw_value)}">
                        <field name="name" class="fw-bolder fs-5" widget="name_with_subtask_count"/>
                        <field name="project_id" invisible="context.get('default_project_id', False)" required="1" class="text-muted"/>
                        <span t-if="record.allow_milestones.raw_value and record.milestone_id.raw_value" t-attf-class="{{record.has_late_and_unreached_milestone.raw_value and !record.state.raw_value.startsWith('1_') ? 'text-danger' : ''}}">
                            <field name="milestone_id" class="text-muted"/>
                        </span>
                        <field t-if="record.partner_id.value" name="partner_id" class="text-truncate text-muted d-block"/>
                    </div>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" context="{'project_id': project_id}"/>
                    <field t-if="record.date_deadline.raw_value" name="date_deadline" invisible="state in ['1_done', '1_canceled']" widget="remaining_days"/>
                    <field t-if="record.displayed_image_id.value" groups="base.group_user" name="displayed_image_id" widget="attachment_image"/>
                    <footer t-if="!selection_mode">
                        <field name="priority" class="me-2" widget="priority"/>
                        <div class="d-flex ms-auto text-truncate">
                            <span t-if="record.portal_user_names.raw_value.length > 0" class="pe-2 text-truncate" t-att-title="record.portal_user_names.raw_value">
                                <t t-set="user_count" t-value="record.portal_user_names.raw_value.split(',').length"/>
                                <t t-if="user_count > 1"><t t-out="user_count"/> assignees</t>
                                <field t-else="" name="portal_user_names"/>
                            </span>
                            <field name="state" widget="project_task_state_selection" options="{'is_toggle_mode': false}"/>
                        </div>
                    </footer>
                </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="project_sharing_project_task_view_tree" model="ir.ui.view">
        <field name="name">project.sharing.project.task.list</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project_task_view_tree_main_base"/>
        <field name="mode">primary</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <list position="attributes">
                <attribute name="delete">0</attribute>
                <attribute name="import">0</attribute>
            </list>
            <xpath expr="//field[@widget='res_partner_many2one']" position="attributes">
                <attribute name="widget">many2one</attribute>
            </xpath>
            <field name="user_ids" position="replace">
                <field name="portal_user_names" string="Assignees"/>
            </field>
            <xpath expr="//field[@name='milestone_id']" position="attributes">
                <attribute name="column_invisible">not context.get('allow_milestones', True)</attribute>
            </xpath>
            <field name="partner_id" position="attributes">
                <attribute name="column_invisible">0</attribute>
            </field>
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
                    <field name="stage_id" widget="statusbar_duration" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="not project_id and not stage_id" />
                </header>
                <div groups="base.group_user" role="status" class="alert alert-info alert-dismissible rounded-0 fade show d-print-none css_editable_mode_hidden">
                    <div class="text-center">This is a preview of how the project will look when it's shared with customers and they have editing access.
                        <a name="action_redirect_to_project_task_form" type="object"><i class="oi oi-arrow-right me-1"/>Back to edit mode</a>
                    </div>
                </div>
                <sheet string="Task">
                    <div class="oe_button_box" name="button_box">
                        <field name="display_parent_task_button" invisible="1"/>
                        <field name="recurrence_id" invisible="1" />
                        <button name="action_project_sharing_view_parent_task" type="object" class="oe_stat_button" icon="fa-tasks" invisible="not display_parent_task_button">
                            <div class="o_stat_info">
                                <span class="o_stat_text">Parent Task</span>
                            </div>
                        </button>
                        <button name="action_project_sharing_recurring_tasks" type="object" invisible="not active or not recurrence_id"
                                class="oe_stat_button" icon="fa-repeat" context="{'default_user_ids': [(6, 0, [uid])]}">
                            <field name="recurring_count" widget="statinfo" string="Recurring Tasks"/>
                        </button>
                        <button name="action_project_sharing_open_subtasks" type="object" class="oe_stat_button" icon="fa-check"
                            invisible="not id or subtask_count == 0" context="{
                                'default_user_ids': [(6, 0, [uid])],
                                'default_project_id': project_id,
                                'default_display_in_project': False,
                            }"
                        >
                            <field name="subtask_count" widget="statinfo" string="Sub-tasks"/>
                            <field name="display_in_project" invisible="True"/>
                        </button>
                        <button name="action_project_sharing_open_blocking" type="object" invisible="not dependent_tasks_count" class="oe_stat_button" icon="fa-tasks">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Blocked Tasks</span>
                                <span class="o_stat_value ">
                                    <field name="dependent_tasks_count" widget="statinfo" nolabel="1" />
                                </span>
                            </div>
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
                            <field name="tag_ids" context="{'project_id': project_id}" widget="many2many_tags" options="{'color_field': 'color', 'no_create': True, 'no_edit': True, 'no_edit_color': True}"/>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="parent_id" invisible="1" />
                            <field name="company_id" invisible="1" />
                            <field name="state" invisible="1" />
                            <field name="depend_on_count" invisible="1" />
                            <field name="allow_task_dependencies" invisible="1" />
                            <field name="current_user_same_company_partner" invisible="1"/>
                            <field name="partner_id" readonly="not current_user_same_company_partner" options="{'no_open': True, 'no_create': True, 'no_edit': True}" invisible="not project_id"/>
                            <field name="date_deadline" decoration-danger="date_deadline &lt; current_date and state not in ['1_done', '1_canceled']"/>
                            <field name="recurring_task" groups="project.group_project_recurring_tasks"/>
                            <label for="repeat_interval" invisible="not recurring_task" groups="project.group_project_recurring_tasks"/>
                            <div invisible="not recurring_task" class="d-flex" groups="project.group_project_recurring_tasks">
                                <field name="repeat_interval" required="recurring_task"
                                       class="me-2" style="max-width: 2rem !important;" />
                                <field name="repeat_unit" required="recurring_task"
                                       class="me-2" style="max-width: 4rem !important;" />
                                <field name="repeat_type" required="recurring_task"
                                       class="me-2" style="max-width: 15rem !important;" />
                                <field name="repeat_until" invisible="not repeat_type == 'until'" required="repeat_type == 'until'"
                                       class="me-2" />
                            </div>
                        </group>
                    </group>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" type="html" placeholder="Add details about this task..."
                                options="{'collaborative': true, 'disableImage': true, 'disableVideo': true, 'disableFile': true}"/>
                        </page>
                        <page name="sub_tasks_page" string="Sub-tasks">
                            <field name="child_ids" context="{
                                'default_project_id': project_id, 'default_display_in_project': False, 'default_parent_id': id,
                                'default_milestone_id': allow_milestones and milestone_id, 'default_partner_id': partner_id,
                                'form_view_ref' : 'project.project_sharing_project_task_view_form',
                            }">
                                <list editable="bottom" decoration-muted="state in ['1_done','1_canceled']">
                                    <field name="project_id" column_invisible="True"/>
                                    <field name="display_in_project" column_invisible="True"/>
                                    <field name="state" column_invisible="True"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="priority" widget="priority" optional="show" nolabel="1" width="20px"/>
                                    <field name="state" widget="project_task_state_selection" nolabel="1" width="20px"/>
                                    <field name="subtask_count" column_invisible="1"/>
                                    <field name="closed_subtask_count" column_invisible="1"/>
                                    <field name="name" widget="name_with_subtask_count"/>
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
                                    <field name="date_deadline" decoration-danger="date_deadline &lt; current_date and state not in ['1_done', '1_canceled']" optional="show"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="stage_id" domain="[('user_id', '=', False), ('project_ids', 'in', [project_id])]"/>
                                    <button name="action_open_task" type="object" title="View Task" string="View Task" class="btn btn-link float-end"
                                            context="{'form_view_ref': 'project.project_sharing_project_task_view_form', 'search_view_ref': 'project.project_sharing_project_task_view_search'}"
                                            invisible="project_id != context.get('active_id')"/>
                                </list>
                            </field>
                        </page>
                        <page name="task_dependencies" string="Blocked By" invisible="not allow_task_dependencies">
                            <field name="depend_on_ids" widget="depend_on_ids_one2many" nolabel="1" options="{'link': false}" readonly="1"
                                context="{
                                    'depend_on_count': depend_on_count,
                                    'list_view_ref': 'project.open_view_blocked_by_list_view',
                                    'search_view_ref': 'project.project_sharing_project_task_view_search',
                                    'search_default_project_id': project_id,
                            }">
                                <list editable="bottom" decoration-muted="state in ['1_done','1_canceled']">
                                    <field name="project_id" column_invisible="True"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="priority" widget="priority" optional="show" nolabel="1" width="20px" readonly="1"/>
                                    <field name="state" widget="project_task_state_selection" nolabel="1" width="20px" readonly="1"/>
                                    <field name="subtask_count" column_invisible="1"/>
                                    <field name="closed_subtask_count" column_invisible="1"/>
                                    <field name="name" widget="name_with_subtask_count"/>
                                    <field name="allow_milestones" column_invisible="True"/>
                                    <field name="milestone_id"
                                        optional="hide"
                                        context="{'default_project_id': project_id}"
                                        invisible="not allow_milestones"
                                        column_invisible="not parent.allow_milestones"
                                    />
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="partner_id" options="{'no_open': True, 'no_create': True, 'no_edit': True}" optional="hide" invisible="not project_id"/>
                                    <field name="portal_user_names" string="Assignees" optional="show"/>
                                    <field name="date_deadline" decoration-danger="date_deadline &lt; current_date and state not in ['1_done', '1_canceled']" optional="show"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    <field name="stage_id" optional="show"/>
                                    <button name="action_open_task" type="object" title="View Task" string="View Task" class="btn btn-link float-end"
                                            context="{'form_view_ref': 'project.project_sharing_project_task_view_form'}"
                                            invisible="project_id != context.get('active_id')"/>
                                </list>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <!-- field used inside the chatter to know if the portal user is follower or not -->
                <field name="message_is_follower" invisible="1"/>
                <field name="display_follow_button" invisible="1"/>
                <chatter/>
            </form>
        </field>
    </record>

    <record id="project_sharing_project_task_view_search" model="ir.ui.view">
        <field name="name">project.task.search.form</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.view_task_search_form_base"/>
        <field name="mode">primary</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <filter name="date_last_stage_update" position="after">
                <filter string="Deadline" name="date_deadline" date="date_deadline">
                    <filter name="deadline_future" string="Future" domain="[('date_deadline', '&gt;', datetime.datetime.combine(context_today(), datetime.time(23,59,59)).to_utc())]"/>
                    <filter name="deadline_this_week" string="This Week" domain="[
                        ('date_deadline', '&gt;=', datetime.datetime.combine(context_today() + relativedelta(weeks=-1,days=1,weekday=0), datetime.time(0,0,0)).to_utc()),
                        ('date_deadline', '&lt;', datetime.datetime.combine(context_today() + relativedelta(days=1,weekday=0), datetime.time(0,0,0)).to_utc()),
                    ]"/>
                    <filter name="deadline_today" string="Today" domain="[
                        ('date_deadline', '&gt;=', datetime.datetime.combine(context_today(), datetime.time(0,0,0)).to_utc()),
                        ('date_deadline', '&lt;=', datetime.datetime.combine(context_today(), datetime.time(23,59,59)).to_utc()),
                    ]"/>
                    <filter name="deadline_past_due" string="Overdue" domain="[('date_deadline', '&lt;', datetime.datetime.combine(context_today(), datetime.time(0,0,0)).to_utc())]"/>
                </filter>
            </filter>
            <filter name="last_stage_update" position="after">
                <filter string="Deadline" name="deadline" context="{'group_by': 'date_deadline'}"/>
            </filter>
            <xpath expr="//search/filter[@name='inactive']" position='attributes'>
                <attribute name='invisible'>1</attribute>
            </xpath>
        </field>
    </record>

    <record id="open_view_blocked_by_list_view" model="ir.ui.view">
        <field name="name">open.view.blocked.by.list.view</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.open_view_all_tasks_list_view"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <list position="attributes">
                <attribute name="default_group_by"/>
            </list>
            <field name="user_ids" position="attributes">
                <attribute name="invisible">1</attribute>
            </field>
            <field name="user_ids" position="after">
                <field name="portal_user_names" string="Assignees" optional="show"/>
            </field>
            
        </field>
    </record>

    <record id="project_sharing_project_task_action" model="ir.actions.act_window">
        <field name="name">Project Sharing</field>
        <field name="res_model">project.task</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="search_view_id" ref="project.project_sharing_project_task_view_search"/>
        <field name="domain">[('project_id', '=', active_id), ('display_in_project', '=', True)]</field>
        <field name="context">{
            'default_project_id': active_id,
            'active_id_chatter': active_id,
            'delete': false,
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
        <field name="view_mode">list</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_tree"/>
    </record>

    <record id="project_sharing_form_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_form"/>
    </record>

    <record id="project_sharing_project_task_action_blocking_tasks" model="ir.actions.act_window">
        <field name="name">Blocking</field>
        <field name="res_model">project.task</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="search_view_id" ref="project.project_sharing_project_task_view_search"/>
        <field name="domain">[('depend_on_ids', '=', active_id), ('id', '!=', active_id)]</field>
        <field name="context">{'default_dependent_ids': active_id}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No tasks found. Let's create one!
            </p><p>
                To get things done, use activities and status on tasks.<br/>
                Chat in real time or by email to collaborate efficiently.
            </p>
        </field>
    </record>

    <record id="project_sharing_blocking_tree_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">list</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action_blocking_tasks"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_tree"/>
    </record>

    <record id="project_sharing_blocking_kanban_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action_blocking_tasks"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_kanban"/>
    </record>

    <record id="project_sharing_blocking_form_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_action_blocking_tasks"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_form"/>
    </record>

    <record id="project_sharing_project_task_action_sub_task" model="ir.actions.act_window">
        <field name="name">Sub-tasks</field>
        <field name="res_model">project.task</field>
        <field name="view_mode">list,kanban,form</field>
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
        <field name="view_mode">list</field>
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

    <record id="project_sharing_project_task_recurring_tasks_action" model="ir.actions.act_window">
        <field name="name">Project Sharing Recurrence</field>
        <field name="res_model">project.task</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="search_view_id" ref="project.project_sharing_project_task_view_search"/>
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

    <record id="project_sharing_recurring_tasks_tree_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">list</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_recurring_tasks_action"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_tree"/>
    </record>

    <record id="project_sharing_recurring_tasks_kanban_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_recurring_tasks_action"/>
        <field name="view_id" ref="project.project_sharing_project_task_view_kanban"/>
    </record>

    <record id="project_sharing_recurring_tasks_form_action_view" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="project.project_sharing_project_task_recurring_tasks_action"/>
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
                <list string="Tags" editable="top" sample="1" multi_edit="1" default_order="name">
                    <field name="name"/>
                    <field name="color" widget="color_picker" optional="show"/>
                </list>
            </field>
        </record>

        <record id="project_tags_action" model="ir.actions.act_window">
            <field name="name">Tags</field>
            <field name="res_model">project.tags</field>
            <field name="path">task-tags</field>
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

        <record id="task_type_tree" model="ir.ui.view">
            <field name="name">project.task.type.list</field>
            <field name="model">project.task.type</field>
            <field name="arch" type="xml">
                <list string="Task Stage" delete="0" sample="1" multi_edit="1" editable="bottom" open_form_view="True">
                    <field name="sequence" widget="handle" optional="show"/>
                    <field name="name" placeholder="e.g. To Do"/>
                    <field name="fold" optional="show"/>
                </list>
            </field>
        </record>

        <record id="task_type_tree_inherited" model="ir.ui.view">
            <field name="name">project.task.type.list.inherited</field>
            <field name="model">project.task.type</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="task_type_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//list" position="attributes">
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
                    <templates>
                        <t t-name="card">
                            <field name="name" class="fw-bolder text-truncate"/>
                            <field name="project_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="open_task_type_form" model="ir.actions.act_window">
            <field name="name">Task Stages</field>
            <field name="res_model">project.task.type</field>
            <field name="path">task-stages</field>
            <field name="view_mode">list,kanban,form</field>
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
            <field name="view_mode">list,kanban,form</field>
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
                    <filter string="Blocking" name="blocking" domain="[('is_closed', '=', False), ('dependent_ids', '!=', False)]" groups="project.group_project_task_dependencies"/>
                    <separator/>
                    <filter string="Last Stage Update" name="date_last_stage_update" date="date_last_stage_update"/>
                    <separator/>
                    <filter string="Open Tasks" name="open_tasks" domain="[('is_closed', '=', False)]"/>
                    <filter string="Closed Tasks" name="closed_tasks" domain="[('is_closed', '=', True)]"/>
                    <filter string="Closed On" name="closed_on" domain="[('is_closed', '=', True)]" date="date_last_stage_update">
                        <filter name="create_date_last_30_days" string="Last 30 Days" domain="[('date_last_stage_update', '&gt;', datetime.datetime.combine(context_today() - relativedelta(days=30), datetime.time(23, 59, 59)).to_utc())]"/>
                        <filter name="create_date_last_365_days" string="Last 365 Days" domain="[('date_last_stage_update', '&gt;', datetime.datetime.combine(context_today() - relativedelta(days=365), datetime.time(23, 59, 59)).to_utc())]"/>
                    </filter>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
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
                 <field name="stage_id" position="before">
                    <field name="user_ids" filter_domain="[('user_ids.name', 'ilike', self), ('user_ids.active', 'in', [True, False])]"/>
                </field>
                <field name="stage_id" position="after">
                    <field name="project_id" string="Project"/>
                </field>
                <field name="partner_id" position="after">
                    <field name="company_id" groups="base.group_multi_company"/>
                    <filter string="My Tasks" name="my_tasks" domain="[('user_ids', 'in', uid)]"/>
                </field>
                <filter name="stage" position="before">
                    <filter string="Assignees" name="user" context="{'group_by': 'user_ids'}"/>
                </filter>
                <filter name="stage" position="after">
                    <filter string="Project" name="project" context="{'group_by': 'project_id'}"/>
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
                    <filter string="Deadline" name="date_deadline" date="date_deadline">
                        <filter name="deadline_future" string="Future" domain="[('date_deadline', '&gt;', datetime.datetime.combine(context_today(), datetime.time(23,59,59)).to_utc())]"/>
                        <filter name="deadline_this_week" string="This Week" domain="[
                            ('date_deadline', '&gt;=', datetime.datetime.combine(context_today() + relativedelta(weeks=-1,days=1,weekday=0), datetime.time(0,0,0)).to_utc()),
                            ('date_deadline', '&lt;', datetime.datetime.combine(context_today() + relativedelta(days=1,weekday=0), datetime.time(0,0,0)).to_utc()),
                        ]"/>
                        <filter name="deadline_today" string="Today" domain="[
                            ('date_deadline', '&gt;=', datetime.datetime.combine(context_today(), datetime.time(0,0,0)).to_utc()),
                            ('date_deadline', '&lt;=', datetime.datetime.combine(context_today(), datetime.time(23,59,59)).to_utc()),
                        ]"/>
                        <filter name="deadline_past_due" string="Overdue" domain="[('date_deadline', '&lt;', datetime.datetime.combine(context_today(), datetime.time(0,0,0)).to_utc())]"/>
                    </filter>
                </filter>
                <filter name="last_stage_update" position="after">
                    <filter string="Deadline" name="deadline" context="{'group_by': 'date_deadline'}"/>
                    <separator/>
                    <filter string="Properties" name="group_by_properties" context="{'group_by': 'task_properties'}"/>
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

        <!-- Task graph view when viewing records of a single project -->
        <record id="view_project_task_graph_inherit" model="ir.ui.view">
            <field name="name">project.task.graph.inherit</field>
            <field name="model">project.task</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="project.view_project_task_graph"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='project_id']" position="attributes">
                    <attribute name="invisible">True</attribute>
                </xpath>
                <xpath expr="//field[@name='stage_id']" position="after">
                    <field name="user_ids"/>
                </xpath>
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
            <field name="path">tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,list,form,calendar,pivot,graph,activity</field>
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

        <record id="project_embedded_action_project_updates" model="ir.embedded.actions">
            <field name="parent_res_model">project.project</field>
            <field name="sequence">110</field>
            <field name="name">Dashboard</field>
            <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
            <field name="action_id" ref="project.project_update_all_action"/>
        </record>

        <record id="project_embedded_action_all_tasks_dashboard" model="ir.embedded.actions">
            <field name="parent_res_model">project.project</field>
            <field name="sequence">20</field>
            <field name="name">Tasks</field>
            <field name="parent_action_id" ref="project.project_update_all_action"/>
            <field name="action_id" ref="project.act_project_project_2_project_task_all"/>
        </record>

    <!-- Set pivot view and arrange in order -->
    <record id="project_task_kanban_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="project.act_project_project_2_project_task_all"/>
    </record>

    <record id="project_task_tree_action_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="20"/>
        <field name="view_mode">list</field>
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
        <field name="view_id" ref="view_project_task_graph_inherit"/>
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
            <field name="view_mode">list,kanban,form,calendar,pivot,graph,activity</field>
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
            <field name="context">{
                'default_composition_mode': 'mass_mail',
            }</field>
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
                    <field name="is_closed" invisible="1"/>
                    <field name="depend_on_count" invisible="1"/>
                    <field name="closed_depend_on_count" invisible="1"/>
                    <field name="html_field_history_metadata" invisible="1"/>
                    <field name="show_display_in_project" invisible="1"/>
                    <header>
                        <field name="stage_id" widget="statusbar_duration" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="not project_id and not stage_id"/>
                        <field name="state" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="1"/>
                        <field name="personal_stage_type_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="project_id" domain="[('user_id', '=', uid)]" string="Personal Stage"/>
                    </header>
                    <!-- Used in inherited views to display the eventual warnings -->
                    <t name="warning_section"/>
                    <sheet string="Task">
                    <div class="oe_button_box" name="button_box" groups="base.group_user">
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
                        <button name="action_open_parent_task" type="object" class="oe_stat_button" icon="fa-check" invisible="not parent_id">
                            <div class="o_stat_info">
                                <span class="o_stat_text">Parent Task</span>
                            </div>
                        </button>
                        <button name="action_recurring_tasks" type="object" invisible="not active or not recurrence_id" class="oe_stat_button" icon="fa-repeat" groups="project.group_project_recurring_tasks">
                            <field name="recurring_count" widget="statinfo" string="Recurring Tasks"/>
                        </button>
                        <button name="%(project_task_action_sub_task)d" type="action" class="oe_stat_button" icon="fa-check"
                            invisible="not id or subtask_count == 0"
                            context="{
                                'default_project_id': project_id,
                                'default_display_in_project': False,
                                'default_user_ids': user_ids,
                                'default_milestone_id': milestone_id,
                                'subtask_action': True,
                            }"
                        >
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Sub-tasks</span>
                                <span class="o_stat_value">
                                    <field name="closed_subtask_count"/> / <field name="subtask_count"/>
                                    (<field name="subtask_completion_percentage" widget="percentage" options="{'digits': [1, 0]}"/>)
                                </span>
                            </div>
                        </button>
                        <button name="action_dependent_tasks" type="object" invisible="dependent_tasks_count == 0" class="oe_stat_button" icon="fa-check" groups="project.group_project_task_dependencies">
                            <field name="dependent_tasks_count" widget="statinfo" string="Blocked Tasks" />
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
                            <label for="project_id"/>
                            <div name="project" class="d-inline-flex w-100">
                                <field name="project_id"
                                       domain="[('active', '=', True), '|', ('company_id', '=', False), ('company_id', '=?', company_id)]"
                                       widget="project"
                                />
                                <field name="display_in_project" string="Display the sub-task in your pipeline"
                                       widget="boolean_icon" options="{'icon': 'fa-eye-slash'}"
                                       class="ms-0" style="width: fit-content;" force_save="1"
                                       invisible="display_in_project or not show_display_in_project"
                                />
                                <field name="display_in_project" string="Hide the sub-task in your pipeline"
                                       widget="boolean_icon" options="{'icon': 'fa-eye'}"
                                       class="ms-0" style="width: fit-content;" force_save="1"
                                       invisible="not display_in_project or not show_display_in_project"
                                />
                            </div>
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
                            <div invisible="not recurring_task" class="d-flex" groups="project.group_project_recurring_tasks" name="repeat_intervals">
                                <field name="repeat_interval" required="recurring_task"
                                       class="me-2" style="max-width: 2rem !important;" />
                                <field name="repeat_unit" required="recurring_task"
                                       class="me-2" style="max-width: 4rem !important;" />
                                <field name="repeat_type" required="recurring_task"
                                       class="me-2" style="max-width: 15rem !important;" />
                                <field name="repeat_until" invisible="repeat_type != 'until'" required="repeat_type == 'until'"
                                       class="me-2" />
                            </div>
                            <!-- Field needed to trigger its compute in project_enterprise, but will be replaced in an override defined in hr_timesheet module -->
                            <field name="allocated_hours" invisible="1"/>
                        </group>
                    </group>
                        <field name="task_properties" columns="2"/>
                    <notebook>
                        <page name="description_page" string="Description">
                            <field name="description" type="html" options="{'collaborative': true, 'resizable': false}" placeholder="Add details about this task..."/>
                        </page>
                        <page name="sub_tasks_page" string="Sub-tasks" invisible="not project_id">
                            <field name="child_ids"
                                   mode="list,kanban"
                                   context="{
                                        'default_project_id': project_id,
                                        'default_display_in_project': False,
                                        'default_user_ids': user_ids,
                                        'default_parent_id': id,
                                        'default_partner_id': partner_id,
                                        'default_milestone_id': allow_milestones and milestone_id,
                                        'kanban_view_ref': 'project.project_sub_task_view_kanban_mobile',
                                        'closed_X2M_count': closed_subtask_count,
                                   }"
                                   widget="subtasks_one2many">
                                <list editable="bottom" decoration-muted="state in ['1_done','1_canceled']" open_form_view="True">
                                    <field name="allow_milestones" column_invisible="True"/>
                                    <field name="display_in_project" column_invisible="True" force_save="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="id" optional="hide" options="{'enable_formatting': False}"/>
                                    <field name="parent_id" column_invisible="True"/>
                                    <field name="priority" widget="priority" nolabel="1" width="20px"/>
                                    <field name="state" widget="project_task_state_selection" nolabel="1" width="20px"/>
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
                                </list>
                            </field>
                        </page>
                        <page name="task_dependencies" string="Blocked By" invisible="not allow_task_dependencies" groups="project.group_project_task_dependencies">
                            <field name="depend_on_ids"
                                   nolabel="1"
                                   mode="list,kanban"
                                   context="{
                                        'default_project_id': project_id,
                                        'search_view_ref' : 'project.view_task_search_form',
                                        'search_default_project_id': project_id,
                                        'list_view_ref': 'project.open_view_all_tasks_list_view',
                                        'search_default_open_tasks': 1,
                                        'kanban_view_ref': 'project.project_sub_task_view_kanban_mobile',
                                        'closed_X2M_count': closed_depend_on_count,
                                   }"
                                   widget="notebook_task_one2many">
                                <list editable="bottom" decoration-muted="state in ['1_done','1_canceled']" open_form_view="True">
                                    <field name="allow_milestones" column_invisible="True"/>
                                    <field name="parent_id" column_invisible="True" />
                                    <field name="subtask_count" column_invisible="True"/>
                                    <field name="closed_subtask_count" column_invisible="True"/>
                                    <field name="id" optional="hide" options="{'enable_formatting': False}"/>
                                    <field name="priority" widget="priority" nolabel="1" width="20px"/>
                                    <field name="state" widget="project_task_state_selection" nolabel="1" width="20px"/>
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
                                </list>
                            </field>
                        </page>
                        <page name="extra_info" string="Extra Info" groups="base.group_no_one">
                            <group>
                                <group>
                                    <field name="parent_id" groups="base.group_no_one" context="{'search_view_ref' : 'project.view_task_search_form','search_default_project_id': project_id, 'search_default_open_tasks': 1}"/>
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
                    <chatter reload_on_follower="True"/>
                </form>
            </field>
        </record>

        <record id="portal_share_action" model="ir.actions.act_window">
            <field name="name">Share Task</field>
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
                    highlight_color="color"
                    default_group_by="stage_id"
                    class="o_kanban_small_column o_kanban_project_tasks"
                    on_create="quick_create"
                    quick_create_view="project.quick_create_task_form"
                    examples="project"
                    js_class="project_task_kanban" sample="1"
                    default_order="priority desc, sequence, state, date_deadline asc, id desc"
                >
                    <field name="stage_id"/>
                    <field name="rating_count"/>
                    <field name="rating_avg"/>
                    <field name="rating_active"/>
                    <field name="has_late_and_unreached_milestone" />
                    <field name="allow_milestones" />
                    <field name="state" />
                    <field name="subtask_count"/>
                    <progressbar field="state" colors='{"1_done": "success-done", "1_canceled": "danger", "03_approved": "success", "02_changes_requested": "warning", "04_waiting_normal": "info", "01_in_progress": "200" }'/>
                    <templates>
                        <t t-name="menu" t-if="!selection_mode" groups="base.group_user">
                            <a t-if="widget.editable" role="menuitem" type="set_cover" class="dropdown-item" data-field="displayed_image_id">Set Cover Image</a>
                            <a name="%(portal.portal_share_action)d" role="menuitem" type="action" class="dropdown-item" context="{'dialog_size': 'medium'}">Share Task</a>
                            <a class="dropdown-item" role="menuitem" type="object" name="copy">Duplicate</a>
                            <div role="separator" class="dropdown-divider"></div>
                            <field name="color" widget="kanban_color_picker"/>
                        </t>
                        <t t-name="card">
                            <main t-att-class="['1_done', '1_canceled'].includes(record.state.raw_value) ? 'opacity-50' : ''">
                                <field name="name" class="fw-bold fs-5"/>
                                <div class="text-muted d-flex flex-column">
                                    <field t-if="record.parent_id.raw_value" invisible="context.get('default_parent_id', False)" name="parent_id"/>
                                    <field invisible="context.get('default_project_id', False)" name="project_id" options="{'no_open': True}"/>
                                    <field name="partner_id"/>    
                                    <field t-if="record.allow_milestones.raw_value and record.milestone_id.raw_value" t-att-class="record.has_late_and_unreached_milestone.raw_value and !record.state.raw_value.startsWith('1_') ? 'text-danger' : ''" name="milestone_id" options="{'no_open': True}"/>
                                </div>
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                <field t-if="record.date_deadline.raw_value" invisible="state in ['1_done', '1_canceled']" name="date_deadline" widget="remaining_days"/>
                                <field name="task_properties" widget="properties"/>
                                <field name="displayed_image_id" widget="attachment_image"/>
                                <footer t-if="!selection_mode" class="pt-1">
                                    <div class="d-flex align-items-center gap-1">
                                        <field name="priority" widget="priority" style="margin-right: 5px;"/>
                                        <field name="activity_ids" widget="kanban_activity" style="margin-right: 2px"/>
                                        <b t-if="record.rating_active.raw_value and record.rating_count.raw_value &gt; 0" groups="project.group_project_rating">
                                            <span class="fa fa-fw fa-smile-o text-success rating_face" t-if="record.rating_avg.raw_value &gt;= 3.66" title="Average Rating: Satisfied" role="img" aria-label="Happy face"/>
                                            <span class="fa fa-fw fa-meh-o text-warning rating_face" t-elif="record.rating_avg.raw_value &gt;= 2.33" title="Average Rating: Okay" role="img" aria-label="Neutral face"/>
                                            <span class="fa fa-fw fa-frown-o text-danger rating_face" t-else="" title="Average Rating: Dissatisfied" role="img" aria-label="Sad face"/>
                                        </b>
                                        <a t-if="!record.project_id.raw_value" class="text-muted" style="font-size: 17px; margin-left: 1.5px">
                                            <i title="Private Task" class="fa fa-lock"/>
                                        </a>
                                        <t t-if="record.project_id.raw_value and record.subtask_count.raw_value">
                                            <widget name="subtask_counter" class="me-1"/>
                                        </t>
                                    </div>
                                    <div class="d-flex ms-auto">
                                        <field name="user_ids" widget="many2many_avatar_user"/>
                                        <field name="state" class="ms-1" widget="project_task_state_selection" options="{'is_toggle_mode': false}"/>
                                    </div>
                                </footer>
                            </main>
                            <widget name="subtask_kanban_list"/>
                        </t>
                    </templates>
                </kanban>
            </field>
         </record>

        <!-- View used in the Sub-tasks, Blocked by notebook page in the task form view in mobile -->
        <record id="project_sub_task_view_kanban_mobile" model="ir.ui.view">
            <field name="name">project.task.kanban</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='parent_id']" position="after">
                    <field invisible="project_id == context.get('active_id', False)" class="mt-1" name="project_id" options="{'no_open': True}"/>
                </xpath>
                <xpath expr="//field[@name='parent_id']" position="replace"/>
            </field>
        </record>

        <!-- The main base of the project.task list view -->
        <record id="project_task_view_tree_main_base" model="ir.ui.view">
            <field name="name">project.task.view.list.main.base</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <list string="Tasks" sample="1" default_order="priority desc, sequence, state, date_deadline asc, id desc">
                    <field name="sequence" readonly="1" column_invisible="True"/>
                    <field name="allow_milestones" column_invisible="True"/>
                    <field name="subtask_count" column_invisible="True"/>
                    <field name="closed_subtask_count" column_invisible="True"/>
                    <field name="id" optional="hide" options="{'enable_formatting': False}"/>
                    <field name="priority" widget="priority" nolabel="1" width="20px"/>
                    <field name="state" widget="project_task_state_selection" nolabel="1" width="20px" options="{'is_toggle_mode': false}"/>
                    <field name="name" string="Title" widget="name_with_subtask_count"/>
                    <field name="project_id" widget="project" optional="show" options="{'no_open': 1}" readonly="1" column_invisible="context.get('default_project_id')"/>
                    <field name="milestone_id" invisible="not allow_milestones" context="{'default_project_id': project_id}" groups="project.group_project_milestone" optional="hide"/>
                    <field name="partner_id" optional="hide" widget="res_partner_many2one" invisible="not project_id" options="{'no_open': True}"/>
                    <field name="user_ids" optional="show" widget="many2many_avatar_user"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show" column_invisible="context.get('default_project_id')" options="{'no_create': True}"/>
                    <field name="company_id" column_invisible="True"/>
                    <field name="date_deadline" optional="hide" widget="remaining_days" invisible="state in ['1_done', '1_canceled']"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show" context="{'project_id': project_id}"/>
                    <field name="date_last_stage_update" optional="hide"/>
                    <field name="stage_id" column_invisible="context.get('set_visible', False)" optional="show"/>
                </list>
            </field>
        </record>

        <!-- common base of the list view in project app -->
        <record id="project_task_view_tree_base" model="ir.ui.view">
            <field name="name">project.task.view.list.base</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project_task_view_tree_main_base"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <list position="attributes">
                    <attribute name="multi_edit">1</attribute>
                </list>
                <field name="date_deadline" position="after">
                    <field name="activity_ids" string="Next Activity" widget="list_activity" optional="show"/>
                    <field name="my_activity_date_deadline" string="My Deadline" widget="remaining_days" optional="hide"/>
                    <field name="rating_active" column_invisible="True"/>
                    <field name="rating_last_text" string="Rating" decoration-danger="rating_last_text == 'ko'"
                        decoration-warning="rating_last_text == 'ok'" decoration-success="rating_last_text == 'top'"
                        invisible="not rating_active or rating_last_text == 'none'"
                        class="fw-bold" widget="badge" optional="hide" groups="project.group_project_rating"/>
                </field>
                <list position="inside">
                    <field name="task_properties"/>
                    <field name="recurrence_id" column_invisible="True" />
                </list>
                <xpath expr="//field[@name='partner_id']" position="after">
                    <field name="parent_id" optional="hide" context="{'search_view_ref': 'project.view_task_search_form', 'search_default_project_id': project_id}" groups="base.group_no_one"/>
                </xpath>
                <xpath expr="//field[@name='stage_id']" position="after">
                    <field name="personal_stage_type_id" string="Personal Stage" optional="hide"/>
                </xpath>
            </field>
        </record>

        <!-- main list view -->
        <record id="view_task_tree2" model="ir.ui.view">
            <field name="name">project.task.list</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project_task_view_tree_base"/>
            <field name="mode">primary</field>
            <field name="priority">2</field>
            <field name="arch" type="xml">
                <list position="attributes">
                    <attribute name="js_class">project_task_list</attribute>
                    <attribute name="default_group_by">stage_id</attribute>
                </list>
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
                          scales="day,week,month,year">
                    <field name="allow_milestones" invisible="1" />
                    <field name="project_id" widget="project" invisible="context.get('default_project_id', False)"/>
                    <field name="display_in_project" invisible="1"/>
                    <field name="subtask_count" invisible="1"/>
                    <field name="milestone_id" invisible="not allow_milestones or not milestone_id"/>
                    <field name="user_ids" widget="many2many_avatar_user" invisible="not user_ids"/>
                    <field name="partner_id" invisible="not partner_id"/>
                    <field name="priority" widget="priority"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" invisible="not tag_ids"/>
                    <field name="stage_id" invisible="not project_id or not stage_id" widget="task_stage_with_state_selection"/>
                    <field name="personal_stage_id" string="Personal Stage" options="{'no_open': True}" invisible="project_id or not personal_stage_id"/>
                    <field name="task_properties" invisible="not task_properties"/>
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


        <record id="project_task_view_activity" model="ir.ui.view">
            <field name="name">project.task.activity</field>
            <field name="model">project.task</field>
            <field name="arch" type="xml">
                <activity string="Project Tasks">
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
            <field name="view_mode">kanban,list,form,calendar,pivot,graph,activity</field>
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
                <list position="attributes">
                    <attribute name="default_group_by">personal_stage_type_id</attribute>
                </list>
            </field>
        </record>

        <record id="open_view_all_tasks_list_view" model="ir.ui.view">
            <field name="name">open.view.all.tasks.list.view</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_tree2"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <list position="attributes">
                    <attribute name="default_group_by"/>
                </list>
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
            <field name="path">my-tasks</field>
            <field name="view_mode">kanban,list,form,calendar,pivot,graph,activity</field>
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
            <field name="view_mode">list</field>
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
            <field name="view_mode">list</field>
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
            <field name="path">all-tasks</field>
            <field name="view_mode">list,kanban,form,calendar,pivot,graph,activity</field>
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
            <field name="view_mode">list</field>
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

        <!-- Views for 'Tasks' stat button via Contact form -->
        <record id="view_task_form_res_partner" model="ir.ui.view">
            <field name="name">project.task.form.res.partner</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_form2"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//group//field[@name='project_id']" position="attributes">
                    <attribute name="required">1</attribute>
                </xpath>
            </field>
        </record>

        <record id="quick_create_task_form_res_partner" model="ir.ui.view">
            <field name="name">project.task.form.quick_create.res.partner</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="quick_create_task_form"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//group//field[@name='project_id']" position="attributes">
                    <attribute name="required">1</attribute>
                    <attribute name="placeholder"></attribute>
                </xpath>
            </field>
        </record>

        <record id="view_task_kanban_res_partner" model="ir.ui.view">
            <field name="name">project.task.kanban.res.partner</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="view_task_kanban_inherit_all_task"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="quick_create_view">project.quick_create_task_form_res_partner</attribute>
                </xpath>
            </field>
        </record>

        <record id="project_task_action_from_partner" model="ir.actions.act_window">
            <field name="name">Partner's Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">list,kanban,form,calendar,pivot,graph,activity</field>
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

        <record id="project_task_action_from_partner_tree_view" model="ir.actions.act_window.view">
            <field name="view_mode">list</field>
            <field name="act_window_id" ref="project_task_action_from_partner"/>
            <field name="view_id" ref="open_view_all_tasks_list_view"/>
        </record>

        <record id="project_task_action_from_partner_kanban_view" model="ir.actions.act_window.view">
            <field name="view_mode">kanban</field>
            <field name="act_window_id" ref="project_task_action_from_partner"/>
            <field name="view_id" ref="view_task_kanban_res_partner"/>
        </record>

        <record id="project_task_action_from_partner_form_view" model="ir.actions.act_window.view">
            <field name="view_mode">form</field>
            <field name="act_window_id" ref="project_task_action_from_partner"/>
            <field name="view_id" ref="view_task_form_res_partner"/>
        </record>

        <record id="project_task_action_from_partner_calendar_view" model="ir.actions.act_window.view">
            <field name="view_mode">calendar</field>
            <field name="act_window_id" ref="project_task_action_from_partner"/>
            <field name="view_id" ref="view_task_all_calendar"/>
        </record>

        <record id="action_view_task_overpassed_draft" model="ir.actions.act_window">
            <field name="name">Overpassed Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">list,form,calendar,graph,kanban</field>
            <field name="domain">[('is_closed', '=', False), ('date_deadline','&lt;',time.strftime('%Y-%m-%d')), ('project_id', '!=', False), ('display_in_project', '=', True)]</field>
            <field name="filter" eval="True"/>
            <field name="search_view_id" ref="view_task_search_form"/>
        </record>

        <!-- Opening task when double clicking on project -->
        <record id="dblc_proj" model="ir.actions.act_window">
            <field name="res_model">project.task</field>
            <field name="name">Project's tasks</field>
            <field name="view_mode">list,form,calendar,graph,kanban</field>
            <field name="domain">[('project_id', '=', active_id), ('display_in_project', '=', True)]</field>
            <field name="context">{'project_id':active_id}</field>
        </record>

        <record id="action_view_task_from_milestone" model="ir.actions.act_window">
            <field name="name">Tasks</field>
            <field name="res_model">project.task</field>
            <field name="view_mode">kanban,list,calendar,pivot,graph,activity,form</field>
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

</odoo>

```

## File: views\project_update_templates.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="project.milestone_deadline">
<t t-if="milestone['deadline']">
(due <t t-out="milestone['deadline']" t-options='{"widget": "date"}'/><t t-if="not milestone['is_reached'] or not milestone['reached_date']">
<t t-if="milestone['can_be_marked_as_done']"> - <font t-att-style="'color: rgb(0, ' + str(color_level) + ', 0)'">ready to be marked as reached</font></t>)</t><t t-else=""> - reached on<t t-if="milestone['reached_date'] &gt; milestone['deadline']">
<font style="color: rgb(255, 0, 0)"><t t-out="milestone['reached_date']" t-options='{"widget": "date"}'/></font>)</t><t t-else="">
<font style="color: rgb(0, 128, 0)"><t t-out="milestone['reached_date']" t-options='{"widget": "date"}'/></font>)</t></t>
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
<t t-out="milestone['name']"/>
<span t-if="milestone['is_deadline_exceeded']"><font style="color: rgb(255, 0, 0);"><t t-call="project.milestone_deadline"/></font></span>
<span t-elif="not milestone['can_be_marked_as_done']"><font style="color: rgb(190, 190, 190);"><t t-set="color_level" t-value="64"/><t t-call="project.milestone_deadline"/></font></span>
<span t-else=""><t t-set="color_level" t-value="128"/><t t-call="project.milestone_deadline"/></span>
</li>
</t>
</ul>

<t t-if="milestones['updated']">
<t t-if="milestones['last_update_date']">Since <t t-out="milestones['last_update_date']" t-options='{"widget": "date"}'/> (last project update), </t>
<t t-if="len(milestones['updated']) > 1">the deadline for the following milestones has been updated:</t>
<t t-else="">the deadline for the following milestone has been updated:</t>
<ul>
<t t-foreach="milestones['updated']" t-as="milestone">
<li>
<t t-out="milestone['name']"/> (<t t-out="milestone['old_value']" t-options='{"widget": "date"}'/> =&gt; <t t-out="milestone['new_value']"  t-options='{"widget": "date"}'/>)
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
<t t-out="milestone['name']"/>
<span t-if="milestone['is_deadline_exceeded']"><font style="color: rgb(255, 0, 0);"><t t-call="project.milestone_deadline"/></font></span>
<span t-else=""><font style="color: rgb(190, 190, 190);"><t t-set="color_level" t-value="128"/><t t-call="project.milestone_deadline"/></font></span>
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
                <chatter class="d-print-none" reload_on_follower="True"/>
            </form>
        </field>
    </record>

    <record id="project_update_view_kanban" model="ir.ui.view">
        <field name="name">project.update.view.kanban</field>
        <field name="model">project.update</field>
        <field name="arch" type="xml">
            <kanban highlight_color="color" sample="1" js_class="project_update_kanban">
                <field name="color"/>
                <templates>
                    <t t-name="card" class="flex-row align-items-center">
                        <!-- Project Update Kanban View is always ungrouped - see js_class -->
                        <div class="pb-0 mb-2 mr8 o_pupdate_kanban_width">
                            <field name="name_cropped" class="fw-bolder"/>
                            <div class="d-flex gap-1">
                                <field name="user_id" widget="many2one_avatar_user"/>
                                <field name="user_id"/>
                            </div>
                        </div>
                        <div class="mr8 text-sm-start o_pupdate_kanban_width">
                            <field name="color" invisible="1"/>
                            <field name="status" class="fw-bolder" readonly="1" widget="status_with_color"/>
                        </div>
                        <div class="mr8 o_pupdate_kanban_width">
                            <field name="progress_percentage" class="fw-bolder" widget="percentage"/>
                            <div>Progress</div>
                        </div>
                        <div class="mr8 o_pupdate_kanban_width">
                            <b id="tasks_stats">
                                <field name="closed_task_count"/> / <field name="task_count"/> Tasks<span invisible="not task_count"> (<field name="closed_task_percentage"/>%)</span>
                            </b>
                        </div>
                        <div class="mr8 o_pupdate_kanban_width">
                            <field name="date" class="fw-bolder"/>
                            <div>Date</div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="project_update_view_tree" model="ir.ui.view">
        <field name="name">project.update.view.list</field>
        <field name="model">project.update</field>
        <field name="arch" type="xml">
            <list sample="1" js_class="project_update_list">
                <field name="name"/>
                <field name="user_id" widget="many2one_avatar_user" class="fw-bolder" optional="show"/>
                <field name="date" optional="show"/>
                <field name="progress" string="Progress" widget="progressbar" optional="show"/>
                <field name="color" column_invisible="True"/>
                <field name="status" widget="status_with_color"/>
            </list>
        </field>
    </record>

    <record id="project_update_all_action" model="ir.actions.act_window">
        <field name="name">Dashboard</field>
        <field name="res_model">project.update</field>
        <field name="path">project-dashboard</field>
        <field name="view_mode">kanban,list,form</field>
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
        <field name="name">rating.rating.list.project</field>
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
            <xpath expr="//field[@name='create_date']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='rating']" position="attributes">
                <attribute name="string">Rating (1-5)</attribute>
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
            <xpath expr="//field[@name='rating']" position="attributes">
                <attribute name="string">Rating (1-5)</attribute>
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
                <filter name="filter_write_date" string="Submitted On" date="write_date" default_period="custom_write_date_last_30_days">
                    <filter name="write_date_last_7_days" string="Last 7 Days" domain="[('write_date', '&gt;', datetime.datetime.combine(context_today() - datetime.timedelta(days=7), datetime.time(23, 59, 59)).to_utc())]"/>
                    <filter name="write_date_last_30_days" string="Last 30 Days" domain="[('write_date', '&gt;', datetime.datetime.combine(context_today() - datetime.timedelta(days=30), datetime.time(23, 59, 59)).to_utc())]"/>
                    <filter name="write_date_last_365_days" string="Last 365 Days" domain="[('write_date', '&gt;', datetime.datetime.combine(context_today() - datetime.timedelta(days=365), datetime.time(23, 59, 59)).to_utc())]"/>
                </filter>
            </xpath>
            <xpath expr="//filter[@name='month']" position="attributes">
                <attribute name="context">{'group_by':'write_date:month'}</attribute>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_action_view_project_rating" model="ir.actions.act_window">
        <field name="name">Ratings</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,list,graph,pivot,form</field>
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
        <field name="view_mode">list</field>
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
        <field name="view_mode">kanban,list,pivot,graph,form</field>
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
        <field name="view_mode">list</field>
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
        <field name="path">task-ratings</field>
        <field name="view_mode">kanban,list,pivot,graph,form</field>
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
            'search_default_filter_write_date': 1,
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
        <field name="view_mode">list</field>
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
                    <button class="oe_stat_button" type="object" name="action_view_tasks" groups="project.group_project_user" icon="fa-tasks">
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

    stage_ids = fields.Many2many('project.project.stage', string='Stages To Delete', ondelete='cascade', context={'active_test': False}, export_string_translation=False)
    projects_count = fields.Integer('Number of Projects', compute='_compute_projects_count', export_string_translation=False)
    stages_active = fields.Boolean(compute='_compute_stages_active', export_string_translation=False)

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

## File: wizard\project_share_collaborator_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProjectSharingCollaboratorWizard(models.TransientModel):
    _name = 'project.share.collaborator.wizard'
    _description = 'Project Sharing Collaborator Wizard'

    parent_wizard_id = fields.Many2one(
        'project.share.wizard',
        export_string_translation=False,
    )
    partner_id = fields.Many2one(
        'res.partner',
        string='Collaborator',
        required=True,
    )
    access_mode = fields.Selection(
        [('read', 'Read'), ('edit_limited', 'Edit with limited access'), ('edit', 'Edit')],
        default='read',
        required=True,
        help="Read: collaborators can view tasks but cannot edit them.\n"
            "Edit with limited access: collaborators can view and edit tasks they follow in the Kanban view.\n"
            "Edit: collaborators can view and edit all tasks in the Kanban view. Additionally, they can choose which tasks they want to follow."
    )
    send_invitation = fields.Boolean(
        string='Send Invitation',
        compute='_compute_send_invitation',
        store=True,
        readonly=False,
        default=True,
    )

    @api.depends('partner_id', 'access_mode')
    def _compute_send_invitation(self):
        project = self.parent_wizard_id.resource_ref
        for collaborator in self:
            if (
                collaborator.partner_id not in project.message_partner_ids
                or (collaborator.access_mode != 'read' and collaborator.partner_id not in project.collaborator_ids.partner_id)
            ):
                collaborator.send_invitation = True

```

## File: wizard\project_share_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import operator

from odoo import Command, api, fields, models, _


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
        if result['res_model'] and result['res_id']:
            project = self.env[result['res_model']].browse(result['res_id'])
            collaborator_vals_list = []
            collaborator_ids = []
            for collaborator in project.collaborator_ids:
                collaborator_ids.append(collaborator.partner_id.id)
                collaborator_vals_list.append({
                    'partner_id': collaborator.partner_id.id,
                    'partner_name': collaborator.partner_id.display_name,
                    'access_mode': 'edit_limited' if collaborator.limited_access else 'edit',
                })
            for follower in project.message_partner_ids:
                if follower.partner_share and follower.id not in collaborator_ids:
                    collaborator_vals_list.append({
                        'partner_id': follower.id,
                        'partner_name': follower.display_name,
                        'access_mode': 'read',
                    })
            if collaborator_vals_list:
                collaborator_vals_list.sort(key=operator.itemgetter('partner_name'))
                result['collaborator_ids'] = [
                    Command.create({'partner_id': collaborator['partner_id'], 'access_mode': collaborator['access_mode'], 'send_invitation': False})
                    for collaborator in collaborator_vals_list
                ]
        return result

    @api.model
    def _selection_target_model(self):
        project_model = self.env['ir.model']._get('project.project')
        return [(project_model.model, project_model.name)]

    share_link = fields.Char("Public Link", help="Anyone with this link can access the project in read mode.")
    collaborator_ids = fields.One2many('project.share.collaborator.wizard', 'parent_wizard_id', string='Collaborators')
    existing_partner_ids = fields.Many2many('res.partner', compute='_compute_existing_partner_ids', export_string_translation=False)

    @api.depends('res_model', 'res_id')
    def _compute_resource_ref(self):
        for wizard in self:
            if wizard.res_model and wizard.res_model == 'project.project':
                wizard.resource_ref = '%s,%s' % (wizard.res_model, wizard.res_id or 0)
            else:
                wizard.resource_ref = None

    @api.depends('collaborator_ids')
    def _compute_existing_partner_ids(self):
        for wizard in self:
            wizard.existing_partner_ids = wizard.collaborator_ids.partner_id

    @api.model_create_multi
    def create(self, vals_list):
        wizards = super().create(vals_list)
        for wizard in wizards:
            collaborator_ids_to_add = []
            collaborator_ids_to_add_with_limited_access = []
            collaborator_ids_vals_list = []
            project = wizard.resource_ref
            project_collaborator_ids_to_remove = [
                c.id
                for c in project.collaborator_ids
                if c.partner_id not in wizard.collaborator_ids.partner_id
            ]
            project_followers = project.message_partner_ids
            project_followers_to_add = []
            project_followers_to_remove = [
                partner.id
                for partner in project_followers
                if partner not in wizard.collaborator_ids.partner_id
            ]
            project_collaborator_per_partner_id = {c.partner_id.id: c for c in project.collaborator_ids}
            for collaborator in wizard.collaborator_ids:
                partner_id = collaborator.partner_id.id
                project_collaborator = project_collaborator_per_partner_id.get(partner_id, self.env['project.collaborator'])
                if collaborator.access_mode in ("edit", "edit_limited"):
                    limited_access = collaborator.access_mode == "edit_limited"
                    if not project_collaborator:
                        if limited_access:
                            collaborator_ids_to_add_with_limited_access.append(partner_id)
                        else:
                            collaborator_ids_to_add.append(partner_id)
                    elif project_collaborator.limited_access != limited_access:
                        collaborator_ids_vals_list.append(
                            Command.update(
                                project_collaborator.id,
                                {'limited_access': limited_access},
                            )
                        )
                elif project_collaborator:
                    project_collaborator_ids_to_remove.append(project_collaborator.id)
                if partner_id not in project_followers.ids:
                    project_followers_to_add.append(partner_id)
            if collaborator_ids_to_add:
                partners = project._get_new_collaborators(self.env['res.partner'].browse(collaborator_ids_to_add))
                collaborator_ids_vals_list.extend(Command.create({'partner_id': partner_id}) for partner_id in partners.ids)
            if collaborator_ids_to_add_with_limited_access:
                partners = project._get_new_collaborators(self.env['res.partner'].browse(collaborator_ids_to_add_with_limited_access))
                collaborator_ids_vals_list.extend(
                    Command.create({'partner_id': partner_id, 'limited_access': True}) for partner_id in partners.ids
                )
            if project_collaborator_ids_to_remove:
                collaborator_ids_vals_list.extend(Command.delete(collaborator_id) for collaborator_id in project_collaborator_ids_to_remove)
            project_vals = {}
            if collaborator_ids_vals_list:
                project_vals['collaborator_ids'] = collaborator_ids_vals_list
            if project_vals:
                project.write(project_vals)
            if project_followers_to_add:
                project._add_followers(self.env['res.partner'].browse(project_followers_to_add))
            if project_followers_to_remove:
                project.message_unsubscribe(project_followers_to_remove)
        return wizards

    def action_share_record(self):
        # Confirmation dialog is only opened if new portal user(s) need to be created in a 'on invitation' website
        self.ensure_one()
        if not self.collaborator_ids:
            return
        on_invite = self.env['res.users']._get_signup_invitation_scope() == 'b2b'
        new_portal_user = self.collaborator_ids.filtered(lambda c: c.send_invitation and not c.partner_id.user_ids) and on_invite
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
        self.env['project.project'].browse(self.res_id).privacy_visibility = 'portal'
        result = {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'message': _("Project shared with your collaborators."),
                'next': {'type': 'ir.actions.act_window_close'},
            }}
        partner_ids_in_readonly_mode = []
        partner_ids_in_edit_mode = []
        for collaborator in self.collaborator_ids:
            if not collaborator.send_invitation:
                continue
            if collaborator.access_mode == 'read':
                partner_ids_in_readonly_mode.append(collaborator.partner_id.id)
            else:
                partner_ids_in_edit_mode.append(collaborator.partner_id.id)
        if partner_ids_in_edit_mode:
            new_collaborators = self.env['res.partner'].browse(partner_ids_in_edit_mode)
            portal_partners = new_collaborators.filtered('user_ids')
            # send mail to users
            self._send_public_link(portal_partners)
            self._send_signup_link(partners=new_collaborators.with_context({'signup_valid': True}) - portal_partners)
        if partner_ids_in_readonly_mode:
            self.partner_ids = self.env['res.partner'].browse(partner_ids_in_readonly_mode)
            super().action_send_mail()
        return result

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
                <group>
                    <field name="share_link" widget="CopyClipboardChar"/>
                </group>
                <field name="collaborator_ids" nolabel="1">
                    <list string="Collaborators" editable="bottom">
                        <field name="partner_id"
                               options="{'no_create': True, 'no_open': True}"
                               domain="[('id', 'not in', parent.existing_partner_ids), ('partner_share', '=', True)]"
                               context="{'show_email': True}"
                        />
                        <field name="access_mode"/>
                        <field name="send_invitation"/>
                    </list>
                </field>
                <p class="text-muted">Choose one of the following access modes for your collaborators:</p>
                <ul class="text-muted">
                    <li>Read: collaborators can view tasks but cannot edit them.</li>
                    <li>Edit with limited access: collaborators can view and edit tasks they follow in the Kanban view.</li>
                    <li>Edit: collaborators can view and edit all tasks in the Kanban view. Additionally, they can choose which tasks they want to follow.</li>
                </ul>
                <footer>
                    <button string="Share Project" name="action_share_record" type="object" class="btn-primary" data-hotkey="q"/>
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
                <p>They can edit shared project tasks and view specific documents in read mode on your website. This includes leads/opportunities, quotations/sales orders, purchase orders, invoices and bills, timesheets, and tickets.</p>
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
    _description = 'Project Task Stage Delete Wizard'

    project_ids = fields.Many2many('project.project', domain="['|', ('active', '=', False), ('active', '=', True)]", string='Projects', ondelete='cascade', export_string_translation=False)
    stage_ids = fields.Many2many('project.task.type', string='Stages To Delete', ondelete='cascade', export_string_translation=False)
    tasks_count = fields.Integer('Number of Tasks', compute='_compute_tasks_count', export_string_translation=False)
    stages_active = fields.Boolean(compute='_compute_stages_active', export_string_translation=False)

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
                        <list>
                            <field name="name"/>
                        </list>
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
from . import project_share_collaborator_wizard

```

