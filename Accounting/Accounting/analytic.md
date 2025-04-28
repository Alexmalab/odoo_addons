# Odoo Module: analytic

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
    'name' : 'Analytic Accounting',
    'version': '1.1',
    'category': 'Accounting/Accounting',
    'depends' : ['base', 'mail', 'uom'],
    'description': """
Module for defining analytic accounting object.
===============================================

In Odoo, analytic accounts are linked to general accounts but are treated
totally independently. So, you can enter various different analytic operations
that have no counterpart in the general financial accounts.
    """,
    'data': [
        'security/analytic_security.xml',
        'security/ir.model.access.csv',
        'views/analytic_account_views.xml',
    ],
    'demo': [
        'data/analytic_demo.xml',
        'data/analytic_account_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\analytic_account_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Analytic Groups -->
        <record id="analytic_group_projects" model="account.analytic.group">
            <field name="name">Projects</field>
        </record>
        <record id="analytic_group_departments" model="account.analytic.group">
            <field name="name">Departments</field>
        </record>
        <record id="analytic_group_internal" model="account.analytic.group">
            <field name="name">Internal</field>
        </record>
        <!-- Analytic Accounts -->
        <record id="analytic_absences" model="account.analytic.account">
            <field name="name">Time Off</field>
            <field name="group_id" ref="analytic.analytic_group_internal"/>
        </record>
        <record id="analytic_internal" model="account.analytic.account">
            <field name="name">Operating Costs</field>
            <field name="group_id" ref="analytic.analytic_group_internal"/>
        </record>
        <record id="analytic_our_super_product" model="account.analytic.account">
            <field name="name">Our Super Product</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_seagate_p2" model="account.analytic.account">
            <field name="name">Seagate P2</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_millennium_industries" model="account.analytic.account">
            <field name="name">Millennium Industries</field>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_integration_c2c" model="account.analytic.account">
            <field name="name">CampToCamp</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_agrolait" model="account.analytic.account">
            <field name="name">Deco Addict</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_asustek" model="account.analytic.account">
            <field name="name">Asustek</field>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_deltapc" model="account.analytic.account">
            <field name="name">Delta PC</field>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_spark" model="account.analytic.account">
            <field name="name">Spark Systems</field>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_nebula" model="account.analytic.account">
            <field name="name">Nebula</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_luminous_technologies" model="account.analytic.account">
            <field name="name">Luminous Technologies</field>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_desertic_hispafuentes" model="account.analytic.account">
            <field name="name">Desertic - Hispafuentes</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_think_big_systems" model="account.analytic.account">
            <field name="name">Lumber Inc</field>
            <field name="partner_id" ref="base.res_partner_18"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_partners_camp_to_camp" model="account.analytic.account">
            <field name="name">Camp to Camp</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="group_id" ref="analytic.analytic_group_projects"/>
        </record>
        <record id="analytic_administratif" model="account.analytic.account">
            <field name="name">Administrative</field>
            <field name="group_id" ref="analytic.analytic_group_departments"/>
        </record>
        <record id="analytic_commercial_marketing" model="account.analytic.account">
            <field name="name">Commercial &amp; Marketing</field>
            <field name="group_id" ref="analytic.analytic_group_departments"/>
        </record>
        <record id="analytic_rd_department" model="account.analytic.account">
            <field name="name">Research &amp; Development</field>
            <field name="group_id" ref="analytic.analytic_group_departments"/>
        </record>
    </data>
</odoo>

```

## File: data\analytic_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="tag_contract" model="account.analytic.tag">
            <field name="name">Contracts</field>
        </record>

    </data>
</odoo>

```

## File: models\analytic_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from odoo import api, fields, models, _
from odoo.osv import expression
from odoo.exceptions import ValidationError


class AccountAnalyticDistribution(models.Model):
    _name = 'account.analytic.distribution'
    _description = 'Analytic Account Distribution'
    _rec_name = 'account_id'

    account_id = fields.Many2one('account.analytic.account', string='Analytic Account', required=True)
    percentage = fields.Float(string='Percentage', required=True, default=100.0)
    name = fields.Char(string='Name', related='account_id.name', readonly=False)
    tag_id = fields.Many2one('account.analytic.tag', string="Parent tag", required=True)

    _sql_constraints = [
        ('check_percentage', 'CHECK(percentage >= 0 AND percentage <= 100)',
         'The percentage of an analytic distribution should be between 0 and 100.')
    ]

class AccountAnalyticTag(models.Model):
    _name = 'account.analytic.tag'
    _description = 'Analytic Tags'
    name = fields.Char(string='Analytic Tag', index=True, required=True)
    color = fields.Integer('Color Index')
    active = fields.Boolean(default=True, help="Set active to false to hide the Analytic Tag without removing it.")
    active_analytic_distribution = fields.Boolean('Analytic Distribution')
    analytic_distribution_ids = fields.One2many('account.analytic.distribution', 'tag_id', string="Analytic Accounts")
    company_id = fields.Many2one('res.company', string='Company')

class AccountAnalyticGroup(models.Model):
    _name = 'account.analytic.group'
    _description = 'Analytic Categories'
    _parent_store = True
    _rec_name = 'complete_name'

    name = fields.Char(required=True)
    description = fields.Text(string='Description')
    parent_id = fields.Many2one('account.analytic.group', string="Parent", ondelete='cascade', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    parent_path = fields.Char(index=True)
    children_ids = fields.One2many('account.analytic.group', 'parent_id', string="Childrens")
    complete_name = fields.Char('Complete Name', compute='_compute_complete_name', store=True)
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)

    @api.depends('name', 'parent_id.complete_name')
    def _compute_complete_name(self):
        for group in self:
            if group.parent_id:
                group.complete_name = '%s / %s' % (group.parent_id.complete_name, group.name)
            else:
                group.complete_name = group.name

class AccountAnalyticAccount(models.Model):
    _name = 'account.analytic.account'
    _inherit = ['mail.thread']
    _description = 'Analytic Account'
    _order = 'code, name asc'
    _check_company_auto = True

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        """
            Override read_group to calculate the sum of the non-stored fields that depend on the user context
        """
        res = super(AccountAnalyticAccount, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)
        accounts = self.env['account.analytic.account']
        for line in res:
            if '__domain' in line:
                accounts = self.search(line['__domain'])
            if 'balance' in fields:
                line['balance'] = sum(accounts.mapped('balance'))
            if 'debit' in fields:
                line['debit'] = sum(accounts.mapped('debit'))
            if 'credit' in fields:
                line['credit'] = sum(accounts.mapped('credit'))
        return res

    @api.depends('line_ids.amount')
    def _compute_debit_credit_balance(self):
        Curr = self.env['res.currency']
        analytic_line_obj = self.env['account.analytic.line']
        domain = [
            ('account_id', 'in', self.ids),
            ('company_id', 'in', [False] + self.env.companies.ids)
        ]
        if self._context.get('from_date', False):
            domain.append(('date', '>=', self._context['from_date']))
        if self._context.get('to_date', False):
            domain.append(('date', '<=', self._context['to_date']))
        if self._context.get('tag_ids'):
            tag_domain = expression.OR([[('tag_ids', 'in', [tag])] for tag in self._context['tag_ids']])
            domain = expression.AND([domain, tag_domain])

        user_currency = self.env.company.currency_id
        credit_groups = analytic_line_obj.read_group(
            domain=domain + [('amount', '>=', 0.0)],
            fields=['account_id', 'currency_id', 'amount'],
            groupby=['account_id', 'currency_id'],
            lazy=False,
        )
        data_credit = defaultdict(float)
        for l in credit_groups:
            data_credit[l['account_id'][0]] += Curr.browse(l['currency_id'][0])._convert(
                l['amount'], user_currency, self.env.company, fields.Date.today())

        debit_groups = analytic_line_obj.read_group(
            domain=domain + [('amount', '<', 0.0)],
            fields=['account_id', 'currency_id', 'amount'],
            groupby=['account_id', 'currency_id'],
            lazy=False,
        )
        data_debit = defaultdict(float)
        for l in debit_groups:
            data_debit[l['account_id'][0]] += Curr.browse(l['currency_id'][0])._convert(
                l['amount'], user_currency, self.env.company, fields.Date.today())

        for account in self:
            account.debit = abs(data_debit.get(account.id, 0.0))
            account.credit = data_credit.get(account.id, 0.0)
            account.balance = account.credit - account.debit

    name = fields.Char(string='Analytic Account', index=True, required=True, tracking=True)
    code = fields.Char(string='Reference', index=True, tracking=True)
    active = fields.Boolean('Active', help="If the active field is set to False, it will allow you to hide the account without removing it.", default=True)

    group_id = fields.Many2one('account.analytic.group', string='Group', check_company=True)

    line_ids = fields.One2many('account.analytic.line', 'account_id', string="Analytic Lines")

    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)

    # use auto_join to speed up name_search call
    partner_id = fields.Many2one('res.partner', string='Customer', auto_join=True, tracking=True, check_company=True)

    balance = fields.Monetary(compute='_compute_debit_credit_balance', string='Balance')
    debit = fields.Monetary(compute='_compute_debit_credit_balance', string='Debit')
    credit = fields.Monetary(compute='_compute_debit_credit_balance', string='Credit')

    currency_id = fields.Many2one(related="company_id.currency_id", string="Currency", readonly=True)

    def name_get(self):
        res = []
        for analytic in self:
            name = analytic.name
            if analytic.code:
                name = '[' + analytic.code + '] ' + name
            if analytic.partner_id.commercial_partner_id.name:
                name = name + ' - ' + analytic.partner_id.commercial_partner_id.name
            res.append((analytic.id, name))
        return res

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        if operator not in ('ilike', 'like', '=', '=like', '=ilike'):
            return super(AccountAnalyticAccount, self)._name_search(name, args, operator, limit, name_get_uid=name_get_uid)
        args = args or []
        if operator == 'ilike' and not (name or '').strip():
            domain = []
        else:
            # `partner_id` is in auto_join and the searches using ORs with auto_join fields doesn't work
            # we have to cut the search in two searches ... https://github.com/odoo/odoo/issues/25175
            partner_ids = self.env['res.partner']._search([('name', operator, name)], limit=limit, access_rights_uid=name_get_uid)
            domain = ['|', '|', ('code', operator, name), ('name', operator, name), ('partner_id', 'in', partner_ids)]
        return self._search(expression.AND([domain, args]), limit=limit, access_rights_uid=name_get_uid)


class AccountAnalyticLine(models.Model):
    _name = 'account.analytic.line'
    _description = 'Analytic Line'
    _order = 'date desc, id desc'
    _check_company_auto = True

    @api.model
    def _default_user(self):
        return self.env.context.get('user_id', self.env.user.id)

    name = fields.Char('Description', required=True)
    date = fields.Date('Date', required=True, index=True, default=fields.Date.context_today)
    amount = fields.Monetary('Amount', required=True, default=0.0)
    unit_amount = fields.Float('Quantity', default=0.0)
    product_uom_id = fields.Many2one('uom.uom', string='Unit of Measure', domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_uom_id.category_id', readonly=True)
    account_id = fields.Many2one('account.analytic.account', 'Analytic Account', required=True, ondelete='restrict', index=True, check_company=True)
    partner_id = fields.Many2one('res.partner', string='Partner', check_company=True)
    user_id = fields.Many2one('res.users', string='User', default=_default_user)
    tag_ids = fields.Many2many('account.analytic.tag', 'account_analytic_line_tag_rel', 'line_id', 'tag_id', string='Tags', copy=True, check_company=True)
    company_id = fields.Many2one('res.company', string='Company', required=True, readonly=True, default=lambda self: self.env.company)
    currency_id = fields.Many2one(related="company_id.currency_id", string="Currency", readonly=True, store=True, compute_sudo=True)
    group_id = fields.Many2one('account.analytic.group', related='account_id.group_id', store=True, readonly=True, compute_sudo=True)

    @api.constrains('company_id', 'account_id')
    def _check_company_id(self):
        for line in self:
            if line.account_id.company_id and line.company_id.id != line.account_id.company_id.id:
                raise ValidationError(_('The selected account belongs to another company than the one you\'re trying to create an analytic item for'))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import analytic_account

```

## File: security\analytic_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

    <record id="analytic_comp_rule" model="ir.rule">
        <field name="name">Analytic multi company rule</field>
        <field name="model_id" ref="model_account_analytic_account"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="analytic_line_comp_rule" model="ir.rule">
        <field name="name">Analytic line multi company rule</field>
        <field name="model_id" ref="model_account_analytic_line"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record id="analytic_group_comp_rule" model="ir.rule">
        <field name="name">Analytic line multi company rule</field>
        <field name="model_id" ref="model_account_analytic_group"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="analytic_tag_comp_rule" model="ir.rule">
        <field name="name">Analytic line multi company rule</field>
        <field name="model_id" ref="model_account_analytic_tag"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>
</data>
<data noupdate="0">

    <record id="group_analytic_accounting" model="res.groups">
        <field name="name">Analytic Accounting</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_analytic_tags" model="res.groups">
        <field name="name">Analytic Accounting Tags</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

</data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_account_analytic_account,access_account_analytic_account,model_account_analytic_account,group_analytic_accounting,1,1,1,1
access_account_analytic_line,access_account_analytic_line,model_account_analytic_line,group_analytic_accounting,1,1,1,1
access_account_analytic_tag,access_account_analytic_tag,model_account_analytic_tag,group_analytic_accounting,1,1,1,1
access_account_analytic_group,access_account_analytic_group,model_account_analytic_group,group_analytic_accounting,1,1,1,1
access_account_analytic_distribution,access_account_analytic_distribution,model_account_analytic_distribution,group_analytic_accounting,1,1,1,1

```

## File: views\analytic_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="account_analytic_tag_tree_view" model="ir.ui.view">
            <field name="name">account.analytic.tag.tree</field>
            <field name="model">account.analytic.tag</field>
            <field name="arch" type="xml">
                <tree string="Analytic Tags">
                    <field name="name"/>
                    <field name="display_name" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="account_analytic_tag_form_view" model="ir.ui.view">
            <field name="name">account.analytic.tag.form</field>
            <field name="model">account.analytic.tag</field>
            <field name="arch" type="xml">
                <form string="Analytic Tags">
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="name"/>
                            <field name="active_analytic_distribution" groups="analytic.group_analytic_accounting"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <field name="analytic_distribution_ids" nolabel="1" widget="one2many"
                            attrs="{'invisible': [('active_analytic_distribution', '=', False)]}" groups="analytic.group_analytic_accounting">
                            <tree string="Analytic Distribution" editable="bottom">
                                <field name="account_id"/>
                                <field name="percentage"/>
                            </tree>
                        </field>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="account_analytic_tag_view_search" model="ir.ui.view">
            <field name="name">account.analytic.tag.view.search</field>
            <field name="model">account.analytic.tag</field>
            <field name="arch" type="xml">
                <search string="Search Analytic Tags">
                    <field name="name" />
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <record id="account_analytic_tag_action" model="ir.actions.act_window">
            <field name="name">Analytic Tags</field>
            <field name="res_model">account.analytic.tag</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new tag
              </p>
            </field>
        </record>

        <record id="account_analytic_group_form_view" model="ir.ui.view">
            <field name="name">account.analytic.group.form</field>
            <field name="model">account.analytic.group</field>
            <field name="arch" type="xml">
                <form string="Analytic Account Groups">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="parent_id"/>
                        </group>
                        <group>
                            <field name="description"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="account_analytic_group_tree_view" model="ir.ui.view">
            <field name="name">account.analytic.group.tree</field>
            <field name="model">account.analytic.group</field>
            <field name="arch" type="xml">
                <tree string="Analytic Account Groups">
                    <field name="name"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="account_analytic_group_action" model="ir.actions.act_window">
            <field name="name">Analytic Account Groups</field>
            <field name="res_model">account.analytic.group</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                  Click to add a new analytic account group.
              </p>
              <p>
                  This allows you to classify your analytic accounts.
              </p>
            </field>
        </record>


        <record id="view_account_analytic_line_tree" model="ir.ui.view">
            <field name="name">account.analytic.line.tree</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <tree string="Analytic Entries">
                    <field name="date" optional="show"/>
                    <field name="name"/>
                    <field name="account_id"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="unit_amount" sum="Quantity" optional="hide"/>
                    <field name="product_uom_id" optional="hide"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show"/>
                    <field name="amount" sum="Total" optional="show"/>
                    <field name="tag_ids" optional="hide" widget="many2many_tags"/>
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="account_analytic_line_action">
            <field name="context">{'search_default_group_date': 1, 'default_account_id': active_id}</field>
            <field name="domain">[('account_id','=', active_id)]</field>
            <field name="name">Costs &amp; Revenues</field>
            <field name="res_model">account.analytic.line</field>
            <field name="view_mode">tree,form,graph,pivot</field>
            <field name="view_id" ref="view_account_analytic_line_tree"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_empty_folder">
                No activity yet on this account
              </p><p>
                In Odoo, sales orders and projects are implemented using
                analytic accounts. You can track costs and revenues to analyse
                your margins easily.
              </p><p>
                Costs will be created automatically when you register supplier
                invoices, expenses or timesheets.
              </p><p>
                Revenues will be created automatically when you create customer
                invoices. Customer invoices can be created based on sales orders
                (fixed price invoices), on timesheets (based on the work done) or
                on expenses (e.g. reinvoicing of travel costs).
              </p>
            </field>
        </record>

        <record id="view_account_analytic_account_form" model="ir.ui.view">
            <field name="name">analytic.analytic.account.form</field>
            <field name="model">account.analytic.account</field>
            <field name="arch" type="xml">
                <form string="Analytic Account">
                    <sheet string="Analytic Account">
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" type="action" name="%(account_analytic_line_action)d"
                            icon="fa-usd"  string="Cost/Revenue" widget="statinfo"/>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only"/>
                            <h1>
                                <field name="name" class="oe_inline" placeholder="e.g. Project XYZ"/>
                            </h1>
                        </div>
                        <div name="project"/>
                        <group name="main">
                            <group>
                                <field name="active" invisible="1"/>
                                <field name="code"/>
                                <field name="partner_id"/>
                            </group>
                            <group>
                                <field name="group_id"/>
                                <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                                <field name="currency_id" options="{'no_create': True}" groups="base.group_multi_currency"/>
                            </group>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="view_account_analytic_account_list" model="ir.ui.view">
            <field name="name">account.analytic.account.list</field>
            <field name="model">account.analytic.account</field>
            <field eval="8" name="priority"/>
            <field name="arch" type="xml">
                <tree string="Analytic Accounts">
                    <field name="name" string="Name"/>
                    <field name="code"/>
                    <field name="partner_id"/>
                    <field name="active" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="debit" sum="Debit"/>
                    <field name="credit" sum="Credit"/>
                    <field name="balance" sum="Balance"/>
                </tree>
            </field>
        </record>

        <record id="view_account_analytic_account_kanban" model="ir.ui.view">
            <field name="name">account.analytic.account.kanban</field>
            <field name="model">account.analytic.account</field>
            <field name="arch" type="xml">
               <kanban class="o_kanban_mobile">
                   <field name="display_name"/>
                   <field name="balance"/>
                   <field name="currency_id"/>
                   <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div t-attf-class="#{!selection_mode ? 'text-center' : ''}">
                                   <strong><span><field name="display_name"/></span></strong>
                                </div>
                                <hr class="mt8 mb8"/>
                                <div class="row">
                                    <div t-attf-class="col-12 #{!selection_mode ? 'text-center' : ''}">
                                        <span>
                                            Balance: <field name="balance" widget="monetary"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_account_analytic_account_search" model="ir.ui.view">
            <field name="name">account.analytic.account.search</field>
            <field name="model">account.analytic.account</field>
            <field name="arch" type="xml">
                <search string="Analytic Account">
                    <field name="name" filter_domain="['|', ('name', 'ilike', self), ('code', 'ilike', self)]" string="Analytic Account"/>
                    <field name="partner_id"/>
                    <separator/>
                    <filter string="Archived" domain="[('active', '=', False)]" name="inactive"/>
                    <group expand="0" string="Group By...">
                        <filter string="Associated Partner" name="associatedpartner" domain="[]" context="{'group_by': 'partner_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="action_analytic_account_form" model="ir.actions.act_window">
            <field name="name">Chart of Analytic Accounts</field>
            <field name="res_model">account.analytic.account</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="search_view_id" ref="view_account_analytic_account_search"/>
            <field name="context">{'search_default_active':1}</field>
            <field name="view_id" ref="view_account_analytic_account_list"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new analytic account
              </p>
            </field>
        </record>

        <record id="action_account_analytic_account_form" model="ir.actions.act_window">
            <field name="name">Analytic Accounts</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">account.analytic.account</field>
            <field name="search_view_id" ref="view_account_analytic_account_search"/>
            <field name="context">{'search_default_active':1}</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new analytic account
              </p>
            </field>
        </record>

        <record id="view_account_analytic_line_form" model="ir.ui.view">
            <field name="name">account.analytic.line.form</field>
            <field name="model">account.analytic.line</field>
            <field name="priority">1</field>
            <field name="arch" type="xml">
                <form string="Analytic Entry">
                <sheet>
                    <group>
                        <group name="analytic_entry" string="Analytic Entry">
                            <field name="name"/>
                            <field name="account_id"/>
                            <field name="tag_ids" widget="many2many_tags"/>
                            <field name="date"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <group name="amount" string="Amount">
                            <field name="amount"/>
                            <field name="unit_amount"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <field name="product_uom_id" class="oe_inline"/>
                            <field name="currency_id" invisible="1"/>
                        </group>
                    </group>
                </sheet>
                </form>
            </field>
        </record>

        <record id="view_account_analytic_line_filter" model="ir.ui.view">
            <field name="name">account.analytic.line.select</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <search string="Search Analytic Lines">
                    <field name="name"/>
                    <field name="date"/>
                    <field name="account_id"/>
                    <field name="tag_ids"/>
                    <filter string="Date" name="date" date="date"/>
                    <group string="Group By..." expand="0" name="groupby">
                        <filter string="Analytic Account" name="account_id" context="{'group_by': 'account_id'}"/>
                        <filter string="Date" name="group_date" context="{'group_by': 'date'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="view_account_analytic_line_graph" model="ir.ui.view">
            <field name="name">account.analytic.line.graph</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <graph string="Analytic Entries" sample="1">
                    <field name="account_id" type="row"/>
                    <field name="unit_amount" type="measure"/>
                    <field name="amount" type="measure"/>
                </graph>
            </field>
        </record>

        <record id="view_account_analytic_line_pivot" model="ir.ui.view">
            <field name="name">account.analytic.line.pivot</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <pivot string="Analytic Entries" sample="1">
                    <field name="account_id" type="row"/>
                    <field name="unit_amount" type="measure"/>
                    <field name="amount" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="view_account_analytic_line_kanban" model="ir.ui.view">
            <field name="name">account.analytic.line.kanban</field>
            <field name="model">account.analytic.line</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="date"/>
                    <field name="name"/>
                    <field name="account_id"/>
                    <field name="currency_id"/>
                    <field name="amount"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-6">
                                        <strong><span><t t-esc="record.name.value"/></span></strong>
                                    </div>
                                    <div class="col-6 text-right">
                                        <strong><t t-esc="record.date.value"/></strong>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-6 text-muted">
                                        <span><t t-esc="record.account_id.value"/></span>
                                    </div>
                                    <div class="col-6">
                                        <span class="float-right text-right">
                                            <field name="amount" widget="monetary"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record model="ir.actions.act_window" id="account_analytic_line_action_entries">
            <field name="name">Analytic Items</field>
            <field name="res_model">account.analytic.line</field>
            <field name="view_mode">tree,kanban,form,graph,pivot</field>
            <field name="view_id" ref="view_account_analytic_line_tree"/>
            <field name="search_view_id" ref="analytic.view_account_analytic_line_filter"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_empty_folder">
                No activity yet
              </p><p>
                In Odoo, sales orders and projects are implemented using
                analytic accounts. You can track costs and revenues to analyse
                your margins easily.
              </p><p>
                Costs will be created automatically when you register supplier
                invoices, expenses or timesheets.
              </p><p>
                Revenues will be created automatically when you create customer
                invoices. Customer invoices can be created based on sales orders
                (fixed price invoices), on timesheets (based on the work done) or
                on expenses (e.g. reinvoicing of travel costs).
              </p>
            </field>
        </record>
</odoo>

```

