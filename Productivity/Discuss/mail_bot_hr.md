# Odoo Module: mail_bot_hr

Category: Productivity/Discuss

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "OdooBot - HR",
    'summary': """Bridge module between hr and mailbot.""",
    'description': """This module adds the OdooBot state and notifications in the user form modified by hr.""",
    'website': "https://www.odoo.com/app/discuss",
    'category': 'Productivity/Discuss',
    'version': '1.0',
    'depends': ['mail_bot', 'hr'],
    'installable': True,
    'auto_install': True,
    'data': [
        'views/res_users_views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <record id="res_users_view_form_simple_modif" model="ir.ui.view">
            <field name="name">res.users.preferences.form.simplified.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="hr.res_users_view_form_simple_modif" />
            <field name="priority">15</field>
            <field name="arch" type="xml">
                <widget name="notification_alert" position="replace" />
            </field>
        </record>

        <record id="res_users_view_form_preferences" model="ir.ui.view">
            <field name="name">res.users.preferences.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id"
                   ref="mail_bot.res_users_view_form_preferences" />
            <field name="arch" type="xml">
                <field name="odoobot_state" position="before">
                    <field name="can_edit" invisible="1"/>
                </field>
                <xpath expr="//field[@name='odoobot_state']" position="attributes">
                    <attribute name="readonly">not can_edit</attribute>
                </xpath>
            </field>
        </record>

        <record id="res_users_view_form_profile" model="ir.ui.view">
            <field name="name">res.users.profile.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id"
                   ref="hr.res_users_view_form_profile" />
            <field name="arch" type="xml">
                <sheet position="before">
                    <widget name="notification_alert" />
                </sheet>
            </field>
        </record>
    </data>
</odoo>

```

