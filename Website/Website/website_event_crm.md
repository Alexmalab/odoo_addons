# Odoo Module: website_event_crm

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Website Events CRM',
    'version': '1.0',
    'category': 'Website/Website',
    'website': 'https://www.odoo.com/page/events',
    'description': "Allow per-order lead creation mode",
    'depends': ['event_crm', 'website_event'],
    'data': [
        'views/event_lead_rule_views.xml',
    ],
    'demo': [
        'data/event_crm_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\event_crm_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <!-- Event CRM Rule 1 -->
        <record id="event_lead_rule_1" model="event.lead.rule">
            <field name="name">Rule per order</field>
            <field name="lead_creation_basis">order</field>
            <field name="event_id" ref="event.event_0"/>
            <field name="lead_user_id" ref="base.user_demo"/>
            <field name="lead_tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor3')]),(6, 0, [ref('sales_team.categ_oppor6')])]"/>
        </record>
    </data>
</odoo>

```

## File: views\event_lead_rule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_lead_rule_view_tree" model="ir.ui.view">
        <field name="name">event.lead.rule.view.tree.inherit.website.event.crm</field>
        <field name="model">event.lead.rule</field>
        <field name="inherit_id" ref="event_crm.event_lead_rule_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lead_creation_basis']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>
    <record id="event_lead_rule_view_form" model="ir.ui.view">
        <field name="name">event.lead.rule.view.form.inherit.website.event.crm</field>
        <field name="model">event.lead.rule</field>
        <field name="inherit_id" ref="event_crm.event_lead_rule_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='lead_creation_basis']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

