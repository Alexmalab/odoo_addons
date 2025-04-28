# Odoo Module: test_base_automation

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
    'name': 'Test - Base Automation',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 9877,
    'summary': 'Base Automation Tests: Ensure Flow Robustness',
    'description': """This module contains tests related to base automation. Those are
present in a separate module as it contains models used only to perform
tests independently to functional aspects of other models.""",
    'depends': ['base_automation'],
    'data': [
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'application': False,
    'license': 'LGPL-3',
}

```

## File: models\test_base_automation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil import relativedelta
from odoo import fields, models, api


class LeadTest(models.Model):
    _name = "base.automation.lead.test"
    _description = "Automated Rule Test"

    name = fields.Char(string='Subject', required=True, index=True)
    user_id = fields.Many2one('res.users', string='Responsible')
    state = fields.Selection([('draft', 'New'), ('cancel', 'Cancelled'), ('open', 'In Progress'),
                              ('pending', 'Pending'), ('done', 'Closed')],
                             string="Status", readonly=True, default='draft')
    active = fields.Boolean(default=True)
    partner_id = fields.Many2one('res.partner', string='Partner')
    date_action_last = fields.Datetime(string='Last Action', readonly=True)
    employee = fields.Boolean(compute='_compute_employee_deadline', store=True)
    line_ids = fields.One2many('base.automation.line.test', 'lead_id')

    priority = fields.Boolean()
    deadline = fields.Boolean(compute='_compute_employee_deadline', store=True)
    is_assigned_to_admin = fields.Boolean(string='Assigned to admin user')

    @api.depends('partner_id.employee', 'priority')
    def _compute_employee_deadline(self):
        # this method computes two fields on purpose; don't split it
        for record in self:
            record.employee = record.partner_id.employee
            if not record.priority:
                record.deadline = False
            else:
                record.deadline = record.create_date + relativedelta.relativedelta(days=3)

    def write(self, vals):
        result = super().write(vals)
        # force recomputation of field 'deadline' via 'employee': the action
        # based on 'deadline' must be triggered
        self.mapped('employee')
        return result


class LineTest(models.Model):
    _name = "base.automation.line.test"
    _description = "Automated Rule Line Test"

    name = fields.Char()
    lead_id = fields.Many2one('base.automation.lead.test', ondelete='cascade')
    user_id = fields.Many2one('res.users')


class ModelWithAccess(models.Model):
    _name = "base.automation.link.test"
    _description = "Automated Rule Link Test"

    name = fields.Char()
    linked_id = fields.Many2one('base.automation.linked.test', ondelete='cascade')


class ModelWithoutAccess(models.Model):
    _name = "base.automation.linked.test"
    _description = "Automated Rule Linked Test"

    name = fields.Char()
    another_field = fields.Char()


class Project(models.Model):
    _name = _description = 'test_base_automation.project'

    name = fields.Char()
    task_ids = fields.One2many('test_base_automation.task', 'project_id')


class Task(models.Model):
    _name = _description = 'test_base_automation.task'

    name = fields.Char()
    parent_id = fields.Many2one('test_base_automation.task')
    project_id = fields.Many2one(
        'test_base_automation.project',
        compute='_compute_project_id', recursive=True, store=True, readonly=False,
    )

    @api.depends('parent_id.project_id')
    def _compute_project_id(self):
        for task in self:
            if not task.project_id:
                task.project_id = task.parent_id.project_id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import test_base_automation
```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_base_automation_lead_test,access_base_automation_lead_test,model_base_automation_lead_test,base.group_system,1,1,1,1
access_base_automation_line_test,access_base_automation_line_test,model_base_automation_line_test,base.group_system,1,1,1,1
access_base_automation_link_test,access_base_automation_link_test,model_base_automation_link_test,,1,1,1,1
access_base_automation_linked_test,access_base_automation_linked_test,model_base_automation_linked_test,,1,1,1,1
access_test_base_automation_project,access_test_base_automation_project,model_test_base_automation_project,,1,1,1,1
access_test_base_automation_task,access_test_base_automation_task,model_test_base_automation_task,,1,1,1,1

```

