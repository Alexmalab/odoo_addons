# Odoo Module: project_sms

Category: Hidden

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
    'name': "Project - SMS",
    'summary': 'Send text messages when project/task stage move',
    'description': "Send text messages when project/task stage move",
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['project', 'sms'],
    'data': [
        'views/project_stage_views.xml',
        'views/project_task_type_views.xml',
        'views/project_views.xml',
        'security/ir.model.access.csv',
        'security/project_sms_security.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProjectProject(models.Model):
    _inherit = "project.project"

    def _send_sms(self):
        for project in self:
            if project.partner_id and project.stage_id and project.stage_id.sms_template_id:
                project._message_sms_with_template(
                    template=project.stage_id.sms_template_id,
                    partner_ids=project.partner_id.ids,
                )

    @api.model_create_multi
    def create(self, vals_list):
        projects = super().create(vals_list)
        projects._send_sms()
        return projects

    def write(self, vals):
        res = super().write(vals)
        if 'stage_id' in vals:
            self._send_sms()
        return res

```

## File: models\project_stage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProjectProjectStage(models.Model):
    _inherit = 'project.project.stage'

    sms_template_id = fields.Many2one('sms.template', string="SMS Template",
        domain=[('model', '=', 'project.project')],
        help="If set, an SMS Text Message will be automatically sent to the customer when the project reaches this stage.")

```

## File: models\project_task.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProjectTask(models.Model):
    _inherit = "project.task"

    def _send_sms(self):
        for task in self:
            if task.partner_id and task.stage_id and task.stage_id.sms_template_id:
                task._message_sms_with_template(
                    template=task.stage_id.sms_template_id,
                    partner_ids=task.partner_id.ids,
                )

    @api.model_create_multi
    def create(self, vals_list):
        tasks = super().create(vals_list)
        tasks._send_sms()
        return tasks

    def write(self, vals):
        res = super().write(vals)

        if 'stage_id' in vals:
            # sudo as sms template model is protected
            self.sudo()._send_sms()
        return res

```

## File: models\project_task_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProjectTaskType(models.Model):
    _inherit = "project.task.type"

    sms_template_id = fields.Many2one('sms.template', string="SMS Template",
        domain=[('model', '=', 'project.task')],
        help="If set, an SMS Text Message will be automatically sent to the customer when the task reaches this stage.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_task_type
from . import project_task
from . import project_stage
from . import project

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_template_project_manager,access.sms.template.project.manager,sms.model_sms_template,project.group_project_manager,1,1,1,1

```

## File: security\project_sms_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="ir_rule_sms_template_project_manager" model="ir.rule">
        <field name="name">SMS Template: project manager CUD on project/task</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('project.group_project_manager'))]"/>
        <field name="domain_force">[('model_id.model', 'in', ('project.task.type', 'project.project.stage'))]</field>
        <field name="perm_read" eval="False"/>
    </record>
</odoo>

```

## File: views\project_stage_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_project_stage_view_tree_inherit_project_sms" model="ir.ui.view">
        <field name="name">project.project.stage.view.tree.inherit.project.sms</field>
        <field name="model">project.project.stage</field>
        <field name="inherit_id" ref="project.project_project_stage_view_tree"/>
        <field name="arch" type="xml">
            <field name="mail_template_id" position="after">
                <field name="sms_template_id" optional="hide" context="{'default_model': 'project.project'}"/>
            </field>
        </field>
    </record>

    <record id="project_project_stage_view_form_inherit_project_sms" model="ir.ui.view">
        <field name="name">project.project.stage.view.form.inherit.project.sms</field>
        <field name="model">project.project.stage</field>
        <field name="inherit_id" ref="project.project_project_stage_view_form"/>
        <field name="arch" type="xml">
            <field name="mail_template_id" position="after">
                <field name="sms_template_id" context="{'default_model': 'project.project'}" options="{'no_quick_create': True}"/>
            </field>
        </field>
    </record>

    <record id="project_project_stage_view_search_inherit_project_sms" model="ir.ui.view">
        <field name="name">project.project.stage.view.search.inherit.project.sms</field>
        <field name="model">project.project.stage</field>
        <field name="inherit_id" ref="project.project_project_stage_view_search"/>
        <field name="arch" type="xml">
            <field name="mail_template_id" position="after">
                <field name="sms_template_id" domain="[('model', '=', 'project.project')]"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\project_task_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="task_type_edit_view_form_inherit_project_sms" model="ir.ui.view">
        <field name="name">project.task.type.view.form.inherit.project.sms</field>
        <field name="model">project.task.type</field>
        <field name="inherit_id" ref="project.task_type_edit"/>
        <field name="arch" type="xml">
            <field name="mail_template_id" position="before">
                <field name="sms_template_id" context="{'default_model': 'project.task'}" options="{'no_quick_create': True}"/>
            </field>
        </field>
    </record>

    <record id="task_type_edit_view_tree_inherit_project_sms" model="ir.ui.view">
        <field name="name">project.task.type.view.tree.inherit.project.sms</field>
        <field name="model">project.task.type</field>
        <field name="inherit_id" ref="project.task_type_tree_inherited"/>
        <field name="arch" type="xml">
            <field name="mail_template_id" position="after">
                <field name="sms_template_id" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="task_type_search_view_search_inherit_project_sms" model="ir.ui.view">
        <field name="name">project.task.type.view.search.inherit.project.sms</field>
        <field name="model">project.task.type</field>
        <field name="inherit_id" ref="project.task_type_search"/>
        <field name="arch" type="xml">
            <field name="rating_template_id" position="after">
                <field name="sms_template_id" domain="[('model', '=', 'project.task')]"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_task_act_window_sms_composer" model="ir.actions.act_window">
        <field name="name">Send SMS Text Message</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_composition_mode': 'mass',
            'default_mass_keep_log': True,
            'default_res_ids': active_ids,
        }</field>
        <field name="binding_model_id" ref="model_project_task"/>
        <field name="binding_view_types">list,form</field>
    </record>

    <record id="project_project_act_window_sms_composer" model="ir.actions.act_window">
        <field name="name">Send SMS Text Message</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_composition_mode': 'mass',
            'default_mass_keep_log': True,
            'default_res_ids': active_ids,
        }</field>
        <field name="binding_model_id" ref="model_project_project"/>
        <field name="binding_view_types">list</field>
    </record>
</odoo>

```

