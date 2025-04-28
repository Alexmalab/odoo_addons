# Odoo Module: hr_holidays_attendance

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "HR Attendance Holidays",
    'summary': """""",
    'category': 'Human Resources',
    'description': """
Hides the attendance presence button when an employee is on leave.
    """,
    'version': '1.0',
    'depends': ['hr_attendance', 'hr_holidays'],
    'auto_install': True,
    'data': [
        'views/hr_employee_views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="hr_user_view_form" model="ir.ui.view">
        <field name="name">hr.user.preferences.view.form.attendance.inherit</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr_attendance.hr_user_view_form"/>
        <field name="arch" type="xml">
            <!-- Hide Attendance button -->
            <xpath expr="//button[@id='hr_attendance_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '&amp;',
                        ('hr_presence_state', '=', 'present'),
                        ('attendance_state', '=', 'checked_out'),
                        ('is_absent', '=', True),
                        ('id', '=', False),
                    ]}
                </attribute>
            </xpath>
            <!-- Merge invisible attr of both module -->
            <xpath expr="//button[@id='hr_presence_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '|',
                        ('is_absent', '=', True),
                        ('hr_presence_state', '=', 'absent'),
                        ('attendance_state', '=', 'checked_in'),
                        ('id', '=', False),
                    ]}
                </attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_view_form" model="ir.ui.view">
        <field name="name">hr.employee.holidays.attendance.inherit</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr_attendance.view_employee_form_inherit_hr_attendance"/>
        <field name="arch" type="xml">
            <!-- Hide Attendance button -->
            <xpath expr="//button[@id='hr_attendance_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '&amp;',
                        ('hr_presence_state', '=', 'present'),
                        ('attendance_state', '=', 'checked_out'),
                        ('is_absent', '=', True),
                        ('id', '=', False),
                    ]}
                </attribute>
            </xpath>
            <!-- Merge invisible attr of both module -->
            <xpath expr="//button[@id='hr_presence_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '|',
                        ('is_absent', '=', True),
                        ('hr_presence_state', '=', 'absent'),
                        ('attendance_state', '=', 'checked_in'),
                        ('id', '=', False),
                    ]}
                </attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

