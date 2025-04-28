# Odoo Module: l10n_mx_hr

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Employees - Mexico',
    'version': '1.0',
    'category': 'Human Resources/Employees',
    'icon': '/l10n_mx/static/description/icon.png',
    'sequence': 120,
    'summary': 'Adds specific fields to Employees for Mexican companies.',
    'depends': ['hr'],
    'data': [
        'views/hr_employee_views.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Employee(models.Model):
    _inherit = 'hr.employee'

    l10n_mx_nss = fields.Char('NSS', groups="hr.group_hr_user", tracking=True)
    l10n_mx_curp = fields.Char('CURP', groups="hr.group_hr_user", tracking=True)
    l10n_mx_rfc = fields.Char('RFC', groups="hr.group_hr_user", tracking=True)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_form" model="ir.ui.view">
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <field name="identification_id" position="after">
                <field name="l10n_mx_nss" attrs="{'invisible': [('company_country_code', '!=', 'MX')]}"/>
            </field>
            <field name="passport_id" position="after">
                <field name="l10n_mx_rfc" attrs="{'invisible': [('company_country_code', '!=', 'MX')]}"/>
                <field name="l10n_mx_curp" attrs="{'invisible': [('company_country_code', '!=', 'MX')]}"/>
            </field>
        </field>
    </record>
</odoo>

```

