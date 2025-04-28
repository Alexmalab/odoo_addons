# Odoo Module: website_form_project

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import controllers

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
Generate tasks in Project app from a form published on your website. This module requires the use of the *Form Builder* module (available in Odoo Enterprise) in order to build the form.
    """,
    'depends': ['website', 'project'],
    'data': [
        'data/website_form_project_data.xml',
        'views/project_portal_project_task_template.xml',
        'views/project_portal_project_project_template.xml',
        ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'website.assets_wysiwyg': [
            'website_form_project/static/src/js/website_form_project_editor.js',
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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website.controllers import form


class WebsiteForm(form.WebsiteForm):
    def insert_record(self, request, model, values, custom, meta=None):
        if model.model == 'project.task':
            visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
            visitor_partner = visitor_sudo.partner_id
            if visitor_partner:
                values['partner_id'] = visitor_partner.id

        return super().insert_record(request, model, values, custom, meta=meta)

```

## File: controllers\__init__.py

```python
from . import main

```

## File: data\website_form_project_data.xml

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
                'description',
                'project_id',
            ]"/>
        </function>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="m30.842 34.612-8.105 10.387L5.452 31.37a3.862 3.862 0 0 1-.616-5.417l5.748-7.243 20.258 15.903Z" fill="#088BF5"/><path d="m22.623 44.909-10.455-8.335 8.128-10.242 10.547 8.28L22.738 45l-.115-.091Z" fill="#144496"/><path d="m22.593 44.886.144.114 22.447-28.767a3.862 3.862 0 0 0-.636-5.393L37.223 5 20.296 26.332l10.547 8.28-8.105 10.387-.144-.113Z" fill="#2EBCFA"/></svg>

```

## File: static\src\js\website_form_project_editor.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import FormEditorRegistry from "@website/js/form_editor_registry";

FormEditorRegistry.add('create_task', {
    formFields: [{
        type: 'char',
        modelRequired: true,
        name: 'name',
        string: _t('Task Title'),
    }, {
        type: 'email',
        custom: true,
        required: true,
        fillWith: 'email',
        name: 'email_from',
        string: _t('Your Email'),
    }, {
        type: 'char',
        name: 'description',
        string: _t('Description'),
    }],
    fields: [{
        name: 'project_id',
        type: 'many2one',
        relation: 'project.project',
        string: _t('Project'),
        createAction: 'project.open_view_project_all',
    }],
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
    <template id="website_portal_my_task" name="My Task website" inherit_id="project.portal_my_task">
        <xpath expr="//t[@t-set='title']" position="replace">
            <t t-set="additional_title" t-value="task.name"/>
        </xpath>
    </template>
</odoo>

```

