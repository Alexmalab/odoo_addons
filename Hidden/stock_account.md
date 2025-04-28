# Odoo Module: stock_account

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import wizard

from odoo import api, SUPERUSER_ID, _, tools

def _configure_journals(cr, registry):
    """Setting journal and property field (if needed)"""

    env = api.Environment(cr, SUPERUSER_ID, {})

    # if we already have a coa installed, create journal and set property field
    company_ids = env['res.company'].search([('chart_template_id', '!=', False)])
    todo_list = [
        'property_stock_account_input_categ_id',
        'property_stock_account_output_categ_id',
        'property_stock_valuation_account_id',
    ]
    # Property Stock Accounts
    categ_values = {category.id: False for category in env['product.category'].search([])}
    for company_id in company_ids:
        # Check if property exists for stock account journal exists
        field = env['ir.model.fields']._get("product.category", "property_stock_journal")
        properties = env['ir.property'].sudo().search([
            ('fields_id', '=', field.id),
            ('company_id', '=', company_id.id)])

        # If not, check if you can find a journal that is already there with the same code, otherwise create one
        if not properties:
            journal_id = env['account.journal'].search([
                ('code', '=', 'STJ'),
                ('company_id', '=', company_id.id),
                ('type', '=', 'general')], limit=1).id
            if not journal_id:
                journal_id = env['account.journal'].create({
                    'name': _('Inventory Valuation'),
                    'type': 'general',
                    'code': 'STJ',
                    'company_id': company_id.id,
                    'show_on_dashboard': False
                }).id
            env['ir.property']._set_default(
                'property_stock_journal',
                'product.category',
                journal_id,
                company_id,
            )

        for name in todo_list:
            account = getattr(company_id, name)
            if account:
                env['ir.property']._set_default(
                    name,
                    'product.category',
                    account,
                    company_id,
                )
            env['ir.property'].with_company(company_id.id)._set_multi(name, 'product.category', categ_values, True)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'WMS Accounting',
    'version': '1.1',
    'summary': 'Inventory, Logistic, Valuation, Accounting',
    'description': """
WMS Accounting module
======================
This module makes the link between the 'stock' and 'account' modules and allows you to create accounting entries to value your stock movements

Key Features
------------
* Stock Valuation (periodical or automatic)
* Invoice from Picking

Dashboard / Reports for Warehouse Management includes:
------------------------------------------------------
* Stock Inventory Value at given date (support dates in the past)
    """,
    'depends': ['stock', 'account'],
    'category': 'Hidden',
    'sequence': 16,
    'data': [
        'security/stock_account_security.xml',
        'security/ir.model.access.csv',
        'data/stock_account_data.xml',
        'views/stock_account_views.xml',
        'views/res_config_settings_views.xml',
        'data/product_data.xml',
        'views/report_invoice.xml',
        'views/stock_valuation_layer_views.xml',
        'views/stock_quant_views.xml',
        'views/product_views.xml',
        'wizard/stock_request_count.xml',
        'wizard/stock_valuation_layer_revaluation_views.xml',
        'wizard/stock_quantity_history.xml',
        'report/report_stock_forecasted.xml',
    ],
    'installable': True,
    'auto_install': True,
    'post_init_hook': '_configure_journals',
    'assets': {
        'web.assets_backend': [
            'stock_account/static/src/stock_account_forecasted/*',
            'stock_account/static/src/xml/inventory_report.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\product_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record forcecreate="True" id="property_stock_account_output_categ_id" model="ir.property">
            <field name="name">property_stock_account_output_categ_id</field>
            <field name="fields_id" search="[('model','=','product.category'),('name','=','property_stock_account_output_categ_id')]"/>
            <field eval="False" name="value"/>
            <field name="company_id" ref="base.main_company"/>
        </record>
        <record forcecreate="True" id="property_stock_account_input_categ_id" model="ir.property">
            <field name="name">property_stock_account_input_categ_id</field>
            <field name="fields_id" search="[('model','=','product.category'),('name','=','property_stock_account_input_categ_id')]"/>
            <field eval="False" name="value"/>
            <field name="company_id" ref="base.main_company"/>
        </record>
    </data>
</odoo>

```

## File: data\stock_account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record forcecreate="True" id="default_category_cost_method" model="ir.property">
            <field name="name">Cost Method Property</field>
            <field name="fields_id" ref="field_product_category__property_cost_method"/>
            <field name="value">standard</field>
            <field name="type">selection</field>
        </record>
        <record forcecreate="True" id="default_category_valuation" model="ir.property">
            <field name="name">Valuation Property</field>
            <field name="fields_id" ref="field_product_category__property_valuation"/>
            <field name="value">manual_periodic</field>
            <field name="type">selection</field>
        </record>
    </data>
</odoo>


```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _

import logging
_logger = logging.getLogger(__name__)


class AccountChartTemplate(models.Model):
    _inherit = "account.chart.template"

    @api.model
    def generate_journals(self, acc_template_ref, company, journals_dict=None):
        journal_to_add = (journals_dict or []) + [{'name': _('Inventory Valuation'), 'type': 'general', 'code': 'STJ', 'favorite': False, 'sequence': 8}]
        return super(AccountChartTemplate, self).generate_journals(acc_template_ref=acc_template_ref, company=company, journals_dict=journal_to_add)

    def generate_properties(self, acc_template_ref, company, property_list=None):
        res = super(AccountChartTemplate, self).generate_properties(acc_template_ref=acc_template_ref, company=company)
        PropertyObj = self.env['ir.property']  # Property Stock Journal
        value = self.env['account.journal'].search([('company_id', '=', company.id), ('code', '=', 'STJ'), ('type', '=', 'general')], limit=1)
        if value:
            PropertyObj._set_default("property_stock_journal", "product.category", value, company)

        todo_list = [  # Property Stock Accounts
            'property_stock_account_input_categ_id',
            'property_stock_account_output_categ_id',
            'property_stock_valuation_account_id',
        ]
        categ_values = {category.id: False for category in self.env['product.category'].search([])}
        for field in todo_list:
            account = self[field]
            value = acc_template_ref[account].id if account else False
            PropertyObj._set_default(field, "product.category", value, company)
            PropertyObj._set_multi(field, "product.category", categ_values, True)

        return res

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models, api
from odoo.tools import float_compare, float_is_zero


class AccountMove(models.Model):
    _inherit = 'account.move'

    stock_move_id = fields.Many2one('stock.move', string='Stock Move', index='btree_not_null')
    stock_valuation_layer_ids = fields.One2many('stock.valuation.layer', 'account_move_id', string='Stock Valuation Layer')

    def _compute_show_reset_to_draft_button(self):
        super()._compute_show_reset_to_draft_button()
        for move in self:
            if move.sudo().line_ids.stock_valuation_layer_ids:
                move.show_reset_to_draft_button = False

    # -------------------------------------------------------------------------
    # OVERRIDE METHODS
    # -------------------------------------------------------------------------

    def _get_lines_onchange_currency(self):
        # OVERRIDE
        return self.line_ids.filtered(lambda l: l.display_type != 'cogs')

    def copy_data(self, default=None):
        # OVERRIDE
        # Don't keep anglo-saxon lines when copying a journal entry.
        res = super().copy_data(default=default)

        if not self._context.get('move_reverse_cancel'):
            for copy_vals in res:
                if 'line_ids' in copy_vals:
                    copy_vals['line_ids'] = [line_vals for line_vals in copy_vals['line_ids']
                                             if line_vals[0] != 0 or line_vals[2].get('display_type') != 'cogs']

        return res

    def _post(self, soft=True):
        # OVERRIDE

        # Don't change anything on moves used to cancel another ones.
        if self._context.get('move_reverse_cancel'):
            return super()._post(soft)

        # Create additional COGS lines for customer invoices.
        self.env['account.move.line'].create(self._stock_account_prepare_anglo_saxon_out_lines_vals())

        # Post entries.
        posted = super()._post(soft)

        # Reconcile COGS lines in case of anglo-saxon accounting with perpetual valuation.
        if not self.env.context.get('skip_cogs_reconciliation'):
            posted._stock_account_anglo_saxon_reconcile_valuation()
        return posted

    def button_draft(self):
        res = super(AccountMove, self).button_draft()

        # Unlink the COGS lines generated during the 'post' method.
        self.mapped('line_ids').filtered(lambda line: line.display_type == 'cogs').unlink()
        return res

    def button_cancel(self):
        # OVERRIDE
        res = super(AccountMove, self).button_cancel()

        # Unlink the COGS lines generated during the 'post' method.
        # In most cases it shouldn't be necessary since they should be unlinked with 'button_draft'.
        # However, since it can be called in RPC, better be safe.
        self.mapped('line_ids').filtered(lambda line: line.display_type == 'cogs').unlink()
        return res

    # -------------------------------------------------------------------------
    # COGS METHODS
    # -------------------------------------------------------------------------

    def _stock_account_prepare_anglo_saxon_out_lines_vals(self):
        ''' Prepare values used to create the journal items (account.move.line) corresponding to the Cost of Good Sold
        lines (COGS) for customer invoices.

        Example:

        Buy a product having a cost of 9 being a storable product and having a perpetual valuation in FIFO.
        Sell this product at a price of 10. The customer invoice's journal entries looks like:

        Account                                     | Debit | Credit
        ---------------------------------------------------------------
        200000 Product Sales                        |       | 10.0
        ---------------------------------------------------------------
        101200 Account Receivable                   | 10.0  |
        ---------------------------------------------------------------

        This method computes values used to make two additional journal items:

        ---------------------------------------------------------------
        220000 Expenses                             | 9.0   |
        ---------------------------------------------------------------
        101130 Stock Interim Account (Delivered)    |       | 9.0
        ---------------------------------------------------------------

        Note: COGS are only generated for customer invoices except refund made to cancel an invoice.

        :return: A list of Python dictionary to be passed to env['account.move.line'].create.
        '''
        lines_vals_list = []
        price_unit_prec = self.env['decimal.precision'].precision_get('Product Price')
        for move in self:
            # Make the loop multi-company safe when accessing models like product.product
            move = move.with_company(move.company_id)

            if not move.is_sale_document(include_receipts=True) or not move.company_id.anglo_saxon_accounting:
                continue

            anglo_saxon_price_ctx = move._get_anglo_saxon_price_ctx()

            for line in move.invoice_line_ids:

                # Filter out lines being not eligible for COGS.
                if not line._eligible_for_cogs():
                    continue

                # Retrieve accounts needed to generate the COGS.
                accounts = line.product_id.product_tmpl_id.get_product_accounts(fiscal_pos=move.fiscal_position_id)
                debit_interim_account = accounts['stock_output']
                credit_expense_account = accounts['expense'] or move.journal_id.default_account_id
                if not debit_interim_account or not credit_expense_account:
                    continue

                # Compute accounting fields.
                sign = -1 if move.move_type == 'out_refund' else 1
                price_unit = line.with_context(anglo_saxon_price_ctx)._stock_account_get_anglo_saxon_price_unit()
                amount_currency = sign * line.quantity * price_unit

                if move.currency_id.is_zero(amount_currency) or float_is_zero(price_unit, precision_digits=price_unit_prec):
                    continue

                # Add interim account line.
                lines_vals_list.append({
                    'name': line.name[:64],
                    'move_id': move.id,
                    'partner_id': move.commercial_partner_id.id,
                    'product_id': line.product_id.id,
                    'product_uom_id': line.product_uom_id.id,
                    'quantity': line.quantity,
                    'price_unit': price_unit,
                    'amount_currency': -amount_currency,
                    'account_id': debit_interim_account.id,
                    'display_type': 'cogs',
                    'tax_ids': [],
                })

                # Add expense account line.
                lines_vals_list.append({
                    'name': line.name[:64],
                    'move_id': move.id,
                    'partner_id': move.commercial_partner_id.id,
                    'product_id': line.product_id.id,
                    'product_uom_id': line.product_uom_id.id,
                    'quantity': line.quantity,
                    'price_unit': -price_unit,
                    'amount_currency': amount_currency,
                    'account_id': credit_expense_account.id,
                    'analytic_distribution': line.analytic_distribution,
                    'display_type': 'cogs',
                    'tax_ids': [],
                })
        return lines_vals_list

    def _get_anglo_saxon_price_ctx(self):
        """ To be overriden in modules overriding _stock_account_get_anglo_saxon_price_unit
        to optimize computations that only depend on account.move and not account.move.line
        """
        return self.env.context

    def _stock_account_get_last_step_stock_moves(self):
        """ To be overridden for customer invoices and vendor bills in order to
        return the stock moves related to the invoices in self.
        """
        return self.env['stock.move']

    def _stock_account_anglo_saxon_reconcile_valuation(self, product=False):
        """ Reconciles the entries made in the interim accounts in anglosaxon accounting,
        reconciling stock valuation move lines with the invoice's.
        """
        for move in self:
            if not move.is_invoice():
                continue
            if not move.company_id.anglo_saxon_accounting:
                continue

            stock_moves = move._stock_account_get_last_step_stock_moves()
            # In case we return a return, we have to provide the related AMLs so all can be reconciled
            stock_moves |= stock_moves.origin_returned_move_id

            if not stock_moves:
                continue

            products = product or move.mapped('invoice_line_ids.product_id')
            for prod in products:
                if prod.valuation != 'real_time':
                    continue

                # We first get the invoices move lines (taking the invoice and the previous ones into account)...
                product_accounts = prod.product_tmpl_id._get_product_accounts()
                if move.is_sale_document():
                    product_interim_account = product_accounts['stock_output']
                else:
                    product_interim_account = product_accounts['stock_input']

                if product_interim_account.reconcile:
                    # Search for anglo-saxon lines linked to the product in the journal entry.
                    product_account_moves = move.line_ids.filtered(
                        lambda line: line.product_id == prod and line.account_id == product_interim_account and not line.reconciled)

                    # Search for anglo-saxon lines linked to the product in the stock moves.
                    product_stock_moves = stock_moves._get_all_related_sm(prod)
                    product_account_moves |= product_stock_moves._get_all_related_aml().filtered(
                        lambda line: line.account_id == product_interim_account and not line.reconciled and line.move_id.state == "posted"
                    )

                    correction_amls = product_account_moves.filtered(
                        lambda aml: aml.move_id.sudo().stock_valuation_layer_ids.stock_valuation_layer_id or (aml.display_type == 'cogs' and not aml.quantity)
                    )
                    invoice_aml = product_account_moves.filtered(lambda aml: aml not in correction_amls and aml.move_id == move)
                    stock_aml = product_account_moves - correction_amls - invoice_aml
                    # Reconcile:
                    # In case there is a move with correcting lines that has not been posted
                    # (e.g., it's dated for some time in the future) we should defer any
                    # reconciliation with exchange difference.
                    if correction_amls or 'draft' in move.line_ids.sudo().stock_valuation_layer_ids.account_move_id.mapped('state'):
                        if sum(correction_amls.mapped('balance')) > 0:
                            product_account_moves.with_context(no_exchange_difference=True).reconcile()
                        else:
                            (invoice_aml | correction_amls).with_context(no_exchange_difference=True).reconcile()
                            (invoice_aml.filtered(lambda aml: not aml.reconciled) | stock_aml).with_context(no_exchange_difference=True).reconcile()
                    else:
                        product_account_moves.reconcile()

    def _get_invoiced_lot_values(self):
        return []


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    stock_valuation_layer_ids = fields.One2many('stock.valuation.layer', 'account_move_line_id', string='Stock Valuation Layer')

    def _compute_account_id(self):
        super()._compute_account_id()
        input_lines = self.filtered(lambda line: (
            line._can_use_stock_accounts()
            and line.move_id.company_id.anglo_saxon_accounting
            and line.move_id.is_purchase_document()
        ))
        for line in input_lines:
            line = line.with_company(line.move_id.journal_id.company_id)
            fiscal_position = line.move_id.fiscal_position_id
            accounts = line.product_id.product_tmpl_id.get_product_accounts(fiscal_pos=fiscal_position)
            if accounts['stock_input']:
                line.account_id = accounts['stock_input']

    def _eligible_for_cogs(self):
        self.ensure_one()
        return self.product_id.type == 'product' and self.product_id.valuation == 'real_time'

    def _get_gross_unit_price(self):
        if float_is_zero(self.quantity, precision_rounding=self.product_uom_id.rounding):
            return self.price_unit

        price_unit = self.price_subtotal / self.quantity
        return -price_unit if self.move_id.move_type == 'in_refund' else price_unit

    def _get_stock_valuation_layers(self, move):
        valued_moves = self._get_valued_in_moves()
        if move.move_type == 'in_refund':
            valued_moves = valued_moves.filtered(lambda stock_move: stock_move._is_out())
        else:
            valued_moves = valued_moves.filtered(lambda stock_move: stock_move._is_in())
        return valued_moves.stock_valuation_layer_ids

    def _get_valued_in_moves(self):
        return self.env['stock.move']

    def _can_use_stock_accounts(self):
        return self.product_id.type == 'product' and self.product_id.categ_id.property_valuation == 'real_time'

    def _stock_account_get_anglo_saxon_price_unit(self):
        self.ensure_one()
        if not self.product_id:
            return self.price_unit
        original_line = self.move_id.reversed_entry_id.line_ids.filtered(
            lambda l: l.display_type == 'cogs' and l.product_id == self.product_id and
            l.product_uom_id == self.product_uom_id and l.price_unit >= 0)
        original_line = original_line and original_line[0]
        return original_line.price_unit if original_line \
            else self.product_id.with_company(self.company_id)._stock_account_get_anglo_saxon_price_unit(uom=self.product_uom_id)

    @api.onchange('product_id')
    def _inverse_product_id(self):
        super(AccountMoveLine, self.filtered(lambda l: l.display_type != 'cogs'))._inverse_product_id()

    def _deduce_anglo_saxon_unit_price(self, account_moves, stock_moves):
        self.ensure_one()

        move_is_downpayment = self.env.context.get("move_is_downpayment")
        if move_is_downpayment is None:
            move_is_downpayment = self.move_id.invoice_line_ids.filtered(
                lambda line: any(line.sale_line_ids.mapped("is_downpayment"))
            )

        is_line_reversing = False
        if self.move_id.move_type == 'out_refund' and not move_is_downpayment:
            is_line_reversing = True
        qty_to_invoice = self.product_uom_id._compute_quantity(self.quantity, self.product_id.uom_id)
        if self.move_id.move_type == 'out_refund' and move_is_downpayment:
            qty_to_invoice = -qty_to_invoice
        account_moves = account_moves.filtered(lambda m: m.state == 'posted' and bool(m.reversed_entry_id) == is_line_reversing)

        posted_cogs = self.env['account.move.line'].search([
            ('move_id', 'in', account_moves.ids),
            ('display_type', '=', 'cogs'),
            ('product_id', '=', self.product_id.id),
            ('balance', '>', 0),
        ])
        qty_invoiced = 0
        product_uom = self.product_id.uom_id
        for line in posted_cogs:
            if float_compare(line.quantity, 0, precision_rounding=product_uom.rounding) and line.move_id.move_type == 'out_refund' and any(line.move_id.invoice_line_ids.sale_line_ids.mapped('is_downpayment')):
                qty_invoiced += line.product_uom_id._compute_quantity(abs(line.quantity), line.product_id.uom_id)
            else:
                qty_invoiced += line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id)
        value_invoiced = sum(posted_cogs.mapped('balance'))
        reversal_moves = self.env['account.move']._search([('reversed_entry_id', 'in', posted_cogs.move_id.ids)])
        reversal_cogs = self.env['account.move.line'].search([
            ('move_id', 'in', reversal_moves),
            ('display_type', '=', 'cogs'),
            ('product_id', '=', self.product_id.id),
            ('balance', '>', 0)
        ])
        for line in reversal_cogs:
            if float_compare(line.quantity, 0, precision_rounding=product_uom.rounding) and line.move_id.move_type == 'out_refund' and any(line.move_id.invoice_line_ids.sale_line_ids.mapped('is_downpayment')):
                qty_invoiced -= line.product_uom_id._compute_quantity(abs(line.quantity), line.product_id.uom_id)
            else:
                qty_invoiced -= line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id)
        value_invoiced -= sum(reversal_cogs.mapped('balance'))

        product = self.product_id.with_company(self.company_id).with_context(value_invoiced=value_invoiced)
        average_price_unit = product._compute_average_price(qty_invoiced, qty_to_invoice, stock_moves, is_returned=is_line_reversing)
        price_unit = self.product_id.uom_id.with_company(self.company_id)._compute_price(average_price_unit, self.product_uom_id)

        return price_unit

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import clean_context, float_is_zero, float_repr, float_round, float_compare
from odoo.exceptions import ValidationError
from collections import defaultdict
from datetime import datetime


class ProductTemplate(models.Model):
    _name = 'product.template'
    _inherit = 'product.template'

    cost_method = fields.Selection(related="categ_id.property_cost_method", readonly=True)
    valuation = fields.Selection(related="categ_id.property_valuation", readonly=True)

    def write(self, vals):
        impacted_templates = {}
        move_vals_list = []
        Product = self.env['product.product']
        SVL = self.env['stock.valuation.layer']

        if 'categ_id' in vals:
            # When a change of category implies a change of cost method, we empty out and replenish
            # the stock.
            new_product_category = self.env['product.category'].browse(vals.get('categ_id'))

            for product_template in self:
                product_template = product_template.with_company(product_template.company_id)
                valuation_impacted = False
                if product_template.cost_method != new_product_category.property_cost_method:
                    valuation_impacted = True
                if product_template.valuation != new_product_category.property_valuation:
                    valuation_impacted = True
                if valuation_impacted is False:
                    continue

                # Empty out the stock with the current cost method.
                description = _("Due to a change of product category (from %s to %s), the costing method\
                                has changed for product template %s: from %s to %s.") %\
                    (product_template.categ_id.display_name, new_product_category.display_name,
                     product_template.display_name, product_template.cost_method, new_product_category.property_cost_method)
                out_svl_vals_list, products_orig_quantity_svl, products = Product\
                    ._svl_empty_stock(description, product_template=product_template)
                out_stock_valuation_layers = SVL.create(out_svl_vals_list)
                if product_template.valuation == 'real_time':
                    move_vals_list += Product._svl_empty_stock_am(out_stock_valuation_layers)
                impacted_templates[product_template] = (products, description, products_orig_quantity_svl)

        res = super(ProductTemplate, self).write(vals)

        for product_template, (products, description, products_orig_quantity_svl) in impacted_templates.items():
            # Replenish the stock with the new cost method.
            in_svl_vals_list = products._svl_replenish_stock(description, products_orig_quantity_svl)
            in_stock_valuation_layers = SVL.create(in_svl_vals_list)
            if product_template.valuation == 'real_time':
                move_vals_list += Product._svl_replenish_stock_am(in_stock_valuation_layers)

        # Check access right
        if move_vals_list and not self.env['stock.valuation.layer'].check_access_rights('read', raise_exception=False):
            raise UserError(_("The action leads to the creation of a journal entry, for which you don't have the access rights."))
        # Create the account moves.
        if move_vals_list:
            account_moves = self.env['account.move'].sudo().with_context(clean_context(self._context)).create(move_vals_list)
            account_moves._post()
        return res

    # -------------------------------------------------------------------------
    # Misc.
    # -------------------------------------------------------------------------
    def _get_product_accounts(self):
        """ Add the stock accounts related to product to the result of super()
        @return: dictionary which contains information regarding stock accounts and super (income+expense accounts)
        """
        accounts = super(ProductTemplate, self)._get_product_accounts()
        res = self._get_asset_accounts()
        accounts.update({
            'stock_input': res['stock_input'] or self.categ_id.property_stock_account_input_categ_id,
            'stock_output': res['stock_output'] or self.categ_id.property_stock_account_output_categ_id,
            'stock_valuation': self.categ_id.property_stock_valuation_account_id or False,
        })
        return accounts

    def get_product_accounts(self, fiscal_pos=None):
        """ Add the stock journal related to product to the result of super()
        @return: dictionary which contains all needed information regarding stock accounts and journal and super (income+expense accounts)
        """
        accounts = super(ProductTemplate, self).get_product_accounts(fiscal_pos=fiscal_pos)
        accounts.update({'stock_journal': self.categ_id.property_stock_journal or False})
        return accounts


class ProductProduct(models.Model):
    _inherit = 'product.product'

    value_svl = fields.Float(compute='_compute_value_svl', compute_sudo=True)
    quantity_svl = fields.Float(compute='_compute_value_svl', compute_sudo=True)
    avg_cost = fields.Monetary(string="Average Cost", compute='_compute_value_svl', compute_sudo=True, currency_field='company_currency_id')
    total_value = fields.Monetary(string="Total Value", compute='_compute_value_svl', compute_sudo=True, currency_field='company_currency_id')
    company_currency_id = fields.Many2one(
        'res.currency', 'Valuation Currency', compute='_compute_value_svl', compute_sudo=True,
        help="Technical field to correctly show the currently selected company's currency that corresponds "
             "to the totaled value of the product's valuation layers")
    stock_valuation_layer_ids = fields.One2many('stock.valuation.layer', 'product_id')
    valuation = fields.Selection(related="categ_id.property_valuation", readonly=True)
    cost_method = fields.Selection(related="categ_id.property_cost_method", readonly=True)

    def write(self, vals):
        if 'standard_price' in vals and not self.env.context.get('disable_auto_svl'):
            self.filtered(lambda p: p.cost_method != 'fifo')._change_standard_price(vals['standard_price'])
        return super(ProductProduct, self).write(vals)

    @api.depends('stock_valuation_layer_ids')
    @api.depends_context('to_date', 'company')
    def _compute_value_svl(self):
        """Compute totals of multiple svl related values"""
        company_id = self.env.company
        self.company_currency_id = company_id.currency_id
        domain = [
            ('product_id', 'in', self.ids),
            ('company_id', '=', company_id.id),
        ]
        if self.env.context.get('to_date'):
            to_date = fields.Datetime.to_datetime(self.env.context['to_date'])
            domain.append(('create_date', '<=', to_date))
        groups = self.env['stock.valuation.layer']._read_group(domain, ['value:sum', 'quantity:sum'], ['product_id'])
        products = self.browse()
        # Browse all products and compute products' quantities_dict in batch.
        self.env['product.product'].browse([group['product_id'][0] for group in groups]).sudo(False).mapped('qty_available')
        for group in groups:
            product = self.browse(group['product_id'][0])
            value_svl = company_id.currency_id.round(group['value'])
            avg_cost = value_svl / group['quantity'] if group['quantity'] else 0
            product.value_svl = value_svl
            product.quantity_svl = group['quantity']
            product.avg_cost = avg_cost
            product.total_value = avg_cost * product.sudo(False).qty_available
            products |= product
        remaining = (self - products)
        remaining.value_svl = 0
        remaining.quantity_svl = 0
        remaining.avg_cost = 0
        remaining.total_value = 0

    # -------------------------------------------------------------------------
    # Actions
    # -------------------------------------------------------------------------
    def action_revaluation(self):
        self.ensure_one()
        ctx = dict(self._context, default_product_id=self.id, default_company_id=self.env.company.id)
        return {
            'name': _("Product Revaluation"),
            'view_mode': 'form',
            'res_model': 'stock.valuation.layer.revaluation',
            'view_id': self.env.ref('stock_account.stock_valuation_layer_revaluation_form_view').id,
            'type': 'ir.actions.act_window',
            'context': ctx,
            'target': 'new'
        }

    # -------------------------------------------------------------------------
    # SVL creation helpers
    # -------------------------------------------------------------------------
    def _prepare_in_svl_vals(self, quantity, unit_cost):
        """Prepare the values for a stock valuation layer created by a receipt.

        :param quantity: the quantity to value, expressed in `self.uom_id`
        :param unit_cost: the unit cost to value `quantity`
        :return: values to use in a call to create
        :rtype: dict
        """
        self.ensure_one()
        company_id = self.env.context.get('force_company', self.env.company.id)
        company = self.env['res.company'].browse(company_id)
        value = company.currency_id.round(unit_cost * quantity)
        return {
            'product_id': self.id,
            'value': value,
            'unit_cost': unit_cost,
            'quantity': quantity,
            'remaining_qty': quantity,
            'remaining_value': value,
        }

    def _prepare_out_svl_vals(self, quantity, company):
        """Prepare the values for a stock valuation layer created by a delivery.

        :param quantity: the quantity to value, expressed in `self.uom_id`
        :return: values to use in a call to create
        :rtype: dict
        """
        self.ensure_one()
        company_id = self.env.context.get('force_company', self.env.company.id)
        company = self.env['res.company'].browse(company_id)
        currency = company.currency_id
        # Quantity is negative for out valuation layers.
        quantity = -1 * quantity
        vals = {
            'product_id': self.id,
            'value': currency.round(quantity * self.standard_price),
            'unit_cost': self.standard_price,
            'quantity': quantity,
        }
        fifo_vals = self._run_fifo(abs(quantity), company)
        vals['remaining_qty'] = fifo_vals.get('remaining_qty')
        # In case of AVCO, fix rounding issue of standard price when needed.
        if self.product_tmpl_id.cost_method == 'average' and not float_is_zero(self.quantity_svl, precision_rounding=self.uom_id.rounding):
            rounding_error = currency.round(
                (self.standard_price * self.quantity_svl - self.value_svl) * abs(quantity / self.quantity_svl)
            )
            if rounding_error:
                # If it is bigger than the (smallest number of the currency * quantity) / 2,
                # then it isn't a rounding error but a stock valuation error, we shouldn't fix it under the hood ...
                if abs(rounding_error) <= max((abs(quantity) * currency.rounding) / 2, currency.rounding):
                    vals['value'] += rounding_error
                    vals['rounding_adjustment'] = '\nRounding Adjustment: %s%s %s' % (
                        '+' if rounding_error > 0 else '',
                        float_repr(rounding_error, precision_digits=currency.decimal_places),
                        currency.symbol
                    )
        if self.product_tmpl_id.cost_method == 'fifo':
            vals.update(fifo_vals)
        return vals

    def _change_standard_price(self, new_price):
        """Helper to create the stock valuation layers and the account moves
        after an update of standard price.

        :param new_price: new standard price
        """
        # Handle stock valuation layers.

        if self.filtered(lambda p: p.valuation == 'real_time') and not self.env['stock.valuation.layer'].check_access_rights('read', raise_exception=False):
            raise UserError(_("You cannot update the cost of a product in automated valuation as it leads to the creation of a journal entry, for which you don't have the access rights."))

        svl_vals_list = []
        company_id = self.env.company
        price_unit_prec = self.env['decimal.precision'].precision_get('Product Price')
        rounded_new_price = float_round(new_price, precision_digits=price_unit_prec)
        for product in self:
            if product.cost_method not in ('standard', 'average'):
                continue
            quantity_svl = product.sudo().quantity_svl
            if float_compare(quantity_svl, 0.0, precision_rounding=product.uom_id.rounding) <= 0:
                continue
            value_svl = product.sudo().value_svl
            value = company_id.currency_id.round((rounded_new_price * quantity_svl) - value_svl)
            if company_id.currency_id.is_zero(value):
                continue

            svl_vals = {
                'company_id': company_id.id,
                'product_id': product.id,
                'description': _('Product value manually modified (from %s to %s)') % (product.standard_price, rounded_new_price),
                'value': value,
                'quantity': 0,
            }
            svl_vals_list.append(svl_vals)
        stock_valuation_layers = self.env['stock.valuation.layer'].sudo().create(svl_vals_list)

        # Handle account moves.
        product_accounts = {product.id: product.product_tmpl_id.get_product_accounts() for product in self}
        am_vals_list = []
        for stock_valuation_layer in stock_valuation_layers:
            product = stock_valuation_layer.product_id
            value = stock_valuation_layer.value

            if product.type != 'product' or product.valuation != 'real_time':
                continue

            # Sanity check.
            if not product_accounts[product.id].get('expense'):
                raise UserError(_('You must set a counterpart account on your product category.'))
            if not product_accounts[product.id].get('stock_valuation'):
                raise UserError(_('You don\'t have any stock valuation account defined on your product category. You must define one before processing this operation.'))

            if value < 0:
                debit_account_id = product_accounts[product.id]['expense'].id
                credit_account_id = product_accounts[product.id]['stock_valuation'].id
            else:
                debit_account_id = product_accounts[product.id]['stock_valuation'].id
                credit_account_id = product_accounts[product.id]['expense'].id

            move_vals = {
                'journal_id': product_accounts[product.id]['stock_journal'].id,
                'company_id': company_id.id,
                'ref': product.default_code,
                'stock_valuation_layer_ids': [(6, None, [stock_valuation_layer.id])],
                'move_type': 'entry',
                'line_ids': [(0, 0, {
                    'name': _(
                        '%(user)s changed cost from %(previous)s to %(new_price)s - %(product)s',
                        user=self.env.user.name,
                        previous=product.standard_price,
                        new_price=new_price,
                        product=product.display_name
                    ),
                    'account_id': debit_account_id,
                    'debit': abs(value),
                    'credit': 0,
                    'product_id': product.id,
                    'quantity': 0,
                }), (0, 0, {
                    'name': _(
                        '%(user)s changed cost from %(previous)s to %(new_price)s - %(product)s',
                        user=self.env.user.name,
                        previous=product.standard_price,
                        new_price=new_price,
                        product=product.display_name
                    ),
                    'account_id': credit_account_id,
                    'debit': 0,
                    'credit': abs(value),
                    'product_id': product.id,
                    'quantity': 0,
                })],
            }
            am_vals_list.append(move_vals)

        account_moves = self.env['account.move'].sudo().with_context(clean_context(self._context)).create(am_vals_list)
        if account_moves:
            account_moves._post()

    def _get_fifo_candidates_domain(self, company):
        return [
            ("product_id", "=", self.id),
            ("remaining_qty", ">", 0),
            ("company_id", "=", company.id),
        ]

    def _get_fifo_candidates(self, company):
        candidates_domain = self._get_fifo_candidates_domain(company)
        return self.env["stock.valuation.layer"].sudo().search(candidates_domain)

    def _get_qty_taken_on_candidate(self, qty_to_take_on_candidates, candidate):
        return min(qty_to_take_on_candidates, candidate.remaining_qty)

    def _run_fifo(self, quantity, company):
        self.ensure_one()

        # Find back incoming stock valuation layers (called candidates here) to value `quantity`.
        qty_to_take_on_candidates = quantity
        candidates = self._get_fifo_candidates(company)
        new_standard_price = 0
        tmp_value = 0  # to accumulate the value taken on the candidates
        for candidate in candidates:
            qty_taken_on_candidate = self._get_qty_taken_on_candidate(qty_to_take_on_candidates, candidate)

            candidate_unit_cost = candidate.remaining_value / candidate.remaining_qty
            new_standard_price = candidate_unit_cost
            value_taken_on_candidate = qty_taken_on_candidate * candidate_unit_cost
            value_taken_on_candidate = candidate.currency_id.round(value_taken_on_candidate)
            new_remaining_value = candidate.remaining_value - value_taken_on_candidate

            candidate_vals = {
                'remaining_qty': candidate.remaining_qty - qty_taken_on_candidate,
                'remaining_value': new_remaining_value,
            }

            candidate.write(candidate_vals)

            qty_to_take_on_candidates -= qty_taken_on_candidate
            tmp_value += value_taken_on_candidate

            if float_is_zero(qty_to_take_on_candidates, precision_rounding=self.uom_id.rounding):
                if float_is_zero(candidate.remaining_qty, precision_rounding=self.uom_id.rounding):
                    next_candidates = candidates.filtered(lambda svl: svl.remaining_qty > 0)
                    new_standard_price = next_candidates and next_candidates[0].unit_cost or new_standard_price
                break

        # Update the standard price with the price of the last used candidate, if any.
        if new_standard_price and self.cost_method == 'fifo':
            self.sudo().with_company(company.id).with_context(disable_auto_svl=True).standard_price = new_standard_price

        # If there's still quantity to value but we're out of candidates, we fall in the
        # negative stock use case. We chose to value the out move at the price of the
        # last out and a correction entry will be made once `_fifo_vacuum` is called.
        vals = {}
        if float_is_zero(qty_to_take_on_candidates, precision_rounding=self.uom_id.rounding):
            vals = {
                'value': -tmp_value,
                'unit_cost': tmp_value / quantity,
            }
        else:
            assert qty_to_take_on_candidates > 0
            last_fifo_price = new_standard_price or self.standard_price
            negative_stock_value = last_fifo_price * -qty_to_take_on_candidates
            tmp_value += abs(negative_stock_value)
            vals = {
                'remaining_qty': -qty_to_take_on_candidates,
                'value': -tmp_value,
                'unit_cost': last_fifo_price,
            }
        return vals

    def _run_fifo_vacuum(self, company=None):
        """Compensate layer valued at an estimated price with the price of future receipts
        if any. If the estimated price is equals to the real price, no layer is created but
        the original layer is marked as compensated.

        :param company: recordset of `res.company` to limit the execution of the vacuum
        """
        if company is None:
            company = self.env.company
        ValuationLayer = self.env['stock.valuation.layer'].sudo()
        svls_to_vacuum_by_product = defaultdict(lambda: ValuationLayer)
        res = ValuationLayer.read_group([
            ('product_id', 'in', self.ids),
            ('remaining_qty', '<', 0),
            ('stock_move_id', '!=', False),
            ('company_id', '=', company.id),
        ], ['ids:array_agg(id)', 'create_date:min'], ['product_id'], orderby='create_date, id')
        min_create_date = datetime.max
        for group in res:
            svls_to_vacuum_by_product[group['product_id'][0]] = ValuationLayer.browse(group['ids'])
            min_create_date = min(min_create_date, group['create_date'])
        all_candidates_by_product = defaultdict(lambda: ValuationLayer)
        res = ValuationLayer.read_group([
            ('product_id', 'in', self.ids),
            ('remaining_qty', '>', 0),
            ('company_id', '=', company.id),
            ('create_date', '>=', min_create_date),
        ], ['ids:array_agg(id)'], ['product_id'], orderby='id')
        for group in res:
            all_candidates_by_product[group['product_id'][0]] = ValuationLayer.browse(group['ids'])

        new_svl_vals_real_time = []
        new_svl_vals_manual = []
        real_time_svls_to_vacuum = ValuationLayer

        for product in self:
            all_candidates = all_candidates_by_product[product.id]
            current_real_time_svls = ValuationLayer
            for svl_to_vacuum in svls_to_vacuum_by_product[product.id]:
                # We don't use search to avoid executing _flush_search and to decrease interaction with DB
                candidates = all_candidates.filtered(
                    lambda r: r.create_date > svl_to_vacuum.create_date
                    or r.create_date == svl_to_vacuum.create_date
                    and r.id > svl_to_vacuum.id
                )
                if not candidates:
                    break
                qty_to_take_on_candidates = abs(svl_to_vacuum.remaining_qty)
                qty_taken_on_candidates = 0
                tmp_value = 0
                for candidate in candidates:
                    qty_taken_on_candidate = min(candidate.remaining_qty, qty_to_take_on_candidates)
                    qty_taken_on_candidates += qty_taken_on_candidate

                    candidate_unit_cost = candidate.remaining_value / candidate.remaining_qty
                    value_taken_on_candidate = qty_taken_on_candidate * candidate_unit_cost
                    value_taken_on_candidate = candidate.currency_id.round(value_taken_on_candidate)
                    new_remaining_value = candidate.remaining_value - value_taken_on_candidate

                    candidate_vals = {
                        'remaining_qty': candidate.remaining_qty - qty_taken_on_candidate,
                        'remaining_value': new_remaining_value
                    }
                    candidate.write(candidate_vals)
                    if not (candidate.remaining_qty > 0):
                        all_candidates -= candidate

                    qty_to_take_on_candidates -= qty_taken_on_candidate
                    tmp_value += value_taken_on_candidate
                    if float_is_zero(qty_to_take_on_candidates, precision_rounding=product.uom_id.rounding):
                        break

                # Get the estimated value we will correct.
                remaining_value_before_vacuum = svl_to_vacuum.unit_cost * qty_taken_on_candidates
                new_remaining_qty = svl_to_vacuum.remaining_qty + qty_taken_on_candidates
                corrected_value = remaining_value_before_vacuum - tmp_value
                svl_to_vacuum.write({
                    'remaining_qty': new_remaining_qty,
                })

                # Don't create a layer or an accounting entry if the corrected value is zero.
                if svl_to_vacuum.currency_id.is_zero(corrected_value):
                    continue

                corrected_value = svl_to_vacuum.currency_id.round(corrected_value)

                move = svl_to_vacuum.stock_move_id
                new_svl_vals = new_svl_vals_real_time if product.valuation == 'real_time' else new_svl_vals_manual
                new_svl_vals.append({
                    'product_id': product.id,
                    'value': corrected_value,
                    'unit_cost': 0,
                    'quantity': 0,
                    'remaining_qty': 0,
                    'stock_move_id': move.id,
                    'company_id': move.company_id.id,
                    'description': 'Revaluation of %s (negative inventory)' % (move.picking_id.name or move.name),
                    'stock_valuation_layer_id': svl_to_vacuum.id,
                })
                if product.valuation == 'real_time':
                    current_real_time_svls |= svl_to_vacuum
            real_time_svls_to_vacuum |= current_real_time_svls
        ValuationLayer.create(new_svl_vals_manual)
        vacuum_svls = ValuationLayer.create(new_svl_vals_real_time)

        # If some negative stock were fixed, we need to recompute the standard price.
        for product in self:
            product = product.with_company(company.id)
            # Only recompute if we fixed some negative stock
            if not svls_to_vacuum_by_product[product.id]:
                continue
            if product.cost_method == 'average' and not float_is_zero(product.quantity_svl,
                                                                      precision_rounding=product.uom_id.rounding):
                product.sudo().with_context(disable_auto_svl=True).write({'standard_price': product.value_svl / product.quantity_svl})

        vacuum_svls._validate_accounting_entries()
        self._create_fifo_vacuum_anglo_saxon_expense_entries(zip(vacuum_svls, real_time_svls_to_vacuum))

    @api.model
    def _create_fifo_vacuum_anglo_saxon_expense_entries(self, vacuum_pairs):
        """ Batch version of _create_fifo_vacuum_anglo_saxon_expense_entry
        """
        AccountMove = self.env['account.move'].sudo()
        account_move_vals = []
        vacuum_pairs_to_reconcile = []
        svls_accounts = {}
        for vacuum_svl, svl_to_vacuum in vacuum_pairs:
            if not vacuum_svl.company_id.anglo_saxon_accounting or not svl_to_vacuum.stock_move_id._is_out():
                continue
            account_move_lines = svl_to_vacuum.account_move_id.line_ids
            # Find related customer invoice where product is delivered while you don't have units in stock anymore
            reconciled_line_ids = list(set(account_move_lines._reconciled_lines()) - set(account_move_lines.ids))
            account_move = AccountMove.search([('line_ids', 'in', reconciled_line_ids)], limit=1)
            # If delivered quantity is not invoiced then no need to create this entry
            if not account_move:
                continue
            accounts = svl_to_vacuum.product_id.product_tmpl_id.get_product_accounts(fiscal_pos=account_move.fiscal_position_id)
            if not accounts.get('stock_output') or not accounts.get('expense'):
                continue
            svls_accounts[svl_to_vacuum.id] = accounts
            description = "Expenses %s" % (vacuum_svl.description)
            move_lines = vacuum_svl.stock_move_id._prepare_account_move_line(
            vacuum_svl.quantity, vacuum_svl.value * -1,
            accounts['stock_output'].id, accounts['expense'].id,
            vacuum_svl.id, description)
            account_move_vals.append({
                'journal_id': accounts['stock_journal'].id,
                'line_ids': move_lines,
                'date': self._context.get('force_period_date', fields.Date.context_today(self)),
                'ref': description,
                'stock_move_id': vacuum_svl.stock_move_id.id,
                'move_type': 'entry',
            })
            vacuum_pairs_to_reconcile.append((vacuum_svl, svl_to_vacuum))
        new_account_moves = AccountMove.with_context(clean_context(self._context)).create(account_move_vals)
        new_account_moves._post()
        for new_account_move, (vacuum_svl, svl_to_vacuum) in zip(new_account_moves, vacuum_pairs_to_reconcile):
            account = svls_accounts[svl_to_vacuum.id]['stock_output']
            to_reconcile_account_move_lines = vacuum_svl.account_move_id.line_ids.filtered(lambda l: not l.reconciled and l.account_id == account and l.account_id.reconcile)
            to_reconcile_account_move_lines += new_account_move.line_ids.filtered(lambda l: not l.reconciled and l.account_id == account and l.account_id.reconcile)
            to_reconcile_account_move_lines.reconcile()

    # TODO remove in master
    def _create_fifo_vacuum_anglo_saxon_expense_entry(self, vacuum_svl, svl_to_vacuum):
        """ When product is delivered and invoiced while you don't have units in stock anymore, there are chances of that
            product getting undervalued/overvalued. So, we should nevertheless take into account the fact that the product has
            already been delivered and invoiced to the customer by posting the value difference in the expense account also.
            Consider the below case where product is getting undervalued:

            You bought 8 units @ 10$ -> You have a stock valuation of 8 units, unit cost 10.
            Then you deliver 10 units of the product.
            You assumed the missing 2 should go out at a value of 10$ but you are not sure yet as it hasn't been bought in Odoo yet.
            Afterwards, you buy missing 2 units of the same product at 12$ instead of expected 10$.
            In case the product has been undervalued when delivered without stock, the vacuum entry is the following one (this entry already takes place):

            Account                         | Debit   | Credit
            ===================================================
            Stock Valuation                 | 0.00     | 4.00
            Stock Interim (Delivered)       | 4.00     | 0.00

            So, on delivering product with different price, We should create additional journal items like:
            Account                         | Debit    | Credit
            ===================================================
            Stock Interim (Delivered)       | 0.00     | 4.00
            Expenses Revaluation            | 4.00     | 0.00
        """
        if not vacuum_svl.company_id.anglo_saxon_accounting or not svl_to_vacuum.stock_move_id._is_out():
            return False
        AccountMove = self.env['account.move'].sudo()
        account_move_lines = svl_to_vacuum.account_move_id.line_ids
        # Find related customer invoice where product is delivered while you don't have units in stock anymore
        reconciled_line_ids = list(set(account_move_lines._reconciled_lines()) - set(account_move_lines.ids))
        account_move = AccountMove.search([('line_ids','in', reconciled_line_ids)], limit=1)
        # If delivered quantity is not invoiced then no need to create this entry
        if not account_move:
            return False
        accounts = svl_to_vacuum.product_id.product_tmpl_id.get_product_accounts(fiscal_pos=account_move.fiscal_position_id)
        if not accounts.get('stock_output') or not accounts.get('expense'):
            return False
        description = "Expenses %s" % (vacuum_svl.description)
        move_lines = vacuum_svl.stock_move_id._prepare_account_move_line(
            vacuum_svl.quantity, vacuum_svl.value * -1,
            accounts['stock_output'].id, accounts['expense'].id,
            vacuum_svl.id, description)
        new_account_move = AccountMove.sudo().create({
            'journal_id': accounts['stock_journal'].id,
            'line_ids': move_lines,
            'date': self._context.get('force_period_date', fields.Date.context_today(self)),
            'ref': description,
            'stock_move_id': vacuum_svl.stock_move_id.id,
            'move_type': 'entry',
        })
        new_account_move._post()
        to_reconcile_account_move_lines = vacuum_svl.account_move_id.line_ids.filtered(lambda l: not l.reconciled and l.account_id == accounts['stock_output'] and l.account_id.reconcile)
        to_reconcile_account_move_lines += new_account_move.line_ids.filtered(lambda l: not l.reconciled and l.account_id == accounts['stock_output'] and l.account_id.reconcile)
        return to_reconcile_account_move_lines.reconcile()

    @api.model
    def _svl_empty_stock(self, description, product_category=None, product_template=None):
        impacted_product_ids = []
        impacted_products = self.env['product.product']
        products_orig_quantity_svl = {}

        # get the impacted products
        domain = [('type', '=', 'product')]
        if product_category is not None:
            domain += [('categ_id', '=', product_category.id)]
        elif product_template is not None:
            domain += [('product_tmpl_id', '=', product_template.id)]
        else:
            raise ValueError()
        products = self.env['product.product'].search_read(domain, ['quantity_svl'])
        for product in products:
            impacted_product_ids.append(product['id'])
            products_orig_quantity_svl[product['id']] = product['quantity_svl']
        impacted_products |= self.env['product.product'].browse(impacted_product_ids)

        # empty out the stock for the impacted products
        empty_stock_svl_list = []
        for product in impacted_products:
            # FIXME sle: why not use products_orig_quantity_svl here?
            if float_is_zero(product.quantity_svl, precision_rounding=product.uom_id.rounding):
                # FIXME: create an empty layer to track the change?
                continue
            if float_compare(product.quantity_svl, 0, precision_rounding=product.uom_id.rounding) > 0:
                svsl_vals = product._prepare_out_svl_vals(product.quantity_svl, self.env.company)
            else:
                svsl_vals = product._prepare_in_svl_vals(abs(product.quantity_svl), product.value_svl / product.quantity_svl)
            svsl_vals['description'] = description + svsl_vals.pop('rounding_adjustment', '')
            svsl_vals['company_id'] = self.env.company.id
            empty_stock_svl_list.append(svsl_vals)
        return empty_stock_svl_list, products_orig_quantity_svl, impacted_products

    def _svl_replenish_stock(self, description, products_orig_quantity_svl):
        refill_stock_svl_list = []
        for product in self:
            quantity_svl = products_orig_quantity_svl[product.id]
            if quantity_svl:
                if float_compare(quantity_svl, 0, precision_rounding=product.uom_id.rounding) > 0:
                    svl_vals = product._prepare_in_svl_vals(quantity_svl, product.standard_price)
                else:
                    svl_vals = product._prepare_out_svl_vals(abs(quantity_svl), self.env.company)
                svl_vals['description'] = description
                svl_vals['company_id'] = self.env.company.id
                refill_stock_svl_list.append(svl_vals)
        return refill_stock_svl_list

    @api.model
    def _svl_empty_stock_am(self, stock_valuation_layers):
        move_vals_list = []
        product_accounts = {product.id: product.product_tmpl_id.get_product_accounts() for product in stock_valuation_layers.mapped('product_id')}
        for out_stock_valuation_layer in stock_valuation_layers:
            product = out_stock_valuation_layer.product_id
            stock_input_account = product_accounts[product.id].get('stock_input')
            if not stock_input_account:
                raise UserError(_('You don\'t have any stock input account defined on your product category. You must define one before processing this operation.'))
            if not product_accounts[product.id].get('stock_valuation'):
                raise UserError(_('You don\'t have any stock valuation account defined on your product category. You must define one before processing this operation.'))

            debit_account_id = stock_input_account.id
            credit_account_id = product_accounts[product.id]['stock_valuation'].id
            value = out_stock_valuation_layer.value
            move_vals = {
                'journal_id': product_accounts[product.id]['stock_journal'].id,
                'company_id': self.env.company.id,
                'ref': product.default_code,
                'stock_valuation_layer_ids': [(6, None, [out_stock_valuation_layer.id])],
                'line_ids': [(0, 0, {
                    'name': out_stock_valuation_layer.description,
                    'account_id': debit_account_id,
                    'debit': abs(value),
                    'credit': 0,
                    'product_id': product.id,
                }), (0, 0, {
                    'name': out_stock_valuation_layer.description,
                    'account_id': credit_account_id,
                    'debit': 0,
                    'credit': abs(value),
                    'product_id': product.id,
                })],
                'move_type': 'entry',
            }
            move_vals_list.append(move_vals)
        return move_vals_list

    def _svl_replenish_stock_am(self, stock_valuation_layers):
        move_vals_list = []
        product_accounts = {product.id: product.product_tmpl_id.get_product_accounts() for product in stock_valuation_layers.mapped('product_id')}
        for out_stock_valuation_layer in stock_valuation_layers:
            product = out_stock_valuation_layer.product_id
            if not product_accounts[product.id].get('stock_input'):
                raise UserError(_('You don\'t have any input valuation account defined on your product category. You must define one before processing this operation.'))
            if not product_accounts[product.id].get('stock_valuation'):
                raise UserError(_('You don\'t have any stock valuation account defined on your product category. You must define one before processing this operation.'))
            if not product_accounts[product.id].get('stock_output'):
                raise UserError(
                    _('You don\'t have any output valuation account defined on your product '
                      'category. You must define one before processing this operation.')
                )

            precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            if float_compare(out_stock_valuation_layer.quantity, 0, precision_digits=precision) == 1:
                debit_account_id = product_accounts[product.id]['stock_valuation'].id
                credit_account_id = product_accounts[product.id]['stock_input'].id
            else:
                debit_account_id = product_accounts[product.id]['stock_output'].id
                credit_account_id = product_accounts[product.id]['stock_valuation'].id

            value = out_stock_valuation_layer.value
            move_vals = {
                'journal_id': product_accounts[product.id]['stock_journal'].id,
                'company_id': self.env.company.id,
                'ref': product.default_code,
                'stock_valuation_layer_ids': [(6, None, [out_stock_valuation_layer.id])],
                'line_ids': [(0, 0, {
                    'name': out_stock_valuation_layer.description,
                    'account_id': debit_account_id,
                    'debit': abs(value),
                    'credit': 0,
                    'product_id': product.id,
                }), (0, 0, {
                    'name': out_stock_valuation_layer.description,
                    'account_id': credit_account_id,
                    'debit': 0,
                    'credit': abs(value),
                    'product_id': product.id,
                })],
                'move_type': 'entry',
            }
            move_vals_list.append(move_vals)
        return move_vals_list

    # -------------------------------------------------------------------------
    # Anglo saxon helpers
    # -------------------------------------------------------------------------
    def _stock_account_get_anglo_saxon_price_unit(self, uom=False):
        price = self.standard_price
        if not self or not uom or self.uom_id.id == uom.id:
            return price or 0.0
        return self.uom_id._compute_price(price, uom)

    def _compute_average_price(self, qty_invoiced, qty_to_invoice, stock_moves, is_returned=False):
        """Go over the valuation layers of `stock_moves` to value `qty_to_invoice` while taking
        care of ignoring `qty_invoiced`. If `qty_to_invoice` is greater than what's possible to
        value with the valuation layers, use the product's standard price.

        :param qty_invoiced: quantity already invoiced
        :param qty_to_invoice: quantity to invoice
        :param stock_moves: recordset of `stock.move`
        :param is_returned: if True, consider the incoming moves
        :returns: the anglo saxon price unit
        :rtype: float
        """
        self.ensure_one()
        if not qty_to_invoice:
            return 0

        candidates = stock_moves\
            .sudo()\
            .filtered(lambda m: is_returned == bool(m.origin_returned_move_id and sum(m.stock_valuation_layer_ids.mapped('quantity')) >= 0))\
            .mapped('stock_valuation_layer_ids')

        if self.env.context.get('candidates_prefetch_ids'):
            candidates = candidates.with_prefetch(self.env.context.get('candidates_prefetch_ids'))

        if len(candidates) > 1:
            candidates = candidates.sorted(lambda svl: (svl.create_date, svl.id))

        value_invoiced = self.env.context.get('value_invoiced', 0)
        if 'value_invoiced' in self.env.context:
            qty_valued, valuation = candidates._consume_all(qty_invoiced, value_invoiced, qty_to_invoice)
        else:
            qty_valued, valuation = candidates._consume_specific_qty(qty_invoiced, qty_to_invoice)

        # If there's still quantity to invoice but we're out of candidates, we chose the standard
        # price to estimate the anglo saxon price unit.
        missing = qty_to_invoice - qty_valued
        for sml in stock_moves.move_line_ids:
            if not sml._should_exclude_for_valuation():
                continue
            missing -= sml.product_uom_id._compute_quantity(sml.qty_done, self.uom_id, rounding_method='HALF-UP')
        if float_compare(missing, 0, precision_rounding=self.uom_id.rounding) > 0:
            valuation += self.standard_price * missing

        return valuation / qty_to_invoice


class ProductCategory(models.Model):
    _inherit = 'product.category'

    property_valuation = fields.Selection([
        ('manual_periodic', 'Manual'),
        ('real_time', 'Automated')], string='Inventory Valuation',
        company_dependent=True, copy=True, required=True,
        help="""Manual: The accounting entries to value the inventory are not posted automatically.
        Automated: An accounting entry is automatically created to value the inventory when a product enters or leaves the company.
        """)
    property_cost_method = fields.Selection([
        ('standard', 'Standard Price'),
        ('fifo', 'First In First Out (FIFO)'),
        ('average', 'Average Cost (AVCO)')], string="Costing Method",
        company_dependent=True, copy=True, required=True,
        help="""Standard Price: The products are valued at their standard cost defined on the product.
        Average Cost (AVCO): The products are valued at weighted average cost.
        First In First Out (FIFO): The products are valued supposing those that enter the company first will also leave it first.
        """)
    property_stock_journal = fields.Many2one(
        'account.journal', 'Stock Journal', company_dependent=True,
        domain="[('company_id', '=', allowed_company_ids[0])]", check_company=True,
        help="When doing automated inventory valuation, this is the Accounting Journal in which entries will be automatically posted when stock moves are processed.")
    property_stock_account_input_categ_id = fields.Many2one(
        'account.account', 'Stock Input Account', company_dependent=True,
        domain="[('company_id', '=', allowed_company_ids[0]), ('deprecated', '=', False)]", check_company=True,
        help="""Counterpart journal items for all incoming stock moves will be posted in this account, unless there is a specific valuation account
                set on the source location. This is the default value for all products in this category. It can also directly be set on each product.""")
    property_stock_account_output_categ_id = fields.Many2one(
        'account.account', 'Stock Output Account', company_dependent=True,
        domain="[('company_id', '=', allowed_company_ids[0]), ('deprecated', '=', False)]", check_company=True,
        help="""When doing automated inventory valuation, counterpart journal items for all outgoing stock moves will be posted in this account,
                unless there is a specific valuation account set on the destination location. This is the default value for all products in this category.
                It can also directly be set on each product.""")
    property_stock_valuation_account_id = fields.Many2one(
        'account.account', 'Stock Valuation Account', company_dependent=True,
        domain="[('company_id', '=', allowed_company_ids[0]), ('deprecated', '=', False)]", check_company=True,
        help="""When automated inventory valuation is enabled on a product, this account will hold the current value of the products.""",)

    @api.constrains('property_stock_valuation_account_id', 'property_stock_account_output_categ_id', 'property_stock_account_input_categ_id')
    def _check_valuation_accouts(self):
        # Prevent to set the valuation account as the input or output account.
        for category in self:
            valuation_account = category.property_stock_valuation_account_id
            input_and_output_accounts = category.property_stock_account_input_categ_id | category.property_stock_account_output_categ_id
            if valuation_account and valuation_account in input_and_output_accounts:
                raise ValidationError(_('The Stock Input and/or Output accounts cannot be the same as the Stock Valuation account.'))

    @api.onchange('property_cost_method')
    def onchange_property_cost(self):
        if not self._origin:
            # don't display the warning when creating a product category
            return
        return {
            'warning': {
                'title': _("Warning"),
                'message': _("Changing your cost method is an important change that will impact your inventory valuation. Are you sure you want to make that change?"),
            }
        }

    def write(self, vals):
        impacted_categories = {}
        move_vals_list = []
        Product = self.env['product.product']
        SVL = self.env['stock.valuation.layer']

        if 'property_cost_method' in vals or 'property_valuation' in vals:
            # When the cost method or the valuation are changed on a product category, we empty
            # out and replenish the stock for each impacted products.
            new_cost_method = vals.get('property_cost_method')
            new_valuation = vals.get('property_valuation')

            for product_category in self:
                property_stock_fields = ['property_stock_account_input_categ_id', 'property_stock_account_output_categ_id', 'property_stock_valuation_account_id']
                if 'property_valuation' in vals and vals['property_valuation'] == 'manual_periodic' and product_category.property_valuation != 'manual_periodic':
                    for stock_property in property_stock_fields:
                        vals[stock_property] = False
                elif 'property_valuation' in vals and vals['property_valuation'] == 'real_time' and product_category.property_valuation != 'real_time':
                    company_id = self.env.company
                    for stock_property in property_stock_fields:
                        vals[stock_property] = vals.get(stock_property, False) or company_id[stock_property]
                elif product_category.property_valuation == 'manual_periodic':
                    for stock_property in property_stock_fields:
                        if stock_property in vals:
                            vals.pop(stock_property)
                else:
                    for stock_property in property_stock_fields:
                        if stock_property in vals and vals[stock_property] is False:
                            vals.pop(stock_property)
                valuation_impacted = False
                if new_cost_method and new_cost_method != product_category.property_cost_method:
                    valuation_impacted = True
                if new_valuation and new_valuation != product_category.property_valuation:
                    valuation_impacted = True
                if valuation_impacted is False:
                    continue

                # Empty out the stock with the current cost method.
                if new_cost_method:
                    description = _("Costing method change for product category %s: from %s to %s.") \
                        % (product_category.display_name, product_category.property_cost_method, new_cost_method)
                else:
                    description = _("Valuation method change for product category %s: from %s to %s.") \
                        % (product_category.display_name, product_category.property_valuation, new_valuation)
                out_svl_vals_list, products_orig_quantity_svl, products = Product\
                    ._svl_empty_stock(description, product_category=product_category)
                out_stock_valuation_layers = SVL.sudo().create(out_svl_vals_list)
                if product_category.property_valuation == 'real_time':
                    move_vals_list += Product._svl_empty_stock_am(out_stock_valuation_layers)
                for svl in out_stock_valuation_layers:
                    svl.product_id.with_context(disable_auto_svl=True).standard_price = svl.unit_cost
                impacted_categories[product_category] = (products, description, products_orig_quantity_svl)

        res = super(ProductCategory, self).write(vals)

        for product_category, (products, description, products_orig_quantity_svl) in impacted_categories.items():
            # Replenish the stock with the new cost method.
            in_svl_vals_list = products._svl_replenish_stock(description, products_orig_quantity_svl)
            in_stock_valuation_layers = SVL.sudo().create(in_svl_vals_list)
            if product_category.property_valuation == 'real_time':
                move_vals_list += Product._svl_replenish_stock_am(in_stock_valuation_layers)

        # Check access right
        if move_vals_list and not self.env['stock.valuation.layer'].check_access_rights('read', raise_exception=False):
            raise UserError(_("The action leads to the creation of a journal entry, for which you don't have the access rights."))
        # Create the account moves.
        if move_vals_list:
            account_moves = self.env['account.move'].sudo().with_context(clean_context(self._context)).create(move_vals_list)
            account_moves._post()
        return res


    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if 'property_valuation' not in vals or vals['property_valuation'] == 'manual_periodic':
                vals['property_stock_account_input_categ_id'] = False
                vals['property_stock_account_output_categ_id'] = False
                vals['property_stock_valuation_account_id'] = False
            if 'property_valuation' in vals and vals['property_valuation'] == 'real_time':
                company_id = self.env.company
                vals['property_stock_account_input_categ_id'] = vals.get('property_stock_account_input_categ_id', False) or company_id.property_stock_account_input_categ_id
                vals['property_stock_account_output_categ_id'] = vals.get('property_stock_account_output_categ_id', False) or company_id.property_stock_account_output_categ_id
                vals['property_stock_valuation_account_id'] = vals.get('property_stock_valuation_account_id', False) or company_id.property_stock_valuation_account_id

        return super().create(vals_list)

    @api.onchange('property_valuation')
    def onchange_property_valuation(self):
        # Remove or set the account stock properties if necessary
        if self.property_valuation == 'manual_periodic':
            self.property_stock_account_input_categ_id = False
            self.property_stock_account_output_categ_id = False
            self.property_stock_valuation_account_id = False
        if self.property_valuation == 'real_time':
            company_id = self.env.company
            self.property_stock_account_input_categ_id = company_id.property_stock_account_input_categ_id
            self.property_stock_account_output_categ_id = company_id.property_stock_account_output_categ_id
            self.property_stock_valuation_account_id = company_id.property_stock_valuation_account_id

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_stock_landed_costs = fields.Boolean("Landed Costs",
        help="Affect landed costs on reception operations and split them among products to update their cost price.")
    group_lot_on_invoice = fields.Boolean("Display Lots & Serial Numbers on Invoices",
                                          implied_group='stock_account.group_lot_on_invoice')

```

## File: models\stock_location.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockLocation(models.Model):
    _inherit = "stock.location"

    valuation_in_account_id = fields.Many2one(
        'account.account', 'Stock Valuation Account (Incoming)',
        domain=[('account_type', 'not in', ('asset_receivable', 'liability_payable', 'asset_cash', 'liability_credit_card')), ('deprecated', '=', False)],
        help="Used for real-time inventory valuation. When set on a virtual location (non internal type), "
             "this account will be used to hold the value of products being moved from an internal location "
             "into this location, instead of the generic Stock Output Account set on the product. "
             "This has no effect for internal locations.")
    valuation_out_account_id = fields.Many2one(
        'account.account', 'Stock Valuation Account (Outgoing)',
        domain=[('account_type', 'not in', ('asset_receivable', 'liability_payable', 'asset_cash', 'liability_credit_card')), ('deprecated', '=', False)],
        help="Used for real-time inventory valuation. When set on a virtual location (non internal type), "
             "this account will be used to hold the value of products being moved out of this location "
             "and into an internal location, instead of the generic Stock Output Account set on the product. "
             "This has no effect for internal locations.")

    def _should_be_valued(self):
        """ This method returns a boolean reflecting whether the products stored in `self` should
        be considered when valuating the stock of a company.
        """
        self.ensure_one()
        if self.usage == 'internal' or (self.usage == 'transit' and self.company_id):
            return True
        return False

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_compare, float_is_zero, OrderedSet

import logging
_logger = logging.getLogger(__name__)


class StockMove(models.Model):
    _inherit = "stock.move"

    to_refund = fields.Boolean(string="Update quantities on SO/PO", copy=False,
                               help='Trigger a decrease of the delivered/received quantity in the associated Sale Order/Purchase Order')
    account_move_ids = fields.One2many('account.move', 'stock_move_id')
    stock_valuation_layer_ids = fields.One2many('stock.valuation.layer', 'stock_move_id')
    analytic_account_line_id = fields.Many2one(
        'account.analytic.line', copy=False, index='btree_not_null')

    def _filter_anglo_saxon_moves(self, product):
        return self.filtered(lambda m: m.product_id.id == product.id)

    def action_get_account_moves(self):
        self.ensure_one()
        action_data = self.env['ir.actions.act_window']._for_xml_id('account.action_move_journal_line')
        action_data['domain'] = [('id', 'in', self.account_move_ids.ids)]
        return action_data

    def _action_cancel(self):
        self.analytic_account_line_id.unlink()
        return super()._action_cancel()

    def _should_force_price_unit(self):
        self.ensure_one()
        return False

    def _get_price_unit(self):
        """ Returns the unit price to value this stock move """
        self.ensure_one()
        price_unit = self.price_unit
        precision = self.env['decimal.precision'].precision_get('Product Price')
        # If the move is a return, use the original move's price unit.
        if self.origin_returned_move_id and self.origin_returned_move_id.sudo().stock_valuation_layer_ids:
            layers = self.origin_returned_move_id.sudo().stock_valuation_layer_ids
            # dropshipping create additional positive svl to make sure there is no impact on the stock valuation
            # We need to remove them from the computation of the price unit.
            if self.origin_returned_move_id._is_dropshipped() or self.origin_returned_move_id._is_dropshipped_returned():
                layers = layers.filtered(lambda l: float_compare(l.value, 0, precision_rounding=l.product_id.uom_id.rounding) <= 0)
            layers |= layers.stock_valuation_layer_ids
            quantity = sum(layers.mapped("quantity"))
            return sum(layers.mapped("value")) / quantity if not float_is_zero(quantity, precision_rounding=layers.uom_id.rounding) else 0
        return price_unit if not float_is_zero(price_unit, precision) or self._should_force_price_unit() else self.product_id.standard_price

    @api.model
    def _get_valued_types(self):
        """Returns a list of `valued_type` as strings. During `action_done`, we'll call
        `_is_[valued_type]'. If the result of this method is truthy, we'll consider the move to be
        valued.

        :returns: a list of `valued_type`
        :rtype: list
        """
        return ['in', 'out', 'dropshipped', 'dropshipped_returned']

    def _get_in_move_lines(self):
        """ Returns the `stock.move.line` records of `self` considered as incoming. It is done thanks
        to the `_should_be_valued` method of their source and destionation location as well as their
        owner.

        :returns: a subset of `self` containing the incoming records
        :rtype: recordset
        """
        self.ensure_one()
        res = OrderedSet()
        for move_line in self.move_line_ids:
            if move_line._should_exclude_for_valuation():
                continue
            if not move_line.location_id._should_be_valued() and move_line.location_dest_id._should_be_valued():
                res.add(move_line.id)
        return self.env['stock.move.line'].browse(res)

    def _is_in(self):
        """Check if the move should be considered as entering the company so that the cost method
        will be able to apply the correct logic.

        :returns: True if the move is entering the company else False
        :rtype: bool
        """
        self.ensure_one()
        if self._get_in_move_lines() and not self._is_dropshipped_returned():
            return True
        return False

    def _get_out_move_lines(self):
        """ Returns the `stock.move.line` records of `self` considered as outgoing. It is done thanks
        to the `_should_be_valued` method of their source and destionation location as well as their
        owner.

        :returns: a subset of `self` containing the outgoing records
        :rtype: recordset
        """
        res = self.env['stock.move.line']
        for move_line in self.move_line_ids:
            if move_line._should_exclude_for_valuation():
                continue
            if move_line.location_id._should_be_valued() and not move_line.location_dest_id._should_be_valued():
                res |= move_line
        return res

    def _is_out(self):
        """Check if the move should be considered as leaving the company so that the cost method
        will be able to apply the correct logic.

        :returns: True if the move is leaving the company else False
        :rtype: bool
        """
        self.ensure_one()
        if self._get_out_move_lines() and not self._is_dropshipped():
            return True
        return False

    def _is_dropshipped(self):
        """Check if the move should be considered as a dropshipping move so that the cost method
        will be able to apply the correct logic.

        :returns: True if the move is a dropshipping one else False
        :rtype: bool
        """
        self.ensure_one()
        return self.location_id.usage == 'supplier' and self.location_dest_id.usage == 'customer'

    def _is_dropshipped_returned(self):
        """Check if the move should be considered as a returned dropshipping move so that the cost
        method will be able to apply the correct logic.

        :returns: True if the move is a returned dropshipping one else False
        :rtype: bool
        """
        self.ensure_one()
        return self.location_id.usage == 'customer' and self.location_dest_id.usage == 'supplier'

    def _prepare_common_svl_vals(self):
        """When a `stock.valuation.layer` is created from a `stock.move`, we can prepare a dict of
        common vals.

        :returns: the common values when creating a `stock.valuation.layer` from a `stock.move`
        :rtype: dict
        """
        self.ensure_one()
        return {
            'stock_move_id': self.id,
            'company_id': self.company_id.id,
            'product_id': self.product_id.id,
            'description': self.reference and '%s - %s' % (self.reference, self.product_id.name) or self.product_id.name,
        }

    def _create_in_svl(self, forced_quantity=None):
        """Create a `stock.valuation.layer` from `self`.

        :param forced_quantity: under some circunstances, the quantity to value is different than
            the initial demand of the move (Default value = None)
        """
        svl_vals_list = self._get_in_svl_vals(forced_quantity)
        return self.env['stock.valuation.layer'].sudo().create(svl_vals_list)

    def _create_out_svl(self, forced_quantity=None):
        """Create a `stock.valuation.layer` from `self`.

        :param forced_quantity: under some circunstances, the quantity to value is different than
            the initial demand of the move (Default value = None)
        """
        svl_vals_list = []
        for move in self:
            move = move.with_company(move.company_id)
            valued_move_lines = move._get_out_move_lines()
            valued_quantity = 0
            for valued_move_line in valued_move_lines:
                valued_quantity += valued_move_line.product_uom_id._compute_quantity(
                    valued_move_line.qty_done, move.product_id.uom_id, rounding_method="HALF-UP"
                )
            if float_is_zero(forced_quantity or valued_quantity, precision_rounding=move.product_id.uom_id.rounding):
                continue
            svl_vals = move.product_id._prepare_out_svl_vals(forced_quantity or valued_quantity, move.company_id)
            svl_vals.update(move._prepare_common_svl_vals())
            if forced_quantity:
                svl_vals['description'] = 'Correction of %s (modification of past move)' % (move.picking_id.name or move.name)
            svl_vals['description'] += svl_vals.pop('rounding_adjustment', '')
            svl_vals_list.append(svl_vals)
        return self.env['stock.valuation.layer'].sudo().create(svl_vals_list)

    def _create_dropshipped_svl(self, forced_quantity=None):
        """Create a `stock.valuation.layer` from `self`.

        :param forced_quantity: under some circunstances, the quantity to value is different than
            the initial demand of the move (Default value = None)
        """
        svl_vals_list = []
        for move in self:
            move = move.with_company(move.company_id)
            valued_move_lines = move.move_line_ids
            valued_quantity = 0
            for valued_move_line in valued_move_lines:
                valued_quantity += valued_move_line.product_uom_id._compute_quantity(
                    valued_move_line.qty_done, move.product_id.uom_id, rounding_method="HALF-UP"
                )
            quantity = forced_quantity or valued_quantity

            unit_cost = move._get_price_unit()
            if move.product_id.cost_method == 'standard':
                unit_cost = move.product_id.standard_price

            common_vals = dict(move._prepare_common_svl_vals(), remaining_qty=0)

            # create the in if it does not come from a valued location (eg subcontract -> customer)
            if not move.location_id._should_be_valued():
                in_vals = {
                    'unit_cost': unit_cost,
                    'value': unit_cost * quantity,
                    'quantity': quantity,
                }
                in_vals.update(common_vals)
                svl_vals_list.append(in_vals)

            # create the out if it does not go to a valued location (eg customer -> subcontract)
            if not move.location_dest_id._should_be_valued():
                out_vals = {
                    'unit_cost': unit_cost,
                    'value': unit_cost * quantity * -1,
                    'quantity': quantity * -1,
                }
                out_vals.update(common_vals)
                svl_vals_list.append(out_vals)

        return self.env['stock.valuation.layer'].sudo().create(svl_vals_list)

    def _create_dropshipped_returned_svl(self, forced_quantity=None):
        """Create a `stock.valuation.layer` from `self`.

        :param forced_quantity: under some circunstances, the quantity to value is different than
            the initial demand of the move (Default value = None)
        """
        return self._create_dropshipped_svl(forced_quantity=forced_quantity)

    def _action_done(self, cancel_backorder=False):
        # Init a dict that will group the moves by valuation type, according to `move._is_valued_type`.
        valued_moves = {valued_type: self.env['stock.move'] for valued_type in self._get_valued_types()}
        for move in self:
            if float_is_zero(move.quantity_done, precision_rounding=move.product_uom.rounding):
                continue
            for valued_type in self._get_valued_types():
                if getattr(move, '_is_%s' % valued_type)():
                    valued_moves[valued_type] |= move

        # AVCO application
        valued_moves['in'].product_price_update_before_done()

        res = super(StockMove, self)._action_done(cancel_backorder=cancel_backorder)

        # '_action_done' might have deleted some exploded stock moves
        valued_moves = {value_type: moves.exists() for value_type, moves in valued_moves.items()}

        # '_action_done' might have created an extra move to be valued
        for move in res - self:
            for valued_type in self._get_valued_types():
                if getattr(move, '_is_%s' % valued_type)():
                    valued_moves[valued_type] |= move

        stock_valuation_layers = self.env['stock.valuation.layer'].sudo()
        # Create the valuation layers in batch by calling `moves._create_valued_type_svl`.
        for valued_type in self._get_valued_types():
            todo_valued_moves = valued_moves[valued_type]
            if todo_valued_moves:
                todo_valued_moves._sanity_check_for_valuation()
                stock_valuation_layers |= getattr(todo_valued_moves, '_create_%s_svl' % valued_type)()

        stock_valuation_layers._validate_accounting_entries()
        stock_valuation_layers._validate_analytic_accounting_entries()

        stock_valuation_layers._check_company()

        # For every in move, run the vacuum for the linked product.
        products_to_vacuum = valued_moves['in'].mapped('product_id')
        company = valued_moves['in'].mapped('company_id') and valued_moves['in'].mapped('company_id')[0] or self.env.company
        products_to_vacuum._run_fifo_vacuum(company)

        return res

    def _sanity_check_for_valuation(self):
        for move in self:
            # Apply restrictions on the stock move to be able to make
            # consistent accounting entries.
            if move._is_in() and move._is_out():
                raise UserError(_("The move lines are not in a consistent state: some are entering and other are leaving the company."))
            company_src = move.mapped('move_line_ids.location_id.company_id')
            company_dst = move.mapped('move_line_ids.location_dest_id.company_id')
            try:
                if company_src:
                    company_src.ensure_one()
                if company_dst:
                    company_dst.ensure_one()
            except ValueError:
                raise UserError(_("The move lines are not in a consistent states: they do not share the same origin or destination company."))
            if company_src and company_dst and company_src.id != company_dst.id:
                raise UserError(_("The move lines are not in a consistent states: they are doing an intercompany in a single step while they should go through the intercompany transit location."))

    def product_price_update_before_done(self, forced_qty=None):
        tmpl_dict = defaultdict(lambda: 0.0)
        # adapt standard price on incomming moves if the product cost_method is 'average'
        std_price_update = {}
        for move in self.filtered(lambda move: move._is_in() and move.with_company(move.company_id).product_id.cost_method == 'average'):
            product_tot_qty_available = move.product_id.sudo().with_company(move.company_id).quantity_svl + tmpl_dict[move.product_id.id]
            rounding = move.product_id.uom_id.rounding

            valued_move_lines = move._get_in_move_lines()
            qty_done = 0
            for valued_move_line in valued_move_lines:
                qty_done += valued_move_line.product_uom_id._compute_quantity(
                    valued_move_line.qty_done, move.product_id.uom_id, rounding_method="HALF-UP"
                )

            qty = forced_qty or qty_done
            if float_is_zero(product_tot_qty_available, precision_rounding=rounding):
                new_std_price = move._get_price_unit()
            elif float_is_zero(product_tot_qty_available + move.product_qty, precision_rounding=rounding) or \
                    float_is_zero(product_tot_qty_available + qty, precision_rounding=rounding):
                new_std_price = move._get_price_unit()
            else:
                # Get the standard price
                amount_unit = std_price_update.get((move.company_id.id, move.product_id.id)) or move.product_id.with_company(move.company_id).standard_price
                new_std_price = ((amount_unit * product_tot_qty_available) + (move._get_price_unit() * qty)) / (product_tot_qty_available + qty)

            tmpl_dict[move.product_id.id] += qty_done
            # Write the standard price, as SUPERUSER_ID because a warehouse manager may not have the right to write on products
            move.product_id.with_company(move.company_id.id).with_context(disable_auto_svl=True).sudo().write({'standard_price': new_std_price})
            std_price_update[move.company_id.id, move.product_id.id] = new_std_price

        # adapt standard price on incomming moves if the product cost_method is 'fifo'
        for move in self.filtered(lambda move:
                                  move.with_company(move.company_id).product_id.cost_method == 'fifo'
                                  and float_is_zero(move.product_id.sudo().quantity_svl, precision_rounding=move.product_id.uom_id.rounding)):
            move.product_id.with_company(move.company_id.id).sudo().write({'standard_price': move._get_price_unit()})

    def _get_accounting_data_for_valuation(self):
        """ Return the accounts and journal to use to post Journal Entries for
        the real-time valuation of the quant. """
        self.ensure_one()
        self = self.with_company(self.company_id)
        accounts_data = self.product_id.product_tmpl_id.get_product_accounts()

        acc_src = self._get_src_account(accounts_data)
        acc_dest = self._get_dest_account(accounts_data)

        acc_valuation = accounts_data.get('stock_valuation', False)
        if acc_valuation:
            acc_valuation = acc_valuation.id
        if not accounts_data.get('stock_journal', False):
            raise UserError(_('You don\'t have any stock journal defined on your product category, check if you have installed a chart of accounts.'))
        if not acc_src:
            raise UserError(_('Cannot find a stock input account for the product %s. You must define one on the product category, or on the location, before processing this operation.') % (self.product_id.display_name))
        if not acc_dest:
            raise UserError(_('Cannot find a stock output account for the product %s. You must define one on the product category, or on the location, before processing this operation.') % (self.product_id.display_name))
        if not acc_valuation:
            raise UserError(_('You don\'t have any stock valuation account defined on your product category. You must define one before processing this operation.'))
        journal_id = accounts_data['stock_journal'].id
        return journal_id, acc_src, acc_dest, acc_valuation

    def _get_in_svl_vals(self, forced_quantity):
        svl_vals_list = []
        for move in self:
            move = move.with_company(move.company_id)
            valued_move_lines = move._get_in_move_lines()
            valued_quantity = 0
            for valued_move_line in valued_move_lines:
                valued_quantity += valued_move_line.product_uom_id._compute_quantity(
                    valued_move_line.qty_done, move.product_id.uom_id, rounding_method="HALF-UP"
                )
            unit_cost = move.product_id.standard_price
            if move.product_id.cost_method != 'standard':
                unit_cost = abs(move._get_price_unit())  # May be negative (i.e. decrease an out move).
            svl_vals = move.product_id._prepare_in_svl_vals(forced_quantity or valued_quantity, unit_cost)
            svl_vals.update(move._prepare_common_svl_vals())
            if forced_quantity:
                svl_vals['description'] = 'Correction of %s (modification of past move)' % (move.picking_id.name or move.name)
            svl_vals_list.append(svl_vals)
        return svl_vals_list

    def _get_src_account(self, accounts_data):
        return self.location_id.valuation_out_account_id.id or accounts_data['stock_input'].id

    def _get_dest_account(self, accounts_data):
        if not self.location_dest_id.usage in ('production', 'inventory'):
            return accounts_data['stock_output'].id
        else:
            return self.location_dest_id.valuation_in_account_id.id or accounts_data['stock_output'].id

    def _prepare_account_move_line(self, qty, cost, credit_account_id, debit_account_id, svl_id, description):
        """
        Generate the account.move.line values to post to track the stock valuation difference due to the
        processing of the given quant.
        """
        self.ensure_one()

        # the standard_price of the product may be in another decimal precision, or not compatible with the coinage of
        # the company currency... so we need to use round() before creating the accounting entries.
        debit_value = self.company_id.currency_id.round(cost)
        credit_value = debit_value

        valuation_partner_id = self._get_partner_id_for_valuation_lines()
        res = [(0, 0, line_vals) for line_vals in self._generate_valuation_lines_data(valuation_partner_id, qty, debit_value, credit_value, debit_account_id, credit_account_id, svl_id, description).values()]

        return res

    def _prepare_analytic_lines(self):
        self.ensure_one()
        if not self._get_analytic_account():
            return False

        if self.state in ['cancel', 'draft']:
            return False

        amount, unit_amount = 0, 0
        if self.state != 'done':
            unit_amount = self.product_uom._compute_quantity(
                self.quantity_done, self.product_id.uom_id)
            # Falsy in FIFO but since it's an estimation we don't require exact correct cost. Otherwise
            # we would have to recompute all the analytic estimation at each out.
            amount = - unit_amount * self.product_id.standard_price
        elif self.product_id.valuation == 'real_time' and not self._ignore_automatic_valuation():
            accounts_data = self.product_id.product_tmpl_id.get_product_accounts()
            account_valuation = accounts_data.get('stock_valuation', False)
            analytic_line_vals = self.stock_valuation_layer_ids.account_move_id.line_ids.filtered(
                lambda l: l.account_id == account_valuation)._prepare_analytic_lines()
            amount = - sum(vals['amount'] for vals in analytic_line_vals)
            unit_amount = - sum(vals['unit_amount'] for vals in analytic_line_vals)
        elif sum(self.stock_valuation_layer_ids.mapped('quantity')):
            amount = sum(self.stock_valuation_layer_ids.mapped('value'))
            unit_amount = - sum(self.stock_valuation_layer_ids.mapped('quantity'))
        if self.analytic_account_line_id:
            if amount == 0 and unit_amount == 0:
                self.analytic_account_line_id.unlink()
                return False
            self.analytic_account_line_id.unit_amount = unit_amount
            self.analytic_account_line_id.amount = amount
            return False
        elif amount:
            return self._generate_analytic_lines_data(
                unit_amount, amount)

    def _ignore_automatic_valuation(self):
        return False

    def _generate_analytic_lines_data(self, unit_amount, amount):
        self.ensure_one()
        account_id = self._get_analytic_account()
        return {
            'name': self.name,
            'amount': amount,
            'account_id': account_id.id,
            'unit_amount': unit_amount,
            'product_id': self.product_id.id,
            'product_uom_id': self.product_id.uom_id.id,
            'company_id': self.company_id.id,
            'ref': self._description,
            'category': 'other',
        }

    def _generate_valuation_lines_data(self, partner_id, qty, debit_value, credit_value, debit_account_id, credit_account_id, svl_id, description):
        # This method returns a dictionary to provide an easy extension hook to modify the valuation lines (see purchase for an example)
        self.ensure_one()

        line_vals = {
            'name': description,
            'product_id': self.product_id.id,
            'quantity': qty,
            'product_uom_id': self.product_id.uom_id.id,
            'ref': description,
            'partner_id': partner_id,
        }

        svl = self.env['stock.valuation.layer'].browse(svl_id)
        if svl.account_move_line_id.analytic_distribution:
            line_vals['analytic_distribution'] = svl.account_move_line_id.analytic_distribution

        rslt = {
            'credit_line_vals': {
                **line_vals,
                'balance': -credit_value,
                'account_id': credit_account_id,
            },
            'debit_line_vals': {
                **line_vals,
                'balance': debit_value,
                'account_id': debit_account_id,
            },
        }

        if credit_value != debit_value:
            # for supplier returns of product in average costing method, in anglo saxon mode
            diff_amount = debit_value - credit_value
            price_diff_account = self.env.context.get('price_diff_account')
            if not price_diff_account:
                raise UserError(_('Configuration error. Please configure the price difference account on the product or its category to process this operation.'))

            rslt['price_diff_line_vals'] = {
                'name': self.name,
                'product_id': self.product_id.id,
                'quantity': qty,
                'product_uom_id': self.product_id.uom_id.id,
                'balance': -diff_amount,
                'ref': description,
                'partner_id': partner_id,
                'account_id': price_diff_account.id,
            }
        return rslt

    def _get_partner_id_for_valuation_lines(self):
        return (self.picking_id.partner_id and self.env['res.partner']._find_accounting_partner(self.picking_id.partner_id).id) or False

    def _prepare_move_split_vals(self, uom_qty):
        vals = super(StockMove, self)._prepare_move_split_vals(uom_qty)
        vals['to_refund'] = self.to_refund
        return vals

    def _prepare_account_move_vals(self, credit_account_id, debit_account_id, journal_id, qty, description, svl_id, cost):
        self.ensure_one()
        valuation_partner_id = self._get_partner_id_for_valuation_lines()
        move_ids = self._prepare_account_move_line(qty, cost, credit_account_id, debit_account_id, svl_id, description)
        svl = self.env['stock.valuation.layer'].browse(svl_id)
        if self.env.context.get('force_period_date'):
            date = self.env.context.get('force_period_date')
        elif svl.account_move_line_id:
            date = svl.account_move_line_id.date
        else:
            date = fields.Date.context_today(self)
        return {
            'journal_id': journal_id,
            'line_ids': move_ids,
            'partner_id': valuation_partner_id,
            'date': date,
            'ref': description,
            'stock_move_id': self.id,
            'stock_valuation_layer_ids': [(6, None, [svl_id])],
            'move_type': 'entry',
            'is_storno': self.env.context.get('is_returned') and self.env.company.account_storno,
        }

    def _account_analytic_entry_move(self):
        analytic_lines_vals = []
        moves_to_link = []
        for move in self:
            analytic_line_vals = move._prepare_analytic_lines()
            if not analytic_line_vals:
                continue
            moves_to_link.append(move.id)
            analytic_lines_vals.append(analytic_line_vals)
        analytic_lines = self.env['account.analytic.line'].sudo().create(analytic_lines_vals)
        for move_id, analytic_line in zip(moves_to_link, analytic_lines):
            self.env['stock.move'].browse(
                move_id).analytic_account_line_id = analytic_line

    def _should_exclude_for_valuation(self):
        """Determines if this move should be excluded from valuation based on its partner.
        :return: True if the move's restrict_partner_id is different from the company's partner (indicating
                it should be excluded from valuation), False otherwise.
        """
        self.ensure_one()
        return self.restrict_partner_id and self.restrict_partner_id != self.company_id.partner_id

    def _account_entry_move(self, qty, description, svl_id, cost):
        """ Accounting Valuation Entries """
        self.ensure_one()
        am_vals = []
        if self.product_id.type != 'product':
            # no stock valuation for consumable products
            return am_vals
        if self._should_exclude_for_valuation():
            return am_vals

        company_from = self._is_out() and self.mapped('move_line_ids.location_id.company_id') or False
        company_to = self._is_in() and self.mapped('move_line_ids.location_dest_id.company_id') or False

        journal_id, acc_src, acc_dest, acc_valuation = self._get_accounting_data_for_valuation()
        # Create Journal Entry for products arriving in the company; in case of routes making the link between several
        # warehouse of the same company, the transit location belongs to this company, so we don't need to create accounting entries
        if self._is_in():
            if self._is_returned(valued_type='in'):
                am_vals.append(self.with_company(company_to).with_context(is_returned=True)._prepare_account_move_vals(acc_dest, acc_valuation, journal_id, qty, description, svl_id, cost))
            else:
                am_vals.append(self.with_company(company_to)._prepare_account_move_vals(acc_src, acc_valuation, journal_id, qty, description, svl_id, cost))

        # Create Journal Entry for products leaving the company
        if self._is_out():
            cost = -1 * cost
            if self._is_returned(valued_type='out'):
                am_vals.append(self.with_company(company_from).with_context(is_returned=True)._prepare_account_move_vals(acc_valuation, acc_src, journal_id, qty, description, svl_id, cost))
            else:
                am_vals.append(self.with_company(company_from)._prepare_account_move_vals(acc_valuation, acc_dest, journal_id, qty, description, svl_id, cost))

        if self.company_id.anglo_saxon_accounting:
            # Creates an account entry from stock_input to stock_output on a dropship move. https://github.com/odoo/odoo/issues/12687
            anglosaxon_am_vals = self._prepare_anglosaxon_account_move_vals(acc_src, acc_dest, acc_valuation, journal_id, qty, description, svl_id, cost)
            if anglosaxon_am_vals:
                am_vals.append(anglosaxon_am_vals)

        return am_vals

    def _prepare_anglosaxon_account_move_vals(self, acc_src, acc_dest, acc_valuation, journal_id, qty, description, svl_id, cost):
        anglosaxon_am_vals = {}
        if self._is_dropshipped():
            if cost > 0:
                anglosaxon_am_vals = self.with_company(self.company_id)._prepare_account_move_vals(acc_src, acc_valuation, journal_id, qty, description, svl_id, cost)
            else:
                cost = -1 * cost
                anglosaxon_am_vals = self.with_company(self.company_id)._prepare_account_move_vals(acc_valuation, acc_dest, journal_id, qty, description, svl_id, cost)
        elif self._is_dropshipped_returned():
            if cost > 0 and self.location_dest_id._should_be_valued():
                anglosaxon_am_vals = self.with_company(self.company_id).with_context(is_returned=True)._prepare_account_move_vals(acc_valuation, acc_src, journal_id, qty, description, svl_id, cost)
            elif cost > 0:
                anglosaxon_am_vals = self.with_company(self.company_id).with_context(is_returned=True)._prepare_account_move_vals(acc_dest, acc_valuation, journal_id, qty, description, svl_id, cost)
            else:
                cost = -1 * cost
                anglosaxon_am_vals = self.with_company(self.company_id).with_context(is_returned=True)._prepare_account_move_vals(acc_valuation, acc_src, journal_id, qty, description, svl_id, cost)
        return anglosaxon_am_vals

    def _get_analytic_account(self):
        return False

    def _get_related_invoices(self):  # To be overridden in purchase and sale_stock
        """ This method is overrided in both purchase and sale_stock modules to adapt
        to the way they mix stock moves with invoices.
        """
        return self.env['account.move']

    def _is_returned(self, valued_type):
        self.ensure_one()
        if valued_type == 'in':
            return self.location_id and self.location_id.usage == 'customer'   # goods returned from customer
        if valued_type == 'out':
            return self.location_dest_id and self.location_dest_id.usage == 'supplier'   # goods returned to supplier

    def _get_all_related_aml(self):
        return self.account_move_ids.line_ids

    def _get_all_related_sm(self, product):
        return self.filtered(lambda m: m.product_id == product)

```

## File: models\stock_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.tools import float_compare, float_is_zero


class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    # -------------------------------------------------------------------------
    # CRUD
    # -------------------------------------------------------------------------
    @api.model_create_multi
    def create(self, vals_list):
        analytic_move_to_recompute = set()
        move_lines = super(StockMoveLine, self).create(vals_list)
        for move_line in move_lines:
            move = move_line.move_id
            analytic_move_to_recompute.add(move.id)
            if move_line.state != 'done':
                continue
            product_uom = move_line.product_id.uom_id
            diff = move_line.product_uom_id._compute_quantity(move_line.qty_done, product_uom)
            if float_is_zero(diff, precision_rounding=product_uom.rounding):
                continue
            self._create_correction_svl(move, diff)
        if analytic_move_to_recompute:
            self.env['stock.move'].browse(
                analytic_move_to_recompute)._account_analytic_entry_move()
        return move_lines

    def write(self, vals):
        analytic_move_to_recompute = set()
        if 'qty_done' in vals or 'move_id' in vals:
            for move_line in self:
                move_id = vals.get('move_id') if vals.get('move_id') else move_line.move_id.id
                analytic_move_to_recompute.add(move_id)
        if 'qty_done' in vals:
            for move_line in self:
                if move_line.state != 'done':
                    continue
                product_uom = move_line.product_id.uom_id
                diff = move_line.product_uom_id._compute_quantity(vals['qty_done'] - move_line.qty_done, product_uom, rounding_method='HALF-UP')
                if float_is_zero(diff, precision_rounding=product_uom.rounding):
                    continue
                self._create_correction_svl(move_line.move_id, diff)
        res = super(StockMoveLine, self).write(vals)
        if analytic_move_to_recompute:
            self.env['stock.move'].browse(analytic_move_to_recompute)._account_analytic_entry_move()
        return res

    # -------------------------------------------------------------------------
    # SVL creation helpers
    # -------------------------------------------------------------------------
    @api.model
    def _create_correction_svl(self, move, diff):
        stock_valuation_layers = self.env['stock.valuation.layer']
        if move._is_in() and diff > 0 or move._is_out() and diff < 0:
            move.product_price_update_before_done(forced_qty=diff)
            stock_valuation_layers |= move._create_in_svl(forced_quantity=abs(diff))
            if move.product_id.cost_method in ('average', 'fifo'):
                move.product_id._run_fifo_vacuum(move.company_id)
        elif move._is_in() and diff < 0 or move._is_out() and diff > 0:
            stock_valuation_layers |= move._create_out_svl(forced_quantity=abs(diff))
        elif move._is_dropshipped() and diff > 0 or move._is_dropshipped_returned() and diff < 0:
            stock_valuation_layers |= move._create_dropshipped_svl(forced_quantity=abs(diff))
        elif move._is_dropshipped() and diff < 0 or move._is_dropshipped_returned() and diff > 0:
            stock_valuation_layers |= move._create_dropshipped_returned_svl(forced_quantity=abs(diff))

        stock_valuation_layers._validate_accounting_entries()

    @api.model
    def _should_exclude_for_valuation(self):
        """
        Determines if this move line should be excluded from valuation based on its ownership.
        :return: True if the move line's owner is different from the company's partner (indicating
                it should be excluded from valuation), False otherwise.
        """
        self.ensure_one()
        return self.owner_id and self.owner_id != self.company_id.partner_id

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import models, fields


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    country_code = fields.Char(related="company_id.account_fiscal_country_id.code")

    def action_view_stock_valuation_layers(self):
        self.ensure_one()
        scraps = self.env['stock.scrap'].search([('picking_id', '=', self.id)])
        domain = [('id', 'in', (self.move_ids + scraps.move_id).stock_valuation_layer_ids.ids)]
        action = self.env["ir.actions.actions"]._for_xml_id("stock_account.stock_valuation_layer_action")
        context = literal_eval(action['context'])
        context.update(self.env.context)
        context['no_at_date'] = True
        return dict(action, domain=domain, context=context)

```

## File: models\stock_quant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools.float_utils import float_is_zero
from odoo.tools.misc import groupby


class StockQuant(models.Model):
    _inherit = 'stock.quant'

    value = fields.Monetary('Value', compute='_compute_value', groups='stock.group_stock_manager')
    currency_id = fields.Many2one('res.currency', compute='_compute_value', groups='stock.group_stock_manager')
    accounting_date = fields.Date(
        'Accounting Date',
        help="Date at which the accounting entries will be created"
             " in case of automated inventory valuation."
             " If empty, the inventory date will be used.")
    cost_method = fields.Selection(related="product_categ_id.property_cost_method")

    @api.model
    def _should_exclude_for_valuation(self):
        """
        Determines if a quant should be excluded from valuation based on its ownership.
        :return: True if the quant should be excluded from valuation, False otherwise.
        """
        self.ensure_one()
        return self.owner_id and self.owner_id != self.company_id.partner_id

    @api.depends('company_id', 'location_id', 'owner_id', 'product_id', 'quantity')
    def _compute_value(self):
        """ (Product.value_svl / Product.quantity_svl) * quant.quantity, i.e. average unit cost * on hand qty
        """
        for quant in self:
            quant.currency_id = quant.company_id.currency_id
            if not quant.location_id or not quant.product_id or\
                    not quant.location_id._should_be_valued() or\
                    quant._should_exclude_for_valuation() or\
                    float_is_zero(quant.quantity, precision_rounding=quant.product_id.uom_id.rounding):
                quant.value = 0
                continue
            quantity = quant.product_id.with_company(quant.company_id).quantity_svl
            if float_is_zero(quantity, precision_rounding=quant.product_id.uom_id.rounding):
                quant.value = 0.0
                continue
            quant.value = quant.quantity * quant.product_id.with_company(quant.company_id).value_svl / quantity

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        """ This override is done in order for the grouped list view to display the total value of
        the quants inside a location. This doesn't work out of the box because `value` is a computed
        field.
        """
        if 'value' not in fields:
            return super(StockQuant, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)
        res = super(StockQuant, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)
        for group in res:
            if group.get('__domain'):
                quants = self.search(group['__domain'])
                group['value'] = sum(quant.value for quant in quants)
        return res

    def _apply_inventory(self):
        for accounting_date, inventory_ids in groupby(self, key=lambda q: q.accounting_date):
            inventories = self.env['stock.quant'].concat(*inventory_ids)
            if accounting_date:
                super(StockQuant, inventories.with_context(force_period_date=accounting_date))._apply_inventory()
                inventories.accounting_date = False
            else:
                super(StockQuant, inventories)._apply_inventory()

    def _get_inventory_move_values(self, qty, location_id, location_dest_id, out=False):
        res_move = super()._get_inventory_move_values(qty, location_id, location_dest_id, out)
        if not self.env.context.get('inventory_name'):
            force_period_date = self.env.context.get('force_period_date', False)
            if force_period_date:
                res_move['name'] += _(' [Accounted on %s]', force_period_date)
        return res_move

    @api.model
    def _get_inventory_fields_write(self):
        """ Returns a list of fields user can edit when editing a quant in `inventory_mode`."""
        res = super()._get_inventory_fields_write()
        res += ['accounting_date']
        return res

```

## File: models\stock_valuation_layer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools
from odoo.tools import float_compare, float_is_zero

from itertools import chain
from odoo.tools import groupby
from collections import defaultdict


class StockValuationLayer(models.Model):
    """Stock Valuation Layer"""

    _name = 'stock.valuation.layer'
    _description = 'Stock Valuation Layer'
    _order = 'create_date, id'

    _rec_name = 'product_id'

    company_id = fields.Many2one('res.company', 'Company', readonly=True, required=True)
    product_id = fields.Many2one('product.product', 'Product', readonly=True, required=True, check_company=True, auto_join=True)
    categ_id = fields.Many2one('product.category', related='product_id.categ_id')
    product_tmpl_id = fields.Many2one('product.template', related='product_id.product_tmpl_id')
    quantity = fields.Float('Quantity', readonly=True, digits='Product Unit of Measure')
    uom_id = fields.Many2one(related='product_id.uom_id', readonly=True, required=True)
    currency_id = fields.Many2one('res.currency', 'Currency', related='company_id.currency_id', readonly=True, required=True)
    unit_cost = fields.Monetary('Unit Value', readonly=True)
    value = fields.Monetary('Total Value', readonly=True)
    remaining_qty = fields.Float(readonly=True, digits='Product Unit of Measure')
    remaining_value = fields.Monetary('Remaining Value', readonly=True)
    description = fields.Char('Description', readonly=True)
    stock_valuation_layer_id = fields.Many2one('stock.valuation.layer', 'Linked To', readonly=True, check_company=True, index=True)
    stock_valuation_layer_ids = fields.One2many('stock.valuation.layer', 'stock_valuation_layer_id')
    stock_move_id = fields.Many2one('stock.move', 'Stock Move', readonly=True, check_company=True, index=True)
    account_move_id = fields.Many2one('account.move', 'Journal Entry', readonly=True, check_company=True, index="btree_not_null")
    account_move_line_id = fields.Many2one('account.move.line', 'Invoice Line', readonly=True, check_company=True, index="btree_not_null")
    reference = fields.Char(related='stock_move_id.reference')
    price_diff_value = fields.Float('Invoice value correction with invoice currency')

    def init(self):
        tools.create_index(
            self._cr, 'stock_valuation_layer_index',
            self._table, ['product_id', 'remaining_qty', 'stock_move_id', 'company_id', 'create_date']
        )
        tools.create_index(
            self._cr, 'stock_valuation_company_product_index',
            self._table, ['product_id', 'company_id', 'id', 'value', 'quantity']
        )

    def _validate_accounting_entries(self):
        am_vals = []
        aml_to_reconcile = defaultdict(set)
        for svl in self:
            if not svl.with_company(svl.company_id).product_id.valuation == 'real_time':
                continue
            if svl.currency_id.is_zero(svl.value):
                continue
            move = svl.stock_move_id
            if not move:
                move = svl.stock_valuation_layer_id.stock_move_id
            am_vals += move.with_company(svl.company_id)._account_entry_move(svl.quantity, svl.description, svl.id, svl.value)
        if am_vals:
            account_moves = self.env['account.move'].sudo().create(am_vals)
            account_moves._post()
        products_svl = groupby(self, lambda svl: (svl.product_id, svl.company_id.anglo_saxon_accounting))
        for (product, anglo_saxon_accounting), svls in products_svl:
            svls = self.browse(svl.id for svl in svls)
            moves = svls.stock_move_id
            if anglo_saxon_accounting:
                moves._get_related_invoices()._stock_account_anglo_saxon_reconcile_valuation(product=product)
            moves = (moves | moves.origin_returned_move_id).with_prefetch(chain(moves._prefetch_ids, moves.origin_returned_move_id._prefetch_ids))
            for aml in moves._get_all_related_aml():
                if aml.reconciled or aml.move_id.state != "posted" or not aml.account_id.reconcile:
                    continue
                aml_to_reconcile[(product, aml.account_id)].add(aml.id)
        for aml_ids in aml_to_reconcile.values():
            self.env['account.move.line'].browse(aml_ids).reconcile()

    def _validate_analytic_accounting_entries(self):
        for svl in self:
            svl.stock_move_id._account_analytic_entry_move()

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if 'unit_cost' in fields:
            fields.remove('unit_cost')
        return super().read_group(domain, fields, groupby, offset, limit, orderby, lazy)

    def action_open_layer(self):
        self.ensure_one()
        return {
            'res_model': self._name,
            'type': 'ir.actions.act_window',
            'views': [[False, "form"]],
            'res_id': self.id,
        }

    def action_open_reference(self):
        self.ensure_one()
        if self.stock_move_id:
            action = self.stock_move_id.action_open_reference()
            if action['res_model'] != 'stock.move':
                return action
        return {
            'res_model': self._name,
            'type': 'ir.actions.act_window',
            'views': [[False, "form"]],
            'res_id': self.id,
        }

    def _consume_specific_qty(self, qty_valued, qty_to_value):
        """
        Iterate on the SVL to first skip the qty already valued. Then, keep
        iterating to consume `qty_to_value` and stop
        The method returns the valued quantity and its valuation
        """
        if not self:
            return 0, 0

        rounding = self.product_id.uom_id.rounding
        qty_to_take_on_candidates = qty_to_value
        tmp_value = 0  # to accumulate the value taken on the candidates
        for candidate in self:
            if float_is_zero(candidate.quantity, precision_rounding=rounding):
                continue
            candidate_quantity = abs(candidate.quantity)
            returned_qty = sum([sm.product_uom._compute_quantity(sm.quantity_done, self.uom_id)
                                for sm in candidate.stock_move_id.returned_move_ids if sm.state == 'done'])
            candidate_quantity -= returned_qty
            if float_is_zero(candidate_quantity, precision_rounding=rounding):
                continue
            if not float_is_zero(qty_valued, precision_rounding=rounding):
                qty_ignored = min(qty_valued, candidate_quantity)
                qty_valued -= qty_ignored
                candidate_quantity -= qty_ignored
                if float_is_zero(candidate_quantity, precision_rounding=rounding):
                    continue
            qty_taken_on_candidate = min(qty_to_take_on_candidates, candidate_quantity)

            qty_to_take_on_candidates -= qty_taken_on_candidate
            tmp_value += qty_taken_on_candidate * ((candidate.value + sum(candidate.stock_valuation_layer_ids.mapped('value'))) / candidate.quantity)
            if float_is_zero(qty_to_take_on_candidates, precision_rounding=rounding):
                break

        return qty_to_value - qty_to_take_on_candidates, tmp_value

    def _consume_all(self, qty_valued, valued, qty_to_value):
        """
        The method consumes all svl to get the total qty/value. Then it deducts
        the already consumed qty/value. Finally, it tries to consume the `qty_to_value`
        The method returns the valued quantity and its valuation
        """
        if not self:
            return 0, 0

        rounding = self.product_id.uom_id.rounding
        qty_total = -qty_valued
        value_total = -valued
        new_valued_qty = 0
        new_valuation = 0

        for svl in self:
            if float_is_zero(svl.quantity, precision_rounding=rounding):
                continue
            relevant_qty = abs(svl.quantity)
            returned_qty = sum([sm.product_uom._compute_quantity(sm.quantity_done, self.uom_id)
                                for sm in svl.stock_move_id.returned_move_ids if sm.state == 'done'])
            relevant_qty -= returned_qty
            if float_is_zero(relevant_qty, precision_rounding=rounding):
                continue
            qty_total += relevant_qty
            value_total += relevant_qty * ((svl.value + sum(svl.stock_valuation_layer_ids.mapped('value'))) / svl.quantity)

        if float_compare(qty_total, 0, precision_rounding=rounding) > 0:
            unit_cost = value_total / qty_total
            new_valued_qty = min(qty_total, qty_to_value)
            new_valuation = unit_cost * new_valued_qty

        return new_valued_qty, new_valuation

    def _should_impact_price_unit_receipt_value(self):
        self.ensure_one()
        return True

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_chart_template
from . import account_move
from . import product
from . import stock_move
from . import stock_location
from . import stock_move_line
from . import stock_picking
from . import stock_quant
from . import stock_valuation_layer
from . import res_config_settings

```

## File: report\report_stock_forecasted.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="stock_account_report_product_product_replenishment" inherit_id="stock.report_replenishment_header">
    </template>
</odoo>

```

## File: report\stock_forecasted.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools.float_utils import float_is_zero, float_repr


class ReplenishmentReport(models.AbstractModel):
    _inherit = 'report.stock.report_product_product_replenishment'

    def _compute_draft_quantity_count(self, product_template_ids, product_variant_ids, wh_location_ids):
        """ Overrides to computes the valuations of the stock. """
        res = super()._compute_draft_quantity_count(product_template_ids, product_variant_ids, wh_location_ids)
        if not self.user_has_groups('stock.group_stock_manager'):
            return res
        domain = self._product_domain(product_template_ids, product_variant_ids)
        company = self.env['stock.location'].browse(wh_location_ids[0]).company_id
        svl = self.env['stock.valuation.layer'].search(domain + [('company_id', '=', company.id)])
        domain_quants = [
            ('company_id', '=', company.id),
            ('location_id', 'in', wh_location_ids)
        ]
        if product_template_ids:
            domain_quants += [('product_id.product_tmpl_id', 'in', product_template_ids)]
        else:
            domain_quants += [('product_id', 'in', product_variant_ids)]
        quants = self.env['stock.quant'].search(domain_quants)
        currency = svl.currency_id or self.env.company.currency_id
        total_quantity = sum(svl.mapped('quantity'))
        # Because we can have negative quantities, `total_quantity` may be equal to zero even if the warehouse's `quantity` is positive.
        if svl and not float_is_zero(total_quantity, precision_rounding=svl.product_id.uom_id.rounding):
            value = sum(svl.mapped('value')) * (sum(quants.mapped('quantity')) / total_quantity)
        else:
            value = 0
        value = float_repr(value, precision_digits=currency.decimal_places)
        if currency.position == 'after':
            value = '%s %s' % (value, currency.symbol)
        else:
            value = '%s %s' % (currency.symbol, value)
        res['value'] = value
        return res

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_forecasted

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_account_account_stock_manager,account.account stock manager,account.model_account_account,stock.group_stock_manager,1,0,0,0
access_account_journal_stock_manager,account.journal stock manager,account.model_account_journal,stock.group_stock_manager,1,0,0,0
access_stock_picking_invoicing_payments_readonly,stock.picking,stock.model_stock_picking,account.group_account_readonly,1,0,0,0
access_stock_picking_invoicing_payments,stock.picking,stock.model_stock_picking,account.group_account_invoice,1,1,1,0
access_stock_move_invoicing_payments_readonly,stock.move,model_stock_move,account.group_account_readonly,1,0,0,0
access_stock_move_invoicing_payments,stock.move,model_stock_move,account.group_account_invoice,1,1,1,0
access_stock_valuation_layer,access_stock_valuation_layer,model_stock_valuation_layer,stock.group_stock_manager,1,1,1,0
access_stock_valuation_layer_revaluation,access_stock_valuation_layer_revaluation,model_stock_valuation_layer_revaluation,stock.group_stock_manager,1,1,1,0
```

## File: security\stock_account_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record model="ir.rule" id="stock_valuation_layer_company_rule">
        <field name="name">Stock Valuation Layer Multicompany</field>
        <field name="model_id" search="[('model','=','stock.valuation.layer')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record id="group_lot_on_invoice" model="res.groups">
        <field name="name">Display Serial &amp; Lot Number on Invoices</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

</odoo>

```

## File: static\src\stock_account_forecasted\forecasted_header.js

```javascript
/** @odoo-module **/
import { patch } from '@web/core/utils/patch';

import { ForecastedHeader as Parent } from "@stock/stock_forecasted/forecasted_header";

export class StockAccountForecastedHeader extends Parent{}

patch(Parent.prototype, 'stock_account.ForecastedHeader', {
    async _onClickValuation() {
        const templates = this.props.docs.product_templates_ids;
        const variants = this.props.docs.product_variants_ids;
        const context = Object.assign({}, this.context);
        if (templates) {
            context.search_default_product_tmpl_id = templates;
        } else {
            context.search_default_product_id = variants;
        }
        return this.action.doAction({
            name: this.env._t('Stock Valuation'),
            res_model: 'stock.valuation.layer',
            type: 'ir.actions.act_window',
            view_mode: 'list,form',
            views: [[false, 'list'], [false, 'form']],
            target: 'current',
            context: context,
        });
    }
});

StockAccountForecastedHeader.template = 'stock_account.ForecastedHeader';

```

## File: static\src\stock_account_forecasted\forecasted_header.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template">
    <t t-name="stock_account.ForecastedHeader" owl="1" t-inherit="stock.ForecastedHeader" t-inherit-mode="extension">
        <xpath expr="//h6[@name='product_variants']" position="after">
            <h6 t-if="() => env.user.has_group('stock.group_stock_manager')">
                Value On Hand:
                <a href="#" class="o_report_open_valuation_report"
                   t-out="props.docs.value"
                   t-on-click.prevent="_onClickValuation"/>
            </h6>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\inventory_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

<t t-inherit="stock.InventoryReport.Buttons" t-inherit-mode="extension" owl="1">
    <xpath expr="//button[hasclass('o_button_at_date')]" position="after">
        <button t-if="props.resModel == 'stock.valuation.layer'" type="button"
            class="btn btn-primary o_button_at_date me-2" t-on-click="onClickInventoryAtDate">
            Valuation at Date
        </button>
    </xpath>
</t>

</templates>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="product_template_tree_view" model="ir.ui.view">
            <field name="name">product.template.tree.inherit.stock.account</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_tree_view"/>
            <field name="arch" type="xml">
                <field name="standard_price" position="attributes">
                    <attribute name="readonly">1</attribute>
                </field>
            </field>
        </record>

        <record id="view_category_property_form_stock" model="ir.ui.view">
            <field name="name">product.category.stock.property.form.inherit.stock</field>
            <field name="model">product.category</field>
            <field name="inherit_id" ref="stock.product_category_form_view_inherit"/>
            <field name="arch" type="xml">
                <group name="logistics" position="after">
                    <group string="Inventory Valuation">
                        <field name="property_cost_method"/>
                        <field name="property_valuation" groups="account.group_account_readonly,stock.group_stock_manager"/>
                    </group>
                </group>
            </field>
        </record>

        <record id="view_category_property_form" model="ir.ui.view">
            <field name="name">product.category.stock.property.form.inherit</field>
            <field name="model">product.category</field>
            <field name="inherit_id" ref="account.view_category_property_form"/>
            <field name="arch" type="xml">
                <group name="account_property" position="inside">
                    <group name="account_stock_property" string="Account Stock Properties" groups="account.group_account_readonly" attrs="{'invisible':[('property_valuation', '=', 'manual_periodic')]}">
                        <field name="property_valuation" invisible="1"/>
                        <field name="property_stock_valuation_account_id" options="{'no_create': True}" attrs="{'required':[('property_valuation', '=', 'real_time')]}"/>
                        <field name="property_stock_journal" attrs="{'required':[('property_valuation', '=', 'real_time')]}" />
                        <field name="property_stock_account_input_categ_id" options="{'no_create': True}" attrs="{'required':[ ('property_valuation', '=', 'real_time')]}" />
                        <field name="property_stock_account_output_categ_id" options="{'no_create': True}" attrs="{'required':[ ('property_valuation', '=', 'real_time')]}" />
                        <div colspan="2" class="alert alert-info mt16" role="status">
                            <b>Set other input/output accounts on specific </b><button name="%(stock.action_prod_inv_location_form)d" role="button" type="action" class="btn-link" style="padding: 0;vertical-align: baseline;" string="locations"/>.
                        </div>
                    </group>
                </group>
            </field>
        </record>

        <!-- Stock Report View -->
        <record model="ir.ui.view" id="product_product_stock_tree_inherit_stock_account">
            <field name="name">product.product.stock.tree.inherit.stock.account</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="stock.product_product_stock_tree"/>
            <field name="arch" type="xml">
                <field name="qty_available" position="before">
                    <field name="company_currency_id" invisible="1"/>
                    <field name="cost_method" invisible="1"/>
                    <field name="avg_cost" string="Unit Cost" optional="show" widget='monetary' options="{'currency_field': 'company_currency_id'}"/>
                    <field name="total_value" string="Total Value" optional="show" widget='monetary' options="{'currency_field': 'company_currency_id'}" sum="Total Value"/>
                    <button name="%(stock_valuation_layer_action)d" title="Valuation Report" type="action" class="btn-link"
                        icon="fa-bar-chart" context="{'search_default_product_id': id, 'default_product_id': id}"  attrs="{'invisible': [('cost_method', '!=', 'average')]}"/>
                    <button name="%(stock_valuation_layer_report_action)d" title="Valuation Report" type="action" class="btn-link"
                        icon="fa-bar-chart" context="{'search_default_product_id': id, 'default_product_id': id}"  attrs="{'invisible': [('cost_method', '!=', 'fifo')]}"/>
                </field>
            </field>
        </record>
   </data>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="stock_account_report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='total']" position="after">
          <t groups="stock_account.group_lot_on_invoice">
            <t t-set="lot_values" t-value="o._get_invoiced_lot_values()"/>
            <t t-if="lot_values">
                <br/>
                <table class="table table-sm" style="width: 50%;" name="invoice_snln_table">
                    <thead>
                        <tr>
                            <th><span>Product</span></th>
                            <th class="text-end"><span>Quantity</span></th>
                            <th class="text-end"><span>SN/LN</span></th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="lot_values" t-as="snln_line">
                            <tr>
                                <td><t t-esc="snln_line['product_name']"/></td>
                                <td class="text-end">
                                    <t t-esc="snln_line['quantity']"/>
                                    <t t-esc="snln_line['uom_name']" groups="uom.group_uom"/>
                                </td>
                                <td class="text-end"><t t-esc="snln_line['lot_name']"/></td>
                            </tr>
                        </t>
                    </tbody>
                </table>
            </t>
          </t>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.stock.account</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="stock.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <div id="production_lot_info" position="after">
                    <h2>Valuation</h2>
                    <div class="row mt16 o_settings_container" name="valuation_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="additional_cost_setting"
                            title="Affect landed costs on reception operations and split them among products to update their cost price.">
                            <div class="o_setting_left_pane">
                                <field name="module_stock_landed_costs"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_stock_landed_costs"/>
                                <a href="https://www.odoo.com/documentation/16.0/applications/inventory_and_mrp/inventory/management/reporting/integrating_landed_costs.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Add additional cost (transport, customs, ...) in the value of the product.
                                </div>
                                <div class="content-group">
                                    <div name="landed_cost_info"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" attrs="{'invisible': [('group_stock_production_lot', '=', False)]}" id="group_lot_on_invoice">
                            <div class="o_setting_left_pane">
                                <field name="group_lot_on_invoice"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_lot_on_invoice"/>
                                <div class="text-muted">
                                    Lots &amp; Serial numbers will appear on the invoice
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\stock_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_move_form_inherit" model="ir.ui.view">
            <field name="name">stock.move.form.inherit</field>
            <field name="model">stock.move</field>
            <field name="inherit_id" ref="stock.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='button_box']" position="inside" >
                    <button name="action_get_account_moves" icon="fa-usd" class="oe_stat_button" string="Accounting Entries" type="object" groups="account.group_account_readonly"/>
                </xpath>
            </field>
        </record>

        <record id="view_stock_quant_tree_inventory_editable_inherit_stock_account" model="ir.ui.view">
            <field name="name">stock.quant.inventory.tree.editable.inherit.stock.account</field>
            <field name="model">stock.quant</field>
            <field name="inherit_id" ref="stock.view_stock_quant_tree_inventory_editable"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='product_uom_id']" position="after">
                    <field name="accounting_date" optional="hide"/>
                </xpath>
            </field>
        </record>

        <record id="view_location_form_inherit" model="ir.ui.view">
            <field name="name">stock.location.form.inherit</field>
            <field name="model">stock.location</field>
            <field name="inherit_id" ref="stock.view_location_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='additional_info']" position="after">
                    <group string="Accounting Information" attrs="{'invisible':[('usage','not in',('inventory','production'))]}">
                        <field name="valuation_in_account_id" options="{'no_create': True}"/>
                        <field name="valuation_out_account_id" options="{'no_create': True}"/>
                    </group>
                </xpath>
            </field>
        </record>

        <record id="view_stock_return_picking_form_inherit_stock_account" model="ir.ui.view">
            <field name="name">stock.return.picking.stock.account.form</field>
            <field name="inherit_id" ref="stock.view_stock_return_picking_form"/>
            <field name="model">stock.return.picking</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='product_return_moves']/tree" position="inside">
                    <field name="to_refund" groups="base.group_no_one"/>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\stock_quant_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="view_stock_quant_tree_inherit">
        <field name="name">stock.quant.tree.inherit</field>
        <field name="model">stock.quant</field>
        <field name="inherit_id" ref="stock.view_stock_quant_tree"></field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_uom_id']" position="after">
                <field name="currency_id" invisible="1"/>
                <field name="value" optional="hidden"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="view_stock_quant_tree_editable_inherit">
        <field name="name">stock.quant.tree.editable.inherit</field>
        <field name="model">stock.quant</field>
        <field name="inherit_id" ref="stock.view_stock_quant_tree_editable"></field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_uom_id']" position="after">
                <field name="currency_id" invisible="1"/>
                <field name="cost_method" invisible="1"/>
                <field name="value" optional="hidden" sum="Total Value"/>
            </xpath>
            <xpath expr="//button[@name='action_view_orderpoints']" position="after">
                <button name="%(stock_valuation_layer_report_action)d" title="Stock Valuation"
                        string="Valuation" type="action" class="btn-link" icon="fa-bar-chart"
                        context="{'search_default_product_id': product_id}"
                        attrs="{'invisible': [('cost_method', '!=', 'fifo')]}"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\stock_valuation_layer_views.xml

```xml
<odoo>
    <record id="stock_valuation_layer_form" model="ir.ui.view">
        <field name="name">stock.valuation.layer.form</field>
        <field name="model">stock.valuation.layer</field>
        <field name="arch" type="xml">
            <form edit="0" create="0">
                <sheet>
                    <group>
                        <group>
                            <field name="create_date" string="Date" />
                            <field name="product_id" />
                            <field name="stock_move_id" attrs="{'invisible': [('stock_move_id', '=', False)]}" />
                        </group>
                    </group>
                    <notebook>
                        <page string="Valuation" name="valuation">
                            <group>
                                <field name="quantity" />
                                <field name="uom_id" groups="uom.group_uom" />
                                <field name="currency_id" invisible="1" />
                                <field name="unit_cost" />
                                <field name="value" />
                                <field name="remaining_qty" />
                            </group>
                        </page>
                        <page string="Other Info" name="other_info">
                            <group>
                                <field name="description" />
                                <field name="account_move_id" groups="account.group_account_invoice" attrs="{'invisible': [('account_move_id', '=', False)]}" />
                                <field name="company_id" groups="base.group_multi_company" />
                                <field name="stock_valuation_layer_id" attrs="{'invisible': [('stock_valuation_layer_id', '=', False)]}" />
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="stock_valuation_layer_tree" model="ir.ui.view">
        <field name="name">stock.valuation.layer.tree</field>
        <field name="model">stock.valuation.layer</field>
        <field name="arch" type="xml">
            <tree default_order="id desc" create="0"
                  import="0" js_class="inventory_report_list"
                  action="action_open_reference" type="object">
                <field name="create_date" string="Date" />
                <field name="reference"/>
                <button name="action_open_layer" icon="fa-arrow-right" title="Open Valuation Layer" type="object"/>
                <field name="product_id" />
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="quantity" string="Moved Quantity"/>
                <field name="unit_cost" />
                <field name="uom_id" groups="uom.group_uom" />
                <field name="currency_id" invisible="1" />
                <field name="value" sum="Total Value"/>
                <groupby name="product_id">
                    <field name="cost_method" invisible="1"/>
                    <field name="quantity_svl" invisible="1"/>
                    <button name="action_revaluation" icon="fa-plus" title="Add Manual Valuation" type="object" attrs="{'invisible':['|', ('cost_method', '=', 'standard'), ('quantity_svl', '&lt;=', 0)]}" />
                </groupby>
            </tree>
        </field>
    </record>

    <record id="stock_valuation_layer_pivot" model="ir.ui.view">
        <field name="name">stock.valuation.layer.pivot</field>
        <field name="model">stock.valuation.layer</field>
        <field name="arch" type="xml">
            <pivot>
                <field name="quantity" type="measure"/>
                <field name="value" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="stock_valuation_layer_action" model="ir.actions.act_window">
        <field name="name">Stock Valuation</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">stock.valuation.layer</field>
        <field name="view_mode">tree,form,pivot</field>
        <field name="view_id" ref="stock_valuation_layer_tree"/>
        <field name="domain">[('product_id.type', '=', 'product')]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face"/>
            <p>
                There are no valuation layers. Valuation layers are created when there are product moves that impact the valuation of the stock.
            </p>
        </field>
    </record>

    <record id="view_inventory_valuation_search" model="ir.ui.view">
        <field name="name">Inventory Valuation</field>
        <field name="model">stock.valuation.layer</field>
        <field name="arch" type="xml">
            <search string="Inventory Valuation">
                <field name="product_id"/>
                <field name="categ_id" />
                <field name="product_tmpl_id" />
                <separator/>
                <filter string="Incoming" name="incoming" domain="[('stock_move_id.location_id.usage', 'not in', ('internal', 'transit')), ('stock_move_id.location_dest_id.usage', 'in', ('internal', 'transit'))]"/>
                <filter string="Outgoing" name="outgoing" domain="[('stock_move_id.location_id.usage', 'in', ('internal', 'transit')), ('stock_move_id.location_dest_id.usage', 'not in', ('internal', 'transit'))]"/>
                <separator/>
                <filter string="Has Remaining Qty" name="has_remaining_qty" domain="[('remaining_qty', '>', 0)]"/>
                <group expand='0' string='Group by...'>
                    <filter string='Product' name="group_by_product_id" context="{'group_by': 'product_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <!-- reporting view -->
    <record id="stock_valuation_layer_report_tree" model="ir.ui.view">
        <field name="name">stock.valuation.layer.report.tree</field>
        <field name="model">stock.valuation.layer</field>
        <field name="inherit_id" ref="stock_valuation_layer_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <field name="quantity" position="attributes">
                <attribute name="invisible">1</attribute>
            </field>
            <field name="unit_cost" position="after">
                <field name="remaining_qty" sum="Total Remaining Quantity"/>
            </field>
            <field name="value" position="before">
                <field name="remaining_value" sum="Total Remaining Value"/>
            </field>
        </field>
    </record>

    <record id="stock_valuation_layer_report_action" model="ir.actions.act_window">
        <field name="name">Stock Valuation</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">stock.valuation.layer</field>
        <field name="view_mode">tree,form,pivot</field>
        <field name="view_id" ref="stock_valuation_layer_report_tree"/>
        <field name="context">{'search_default_has_remaining_qty': 1}</field>
        <field name="domain">[('product_id.type', '=', 'product')]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face"/>
            <p>
                There are no valuation layers. Valuation layers are created when there are product moves that impact the valuation of the stock.
            </p>
        </field>
    </record>

    <menuitem id="menu_valuation" name="Valuation" parent="stock.menu_warehouse_report" sequence="250" action="stock_valuation_layer_action" groups="base.group_no_one"/>

    <record id="stock_valuation_layer_picking" model="ir.ui.view">
        <field name="name">stock.valuation.layer.picking</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button string="Valuation" type="object"
                    name="action_view_stock_valuation_layers"
                    class="oe_stat_button" icon="fa-dollar" groups="base.group_no_one"
                    attrs="{'invisible': [('state', 'not in', ['done'])]}" />
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\stock_picking_return.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockReturnPicking(models.TransientModel):
    _inherit = "stock.return.picking"

    @api.model
    def default_get(self, default_fields):
        res = super(StockReturnPicking, self).default_get(default_fields)
        for i, k, vals in res.get('product_return_moves', []):
            vals.update({'to_refund': True})
        return res

    def _create_returns(self):
        new_picking_id, pick_type_id = super(StockReturnPicking, self)._create_returns()
        new_picking = self.env['stock.picking'].browse([new_picking_id])
        for move in new_picking.move_ids:
            return_picking_line = self.product_return_moves.filtered(lambda r: r.move_id == move.origin_returned_move_id)[:1]
            if return_picking_line and return_picking_line.to_refund:
                move.to_refund = True
        return new_picking_id, pick_type_id


class StockReturnPickingLine(models.TransientModel):
    _inherit = "stock.return.picking.line"

    to_refund = fields.Boolean(string="Update quantities on SO/PO", default=True,
        help='Trigger a decrease of the delivered/received quantity in the associated Sale Order/Purchase Order')

```

## File: wizard\stock_quantity_history.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools.misc import format_datetime


class StockQuantityHistory(models.TransientModel):
    _inherit = 'stock.quantity.history'

    def open_at_date(self):
        active_model = self.env.context.get('active_model')
        if active_model == 'stock.valuation.layer':
            action = self.env["ir.actions.actions"]._for_xml_id("stock_account.stock_valuation_layer_action")
            action['domain'] = [('create_date', '<=', self.inventory_datetime), ('product_id.type', '=', 'product')]
            action['display_name'] = format_datetime(self.env, self.inventory_datetime)
            return action

        return super(StockQuantityHistory, self).open_at_date()

```

## File: wizard\stock_quantity_history.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_stock_quantity_history_inherit_stock_account" model="ir.ui.view">
        <field name="name">Valuation Report at Date</field>
        <field name="model">stock.quantity.history</field>
        <field name="inherit_id" ref="stock.view_stock_quantity_history"></field>
        <field name="arch" type="xml">
            <!-- ensure string/description matches the model we're looking at the history of -->
            <field name="inventory_datetime" position="attributes">
                <attribute name="invisible">
                    context.get('active_model') != 'stock.quant'
                </attribute>
            </field>
            <field name="inventory_datetime" position="after">
                <field name="inventory_datetime" string="Valuation at Date"
                       help="Choose a date to get the valuation at that date"
                       invisible="context.get('active_model') != 'stock.valuation.layer'"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: wizard\stock_request_count.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockRequestCount(models.TransientModel):
    _inherit = 'stock.request.count'

    accounting_date = fields.Date('Accounting Date')

    def _get_values_to_write(self):
        res = super()._get_values_to_write()
        if self.accounting_date:
            res['accounting_date'] = self.accounting_date
        return res

```

## File: wizard\stock_request_count.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_inventory_request_count_form_view_inherit_stock_account" model="ir.ui.view">
        <field name="name">stock.request.count.form.view.inherit.stock.account</field>
        <field name="model">stock.request.count</field>
        <field name="inherit_id" ref="stock.stock_inventory_request_count_form_view"/>
        <field name="arch" type="xml">
            <field name="user_id" position="after">
                <field name="accounting_date"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: wizard\stock_valuation_layer_revaluation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.tools import float_compare, float_is_zero


class StockValuationLayerRevaluation(models.TransientModel):
    _name = 'stock.valuation.layer.revaluation'
    _description = "Wizard model to reavaluate a stock inventory for a product"
    _check_company_auto = True

    @api.model
    def default_get(self, default_fields):
        res = super().default_get(default_fields)
        if res.get('product_id'):
            product = self.env['product.product'].browse(res['product_id'])
            if product.categ_id.property_cost_method == 'standard':
                raise UserError(_("You cannot revalue a product with a standard cost method."))
            if product.quantity_svl <= 0:
                raise UserError(_("You cannot revalue a product with an empty or negative stock."))
            if 'account_journal_id' not in res and 'account_journal_id' in default_fields and product.categ_id.property_valuation == 'real_time':
                accounts = product.product_tmpl_id.get_product_accounts()
                res['account_journal_id'] = accounts['stock_journal'].id
        return res

    company_id = fields.Many2one('res.company', "Company", readonly=True, required=True)
    currency_id = fields.Many2one('res.currency', "Currency", related='company_id.currency_id', required=True)

    product_id = fields.Many2one('product.product', "Related product", required=True, check_company=True)
    property_valuation = fields.Selection(related='product_id.categ_id.property_valuation')
    product_uom_name = fields.Char("Unit of Measure", related='product_id.uom_id.name')
    current_value_svl = fields.Float("Current Value", related="product_id.value_svl")
    current_quantity_svl = fields.Float("Current Quantity", related="product_id.quantity_svl")

    added_value = fields.Monetary("Added value", required=True)
    new_value = fields.Monetary("New value", compute='_compute_new_value')
    new_value_by_qty = fields.Monetary("New value by quantity", compute='_compute_new_value')
    reason = fields.Char("Reason", help="Reason of the revaluation")

    account_journal_id = fields.Many2one('account.journal', "Journal", check_company=True)
    account_id = fields.Many2one('account.account', "Counterpart Account", domain=[('deprecated', '=', False)], check_company=True)
    date = fields.Date("Accounting Date")

    @api.depends('current_value_svl', 'current_quantity_svl', 'added_value')
    def _compute_new_value(self):
        for reval in self:
            reval.new_value = reval.current_value_svl + reval.added_value
            if not float_is_zero(reval.current_quantity_svl, precision_rounding=self.product_id.uom_id.rounding):
                reval.new_value_by_qty = reval.new_value / reval.current_quantity_svl
            else:
                reval.new_value_by_qty = 0.0

    def action_validate_revaluation(self):
        """ Revaluate the stock for `self.product_id` in `self.company_id`.

        - Change the stardard price with the new valuation by product unit.
        - Create a manual stock valuation layer with the `added_value` of `self`.
        - Distribute the `added_value` on the remaining_value of layers still in stock (with a remaining quantity)
        - If the Inventory Valuation of the product category is automated, create
        related account move.
        """
        self.ensure_one()
        if self.currency_id.is_zero(self.added_value):
            raise UserError(_("The added value doesn't have any impact on the stock valuation"))

        product_id = self.product_id.with_company(self.company_id)

        remaining_svls = self.env['stock.valuation.layer'].search([
            ('product_id', '=', product_id.id),
            ('remaining_qty', '>', 0),
            ('company_id', '=', self.company_id.id),
        ])

        # Create a manual stock valuation layer
        if self.reason:
            description = _("Manual Stock Valuation: %s.", self.reason)
        else:
            description = _("Manual Stock Valuation: No Reason Given.")
        if product_id.categ_id.property_cost_method == 'average':
            description += _(
                " Product cost updated from %(previous)s to %(new_cost)s.",
                previous=product_id.standard_price,
                new_cost=product_id.standard_price + self.added_value / self.current_quantity_svl
            )
        revaluation_svl_vals = {
            'company_id': self.company_id.id,
            'product_id': product_id.id,
            'description': description,
            'value': self.added_value,
            'quantity': 0,
        }

        remaining_qty = sum(remaining_svls.mapped('remaining_qty'))
        remaining_value = self.added_value
        remaining_value_unit_cost = self.currency_id.round(remaining_value / remaining_qty)
        for svl in remaining_svls:
            if float_is_zero(svl.remaining_qty - remaining_qty, precision_rounding=self.product_id.uom_id.rounding):
                taken_remaining_value = remaining_value
            else:
                taken_remaining_value = remaining_value_unit_cost * svl.remaining_qty
            if float_compare(svl.remaining_value + taken_remaining_value, 0, precision_rounding=self.product_id.uom_id.rounding) < 0:
                raise UserError(_('The value of a stock valuation layer cannot be negative. Landed cost could be use to correct a specific transfer.'))

            svl.remaining_value += taken_remaining_value
            remaining_value -= taken_remaining_value
            remaining_qty -= svl.remaining_qty

        previous_value_svl = self.current_value_svl
        revaluation_svl = self.env['stock.valuation.layer'].create(revaluation_svl_vals)

        # Update the stardard price in case of AVCO
        if product_id.categ_id.property_cost_method in ('average', 'fifo'):
            product_id.with_context(disable_auto_svl=True).standard_price += self.added_value / self.current_quantity_svl

        # If the Inventory Valuation of the product category is automated, create related account move.
        if self.property_valuation != 'real_time':
            return True

        accounts = product_id.product_tmpl_id.get_product_accounts()

        if self.added_value < 0:
            debit_account_id = self.account_id.id
            credit_account_id = accounts.get('stock_valuation') and accounts['stock_valuation'].id
        else:
            debit_account_id = accounts.get('stock_valuation') and accounts['stock_valuation'].id
            credit_account_id = self.account_id.id

        move_vals = {
            'journal_id': self.account_journal_id.id or accounts['stock_journal'].id,
            'company_id': self.company_id.id,
            'ref': _("Revaluation of %s", product_id.display_name),
            'stock_valuation_layer_ids': [(6, None, [revaluation_svl.id])],
            'date': self.date or fields.Date.today(),
            'move_type': 'entry',
            'line_ids': [(0, 0, {
                'name': _('%(user)s changed stock valuation from  %(previous)s to %(new_value)s - %(product)s',
                    user=self.env.user.name,
                    previous=previous_value_svl,
                    new_value=previous_value_svl + self.added_value,
                    product=product_id.display_name,
                ),
                'account_id': debit_account_id,
                'debit': abs(self.added_value),
                'credit': 0,
                'product_id': product_id.id,
            }), (0, 0, {
                'name': _('%(user)s changed stock valuation from  %(previous)s to %(new_value)s - %(product)s',
                    user=self.env.user.name,
                    previous=previous_value_svl,
                    new_value=previous_value_svl + self.added_value,
                    product=product_id.display_name,
                ),
                'account_id': credit_account_id,
                'debit': 0,
                'credit': abs(self.added_value),
                'product_id': product_id.id,
            })],
        }
        account_move = self.env['account.move'].create(move_vals)
        account_move._post()

        return True

```

## File: wizard\stock_valuation_layer_revaluation_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="stock_valuation_layer_revaluation_form_view" model="ir.ui.view">
        <field name="name">stock.valuation.layer.revaluation.form</field>
        <field name="model">stock.valuation.layer.revaluation</field>
        <field name="arch" type="xml">
            <form string="Product Revaluation">
                <sheet>
                    <group>
                        <label for="current_value_svl" string="Current Value"/>
                        <div class="o_row">
                            <span>
                            <field name="current_value_svl" class="oe_inline" widget="monetary"/> for <field name="current_quantity_svl" class="oe_inline"/> <field name="product_uom_name" class="oe_inline"/>
                            </span>
                        </div>
                        <label for="added_value" string="Added Value"/>
                        <div class="o_row">
                            <span><field name="added_value" class="oe_inline"/> = <field name="new_value" class="oe_inline"/> (<field name="new_value_by_qty" class="oe_inline ms-1"/> by <field name="product_uom_name" class="oe_inline me-1"/>)
                            <small class="mx-2 fst-italic">Use a negative added value to record a decrease in the product value</small></span>
                        </div>
                        <field name="company_id" invisible="1"/>
                        <field name="currency_id" invisible="1"/>
                        <field name="product_id" invisible="1"/>
                    </group>
                    <group>
                        <field name="property_valuation" invisible="1"/>
                        <group>
                            <field name="reason"/>
                            <field name="account_journal_id" attrs="{'invisible':[('property_valuation', '!=', 'real_time')], 'required': [('property_valuation', '=', 'real_time')]}"/>
                        </group>
                        <group>
                            <field name="account_id" attrs="{'invisible':[('property_valuation', '!=', 'real_time')], 'required': [('property_valuation', '=', 'real_time')]}"/>
                            <field name="date" attrs="{'invisible':[('property_valuation', '!=', 'real_time')]}"/>
                        </group>
                    </group>
                </sheet>
                <footer>
                    <button name="action_validate_revaluation" string="Revalue" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_quantity_history
from . import stock_picking_return
from . import stock_request_count
from . import stock_valuation_layer_revaluation

```

