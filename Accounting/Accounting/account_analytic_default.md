# Odoo Module: account_analytic_default

Category: Accounting/Accounting

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
    'name': 'Account Analytic Defaults',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'description': """
Set default values for your analytic accounts.
==============================================

Allows to automatically select analytic accounts based on criterions:
---------------------------------------------------------------------
    * Product
    * Partner
    * User
    * Company
    * Date
    """,
    'depends': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'security/account_analytic_default_security.xml',
        'views/account_analytic_default_view.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: models\account_analytic_default.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class AccountAnalyticDefault(models.Model):
    _name = "account.analytic.default"
    _description = "Analytic Distribution"
    _rec_name = "analytic_id"
    _order = "sequence"

    sequence = fields.Integer(string='Sequence', help="Gives the sequence order when displaying a list of analytic distribution")
    analytic_id = fields.Many2one('account.analytic.account', string='Analytic Account')
    analytic_tag_ids = fields.Many2many('account.analytic.tag', string='Analytic Tags')
    product_id = fields.Many2one('product.product', string='Product', ondelete='cascade', help="Select a product which will use analytic account specified in analytic default (e.g. create new customer invoice or Sales order if we select this product, it will automatically take this as an analytic account)")
    partner_id = fields.Many2one('res.partner', string='Partner', ondelete='cascade', help="Select a partner which will use analytic account specified in analytic default (e.g. create new customer invoice or Sales order if we select this partner, it will automatically take this as an analytic account)")
    account_id = fields.Many2one('account.account', string='Account', ondelete='cascade', help="Select an accounting account which will use analytic account specified in analytic default (e.g. create new customer invoice or Sales order if we select this account, it will automatically take this as an analytic account)")
    user_id = fields.Many2one('res.users', string='User', ondelete='cascade', help="Select a user which will use analytic account specified in analytic default.")
    company_id = fields.Many2one('res.company', string='Company', ondelete='cascade', help="Select a company which will use analytic account specified in analytic default (e.g. create new customer invoice or Sales order if we select this company, it will automatically take this as an analytic account)")
    date_start = fields.Date(string='Start Date', help="Default start date for this Analytic Account.")
    date_stop = fields.Date(string='End Date', help="Default end date for this Analytic Account.")

    @api.constrains('analytic_id', 'analytic_tag_ids')
    def _check_account_or_tags(self):
        if any(not default.analytic_id
               and not any(tag.analytic_distribution_ids for tag in default.analytic_tag_ids)
               for default in self
               ):
            raise ValidationError(_('An analytic default requires an analytic account or an analytic tag used for analytic distribution.'))

    @api.model
    def account_get(self, product_id=None, partner_id=None, account_id=None, user_id=None, date=None, company_id=None):
        domain = []
        if product_id:
            domain += ['|', ('product_id', '=', product_id)]
        domain += [('product_id', '=', False)]
        if partner_id:
            domain += ['|', ('partner_id', '=', partner_id)]
        domain += [('partner_id', '=', False)]
        if account_id:
            domain += ['|', ('account_id', '=', account_id)]
        domain += [('account_id', '=', False)]
        if company_id:
            domain += ['|', ('company_id', '=', company_id)]
        domain += [('company_id', '=', False)]
        if user_id:
            domain += ['|', ('user_id', '=', user_id)]
        domain += [('user_id', '=', False)]
        if date:
            domain += ['|', ('date_start', '<=', date), ('date_start', '=', False)]
            domain += ['|', ('date_stop', '>=', date), ('date_stop', '=', False)]
        best_index = -1
        res = self.env['account.analytic.default']
        for rec in self.search(domain):
            index = 0
            if rec.product_id: index += 1
            if rec.partner_id: index += 1
            if rec.account_id: index += 1
            if rec.company_id: index += 1
            if rec.user_id: index += 1
            if rec.date_start: index += 1
            if rec.date_stop: index += 1
            if index > best_index:
                res = rec
                best_index = index
        return res


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    # Overload of fields defined in account
    analytic_account_id = fields.Many2one(compute="_compute_analytic_account", store=True, readonly=False, copy=True)
    analytic_tag_ids = fields.Many2many(compute="_compute_analytic_account", store=True, readonly=False, copy=True)

    @api.depends('product_id', 'account_id', 'partner_id', 'date_maturity')
    def _compute_analytic_account(self):
        for record in self:
            record.analytic_account_id = record.analytic_account_id
            record.analytic_tag_ids = record.analytic_tag_ids
            rec = self.env['account.analytic.default'].account_get(
                product_id=record.product_id.id,
                partner_id=record.partner_id.commercial_partner_id.id or record.move_id.partner_id.commercial_partner_id.id,
                account_id=record.account_id.id,
                user_id=record.env.uid,
                date=record.date_maturity,
                company_id=record.move_id.company_id.id
            )
            if rec and not record.exclude_from_invoice_tab:
                record.analytic_account_id = rec.analytic_id
                record.analytic_tag_ids = rec.analytic_tag_ids

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_analytic_default

```

## File: security\account_analytic_default_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

    <record id="analytic_default_comp_rule" model="ir.rule">
        <field name="name">Analytic Default multi company rule</field>
        <field name="model_id" ref="model_account_analytic_default"/>
        <field eval="True" name="global"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
    </record>
     
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_account_analytic_default,account.analytic.default,model_account_analytic_default,account.group_account_user,1,0,0,0
access_account_analytic_default_analytic,account.analytic.default analytic,model_account_analytic_default,analytic.group_analytic_accounting,1,0,0,0
access_account_analytic_default_invoice,account.analytic.default invoice,model_account_analytic_default,account.group_account_invoice,1,1,1,1

```

## File: views\account_analytic_default_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_account_analytic_default_tree" model="ir.ui.view">
            <field name="name">account.analytic.default.tree</field>
            <field name="model">account.analytic.default</field>
            <field name="arch" type="xml">
                <tree string="Analytic Defaults">
                    <field name="sequence" widget="handle"/>
                    <field name="analytic_id" required="0" groups="analytic.group_analytic_accounting"/>
                    <field name="analytic_tag_ids" widget="many2many_tags" groups="analytic.group_analytic_tags"/>
                    <field name="product_id"/>
                    <field name="partner_id"/>
                    <field name="user_id"/>
                    <field name="account_id"/>
                    <field name="date_start"/>
                    <field name="date_stop"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="view_account_analytic_default_form" model="ir.ui.view">
            <field name="name">account.analytic.default.form</field>
            <field name="model">account.analytic.default</field>
            <field name="arch" type="xml">
                <form string="Analytic Defaults">
                <sheet>
                    <group col="4">
                        <separator string="Analytic Default Rule" colspan="4"/>
                        <field name="analytic_id" groups="analytic.group_analytic_accounting" string="Account"/>
                        <field name="analytic_tag_ids" widget="many2many_tags" groups="analytic.group_analytic_tags" string="Tags"/>
                        <separator string="Conditions" colspan="4"/>
                        <field name="product_id"/>
                        <field name="partner_id"/>
                        <field name="user_id"/>
                        <field name="account_id"/>
                        <field name="date_start"/>
                        <field name="date_stop"/>
                        <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                    </group>
                </sheet>
                </form>
            </field>
        </record>

        <record id="view_account_analytic_default_kanban" model="ir.ui.view">
            <field name="name">account.analytic.default.kanban</field>
            <field name="model">account.analytic.default</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="analytic_id"/>
                    <field name="date_start"/>
                    <field name="date_stop"/>
                    <field name="product_id"/>
                    <field name="partner_id"/>
                    <field name="user_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div>
                                    <strong><span><field name="analytic_id"/></span></strong>
                                    <img t-att-src="kanban_image('res.users', 'image_128', record.user_id.raw_value)" t-att-title="record.user_id.value" t-att-alt="record.user_id.value" class="oe_kanban_avatar o_image_24_cover float-right"/>
                                </div>
                                <div t-if="record.date_start.value"><i class="fa fa-calendar"></i> From <field name="date_start"/> <t t-if="record.date_stop.value">to <field name="date_stop"/></t></div>
                                <div t-if="record.product_id.value"><strong>Product</strong> <field name="product_id"/> </div>
                                <div t-if="record.partner_id.value"><strong>Customer</strong> <field name="partner_id"/> </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_account_analytic_default_form_search" model="ir.ui.view">
            <field name="name">account.analytic.default.search</field>
            <field name="model">account.analytic.default</field>
            <field name="arch" type="xml">
                <search string="Accounts">
                    <field name="date_stop"/>
                    <field name="analytic_id" groups="analytic.group_analytic_accounting"/>
                    <field name="product_id"/>
                    <field name="partner_id"/>
                    <field name="user_id"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <group expand="0" string="Group By">
                        <filter string="User" name="user" context="{'group_by':'user_id'}" help="User"/>
                        <filter string="Partner" name="partner" context="{'group_by':'partner_id'}" help="Partner"/>
                        <filter string="Product" name="product" context="{'group_by':'product_id'}" help="Product" />
                        <filter string="Analytic Account" name="analyticacc" context="{'group_by':'analytic_id'}" help="Analytic Account" groups="analytic.group_analytic_accounting"/>
                        <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company" />
                    </group>
                </search>
            </field>
        </record>

        <record id="action_analytic_default_list" model="ir.actions.act_window">
            <field name="name">Analytic Defaults</field>
            <field name="res_model">account.analytic.default</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="search_view_id" ref="view_account_analytic_default_form_search"/>
            <field name="context">{"search_default_current":1}</field>
        </record>

        <record id="action_product_default_list" model="ir.actions.act_window">
            <field name="name">Analytic Rules</field>
            <field name="res_model">account.analytic.default</field>
            <field name="context">{'search_default_product_id': [active_id], 'default_product_id': active_id}</field>
        </record>

        <menuitem id="menu_analytic_default_list"
            action="action_analytic_default_list"
            parent="account.menu_analytic_accounting"/>

        <act_window
            name="Analytic Rules"
            id="analytic_rule_action_user"
            res_model="account.analytic.default"
            binding_model="res.users"
            binding_views="form"
            context="{'search_default_user_id': [active_id], 'default_user_id': active_id}"
            groups="analytic.group_analytic_accounting"/>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.account.analytic.default</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='analytic_account_link']" position="after">
                <div class="mt8">
                    <button name="%(account_analytic_default.action_analytic_default_list)d" icon="fa-arrow-right" type="action" string="Default Analytic Values" class="btn-link"/>
                </div>
            </xpath>
        </field>
    </record>

</odoo>


```

