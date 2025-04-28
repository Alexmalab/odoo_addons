# Odoo Module: pad_project

Category: Operations/Project

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
    'category': 'Operations/Project',
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
        'views/project_views.xml'
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
        project_id = vals.get('project_id', False) or self.default_get(['project_id'])['project_id']
        if not self.env['project.project'].browse(project_id).use_pads:
            self = self.with_context(pad_no_create=True)
        return super(ProjectTask, self).create(vals)


class ProjectProject(models.Model):
    _inherit = "project.project"

    use_pads = fields.Boolean("Use collaborative pads", help="Use collaborative pad for the tasks on this project.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project

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
            <field name="partner_id" position="after">
                <field name="use_pads"/>
            </field>
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
                <div class="content-group">
                    <field name='company_id' invisible="1"/>
                    <div class="row mt16">
                        <label for="pad_server" class="col-lg-3 o_light_label"/>
                        <field name="pad_server"/>
                    </div>
                    <div class="row">
                        <label for="pad_key" class="col-lg-3 o_light_label"/>
                        <field name="pad_key"/>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>
```

