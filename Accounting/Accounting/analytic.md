# Odoo Module: analytic

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import populate

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name' : 'Analytic Accounting',
    'version': '1.2',
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
        'views/analytic_line_views.xml',
        'views/analytic_account_views.xml',
        'views/analytic_plan_views.xml',
        'views/analytic_distribution_model_views.xml',
        'data/analytic_data.xml'
    ],
    'demo': [
        'data/analytic_account_demo.xml'
    ],
    'assets': {
        'web.assets_backend': [
            'analytic/static/src/components/**/*',
            'analytic/static/src/services/**/*',
        ],
        'web.qunit_suite_tests': [
            'analytic/static/tests/*.js',
        ],
    },
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\analytic_account_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Analytic Plans -->
        <record id="analytic_plan_departments" model="account.analytic.plan">
            <field name="name">Departments</field>
            <field name="default_applicability">optional</field>
        </record>
        <record id="analytic_plan_internal" model="account.analytic.plan">
            <field name="name">Internal</field>
            <field name="default_applicability">unavailable</field>
        </record>
        <!-- Analytic Accounts -->
        <record id="analytic_absences" model="account.analytic.account">
            <field name="name">Time Off</field>
            <field name="plan_id" ref="analytic.analytic_plan_internal"/>
        </record>
        <record id="analytic_internal" model="account.analytic.account">
            <field name="name">Operating Costs</field>
            <field name="plan_id" ref="analytic.analytic_plan_internal"/>
        </record>
        <record id="analytic_our_super_product" model="account.analytic.account">
            <field name="name">Our Super Product</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_seagate_p2" model="account.analytic.account">
            <field name="name">Seagate P2</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_millennium_industries" model="account.analytic.account">
            <field name="name">Millennium Industries</field>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_integration_c2c" model="account.analytic.account">
            <field name="name">CampToCamp</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_agrolait" model="account.analytic.account">
            <field name="name">Deco Addict</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_asustek" model="account.analytic.account">
            <field name="name">Asustek</field>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_deltapc" model="account.analytic.account">
            <field name="name">Delta PC</field>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_spark" model="account.analytic.account">
            <field name="name">Spark Systems</field>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_nebula" model="account.analytic.account">
            <field name="name">Nebula</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_luminous_technologies" model="account.analytic.account">
            <field name="name">Luminous Technologies</field>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_desertic_hispafuentes" model="account.analytic.account">
            <field name="name">Desertic - Hispafuentes</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_think_big_systems" model="account.analytic.account">
            <field name="name">Lumber Inc</field>
            <field name="partner_id" ref="base.res_partner_18"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_partners_camp_to_camp" model="account.analytic.account">
            <field name="name">Camp to Camp</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_active_account" model="account.analytic.account">
            <field name="name">Active account</field>
            <field name="active" eval="True"/>
            <field name="plan_id" ref="analytic.analytic_plan_projects"/>
        </record>
        <record id="analytic_administratif" model="account.analytic.account">
            <field name="name">Administrative</field>
            <field name="plan_id" ref="analytic.analytic_plan_departments"/>
        </record>
        <record id="analytic_commercial_marketing" model="account.analytic.account">
            <field name="name">Commercial &amp; Marketing</field>
            <field name="plan_id" ref="analytic.analytic_plan_departments"/>
        </record>
        <record id="analytic_rd_department" model="account.analytic.account">
            <field name="name">Research &amp; Development</field>
            <field name="plan_id" ref="analytic.analytic_plan_departments"/>
        </record>
    </data>
</odoo>

```

## File: data\analytic_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="decimal_percentage_analytic" model="decimal.precision">
            <field name="name">Percentage Analytic</field>
            <field name="digits" eval="2"/>
        </record>

        <!--
            Save which plan is used for Projects. Once it is set, even if not used, it cannot be changed safely without a SQL script.
            The other plans will generate dynamic columns, so changing this param would require renaming the columns also.
        -->
        <function model="ir.config_parameter" name="set_param" eval="('analytic.project_plan', '1')"/>
        <record id="analytic_plan_projects" model="account.analytic.plan">
            <field name="name">Projects</field>
            <field name="default_applicability">optional</field>
        </record>
    </data>
</odoo>

```

## File: migrations\1.2\pre-migrate.py

```python
from odoo.tools import sql


def migrate(cr, version):
    # Select relevant ids to generate the list of x_plan_id column names, removing the id of the project plan
    cr.execute(
        """
        SELECT value::int
          FROM ir_config_parameter
         WHERE key = 'analytic.project_plan'
        """
    )
    [project_plan_id] = cr.fetchone()
    cr.execute("SELECT id FROM account_analytic_plan WHERE id != %s AND parent_id IS NULL", [project_plan_id])
    plan_ids = [r[0] for r in cr.fetchall()]
    column_names = [f"x_plan{id_}_id" for id_ in plan_ids]
    # Update on_delete for existing x_plan_id columns
    cr.execute(
        """
        UPDATE ir_model_fields
           SET on_delete = 'restrict'
         WHERE model = 'account.analytic.line'
           AND on_delete = 'set null'
           AND name = ANY(%s)
        """,
        [column_names],
    )
    # Change the constraint on the table definition
    for column in column_names:
        sql.drop_constraint(cr, 'account_analytic_line', f'account_analytic_line_{column}_fkey')
        sql.add_foreign_key(cr, 'account_analytic_line', column, 'account_analytic_account', 'id', 'restrict')

```

## File: models\analytic_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
import itertools
from odoo import api, fields, models, _
from odoo.exceptions import UserError, RedirectWarning
from odoo.tools import groupby, SQL


class AccountAnalyticAccount(models.Model):
    _name = 'account.analytic.account'
    _inherit = ['mail.thread']
    _description = 'Analytic Account'
    _order = 'plan_id, name asc'
    _check_company_auto = True
    _check_company_domain = models.check_company_domain_parent_of
    _rec_names_search = ['name', 'code']

    name = fields.Char(
        string='Analytic Account',
        index='trigram',
        required=True,
        tracking=True,
        translate=True,
    )
    code = fields.Char(
        string='Reference',
        index='btree',
        tracking=True,
    )
    active = fields.Boolean(
        'Active',
        help="Deactivate the account.",
        default=True,
        tracking=True,
    )
    plan_id = fields.Many2one(
        'account.analytic.plan',
        string='Plan',
        required=True,
    )
    root_plan_id = fields.Many2one(
        'account.analytic.plan',
        string='Root Plan',
        related="plan_id.root_id",
        store=True,
    )
    color = fields.Integer(
        'Color Index',
        related='plan_id.color',
    )

    line_ids = fields.One2many(
        'account.analytic.line',
        'auto_account_id',  # magic link to the right column (plan) by using the context in the view
        string="Analytic Lines",
    )

    company_id = fields.Many2one(
        'res.company',
        string='Company',
        default=lambda self: self.env.company,
    )

    # use auto_join to speed up name_search call
    partner_id = fields.Many2one(
        'res.partner',
        string='Customer',
        auto_join=True,
        tracking=True,
        check_company=True,
    )

    balance = fields.Monetary(
        compute='_compute_debit_credit_balance',
        string='Balance',
    )
    debit = fields.Monetary(
        compute='_compute_debit_credit_balance',
        string='Debit',
    )
    credit = fields.Monetary(
        compute='_compute_debit_credit_balance',
        string='Credit',
    )

    currency_id = fields.Many2one(
        related="company_id.currency_id",
        string="Currency",
    )

    @api.constrains('company_id')
    def _check_company_consistency(self):
        for company, accounts in groupby(self, lambda account: account.company_id):
            if company and self.env['account.analytic.line'].sudo().search_count([
                ('auto_account_id', 'in', [account.id for account in accounts]),
                '!', ('company_id', 'child_of', company.id),
            ], limit=1):
                raise UserError(_("You can't set a different company on your analytic account since there are some analytic items linked to it."))

    @api.depends('code', 'partner_id')
    def _compute_display_name(self):
        for analytic in self:
            name = analytic.name
            if analytic.code:
                name = f'[{analytic.code}] {name}'
            if analytic.partner_id.commercial_partner_id.name:
                name = f'{name} - {analytic.partner_id.commercial_partner_id.name}'
            analytic.display_name = name

    def copy_data(self, default=None):
        default = dict(default or {})
        default.setdefault('name', _("%s (copy)", self.name))
        return super().copy_data(default)

    @api.model
    def _read_group(self, domain, groupby=(), aggregates=(), having=(), offset=0, limit=None, order=None):
        """ Override _read_group to aggregate no-store compute: balance/debit/credit """
        SPECIAL = {'balance:sum', 'debit:sum', 'credit:sum'}
        if SPECIAL.isdisjoint(aggregates):
            return super()._read_group(domain, groupby, aggregates, having, offset, limit, order)

        base_aggregates = [*(agg for agg in aggregates if agg not in SPECIAL), 'id:recordset']
        base_result = super()._read_group(domain, groupby, base_aggregates, having, offset, limit, order)

        # base_result = [(a1, b1, records), (a2, b2, records), ...]
        result = []
        for *other, records in base_result:
            for index, spec in enumerate(itertools.chain(groupby, aggregates)):
                if spec in SPECIAL:
                    field_name = spec.split(':')[0]
                    other.insert(index, sum(records.mapped(field_name)))
            result.append(tuple(other))

        return result

    @api.depends('line_ids.amount')
    def _compute_debit_credit_balance(self):
        def convert(amount, from_currency):
            return from_currency._convert(
                from_amount=amount,
                to_currency=self.env.company.currency_id,
                company=self.env.company,
                date=fields.Date.today(),
            )

        domain = [('company_id', 'in', [False] + self.env.companies.ids)]
        if self._context.get('from_date', False):
            domain.append(('date', '>=', self._context['from_date']))
        if self._context.get('to_date', False):
            domain.append(('date', '<=', self._context['to_date']))

        for plan, accounts in self.grouped('plan_id').items():
            credit_groups = self.env['account.analytic.line']._read_group(
                domain=domain + [(plan._column_name(), 'in', self.ids), ('amount', '>=', 0.0)],
                groupby=[plan._column_name(), 'currency_id'],
                aggregates=['amount:sum'],
            )
            data_credit = defaultdict(float)
            for account, currency, amount_sum in credit_groups:
                data_credit[account.id] += convert(amount_sum, currency)

            debit_groups = self.env['account.analytic.line']._read_group(
                domain=domain + [(plan._column_name(), 'in', self.ids), ('amount', '<', 0.0)],
                groupby=[plan._column_name(), 'currency_id'],
                aggregates=['amount:sum'],
            )
            data_debit = defaultdict(float)
            for account, currency, amount_sum in debit_groups:
                data_debit[account.id] += convert(amount_sum, currency)

            for account in accounts:
                account.debit = -data_debit.get(account.id, 0.0)
                account.credit = data_credit.get(account.id, 0.0)
                account.balance = account.credit - account.debit

    def _update_accounts_in_analytic_lines(self, new_fname, current_fname, accounts):
        if current_fname != new_fname:
            domain = [
                (new_fname, 'not in', accounts.ids + [False]),
                (current_fname, 'in', accounts.ids),
            ]
            if self.env['account.analytic.line'].sudo().search_count(domain, limit=1):
                list_view = self.env.ref('analytic.view_account_analytic_line_tree', raise_if_not_found=False)
                raise RedirectWarning(
                    message=_("Whoa there! Making this change would wipe out your current data. Let's avoid that, shall we?"),
                    action={
                        'res_model': 'account.analytic.line',
                        'type': 'ir.actions.act_window',
                        'domain': domain,
                        'target': 'new',
                        'views': [(list_view and list_view.id, 'list')]
                    },
                    button_text=_("See them"),
                )
            self.env.cr.execute(SQL(
                """
                UPDATE account_analytic_line
                   SET %(new_fname)s = %(current_fname)s,
                       %(current_fname)s = NULL
                 WHERE %(current_fname)s = ANY(%(account_ids)s)
                """,
                new_fname=SQL.identifier(new_fname),
                current_fname=SQL.identifier(current_fname),
                account_ids=accounts.ids,
            ))
            self.env['account.analytic.line'].invalidate_model()

    def write(self, vals):
        if vals.get('plan_id'):
            new_fname = self.env['account.analytic.plan'].browse(vals['plan_id'])._column_name()
            for plan, accounts in self.grouped('plan_id').items():
                current_fname = plan._column_name()
                self._update_accounts_in_analytic_lines(new_fname, current_fname, accounts)
        return super().write(vals)

```

## File: models\analytic_distribution_model.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import SQL
from odoo.exceptions import UserError


class NonMatchingDistribution(Exception):
    pass


class AccountAnalyticDistributionModel(models.Model):
    _name = 'account.analytic.distribution.model'
    _inherit = 'analytic.mixin'
    _description = 'Analytic Distribution Model'
    _rec_name = 'create_date'
    _order = 'id desc'
    _check_company_auto = True
    _check_company_domain = models.check_company_domain_parent_of

    partner_id = fields.Many2one(
        'res.partner',
        string='Partner',
        ondelete='cascade',
        help="Select a partner for which the analytic distribution will be used (e.g. create new customer invoice or Sales order if we select this partner, it will automatically take this as an analytic account)",
    )
    partner_category_id = fields.Many2one(
        'res.partner.category',
        string='Partner Category',
        ondelete='cascade',
        help="Select a partner category for which the analytic distribution will be used (e.g. create new customer invoice or Sales order if we select this partner, it will automatically take this as an analytic account)",
    )
    company_id = fields.Many2one(
        'res.company',
        string='Company',
        default=lambda self: self.env.company,
        ondelete='cascade',
        help="Select a company for which the analytic distribution will be used (e.g. create new customer invoice or Sales order if we select this company, it will automatically take this as an analytic account)",
    )

    @api.constrains('company_id')
    def _check_company_accounts(self):
        """Ensure accounts specific to a company isn't used in any distribution model that wouldn't be specific to the company"""
        query = SQL(
            """
            SELECT model.id
              FROM account_analytic_distribution_model model
              JOIN account_analytic_account account
                ON ARRAY[account.id::text] && %s
             WHERE account.company_id IS NOT NULL AND model.id = ANY(%s)
               AND (model.company_id IS NULL 
                OR model.company_id != account.company_id)
            """,
            self._query_analytic_accounts('model'),
            self.ids,
        )
        self.flush_model(['company_id', 'analytic_distribution'])
        self.env.cr.execute(query)
        if self.env.cr.dictfetchone():
            raise UserError(_('You defined a distribution with analytic account(s) belonging to a specific company but a model shared between companies or with a different company'))

    @api.model
    def _get_distribution(self, vals):
        """ Returns the distribution model that has the most fields that corresponds to the vals given
            This method should be called to prefill analytic distribution field on several models """
        domain = []
        for fname, value in vals.items():
            domain += self._create_domain(fname, value) or []
        best_score = 0
        res = {}
        fnames = set(self._get_fields_to_check())
        for rec in self.search(domain):
            try:
                score = sum(rec._check_score(key, vals.get(key)) for key in fnames)
                if score > best_score:
                    res = rec.analytic_distribution
                    best_score = score
            except NonMatchingDistribution:
                continue
        return res

    def _get_fields_to_check(self):
        return (
            {field.name for field in self._fields.values() if not field.manual}
            - set(self.env['analytic.mixin']._fields)
            - set(models.MAGIC_COLUMNS) - {'display_name', '__last_update'}
        )

    def _check_score(self, key, value):
        self.ensure_one()
        if key == 'company_id':
            if not self.company_id or value == self.company_id.id:
                return 1 if self.company_id else 0.5
            raise NonMatchingDistribution
        if not self[key]:
            return 0
        if value and ((self[key].id in value) if isinstance(value, (list, tuple))
                      else (value.startswith(self[key])) if key.endswith('_prefix')
                      else (value == self[key].id)
                      ):
            return 1
        raise NonMatchingDistribution

    def _create_domain(self, fname, value):
        if not value:
            return False
        if fname == 'partner_category_id':
            value += [False]
            return [(fname, 'in', value)]
        else:
            return [(fname, 'in', [value, False])]

    def action_read_distribution_model(self):
        self.ensure_one()
        return {
            'name': self.display_name,
            'type': 'ir.actions.act_window',
            'view_type': 'form',
            'view_mode': 'form',
            'res_model': 'account.analytic.distribution.model',
            'res_id': self.id,
        }

```

## File: models\analytic_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from lxml.builder import E

from odoo import api, fields, models
from odoo.osv.expression import OR


class AccountAnalyticLine(models.Model):
    _name = 'account.analytic.line'
    _description = 'Analytic Line'
    _order = 'date desc, id desc'
    _check_company_auto = True

    name = fields.Char(
        'Description',
        required=True,
    )
    date = fields.Date(
        'Date',
        required=True,
        index=True,
        default=fields.Date.context_today,
    )
    amount = fields.Monetary(
        'Amount',
        required=True,
        default=0.0,
    )
    unit_amount = fields.Float(
        'Quantity',
        default=0.0,
    )
    product_uom_id = fields.Many2one(
        'uom.uom',
        string='Unit of Measure',
        domain="[('category_id', '=', product_uom_category_id)]",
    )
    product_uom_category_id = fields.Many2one(
        related='product_uom_id.category_id',
        string='UoM Category',
        readonly=True,
    )
    account_id = fields.Many2one(
        'account.analytic.account',
        'Project Account',
        ondelete='restrict',
        index=True,
        check_company=True,
    )
    # Magic column that represents all the plans at the same time, except for the compute
    # where it is context dependent, and needs the id of the desired plan.
    # Used as a syntactic sugar for search views, and magic field for one2many relation
    auto_account_id = fields.Many2one(
        comodel_name='account.analytic.account',
        string='Analytic Account',
        compute='_compute_auto_account',
        inverse='_inverse_auto_account',
        search='_search_auto_account',
    )
    partner_id = fields.Many2one(
        'res.partner',
        string='Partner',
        check_company=True,
    )
    user_id = fields.Many2one(
        'res.users',
        string='User',
        default=lambda self: self.env.context.get('user_id', self.env.user.id),
        index=True,
    )
    company_id = fields.Many2one(
        'res.company',
        string='Company',
        required=True,
        readonly=True,
        default=lambda self: self.env.company,
    )
    currency_id = fields.Many2one(
        related="company_id.currency_id",
        string="Currency",
        readonly=True,
        store=True,
        compute_sudo=True,
    )
    category = fields.Selection(
        [('other', 'Other')],
        default='other',
    )

    @api.depends_context('analytic_plan_id')
    def _compute_auto_account(self):
        plan = self.env['account.analytic.plan'].browse(self.env.context.get('analytic_plan_id'))
        for line in self:
            line.auto_account_id = bool(plan) and line[plan._column_name()]

    def _compute_partner_id(self):
        # TO OVERRIDE
        pass

    def _inverse_auto_account(self):
        for line in self:
            line[line.auto_account_id.plan_id._column_name()] = line.auto_account_id

    def _search_auto_account(self, operator, value):
        project_plan, other_plans = self.env['account.analytic.plan']._get_all_plans()
        return OR([
            [(plan._column_name(), operator, value)]
            for plan in project_plan + other_plans
        ])

    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        if not self._context.get("studio") and self.env['account.analytic.plan'].check_access_rights('read', raise_exception=False):
            project_plan, other_plans = self.env['account.analytic.plan']._get_all_plans()

            # Find main account nodes
            account_node = next(iter(arch.xpath('//field[@name="account_id"]')), None)
            account_filter_node = next(iter(arch.xpath('//filter[@name="account_id"]')), None)

            # Force domain on main account node as the fields_get doesn't do the trick
            if account_node is not None and view_type == 'search':
                account_node.attrib['domain'] = f"[('plan_id', 'child_of', {project_plan.id})]"

            # If there is a main node, append the ones for other plans
            if account_node is not None or account_filter_node is not None:
                for plan in other_plans[::-1]:
                    fname = plan._column_name()
                    if account_node is not None:
                        account_node.addnext(E.field(name=fname, domain=f"[('plan_id', 'child_of', {plan.id})]", optional="show"))
                    if account_filter_node is not None:
                        account_filter_node.addnext(E.filter(name=fname, context=f"{{'group_by': '{fname}'}}"))
        return arch, view

    @api.model
    def fields_get(self, allfields=None, attributes=None):
        fields = super().fields_get(allfields, attributes)
        if not self._context.get("studio") and self.env['account.analytic.plan'].check_access_rights('read', raise_exception=False):
            project_plan, other_plans = self.env['account.analytic.plan']._get_all_plans()
            for plan in project_plan + other_plans:
                fname = plan._column_name()
                if fname in fields:
                    fields[fname]['string'] = plan.name
                    fields[fname]['domain'] = f"[('plan_id', 'child_of', {plan.id})]"
        return fields

```

## File: models\analytic_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api, _
from odoo.tools import SQL, unique
from odoo.tools.float_utils import float_round, float_compare
from odoo.tools.misc import flatten
from odoo.exceptions import UserError, ValidationError

class AnalyticMixin(models.AbstractModel):
    _name = 'analytic.mixin'
    _description = 'Analytic Mixin'

    analytic_distribution = fields.Json(
        'Analytic Distribution',
        compute="_compute_analytic_distribution", store=True, copy=True, readonly=False,
    )
    # Json non stored to be able to search on analytic_distribution.
    analytic_distribution_search = fields.Json(
        store=False,
        search="_search_analytic_distribution"
    )
    analytic_precision = fields.Integer(
        store=False,
        default=lambda self: self.env['decimal.precision'].precision_get("Percentage Analytic"),
    )
    distribution_analytic_account_ids = fields.Many2many(
        comodel_name='account.analytic.account',
        compute='_compute_distribution_analytic_account_ids',
        search='_search_analytic_distribution',
    )

    def init(self):
        # Add a gin index for json search on the keys, on the models that actually have a table
        query = ''' SELECT table_name
                    FROM information_schema.tables
                    WHERE table_name=%s '''
        self.env.cr.execute(query, [self._table])
        if self.env.cr.dictfetchone() and self._fields['analytic_distribution'].store:
            query = fr"""
                CREATE INDEX IF NOT EXISTS {self._table}_analytic_distribution_accounts_gin_index
                                        ON {self._table} USING gin(regexp_split_to_array(jsonb_path_query_array(analytic_distribution, '$.keyvalue()."key"')::text, '\D+'));
            """
            self.env.cr.execute(query)
        super().init()

    @api.model
    def fields_get(self, allfields=None, attributes=None):
        """ Hide analytic_distribution_search from filterable/searchable fields"""
        res = super().fields_get(allfields, attributes)
        if res.get('analytic_distribution_search'):
            res['analytic_distribution_search']['searchable'] = False
        return res

    def _compute_analytic_distribution(self):
        pass

    @api.depends('analytic_distribution')
    def _compute_distribution_analytic_account_ids(self):
        all_ids = {int(_id) for rec in self for key in (rec.analytic_distribution or {}) for _id in key.split(',')}
        existing_accounts_ids = set(self.env['account.analytic.account'].browse(all_ids).exists().ids)
        for rec in self:
            ids = list(unique(int(_id) for key in (rec.analytic_distribution or {}) for _id in key.split(',') if int(_id) in existing_accounts_ids))
            rec.distribution_analytic_account_ids = self.env['account.analytic.account'].browse(ids)

    def _search_analytic_distribution(self, operator, value):
        if operator == 'in' and isinstance(value, (tuple, list)):
            account_ids = value
            operator_inselect = 'inselect'
        elif operator in ('=', '!=', 'ilike', 'not ilike') and isinstance(value, (str, bool)):
            operator_name_search = '=' if operator in ('=', '!=') else 'ilike'
            account_ids = list(self.env['account.analytic.account']._name_search(name=value, operator=operator_name_search))
            operator_inselect = 'inselect' if operator in ('=', 'ilike') else 'not inselect'
        else:
            raise UserError(_('Operation not supported'))

        query = SQL(
            fr"""
            SELECT id
            FROM {self._table}
            WHERE %s && %s
            """,
            [str(account_id) for account_id in account_ids],
            self._query_analytic_accounts(),
        )

        return [('id', operator_inselect, query)]

    def _query_analytic_accounts(self, table=False):
        return SQL(
            r"""regexp_split_to_array(jsonb_path_query_array(%s.analytic_distribution, '$.keyvalue()."key"')::text, '\D+')""",
            SQL(table or self._table),
        )

    @api.model
    def _search(self, domain, offset=0, limit=None, order=None, access_rights_uid=None):
        domain = self._apply_analytic_distribution_domain(domain)
        return super()._search(domain, offset, limit, order, access_rights_uid)

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        domain = self._apply_analytic_distribution_domain(domain)
        return super().read_group(domain, fields, groupby, offset, limit, orderby, lazy)

    def mapped(self, func):
        # Get the related analytic accounts as a recordset instead of the distribution
        if func == 'analytic_distribution' and self.env.context.get('distribution_ids'):
            return self.env['account.analytic.account'].browse(flatten(record._get_analytic_account_ids() for record in self))
        return super().mapped(func)

    def filtered_domain(self, domain):
        # Filter based on the accounts used (i.e. allowing a name_search) instead of the distribution
        # A domain on a binary field doesn't make sense anymore outside of set or not; and it is still doable.
        return super(AnalyticMixin, self.with_context(distribution_ids=True)).filtered_domain(domain)

    def write(self, vals):
        """ Format the analytic_distribution float value, so equality on analytic_distribution can be done """
        decimal_precision = self.env['decimal.precision'].precision_get('Percentage Analytic')
        vals = self._sanitize_values(vals, decimal_precision)
        return super().write(vals)

    @api.model_create_multi
    def create(self, vals_list):
        """ Format the analytic_distribution float value, so equality on analytic_distribution can be done """
        decimal_precision = self.env['decimal.precision'].precision_get('Percentage Analytic')
        vals_list = [self._sanitize_values(vals, decimal_precision) for vals in vals_list]
        return super().create(vals_list)

    def _validate_distribution(self, **kwargs):
        if self.env.context.get('validate_analytic', False):
            mandatory_plans_ids = [plan['id'] for plan in self.env['account.analytic.plan'].sudo().with_company(self.company_id).get_relevant_plans(**kwargs) if plan['applicability'] == 'mandatory']
            if not mandatory_plans_ids:
                return
            decimal_precision = self.env['decimal.precision'].precision_get('Percentage Analytic')
            distribution_by_root_plan = {}
            for analytic_account_ids, percentage in (self.analytic_distribution or {}).items():
                for analytic_account in self.env['account.analytic.account'].browse(map(int, analytic_account_ids.split(","))).exists():
                    root_plan = analytic_account.root_plan_id
                    distribution_by_root_plan[root_plan.id] = distribution_by_root_plan.get(root_plan.id, 0) + percentage

            for plan_id in mandatory_plans_ids:
                if float_compare(distribution_by_root_plan.get(plan_id, 0), 100, precision_digits=decimal_precision) != 0:
                    raise ValidationError(_("One or more lines require a 100% analytic distribution."))

    def _sanitize_values(self, vals, decimal_precision):
        """ Normalize the float of the distribution """
        if 'analytic_distribution' in vals:
            vals['analytic_distribution'] = vals.get('analytic_distribution') and {
                account_id: float_round(distribution, decimal_precision) for account_id, distribution in vals['analytic_distribution'].items()}
        return vals

    def _apply_analytic_distribution_domain(self, domain):
        return [
            ('analytic_distribution_search', leaf[1], leaf[2])
            if len(leaf) == 3 and leaf[0] == 'analytic_distribution' and isinstance(leaf[2], (str, tuple, list))
            else leaf
            for leaf in domain
        ]

    def _get_analytic_account_ids(self) -> list[int]:
        """ Get the analytic account ids from the analytic_distribution dict """
        self.ensure_one()
        return [int(account_id) for ids in (self.analytic_distribution or {}) for account_id in ids.split(',')]

```

## File: models\analytic_plan.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import ormcache, make_index_name, create_index


class AccountAnalyticPlan(models.Model):
    _name = 'account.analytic.plan'
    _description = 'Analytic Plans'
    _parent_store = True
    _rec_name = 'complete_name'
    _order = 'sequence asc, id'

    def _default_color(self):
        return randint(1, 11)

    name = fields.Char(
        required=True,
        translate=True,
        inverse='_inverse_name',
    )
    description = fields.Text(string='Description')
    parent_id = fields.Many2one(
        'account.analytic.plan',
        string="Parent",
        inverse='_inverse_parent_id',
        ondelete='cascade',
        domain="['!', ('id', 'child_of', id)]",
    )
    parent_path = fields.Char(
        index='btree',
        unaccent=False,
    )
    root_id = fields.Many2one(
        'account.analytic.plan',
        compute='_compute_root_id',
    )
    children_ids = fields.One2many(
        'account.analytic.plan',
        'parent_id',
        string="Childrens",
    )
    children_count = fields.Integer(
        'Children Plans Count',
        compute='_compute_children_count',
    )
    complete_name = fields.Char(
        'Complete Name',
        compute='_compute_complete_name',
        recursive=True,
        store=True,
    )
    account_ids = fields.One2many(
        'account.analytic.account',
        'plan_id',
        string="Accounts",
    )
    account_count = fields.Integer(
        'Analytic Accounts Count',
        compute='_compute_analytic_account_count',
    )
    all_account_count = fields.Integer(
        'All Analytic Accounts Count',
        compute='_compute_all_analytic_account_count',
    )
    color = fields.Integer(
        'Color',
        default=_default_color,
    )
    sequence = fields.Integer(default=10)

    default_applicability = fields.Selection(
        selection=[
            ('optional', 'Optional'),
            ('mandatory', 'Mandatory'),
            ('unavailable', 'Unavailable'),
        ],
        string="Default Applicability",
        required=True,
        default='optional',  # actually set in _auto_init because company dependent
        readonly=False,
        company_dependent=True,
    )
    applicability_ids = fields.One2many(
        'account.analytic.applicability',
        'analytic_plan_id',
        string='Applicability',
        domain="[('company_id', '=', current_company_id)]",
    )

    def _auto_init(self):
        super()._auto_init()
        def precommit():
            self.env['ir.property']._set_default(
                name='default_applicability',
                model=self._name,
                value='optional',
            )
        self.env.cr.precommit.add(precommit)

    @ormcache()
    def __get_all_plans(self):
        project_plan = self.browse(int(self.env['ir.config_parameter'].sudo().get_param('analytic.project_plan', 0)))
        if not project_plan:
            raise UserError(_("A 'Project' plan needs to exist and its id needs to be set as `analytic.project_plan` in the system variables"))
        other_plans = self.sudo().search([('parent_id', '=', False)]) - project_plan
        return project_plan.id, other_plans.ids

    def _get_all_plans(self):
        return map(self.browse, self.__get_all_plans())

    def _strict_column_name(self):
        self.ensure_one()
        project_plan, _other_plans = self._get_all_plans()
        return 'account_id' if self == project_plan else f"x_plan{self.id}_id"

    def _column_name(self):
        return self.root_id._strict_column_name()

    def _inverse_name(self):
        self._sync_plan_column()

    def _inverse_parent_id(self):
        self._sync_plan_column()

    @api.depends('parent_id', 'parent_path')
    def _compute_root_id(self):
        for plan in self.sudo():
            plan.root_id = int(plan.parent_path[:-1].split('/')[0]) if plan.parent_path else plan

    @api.depends('name', 'parent_id.complete_name')
    def _compute_complete_name(self):
        for plan in self:
            if plan.parent_id:
                plan.complete_name = '%s / %s' % (plan.parent_id.complete_name, plan.name)
            else:
                plan.complete_name = plan.name

    @api.depends('account_ids')
    def _compute_analytic_account_count(self):
        for plan in self:
            plan.account_count = len(plan.account_ids)

    @api.depends('account_ids', 'children_ids')
    def _compute_all_analytic_account_count(self):
        # Get all children_ids from each plan
        self.env.cr.execute("""
            SELECT parent.id,
                   array_agg(child.id) as children_ids
              FROM account_analytic_plan parent
              JOIN account_analytic_plan child ON child.parent_path LIKE parent.parent_path || '%%'
             WHERE parent.id IN %s
          GROUP BY parent.id
        """, [tuple(self.ids)])
        all_children_ids = dict(self.env.cr.fetchall())

        plans_count = dict(
            self.env['account.analytic.account']._read_group(
                domain=[('plan_id', 'child_of', self.ids)],
                aggregates=['id:count'],
                groupby=['plan_id']
            )
        )
        plans_count = {k.id: v for k, v in plans_count.items()}
        for plan in self:
            plan.all_account_count = sum(plans_count.get(child_id, 0) for child_id in all_children_ids.get(plan.id, []))

    @api.depends('children_ids')
    def _compute_children_count(self):
        for plan in self:
            plan.children_count = len(plan.children_ids)

    @api.onchange('parent_id')
    def _onchange_parent_id(self):
        project_plan, __ = self._get_all_plans()
        if self._origin.id == project_plan.id:
            raise UserError(_("You cannot add a parent to the base plan '%s'", project_plan.name))

    def action_view_analytical_accounts(self):
        result = {
            "type": "ir.actions.act_window",
            "res_model": "account.analytic.account",
            "domain": [('plan_id', "child_of", self.id)],
            "context": {'default_plan_id': self.id},
            "name": _("Analytical Accounts"),
            'view_mode': 'list,form',
        }
        return result

    def action_view_children_plans(self):
        result = {
            "type": "ir.actions.act_window",
            "res_model": "account.analytic.plan",
            "domain": [('parent_id', '=', self.id)],
            "context": {'default_parent_id': self.id,
                        'default_color': self.color},
            "name": _("Analytical Plans"),
            'view_mode': 'list,form',
        }
        return result

    @api.model
    def get_relevant_plans(self, **kwargs):
        """ Returns the list of plans that should be available.
            This list is computed based on the applicabilities of root plans. """
        record_account_ids = kwargs.get('existing_account_ids', [])
        project_plan, other_plans = self.env['account.analytic.plan']._get_all_plans()
        root_plans = (project_plan + other_plans).filtered(lambda p: (
            p.all_account_count > 0
            and not p.parent_id
            and p._get_applicability(**kwargs) != 'unavailable'
        ))
        # If we have accounts that are already selected (before the applicability rules changed or from a model),
        # we want the plans that were unavailable to be shown in the list (and in optional, because the previous
        # percentage could be different from 0)
        forced_plans = self.env['account.analytic.account'].browse(record_account_ids).exists().mapped(
            'root_plan_id') - root_plans
        return [
            {
                "id": plan.id,
                "name": plan.name,
                "color": plan.color,
                "applicability": plan._get_applicability(**kwargs) if plan in root_plans else 'optional',
                "all_account_count": plan.all_account_count
            }
            for plan in (root_plans + forced_plans).sorted('sequence')
        ]

    def _get_applicability(self, **kwargs):
        """ Returns the applicability of the best applicability line or the default applicability """
        self.ensure_one()
        if 'applicability' in kwargs:
            # For models for example, we want all plans to be visible, so we force the applicability
            return kwargs['applicability']
        else:
            score = 0
            applicability = self.default_applicability
            for applicability_rule in self.applicability_ids.filtered(
                    lambda rule:
                    not rule.company_id
                    or not kwargs.get('company_id')
                    or rule.company_id.id == kwargs.get('company_id')
            ):
                score_rule = applicability_rule._get_score(**kwargs)
                if score_rule > score:
                    applicability = applicability_rule.applicability
                    score = score_rule
            return applicability

    def unlink(self):
        # Remove the dynamic field created with the plan (see `_inverse_name`)
        self._find_plan_column().unlink()
        return super().unlink()

    def _find_plan_column(self):
        return self.env['ir.model.fields'].sudo().search([
            ('name', 'in', [plan._strict_column_name() for plan in self]),
            ('model', '=', 'account.analytic.line'),
        ])

    def _sync_plan_column(self):
        # Create/delete a new field/column on analytic lines for this plan, and keep the name in sync.
        for plan in self:
            prev = plan._find_plan_column()
            if plan.parent_id and prev:
                prev.unlink()
            elif prev:
                prev.field_description = plan.name
            elif not plan.parent_id:
                column = plan._strict_column_name()
                field = self.env['ir.model.fields'].with_context(update_custom_fields=True).sudo().create({
                    'name': column,
                    'field_description': plan.name,
                    'state': 'manual',
                    'model': 'account.analytic.line',
                    'model_id': self.env['ir.model']._get_id('account.analytic.line'),
                    'ttype': 'many2one',
                    'relation': 'account.analytic.account',
                    'store': True,
                    'on_delete': 'restrict',
                })
                tablename = self.env['account.analytic.line']._table
                indexname = make_index_name(tablename, column)
                create_index(self.env.cr, indexname, tablename, [column], 'btree', f'{column} IS NOT NULL')
                field.write({
                    'index': True,
                })

    def write(self, vals):
        new_parent = self.env['account.analytic.plan'].browse(vals.get('parent_id'))
        plan2previous_parent = {plan: plan.parent_id for plan in self if plan.parent_id}
        if 'parent_id' in vals and new_parent:
            # Update accounts in analytic lines before _sync_plan_column() unlinks child plan's column
            for plan in self:
                self.env['account.analytic.account']._update_accounts_in_analytic_lines(
                    new_fname=new_parent._column_name(),
                    current_fname=plan._column_name(),
                    accounts=plan.account_ids,
                )

        res = super().write(vals)

        if 'parent_id' in vals and not new_parent:
            # Update accounts in analytic lines after _sync_plan_column() creates the new column
            for plan, previous_parent in plan2previous_parent.items():
                self.env['account.analytic.account']._update_accounts_in_analytic_lines(
                    new_fname=plan._column_name(),
                    current_fname=previous_parent._column_name(),
                    accounts=plan.account_ids,
                )
        return res


class AccountAnalyticApplicability(models.Model):
    _name = 'account.analytic.applicability'
    _description = "Analytic Plan's Applicabilities"
    _check_company_auto = True
    _check_company_domain = models.check_company_domain_parent_of

    analytic_plan_id = fields.Many2one('account.analytic.plan')
    business_domain = fields.Selection(
        selection=[
            ('general', 'Miscellaneous'),
        ],
        required=True,
        string='Domain',
    )
    applicability = fields.Selection([
        ('optional', 'Optional'),
        ('mandatory', 'Mandatory'),
        ('unavailable', 'Unavailable'),
    ],
        required=True,
        string="Applicability",
    )
    company_id = fields.Many2one(
        'res.company',
        string='Company',
        default=lambda self: self.env.company,
    )

    def _get_score(self, **kwargs):
        """ Gives the score of an applicability with the parameters of kwargs """
        self.ensure_one()
        # 0.5 is because company is less important than other fields for an equal number of valid fields
        # No company on the applicability and the kwargs together are not considered a more fitting rule
        score = 0.5 if self.company_id and kwargs.get('company_id') else 0
        if not kwargs.get('business_domain'):
            return score
        else:
            return score + 1 if kwargs.get('business_domain') == self.business_domain else -1

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_analytic_accounting = fields.Boolean(string='Analytic Accounting', implied_group='analytic.group_analytic_accounting')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import analytic_plan
from . import analytic_account
from . import analytic_line
from . import analytic_mixin
from . import analytic_distribution_model
from . import res_config_settings

```

## File: populate\analytic_account.py

```python
from odoo import models
from odoo.tools import populate


class AnalyticAccount(models.Model):
    _inherit = "account.analytic.account"
    _populate_sizes = {
        'small': 100,
        'medium': 1_000,
        'large': 10_000,
    }

    def _populate_factories(self):
        project_plan = self._search_or_create_plan('Projects')
        department_plan = self._search_or_create_plan('Departments')
        return [
            ('company_id', populate.constant(False)),
            ('plan_id', populate.cartesian(
                [project_plan.id, department_plan.id],
                [0.99, 0.01],
            )),
            ('name', populate.constant("Account {counter}")),
        ]

    def _search_or_create_plan(self, name):
        return self.env['account.analytic.plan'].search([
            ('name', '=', name),
        ]) or self.env['account.analytic.plan'].create({
            'name': name,
        })

```

## File: populate\analytic_line.py

```python
import logging

from odoo import models
from odoo.tools import populate
_logger = logging.getLogger(__name__)


class AnalyticLine(models.Model):
    _inherit = "account.analytic.line"
    _populate_sizes = {
        'small': 100,
        'medium': 1_000,
        'large': 10_000_000,
    }

    _populate_dependencies = ['account.analytic.account']

    def _populate_factories(self):
        accounts = self.env['account.analytic.account'].browse(self.env.registry.populated_models['account.analytic.account'])
        grouped_account = accounts.grouped('plan_id')
        project_plan, other_plans = self.env['account.analytic.plan']._get_all_plans()
        return [
            ('amount', populate.randfloat(0, 1000)),
            *[(
                plan._column_name(),
                populate.randomize(grouped_account.get(plan, self.env['account.analytic.account'].browse([False]))._ids)
            ) for plan in project_plan + other_plans],
            ('name', populate.constant("Line {counter}")),
        ]

```

## File: populate\__init__.py

```python
from . import analytic_account
from . import analytic_line

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
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'parent_of', company_ids)]</field>
    </record>

    <record id="analytic_line_comp_rule" model="ir.rule">
        <field name="name">Analytic line multi company rule</field>
        <field name="model_id" ref="model_account_analytic_line"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record id="analytic_applicability_comp_rule" model="ir.rule">
        <field name="name">Analytic applicability multi company rule</field>
        <field name="model_id" ref="model_account_analytic_applicability"/>
        <field eval="True" name="global"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'parent_of', company_ids)]</field>
    </record>

    <record id="analytic_distribution_model_comp_rule" model="ir.rule">
        <field name="name">Analytic distribution model multi company rule</field>
        <field name="model_id" ref="model_account_analytic_distribution_model"/>
        <field eval="True" name="global"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'parent_of', company_ids)]</field>
    </record>
</data>
<data noupdate="0">

    <record id="group_analytic_accounting" model="res.groups">
        <field name="name">Analytic Accounting</field>
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
access_account_analytic_plan,access_account_analytic_plan,model_account_analytic_plan,group_analytic_accounting,1,1,1,1
access_account_analytic_applicability,access_account_analytic_applicability,model_account_analytic_applicability,group_analytic_accounting,1,1,1,1
access_account_analytic_distribution_model,access_account_analytic_distribution_model,model_account_analytic_distribution_model,group_analytic_accounting,1,1,1,1

```

## File: static\src\components\analytic_distribution\analytic_distribution.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { evaluateExpr } from "@web/core/py_js/py";
import { getNextTabableElement, getPreviousTabableElement } from "@web/core/utils/ui";
import { usePosition } from "@web/core/position_hook";
import { getActiveHotkey } from "@web/core/hotkeys/hotkey_service";
import { shallowEqual } from "@web/core/utils/arrays";
import { roundDecimals } from "@web/core/utils/numbers";
import { isMobileOS } from "@web/core/browser/feature_detection";
import { _t } from "@web/core/l10n/translation";
import { useRecordObserver } from "@web/model/relational_model/utils";

import { standardFieldProps } from "@web/views/fields/standard_field_props";
import { TagsList } from "@web/core/tags_list/tags_list";
import { useOpenMany2XRecord } from "@web/views/fields/relational_utils";
import { formatPercentage } from "@web/views/fields/formatters";

import { Record } from "@web/model/record";
import { Field } from "@web/views/fields/field";
import {
    Component,
    useState,
    useRef,
    useExternalListener,
    onWillStart,
    onPatched,
} from "@odoo/owl";

export class AnalyticDistribution extends Component {
    static template = "analytic.AnalyticDistribution";
    static components = {
        TagsList,
        Record,
        Field,
    }

    static props = {
        ...standardFieldProps,
        business_domain: { type: String, optional: true },
        account_field: { type: String, optional: true },
        product_field: { type: String, optional: true },
        amount_field: { type: String, optional: true },
        business_domain_compute: { type: String, optional: true },
        force_applicability: { type: String, optional: true },
        allow_save: { type: Boolean, optional: true },
    }

    setup(){
        this.orm = useService("orm");
        this.batchedOrm = useService("batchedOrm");

        this.state = useState({
            showDropdown: false,
            formattedData: [],
        });

        this.widgetRef = useRef("analyticDistribution");
        this.dropdownRef = useRef("analyticDropdown");
        this.mainRef = useRef("mainElement");
        this.addLineButton = useRef("addLineButton");
        usePosition("analyticDropdown", () => this.widgetRef.el);

        this.nextId = 1;
        this.focusSelector = false;

        this.currentValue = this.props.record.data[this.props.name];

        onWillStart(this.willStart);
        useRecordObserver(this.willUpdateRecord.bind(this));
        onPatched(this.patched);

        useExternalListener(window, "click", this.onWindowClick, true);
        useExternalListener(window, "resize", this.onWindowResized);

        this.openTemplate = useOpenMany2XRecord({
            resModel: "account.analytic.distribution.model",
            activeActions: {
                create: true,
                edit: false,
                write: true,
            },
            isToMany: false,
            onRecordSaved: async (record) => {
                if (!this.props.record.model.multiEdit) {
                    this.mainRef.el.focus();
                }
            },
            onClose: () => {
                if (!this.props.record.model.multiEdit) {
                    this.mainRef.el.focus();
                }
            },
            fieldString: _t("Analytic Distribution Model"),
        });
        this.allPlans = [];
        this.lastAccount = this.props.account_field && this.props.record.data[this.props.account_field] || false;
        this.lastProduct = this.props.product_field && this.props.record.data[this.props.product_field] || false;
    }

    // Lifecycle
    async willStart() {
        if (this.editingRecord) {
            // for performance in list views, plans are not retrieved until they are required.
            await this.fetchAllPlans(this.props);
        }
        await this.jsonToData(this.props.record.data[this.props.name]);
    }

    async willUpdateRecord(record) {
        // Unless force_applicability, Plans need to be retrieved again as the product or account might have changed
        // and thus different applicabilities apply
        // or a model applies that contains unavailable plans
        // This should only execute when these fields have changed, therefore we use the `_field` props.
        const valueChanged =
            JSON.stringify(this.currentValue) !==
            JSON.stringify(record.data[this.props.name]);
        const currentAccount = this.props.account_field && record.data[this.props.account_field] || false;
        const currentProduct = this.props.product_field && record.data[this.props.product_field] || false;
        const accountChanged = !shallowEqual(this.lastAccount, currentAccount);
        const productChanged = !shallowEqual(this.lastProduct, currentProduct);
        if (valueChanged || accountChanged || productChanged) {
            if (!this.props.force_applicability) {
                await this.fetchAllPlans({ record });
            }
            this.lastAccount = accountChanged && currentAccount || this.lastAccount;
            this.lastProduct = productChanged && currentProduct || this.lastProduct;
            await this.jsonToData(record.data[this.props.name]);
        }
        this.currentValue = record.data[this.props.name];
    }

    patched() {
        this.focusToSelector();
    }

    /**
     * Computes the totals for each account, grouped by plan (primarily used in tags)
     * @returns {Object}
     */
    accountTotalsByPlan() {
        const accountTotals = {};
        this.state.formattedData.map((line) => {
            line.analyticAccounts.map((column) => {
                if (column.accountId) {
                    let {
                        accId = column.accountId,
                        accName = column.accountDisplayName,
                        total = 0.0,
                        planId = column.accountRootPlanId,
                        planColor = column.accountColor,
                    } = accountTotals[column.accountRootPlanId]?.[column.accountId] || {};

                    total += roundDecimals(line.percentage, this.decimalPrecision.digits[1] + 2);

                    accountTotals[planId] = accountTotals[planId] || {};
                    accountTotals[planId][accId] = { accId, accName, planId, total, planColor};
                }
            })
        });
        return accountTotals;
    }

    /**
     * Computes the totals for each plan (used in the table headers)
     * @returns {Object}
     */
    planTotals() {
        const summary = this.accountTotalsByPlan();
        this.allPlans.map((plan) => {
            const planTotal = (summary[plan.id] && Object.values(summary[plan.id]) || []).reduce((prev, next) => prev + next.total, 0.0);
            const className = plan.applicability === "mandatory" && !this.planIsComplete(planTotal) ? 'text-danger' : plan.applicability === "mandatory" ? 'text-success' : '';
            summary[plan.id] = {
                value: planTotal,
                formattedValue: formatPercentage(planTotal, this.decimalPrecision),
                class: className,
                applicability: plan.applicability,
            }
        });
        return summary;
    }

    planIsComplete(total) {
        return roundDecimals(total, this.decimalPrecision.digits[1] + 2) === 1;
    }

    /**
     * Converts the account Totals to a list of tags
     * PlanA  PlanB  PlanC  Percentage
     * A1                   100
     *        B1            80.123     => ["A1", "80.12% B1", "C1"]
     *               C1     100
     *
     * PlanA  PlanB  PlanC  Percentage
     * A1     B1     C1     50
     * A2     B1     C1     50         => ["50% A1 | 50% A2 | 50% A3", "150% B1", "C1 | 50% C2"]
     * A3     B1     C2     50
     * @returns [List] of tag objects
     */
    planSummaryTags() {
        const accountTotals = this.accountTotalsByPlan();
        return Object.values(accountTotals).map((planSummary) => {
            const accs = Object.values(planSummary);
            return {
                id: accs[0].planId,
                text: accs.reduce((p, n) => p + (p.length ? " | " : "") + (this.planIsComplete(n.total) ? n.accName : `${formatPercentage(n.total)} ${n.accName}`) , ""),
                colorIndex: accs[0].planColor,
                onClick: (ev) => this.tagClicked(ev),
            };
        });
    }

    plansToArray() {
        return this.allPlans.map((plan) => ({
            planId: plan.id,
            planName: plan.name,
            planColor: plan.color,
        }));
    }

    async jsonToData(jsonFieldValue) {
        const analyticAccountIds = jsonFieldValue ? Object.keys(jsonFieldValue).map((key) => key.split(',')).flat().map((id) => parseInt(id)) : [];
        const analyticAccountDict = analyticAccountIds.length ? await this.fetchAnalyticAccounts([["id", "in", analyticAccountIds]]) : [];

        let distribution = [];
        let accountNotFound = false;

        for (const [accountIds, percentage] of Object.entries(jsonFieldValue)) {
            const defaultVals = this.plansToArray(); // empty if the popup was not opened
            const ids = accountIds.split(',');

            for (const id of ids) {
                const account = analyticAccountDict[parseInt(id)];
                if (account) {
                    // since tags are displayed even though plans might not be retrieved (ie defaultVals is empty)
                    // push the accounts anyway, as order doesn't matter
                    // once the popup is opened, plans are fetched and the analyticAccounts list will be ordered
                    Object.assign(defaultVals.find((plan) => plan.planId == account.root_plan_id[0]) || defaultVals.push({}) && defaultVals[defaultVals.length-1],
                    {
                        accountId: parseInt(id),
                        accountDisplayName: account.display_name,
                        accountColor: account.color,
                        accountRootPlanId: account.root_plan_id[0],
                    });
                } else {
                    accountNotFound = true;
                }
            }
            distribution.push({
                analyticAccounts: defaultVals,
                percentage: percentage / 100,
                id: this.nextId++,
            })
        }
        this.state.formattedData = distribution;
        if (accountNotFound) {
            // Analytic accounts in the json were not found, save the json without them
            await this.save();
        }
    }

    recordProps(line) {
        const analyticAccountFields = {
            id: { type: "int" },
            display_name: { type: "char" },
            color: { type: "int" },
            plan_id: { type: "many2one" },
            root_plan_id: { type: "many2one" },
        };
        let recordFields = {};
        const values = {};
        // Analytic Account fields
        line.analyticAccounts.map((account) => {
            const fieldName = `x_plan${account.planId}_id`;
            recordFields[fieldName] = {
                string: account.planName,
                relation: "account.analytic.account",
                type: "many2one",
                related: {
                    fields: analyticAccountFields,
                    activeFields: analyticAccountFields,
                },
                // company domain might be required here
                domain: [["root_plan_id", "=", account.planId]],
            };
            values[fieldName] =  account?.accountId || false;
        });
        // Percentage field
        recordFields['percentage'] = {
            string: _t("Percentage"),
            type: "percentage",
            cellClass: "numeric_column_width",
            ...this.decimalPrecision,
        };
        values['percentage'] = line.percentage;
        // Value field copied from original
        if (this.props.amount_field) {
            const { string, name, type, currency_field } = this.props.record.fields[this.props.amount_field];
            recordFields[name] = { string, name, type, currency_field, cellClass: "numeric_column_width" };
            values[name] = this.props.record.data[name] * values['percentage'];
            // Currency field
            if (currency_field) {
                // TODO: check web_read network request
                const { string, name, type, relation } = this.props.record.fields[currency_field];
                recordFields[currency_field] = { name, string, type, relation, invisible: true };
                values[currency_field] = this.props.record.data[currency_field][0];
            }
        }
        return {
            fields: recordFields,
            values: values,
            activeFields: recordFields,
            onRecordChanged: async (record, changes) => await this.lineChanged(record, changes, line),
        }
    }

    accountCount(line) {
        return line.analyticAccounts.map((acc) => acc.accountId).filter(Boolean).length;
    }

    lineIsValid(line) {
        return this.accountCount(line) && line.percentage;
    }

    // ORM
    fetchPlansArgs({ record }) {
        let args = {};
        if (this.props.business_domain_compute) {
            args['business_domain'] = evaluateExpr(this.props.business_domain_compute, record.evalContext);
        }
        if (this.props.business_domain) {
            args['business_domain'] = this.props.business_domain;
        }
        if (this.props.product_field && record.data[this.props.product_field]) {
            args['product'] = record.data[this.props.product_field][0];
        }
        if (this.props.account_field && record.data[this.props.account_field]) {
            args['account'] = record.data[this.props.account_field][0];
        }
        if (this.props.force_applicability) {
            args['applicability'] = this.props.force_applicability;
        }
        const existing_account_ids = Object.keys(record.data[this.props.name]).map((k) => k.split(",")).flat().map((i) => parseInt(i));
        if (existing_account_ids.length) {
            args['existing_account_ids'] = existing_account_ids;
        }
        if (record.data.company_id) {
            args['company_id'] = record.data.company_id[0];
        }
        return args;
    }

    async fetchAllPlans(props) {
        const argsPlan = this.fetchPlansArgs(props);
        this.allPlans = await this.orm.call("account.analytic.plan", "get_relevant_plans", [], argsPlan);
    }

    async fetchAnalyticAccounts(domain) {
        const args = {
            domain: domain,
            fields: ["id", "display_name", "root_plan_id", "color"],
            context: [],
        }
        // batched call
        const records = await this.batchedOrm.read("account.analytic.account", domain[0][2], args.fields, {});
        return Object.assign({}, ...records.map((r) => {
            const {id, ...rest} = r;
            return {[id]: rest};
        }));
    }

    // Editing Distributions
    async lineChanged(record, changes, line) {
        // record analytic account changes to the state
        for (const account of line.analyticAccounts) {
            const selected = record.data[`x_plan${account.planId}_id`];
            account.accountId = selected[0];
            account.accountDisplayName = selected[1];
            account.accountColor = account.planColor;
            account.accountRootPlanId = account.planId;
        }
        // record percentage or value changes
        if (changes.percentage != line.percentage) {
            roundDecimals(line.percentage = record.data.percentage, this.decimalPrecision.digits[1] + 2);
        } else if (
            this.valueColumnEnabled &&
            changes[this.props.amount_field] != line[this.props.amount_field]
        ) {
            line.percentage = roundDecimals(
                record.data[this.props.amount_field] / this.props.record.data[this.props.amount_field],
                this.decimalPrecision.digits[1] + 2);
        }
    }

    // Getters
    get valueColumnEnabled() {
        return Boolean(this.props.amount_field && this.props.record.data[this.props.amount_field]);
    }

    get decimalPrecision() {
        return { digits: [12, this.props.record.data.analytic_precision || 2] };
    }

    get allowSave() {
        return this.props.allow_save && this.state.formattedData.some((line) => this.lineIsValid(line));
    }

    get editingRecord() {
        return !this.props.readonly;
    }

    get isDropdownOpen() {
        return this.state.showDropdown && !!this.dropdownRef.el;
    }

    // actions
    addLine() {
        let maxMandatory = 0, maxOptional = 0, hasMandatory = false;

        Object.values(this.planTotals()).filter((plan) => plan.value < 1).map((plan) => {
            if (plan.applicability == "mandatory"){
                maxMandatory = Math.max(plan.value, maxMandatory);
                hasMandatory = true;
            } else {
                maxOptional = Math.max(plan.value, maxOptional);
            }
        });
        let noPlanTotal = this.state.formattedData.filter((line) => !this.accountCount(line)).reduce((p, n) => p + n.percentage, 0);
        const remainder = roundDecimals(1 - (hasMandatory ? maxMandatory : (maxOptional || noPlanTotal)), this.decimalPrecision.digits[1] + 2);
        const lineToAdd = {
            id: this.nextId++,
            analyticAccounts: this.plansToArray(),
            percentage: Math.max(remainder, 0) || 1,
        }
        this.state.formattedData.push(lineToAdd);
        this.setFocusSelector(`[name=line_${this.state.formattedData.length - 1}] td:first-of-type`);
    }

    deleteLine(index) {
        this.state.formattedData.splice(index, 1);
        if (!this.state.formattedData.length) {
            this.addLine();
        }
    }

    dataToJson() {
        const result = {};
        this.state.formattedData = this.state.formattedData.filter((line) => this.accountCount(line));
        this.state.formattedData.map((line) => {
            const key = line.analyticAccounts.reduce((p, n) => p.concat(n.accountId ? n.accountId : []), []);
            result[key] = (result[key] || 0) + line.percentage * 100;
        });
        return result;
    }

    async save() {
        await this.props.record.update({ [this.props.name]: this.dataToJson() });
    }

    onSaveNew() {
        this.closeAnalyticEditor();
        const { record, product_field, account_field } = this.props;
        this.openTemplate({ resId: false, context: {
            'default_analytic_distribution': this.dataToJson(),
            'default_partner_id': record.data['partner_id'] ? record.data['partner_id'][0] : undefined,
            'default_product_id': product_field ? record.data[product_field][0] : undefined,
            'default_account_prefix': account_field ? record.data[account_field][1].substr(0, 3) : undefined,
        }});
    }

    forceCloseEditor() {
        // focus to the main Element but the dropdown should not open
        this.preventOpen = true;
        this.closeAnalyticEditor();
        this.mainRef.el.focus();
        this.preventOpen = false;
    }

    closeAnalyticEditor() {
        this.save();
        this.state.showDropdown = false;
    }

    async openAnalyticEditor() {
        if (!this.allPlans.length) {
            await this.fetchAllPlans(this.props);
            await this.jsonToData(this.props.record.data[this.props.name]);
        }
        if (!this.state.formattedData.length) {
            await this.addLine();
        }
        this.setFocusSelector("[name='line_0'] td:first-of-type");
        this.state.showDropdown = true;
    }

    async tagClicked(ev) {
        if (this.editingRecord && !this.isDropdownOpen) {
            // TODO: focus is not working when tag is clicked while on an editable line
            await this.openAnalyticEditor();
        }
        if (this.isDropdownOpen) {
            this.setFocusSelector("[name='line_0'] td:first-of-type");
            this.focusToSelector();
            ev.stopPropagation();
        }
    }

    // Focus
    onMainElementFocus(ev) {
        if (!this.isDropdownOpen && !this.preventOpen) {
            this.openAnalyticEditor();
        }
    }

    focusToSelector() {
        if (this.focusSelector && this.isDropdownOpen) {
            this.focus(this.adjacentElementToFocus("next", this.dropdownRef.el.querySelector(this.focusSelector)));
        }
        this.focusSelector = false;
    }

    setFocusSelector(selector) {
        this.focusSelector = selector;
    }

    adjacentElementToFocus(direction, el = null) {
        if (!this.isDropdownOpen) {
            return null;
        }
        if (!el) {
            el = this.dropdownRef.el;
        }
        return direction == "next" ? getNextTabableElement(el) : getPreviousTabableElement(el);
    }

    focusAdjacent(direction) {
        const elementToFocus = this.adjacentElementToFocus(direction);
        if (elementToFocus){
            this.focus(elementToFocus);
            return true;
        }
        return false;
    }

    focus(el) {
        if (!el) return;
        el.focus();
        if (["INPUT", "TEXTAREA"].includes(el.tagName)) {
            if (el.selectionStart) {
                el.selectionStart = 0;
                el.selectionEnd = el.value.length;
            }
            el.select();
        }
    }

    // Keys and Clicks
    async onWidgetKeydown(ev) {
        if (!this.editingRecord) {
            return;
        }
        const hotkey = getActiveHotkey(ev);
        switch (hotkey) {
            case "enter":
            case "tab": {
                if (this.isDropdownOpen) {
                    const closestCell = ev.target.closest("td, th");
                    const row = closestCell.parentElement;
                    const line = this.state.formattedData[parseInt(row.id)];
                    if (this.adjacentElementToFocus("next") == this.addLineButton.el && line && this.lineIsValid(line)) {
                        this.addLine();
                        break;
                    }
                    this.focusAdjacent("next") || this.forceCloseEditor();
                    break;
                };
                return;
            }
            case "shift+tab": {
                if (this.isDropdownOpen) {
                    this.focusAdjacent("previous") || this.forceCloseEditor();
                    break;
                };
                return;
            }
            case "escape": {
                if (this.isDropdownOpen) {
                    this.forceCloseEditor();
                    break;
                }
            }
            case "arrowdown": {
                if (!this.isDropdownOpen) {
                    this.onMainElementFocus();
                    break;
                }
                return;
            }
            default: {
                return;
            }
        }
        ev.preventDefault();
        ev.stopPropagation();
    }

    onWindowClick(ev) {
        /*
        Dropdown should be closed only if all these condition are true:
            - dropdown is open
            - click is outside widget element (widgetRef)
            - there is no active modal containing a list/kanban view (search more modal)
            - there is no popover (click is not in search modal's search bar menu)
            - click is not targeting document dom element (drag and drop search more modal)
        */

        const selectors = [
            ".o_popover",
            ".modal:not(.o_inactive_modal):not(:has(.o_act_window))",
        ];
        if (this.isDropdownOpen
            && !this.widgetRef.el.contains(ev.target)
            && !ev.target.closest(selectors.join(","))
            && !ev.target.isSameNode(document.documentElement)
           ) {
            this.forceCloseEditor();
        }
    }

    onWindowResized() {
        // popup ui is ugly when window is resized, so close it
        if (this.isDropdownOpen && !isMobileOS()) {
            this.forceCloseEditor();
        }
    }
}

export const analyticDistribution = {
    component: AnalyticDistribution,
    supportedTypes: ["char", "text"],
    fieldDependencies: [{ name:"analytic_precision", type: "integer" }],
    supportedOptions: [
        {
            label: _t("Disable save"),
            name: "disable_save",
            type: "boolean",
        },
        {
            label: _t("Force applicability"),
            name: "force_applicability",
            type: "boolean",
        },
        {
            label: _t("Business domain"),
            name: "business_domain",
            type: "string",
        },
        {
            label: _t("Product field"),
            name: "product_field",
            type: "field",
            availableTypes: ["many2one"],
        },
        {
            label: _t("Amount field"),
            name: "amount_field",
            type: "field",
            availableTypes: ["monetary"],
        },
        {
            label: _t("Account field"),
            name: "account_field",
            type: "field",
            availableTypes: ["many2one"],
        }
    ],
    extractProps: ({ attrs, options }) => ({
        business_domain: options.business_domain,
        account_field: options.account_field,
        product_field: options.product_field,
        amount_field: options.amount_field,
        business_domain_compute: attrs.business_domain_compute,
        force_applicability: options.force_applicability,
        allow_save: !options.disable_save,
    }),
};

registry.category("fields").add("analytic_distribution", analyticDistribution);

```

## File: static\src\components\analytic_distribution\analytic_distribution.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="analytic.AnalyticDistribution">
        <div class="o_field_tags d-inline-flex flex-wrap mw-100" t-att-class="{'o_tags_input o_input': !props.readonly}" t-ref="analyticDistribution" t-on-keydown="onWidgetKeydown">
            <TagsList tags="planSummaryTags()"/>
            <div t-if="!props.readonly" class="o_input_dropdown d-inline-flex w-100" tabindex="0" t-ref="mainElement" t-on-focus="onMainElementFocus" t-on-click="onMainElementFocus">
                <span class="analytic_distribution_placeholder"/>
                <span class="o_dropdown_button" />
                <t t-call="analytic.AnalyticDistributionPopup"/>
            </div>
        </div>
    </t>

    <t t-name="analytic.AnalyticDistributionPopup">
        <div class="analytic_distribution_popup dropdown-menu o-dropdown--menu show rounded py-0 overflow-x-hidden" t-if="state.showDropdown" t-ref="analyticDropdown">
            <div class="popover-header sticky-top">
                <div class="d-flex">
                    <div class="h5 mt-2 me-auto">
                        Analytic
                        <span t-if="allowSave" class="btn btn-link" t-on-click="onSaveNew" title="Save as new analytic distribution model">New Model</span>
                    </div>
                    <div class="popupButtons">
                        <span class="btn o_button" t-on-click.stop="() => this.closeAnalyticEditor()" title="Close"><span class="fa fa-close"/></span>
                    </div>
                </div>
            </div>
            <div class="p-2 table-responsive">
                <span t-if="!allPlans.length">No analytic plans found</span>
                <table t-else="" class="o_list_table table table-sm table-hover o_analytic_table mb-2 table-striped">
                    <t t-set="totals" t-value="planTotals()"/>
                    <thead>
                        <tr class="border-bottom">
                            <th t-foreach="allPlans" t-as="plan" t-key="plan.id">
                                <span t-out="plan.name"/> (<span t-att-class="totals[plan.id].class" t-out="totals[plan.id].formattedValue"/>)
                            </th>
                            <th t-out="'Percentage'" class="numeric_column_width"/>
                            <th t-if="valueColumnEnabled" class="numeric_column_width" t-out="props.record.fields[props.amount_field].string"/>
                            <th class="deleteColumn w-20px"/>
                        </tr>
                    </thead>
                    <tbody>
                        <tr t-foreach="state.formattedData" t-as="line" t-key="line.id" t-att-id="line_index" t-att-name="'line_' + line_index">
                            <Record t-props="recordProps(line)" t-slot-scope="data">
                                <td t-foreach="Object.keys(data.record.fields).filter((f) => f.startsWith('x_plan'))" t-as="field" t-key="field">
                                    <Field id="field" name="field" record="data.record" domain="data.record.fields[field].domain" canOpen="false" canCreate="false" canCreateEdit="false" canQuickCreate="false"/>
                                </td>
                                <td class="numeric_column_width">
                                    <Field id="'percentage'" name="'percentage'" record="data.record"/>
                                </td>
                                <td t-if="valueColumnEnabled" class="numeric_column_width">
                                    <Field id="props.amount_field" name="props.amount_field" record="data.record"/>
                                </td>
                                <td class="w-20px">
                                    <span class="fa fa-trash-o cursor-pointer" t-on-click.stop="() => this.deleteLine(line_index)"/>
                                </td>
                            </Record>
                        </tr>
                        <tr>
                            <td t-on-click.stop.prevent="addLine" class="o_field_x2many_list_row_add" t-att-colspan="allPlans.length + 2 + valueColumnEnabled">
                                <a href="#" t-ref="addLineButton" tabindex="0">Add a Line</a>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\services\batched_orm_service.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { ORM } from "@web/core/orm_service";
import { unique } from "@web/core/utils/arrays";
import { Deferred } from "@web/core/utils/concurrency";

class RequestBatcherORM extends ORM {
    constructor() {
        super(...arguments);
        this.searchReadBatches = {};
        this.searchReadBatchId = 1;
        this.batches = {};
    }

    /**
     * @param {number[]} ids
     * @param {any[]} keys
     * @param {Function} callback
     * @returns {Promise<any>}
     */
    async batch(ids, keys, callback) {
        const key = JSON.stringify(keys);
        let batch = this.batches[key];
        if (!batch) {
            batch = {
                deferred: new Deferred(),
                scheduled: false,
                ids: [],
            };
            this.batches[key] = batch;
        }
        batch.ids = unique([...batch.ids, ...ids]);

        if (!batch.scheduled) {
            batch.scheduled = true;
            Promise.resolve().then(async () => {
                delete this.batches[key];
                let result;
                try {
                    result = await callback(batch.ids);
                } catch (e) {
                    return batch.deferred.reject(e);
                }
                batch.deferred.resolve(result);
            });
        }

        return batch.deferred;
    }

    /**
     * Entry point to batch "read" calls. If the `fields` and `resModel`
     * arguments have already been called, the given ids are added to the
     * previous list of ids to perform a single read call. Once the server
     * responds, records are then dispatched to the callees based on the
     * given ids arguments (kept in the closure).
     *
     * @param {string} resModel
     * @param {number[]} resIds
     * @param {string[]} fields
     * @returns {Promise<Object[]>}
     */
    async read(resModel, resIds, fields, kwargs) {
        const records = await this.batch(resIds, ["read", resModel, fields, kwargs], (resIds) =>
            super.read(resModel, resIds, fields, kwargs)
        );
        return records.filter((r) => resIds.includes(r.id));
    }
}

export const batchedOrmService = {
    dependencies: ["rpc", "user"],
    async: [
        "call",
        "create",
        "nameGet",
        "read",
        "readGroup",
        "search",
        "searchRead",
        "unlink",
        "webSearchRead",
        "write",
    ],
    start(env, { rpc, user }) {
        return new RequestBatcherORM(rpc, user);
    },
};

registry.category("services").add("batchedOrm", batchedOrmService);

```

## File: views\analytic_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_account_analytic_account_form" model="ir.ui.view">
            <field name="name">analytic.analytic.account.form</field>
            <field name="model">account.analytic.account</field>
            <field name="arch" type="xml">
                <form string="Analytic Account">
                    <field name="company_id" invisible="1"/>
                    <sheet string="Analytic Account">
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" type="action" name="%(account_analytic_line_action)d" icon="fa-usd">
                                <div class="o_form_field o_stat_info">
                                    <span class="o_stat_text">Gross Margin</span>
                                    <span class="o_stat_value">
                                        <field name="balance" widget='monetary'/>
                                    </span>
                                </div>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <div class="oe_title">
                            <label for="name"/>
                            <h1>
                                <field name="name" class="oe_inline" placeholder="e.g. Project XYZ"/>
                            </h1>
                        </div>
                        <div name="project"/>
                        <group name="main">
                            <group>
                                <field name="active" invisible="1"/>
                                <field name="partner_id"/>
                                <field name="code"/>
                            </group>
                            <group>
                                <field name="plan_id" options="{'no_quick_create': True}"/>
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
                <tree string="Analytic Accounts" multi_edit="1">
                    <field name="company_id" column_invisible="True"/>
                    <field name="currency_id" column_invisible="True"/>
                    <field name="name" string="Name"/>
                    <field name="code"/>
                    <field name="partner_id"/>
                    <field name="plan_id"/>
                    <field name="active" column_invisible="True"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="debit" sum="Debit" column_invisible="True"/>
                    <field name="credit" sum="Credit" column_invisible="True"/>
                    <field name="balance" sum="Balance"/>
                </tree>
            </field>
        </record>

        <record id="view_account_analytic_account_list_select" model="ir.ui.view">
            <field name="name">account.analytic.account.list.select</field>
            <field name="model">account.analytic.account</field>
            <field name="mode">primary</field>
            <field eval="18" name="priority"/>
            <field name="inherit_id" ref="analytic.view_account_analytic_account_list"/>
            <field name="arch" type="xml">
                <tree position="attributes">
                    <attribute name="multi_edit">0</attribute>
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
</odoo>

```

## File: views\analytic_distribution_model_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_analytic_distribution_model_tree_view" model="ir.ui.view">
        <field name="name">account.analytic.distribution.model.tree</field>
        <field name="model">account.analytic.distribution.model</field>
        <field name="arch" type="xml">
            <tree string="Analytic Distribution Model" editable="top" multi_edit="1">
                <field name="partner_id" optional="show"/>
                <field name="partner_category_id" optional="hide"/>
                <field name="company_id" column_invisible="True"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
                <field name="analytic_distribution" widget="analytic_distribution" optional="show"
                       options="{'force_applicability': 'optional', 'disable_save': true}"/>
                <button name="action_read_distribution_model" type="object" string="View" class="float-end btn-secondary"/>
            </tree>
        </field>
    </record>

    <record id="account_analytic_distribution_model_form_view" model="ir.ui.view">
        <field name="name">account.analytic.distribution.model.form</field>
        <field name="model">account.analytic.distribution.model</field>
        <field name="arch" type="xml">
            <form string="Analytic Distribution Model">
                <sheet>
                    <group>
                        <group string="Conditions to meet" colspan="2">
                            <group>
                                <field name="partner_id"/>
                                <field name="partner_category_id"/>
                            </group>
                            <group>
                                <field name="company_id" groups="base.group_multi_company"/>
                                <field name="company_id" invisible="1"/>
                            </group>
                        </group>
                        <group string="Distribution to apply" colspan="2">
                            <field name="analytic_distribution" widget="analytic_distribution"
                                   options="{'force_applicability': 'optional', 'disable_save': true}"
                                   class="w-50"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="action_analytic_distribution_model" model="ir.actions.act_window">
        <field name="name">Analytic Distribution Models</field>
        <field name="res_model">account.analytic.distribution.model</field>
        <field name="view_mode">tree,form</field>
    </record>
</odoo>

```

## File: views\analytic_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_analytic_line_tree" model="ir.ui.view">
        <field name="name">account.analytic.line.tree</field>
        <field name="model">account.analytic.line</field>
        <field name="arch" type="xml">
            <tree string="Analytic Items" multi_edit="1">
                <field name="company_id" column_invisible="True"/>
                <field name="product_uom_category_id" column_invisible="True"/>
                <field name="date" optional="show"/>
                <field name="name"/>
                <field name="account_id"/>
                <field name="currency_id" column_invisible="True"/>
                <field name="unit_amount" sum="Quantity" optional="hide"/>
                <field name="product_uom_id" optional="hide"/>
                <field name="partner_id" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
                <field name="amount" sum="Total" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="view_account_analytic_line_filter" model="ir.ui.view">
        <field name="name">account.analytic.line.select</field>
        <field name="model">account.analytic.line</field>
        <field name="arch" type="xml">
            <search string="Search Analytic Lines">
                <field name="name"/>
                <field name="date"/>
                <separator/>
                <filter name="month" string="Date" date="date"/>
                <group string="Group By..." expand="0" name="groupby">
                    <separator/>
                    <filter string="Date" name="group_date" context="{'group_by': 'date'}"/>
                </group>
            </search>
        </field>
    </record>

    <record model="ir.actions.act_window" id="account_analytic_line_action">
        <field name="context">{'search_default_group_date': 1}</field>
        <field name="domain">[('auto_account_id','=', active_id)]</field>
        <field name="name">Gross Margin</field>
        <field name="res_model">account.analytic.line</field>
        <field name="view_mode">tree,form,graph,pivot</field>
        <field name="view_id" ref="view_account_analytic_line_tree"/>
        <field name="search_view_id" ref="view_account_analytic_line_filter"/>
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

    <record id="view_account_analytic_line_form" model="ir.ui.view">
        <field name="name">account.analytic.line.form</field>
        <field name="model">account.analytic.line</field>
        <field name="priority">1</field>
        <field name="arch" type="xml">
            <form string="Analytic Item">
            <field name="company_id" invisible="1"/>
            <sheet>
                <group>
                    <group name="analytic_item" string="Analytic Item">
                        <field name="name"/>
                        <field name="account_id"/>
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

    <record id="view_account_analytic_line_graph" model="ir.ui.view">
        <field name="name">account.analytic.line.graph</field>
        <field name="model">account.analytic.line</field>
        <field name="arch" type="xml">
            <graph string="Analytic Items" sample="1">
                <field name="account_id"/>
                <field name="unit_amount" type="measure" widget="float_time"/>
                <field name="amount" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="view_account_analytic_line_pivot" model="ir.ui.view">
        <field name="name">account.analytic.line.pivot</field>
        <field name="model">account.analytic.line</field>
        <field name="arch" type="xml">
            <pivot string="Analytic Items" sample="1">
                <field name="account_id"/>
                <field name="date" interval="month" type="col"/>
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
                                    <strong><span><t t-out="record.name.value"/></span></strong>
                                </div>
                                <div class="col-6 text-end">
                                    <strong><t t-out="record.date.value"/></strong>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-6 text-muted">
                                    <span><t t-out="record.account_id.value"/></span>
                                </div>
                                <div class="col-6">
                                    <span class="float-end text-end">
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

## File: views\analytic_plan_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_analytic_plan_form_view" model="ir.ui.view">
        <field name="name">account.analytic.plan.form</field>
        <field name="model">account.analytic.plan</field>
        <field name="arch" type="xml">
            <form string="Analytic Plans">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_children_plans" type="object" class="oe_stat_button" icon="fa-bars">
                            <field string="Subplans" name="children_count" widget="statinfo"/>
                        </button>
                        <button name="action_view_analytical_accounts" type="object" class="oe_stat_button" icon="fa-bars">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="all_account_count"/></span>
                                <span class="o_stat_text">Analytic Accounts</span>
                            </div>
                        </button>
                    </div>
                    <div class="oe_title">
                        <h1>
                            <field name="name"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="parent_id"/>
                            <field name="default_applicability"
                                   invisible="parent_id"/>
                        <field name="color" widget="color_picker"/>
                        </group>
                        <group>
                        </group>
                    </group>

                    <notebook>
                        <page string="Applicability" name="applicability"
                              invisible="parent_id">
                            <field name="applicability_ids">
                                <tree editable="bottom">
                                    <field name="business_domain"/>
                                    <field name="applicability"/>
                                    <field name="company_id" groups="base.group_multi_company"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="account_analytic_plan_tree_view" model="ir.ui.view">
        <field name="name">account.analytic.plan.tree</field>
        <field name="model">account.analytic.plan</field>
        <field name="arch" type="xml">
            <tree string="Analytic Plans" multi_edit="1">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="default_applicability"/>
                <field name="color" widget="color_picker"/>
            </tree>
        </field>
    </record>

    <record id="account_analytic_plan_action" model="ir.actions.act_window">
        <field name="name">Analytic Plans</field>
        <field name="res_model">account.analytic.plan</field>
        <field name="view_mode">tree,form</field>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'tree'}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('account_analytic_plan_form_view')})]"/>
        <field name="domain">[('parent_id', '=', False)]</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
              Click to add a new analytic account plan.
          </p>
        </field>
    </record>
</odoo>

```

