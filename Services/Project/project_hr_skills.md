# Odoo Module: project_hr_skills

Category: Services/Project

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Project - Skills',
    'summary': 'Project skills',
    'description': """
        Search project tasks according to the assignees' skills
    """,
    'category': 'Services/Project',
    'version': '1.0',
    'depends': ['project', 'hr_skills'],
    'auto_install': True,
    'data': [
        'views/project_task_views.xml',
    ],
    'license': 'OEEL-1',
}

```

## File: models\project_task.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ProjectTask(models.Model):
    _inherit = "project.task"

    user_skill_ids = fields.One2many('hr.employee.skill', related='user_ids.employee_skill_ids')

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_task

```

## File: report\report_project_task_user.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ReportProjectTaskUser(models.Model):
    _inherit = 'report.project.task.user'

    user_skill_ids = fields.One2many('hr.employee.skill', related='user_ids.employee_skill_ids', string='Skills')

```

## File: report\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import report_project_task_user

```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_task_search_form_project_fsm_base_inherit" model="ir.ui.view">
        <field name="name">search.view.inherit.project.hr.skills</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.view_task_search_form_project_fsm_base"/>
        <field name="arch" type="xml">
            <field name='partner_id' position="after">
                <field name="user_skill_ids" string="Skills" filter_domain="['|', ('user_ids', '=', False), ('user_skill_ids', 'ilike', self)]"/>
            </field>
        </field>
    </record>
</odoo>

```

