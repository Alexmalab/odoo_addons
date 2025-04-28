# Odoo Module: pad_project

Category: Services/Project

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Pad on tasks',
    'category': 'Services/Project',
    'description': """
This module adds a PAD in all project form views.
=================================================
    """,
    'depends': [
        'project',
        'pad'
    ],
    'data': [
        'views/res_config_settings_views.xml',
        'views/project_views.xml',
        'views/project_portal_templates.xml',
        'views/project_portal_assets.xml'
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProjectTask(models.Model):
    _name = "project.task"
    _inherit = ["project.task", 'pad.common']
    _description = 'Task'

    description_pad = fields.Char('Pad URL', pad_content_field='description', copy=False)
    use_pad = fields.Boolean(related="project_id.use_pads", string="Use collaborative pad", readonly=True)
    pad_availability = fields.Selection(
        related="project_id.pad_availability",
        string="Availability of collaborative pads",
        readonly=True)

    @api.onchange('use_pad')
    def _onchange_use_pads(self):
        """ Copy the content in the pad when the user change the project of the task to the one with no pads enabled.

            This case is when the use_pad becomes False and we have already generated the url pad,
            that is the description_pad field contains the url of the pad.
        """
        if not self.use_pad and self.description_pad:
            vals = {'description_pad': self.description_pad}
            self._set_pad_to_field(vals)
            self.description = vals['description']

    @api.model
    def create(self, vals):
        # When using quick create, the project_id is in the context, not in the vals
        project_id = vals.get('project_id', False) or self.default_get(['project_id']).get('project_id', False)
        if not self.env['project.project'].browse(project_id).use_pads:
            self = self.with_context(pad_no_create=True)
        return super(ProjectTask, self).create(vals)

    def _use_portal_pad(self):
        """
        Indicates if the task configuration requires to provide
        an access to a portal pad.
        """
        self.ensure_one()
        return self.use_pad and self.pad_availability == 'portal'

    def _get_pad_content(self):
        """
        Gets the content of the pad used to edit the task description
        and returns it.
        """
        self.ensure_one()
        return self.pad_get_content(self.description_pad)


class ProjectProject(models.Model):
    _name = "project.project"
    _inherit = ["project.project", 'pad.common']
    _description = 'Project'

    description_pad = fields.Char('Pad URL', pad_content_field='description', copy=False)
    use_pads = fields.Boolean("Use collaborative pads", default=True,
        help="Use collaborative pad for the tasks on this project.")

    pad_availability = fields.Selection([
        ('internal', 'Internal Users'),
        ('portal', 'Internal Users & Portal Users')
        ], compute='_compute_pad_availability', store=True, readonly=False,
        string='Availability of collaborative pads', required=True, default='internal')

    @api.depends('use_pads', 'privacy_visibility')
    def _compute_pad_availability(self):
        for project in self:
            if project.privacy_visibility != 'portal' or not project.use_pads:
                project.pad_availability = 'internal'

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project

```

## File: views\project_portal_assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="project_portal_assets_frontend" name="project portal assets" inherit_id="web.assets_frontend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" href="/pad_project/static/src/css/pad_project.css"/>
        </xpath>
    </template>
</odoo>

```

## File: views\project_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="portal_my_task" inherit_id="project.portal_my_task" priority="40">

        <!-- add a button to the right of the 'description' title according to the mode (edit/read) -->
        <xpath expr="//div[@t-if='task.description']//div[hasclass('d-flex')]" position="inside">
            <t t-if="task._use_portal_pad()">
                <t t-if="request.params.get('edit')">
                    <a role="button" class="btn btn-primary btn-sm ml-auto" t-attf-href="/my/task/#{task.id}">Save</a>
                </t>
                <t t-else="">
                    <a role="button" class="btn btn-primary btn-sm ml-auto" t-attf-href="/my/task/#{task.id}?edit=1">Edit</a>
                </t>
            </t>
        </xpath>

        <!-- show the description (read mode) or a pad (edit mode) -->
        <xpath expr="//div[@t-field='task.description']" position="replace">
            <t t-if="task._use_portal_pad()">
                <t t-if="request.params.get('edit')">
                    <div class="o_pad_project_container">
                        <iframe width="100%" height="100%" frameborder="0" t-att-src="task.description_pad + '?showChat=false&amp;userName=' + request.env.user.name"/>
                    </div>
                </t>
                <t t-else="">
                    <div class="py-1 px-2 bg-100 small" t-raw="task._get_pad_content()"/>
                </t>
            </t>
            <t t-else="">
                <t>$0</t>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_task_form2_inherit_pad_project" model="ir.ui.view">
        <field name="name">project.task.form.inherit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.view_task_form2"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='description']" position="attributes">
                <attribute name="attrs">{'invisible': [('use_pad', '=', True)], 'readonly': [('use_pad', '=', True)]}</attribute>
            </xpath>
            <field name="description" position="after">
                <field name="use_pad" invisible="1"/>
                <field name="description_pad" widget="pad" attrs="{'invisible': [('use_pad', '=', False)], 'readonly': [('use_pad', '=', False)]}"/>
            </field>
        </field>
    </record>

    <record id="project_project_view_form" model="ir.ui.view">
        <field name="name">project.project.view.form</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.edit_project"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='description']" position="attributes">
                <attribute name="attrs">{'invisible': [('use_pads', '=', True)], 'readonly': [('use_pads', '=', True)]}</attribute>
            </xpath>
            <field name="description" position="after">
                <field name="use_pads" invisible="1"/>
                <field name="description_pad" widget="pad" nolabel="1"
                       attrs="{'invisible': [('use_pads', '=', False)], 'readonly': [('use_pads', '=', False)]}"/>
            </field>
            <xpath expr="//div[@id='rating_settings']" position="after">
                <div class="col-lg-6 o_setting_box" id="pad_settings">
                    <div class="o_setting_left_pane">
                        <field name="use_pads"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="use_pads" string="Collaborative Pads"/>
                        <div class="text-muted">
                            Edit tasks' description collaboratively in real time.<br/>
                            See each author's text in a distinct color.
                        </div>
                        <div class="content-group" attrs="{'invisible': ['|', ('use_pads', '=', False), ('privacy_visibility', '!=', 'portal')]}">
                            <div class="mt16 row">
                                <label for="pad_availability" string="Available to:" class="col-3 col-lg-3 o_light_label"/>
                                <field name="pad_availability" widget="radio"/>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pad.project</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="project.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div name="pad_project_right_pane" position="inside">
                <div class="content-group" attrs="{'invisible': [('module_pad', '=', False)]}">
                    <div class="row mt16">
                        <label for="pad_server" class="col-lg-3 o_light_label"/>
                        <field name="pad_server" attrs="{'required': [('module_pad', '!=', False)]}"/>
                    </div>
                    <div class="row">
                        <label for="pad_key" class="col-lg-3 o_light_label"/>
                        <field name="pad_key" attrs="{'required': [('module_pad', '!=', False)]}"/>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>
```

