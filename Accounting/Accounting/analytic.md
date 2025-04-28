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
        <record id="analytic_plan_projects" model="account.analytic.plan">
            <field name="name">Projects</field>
            <field name="default_applicability">optional</field>
        </record>
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
    </data>
</odoo>

```

## File: models\analytic_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from odoo import api, fields, models, _
from odoo.exceptions import UserError


class AccountAnalyticAccount(models.Model):
    _name = 'account.analytic.account'
    _inherit = ['mail.thread']
    _description = 'Analytic Account'
    _order = 'plan_id, name asc'
    _check_company_auto = True
    _rec_names_search = ['name', 'code']

    name = fields.Char(
        string='Analytic Account',
        index='trigram',
        required=True,
        tracking=True,
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
        check_company=True,
        required=True,
    )
    root_plan_id = fields.Many2one(
        'account.analytic.plan',
        string='Root Plan',
        check_company=True,
        compute="_compute_root_plan",
        store=True,
    )
    color = fields.Integer(
        'Color Index',
        related='plan_id.color',
    )

    line_ids = fields.One2many(
        'account.analytic.line',
        'account_id',
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
        analytic_accounts = self.filtered('company_id')

        if not analytic_accounts:
            return

        self.flush_recordset(['company_id'])
        self.env['account.analytic.line'].flush_model(['account_id', 'company_id'])

        self._cr.execute('''
            SELECT line.account_id
            FROM account_analytic_line line
            JOIN account_analytic_account account ON line.account_id = account.id
            WHERE line.company_id != account.company_id and account.company_id IS NOT NULL
            AND account.id IN %s
        ''', [tuple(self.ids)])

        if self._cr.fetchone():
            raise UserError(_("You can't set a different company on your analytic account since there are some analytic items linked to it."))

    def name_get(self):
        res = []
        for analytic in self:
            name = analytic.name
            if analytic.code:
                name = f'[{analytic.code}] {name}'
            if analytic.partner_id.commercial_partner_id.name:
                name = f'{name} - {analytic.partner_id.commercial_partner_id.name}'
            res.append((analytic.id, name))
        return res

    def copy_data(self, default=None):
        default = dict(default or {})
        default.setdefault('name', _("%s (copy)", self.name))
        return super().copy_data(default)

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

    @api.depends('plan_id', 'plan_id.parent_path')
    def _compute_root_plan(self):
        for account in self:
            account.root_plan_id = int(account.plan_id.parent_path[:-1].split('/')[0]) if account.plan_id.parent_path else None

```

## File: models\analytic_distribution_model.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class NonMatchingDistribution(Exception):
    pass


class AccountAnalyticDistributionModel(models.Model):
    _name = 'account.analytic.distribution.model'
    _inherit = 'analytic.mixin'
    _description = 'Analytic Distribution Model'
    _rec_name = 'create_date'
    _order = 'id desc'

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
        query = """
            SELECT model.id
              FROM account_analytic_distribution_model model
              JOIN account_analytic_account account
                ON model.analytic_distribution ? CAST(account.id AS VARCHAR)
             WHERE account.company_id IS NOT NULL 
               AND (model.company_id IS NULL 
                OR model.company_id != account.company_id)
        """
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

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


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
        'Analytic Account',
        required=True,
        ondelete='restrict',
        index=True,
        check_company=True,
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
    plan_id = fields.Many2one(
        'account.analytic.plan',
        related='account_id.plan_id',
        store=True,
        readonly=True,
        compute_sudo=True,
    )
    category = fields.Selection(
        [('other', 'Other')],
        default='other',
    )

    @api.constrains('company_id', 'account_id')
    def _check_company_id(self):
        for line in self:
            if line.account_id.company_id and line.company_id.id != line.account_id.company_id.id:
                raise ValidationError(_('The selected account belongs to another company than the one you\'re trying to create an analytic item for'))

```

## File: models\analytic_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api, _
from odoo.tools.float_utils import float_round, float_compare
from odoo.exceptions import UserError, ValidationError

class AnalyticMixin(models.AbstractModel):
    _name = 'analytic.mixin'
    _description = 'Analytic Mixin'

    analytic_distribution = fields.Json(
        'Analytic',
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

    def init(self):
        # Add a gin index for json search on the keys, on the models that actually have a table
        query = ''' SELECT table_name
                    FROM information_schema.tables
                    WHERE table_name=%s '''
        self.env.cr.execute(query, [self._table])
        if self.env.cr.dictfetchone():
            query = f"""
                CREATE INDEX IF NOT EXISTS {self._table}_analytic_distribution_gin_index
                                        ON {self._table} USING gin(analytic_distribution);
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

        query = f"""
            SELECT id
            FROM {self._table}
            WHERE analytic_distribution ?| array[%s]
        """
        return [('id', operator_inselect, (query, [[str(account_id) for account_id in account_ids]]))]

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        args = self._apply_analytic_distribution_domain(args)
        return super()._search(args, offset, limit, order, count, access_rights_uid)

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        domain = self._apply_analytic_distribution_domain(domain)
        return super().read_group(domain, fields, groupby, offset, limit, orderby, lazy)

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
            mandatory_plans_ids = [plan['id'] for plan in self.env['account.analytic.plan'].sudo().get_relevant_plans(**kwargs) if plan['applicability'] == 'mandatory']
            if not mandatory_plans_ids:
                return
            decimal_precision = self.env['decimal.precision'].precision_get('Percentage Analytic')
            distribution_by_root_plan = {}
            for analytic_account_id, percentage in (self.analytic_distribution or {}).items():
                root_plan = self.env['account.analytic.account'].browse(int(analytic_account_id)).root_plan_id
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

```

## File: models\analytic_plan.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from random import randint


class AccountAnalyticPlan(models.Model):
    _name = 'account.analytic.plan'
    _description = 'Analytic Plans'
    _parent_store = True
    _rec_name = 'complete_name'
    _order = 'complete_name asc'
    _check_company_auto = True

    def _default_color(self):
        return randint(1, 11)

    name = fields.Char(required=True)
    description = fields.Text(string='Description')
    parent_id = fields.Many2one(
        'account.analytic.plan',
        string="Parent",
        ondelete='cascade',
        domain="[('id', '!=', id), ('company_id', 'in', [False, company_id])]",
        check_company=True,
    )
    parent_path = fields.Char(
        index='btree',
        unaccent=False,
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
    company_id = fields.Many2one(
        'res.company',
        string='Company',
        default=lambda self: self.env.company,
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

    default_applicability = fields.Selection(
        selection=[
            ('optional', 'Optional'),
            ('mandatory', 'Mandatory'),
            ('unavailable', 'Unavailable'),
        ],
        string="Default Applicability",
        required=True,
        default='optional',
        readonly=False,
    )
    applicability_ids = fields.One2many(
        'account.analytic.applicability',
        'analytic_plan_id',
        string='Applicability',
    )

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
        for plan in self:
            plan.all_account_count = self.env['account.analytic.account'].search_count([('plan_id', "child_of", plan.id)])

    @api.depends('children_ids')
    def _compute_children_count(self):
        for plan in self:
            plan.children_count = len(plan.children_ids)

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
        company_id = kwargs.get('company_id', self.env.company.id)
        record_account_ids = kwargs.get('existing_account_ids', [])
        all_plans = self.search([
            ('account_ids', '!=', False),
            '|', ('company_id', '=', company_id), ('company_id', '=', False),
        ])
        root_plans = self.browse({
            int(plan.parent_path.split('/')[0])
            for plan in all_plans
        }).filtered(lambda p: p._get_applicability(**kwargs) != 'unavailable')
        # If we have accounts that are already selected (before the applicability rules changed or from a model),
        # we want the plans that were unavailable to be shown in the list (and in optional, because the previous
        # percentage could be different from 0)
        forced_plans = self.env['account.analytic.account'].browse(record_account_ids).exists().mapped(
            'root_plan_id') - root_plans
        return sorted([
            {
                "id": plan.id,
                "name": plan.name,
                "color": plan.color,
                "applicability": plan._get_applicability(**kwargs) if plan in root_plans else 'optional',
                "all_account_count": plan.all_account_count
            }
            for plan in root_plans + forced_plans
        ], key=lambda d: (d['applicability'], d['id']))

    def _get_applicability(self, **kwargs):
        """ Returns the applicability of the best applicability line or the default applicability """
        self.ensure_one()
        if 'applicability' in kwargs:
            # For models for example, we want all plans to be visible, so we force the applicability
            return kwargs['applicability']
        else:
            score = 0
            applicability = self.default_applicability
            for applicability_rule in self.applicability_ids:
                score_rule = applicability_rule._get_score(**kwargs)
                if score_rule > score:
                    applicability = applicability_rule.applicability
                    score = score_rule
            return applicability

    def _get_default(self):
        plan = self.env['account.analytic.plan'].sudo().search(
            ['|', ('company_id', '=', False), ('company_id', '=', self.env.company.id)],
            limit=1)
        if plan:
            return plan
        else:
            return self.env['account.analytic.plan'].create({
                'name': 'Default',
                'company_id': self.env.company.id,
            })


class AccountAnalyticApplicability(models.Model):
    _name = 'account.analytic.applicability'
    _description = "Analytic Plan's Applicabilities"

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

    def _get_score(self, **kwargs):
        """ Gives the score of an applicability with the parameters of kwargs """
        self.ensure_one()
        if not kwargs.get('business_domain'):
            return 0
        else:
            return 1 if kwargs.get('business_domain') == self.business_domain else -1

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

    <record id="analytic_plan_comp_rule" model="ir.rule">
        <field name="name">Analytic plan multi company rule</field>
        <field name="model_id" ref="model_account_analytic_plan"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="analytic_distribution_model_comp_rule" model="ir.rule">
        <field name="name">Analytic distribution model multi company rule</field>
        <field name="model_id" ref="model_account_analytic_distribution_model"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
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
import { useService, useOwnedDialogs } from "@web/core/utils/hooks";
import { evaluateExpr } from "@web/core/py_js/py";
import { getNextTabableElement, getPreviousTabableElement } from "@web/core/utils/ui";
import { usePosition } from "@web/core/position_hook";
import { getActiveHotkey } from "@web/core/hotkeys/hotkey_service";
import { shallowEqual } from "@web/core/utils/arrays";
import { sprintf } from "@web/core/utils/strings";
import { _lt } from "@web/core/l10n/translation";
import { AutoComplete } from "@web/core/autocomplete/autocomplete";

import { standardFieldProps } from "@web/views/fields/standard_field_props";
import { TagsList } from "@web/views/fields/many2many_tags/tags_list";
import { useOpenMany2XRecord } from "@web/views/fields/relational_utils";
import { parseFloat as oParseFloat } from "@web/views/fields/parsers";
import { formatPercentage } from "@web/views/fields/formatters";
import { SelectCreateDialog } from "@web/views/view_dialogs/select_create_dialog";

const { Component, useState, useRef, useExternalListener, onWillUpdateProps, onWillStart, onPatched } = owl;

const PLAN_APPLICABILITY = {
    mandatory: _lt("Mandatory"),
    optional: _lt("Optional"),
}
const PLAN_STATUS = {
    invalid: _lt("Invalid"),
    ok: _lt("OK"),
}
export class AnalyticDistribution extends Component {
    setup(){
        this.orm = useService("orm");

        this.state = useState({
            showDropdown: false,
            list: {},
        });

        this.widgetRef = useRef("analyticDistribution");
        this.dropdownRef = useRef("analyticDropdown");
        this.mainRef = useRef("mainElement");
        usePosition(() => this.widgetRef.el, {
            popper: "analyticDropdown",
        });

        this.nextId = 1;
        this.focusSelector = false;

        onWillStart(this.willStart);
        onWillUpdateProps(this.willUpdate);
        onPatched(this.patched);

        useExternalListener(window, "click", this.onWindowClick, true);

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
            fieldString: this.env._t("Analytic Distribution Template"),
        });
        this.allPlans = [];
        this.lastAccount = this.props.account_field && this.props.record.data[this.props.account_field] || false;
        this.lastProduct = this.props.product_field && this.props.record.data[this.props.product_field] || false;

        this.selectCreateIsOpen = false;
        this.addDialog = useOwnedDialogs();
        this.onSearchMore = this._onSearchMore.bind(this);
    }

    // Lifecycle
    async willStart() {
        if (this.editingRecord) {
            await this.fetchAllPlans(this.props);
        }
        await this.formatData(this.props);
    }

    async willUpdate(nextProps) {
        // Unless force_applicability, Plans need to be retrieved again as the product or account might have changed
        // and thus different applicabilities apply
        // or a model applies that contains unavailable plans
        // This should only execute when these fields have changed, therefore we use the `_field` props.
        const valueChanged = JSON.stringify(this.props.value) !== JSON.stringify(nextProps.value);
        const currentAccount = this.props.account_field && this.props.record.data[this.props.account_field] || false;
        const currentProduct = this.props.product_field && this.props.record.data[this.props.product_field] || false;
        const accountChanged = !shallowEqual(this.lastAccount, currentAccount);
        const productChanged = !shallowEqual(this.lastProduct, currentProduct);
        if (valueChanged || accountChanged || productChanged) {
            if (!this.props.force_applicability) {
                await this.fetchAllPlans(nextProps);
            }
            this.lastAccount = accountChanged && currentAccount || this.lastAccount;
            this.lastProduct = productChanged && currentProduct || this.lastProduct;
            await this.formatData(nextProps);
        }
    }

    patched() {
        this.focusToSelector();
    }

    async formatData(nextProps) {
        const data = nextProps.value;
        const analytic_account_ids = Object.keys(data).map((id) => parseInt(id));
        const records = analytic_account_ids.length ? await this.fetchAnalyticAccounts([["id", "in", analytic_account_ids]]) : [];
        let widgetData = Object.assign({}, ...this.allPlans.map((plan) => ({[plan.id]: {...plan, distribution: []}})));
        records.map((record) => {
            if (!widgetData[record.root_plan_id[0]]) {
                // plans might not have been retrieved
                widgetData[record.root_plan_id[0]] = { distribution: [] }
            }
            widgetData[record.root_plan_id[0]].distribution.push({
                analytic_account_id: record.id,
                percentage: data[record.id],
                id: this.nextId++,
                group_id: record.root_plan_id[0],
                analytic_account_name: record.display_name,
                color: record.color,
            });
        });

        this.state.list = widgetData;
        if (records.length < Object.keys(data).length) {
            // analytic accounts were not found for some keys in the json data, they may have been deleted
            // save the json without them
            this.save();
        }
    }

    // ORM
    fetchPlansArgs(nextProps) {
        let args = {};
        if (this.props.business_domain_compute) {
            args['business_domain'] = evaluateExpr(this.props.business_domain_compute, this.props.record.evalContext);
        }
        if (this.props.business_domain) {
            args['business_domain'] = this.props.business_domain;
        }
        if (this.props.product_field && this.props.record.data[this.props.product_field]) {
            args['product'] = this.props.record.data[this.props.product_field][0];
        }
        if (this.props.account_field && this.props.record.data[this.props.account_field]) {
            args['account'] = this.props.record.data[this.props.account_field][0];
        }
        if (this.props.force_applicability) {
            args['applicability'] = this.props.force_applicability;
        }
        const existing_account_ids = Object.keys(nextProps.value).map((i) => parseInt(i));
        if (existing_account_ids.length) {
            args['existing_account_ids'] = existing_account_ids;
        }
        if (this.props.record.data.company_id) {
            args['company_id'] = this.props.record.data.company_id[0];
        }
        return args;
    }

    async fetchAllPlans(nextProps) {
        // TODO: Optimize to execute once for all records when `force_applicability` is set
        const argsPlan =  this.fetchPlansArgs(nextProps);
        this.allPlans = await this.orm.call("account.analytic.plan", "get_relevant_plans", [], argsPlan);
    }

    async fetchAnalyticAccounts(domain, limit=null) {
        const args = {
            domain: domain,
            fields: ["id", "display_name", "root_plan_id", "color"],
            context: [],
        }
        if (limit) {
            args['limit'] = limit;
        }
        if (domain.length === 1 && domain[0][0] === "id") {
            //batch these orm calls
            return await this.props.record.model.orm.read("account.analytic.account", domain[0][2], args.fields, {});
        }
        return await this.orm.call("account.analytic.account", "search_read", [], args);
    }

    // Autocomplete
    sourcesAnalyticAccount(groupId) {
        return [this.optionsSourceAnalytic(groupId)];
    }

    optionsSourceAnalytic(groupId) {
        return {
            placeholder: this.env._t("Loading..."),
            options:(searchTerm) => this.loadOptionsSourceAnalytic(groupId, searchTerm),
        };
    }

    analyticAccountDomain(groupId=null) {
        let domain = [['id', 'not in', this.existingAnalyticAccountIDs]];
        if (this.props.record.data.company_id){
            domain.push(
                '|',
                ['company_id', '=', this.props.record.data.company_id[0]],
                ['company_id', '=', false]
            );
        }

        if (groupId) {
            domain.push(['root_plan_id', '=', groupId]);
        }
        return domain;
    }

    searchAnalyticDomain(searchTerm) {
        return [
            '|',
            ["name", "ilike", searchTerm],
            '|',
            ['code', 'ilike', searchTerm],
            ['partner_id', 'ilike', searchTerm],
        ];
    }

    async loadOptionsSourceAnalytic(groupId, searchTerm) {
        const searchLimit = 6;

        const records = await this.fetchAnalyticAccounts([
            ...this.analyticAccountDomain(groupId),
            ...this.searchAnalyticDomain(searchTerm)], searchLimit + 1);

        let options = records.map((result) => ({
            value: result.id,
            label: result.display_name,
            group_id: result.root_plan_id[0],
            color: result.color,
        }));

        if (searchLimit < records.length) {
            options.push({
                label: this.env._t("Search More..."),
                action: (editedTag) => this.onSearchMore(searchTerm, editedTag),
                classList: "o_m2o_dropdown_option o_m2o_dropdown_option_search_more",
            });
        }

        if (!options.length) {
            options.push({
                label: this.env._t("No Analytic Accounts for this plan"),
                classList: "o_m2o_no_result",
                unselectable: true,
            });
        }

        return options;
    }

    async _onSearchMore(searchTerm, editedTag) {
        let dynamicFilters = [];
        if (searchTerm.length) {
            dynamicFilters = [
                {
                    description: sprintf(this.env._t("Quick search: %s"), searchTerm),
                    domain: this.searchAnalyticDomain(searchTerm),
                },
            ];
        }
        this.selectCreateIsOpen = true;
        this.addDialog(SelectCreateDialog, {
            title: this.env._t("Search: Analytic Account"),
            noCreate: true,
            multiSelect: true,
            resModel: 'account.analytic.account',
            context: {
                tree_view_ref: "analytic.view_account_analytic_account_list_select",
            },
            domain: this.analyticAccountDomain(editedTag.group_id),
            dynamicFilters: dynamicFilters,
            onSelected: async (resIds) => {
                const analytic_accounts = await this.fetchAnalyticAccounts([["id", "in", resIds]]);
                // modify the edited tag
                editedTag.analytic_account_id = analytic_accounts[0].id;
                editedTag.analytic_account_name = analytic_accounts[0].display_name;
                this.setFocusSelector(`.tag_${editedTag.id} .o_analytic_percentage`);
                if (analytic_accounts.length > 1) {
                    const planId = editedTag.group_id;
                    // remove the autofill line
                    this.list[planId].distribution = this.list[planId].distribution.filter((t) => !!t.analytic_account_id);
                    for (const account of analytic_accounts.slice(1)) {
                        // add new tags
                        const tag = this.newTag(planId);
                        tag.analytic_account_id = account.id;
                        tag.analytic_account_name = account.display_name;
                        this.list[planId].distribution.push(tag);
                    }
                }
                this.autoFill();
            },
            onCreateEdit: () => {},
        }, {
            onClose: () => {
                if (!editedTag.analytic_account_id) {
                    this.setFocusSelector(`.tag_${editedTag.id} .o_analytic_account_name`);
                    this.focusToSelector();
                }
                this.selectCreateIsOpen = false;
            },
        });
    }

    autoCompleteInputChanged(distTag, inputValue) {
        if (inputValue === "" && distTag.analytic_account_id) {
            this.deleteTag(distTag.id, distTag.group_id);
        }
    }

    // Editing Distributions
    async onSelect(option, params, tag) {
        if (option.action) {
            return option.action(tag);
        }
        const selected_option = Object.getPrototypeOf(option);
        tag.analytic_account_id = parseInt(selected_option.value);
        tag.analytic_account_name = selected_option.label;
        tag.color = selected_option.color;
        this.setFocusSelector(`.tag_${tag.id} .o_analytic_percentage`);
        this.autoFill();
    }

    async percentageChanged(dist_tag, ev) {
        dist_tag.percentage = this.parse(ev.target.value);
        if (dist_tag.percentage == 0) {
            this.deleteTag(dist_tag.id, dist_tag.group_id);
        }
        this.autoFill();
    }

    deleteTag(id, fromGroup) {
        // find the next tag to focus to before deleting the tag
        const allTags = this.listFlat;
        const currentTagIndex = allTags.findIndex((t) => t.id === id);
        const nextTag = allTags[(currentTagIndex + 1) % allTags.length];
        // remove the tag from the groups distribution
        this.list[fromGroup].distribution = this.list[fromGroup].distribution.filter((dist_tag) => dist_tag.id != id);
        if (!this.isDropdownOpen){
            this.save();
        } else {
            this.setFocusSelector(`.tag_${nextTag.id} .o_analytic_account_name`);
            this.autoFill();
        }
    }

    // Getters
    get tags() {
        return this.listReady.map((dist_tag) => ({
            id: dist_tag.id,
            text: `${dist_tag.analytic_account_name}${dist_tag.percentage > 99.99 && dist_tag.percentage < 100.01 ? "" : " " + this.formatPercentage(dist_tag.percentage)}`,
            colorIndex: dist_tag.color,
            group_id: dist_tag.group_id,
            onClick: (ev) => this.tagClicked(ev, dist_tag.id),
            onDelete: this.editingRecord ? () => this.deleteTag(dist_tag.id, dist_tag.group_id) : undefined
        }));
    }

    get listForJson() {
        let res = {};
        this.listReady.map(({analytic_account_id, percentage}) => {
            res[parseInt(analytic_account_id)] = percentage;
        });
        return res;
    }

    get firstIncompletePlanId() {
        for (const group_id in this.list) {
            if (this.groupStatus(group_id) == "invalid") {
                return group_id;
            }
        }
        return 0;
    }

    get existingAnalyticAccountIDs() {
        return this.listFlat.filter((i) => !!i.analytic_account_id).map((i) => i.analytic_account_id);
    }

    get listReady() {
        return this.listFlat.filter((dist_tag) => this.tagIsReady(dist_tag));
    }

    get listFlat() {
        return Object.values(this.list).flatMap((g) => g.distribution);
    }

    get list() {
        return this.state.list;
    }

    get sortedList() {
        return Object.values(this.list).sort((a, b) => {
            const aApp = a.applicability,
                  bApp = b.applicability;
            return aApp > bApp ? 1 : aApp < bApp ? -1 : 0;
        });
    }

    get allowSave() {
        if (this.firstIncompletePlanId > 0) {
            return false;
        }
        return this.props.allow_save;
    }

    get editingRecord() {
        return !this.props.readonly;
    }

    get isDropdownOpen() {
        return this.state.showDropdown && !!this.dropdownRef.el;
    }

    statusDescription(group_id) {
        const group = this.list[group_id];
        const applicability = PLAN_APPLICABILITY[group.applicability];
        const status = PLAN_STATUS[this.groupStatus(group_id)];
        return `${applicability} - ${status} ${this.formatPercentage(this.sumByGroup(group_id))}`;
    }

    groupStatus(id) {
        const group = this.list[id];
        if (group.applicability === 'mandatory') {
            const sum = this.sumByGroup(id);
            if (sum < 99.99 || sum >= 100.01) {
                return 'invalid';
            }
        }
        return 'ok';
    }

    listReadyByGroup(id) {
        return this.list[id].distribution.filter((tag) => this.tagIsReady(tag));
    }

    tagIsReady({analytic_account_id, percentage}) {
        return !!analytic_account_id && !!percentage;
    }

    sumByGroup(id) {
        return this.listReadyByGroup(id).reduce((prev, next) => prev + (parseFloat(next.percentage) || 0), 0);
    }

    remainderByGroup(id) {
        return 100 - Math.min(this.sumByGroup(id), 100);
    }

    // actions
    newTag(group_id) {
        return {
            id: this.nextId++,
            group_id: group_id,
            analytic_account_id: null,
            analytic_account_name: "",
            percentage: this.remainderByGroup(group_id),
            color: this.list[group_id].color,
        }
    }

    /**
     * This method is typically called when opening the popup and after any change to the distribution.
     * The remainder, will be placed in the first tag with 0%.
     * It adds an empty tag allowing users to continue the distribution (replaced 'Add a Line').
     * Where an empty tag exists, the percentage is updated.
     */
    autoFill() {
        for (const group of this.allPlans.filter((p) => p.all_account_count > 0)) {
            // update the first unmodified tag containing 0%
            const tagToUpdate = this.list[group.id].distribution.find((t) => t.percentage == 0);
            if (tagToUpdate) {
                tagToUpdate.percentage = this.remainderByGroup(group.id);
            }
            // a tag with no analytic account must always be added / updated
            const emptyTag = this.list[group.id].distribution.find((t) => !t.analytic_account_id);
            if (emptyTag) {
                emptyTag.percentage = this.remainderByGroup(group.id);
            } else {
                this.list[group.id].distribution.push(this.newTag(group.id));
            }
        }
    }

    cleanUp() {
        for (const group_id in this.list){
            this.list[group_id].distribution = this.listReadyByGroup(group_id);
        }
    }

    async save() {
        const currentDistribution = this.listForJson;
        await this.props.update(currentDistribution);
    }

    onSaveNew() {
        const { record, product_field, account_field } = this.props;
        this.openTemplate({
            resId: false,
            context: {
                default_analytic_distribution: this.listForJson,
                default_partner_id: record.data['partner_id'] ? record.data['partner_id'][0] : undefined,
                default_product_id: product_field ? record.data[product_field][0] : undefined,
                default_account_prefix: account_field ? record.data[account_field][1].substr(0, 3) : undefined,
            },
        });

        this.closeAnalyticEditor();
    }

    forceCloseEditor() {
        // focus to the main Element but the dropdown should not open
        this.preventOpen = true;
        this.closeAnalyticEditor();
        this.mainRef.el.focus();
        this.preventOpen = false;
    }

    closeAnalyticEditor() {
        this.cleanUp();
        this.save();
        this.state.showDropdown = false;
    }

    async openAnalyticEditor() {
        if (!this.allPlans.length) {
            await this.fetchAllPlans(this.props);
            await this.formatData(this.props);
        }
        this.autoFill();
        const incompletePlan = this.firstIncompletePlanId;
        this.setFocusSelector(incompletePlan ? `#plan_${incompletePlan} .incomplete`: ".analytic_json_popup");
        this.state.showDropdown = true;
    }

    tagClicked(ev, id) {
        if (this.editingRecord && !this.isDropdownOpen) {
            this.openAnalyticEditor();
        }
        if (this.isDropdownOpen) {
            this.setFocusSelector(`.tag_${id} .o_analytic_percentage`);
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
        if (!!this.focusSelector && this.isDropdownOpen) {
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
    onWidgetKeydown(ev) {
        if (!this.editingRecord) {
            return;
        }
        const hotkey = getActiveHotkey(ev);
        switch (hotkey) {
            case "enter":
            case "tab": {
                if (this.isDropdownOpen) {
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
        if (this.isDropdownOpen
            && this.dropdownRef.el && !this.dropdownRef.el.contains(ev.target)
            && !this.widgetRef.el.contains(ev.target)
            && !this.selectCreateIsOpen) {
            this.forceCloseEditor();
        }
    }

    // formatters and parsers
    parse(value) {
        try {
            return typeof value === 'string' || value instanceof String ? oParseFloat(value.replace('%', '')) : value;
        } catch (_error) {
            return 0;
        }
    }

    formatPercentage(value) {
        return formatPercentage(value / 100, { digits: [false, this.props.record.data.analytic_precision || 2] });
    }
}
AnalyticDistribution.template = "analytic.AnalyticDistribution";
AnalyticDistribution.supportedTypes = ["char", "text"];
AnalyticDistribution.components = {
    AutoComplete,
    TagsList,
}

AnalyticDistribution.fieldDependencies = {
    analytic_precision: { type: 'integer' },
}
AnalyticDistribution.props = {
    ...standardFieldProps,
    business_domain: { type: String, optional: true },
    account_field: { type: String, optional: true },
    product_field: { type: String, optional: true },
    business_domain_compute: { type: String, optional: true },
    force_applicability: { type: String, optional: true },
    allow_save: { type: Boolean },
}
AnalyticDistribution.extractProps = ({ field, attrs }) => {
    return {
        business_domain: attrs.options.business_domain,
        account_field: attrs.options.account_field,
        product_field: attrs.options.product_field,
        business_domain_compute: attrs.business_domain_compute,
        force_applicability: attrs.options.force_applicability,
        allow_save: !Boolean(attrs.options.disable_save),
    };
};

registry.category("fields").add("analytic_distribution", AnalyticDistribution);

```

## File: static\src\components\analytic_distribution\analytic_distribution.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="analytic.AnalyticDistribution" owl="1">
        <div class="o_field_tags d-inline-flex flex-wrap mw-100" t-att-class="{'o_tags_input o_input': !props.readonly}" t-ref="analyticDistribution" t-on-keydown="onWidgetKeydown">
            <TagsList tags="this.tags"/>
            <div t-if="!props.readonly" class="o_input_dropdown d-inline-flex w-100" tabindex="0" t-ref="mainElement" t-on-focus="onMainElementFocus" t-on-click="onMainElementFocus">
                <span class="analytic_distribution_placeholder"/>
                <a role="button" class="o_dropdown_button" draggable="false"/>
                <t t-call="analytic.AnalyticDistributionPopup"/>
            </div>
        </div>
    </t>

    <t t-name="analytic.AnalyticDistributionPopup" owl="1">
        <div class="analytic_distribution_popup o-dropdown-menu show rounded py-0" t-if="state.showDropdown" t-ref="analyticDropdown">
            <div class="popover-header sticky-top">
                <div class="d-flex">
                    <div class="h5 mt-2 me-auto">
                        Analytic
                        <span t-if="tags.length and allowSave" class="btn btn-link" t-on-click="onSaveNew">New Model</span>
                    </div>
                    <div class="popupButtons">
                        <span class="o_button ms-2 cursor-pointer" t-on-click.stop="() => this.closeAnalyticEditor()"><span class="fa fa-close"/></span>
                    </div>
                </div>
            </div>
            <div class="p-2">
                <span t-if="!sortedList.length">No plans available</span>
                <t t-foreach="sortedList" t-as="plan" t-key="plan.id">
                    <table class="o_list_table table table-sm table-hover o_analytic_table mb-2 table-borderless" t-attf-id="plan_{{plan.id}}">
                        <tr class="border-bottom">
                            <th class="o_analytic_account_name">
                                <t t-esc="plan.name"/>
                                <t t-if="plan.account_count === 0"> (no accounts)</t>
                                <span t-if="plan.applicability === 'mandatory'" t-attf-class="o_status d-inline-block o_analytic_status_{{groupStatus(plan.id)}}" t-att-title="statusDescription(plan.id)"/>
                            </th>
                        </tr>
                        <t t-foreach="plan.distribution" t-as="dist_tag" t-key="dist_tag.id">
                            <tr t-attf-class="{{tagIsReady(dist_tag) and 'ready' or !!dist_tag.analytic_account_id and 'to_remove' or 'incomplete'}} tag_{{dist_tag.id}}">
                                <td class="o_analytic_account_name">
                                    <AutoComplete
                                        id="dist_tag.id.toString()"
                                        placeholder="'Search Analytic Account'"
                                        value="dist_tag.analytic_account_name"
                                        sources="sourcesAnalyticAccount(plan.id)"
                                        autoSelect="true"
                                        onSelect.alike="(option, params) => this.onSelect(option, params, dist_tag)"
                                        onChange.alike="({inputValue}) => this.autoCompleteInputChanged(dist_tag, inputValue)"/>
                                </td>
                                <td class="o_analytic_percentage">
                                    <input
                                        class="o_input"
                                        inputmode="numeric"
                                        type="text"
                                        t-att-value="formatPercentage(dist_tag.percentage)"
                                        t-on-click.stop=""
                                        t-on-change="(ev) => this.percentageChanged(dist_tag, ev)"/>
                                </td>
                                <td>
                                    <span t-if="dist_tag.analytic_account_id" class="fa fa-trash-o cursor-pointer" t-on-click.stop="() => this.deleteTag(dist_tag.id, dist_tag.group_id)"/>
                                </td>
                            </tr>
                        </t>
                    </table>
                </t>
                <div tabindex="0" class="hidden-focus"/>
            </div>
        </div>
    </t>

</templates>

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
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
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
                    <field name="company_id" invisible="1"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="name" string="Name"/>
                    <field name="code"/>
                    <field name="partner_id"/>
                    <field name="plan_id"/>
                    <field name="active" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="debit" sum="Debit" invisible="1"/>
                    <field name="credit" sum="Credit" invisible="1"/>
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
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
                <field name="analytic_distribution" widget="analytic_distribution" optional="show"/>
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
                        <group string="Simultaneous conditions to meet" colspan="2">
                            <group>
                                <field name="partner_id"/>
                                <field name="partner_category_id"/>
                            </group>
                            <group>
                                <field name="company_id" groups="base.group_multi_company"/>
                            </group>
                        </group>
                        <group string="Analytic distribution to apply" colspan="2">
                            <field name="analytic_distribution" widget="analytic_distribution"
                                   options="{'force_applicability': 'optional', 'disable_save': true}"/>
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
                <field name="company_id" invisible="1"/>
                <field name="product_uom_category_id" invisible="1"/>
                <field name="date" optional="show"/>
                <field name="name"/>
                <field name="account_id"/>
                <field name="plan_id"/>
                <field name="currency_id" invisible="1"/>
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
                <field name="account_id"/>
                <field name="plan_id"/>
                <filter string="Date" name="date" date="date"/>
                <filter string="Analytic Account" name="group_by_analytic_account" context="{'group_by': 'account_id'}"/>
                <filter string="Analytic Plan" name="group_by_analytic_plan" context="{'group_by': 'plan_id'}"/>
                <group string="Group By..." expand="0" name="groupby">
                    <filter string="Date" name="group_date" context="{'group_by': 'date'}"/>
                </group>
            </search>
        </field>
    </record>

    <record model="ir.actions.act_window" id="account_analytic_line_action">
        <field name="context">{'search_default_group_date': 1, 'default_account_id': active_id}</field>
        <field name="domain">[('account_id','=', active_id)]</field>
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
                <field name="unit_amount" type="measure"/>
                <field name="amount" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="view_account_analytic_line_pivot" model="ir.ui.view">
        <field name="name">account.analytic.line.pivot</field>
        <field name="model">account.analytic.line</field>
        <field name="arch" type="xml">
            <pivot string="Analytic Items" sample="1">
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
                                <div class="col-6 text-end">
                                    <strong><t t-esc="record.date.value"/></strong>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-6 text-muted">
                                    <span><t t-esc="record.account_id.value"/></span>
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
                <field name="company_id" invisible="1"/>
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
                                   attrs="{'invisible': [('parent_id', '!=', False)]}"/>
                        <field name="color" widget="color_picker"/>
                        </group>
                        <group>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>

                    <notebook>
                        <page string="Applicability" name="applicability"
                              attrs="{'invisible': [('parent_id', '!=', False)]}">
                            <field name="applicability_ids">
                                <tree editable="bottom">
                                    <field name="business_domain"/>
                                    <field name="applicability"/>
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
                <field name="name"/>
                <field name="default_applicability"/>
                <field name="color" widget="color_picker"/>
                <field name="company_id" groups="base.group_multi_company"/>
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

