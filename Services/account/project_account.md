# Odoo Module: project_account

Category: Services/account

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Project Accounting',
    'version': '1.0',
    'category': 'Services/account',
    'summary': 'Project accounting',
    'description': 'Bridge created to remove the profitability setting if the account module is installed',
    'depends': ['project', 'account'],
    'data': [
        'views/project_project_templates.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: views\project_project_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.project</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" eval="False" />
            <field name="arch" type="xml">
                <form/>
            </field>
        </record>
</odoo>

```

