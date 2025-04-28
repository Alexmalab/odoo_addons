# Odoo Module: hr_hourly_cost

Category: Services/Employee Hourly Cost

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
    'name': 'Employee Hourly Wage',
    'version': '1.0',
    'category': 'Services/Employee Hourly Cost',
    'summary': 'Employee Hourly Wage',
    'description': """
This module assigns an hourly wage to employees to be used by other modules.
============================================================================

    """,
    'depends': ['hr'],
    'data': [
        'views/hr_employee_views.xml',
    ],
    'demo': [
        'data/hr_hourly_cost_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\hr_hourly_cost_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>    
    <!-- Employee -->
    <record id="hr.employee_admin" model="hr.employee">
        <field name="hourly_cost">100</field>
    </record>

    <record id="hr.employee_vad" model="hr.employee">
        <field name="hourly_cost">35</field>
    </record>

    <record id="hr.employee_jth" model="hr.employee">
        <field name="hourly_cost">25</field>
    </record>

    <record id="hr.employee_niv" model="hr.employee">
        <field name="hourly_cost">45</field>
    </record>

    <record id="hr.employee_jod" model="hr.employee">
        <field name="hourly_cost">55</field>
    </record>

    <record id="hr.employee_jve" model="hr.employee">
        <field name="hourly_cost">15</field>
    </record>

    <record id="hr.employee_fme" model="hr.employee">
        <field name="hourly_cost">45</field>
    </record>

    <record id="hr.employee_chs" model="hr.employee">
        <field name="hourly_cost">20</field>
    </record>

    <record id="hr.employee_ngh" model="hr.employee">
        <field name="hourly_cost">40</field>
    </record>

    <record id="hr.employee_jgo" model="hr.employee">
        <field name="hourly_cost">45</field>
    </record>

    <record id="hr.employee_lur" model="hr.employee">
        <field name="hourly_cost">35</field>
    </record>

    <record id="hr.employee_jep" model="hr.employee">
        <field name="hourly_cost">25</field>
    </record>

    <record id="hr.employee_jog" model="hr.employee">
        <field name="hourly_cost">40</field>
    </record>

    <record id="hr.employee_fpi" model="hr.employee">
        <field name="hourly_cost">50</field>
    </record>

    <record id="hr.employee_mit" model="hr.employee">
        <field name="hourly_cost">15</field>
    </record>

    <record id="hr.employee_hne" model="hr.employee">
        <field name="hourly_cost">10</field>
    </record>

    <record id="hr.employee_qdp" model="hr.employee">
        <field name="hourly_cost">75</field>
    </record>

    <record id="hr.employee_stw" model="hr.employee">
        <field name="hourly_cost">65</field>
    </record>

    <record id="hr.employee_al" model="hr.employee">
        <field name="hourly_cost">50</field>
    </record>

    <record id="hr.employee_han" model="hr.employee">
        <field name="hourly_cost">35</field>
    </record>
</odoo>

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    hourly_cost = fields.Monetary('Hourly Cost', currency_field='currency_id',
        groups="hr.group_hr_user", default=0.0)

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
    <record id="view_employee_form" model="ir.ui.view">
        <field name="name">view.employee.form.inherit.hr.employee.hourly.wage</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="priority" eval="40"/>
        <field name="arch" type="xml">
            <group name="application_group" position="attributes">
                <attribute name="invisible">0</attribute>
            </group>
            <group name="application_group" position="inside">
                <label for="hourly_cost"/>
                <div name="hourly_cost">
                    <field name="hourly_cost" class="oe_inline"/>
                    <field name="currency_id" invisible="1"/>
                </div>
            </group>
        </field>
    </record>
</odoo>

```

