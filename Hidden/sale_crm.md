# Odoo Module: sale_crm

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

from odoo import api, SUPERUSER_ID

def uninstall_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
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
    'depends': ['sale_management', 'crm'],
    'data': [
        'security/ir.model.access.csv',
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

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import api, fields, models


class CrmLead(models.Model):
    _inherit = 'crm.lead'

    sale_amount_total = fields.Monetary(compute='_compute_sale_data', string="Sum of Orders", help="Untaxed Total of Confirmed Orders", currency_field='company_currency')
    quotation_count = fields.Integer(compute='_compute_sale_data', string="Number of Quotations")
    sale_order_count = fields.Integer(compute='_compute_sale_data', string="Number of Sale Orders")
    order_ids = fields.One2many('sale.order', 'opportunity_id', string='Orders')

    @api.depends('order_ids.state', 'order_ids.currency_id', 'order_ids.amount_untaxed', 'order_ids.date_order', 'order_ids.company_id')
    def _compute_sale_data(self):
        for lead in self:
            total = 0.0
            quotation_cnt = 0
            sale_order_cnt = 0
            company_currency = lead.company_currency or self.env.company.currency_id
            for order in lead.order_ids:
                if order.state in ('draft', 'sent'):
                    quotation_cnt += 1
                if order.state not in ('draft', 'sent', 'cancel'):
                    sale_order_cnt += 1
                    total += order.currency_id._convert(
                        order.amount_untaxed, company_currency, order.company_id, order.date_order or fields.Date.today())
            lead.sale_amount_total = total
            lead.quotation_count = quotation_cnt
            lead.sale_order_count = sale_order_cnt

    def action_sale_quotations_new(self):
        if not self.partner_id:
            return self.env["ir.actions.actions"]._for_xml_id("sale_crm.crm_quotation_partner_action")
        else:
            return self.action_new_quotation()

    def action_new_quotation(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale_crm.sale_action_quotations_new")
        action['context'] = {
            'search_default_opportunity_id': self.id,
            'default_opportunity_id': self.id,
            'search_default_partner_id': self.partner_id.id,
            'default_partner_id': self.partner_id.id,
            'default_campaign_id': self.campaign_id.id,
            'default_medium_id': self.medium_id.id,
            'default_origin': self.name,
            'default_source_id': self.source_id.id,
            'default_company_id': self.company_id.id or self.env.company.id,
            'default_tag_ids': [(6, 0, self.tag_ids.ids)]
        }
        if self.team_id:
            action['context']['default_team_id'] = self.team_id.id,
        if self.user_id:
            action['context']['default_user_id'] = self.user_id.id
        return action

    def action_view_sale_quotation(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_quotations_with_onboarding")
        action['context'] = {
            'search_default_draft': 1,
            'search_default_partner_id': self.partner_id.id,
            'default_partner_id': self.partner_id.id,
            'default_opportunity_id': self.id
        }
        action['domain'] = [('opportunity_id', '=', self.id), ('state', 'in', ['draft', 'sent'])]
        quotations = self.mapped('order_ids').filtered(lambda l: l.state in ('draft', 'sent'))
        if len(quotations) == 1:
            action['views'] = [(self.env.ref('sale.view_order_form').id, 'form')]
            action['res_id'] = quotations.id
        return action

    def action_view_sale_order(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_orders")
        action['context'] = {
            'search_default_partner_id': self.partner_id.id,
            'default_partner_id': self.partner_id.id,
            'default_opportunity_id': self.id,
        }
        action['domain'] = [('opportunity_id', '=', self.id), ('state', 'not in', ('draft', 'sent', 'cancel'))]
        orders = self.mapped('order_ids').filtered(lambda l: l.state not in ('draft', 'sent', 'cancel'))
        if len(orders) == 1:
            action['views'] = [(self.env.ref('sale.view_order_form').id, 'form')]
            action['res_id'] = orders.id
        return action

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
            return "AND state in ('sale', 'done', 'pos_done')"
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
                    <button string="New Quotation" name="action_sale_quotations_new" type="object" class="oe_highlight"
                        attrs="{'invisible': ['|', ('type', '=', 'lead'), '&amp;', ('probability', '=', 0), ('active', '=', False)]}"/>
                </xpath>
                <button name="action_schedule_meeting" position="after">
                    <button class="oe_stat_button" type="object"
                        name="action_view_sale_quotation" icon="fa-pencil-square-o" attrs="{'invisible': [('type', '=', 'lead')]}">
                        <field name="quotation_count" widget="statinfo" string="Quotations"/>
                    </button>
                    <button class="oe_stat_button" type="object" attrs="{'invisible': ['|', ('sale_order_count', '=', 0), ('type', '=', 'lead')]}"
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
            <field name="domain">['|', ('res_model_id', '=', False), ('res_model_id.model', 'in', ['crm.lead', 'sale.order', 'res.partner', 'product.template', 'product.product'])]</field>
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

    <record id="sales_team.crm_team_salesteams_act" model="ir.actions.act_window">
        <field name="context">{'in_sales_app': True}</field>
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
            <xpath expr="//group/group/field[@name='type']" position="attributes">
                <attribute name="groups">sale.group_delivery_invoice_address</attribute>
            </xpath>
            <xpath expr="//field[@name='child_ids']//field[@name='type']" position="attributes">
                <attribute name="groups">sale.group_delivery_invoice_address</attribute>
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
            <xpath expr="//group[@name='technical']" position="inside">
                <field name="opportunity_id" help="Log in the chatter from which opportunity the order originates" groups="base.group_no_one"/>
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
            self.lead_id.handle_partner_assignment(create_missing=True)
        elif self.action == 'exist':
            self.lead_id.handle_partner_assignment(force_partner_id=self.partner_id.id, create_missing=False)
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
                <group attrs="{'invisible': [('action','!=','exist')], 'required':[('action', '=','exist')]}">
                    <group>
                        <field name="partner_id" attrs="{'invisible': [('action','!=','exist')], 'required':[('action', '=','exist')]}"
                               context="{'res_partner_search_mode': 'customer'}" />
                    </group>
                </group>
                <footer>
                    <button name="action_apply" string="Confirm" type="object" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="crm_quotation_partner_action" model="ir.actions.act_window">
        <field name="name">New Quotation</field>
        <field name="type">ir.actions.act_window</field>
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

