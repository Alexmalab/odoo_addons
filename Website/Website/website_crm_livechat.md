# Odoo Module: website_crm_livechat

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
    'name': 'Lead Livechat Sessions',
    'category': 'Website/Website',
    'summary': 'View livechat sessions for leads',
    'version': '1.0',
    'description': """ Adds a stat button on lead form view to access their livechat sessions.""",
    'depends': ['website_crm', 'website_livechat'],
    'data': [
        'views/website_crm_lead_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Lead(models.Model):
    _inherit = 'crm.lead'

    visitor_sessions_count = fields.Integer('# Sessions', compute="_compute_visitor_sessions_count", groups="im_livechat.im_livechat_group_user")

    @api.depends('visitor_ids.mail_channel_ids')
    def _compute_visitor_sessions_count(self):
        for lead in self:
            lead.visitor_sessions_count = len(lead.visitor_ids.mail_channel_ids)

    def action_redirect_to_livechat_sessions(self):
        visitors = self.visitor_ids
        action = self.env["ir.actions.actions"]._for_xml_id("website_livechat.website_visitor_livechat_session_action")
        action['domain'] = [('livechat_visitor_id', 'in', visitors.ids), ('has_message', '=', True)]
        return action

```

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailChannel(models.Model):
    _inherit = 'mail.channel'

    def _convert_visitor_to_lead(self, partner, key):
        """ When website is installed, we can link the created lead from /lead command
         to the current website_visitor. We do not use the lead name as it does not correspond
         to the lead contact name."""
        lead = super(MailChannel, self)._convert_visitor_to_lead(partner, key)
        visitor_sudo = self.livechat_visitor_id.sudo()
        if visitor_sudo:
            visitor_sudo.write({'lead_ids': [(4, lead.id)]})
            lead.country_id = lead.country_id or visitor_sudo.country_id
        return lead

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import mail_channel

```

## File: views\website_crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="crm_lead_view_form" model="ir.ui.view">
        <field name="name">crm.lead.view.form.inherit.website.crm.livechat</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="website_crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_redirect_to_page_views']" position="after">
                <button name="action_redirect_to_livechat_sessions" type="object" class="oe_stat_button" icon="fa-comment"
                        attrs="{'invisible': [('visitor_sessions_count', '=', 0)]}" groups="im_livechat.im_livechat_group_user">
                    <field name="visitor_sessions_count" widget="statinfo" string="Sessions"/>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>

```

