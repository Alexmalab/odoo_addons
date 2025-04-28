# Odoo Module: mass_mailing_crm

Category: Hidden

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
    'name': 'Mass mailing on lead / opportunities',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Add lead / opportunities UTM info on mass mailing',
    'description': """UTM and mass mailing on lead / opportunities""",
    'depends': ['crm', 'mass_mailing'],
    'data': [
        'views/mailing_mailing_views.xml',
    ],
    'demo': [
        'data/mass_mailing_demo.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\mass_mailing_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
    <record id="mass_mail_lead_0" model="mailing.mailing">
        <field name="name">Lead Recall</field>
        <field name="subject">We want to hear from you !</field>
        <field name="state">in_queue</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="schedule_date" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="campaign_id" eval="ref('mass_mailing.mass_mail_campaign_1')"/>
        <field name="source_id" ref="utm.utm_source_mailing"/>
        <field name="mailing_model_id" ref="crm.model_crm_lead"/>
        <field name="mailing_domain">[]</field>
        <field name="reply_to_mode">email</field>
        <field name="reply_to">${object.company_id.email}</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 24px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="100%" style="background-color: white; padding: 0; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Hello</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        ${object.name}
                    </span>
                </td><td valign="middle" align="right">
                    <img src="${'/logo.png?company=%s' % object.company_id.id}" style="padding: 0px; margin: 0px; height: 48px;" alt="${object.company_id.name}"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:4px 0px 32px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td style="min-width: 590px;">
            <p>Dear ${object.name}</p>
            <p>Great stories have personality. Consider telling a great story that provides personality. Writing a story with personality for potential clients will assist with making a relationship connection. This shows up in small quirks like word choices or phrases. Write from your point of view, not from someone else's experience.</p>
            <p>Great stories are for everyone even when only written for just one person. If you try to write with a wide general audience in mind, your story will ring false and be bland. No one will be interested. Write for one person. If it’s genuine for the one, it’s genuine for the rest.</p>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px; padding: 0 8px 0 8px; font-size:11px;">
            <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 4px 0px;"/>
            <b>${object.company_id.name}</b><br/>
            <div style="color: #999999;">
                ${object.company_id.phone}
                % if object.company_id.email
                |
                    <a href="${'mailto:%s' % object.company_id.email}" style="text-decoration:none; color: #999999;">
                        ${object.company_id.email}
                    </a>
                % endif
                % if object.company_id.website
                |
                    <a href="${'%s' % object.company_id.website}" style="text-decoration:none; color: #999999;">
                        ${object.company_id.website}
                    </a>
                % endif
            </div>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=email" style="color: #875A7B;">Odoo</a>
</td></tr>
</table></field>
    </record>
</data>
</odoo>

```

## File: models\mailing_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import api, fields, models
from odoo.osv import expression


class MassMailing(models.Model):
    _name = 'mailing.mailing'
    _inherit = 'mailing.mailing'

    crm_lead_activated = fields.Boolean('Use Leads', compute='_compute_crm_lead_activated')
    crm_lead_count = fields.Integer('Lead Count', groups='sales_team.group_sale_salesman', compute='_compute_crm_lead_and_opportunities_count')
    crm_opportunities_count = fields.Integer('Opportunities Count', groups='sales_team.group_sale_salesman', compute='_compute_crm_lead_and_opportunities_count')

    def _compute_crm_lead_activated(self):
        for mass_mailing in self:
            mass_mailing.crm_lead_activated = self.env.user.has_group('crm.group_use_lead')

    @api.depends('crm_lead_activated')
    def _compute_crm_lead_and_opportunities_count(self):
        for mass_mailing in self:
            lead_and_opportunities_count = mass_mailing.crm_lead_count = self.env['crm.lead'] \
                    .with_context(active_test=False) \
                    .search_count(self._get_crm_utm_domain())
            if mass_mailing.crm_lead_activated:
                mass_mailing.crm_lead_count = lead_and_opportunities_count
                mass_mailing.crm_opportunities_count = 0
            else:
                mass_mailing.crm_lead_count = 0
                mass_mailing.crm_opportunities_count = lead_and_opportunities_count

    def action_redirect_to_leads(self):
        action = self.env.ref('crm.crm_lead_all_leads').read()[0]
        action['domain'] = self._get_crm_utm_domain()
        action['context'] = {'default_type': 'lead', 'active_test': False, 'create': False}
        return action

    def action_redirect_to_opportunities(self):
        action = self.env.ref('crm.crm_lead_opportunities').read()[0]
        action['view_mode'] = 'tree,kanban,graph,pivot,form,calendar'
        action['domain'] = self._get_crm_utm_domain()
        action['context'] = {'active_test': False, 'create': False}
        return action

    def _get_crm_utm_domain(self):
        """ We want all records that match the UTMs """
        domain = []
        if self.campaign_id:
            domain = expression.AND([domain, [('campaign_id', '=', self.campaign_id.id)]])
        if self.source_id:
            domain = expression.AND([domain, [('source_id', '=', self.source_id.id)]])
        if self.medium_id:
            domain = expression.AND([domain, [('medium_id', '=', self.medium_id.id)]])
        if not domain:
            domain = expression.AND([domain, [(0, '=', 1)]])

        return domain

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_mailing

```

## File: views\mailing_mailing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_mailing_view_form" model="ir.ui.view">
        <field name="name">mailing.mailing.view.form.inherit.crm</field>
        <field name="model">mailing.mailing</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_form"/>
        <field name="arch" type="xml">
    		<xpath expr="//button[@id='button_view_sent']" position="before">
                <field name="crm_lead_activated" invisible="1"/>
                <button name="action_redirect_to_leads"
                    type="object"
                    icon="fa-star"
                    class="oe_stat_button"
                    groups="sales_team.group_sale_salesman"
                    attrs="{'invisible': ['|', ('state', '=', 'draft'), ('crm_lead_activated', '=', False)]}" >
                    <field name="crm_lead_count" string="Leads" widget="statinfo"/>
                </button>
                <button name="action_redirect_to_opportunities"
                    type="object"
                    icon="fa-star"
                    class="oe_stat_button"
                    groups="sales_team.group_sale_salesman"
                    attrs="{'invisible': ['|', ('state', '=', 'draft'), ('crm_lead_activated', '=', True)]}" >
                    <field name="crm_opportunities_count" string="Opportunities" widget="statinfo"/>
                </button>
    		</xpath>
        </field>
    </record>
</odoo>
```

