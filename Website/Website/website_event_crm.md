# Odoo Module: website_event_crm

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Website Events CRM',
    'version': '1.0',
    'category': 'Website/Website',
    'website': 'https://www.odoo.com/app/events',
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

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from markupsafe import Markup


class EventRegistration(models.Model):
    _inherit = 'event.registration'

    def _get_lead_description_registration(self, line_suffix=''):
        """Add the questions and answers linked to the registrations into the description of the lead."""
        reg_description = super(EventRegistration, self)._get_lead_description_registration(line_suffix=line_suffix)
        if not self.registration_answer_ids:
            return reg_description

        answer_descriptions = []
        for answer in self.registration_answer_ids:
            answer_value = answer.value_answer_id.name if answer.question_type == "simple_choice" else answer.value_text_box
            answer_value = Markup("<br/>").join(["    %s" % line for line in answer_value.split('\n')])
            answer_descriptions.append(Markup("  - %s<br/>%s") % (answer.question_id.title, answer_value))
        return Markup("%s%s<br/>%s") % (reg_description, _("Questions"), Markup('<br/>').join(answer_descriptions))

    def _get_lead_description_fields(self):
        res = super(EventRegistration, self)._get_lead_description_fields()
        res.append('registration_answer_ids')
        return res

    def _get_lead_values(self, rule):
        """Update lead values from Lead Generation rules to include the visitor and their language"""
        lead_values = super()._get_lead_values(rule)
        if self.visitor_id:
            lead_values['visitor_ids'] = self.visitor_id
        if self.visitor_id.lang_id:
            lead_values['lang_id'] = self.visitor_id.lang_id[0].id
        return lead_values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_registration

```

## File: views\event_lead_rule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_lead_rule_view_tree" model="ir.ui.view">
        <field name="name">event.lead.rule.view.list.inherit.website.event.crm</field>
        <field name="model">event.lead.rule</field>
        <field name="inherit_id" ref="event_crm.event_lead_rule_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lead_creation_basis']" position="attributes">
                <attribute name="column_invisible">False</attribute>
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

