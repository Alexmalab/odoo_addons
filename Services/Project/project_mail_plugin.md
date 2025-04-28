# Odoo Module: project_mail_plugin

Category: Services/Project

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Project Mail Plugin',
    'version': '1.0',
    'category': 'Services/Project',
    'sequence': 5,
    'summary': 'Integrate your inbox with projects',
    'description': "Turn emails received in your mailbox into tasks and log their content as internal notes.",
    'data': [
        'views/project_task_views.xml'
    ],
    'website': 'https://www.odoo.com/app/project',
    'depends': [
        'project',
        'mail_plugin',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\mail_plugin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo.http import request

from odoo.addons.mail_plugin.controllers import mail_plugin

_logger = logging.getLogger(__name__)


class MailPluginController(mail_plugin.MailPluginController):

    def _get_contact_data(self, partner):
        """
        Overrides the base module's get_contact_data method by Adding the "tasks" key within the initial contact
        information dict loaded when opening an email on Outlook.
        This is structured this way to enable the "project" feature on the Outlook side only if the Odoo version
        supports it.

        Return the tasks key only if the current user can create tasks. So, if they can not
        create tasks, the section won't be visible on the addin side (like if the project
        module was not installed on the database).
        """
        contact_values = super(MailPluginController, self)._get_contact_data(partner)

        if not request.env['project.task'].has_access('create'):
            return contact_values

        if not partner:
            contact_values['tasks'] = []
        else:
            partner_tasks = request.env['project.task'].search(
                [('partner_id', '=', partner.id)], offset=0, limit=5)

            accessible_projects = partner_tasks.project_id._filtered_access('read').ids

            tasks_values = [
                {
                    'task_id': task.id,
                    'name': task.name,
                    'project_name': task.project_id.name,
                } for task in partner_tasks if task.project_id.id in accessible_projects]

            contact_values['tasks'] = tasks_values
            contact_values['can_create_project'] = request.env['project.project'].has_access('create')

        return contact_values

    def _mail_content_logging_models_whitelist(self):
        models_whitelist = super(MailPluginController, self)._mail_content_logging_models_whitelist()
        if not request.env['project.task'].has_access('create'):
            return models_whitelist
        return models_whitelist + ['project.task']

    def _translation_modules_whitelist(self):
        modules_whitelist = super(MailPluginController, self)._translation_modules_whitelist()
        if not request.env['project.task'].has_access('create'):
            return modules_whitelist
        return modules_whitelist + ['project_mail_plugin']

```

## File: controllers\project_client.py

```python

from odoo import Command, http, _
from odoo.http import request


class ProjectClient(http.Controller):
    @http.route('/mail_plugin/project/search', type='json', auth='outlook', cors="*")
    def projects_search(self, search_term, limit=5):
        """
        Used in the plugin side when searching for projects.
        Fetches projects that have names containing the search_term.
        """
        projects = request.env['project.project'].search([('name', 'ilike', search_term)], limit=limit)

        return [
            {
                'project_id': project.id,
                'name': project.name,
                'partner_name': project.partner_id.name,
                'company_id': project.company_id.id
            }
            for project in projects.sudo()
        ]

    @http.route('/mail_plugin/task/create', type='json', auth='outlook', cors="*")
    def task_create(self, email_subject, email_body, project_id, partner_id):
        partner = request.env['res.partner'].browse(partner_id).exists()
        if not partner:
            return {'error': 'partner_not_found'}

        if not request.env['project.project'].browse(project_id).exists():
            return {'error': 'project_not_found'}

        if not email_subject:
            email_subject = _('Task for %s', partner.name)

        record = request.env['project.task'].with_company(partner.company_id).create({
            'name': email_subject,
            'partner_id': partner_id,
            'description': email_body,
            'project_id': project_id,
            'user_ids': [Command.link(request.env.uid)],
        })

        return {'task_id': record.id, 'name': record.name}

    @http.route('/mail_plugin/project/create', type='json', auth='outlook', cors="*")
    def project_create(self, name):
        record = request.env['project.project'].create({'name': name})
        return {"project_id": record.id, "name": record.name}

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_plugin
from . import project_client

```

## File: static\src\to_translate\translations_gmail.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
    Terms to translate for the Gmail plugin
-->
<ressources>
    <string>Task</string>
    <string>Could not create the project</string>
    <string>Could not create the task</string>
    <string>Search</string>
    <string>Search a Project</string>
    <string>Create a Task in a new Project</string>
    <string>Create a Task in an existing Project</string>
    <string>Project Name</string>
    <string>Create Project &amp; Task</string>
    <string>Email already logged on the task</string>
    <string>Tasks (%s)</string>
    <string>Create</string>
    <string>OR</string>
    <string>Log the email on the task</string>
    <string>Save the contact to create new tasks.</string>
    <string>The project name is required</string>
    <string>No project found.</string>
    <string>The Contact needs to exist to create Task.</string>
    <string>No project</string>
    <string>There are no project in your database. Please ask your project manager to create one.</string>
</ressources>

```

## File: static\src\to_translate\translations_outlook.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--Terms to translate for the outlook plugin -->
<ressources>
    <string>Task</string>
    <string>Save Contact to create new Tasks.</string>
    <string>No tasks found for this contact.</string>
    <string>Tasks (%(count)s)</string>
    <string>Pick a Project to create a Task</string>
    <string>Create %(name)s</string>
    <string>Search Projects...</string>
    <string>Log Email Into Task</string>
    <string>The Contact needs to exist to create Task.</string>
    <string>No Project Found</string>
    <string>No project</string>
</ressources>

```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- action used by the mail plugins in order to redirect the user to the newly created task in edit mode-->
    <record id="project_task_action_form_edit" model="ir.actions.act_window">
      <field name="name">Task: redirect to form in edit mode</field>
      <field name="res_model">project.task</field>
      <field name="view_mode">form</field>
      <field name="view_id" ref="project.view_task_form2"/>
    </record>
</odoo>

```

