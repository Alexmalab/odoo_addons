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
        <field name="campaign_id" ref="mass_mailing.mass_mail_campaign_1"/>
        <field name="source_id" ref="utm.utm_source_mailing"/>
        <field name="mailing_model_id" ref="crm.model_crm_lead"/>
        <field name="mailing_domain">[]</field>
        <field name="reply_to_mode">new</field>
        <field name="reply_to">{{ object.company_id.email }}</field>
        <field name="body_html" type="html">
<div class="oe_structure o_editable">
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
                        <t t-out="object.name" />
                    </span>
                </td><td valign="middle" align="right">
                    <img t-att-src="'/logo.png?company=%s' % object.company_id.id" style="padding: 0px; margin: 0px; height: 48px;" t-att-alt="object.company_id.name"/>
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
            <p>Dear <t t-out="object.name" /></p>
            <p>Great stories have personality. Consider telling a great story that provides personality. Writing a story with personality for potential clients will assist with making a relationship connection. This shows up in small quirks like word choices or phrases. Write from your point of view, not from someone else's experience.</p>
            <p>Great stories are for everyone even when only written for just one person. If you try to write with a wide general audience in mind, your story will ring false and be bland. No one will be interested. Write for one person. If it’s genuine for the one, it’s genuine for the rest.</p>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px; padding: 0 8px 0 8px; font-size:11px;">
            <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 4px 0px;"/>
            <b><t t-out="object.company_id.name" /></b><br/>
            <div style="color: #999999;">
                <t t-out="object.company_id.phone" />
                <t t-if="object.company_id.email">
                |
                    <a t-att-href="'mailto:%s' % object.company_id.email" style="text-decoration:none; color: #999999;">
                        <t t-out="object.company_id.email" />
                    </a>
                </t>
                <t t-if="object.company_id.website">
                |
                    <a t-att-href="'%s' % object.company_id.website" style="text-decoration:none; color: #999999;">
                        <t t-out="object.company_id.website" />
                    </a>
                </t>
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
</table></div></field>
    </record>
</data>
</odoo>

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class CrmLead(models.Model):
    _inherit = 'crm.lead'
    _mailing_enabled = True

```

## File: models\mailing_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _, tools


class MassMailing(models.Model):
    _name = 'mailing.mailing'
    _inherit = 'mailing.mailing'

    use_leads = fields.Boolean('Use Leads', compute='_compute_use_leads')
    crm_lead_count = fields.Integer('Leads/Opportunities Count', groups='sales_team.group_sale_salesman', compute='_compute_crm_lead_count')

    def _compute_use_leads(self):
        self.use_leads = self.env.user.has_group('crm.group_use_lead')

    def _compute_crm_lead_count(self):
        lead_data = self.env['crm.lead'].with_context(active_test=False).read_group(
            [('source_id', 'in', self.source_id.ids)],
            ['source_id'], ['source_id']
        )
        mapped_data = {datum['source_id'][0]: datum['source_id_count'] for datum in lead_data}
        for mass_mailing in self:
            mass_mailing.crm_lead_count = mapped_data.get(mass_mailing.source_id.id, 0)

    def action_redirect_to_leads_and_opportunities(self):
        view = 'crm.crm_lead_all_leads' if self.use_leads else 'crm.crm_lead_opportunities'
        action = self.env.ref(view).sudo().read()[0]
        action['view_mode'] = 'tree,kanban,graph,pivot,form,calendar'
        action['domain'] = [('source_id', 'in', self.source_id.ids)]
        action['context'] = {'active_test': False, 'create': False}
        return action

    def _prepare_statistics_email_values(self):
        self.ensure_one()
        values = super(MassMailing, self)._prepare_statistics_email_values()
        if not self.user_id:
            return values
        if not self.env['crm.lead'].check_access_rights('read', raise_exception=False):
            return values
        values['kpi_data'][1]['kpi_col1'] = {
            'value': tools.format_decimalized_number(self.crm_lead_count, decimal=0),
            'col_subtitle': _('LEADS'),
        }
        values['kpi_data'][1]['kpi_name'] = 'lead'
        return values

```

## File: models\utm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    ab_testing_winner_selection = fields.Selection(selection_add=[('crm_lead_count', 'Leads')])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import mailing_mailing
from . import utm

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
    		<xpath expr="//button[@id='button_view_delivered']" position="before">
                <button name="action_redirect_to_leads_and_opportunities"
                    type="object"
                    icon="fa-star"
                    class="oe_stat_button"
                    groups="sales_team.group_sale_salesman"
                    attrs="{'invisible': [('state', '=', 'draft')]}">
                    <div class="o_field_widget o_stat_info">
                        <field name="use_leads" invisible="1"/>
                        <span class="o_stat_value"><field nolabel="1" name="crm_lead_count"/></span>
                        <span class="o_stat_text" attrs="{'invisible': [('use_leads', '=', False)]}">Leads</span>
                        <span class="o_stat_text" attrs="{'invisible': [('use_leads', '=', True)]}">Opportunities</span>
                    </div>
                </button>
    		</xpath>
        </field>
    </record>
</odoo>
```

