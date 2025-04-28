# Odoo Module: crm_sms

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
    'name': 'SMS in CRM',
    'version': '1.0',
    'category': 'Sales/CRM',
    'summary': 'Add SMS capabilities to CRM',
    'description': "",
    'depends': ['crm', 'sms'],
    'data': [
        'views/crm_lead_views.xml',
        'security/ir.model.access.csv',
        'security/sms_security.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class CrmLead(models.Model):
    _inherit = 'crm.lead'

    def _sms_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        # TDE FIXME: to be cleaned in 14.4+ as it conflicts with _phone_get_number_fields
        return ['mobile', 'phone']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_template_sale_manager,access.sms.template.sale.manager,sms.model_sms_template,sales_team.group_sale_manager,1,1,1,1

```

## File: security\sms_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="ir_rule_sms_template_sale_manager" model="ir.rule">
        <field name="name">SMS Template: sale manager CUD on opportunity / partner templates</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
        <field name="domain_force">[('model_id.model', 'in', ('crm.lead', 'res.partner'))]</field>
        <field name="perm_read" eval="False"/>
    </record>
</odoo>

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Add action entry in the Action Menu for Leads -->
    <record id="crm_lead_act_window_sms_composer_single" model="ir.actions.act_window">
        <field name="name">Send SMS Text Message</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_composition_mode': 'mass',
            'default_mass_keep_log': True,
            'default_res_ids': active_ids
        }</field>
        <field name="binding_model_id" ref="model_crm_lead"/>
        <field name="binding_view_types">list</field>
    </record>
    <record id="crm_lead_act_window_sms_composer_multi" model="ir.actions.act_window">
        <field name="name">Send SMS Text Message</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_composition_mode': 'comment',
            'default_res_id': active_id,
        }</field>
        <field name="binding_model_id" ref="model_crm_lead"/>
        <field name="binding_view_types">form</field>
    </record>

    <record id="crm_lead_view_list_activities" model="ir.ui.view">
        <field name="name">crm.lead.view.list.activities.inherit.sms</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_list_activities"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="%(crm_sms.crm_lead_act_window_sms_composer_single)d" type="action" string="SMS" />
            </xpath>
            <xpath expr="//button[@name='%(crm.crm_lead_act_window_compose)d']" position="after">
                <button name="%(crm_sms.crm_lead_act_window_sms_composer_multi)d" type="action" string="SMS" icon="fa-comments" />
            </xpath>
        </field>
    </record>

</odoo>

```

