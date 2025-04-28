# Odoo Module: sale_crm

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

def uninstall_hook(env):
    teams = env['crm.team'].search([('use_opportunities', '=', False)])
    teams.write({'use_opportunities': True})
```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Opportunity to Quotation',
    'version': '1.0',
    'category': 'Hidden',
    'description': """
This module adds a shortcut on one or several opportunity cases in the CRM.
===========================================================================

This shortcut allows you to generate a sales order based on the selected case.
If different cases are open (a list), it generates one sales order by case.
The case is then closed and linked to the generated sales order.

We suggest you to install this module, if you installed both the sale and the crm
modules.
    """,
    'depends': ['sale', 'crm'],
    'data': [
        'security/ir.model.access.csv',
        'data/crm_lead_merge_template.xml',
        'views/partner_views.xml',
        'views/sale_order_views.xml',
        'views/crm_lead_views.xml',
        'views/crm_team_views.xml',
        'wizard/crm_opportunity_to_quotation_views.xml'
    ],
    'auto_install': True,
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: data\crm_lead_merge_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="crm_lead_merge_summary_inherit_sale_crm" inherit_id="crm.crm_lead_merge_summary">
    <xpath expr="//blockquote/*[last()]" position="after">
        <div t-if="lead.order_ids">
            <br/>
            <div class="fw-bold">
                Sale Orders
            </div>
            <ul>
                <li t-foreach="lead.order_ids" t-as="order" t-esc="order.name"/>
            </ul>
        </div>
    </xpath>
</template>

</odoo>

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import api, fields, models
from odoo.osv import expression


class CrmLead(models.Model):
    _inherit = 'crm.lead'

    sale_amount_total = fields.Monetary(compute='_compute_sale_data', string="Sum of Orders", help="Untaxed Total of Confirmed Orders", currency_field='company_currency')
    quotation_count = fields.Integer(compute='_compute_sale_data', string="Number of Quotations")
    sale_order_count = fields.Integer(compute='_compute_sale_data', string="Number of Sale Orders")
    order_ids = fields.One2many('sale.order', 'opportunity_id', string='Orders')

    @api.depends('order_ids.state', 'order_ids.currency_id', 'order_ids.amount_untaxed', 'order_ids.date_order', 'order_ids.company_id')
    def _compute_sale_data(self):
        for lead in self:
            company_currency = lead.company_currency or self.env.company.currency_id
            sale_orders = lead.order_ids.filtered_domain(self._get_lead_sale_order_domain())
            lead.sale_amount_total = sum(
                order.currency_id._convert(
                    order.amount_untaxed, company_currency, order.company_id, order.date_order or fields.Date.today()
                )
                for order in sale_orders
            )
            lead.quotation_count = len(lead.order_ids.filtered_domain(self._get_lead_quotation_domain()))
            lead.sale_order_count = len(sale_orders)

    def action_sale_quotations_new(self):
        if not self.partner_id:
            return self.env["ir.actions.actions"]._for_xml_id("sale_crm.crm_quotation_partner_action")
        else:
            return self.action_new_quotation()

    def action_new_quotation(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale_crm.sale_action_quotations_new")
        action['context'] = self._prepare_opportunity_quotation_context()
        action['context']['search_default_opportunity_id'] = self.id
        return action

    def action_view_sale_quotation(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_quotations_with_onboarding")
        action['context'] = self._prepare_opportunity_quotation_context()
        action['context']['search_default_draft'] = 1
        action['domain'] = expression.AND([[('opportunity_id', '=', self.id)], self._get_action_view_sale_quotation_domain()])
        quotations = self.order_ids.filtered_domain(self._get_action_view_sale_quotation_domain())
        if len(quotations) == 1:
            action['views'] = [(self.env.ref('sale.view_order_form').id, 'form')]
            action['res_id'] = quotations.id
        return action

    def action_view_sale_order(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_orders")
        action['context'] = {
            'search_default_partner_id': self.partner_id.id,
            'default_partner_id': self.partner_id.id,
            'default_opportunity_id': self.id,
        }
        action['domain'] = expression.AND([[('opportunity_id', '=', self.id)], self._get_lead_sale_order_domain()])
        orders = self.order_ids.filtered_domain(self._get_lead_sale_order_domain())
        if len(orders) == 1:
            action['views'] = [(self.env.ref('sale.view_order_form').id, 'form')]
            action['res_id'] = orders.id
        return action

    def _get_action_view_sale_quotation_domain(self):
        return [('state', 'in', ('draft', 'sent', 'cancel'))]

    def _get_lead_quotation_domain(self):
        return [('state', 'in', ('draft', 'sent'))]

    def _get_lead_sale_order_domain(self):
        return [('state', 'not in', ('draft', 'sent', 'cancel'))]

    def _prepare_opportunity_quotation_context(self):
        """ Prepares the context for a new quotation (sale.order) by sharing the values of common fields """
        self.ensure_one()
        quotation_context = {
            'default_opportunity_id': self.id,
            'default_partner_id': self.partner_id.id,
            'default_campaign_id': self.campaign_id.id,
            'default_medium_id': self.medium_id.id,
            'default_origin': self.name,
            'default_source_id': self.source_id.id,
            'default_company_id': self.company_id.id or self.env.company.id,
            'default_tag_ids': [(6, 0, self.tag_ids.ids)]
        }
        if self.team_id:
            quotation_context['default_team_id'] = self.team_id.id
        if self.user_id:
            quotation_context['default_user_id'] = self.user_id.id
        return quotation_context

    def _merge_get_fields_specific(self):
        fields_info = super(CrmLead, self)._merge_get_fields_specific()
        # add all the orders from all lead to merge
        fields_info['order_ids'] = lambda fname, leads: [(4, order.id) for order in leads.order_ids]
        return fields_info

```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models,fields, api, _


class CrmTeam(models.Model):
    _inherit = 'crm.team'

    def _compute_dashboard_button_name(self):
        super(CrmTeam, self)._compute_dashboard_button_name()
        teams_with_opp = self.filtered(lambda team: team.use_opportunities)
        if self._context.get('in_sales_app'):
            teams_with_opp.update({'dashboard_button_name': _("Sales Analysis")})

    def action_primary_channel_button(self):
        if self._context.get('in_sales_app') and self.use_opportunities:
            return self.env["ir.actions.actions"]._for_xml_id("sale.action_order_report_so_salesteam")
        return super(CrmTeam,self).action_primary_channel_button()

    def _graph_get_model(self):
        if self.use_opportunities and self._context.get('in_sales_app') :
            return 'sale.report'
        return super(CrmTeam,self)._graph_get_model()

    def _graph_date_column(self):
        if self.use_opportunities and self._context.get('in_sales_app'):
            return 'date'
        return super(CrmTeam,self)._graph_date_column()

    def _graph_y_query(self):
        if self.use_opportunities and self._context.get('in_sales_app'):
            return 'SUM(price_subtotal)'
        return super(CrmTeam,self)._graph_y_query()

    def _graph_title_and_key(self):
        if self.use_opportunities and self._context.get('in_sales_app'):
            return ['', _('Sales: Untaxed Total')]
        return super(CrmTeam,self)._graph_title_and_key()

    def _extra_sql_conditions(self):
        if self.use_opportunities and self._context.get('in_sales_app'):
            return "AND state = 'sale'"
        return super(CrmTeam,self)._extra_sql_conditions()

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResUsers(models.Model):
    _inherit = 'res.users'

    target_sales_invoiced = fields.Integer('Invoiced in Sales Orders Target')

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    opportunity_id = fields.Many2one(
        'crm.lead', string='Opportunity', check_company=True,
        domain="[('type', '=', 'opportunity'), '|', ('company_id', '=', False), ('company_id', '=', company_id)]")

    def action_confirm(self):
        return super(SaleOrder, self.with_context({k:v for k,v in self._context.items() if k != 'default_tag_ids'})).action_confirm()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import crm_team
from . import res_users
from . import sale_order

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_crm_quotation_partner,access.crm.quotation.partner,model_crm_quotation_partner,sales_team.group_sale_salesman,1,1,1,0

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="crm_case_form_view_oppor" model="ir.ui.view">
            <field name="name">crm.lead.oppor.inherited.crm</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_lead_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_set_won_rainbowman']" position="before">
                    <button string="New Quotation" name="action_sale_quotations_new" type="object" class="oe_highlight" data-hotkey="q"
                        title="Create new quotation"
                        invisible="type == 'lead' or probability == 0 and not active"/>
                </xpath>
                <button name="action_set_won_rainbowman" position="attributes">
                    <attribute name="class" remove="oe_highlight"/>
                </button>
                <button name="action_schedule_meeting" position="after">
                    <button class="oe_stat_button" type="object"
                        name="action_view_sale_quotation" icon="fa-pencil-square-o" invisible="type == 'lead'">
                        <field name="quotation_count" widget="statinfo" string="Quotations"/>
                    </button>
                    <button class="oe_stat_button" type="object" invisible="sale_order_count == 0 or type == 'lead'"
                        name="action_view_sale_order" icon="fa-usd">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value"><field name="sale_amount_total" widget="monetary" options="{'currency_field': 'company_currency'}"/></span>
                            <span class="o_stat_text"> Orders</span>
                            <field name="sale_order_count" invisible="1"/>
                        </div>
                    </button>
                </button>
            </field>
        </record>

        <!-- add needaction_menu_ref to reload quotation needaction when opportunity needaction is reloaded -->
        <record id="crm.crm_lead_opportunities" model="ir.actions.act_window">
            <field name="context">{'default_type': 'opportunity', 'default_user_id': uid}</field>
        </record>

        <record id="sales_team.mail_activity_type_action_config_sales" model="ir.actions.act_window">
            <field name="domain">['|', ('res_model', '=', False), ('res_model', 'in', ['crm.lead', 'sale.order', 'res.partner', 'product.template', 'product.product'])]</field>
        </record>

</odoo>

```

## File: views\crm_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_team_salesteams_view_form_in_sale_crm" model="ir.ui.view">
        <field name="name">crm.team.form</field>
        <field name="model">crm.team</field>
        <field name="inherit_id" ref="crm.sales_team_form_view_in_crm"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//span[@name='opportunities']" position="attributes">
                    <attribute name="invisible" value="0"/>
                </xpath>
            </data>
        </field>
    </record>
</odoo>

```

## File: views\partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_partner_address_type" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.base</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form" />
        <field name="arch" type="xml">
            <xpath expr="//group/group/span/field[@name='type']" position="attributes">
                <attribute name="groups">account.group_delivery_invoice_address</attribute>
            </xpath>
            <xpath expr="//field[@name='child_ids']//field[@name='type']" position="attributes">
                <attribute name="groups">account.group_delivery_invoice_address</attribute>
            </xpath>
            <xpath expr="//group/group/span/field[@name='type']" position="after">
                <field name="type" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='child_ids']//field[@name='type']" position="after">
                <field name="type" invisible="1"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="sale_action_quotations_new" model="ir.actions.act_window">
        <field name="name">Quotation</field>
        <field name="res_model">sale.order</field>
        <field name="view_mode">form,tree,graph</field>
        <field name="domain">[('opportunity_id', '=', active_id)]</field>
        <field name="context">{'search_default_opportunity_id': active_id, 'default_opportunity_id': active_id}</field>
    </record>

    <record id="sale_view_inherit123" model="ir.ui.view">
        <field name="name">sale.order.form.inherit.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='origin']" position="after">
                <field name="opportunity_id" context="{
                    'default_campaign_id': campaign_id,
                    'default_company_id': company_id,
                    'default_medium_id': medium_id,
                    'default_partner_id': partner_id,
                    'default_source_id': source_id,
                    'default_tag_ids': tag_ids,
                    'default_type': 'opportunity',
                }"/>
            </xpath>
        </field>
    </record>

    <!-- This menu is display in CRM app when sale is installed-->
    <menuitem
        id="sale_order_menu_quotations_crm"
        name="My Quotations"
        action="sale.action_quotations"
        parent="crm.crm_menu_sales"
        sequence="2"/>

</odoo>

```

## File: wizard\crm_opportunity_to_quotation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class Opportunity2Quotation(models.TransientModel):
    _name = 'crm.quotation.partner'
    _description = 'Create new or use existing Customer on new Quotation'

    @api.model
    def default_get(self, fields):
        result = super(Opportunity2Quotation, self).default_get(fields)

        active_model = self._context.get('active_model')
        if active_model != 'crm.lead':
            raise UserError(_('You can only apply this action from a lead.'))

        lead = False
        if result.get('lead_id'):
            lead = self.env['crm.lead'].browse(result['lead_id'])
        elif 'lead_id' in fields and self._context.get('active_id'):
            lead = self.env['crm.lead'].browse(self._context['active_id'])
        if lead:
            result['lead_id'] = lead.id
            partner_id = result.get('partner_id') or lead._find_matching_partner().id
            if 'action' in fields and not result.get('action'):
                result['action'] = 'exist' if partner_id else 'create'
            if 'partner_id' in fields and not result.get('partner_id'):
                result['partner_id'] = partner_id

        return result

    action = fields.Selection([
        ('create', 'Create a new customer'),
        ('exist', 'Link to an existing customer'),
        ('nothing', 'Do not link to a customer')
    ], string='Quotation Customer', required=True)
    lead_id = fields.Many2one('crm.lead', "Associated Lead", required=True)
    partner_id = fields.Many2one('res.partner', 'Customer')

    def action_apply(self):
        """ Convert lead to opportunity or merge lead and opportunity and open
            the freshly created opportunity view.
        """
        self.ensure_one()
        if self.action == 'create':
            self.lead_id._handle_partner_assignment(create_missing=True)
        elif self.action == 'exist':
            self.lead_id._handle_partner_assignment(force_partner_id=self.partner_id.id, create_missing=False)
        return self.lead_id.action_new_quotation()

```

## File: wizard\crm_opportunity_to_quotation_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_quotation_partner_view_form" model="ir.ui.view">
        <field name="name">crm.quotation.partner.view.form</field>
        <field name="model">crm.quotation.partner</field>
        <field name="arch" type="xml">
            <form string="New Quotation">
                <group>
                    <group>
                        <field name="action" widget="radio"/>
                        <field name="lead_id" invisible="1"/>
                    </group>
                </group>
                <group invisible="action != 'exist'" required="action == 'exist'">
                    <group>
                        <field name="partner_id" invisible="action != 'exist'" required="action == 'exist'"
                               context="{'res_partner_search_mode': 'customer'}" />
                    </group>
                </group>
                <footer>
                    <button name="action_apply" string="Confirm" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="crm_quotation_partner_action" model="ir.actions.act_window">
        <field name="name">New Quotation</field>
        <field name="res_model">crm.quotation.partner</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="crm_quotation_partner_view_form"/>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_opportunity_to_quotation

```

