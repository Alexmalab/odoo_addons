# Odoo Module: website_sms

Category: Website/Website

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
    'name': 'Send SMS to Visitor',
    'category': 'Website/Website',
    'sequence': 54,
    'summary': 'Allows to send sms to website visitor',
    'version': '1.0',
    'description': """Allows to send sms to website visitor if the visitor is linked to a partner.""",
    'depends': ['website', 'sms'],
    'data': [
        'views/website_visitor_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.exceptions import UserError


class WebsiteVisitor(models.Model):
    _inherit = 'website.visitor'

    def _prepare_visitor_send_sms_values(self):
        if self.partner_id.mobile:
            return {
                'res_model': 'res.partner',
                'res_id': self.partner_id.id,
                'partner_ids': [self.partner_id.id],
                'number_field_name': 'mobile',
            }
        return {}

    def action_send_sms(self):
        self.ensure_one()
        visitor_sms_values = self._prepare_visitor_send_sms_values()
        if not visitor_sms_values:
            raise UserError(_("There is no mobile phone number linked to this visitor."))

        context = dict(self.env.context)
        context.update({
            'default_res_model': visitor_sms_values['res_model'],
            'default_res_id': visitor_sms_values['res_id'],
            'default_composition_mode': 'comment',
            'default_number_field_name': visitor_sms_values['number_field_name'],
        })

        return {
            "type": "ir.actions.act_window",
            "res_model": "sms.composer",
            "view_mode": 'form',
            "context": context,
            "name": "Send SMS Text Message",
            "target": "new",
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website_visitor

```

## File: views\website_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
     <record id="website_visitor_view_form" model="ir.ui.view">
        <field name="name">website.visitor.view.form.inherit.website.mass.mailing.sms</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="action_send_sms" type="object" class="btn btn-primary"
                        attrs="{'invisible': [('mobile', '=', False)]}" string="Send SMS"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_kanban" model="ir.ui.view">
        <field name="name">website.visitor.view.kanban.inherit.website.sms</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_kanban"/>
        <field name="arch" type="xml">
            <field name="page_ids" position="after">
                <field name="mobile" widget="phone"/>
            </field>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions')]" position="inside">
                <button name="action_send_sms" type="object" class="btn btn-secondary"
                        attrs="{'invisible': [('mobile', '=', False)]}">SMS
                </button>
            </xpath>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions_ungrouped')]" position="inside">
                <button name="action_send_sms" type="object" class="btn btn-secondary border"
                        attrs="{'invisible': [('mobile', '=', False)]}">SMS
                </button>
            </xpath>
        </field>
    </record>

     <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.tree.inherit.website.sms</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_send_mail']" position="after">
                <field name="mobile" invisible="1"/>
                <button name="action_send_sms" type="object" icon="fa-mobile"
                        attrs="{'invisible': [('mobile', '=', False)]}" string="Send SMS"/>
            </xpath>
        </field>
    </record>
</data></odoo>

```

