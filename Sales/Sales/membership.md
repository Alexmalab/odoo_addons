# Odoo Module: membership

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Members',
    'version': '1.0',
    'category': 'Sales/Sales',
    'description': """
This module allows you to manage all operations for managing memberships.
=========================================================================

It supports different kind of members:
--------------------------------------
    * Free member
    * Associated member (e.g.: a group subscribes to a membership for all subsidiaries)
    * Paid members
    * Special member prices

It is integrated with sales and accounting to allow you to automatically
invoice and send propositions for membership renewal.
    """,
    'depends': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'wizard/membership_invoice_views.xml',
        'data/membership_data.xml',
        'views/product_views.xml',
        'views/partner_views.xml',
        'report/report_membership_views.xml',
    ],
    'website': 'https://www.odoo.com/app/forum',
    'license': 'LGPL-3',
}

```

## File: data\membership_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record id="ir_cron_update_membership" model="ir.cron">
            <field name="name">Membership: update memberships</field>
            <field name="model_id" ref="base.model_res_partner"/>
            <field name="state">code</field>
            <field name="code">model._cron_update_membership()</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

from datetime import date


class AccountMove(models.Model):
    _inherit = 'account.move'

    def button_draft(self):
        # OVERRIDE to update the cancel date.
        res = super(AccountMove, self).button_draft()
        for move in self:
            if move.move_type == 'out_invoice':
                self.env['membership.membership_line'].search([
                    ('account_invoice_line', 'in', move.mapped('invoice_line_ids').ids)
                ]).write({'date_cancel': False})
        return res

    def button_cancel(self):
        # OVERRIDE to update the cancel date.
        res = super(AccountMove, self).button_cancel()
        for move in self:
            if move.move_type == 'out_invoice':
                self.env['membership.membership_line'].search([
                    ('account_invoice_line', 'in', move.mapped('invoice_line_ids').ids)
                ]).write({'date_cancel': fields.Date.today()})
        return res

    def write(self, vals):
        # OVERRIDE to write the partner on the membership lines.
        res = super(AccountMove, self).write(vals)
        if 'partner_id' in vals:
            self.env['membership.membership_line'].search([
                ('account_invoice_line', 'in', self.mapped('invoice_line_ids').ids)
            ]).write({'partner': vals['partner_id']})
        return res


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def write(self, vals):
        # OVERRIDE
        res = super(AccountMoveLine, self).write(vals)

        to_process = self.filtered(lambda line: line.move_id.move_type == 'out_invoice' and line.product_id.membership)

        # Nothing to process, break.
        if not to_process:
            return res

        existing_memberships = self.env['membership.membership_line'].search([
            ('account_invoice_line', 'in', to_process.ids)])
        to_process = to_process - existing_memberships.mapped('account_invoice_line')

        # All memberships already exist, break.
        if not to_process:
            return res

        memberships_vals = []
        for line in to_process:
            date_from = line.product_id.membership_date_from
            date_to = line.product_id.membership_date_to
            if (date_from and date_from < (line.move_id.invoice_date or date.min) < (date_to or date.min)):
                date_from = line.move_id.invoice_date
            memberships_vals.append({
                'partner': line.move_id.partner_id.id,
                'membership_id': line.product_id.id,
                'member_price': line.price_unit,
                'date': fields.Date.today(),
                'date_from': date_from,
                'date_to': date_to,
                'account_invoice_line': line.id,
            })
        self.env['membership.membership_line'].create(memberships_vals)
        return res

    @api.model_create_multi
    def create(self, vals_list):
        # OVERRIDE
        lines = super(AccountMoveLine, self).create(vals_list)
        to_process = lines.filtered(lambda line: line.move_id.move_type == 'out_invoice' and line.product_id.membership)

        # Nothing to process, break.
        if not to_process:
            return lines

        existing_memberships = self.env['membership.membership_line'].search([
            ('account_invoice_line', 'in', to_process.ids)])
        to_process = to_process - existing_memberships.mapped('account_invoice_line')

        # All memberships already exist, break.
        if not to_process:
            return lines

        memberships_vals = []
        for line in to_process:
            date_from = line.product_id.membership_date_from
            date_to = line.product_id.membership_date_to
            if (date_from and date_from < (line.move_id.invoice_date or date.min) < (date_to or date.min)):
                date_from = line.move_id.invoice_date
            memberships_vals.append({
                'partner': line.move_id.partner_id.id,
                'membership_id': line.product_id.id,
                'member_price': line.price_unit,
                'date': fields.Date.today(),
                'date_from': date_from,
                'date_to': date_to,
                'account_invoice_line': line.id,
            })
        self.env['membership.membership_line'].create(memberships_vals)
        return lines

```

## File: models\membership.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

STATE = [
    ('none', 'Non Member'),
    ('canceled', 'Cancelled Member'),
    ('old', 'Old Member'),
    ('waiting', 'Waiting Member'),
    ('invoiced', 'Invoiced Member'),
    ('free', 'Free Member'),
    ('paid', 'Paid Member'),
]


class MembershipLine(models.Model):
    _name = 'membership.membership_line'
    _rec_name = 'partner'
    _order = 'id desc'
    _description = 'Membership Line'

    partner = fields.Many2one('res.partner', string='Partner', ondelete='cascade', index=True)
    membership_id = fields.Many2one('product.product', string="Membership", required=True)
    date_from = fields.Date(string='From', readonly=True)
    date_to = fields.Date(string='To', readonly=True)
    date_cancel = fields.Date(string='Cancel date')
    date = fields.Date(string='Join Date',
        help="Date on which member has joined the membership")
    member_price = fields.Float(string='Membership Fee',
        digits='Product Price', required=True,
        help='Amount for the membership')
    account_invoice_line = fields.Many2one('account.move.line', string='Account Invoice line', readonly=True, ondelete='cascade')
    account_invoice_id = fields.Many2one('account.move', related='account_invoice_line.move_id', string='Invoice', readonly=True)
    company_id = fields.Many2one('res.company', related='account_invoice_line.move_id.company_id', string="Company", readonly=True, store=True)
    state = fields.Selection(STATE, compute='_compute_state', string='Membership Status', store=True,
        help="It indicates the membership status.\n"
             "-Non Member: A member who has not applied for any membership.\n"
             "-Cancelled Member: A member who has cancelled his membership.\n"
             "-Old Member: A member whose membership date has expired.\n"
             "-Waiting Member: A member who has applied for the membership and whose invoice is going to be created.\n"
             "-Invoiced Member: A member whose invoice has been created.\n"
             "-Paid Member: A member who has paid the membership amount.")

    @api.depends('account_invoice_id.state',
                 'account_invoice_id.amount_residual',
                 'account_invoice_id.payment_state')
    def _compute_state(self):
        """Compute the state lines """
        if not self:
            return

        self._cr.execute('''
            SELECT reversed_entry_id, COUNT(id)
            FROM account_move
            WHERE reversed_entry_id IN %s
            GROUP BY reversed_entry_id
        ''', [tuple(self.mapped('account_invoice_id.id'))])
        reverse_map = dict(self._cr.fetchall())
        for line in self:
            move_state = line.account_invoice_id.state
            payment_state = line.account_invoice_id.payment_state

            line.state = 'none'
            if move_state == 'draft':
                line.state = 'waiting'
            elif move_state == 'posted':
                if payment_state == 'paid':
                    if reverse_map.get(line.account_invoice_id.id):
                        line.state = 'canceled'
                    else:
                        line.state = 'paid'
                elif payment_state == 'in_payment':
                    line.state = 'paid'
                elif payment_state in ('not_paid', 'partial'):
                    line.state = 'invoiced'
            elif move_state == 'cancel':
                line.state = 'canceled'

```

## File: models\partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import date
from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from . import membership


class Partner(models.Model):
    _inherit = 'res.partner'

    associate_member = fields.Many2one('res.partner', string='Associate Member',
        help="A member with whom you want to associate your membership."
             "It will consider the membership state of the associated member.")
    member_lines = fields.One2many('membership.membership_line', 'partner', string='Membership')
    free_member = fields.Boolean(string='Free Member',
        help="Select if you want to give free membership.")
    membership_amount = fields.Float(string='Membership Amount', digits=(16, 2),
        help='The price negotiated by the partner')
    membership_state = fields.Selection(membership.STATE, compute='_compute_membership_state',
        string='Current Membership Status', store=True, recursive=True,
        help='It indicates the membership state.\n'
             '-Non Member: A partner who has not applied for any membership.\n'
             '-Cancelled Member: A member who has cancelled his membership.\n'
             '-Old Member: A member whose membership date has expired.\n'
             '-Waiting Member: A member who has applied for the membership and whose invoice is going to be created.\n'
             '-Invoiced Member: A member whose invoice has been created.\n'
             '-Paying member: A member who has paid the membership fee.')
    membership_start = fields.Date(compute='_compute_membership_state',
        string ='Membership Start Date', store=True,
        help="Date from which membership becomes active.")
    membership_stop = fields.Date(compute='_compute_membership_state',
        string ='Membership End Date', store=True,
        help="Date until which membership remains active.")
    membership_cancel = fields.Date(compute='_compute_membership_state',
        string ='Cancel Membership Date', store=True,
        help="Date on which membership has been cancelled")

    @api.depends('member_lines.account_invoice_line',
                 'member_lines.account_invoice_line.move_id.state',
                 'member_lines.account_invoice_line.move_id.payment_state',
                 'member_lines.account_invoice_line.move_id.partner_id',
                 'free_member',
                 'member_lines.date_to', 'member_lines.date_from',
                 'associate_member', 'associate_member.membership_state')
    def _compute_membership_state(self):
        today = fields.Date.today()
        for partner in self:
            partner.membership_start = self.env['membership.membership_line'].search([
                ('partner', '=', partner.associate_member.id or partner.id), ('date_cancel', '=', False)
            ], limit=1, order='date_from').date_from
            partner.membership_stop = self.env['membership.membership_line'].search([
                ('partner', '=', partner.associate_member.id or partner.id), ('date_cancel', '=', False)
            ], limit=1, order='date_to desc').date_to
            partner.membership_cancel = self.env['membership.membership_line'].search([
                ('partner', '=', partner.id)
            ], limit=1, order='date_cancel').date_cancel

            if partner.associate_member:
                partner.membership_state = partner.associate_member.membership_state
                continue

            if partner.free_member and partner.membership_state != 'paid':
                partner.membership_state = 'free'
                continue

            for mline in partner.member_lines:
                if (mline.date_to or date.min) >= today and (mline.date_from or date.min) <= today:
                    partner.membership_state = mline.state
                    break
                elif ((mline.date_from or date.min) < today and (mline.date_to or date.min) <= today and \
                      (mline.date_from or date.min) < (mline.date_to or date.min)):
                    if mline.account_invoice_id and mline.account_invoice_id.payment_state in ('in_payment', 'paid'):
                        partner.membership_state = 'old'
                    elif mline.account_invoice_id and mline.account_invoice_id.state == 'cancel':
                        partner.membership_state = 'canceled'
                    break
            else:
                partner.membership_state = 'none'

    @api.constrains('associate_member')
    def _check_recursion_associate_member(self):
        for partner in self:
            level = 100
            while partner:
                partner = partner.associate_member
                if not level:
                    raise ValidationError(_('You cannot create recursive associated members.'))
                level -= 1

    @api.model
    def _cron_update_membership(self):
        partners = self.search([('membership_state', 'in', ['invoiced', 'paid'])])
        # mark the field to be recomputed, and recompute it
        self.env.add_to_compute(self._fields['membership_state'], partners)

    def create_membership_invoice(self, product, amount):
        """ Create Customer Invoice of Membership for partners.
        """
        invoice_vals_list = []
        for partner in self:
            addr = partner.address_get(['invoice'])
            if partner.free_member:
                raise UserError(_("Partner is a free Member."))
            if not addr.get('invoice', False):
                raise UserError(_("Partner doesn't have an address to make the invoice."))

            invoice_vals_list.append({
                'move_type': 'out_invoice',
                'partner_id': partner.id,
                'invoice_payment_term_id': partner.property_payment_term_id.id,
                'invoice_line_ids': [
                    (0, None, {'product_id': product.id, 'quantity': 1, 'price_unit': amount, 'tax_ids': [(6, 0, product.taxes_id.ids)]})
                ]
            })

        return self.env['account.move'].create(invoice_vals_list)

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Product(models.Model):
    _inherit = 'product.template'

    membership = fields.Boolean(help='Check if the product is eligible for membership.')
    membership_date_from = fields.Date(string='Membership Start Date',
        help='Date from which membership becomes active.')
    membership_date_to = fields.Date(string='Membership End Date',
        help='Date until which membership remains active.')

    _sql_constraints = [
        ('membership_date_greater', 'check(membership_date_to >= membership_date_from)', 'Error ! Ending Date cannot be set before Beginning Date.')
    ]

    @api.model
    def fields_view_get(self, view_id=None, view_type='form', toolbar=False, submenu=False):
        if self._context.get('product') == 'membership_product':
            if view_type == 'form':
                view_id = self.env.ref('membership.membership_products_form').id
            else:
                view_id = self.env.ref('membership.membership_products_tree').id
        return super(Product, self).fields_view_get(view_id=view_id, view_type=view_type, toolbar=toolbar, submenu=submenu)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import membership
from . import partner
from . import product
from . import account_move

```

## File: report\report_membership.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools

STATE = [
    ('none', 'Non Member'),
    ('canceled', 'Cancelled Member'),
    ('old', 'Old Member'),
    ('waiting', 'Waiting Member'),
    ('invoiced', 'Invoiced Member'),
    ('free', 'Free Member'),
    ('paid', 'Paid Member'),
]


class ReportMembership(models.Model):
    '''Membership Analysis'''

    _name = 'report.membership'
    _description = 'Membership Analysis'
    _auto = False
    _rec_name = 'start_date'

    start_date = fields.Date(string='Start Date', readonly=True)
    date_to = fields.Date(string='End Date', readonly=True, help="End membership date")
    num_waiting = fields.Integer(string='# Waiting', readonly=True)
    num_invoiced = fields.Integer(string='# Invoiced', readonly=True)
    num_paid = fields.Integer(string='# Paid', readonly=True)
    tot_pending = fields.Float(string='Pending Amount', digits=0, readonly=True)
    tot_earned = fields.Float(string='Earned Amount', digits=0, readonly=True)
    partner_id = fields.Many2one('res.partner', string='Member', readonly=True)
    associate_member_id = fields.Many2one('res.partner', string='Associate Member', readonly=True)
    membership_id = fields.Many2one('product.product', string='Membership Product', readonly=True)
    membership_state = fields.Selection(STATE, string='Current Membership State', readonly=True)
    user_id = fields.Many2one('res.users', string='Salesperson', readonly=True)
    company_id = fields.Many2one('res.company', string='Company', readonly=True)
    quantity = fields.Integer(readonly=True)

    def init(self):
        '''Create the view'''
        tools.drop_view_if_exists(self._cr, self._table)
        self._cr.execute("""
        CREATE OR REPLACE VIEW %s AS (
        SELECT
        MIN(id) AS id,
        partner_id,
        count(membership_id) as quantity,
        user_id,
        membership_state,
        associate_member_id,
        membership_amount,
        date_to,
        start_date,
        COUNT(num_waiting) AS num_waiting,
        COUNT(num_invoiced) AS num_invoiced,
        COUNT(num_paid) AS num_paid,
        SUM(tot_pending) AS tot_pending,
        SUM(tot_earned) AS tot_earned,
        membership_id,
        company_id
        FROM
        (SELECT
            MIN(p.id) AS id,
            p.id AS partner_id,
            p.user_id AS user_id,
            p.membership_state AS membership_state,
            p.associate_member AS associate_member_id,
            p.membership_amount AS membership_amount,
            p.membership_stop AS date_to,
            p.membership_start AS start_date,
            CASE WHEN ml.state = 'waiting'  THEN ml.id END AS num_waiting,
            CASE WHEN ml.state = 'invoiced' THEN ml.id END AS num_invoiced,
            CASE WHEN ml.state = 'paid'     THEN ml.id END AS num_paid,
            CASE WHEN ml.state IN ('waiting', 'invoiced') THEN SUM(aml.price_subtotal) ELSE 0 END AS tot_pending,
            CASE WHEN ml.state = 'paid' OR p.membership_state = 'old' THEN SUM(aml.price_subtotal) ELSE 0 END AS tot_earned,
            ml.membership_id AS membership_id,
            p.company_id AS company_id
            FROM res_partner p
            LEFT JOIN membership_membership_line ml ON (ml.partner = p.id)
            LEFT JOIN account_move_line aml ON (ml.account_invoice_line = aml.id)
            LEFT JOIN account_move am ON (aml.move_id = am.id)
            WHERE p.membership_state != 'none' and p.active = 'true'
            GROUP BY
              p.id,
              p.user_id,
              p.membership_state,
              p.associate_member,
              p.membership_amount,
              p.membership_start,
              ml.membership_id,
              p.company_id,
              ml.state,
              ml.id
        ) AS foo
        GROUP BY
            start_date,
            date_to,
            partner_id,
            user_id,
            membership_id,
            company_id,
            membership_state,
            associate_member_id,
            membership_amount
        )""" % (self._table,))

```

## File: report\report_membership_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- REPORTING/MEMBERSHIP BY YEAR  -->
        <record model="ir.ui.view" id="view_report_membership_search">
            <field name="name">report.membership.search</field>
            <field name="model">report.membership</field>
            <field name="arch" type="xml">
                <search string="Membership">
                    <filter string="Forecast" name="forecast" context="{'waiting_invoiced_totpending_visible':0}" help="This will display waiting, invoiced and total pending columns"/>
                    <filter string="Revenue Done" name="Revenue" context="{'paid_old_totearned_visible':0}" help="This will display paid, old and total earned columns"/>
                    <separator/>
                    <filter name="filter_start_date" date="start_date"/>
                    <field name="partner_id"/>
                    <field name="membership_id"/>
                    <field name="user_id"/>
                    <group expand="1" string="Group By">
                        <filter string="Salesperson" name="salesman"
                            context="{'group_by':'user_id'}"/>
                        <filter string="Associated Partner" name="associate_member_id"
                            context="{'group_by':'associate_member_id'}"/>
                        <filter string="Membership Product" name="product"
                            context="{'group_by':'membership_id'}"/>
                        <filter string="Current Membership State" name="membership_state"
                            context="{'group_by':'membership_state'}"/>
                        <filter string="Company" name="company"
                            context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Month" name="start_date"
                            context="{'group_by':'start_date:month'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record model="ir.ui.view" id="view_report_membership_pivot">
            <field name="name">report.membership.pivot</field>
            <field name="model">report.membership</field>
            <field name="arch" type="xml">
                <pivot string="Membership" sample="1">
                    <field name="membership_id" type="row"/>
                    <field name="start_date" interval="month" type="col"/>
                    <field name="quantity" type="measure"/>
                    <field name="num_paid" type="measure"/>
                    <field name="num_invoiced" type="measure"/>
                    <field name="tot_earned" type="measure"/>
                </pivot>
            </field>
        </record>

        <record model="ir.ui.view" id="view_report_membership_graph1">
            <field name="name">report.membership.graph1</field>
            <field name="model">report.membership</field>
            <field name="arch" type="xml">
                <graph string="Membership" sample="1">
                    <field name="membership_id"/>
                    <field name="start_date" interval="month"/>
                    <field name="quantity" type="measure"/>
                </graph>
            </field>
        </record>

    <record id="report_membership_view_tree" model="ir.ui.view">
        <field name="name">report.membership.view.tree</field>
        <field name="model">report.membership</field>
        <field name="arch" type="xml">
            <tree string="Membership">
                <field name="partner_id"/>
                <field name="membership_id" optional="hide"/>
                <field name="start_date" optional="hide"/>
                <field name="date_to" optional="hide"/>
                <field name="user_id" optional="show" widget="many2one_avatar_user"/>
                <field name="company_id" optional="show" groups="base.group_multi_company"/>
                <field name="num_invoiced" optional="show" sum="Sum of # Invoiced"/>
                <field name="num_paid" optional="show" sum="Sum of # Paid"/>
                <field name="quantity" optional="show" sum="Sum of Quantity"/>
                <field name="tot_earned" optional="show" sum="Sum of Earned Amount"/>
                <field name="membership_state" optional="hide"/>
            </tree>
        </field>
    </record>

        <record model="ir.actions.act_window" id="action_report_membership_tree">
            <field name="name">Members Analysis</field>
            <field name="res_model">report.membership</field>
            <field name="view_mode">graph,pivot</field>
            <field name="search_view_id" ref="view_report_membership_search"/>
            <field name="context">{"search_default_start_date":1,"search_default_member":1, 'search_default_Revenue':1, 'search_default_this_month':1, 'search_default_salesman':1,'group_by_no_leaf':1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p>
            </field>
        </record>

        <menuitem name="Reporting" parent="menu_association"
            sequence="99"
            action="action_report_membership_tree"
            id="menu_report_membership"
            groups="base.group_partner_manager"/>

</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import report_membership

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_membership_membership_line_partner_manager,membership.membership_line partner_manager,model_membership_membership_line,base.group_partner_manager,1,1,1,1
access_membership_membership_line,membership.membership_line,model_membership_membership_line,,1,0,0,0
access_report_membership,report.membership,model_report_membership,base.group_partner_manager,1,0,0,0
access_membership_invoice,access.membership.invoice,model_membership_invoice,account.group_account_user,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M35.5 16.633c6.472 0 11.719 5.083 11.719 11.354 0 6.27-5.247 11.354-11.719 11.354-6.472 0-11.719-5.084-11.719-11.354S29.028 16.633 35.5 16.633zm13.424 23.688l-5.223-1.265c-5.488 3.824-12.14 2.971-16.402 0l-5.223 1.265c-3.13.759-5.326 3.483-5.326 6.61v2.628c0 1.881 1.574 3.406 3.516 3.406h30.468c1.942 0 3.516-1.525 3.516-3.406V46.93c0-3.126-2.196-5.85-5.326-6.609z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-.146-4-4.082V44.253l26.224-25.045 20.526 9.775-8.576 13.025L53 42.469l1.122 5.966L41.75 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><path fill="#FFF" fill-rule="nonzero" d="M35.5 14.633c6.472 0 11.719 5.083 11.719 11.354 0 6.27-5.247 11.354-11.719 11.354-6.472 0-11.719-5.084-11.719-11.354S29.028 14.633 35.5 14.633zm13.424 23.688l-5.223-1.265c-5.488 3.824-12.14 2.971-16.402 0l-5.223 1.265c-3.13.759-5.326 3.483-5.326 6.61v2.628c0 1.881 1.574 3.406 3.516 3.406h30.468c1.942 0 3.516-1.525 3.516-3.406V44.93c0-3.126-2.196-5.85-5.326-6.609z"/></g></g></svg>
```

## File: views\partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- MEMBERSHIP/CURRENT MEMBERS -->

        <record model="ir.ui.view" id="membership_members_tree">
            <field name="name">Members</field>
            <field name="model">res.partner</field>
            <field name="arch" type="xml">
                <tree string="Members">
                    <field name="name"/>
                    <field name="membership_state"/>
                    <field name="associate_member"/>
                    <field name="membership_start"/>
                    <field name="membership_stop"/>
                    <field name="user_id" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="view_res_partner_member_filter" model="ir.ui.view">
            <field name="name">res.partner.select</field>
            <field name="model">res.partner</field>
            <field name="priority">50</field>
            <field name="arch" type="xml">
                <search string="Membership Partners">
                    <field name="name"
                       filter_domain="['|', '|', ('name', 'ilike', self), ('parent_id', 'ilike', self), ('ref' , '=', self)]"/>
                    <field name="category_id"/>
                    <field name="membership_start" invisible="1"/>
                    <field name="membership_stop" string="End Membership Date"/>
                    <filter name="customer" string="Customers" domain="[('customer_rank' ,'>', 0)]"/>
                    <filter name="supplier" string="Vendors" domain="[('supplier_rank', '>', 0)]"/>
                    <separator/>
                    <filter name="all_members" string="Members" domain="[('membership_state', 'in', ['invoiced', 'paid', 'free'])]" help="Invoiced/Paid/Free"/>
                    <separator/>
                    <filter string="Start Date" name="start_date" date="membership_start"/>
                    <filter string="End Date" name="end_date" date="membership_stop"/>
                    <group expand="0" string="Group By" colspan="10" col="8">
                        <filter string="Salesperson" name="salesperson" domain="[]" context="{'group_by' : 'user_id'}"/>
                        <filter string="Associate Member" name = "associate" domain="[]" context="{'group_by': 'associate_member'}"/>
                        <filter string="Membership State" name="membership_state" domain="[]" context="{'group_by': 'membership_state'}"/>
                        <filter string="Start Date" name="start_month" help="Starting Date Of Membership" domain="[]" context="{'group_by': 'membership_start'}"/>
                        <filter string="End Date" name="end_month" help="Ending Date Of Membership" domain="[]" context="{'group_by': 'membership_stop'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_membership_members">
            <field name="name">Members</field>
            <field name="res_model">res.partner</field>
            <field name="view_mode">kanban,tree,form,activity</field>
            <field name="search_view_id" ref="view_res_partner_member_filter"/>
            <field name="context">{"search_default_all_members": 1, "default_free_member": True}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                  Add a new member
                </p><p>
                  Odoo helps you easily track all activities related to a member: 
                  Current Membership Status, Discussions and History of Membership, etc.
                </p>
            </field>
        </record>

        <record model="ir.actions.act_window.view" id="action_membership_members_view_tree">
            <field name="sequence" eval="2"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="membership_members_tree"/>
            <field name="act_window_id" ref="action_membership_members"/>
        </record>

        <record model="ir.actions.act_window.view" id="action_membership_members_view_form">
            <field name="sequence" eval="3"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="base.view_partner_form"/>
            <field name="act_window_id" ref="action_membership_members"/>
        </record>
         <record model="ir.actions.act_window.view" id="action_membership_members_view_kanban">
            <field name="sequence" eval="1"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="base.res_partner_kanban_view"/>
            <field name="act_window_id" ref="action_membership_members"/>
        </record>
        <menuitem name="Members" id="menu_membership" sequence="0" parent="menu_association" action="action_membership_members"/>

        <!-- PARTNERS -->

        <record model="ir.ui.view" id="view_partner_tree">
            <field name="name">res.partner.tree.form.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_tree"/>
            <field name="arch" type="xml">
                <field name="category_id" position="after">
                    <field name="membership_state" optional="hide"/>
                </field>
            </field>
        </record>

        <record model="ir.ui.view" id="view_partner_form">
            <field name="name">res.partner.form.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <notebook position="inside">
                    <page string="Membership" name="membership">
                        <group>
                            <group>
                                <field name="free_member"/>
                                <label for="membership_state"/>
                                <div>
                                    <field name="membership_state"/>
                                    <button name="%(action_membership_invoice_view)d" type="action" string="Buy Membership" 
                                        attrs="{'invisible':[('free_member','=',True)]}" class="oe_link"/>
                                </div>
                            </group>
                            <group>
                                <field name="associate_member" attrs="{'invisible':[('free_member','=',True)]}"/>
                                <field name="membership_start" attrs="{'invisible':[('membership_start','=',False)]}"/>
                                <field name="membership_stop" attrs="{'invisible':[('membership_stop','=',False)]}"/>
                                <field name="membership_cancel" attrs="{'invisible':[('membership_cancel','=',False)]}"/>
                            </group>
                        </group>
                        <field name="member_lines" nolabel="1" colspan="4" readonly="1">
                            <tree string="Memberships">
                                <field name="date"/>
                                <field name="membership_id"/>
                                <field name="member_price"/>
                                <field name="account_invoice_id"/>
                                <field name="state"/>
                            </tree>
                            <form string="Memberships">
                                <group col="2">
                                    <group>
                                        <field name="membership_id"/>
                                        <field name="date"/>
                                        <field name="state"/>
                                    </group>
                                    <group>
                                        <field name="member_price"/>
                                        <field name="account_invoice_id" context="{'form_view_ref': 'account.view_move_form'}"/>
                                    </group>
                                </group>
                            </form>
                        </field>

                    </page>
                </notebook>
            </field>
        </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- MEMBERSHIP -->

        <!-- MEMBERSHIP/MEMBERSHIP PRODUCTS -->

        <record model="ir.ui.view" id="membership_product_search_form_view">
            <field name="name">membership.product.search.form</field>
            <field name="model">product.template</field>
            <field name="priority">50</field>
            <field name="arch" type="xml">
                <search string="Membership Products">
                    <field name="name" string="Membership Product"/>
                    <filter string="Inactive" name="inactive" domain="[('active','=',False)]"/>
                    <field name="categ_id" operator="child_of"/>
                    <group  expand='0' string='Group by...'>
                        <filter string='Category' name="category" domain="[]" context="{'group_by' : 'categ_id'}"/>
                        <filter string='Start Date' name="from_month" domain="[]" context="{'group_by' : 'membership_date_from'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record model="ir.ui.view" id="membership_products_tree">
            <field name="name">Membership products</field>
            <field name="model">product.template</field>
            <field name="priority">50</field>
            <field name="arch" type="xml">
                <tree string="Membership products">
                    <field name="name"/>
                    <field name="membership_date_from"/>
                    <field name="membership_date_to"/>
                    <field name="list_price" string="Membership Fee"/>
                    <field name="categ_id" invisible="1"/>
                    <field name="uom_id" invisible="1"/>
                    <field name="type" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="membership_products_kanban" model="ir.ui.view">
            <field name="name">product.template.kanban</field>
            <field name="model">product.template</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="name"/>
                    <field name="membership_date_from"/>
                    <field name="membership_date_to"/>
                    <field name="list_price"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title"><span class="mt4"><field name="name"/></span></strong>
                                    </div>
                                    <span class="badge badge-pill"><i class="fa fa-money" role="img" aria-label="Price" title="Price"/> <field name="list_price"/></span>
                                </div>
                                <div class="o_kanban_record_body">
                                    <i class="fa fa-clock-o" role="img" aria-label="Period" title="Period"></i><strong> From: </strong><field name="membership_date_from"/><strong> To:</strong> <field name="membership_date_to"/>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record model="ir.ui.view" id="membership_products_form">
            <field name="name">Membership Products</field>
            <field name="model">product.template</field>
            <field name="priority">50</field>
            <field name="arch" type="xml">
                <form string="Membership products">
                    <sheet>
                        <label for="name" string="Product Name"/>
                        <h1>
                            <field name="name"/>
                        </h1>
                        <group>
                            <group name="product_details">
                                <field name="default_code"/>
                                <field name="categ_id"/>
                                <field name="membership" invisible="1"/>
                                <field name="company_id"
                                    groups="base.group_multi_company"
                                    options="{'no_create': True}"/>
                                <field name="active" widget="boolean_toggle"/>
                            </group>
                            <group name="membership_details">
                                <label for="membership_date_from" string="Membership Duration"/>
                                <div class="o_row">
                                    <field name="membership_date_from" required="1"/> -
                                    <field name="membership_date_to" required="1"/>
                                </div>
                                <field name="list_price" string="Membership Fee"/>
                                <field
                                    name="property_account_income_id"/>
                                <field name="taxes_id" widget="many2many_tags" string="Taxes"/>
                            </group>
                        </group>
                        <label for="description"/>
                        <field colspan="4" name="description" placeholder="Add a description..."/>
                        <label for="description_sale"/>
                        <field colspan="4" name="description_sale" placeholder="This note will be displayed on quotations..."/>
                    </sheet>
                 </form>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_membership_products">
            <field name="name">Membership Products</field>
            <field name="res_model">product.template</field>
            <field name="domain">[('membership','=',True), ('type', '=', 'service')]</field>
            <field name="context">{'membership':True, 'type':'service', 'default_membership': True, 'default_detailed_type': 'service'}</field>
            <field name="search_view_id" ref="membership_product_search_form_view"/>
        </record>

        <record model="ir.actions.act_window.view" id="action_membership_product_view_tree">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="membership_products_tree"/>
            <field name="act_window_id" ref="action_membership_products"/>
        </record>

        <record model="ir.actions.act_window.view" id="action_membership_product_view_form">
            <field name="sequence" eval="2"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="membership_products_form"/>
            <field name="act_window_id" ref="action_membership_products"/>
        </record>

        <record model="ir.actions.act_window.view" id="action_membership_product_view_kanban">
            <field name="sequence" eval="3"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="membership_products_kanban"/>
            <field name="act_window_id" ref="action_membership_products"/>
        </record>

        <menuitem name="Members" id="menu_association" sequence="15" web_icon="membership,static/description/icon.png"/>
        <menuitem name="Configuration" id="menu_marketing_config_association"
            parent="menu_association" sequence="100"/>
        <menuitem name="Membership Products" id="menu_membership_products" parent="menu_marketing_config_association"
            action="action_membership_products"/>

</odoo>

```

## File: wizard\membership_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MembershipInvoice(models.TransientModel):
    _name = "membership.invoice"
    _description = "Membership Invoice"

    product_id = fields.Many2one('product.product', string='Membership', required=True)
    member_price = fields.Float(string='Member Price', digits='Product Price', required=True)

    @api.onchange('product_id')
    def onchange_product(self):
        """This function returns value of  product's member price based on product id.
        """
        price_dict = self.product_id.price_compute('list_price')
        self.member_price = price_dict.get(self.product_id.id) or False

    def membership_invoice(self):
        invoice_list = self.env['res.partner'].browse(self._context.get('active_ids')).create_membership_invoice(self.product_id, self.member_price)

        search_view_ref = self.env.ref('account.view_account_invoice_filter', False)
        form_view_ref = self.env.ref('account.view_move_form', False)
        tree_view_ref = self.env.ref('account.view_move_tree', False)

        return  {
            'domain': [('id', 'in', invoice_list.ids)],
            'name': 'Membership Invoices',
            'res_model': 'account.move',
            'type': 'ir.actions.act_window',
            'views': [(tree_view_ref.id, 'tree'), (form_view_ref.id, 'form')],
            'search_view_id': search_view_ref and [search_view_ref.id],
        }

```

## File: wizard\membership_invoice_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="view_membership_invoice_view" model="ir.ui.view">
            <field name="name">membership.invoice.view.form</field>
            <field name="model">membership.invoice</field>
            <field name="arch" type="xml">
                <form string="Membership Invoice">
                    <group>
                        <field name="product_id" domain="[('membership','=',True)]" options="{'no_open': True, 'no_create': True}"/>
                        <field name="member_price"/>
                    </group>
                    <footer>
                        <button string="Invoice Membership" name="membership_invoice" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_membership_invoice_view" model="ir.actions.act_window">
            <field name="name">Join Membership</field>
            <field name="res_model">membership.invoice</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="view_membership_invoice_view"/>
            <field name="target">new</field>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import membership_invoice

```

