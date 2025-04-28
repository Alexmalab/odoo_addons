# Odoo Module: contacts

Category: Sales/CRM

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
    'name': 'Contacts',
    'category': 'Sales/CRM',
    'sequence': 150,
    'summary': 'Centralize your address book',
    'description': """
This module gives you a quick view of your contacts directory, accessible from your home page.
You can track your vendors, customers and other contacts.
""",
    'depends': ['base', 'mail'],
    'data': [
        'views/contact_views.xml',
    ],
    'demo': [
        'data/mail_demo.xml',
    ],
    'application': True,
    'license': 'LGPL-3',
    'assets': {
        'web.assets_tests': [
            'contacts/static/tests/tours/**/*',
        ],
    }
}

```

## File: data\mail_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <!-- Message with scheduled notification -->
    <record id="message_demo_partner_2_0" model="mail.message">
        <field name="author_id" ref="base.partner_demo"/>
        <field name="body" type="html"><p>Hello! Could you send us your address?</p></field>
        <field name="date" eval="(DateTime.today() - timedelta(days=1)).strftime('%Y-%m-%d %H:%M:00')"/>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_2"/>
        <field name="message_type">email</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
    </record>
    <record id="message_demo_partner_1_5_notif_0" model="mail.message.schedule">
        <field name="mail_message_id" ref="message_demo_partner_2_0"/>
        <field name="scheduled_datetime" eval="(DateTime.today() + timedelta(days=2)).strftime('%Y-%m-%d %H:%M:00')"/>
    </record>

</data></odoo>

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, modules


class Users(models.Model):
    _name = 'res.users'
    _inherit = ['res.users']

    @api.model
    def systray_get_activities(self):
        """ Update the systray icon of res.partner activities to use the
        contact application one instead of base icon. """
        activities = super(Users, self).systray_get_activities()
        for activity in activities:
            if activity['model'] != 'res.partner':
                continue
            activity['icon'] = modules.module.get_module_icon('contacts')
        return activities

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_users

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M39 46a8 8 0 0 0 8-8V8H13a4 4 0 0 0-4 4v34h30Z" fill="#FC868B"/><path fill-rule="evenodd" clip-rule="evenodd" d="M39 42a4 4 0 0 0 4-4V4H13a8 8 0 0 0-8 8v30h34Z" fill="#1AD3BB"/><path fill-rule="evenodd" clip-rule="evenodd" d="M43 38a4 4 0 0 1-4 4H9V12a4 4 0 0 1 4-4h30v30Z" fill="#1A6F66"/><path d="M25 28a9 9 0 0 0-9 9h10a9 9 0 0 0 9-9H25Z" fill="#fff"/><circle cx="25" cy="19" r="6" fill="#fff"/></svg>

```

## File: views\contact_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_contacts" model="ir.actions.act_window">
        <field name="name">Contacts</field>
        <field name="res_model">res.partner</field>
        <field name="view_mode">kanban,tree,form,activity</field>
        <field name="search_view_id" ref="base.view_res_partner_filter"/>
        <field name="context">{'default_is_company': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a Contact in your address book
          </p><p>
            Odoo helps you track all activities related to your contacts.
          </p>
        </field>
    </record>
    <record id="action_contacts_view_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="0"/>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="base.res_partner_kanban_view"/>
        <field name="act_window_id" ref="action_contacts"/>
    </record>
    <record id="action_contacts_view_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="base.view_partner_tree"/>
        <field name="act_window_id" ref="action_contacts"/>
    </record>
    <record id="action_contacts_view_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="2"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="base.view_partner_form"/>
        <field name="act_window_id" ref="action_contacts"/>
    </record>


    <menuitem name="Contacts"
        id="menu_contacts"
        sequence="20"
        web_icon="contacts,static/description/icon.png"
        groups="base.group_user,base.group_partner_manager"/>

    <menuitem id="res_partner_menu_contacts"
        name="Contacts"
        action="action_contacts"
        parent="menu_contacts"
        sequence="2"/>

    <menuitem id="res_partner_menu_config"
        name="Configuration"
        parent="menu_contacts"
        groups="base.group_system"
        sequence="2"/>

    <menuitem id="menu_partner_category_form"
        action="base.action_partner_category_form"
        name="Contact Tags"
        sequence="1" parent="res_partner_menu_config"/>

    <menuitem id="menu_partner_title_contact"
        action="base.action_partner_title_contact"
        name="Contact Titles" parent="res_partner_menu_config"
        sequence="3"/>

    <menuitem id="res_partner_industry_menu" name="Industries"
        action="base.res_partner_industry_action" parent="res_partner_menu_config"
        sequence="4"/>

    <menuitem id="menu_localisation" name="Localization"
        parent="res_partner_menu_config" sequence="5"/>

    <menuitem id="menu_country_partner"
        action="base.action_country" parent="menu_localisation"
        sequence="1"/>

    <menuitem id="menu_country_group"
        action="base.action_country_group"
        name="Country Group" parent="menu_localisation"
        sequence="3"/>

    <menuitem id="menu_country_state_partner"
        action="base.action_country_state"
        parent="menu_localisation"
        sequence="2"/>

    <menuitem id="menu_config_bank_accounts"
        name="Bank Accounts"
        parent="res_partner_menu_config"
        sequence="6"/>

    <menuitem id="menu_action_res_bank_form"
        action="base.action_res_bank_form"
        parent="menu_config_bank_accounts"
        sequence="1"/>

    <menuitem id="menu_action_res_partner_bank_form"
        action="base.action_res_partner_bank_account_form"
        parent="menu_config_bank_accounts"
        sequence="2"/>
</odoo>

```

