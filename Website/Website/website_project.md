# Odoo Module: website_project

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Online Task Submission',
    'category': 'Website/Website',
    'summary': 'Add a task suggestion form to your website',
    'version': '1.0',
    'description': """
Generate tasks in Project app from a form published on your website. This module requires the use of the *Form Builder* module in order to build the form.
    """,
    'depends': ['website', 'project'],
    'data': [
        'data/website_project_data.xml',
        'views/project_portal_project_task_template.xml',
        'views/project_portal_project_project_template.xml',
        ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'website.assets_wysiwyg': [
            'website_project/static/src/js/website_project_editor.js',
        ],
        'project.webclient': [
            # In website, there is a patch of the LinkDialog (see
            # website/static/src/js/editor/editor.js) that require the utils.js.
            # Thus, when website is installed, this bundle need to have the
            # utils.js in its assets, otherwise, there will be an unmet
            # dependency.
            'website/static/src/js/utils.js',
            'web/static/src/core/autocomplete/*',
            'website/static/src/components/autocomplete_with_pages/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request

from odoo import _
from odoo.addons.base.models.ir_qweb_fields import nl2br, nl2br_enclose
from odoo.addons.website.controllers import form
from odoo.tools import html2plaintext


class WebsiteForm(form.WebsiteForm):
    def insert_record(self, request, model, values, custom, meta=None):
        model_name = model.sudo().model
        if model_name == 'project.task':
            visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
            visitor_partner = visitor_sudo.partner_id
            if visitor_partner:
                values['partner_id'] = visitor_partner.id
            # When a task is created from the web editor, if the key 'user_ids' is not present, the user_ids is filled with the odoo bot. We set it to False to ensure it is not.
            values.setdefault('user_ids', False)

        res = super().insert_record(request, model, values, custom, meta=meta)
        if model_name != 'project.task':
            return res
        task = request.env['project.task'].sudo().browse(res)
        custom = custom.replace('email_from', _('Email'))
        custom_label = nl2br_enclose(_("Other Information"), 'h4')  # Title for custom fields
        default_field = model.website_form_default_field_id
        default_field_data = values.get(default_field.name, '')
        default_field_content = nl2br_enclose(default_field.name.capitalize(), 'h4') + nl2br_enclose(html2plaintext(default_field_data), 'p')
        custom_content = (default_field_content if default_field_data else '') \
                        + (custom_label + custom if custom else '') \
                        + (self._meta_label + meta if meta else '')

        if default_field.name:
            if default_field.ttype == 'html':
                custom_content = nl2br(custom_content)
            task[default_field.name] = custom_content
            task._message_log(
                body=custom_content,
                message_type='comment',
            )
        return res

    def extract_data(self, model, values):
        data = super().extract_data(model, values)
        if model.sudo().model == 'project.task' and values.get('email_from'):
            partners_list = request.env['mail.thread'].sudo()._mail_find_partner_from_emails([values['email_from']])
            partner = partners_list[0] if partners_list else self.env['res.partner']
            data['record']['partner_id'] = partner.id
            data['record']['email_from'] = values['email_from']
            if partner:
                if not partner.phone and values.get('partner_phone'):
                    data['record']['partner_phone'] = values['partner_phone']
                if not partner.name:
                    data['record']['partner_name'] = values['partner_name']
                if not partner.company_name and values.get('partner_company_name'):
                    data['record']['partner_company_name'] = values['partner_company_name']
            else:
                data['record']['email_cc'] = values['email_from']
                if values.get('partner_phone'):
                    data['record']['partner_phone'] = values['partner_phone']
                data['record']['partner_name'] = values['partner_name']
                if values.get('partner_company_name'):
                    data['record']['partner_company_name'] = values['partner_company_name']
        return data

```

## File: controllers\__init__.py

```python
from . import main

```

## File: data\website_project_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="project.model_project_task" model="ir.model">
            <field name="website_form_key">create_task</field>
            <field name="website_form_default_field_id" ref="project.field_project_task__description" />
            <field name="website_form_access">True</field>
            <field name="website_form_label">Create a Task</field>
        </record>

        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>project.task</value>
            <value eval="[
                'name',
                'partner_name',
                'partner_phone',
                'partner_company_name',
                'description',
                'project_id',
                'task_properties',
            ]"/>
        </function>

</odoo>

```

## File: models\project_task.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields


class ProjectTask(models.Model):
    _inherit = 'project.task'

    # Need this field to check there is no email loops when Odoo reply automatically
    email_from = fields.Char('Email From')
    # Used to submit tasks from a contact form
    partner_name = fields.Char(string='Customer Name', related="partner_id.name", store=True, readonly=False, tracking=False)
    partner_phone = fields.Char(
        compute='_compute_partner_phone', inverse='_inverse_partner_phone',
        string="Contact Number", readonly=False, store=True, copy=False)
    partner_company_name = fields.Char(string='Company Name', related="partner_id.company_name", store=True, readonly=False, tracking=False)

    @api.depends('partner_id.phone', 'partner_id.mobile')
    def _compute_partner_phone(self):
        for task in self:
            task.partner_phone = task.partner_id.mobile or task.partner_id.phone or False

    def _inverse_partner_phone(self):
        for task in self:
            if task.partner_id:
                if task.partner_id.mobile or not task.partner_id.phone:
                    task.partner_id.mobile = task.partner_phone
                else:
                    task.partner_id.phone = task.partner_phone

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_task

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="m30.842 34.612-8.105 10.387L5.452 31.37a3.862 3.862 0 0 1-.616-5.417l5.748-7.243 20.258 15.903Z" fill="#088BF5"/><path d="m22.623 44.909-10.455-8.335 8.128-10.242 10.547 8.28L22.738 45l-.115-.091Z" fill="#144496"/><path d="m22.593 44.886.144.114 22.447-28.767a3.862 3.862 0 0 0-.636-5.393L37.223 5 20.296 26.332l10.547 8.28-8.105 10.387-.144-.113Z" fill="#2EBCFA"/></svg>

```

## File: static\src\js\website_project_editor.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import FormEditorRegistry from "@website/js/form_editor_registry";

FormEditorRegistry.add('create_task', {
    formFields: [{
        type: 'char',
        required: true,
        fillWith: 'name',
        name: 'partner_name',
        string: _t('Full Name'),
    }, {
        type: 'tel',
        fillWith: 'phone',
        name: 'partner_phone',
        string: _t('Phone Number'),
    }, {
        type: 'email',
        custom: true,
        required: true,
        fillWith: 'email',
        name: 'email_from',
        string: _t('Email Address'),
    }, {
        type: 'char',
        fillWith: 'commercial_company_name',
        name: 'partner_company_name',
        string: _t('Company Name'),
    }, {
        type: 'char',
        modelRequired: true,
        name: 'name',
        string: _t('Message Subject'),
    }, {
        type: 'text',
        required: true,
        name: 'description',
        string: _t('Ask Your Question'),
    }, {
        type: 'binary',
        custom: true,
        name: _t('Attach File'),
    }],
    fields: [{
        name: 'project_id',
        type: 'many2one',
        relation: 'project.project',
        string: _t('Project'),
        createAction: 'project.open_view_project_all',
    }],
    successPage: '/your-task-has-been-submitted',
});

```

## File: views\project_portal_project_project_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="website_portal_my_project" name="My Project website" inherit_id="project.portal_my_project">
        <xpath expr="//t[@t-set='title']" position="replace">
            <t t-set="additional_title" t-value="project.name"/>
        </xpath>
    </template>
</odoo>

```

## File: views\project_portal_project_task_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="task_submitted" name="Task Submitted">
        <t t-call="website.layout">
            <div class="oe_structure oe_empty h-100">
                <div class="container d-flex flex-column justify-content-center h-100">
                    <div class="row justify-content-center mb16">
                        <t t-if="request.session.get('form_builder_model_model', '') == 'project.task'">
                            <t t-set="task" t-value="request.website._website_form_last_record()"/>
                        </t>
                        <h1 class="text-center">
                            <i class="fa fa-check-circle fa-1x text-success me-2" role="img" aria-label="Success" title="Success"/>
                                <t t-if="task">
                                    <span>
                                    Your Task Number is
                                        <a t-if="request.session.uid and task.sudo().project_id.id and task.project_privacy_visibility != 'followers'"
                                            t-attf-href="/my/task/#{task.id}">
                                            #<span t-field="task.id"/>
                                        </a>
                                        <t t-else="">#<span t-field="task.id"/></t>.
                                    </span>
                                </t>
                        </h1>
                        <h2 class="text-center">Thank you for contacting us, our team will get right on it!</h2>
                        <div class="text-center">
                            <a class="btn btn-primary" t-attf-href="/my/task/#{task.id}"
                                t-if="task and task.id and request.session.uid and task.project_id.id and task.project_privacy_visibility != 'followers'">
                                View Task
                            </a>
                            <a class="btn btn-primary" href='/'>Go to the Homepage</a>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <record id="task_submitted_page" model="website.page">
        <field name="is_published">True</field>
        <field name="url">/your-task-has-been-submitted</field>
        <field name="website_indexed" eval="False"/>
        <field name="view_id" ref="task_submitted"/>
    </record>

    <template id="website_portal_my_task" name="My Task website" inherit_id="project.portal_my_task">
        <xpath expr="//t[@t-set='title']" position="replace">
            <t t-set="additional_title" t-value="task.name"/>
        </xpath>
    </template>
</odoo>

```

