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
    'countries': ['mx'],
    'version': '1.0',
    'category': 'Human Resources/Employees',
    'sequence': 120,
    'summary': 'Adds specific fields to Employees for Mexican companies.',
    'depends': ['hr'],
    'data': [
        'views/hr_employee_views.xml',
    ],
    'demo': [
        'data/l10n_mx_hr_demo.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\l10n_mx_hr_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="base.demo_company_mx" model="res.company" forcecreate="1">
        <field name="name">ESCUELA KEMPER URGATE</field>
        <field name="currency_id" ref="base.MXN"/>
        <field name="country_id" ref="base.mx"/>
    </record>

    <record id="base.user_admin" model="res.users">
        <field name="company_ids" eval="[(4, ref('base.demo_company_mx'))]"/>
    </record>

    <record id="base.hr_employee_cesar" model="hr.employee" forcecreate="1">
        <field name="name">Cesar Osbaldo Cruz Solorzano</field>
        <field name="l10n_mx_rfc">CUSC850516316</field>
        <field name="l10n_mx_curp">CUSC850516XNERLS05</field>
        <field name="company_id" ref="base.demo_company_mx" />
    </record>

    <record id="base.hr_employee_cecilia" model="hr.employee" forcecreate="1">
        <field name="name">Cecilia Miranda Sanchez</field>
        <field name="l10n_mx_rfc">MISC491214B86</field>
        <field name="l10n_mx_curp">MISC491214MDFRNC06</field>
        <field name="company_id" ref="base.demo_company_mx" />
    </record>

    <record id="base.hr_employee_xochilt" model="hr.employee" forcecreate="1">
        <field name="name">Xochilt Casas Chavez</field>
        <field name="l10n_mx_rfc">CACX7605011P8</field>
        <field name="l10n_mx_curp">CACX760431HNESHC03</field>
        <field name="company_id" ref="base.demo_company_mx" />
    </record>

    <record id="base.hr_employee_karla" model="hr.employee" forcecreate="1">
        <field name="name">Karla Fuente Nolasco</field>
        <field name="l10n_mx_rfc">FUNK671228PH6</field>
        <field name="l10n_mx_curp">FUNK671228MYNNLR02</field>
        <field name="company_id" ref="base.demo_company_mx" />
    </record>
</odoo>

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Employee(models.Model):
    _inherit = 'hr.employee'

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
            <field name="passport_id" position="after">
                <field name="l10n_mx_rfc" invisible="company_country_code != 'MX'"/>
                <field name="l10n_mx_curp" invisible="company_country_code != 'MX'"/>
            </field>
        </field>
    </record>
</odoo>

```

