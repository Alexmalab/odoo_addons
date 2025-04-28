# Odoo Module: account_sale_timesheet

Category: Account/sale/timesheet

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
    'name': 'Account Sale Timesheet',
    'version': '1.0',
    'category': 'Account/sale/timesheet',
    'summary': 'Account sale timesheet',
    'description': 'Bridge created to add the number of invoices linked to an AA to a project form',
    'depends': ['account', 'sale_timesheet'],
    'data': [
        'views/project_project_views.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _lt

class Project(models.Model):
    _inherit = 'project.project'

    invoice_count = fields.Integer(related='analytic_account_id.invoice_count', groups='account.group_account_readonly')

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if self.user_has_groups('account.group_account_readonly'):
            buttons.append({
                'icon': 'pencil-square-o',
                'text': _lt('Invoices'),
                'number': self.invoice_count,
                'action_type': 'object',
                'action': 'action_open_project_invoices',
                'show': bool(self.analytic_account_id) and self.invoice_count > 0,
                'sequence': 11,
            })
        return buttons

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project

```

## File: views\project_project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_project_view_form" model="ir.ui.view">
        <field name="name">project.project.form.inherit</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.edit_project"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(project.action_project_task_burndown_chart_report)d']" position="after">
                <button name="action_open_project_invoices" type="object" class="oe_stat_button" icon="fa-pencil-square-o"
                        attrs="{'invisible': ['|', ('invoice_count', '=', 0), ('analytic_account_id', '=', False)]}"
                        groups="account.group_account_readonly">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="invoice_count" nolabel="1"/>
                        </span>
                        <span class="o_stat_text">
                            Invoices
                        </span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

</odoo>

```

