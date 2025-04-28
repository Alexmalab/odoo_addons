# Odoo Module: purchase_stock

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import populate
from . import wizard

from odoo import api, SUPERUSER_ID


def _create_buy_rules(cr, registry):
    """ This hook is used to add a default buy_pull_id on every warehouse. It is
    necessary if the purchase_stock module is installed after some warehouses
    were already created.
    """
    env = api.Environment(cr, SUPERUSER_ID, {})
    warehouse_ids = env['stock.warehouse'].search([('buy_pull_id', '=', False)])
    warehouse_ids.write({'buy_to_resupply': True})

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Purchase Stock',
    'version': '1.2',
    'category': 'Inventory/Purchase',
    'sequence': 60,
    'summary': 'Purchase Orders, Receipts, Vendor Bills for Stock',
    'depends': ['stock_account', 'purchase'],
    'data': [
        'security/ir.model.access.csv',
        'data/purchase_stock_data.xml',
        'data/mail_templates.xml',
        'report/vendor_delay_report.xml',
        'views/purchase_views.xml',
        'views/stock_views.xml',
        'views/stock_rule_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/stock_lot_views.xml',
        'report/purchase_report_views.xml',
        'report/purchase_report_templates.xml',
        'report/report_stock_forecasted.xml',
        'report/report_stock_rule.xml',
        'wizard/stock_replenishment_info.xml'
    ],
    'demo': [
        'data/purchase_stock_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'post_init_hook': '_create_buy_rules',
    'assets': {
        'web.assets_backend': [
            'purchase_stock/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="exception_on_po">
        <div>
          Exception(s) occurred on the purchase order(s):
          <t t-foreach="purchase_order_ids" t-as="purchase_order">
              <a href="#" data-oe-model="purchase.order" t-att-data-oe-id="purchase_order.id"><t t-esc="purchase_order.name"/></a>.
          </t>
          Manual actions may be needed.
          <div class="mt16">
              <p>Exception(s):</p>
              <ul t-foreach="order_exceptions" t-as="exception">
                  <li>
                      <t t-set="order_line" t-value="exception[0]"/>
                      <t t-set="new_qty" t-value="exception[1][0]"/>
                      <t t-set="old_qty" t-value="exception[1][1]"/>
                      <t t-esc="new_qty"/> <t t-esc="order_line.product_uom.name"/> of <t t-esc="order_line.product_id.name"/>
                      ordered instead of <t t-esc="old_qty"/> <t t-esc="order_line.product_uom.name"/>
                  </li>
              </ul>
          </div>
          <div class="mt16" t-if="impacted_pickings">
              <p>Next transfer(s) impacted:</p>
              <ul t-foreach="impacted_pickings" t-as="picking">
                  <li><a href="#" data-oe-model="stock.picking" t-att-data-oe-id="picking.id"><t t-esc="picking.name"/></a></li>
              </ul>
          </div>
        </div>
    </template>
</odoo>

```

## File: data\purchase_stock_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <function model="purchase.order.line" name="_update_qty_received_method" />

		<!--
             Stock rules and routes
        -->
        <record id="route_warehouse0_buy" model='stock.route'>
            <field name="name">Buy</field>
            <field name="company_id"></field>
            <field name="sequence">5</field>
        </record>

        <!-- enable purchase on main warehouse -->
        <record id="stock.warehouse0" model="stock.warehouse">
            <field name="buy_to_resupply" eval="True"/>
        </record>

    </data>
</odoo>

```

## File: data\purchase_stock_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="stock.res_company_1" model="res.company">
            <field eval="1.0" name="po_lead"/>
        </record>

        <record id="product.product_delivery_01" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_delivery_02" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_5" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_9" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_12" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_13" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_16" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_20" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="purchase_order_8" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="state">draft</field>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="date_planned" eval="(datetime.now() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
                (0, 0, {
                    'product_id': ref('product.product_product_25'),
                    'name': obj().env.ref('product.product_product_25').partner_ref,
                    'price_unit': 286.80,
                    'product_qty': 20.0,
                    'product_uom': ref('uom.product_uom_unit'),
                    'date_planned': DateTime.today()}),
            ]"/>
        </record>

        <function model="purchase.order" name="button_confirm" eval="[[ref('purchase_order_8')]]"/>

    </data>

    <data noupdate="0">

        <record id="stock.stock_warehouse_shop0" model="stock.warehouse">
            <field name="buy_to_resupply" eval="True"/>
        </record>

        <function model="stock.warehouse" name="write">
          <value model="stock.warehouse" search="[('partner_id', '=', ref('stock.res_partner_company_1'))]"/>
          <value eval="{'buy_to_resupply': True}"/>
        </function>

    </data>

</odoo>

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools import float_compare, float_is_zero
from odoo.tools.misc import groupby


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _stock_account_prepare_anglo_saxon_in_lines_vals(self):
        ''' Prepare values used to create the journal items (account.move.line) corresponding to the price difference
         lines for vendor bills. It only concerns the quantities that have been delivered before the bill
        Example:
        Buy a product having a cost of 9 and a supplier price of 10 and being a storable product and having a perpetual
        valuation in FIFO. Deliver the product and then post the bill. The vendor bill's journal entries looks like:
        Account                                     | Debit | Credit
        ---------------------------------------------------------------
        101120 Stock Interim Account (Received)     | 10.0  |
        ---------------------------------------------------------------
        101100 Account Payable                      |       | 10.0
        ---------------------------------------------------------------
        This method computes values used to make two additional journal items:
        ---------------------------------------------------------------
        101120 Stock Interim Account (Received)     |       | 1.0
        ---------------------------------------------------------------
        xxxxxx Expenses                             | 1.0   |
        ---------------------------------------------------------------
        :return: A list of Python dictionary to be passed to env['account.move.line'].create.
        '''
        lines_vals_list = []
        price_unit_prec = self.env['decimal.precision'].precision_get('Product Price')

        for move in self:
            if move.move_type not in ('in_invoice', 'in_refund', 'in_receipt') or not move.company_id.anglo_saxon_accounting:
                continue

            move = move.with_company(move.company_id)
            for line in move.invoice_line_ids:
                # Filter out lines being not eligible for price difference.
                # Moreover, this function is used for standard cost method only.
                if line.product_id.type != 'product' or line.product_id.valuation != 'real_time' or line.product_id.cost_method != 'standard':
                    continue

                # Retrieve accounts needed to generate the price difference.
                debit_expense_account = line._get_price_diff_account()
                if not debit_expense_account:
                    continue
                # Retrieve stock valuation moves.
                valuation_stock_moves = self.env['stock.move'].search([
                    ('purchase_line_id', '=', line.purchase_line_id.id),
                    ('state', '=', 'done'),
                    ('product_qty', '!=', 0.0),
                ]) if line.purchase_line_id else self.env['stock.move']

                if line.product_id.cost_method != 'standard' and line.purchase_line_id:
                    if move.move_type == 'in_refund':
                        valuation_stock_moves = valuation_stock_moves.filtered(lambda stock_move: stock_move._is_out())
                    else:
                        valuation_stock_moves = valuation_stock_moves.filtered(lambda stock_move: stock_move._is_in())

                    if not valuation_stock_moves:
                        continue

                    valuation_price_unit_total, valuation_total_qty = valuation_stock_moves._get_valuation_price_and_qty(line, move.currency_id)
                    valuation_price_unit = valuation_price_unit_total / valuation_total_qty
                    valuation_price_unit = line.product_id.uom_id._compute_price(valuation_price_unit, line.product_uom_id)
                else:
                    # Valuation_price unit is always expressed in invoice currency, so that it can always be computed with the good rate
                    price_unit = line.product_id.uom_id._compute_price(line.product_id.standard_price, line.product_uom_id)
                    price_unit = -price_unit if line.move_id.move_type == 'in_refund' else price_unit
                    valuation_date = valuation_stock_moves and max(valuation_stock_moves.mapped('date')) or move.date
                    valuation_price_unit = line.company_currency_id._convert(
                        price_unit, move.currency_id,
                        move.company_id, valuation_date, round=False
                    )


                price_unit = line._get_gross_unit_price()

                price_unit_val_dif = price_unit - valuation_price_unit
                # If there are some valued moves, we only consider their quantity already used
                if line.product_id.cost_method == 'standard':
                    relevant_qty = line.quantity
                else:
                    relevant_qty = line._get_out_and_not_invoiced_qty(valuation_stock_moves)
                price_subtotal = relevant_qty * price_unit_val_dif

                # We consider there is a price difference if the subtotal is not zero. In case a
                # discount has been applied, we can't round the price unit anymore, and hence we
                # can't compare them.
                if (
                    not move.currency_id.is_zero(price_subtotal)
                    and float_compare(line["price_unit"], line.price_unit, precision_digits=price_unit_prec) == 0
                ):

                    # Add price difference account line.
                    vals = {
                        'name': line.name[:64],
                        'move_id': move.id,
                        'partner_id': line.partner_id.id or move.commercial_partner_id.id,
                        'currency_id': line.currency_id.id,
                        'product_id': line.product_id.id,
                        'product_uom_id': line.product_uom_id.id,
                        'quantity': relevant_qty,
                        'price_unit': price_unit_val_dif,
                        'price_subtotal': relevant_qty * price_unit_val_dif,
                        'amount_currency': relevant_qty * price_unit_val_dif * line.move_id.direction_sign,
                        'balance': line.currency_id._convert(
                            relevant_qty * price_unit_val_dif * line.move_id.direction_sign,
                            line.company_currency_id,
                            line.company_id, fields.Date.today(),
                        ),
                        'account_id': debit_expense_account.id,
                        'analytic_distribution': line.analytic_distribution,
                        'display_type': 'cogs',
                    }
                    lines_vals_list.append(vals)

                    # Correct the amount of the current line.
                    vals = {
                        'name': line.name[:64],
                        'move_id': move.id,
                        'partner_id': line.partner_id.id or move.commercial_partner_id.id,
                        'currency_id': line.currency_id.id,
                        'product_id': line.product_id.id,
                        'product_uom_id': line.product_uom_id.id,
                        'quantity': relevant_qty,
                        'price_unit': -price_unit_val_dif,
                        'price_subtotal': relevant_qty * -price_unit_val_dif,
                        'amount_currency': relevant_qty * -price_unit_val_dif * line.move_id.direction_sign,
                        'balance': line.currency_id._convert(
                            relevant_qty * -price_unit_val_dif * line.move_id.direction_sign,
                            line.company_currency_id,
                            line.company_id, fields.Date.today(),
                        ),
                        'account_id': line.account_id.id,
                        'analytic_distribution': line.analytic_distribution,
                        'display_type': 'cogs',
                    }
                    lines_vals_list.append(vals)
        return lines_vals_list

    def _post(self, soft=True):
        if not self._context.get('move_reverse_cancel'):
            self.env['account.move.line'].create(self._stock_account_prepare_anglo_saxon_in_lines_vals())

        # Create correction layer and impact accounts if invoice price is different
        stock_valuation_layers = self.env['stock.valuation.layer'].sudo()
        valued_lines = self.env['account.move.line'].sudo()
        for invoice in self:
            if invoice.sudo().stock_valuation_layer_ids:
                continue
            if invoice.move_type in ('in_invoice', 'in_refund', 'in_receipt'):
                valued_lines |= invoice.invoice_line_ids.filtered(
                    lambda l: l.product_id and l.product_id.cost_method != 'standard')
        if valued_lines:
            svls, _amls = valued_lines._apply_price_difference()
            stock_valuation_layers |= svls

        for (product, company), dummy in groupby(stock_valuation_layers, key=lambda svl: (svl.product_id, svl.company_id)):
            product = product.with_company(company.id)
            if not float_is_zero(product.quantity_svl, precision_rounding=product.uom_id.rounding):
                product.sudo().with_context(disable_auto_svl=True).write({'standard_price': product.value_svl / product.quantity_svl})

        posted = super(AccountMove, self.with_context(skip_cogs_reconciliation=True))._post(soft)

        # The invoice reference is set during the super call
        for layer in stock_valuation_layers:
            description = f"{layer.account_move_line_id.move_id.display_name} - {layer.product_id.display_name}"
            layer.description = description

        if stock_valuation_layers:
            stock_valuation_layers._validate_accounting_entries()

        self._stock_account_anglo_saxon_reconcile_valuation()

        return posted

    def _stock_account_get_last_step_stock_moves(self):
        """ Overridden from stock_account.
        Returns the stock moves associated to this invoice."""
        rslt = super(AccountMove, self)._stock_account_get_last_step_stock_moves()
        for invoice in self.filtered(lambda x: x.move_type == 'in_invoice'):
            rslt += invoice.mapped('invoice_line_ids.purchase_line_id.move_ids').filtered(lambda x: x.state == 'done' and x.location_id.usage == 'supplier')
        for invoice in self.filtered(lambda x: x.move_type == 'in_refund'):
            rslt += invoice.mapped('invoice_line_ids.purchase_line_id.move_ids').filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'supplier')
        return rslt

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools.float_utils import float_compare, float_is_zero

from collections import defaultdict


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _get_valued_in_moves(self):
        self.ensure_one()
        return self.purchase_line_id.move_ids.filtered(
            lambda m: m.state == 'done' and m.product_qty != 0)

    def _get_out_and_not_invoiced_qty(self, in_moves):
        self.ensure_one()
        if not in_moves:
            return 0
        aml_qty = self.product_uom_id._compute_quantity(self.quantity, self.product_id.uom_id)
        invoiced_qty = sum(line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id)
                           for line in self.purchase_line_id.invoice_lines - self)
        layers = in_moves.stock_valuation_layer_ids
        layers_qty = sum(layers.mapped('quantity'))
        out_qty = layers_qty - sum(layers.mapped('remaining_qty'))
        total_out_and_not_invoiced_qty = max(0, out_qty - invoiced_qty)
        out_and_not_invoiced_qty = min(aml_qty, total_out_and_not_invoiced_qty)
        return self.product_id.uom_id._compute_quantity(out_and_not_invoiced_qty, self.product_uom_id)

    def _get_price_diff_account(self):
        self.ensure_one()
        if self.product_id.cost_method == 'standard':
            return False
        accounts = self.product_id.product_tmpl_id.get_product_accounts(fiscal_pos=self.move_id.fiscal_position_id)
        return accounts['expense']

    def _create_in_invoice_svl(self):
        # TODO master delete (dead code)
        svl_vals_list = []
        for line in self:
            line = line.with_company(line.company_id)
            move = line.move_id.with_company(line.move_id.company_id)
            po_line = line.purchase_line_id
            uom = line.product_uom_id or line.product_id.uom_id

            # Don't create value for more quantity than received
            quantity = po_line.qty_received - (po_line.qty_invoiced - line.quantity)
            quantity = max(min(line.quantity, quantity), 0)
            if float_is_zero(quantity, precision_rounding=uom.rounding):
                continue

            layers = line._get_stock_valuation_layers(move)
            # Retrieves SVL linked to a return.
            if not layers:
                continue

            price_unit = line._get_gross_unit_price()
            price_unit = line.currency_id._convert(price_unit, line.company_id.currency_id, line.company_id, line.date, round=False)
            price_unit = line.product_uom_id._compute_price(price_unit, line.product_id.uom_id)
            layers_price_unit = line._get_stock_valuation_layers_price_unit(layers)
            layers_to_correct = line._get_stock_layer_price_difference(layers, layers_price_unit, price_unit)
            svl_vals_list += line._prepare_in_invoice_svl_vals(layers_to_correct)
        return self.env['stock.valuation.layer'].sudo().create(svl_vals_list)

    def _get_stock_valuation_layers_price_unit(self, layers):
        # TODO master delete (dead code)
        price_unit_by_layer = {}
        for layer in layers:
            price_unit_by_layer[layer] = layer.value / layer.quantity
        return price_unit_by_layer

    def _get_stock_layer_price_difference(self, layers, layers_price_unit, price_unit):
        # TODO master delete (dead code)
        self.ensure_one()
        po_line = self.purchase_line_id
        aml_qty = self.product_uom_id._compute_quantity(self.quantity, self.product_id.uom_id)
        invoice_lines = po_line.invoice_lines - self
        invoices_qty = 0
        for invoice_line in invoice_lines:
            invoices_qty += invoice_line.product_uom_id._compute_quantity(invoice_line.quantity, invoice_line.product_id.uom_id)
        qty_received = po_line.product_uom._compute_quantity(po_line.qty_received, self.product_id.uom_id)
        out_qty = qty_received - sum(layers.mapped('remaining_qty'))
        out_and_not_billed_qty = max(0, out_qty - invoices_qty)
        total_to_correct = max(0, aml_qty - out_and_not_billed_qty)
        # we also need to skip the remaining qty that is already billed
        total_to_skip = max(0, invoices_qty - out_qty)
        layers_to_correct = {}
        for layer in layers:
            if float_compare(total_to_correct, 0, precision_rounding=self.product_id.uom_id.rounding) <= 0:
                break
            remaining_qty = layer.remaining_qty
            qty_to_skip = min(total_to_skip, remaining_qty)
            remaining_qty = max(0, remaining_qty - qty_to_skip)
            qty_to_correct = min(total_to_correct, remaining_qty)
            total_to_skip -= qty_to_skip
            total_to_correct -= qty_to_correct
            unit_valuation_difference = price_unit - layers_price_unit[layer]
            if float_is_zero(unit_valuation_difference * qty_to_correct, precision_rounding=self.company_id.currency_id.rounding):
                continue
            po_pu_curr = po_line.currency_id._convert(po_line.price_unit, self.currency_id, self.company_id, self.date, round=False)
            price_difference_curr = po_pu_curr - self._get_gross_unit_price()
            layers_to_correct[layer] = (qty_to_correct, unit_valuation_difference, price_difference_curr)
        return layers_to_correct

    def _prepare_in_invoice_svl_vals(self, layers_correction):
        # TODO master delete (dead code)
        svl_vals_list = []
        invoiced_qty = self.quantity
        common_svl_vals = {
            'account_move_id': self.move_id.id,
            'account_move_line_id': self.id,
            'company_id': self.company_id.id,
            'product_id': self.product_id.id,
            'quantity': 0,
            'unit_cost': 0,
            'remaining_qty': 0,
            'remaining_value': 0,
            'description': self.move_id.name and '%s - %s' % (self.move_id.name, self.product_id.name) or self.product_id.name,
        }
        for layer, (quantity, price_difference, price_difference_curr) in layers_correction.items():
            svl_vals = self.product_id._prepare_in_svl_vals(quantity, price_difference)
            diff_value_curr = self.currency_id.round(price_difference_curr * quantity)
            svl_vals.update(**common_svl_vals, stock_valuation_layer_id=layer.id, price_diff_value=diff_value_curr)
            svl_vals_list.append(svl_vals)
            # Adds the difference into the last SVL's remaining value.
            layer.remaining_value += svl_vals['value']
            if float_compare(invoiced_qty, 0, self.product_id.uom_id.rounding) <= 0:
                break

        return svl_vals_list

    def _apply_price_difference(self):
        svl_vals_list = []
        aml_vals_list = []
        for line in self:
            line = line.with_company(line.company_id)
            po_line = line.purchase_line_id
            uom = line.product_uom_id or line.product_id.uom_id

            # Don't create value for more quantity than received
            quantity = po_line.qty_received - (po_line.qty_invoiced - line.quantity)
            quantity = max(min(line.quantity, quantity), 0)
            if float_is_zero(quantity, precision_rounding=uom.rounding):
                continue

            layers = line._get_valued_in_moves().stock_valuation_layer_ids.filtered(lambda svl: svl.product_id == line.product_id and not svl.stock_valuation_layer_id)
            if not layers:
                continue

            new_svl_vals_list, new_aml_vals_list = line._generate_price_difference_vals(layers)
            svl_vals_list += new_svl_vals_list
            aml_vals_list += new_aml_vals_list
        return self.env['stock.valuation.layer'].sudo().create(svl_vals_list), self.env['account.move.line'].sudo().create(aml_vals_list)

    def _generate_price_difference_vals(self, layers):
        """
        The method will determine which layers are impacted by the AML (`self`) and, in case of a price difference, it
        will then return the values of the new AMLs and SVLs
        """
        self.ensure_one()
        po_line = self.purchase_line_id
        product_uom = self.product_id.uom_id

        # `history` is a list of tuples: (time, aml, layer)
        # aml and layer will never be both defined
        # we use this to get an order between posted AML and layers
        history = [(layer.create_date, False, layer) for layer in layers]
        am_state_field = self.env['ir.model.fields'].search([('model', '=', 'account.move'), ('name', '=', 'state')], limit=1)
        for aml in po_line.invoice_lines:
            move = aml.move_id
            if move.state != 'posted':
                continue
            state_trackings = move.message_ids.tracking_value_ids.filtered(lambda t: t.field == am_state_field).sorted('id')
            time = state_trackings[-1:].create_date or move.create_date  # `or` in case it has been created in posted state
            history.append((time, aml, False))
        # Sort history based on the datetime. In case of equality, the prority is given to SVLs, then to IDs.
        # That way, we ensure a deterministic behaviour
        history.sort(key=lambda item: (item[0], bool(item[1]), (item[1] or item[2]).id))

        # the next dict is a matrix [layer L, invoice I] where each cell gives two info:
        # [initial qty of L invoiced by I, remaining invoiced qty]
        # the second info is usefull in case of a refund
        layers_and_invoices_qties = defaultdict(lambda: [0, 0])

        # the next dict will also provide two info:
        # [total qty to invoice, remaining qty to invoice]
        # we need the total qty to invoice, so we will be able to deduce the invoiced qty before `self`
        qty_to_invoice_per_layer = defaultdict(lambda: [0, 0])

        # Replay the whole history: we want to know what are the links between each layer and each invoice,
        # and then the links between `self` and the layers
        history.append((False, self, False))  # time was only usefull for the sorting
        for _time, aml, layer in history:
            if layer:
                total_layer_qty_to_invoice = abs(layer.quantity)
                initial_layer = layer.stock_move_id.origin_returned_move_id.stock_valuation_layer_ids
                if initial_layer:
                    # `layer` is a return. We will cancel the qty to invoice of the returned layer
                    # /!\ we will cancel the qty not yet invoiced only
                    initial_layer_remaining_qty = qty_to_invoice_per_layer[initial_layer][1]
                    common_qty = min(initial_layer_remaining_qty, total_layer_qty_to_invoice)
                    qty_to_invoice_per_layer[initial_layer][0] -= common_qty
                    qty_to_invoice_per_layer[initial_layer][1] -= common_qty
                    total_layer_qty_to_invoice = max(0, total_layer_qty_to_invoice - common_qty)
                if float_compare(total_layer_qty_to_invoice, 0, precision_rounding=product_uom.rounding) > 0:
                    qty_to_invoice_per_layer[layer] = [total_layer_qty_to_invoice, total_layer_qty_to_invoice]
            else:
                invoice = aml.move_id
                impacted_invoice = False
                aml_qty = aml.product_uom_id._compute_quantity(aml.quantity, product_uom)
                if aml.is_refund:
                    reversed_invoice = aml.move_id.reversed_entry_id
                    if reversed_invoice:
                        sign = -1
                        impacted_invoice = reversed_invoice
                        # it's a refund, therefore we can only consume the quantities invoiced by
                        # the initial invoice (`reversed_invoice`)
                        layers_to_consume = []
                        for layer in layers:
                            remaining_invoiced_qty = layers_and_invoices_qties[(layer, reversed_invoice)][1]
                            layers_to_consume.append((layer, remaining_invoiced_qty))
                    else:
                        # the refund has been generated because of a stock return, let's find and use it
                        sign = 1
                        layers_to_consume = []
                        for layer in qty_to_invoice_per_layer:
                            if layer.stock_move_id._is_out():
                                layers_to_consume.append((layer, qty_to_invoice_per_layer[layer][1]))
                else:
                    # classic case, we are billing a received quantity so let's use the incoming SVLs
                    sign = 1
                    layers_to_consume = []
                    for layer in qty_to_invoice_per_layer:
                        if layer.stock_move_id._is_in():
                            layers_to_consume.append((layer, qty_to_invoice_per_layer[layer][1]))
                while float_compare(aml_qty, 0, precision_rounding=product_uom.rounding) > 0 and layers_to_consume:
                    layer, total_layer_qty_to_invoice = layers_to_consume[0]
                    layers_to_consume = layers_to_consume[1:]
                    if float_is_zero(total_layer_qty_to_invoice, precision_rounding=product_uom.rounding):
                        continue
                    common_qty = min(aml_qty, total_layer_qty_to_invoice)
                    aml_qty -= common_qty
                    qty_to_invoice_per_layer[layer][1] -= sign * common_qty
                    layers_and_invoices_qties[(layer, invoice)] = [common_qty, common_qty]
                    layers_and_invoices_qties[(layer, impacted_invoice)][1] -= common_qty

        # Now we know what layers does `self` use, let's check if we have to create a pdiff SVL
        # (or cancel such an SVL in case of a refund)
        invoice = self.move_id
        svl_vals_list = []
        aml_vals_list = []
        for layer in layers:
            # use the link between `self` and `layer` (i.e. the qty of `layer` billed by `self`)
            invoicing_layer_qty = layers_and_invoices_qties[(layer, invoice)][1]
            if float_is_zero(invoicing_layer_qty, precision_rounding=product_uom.rounding):
                continue
            # We will only consider the total quantity to invoice of the layer because we don't
            # want to invoice a part of the layer that has not been invoiced and that has been
            # returned in the meantime
            total_layer_qty_to_invoice = qty_to_invoice_per_layer[layer][0]
            remaining_qty = layer.remaining_qty
            out_layer_qty = total_layer_qty_to_invoice - remaining_qty
            if self.is_refund:
                sign = -1
                reversed_invoice = invoice.reversed_entry_id
                if not reversed_invoice:
                    # this is a refund for a returned quantity, we don't have anything to do
                    continue
                initial_invoiced_qty = layers_and_invoices_qties[(layer, reversed_invoice)][0]
                initial_pdiff_svl = layer.stock_valuation_layer_ids.filtered(lambda svl: svl.account_move_line_id.move_id == reversed_invoice)
                if not initial_pdiff_svl or float_is_zero(initial_invoiced_qty, precision_rounding=product_uom.rounding):
                    continue
                # We have an already-out quantity: we must skip the part already invoiced. So, first,
                # let's compute the already invoiced quantity...
                previously_invoiced_qty = 0
                for item in history:
                    previous_aml = item[1]
                    if not previous_aml or previous_aml.is_refund:
                        continue
                    previous_invoice = previous_aml.move_id
                    if previous_invoice == reversed_invoice:
                        break
                    previously_invoiced_qty += layers_and_invoices_qties[(layer, previous_invoice,)][1]
                # ... Second, skip it:
                out_qty_to_invoice = max(0, out_layer_qty - previously_invoiced_qty)
                qty_to_correct = max(0, invoicing_layer_qty - out_qty_to_invoice)
                if out_qty_to_invoice:
                    # In case the out qty is different from the one posted by the initial bill, we should compensate
                    # this quantity with debit/credit between stock_in and expense, but we are reversing an initial
                    # invoice and don't want to do more than the original one
                    out_qty_to_invoice = 0
                aml = initial_pdiff_svl.account_move_line_id
                parent_layer = initial_pdiff_svl.stock_valuation_layer_id
                layer_price_unit = parent_layer.value / parent_layer.quantity
            else:
                sign = 1
                # get the invoiced qty of the layer without considering `self`
                invoiced_layer_qty = total_layer_qty_to_invoice - qty_to_invoice_per_layer[layer][1] - invoicing_layer_qty
                remaining_out_qty_to_invoice = max(0, out_layer_qty - invoiced_layer_qty)
                out_qty_to_invoice = min(remaining_out_qty_to_invoice, invoicing_layer_qty)
                qty_to_correct = invoicing_layer_qty - out_qty_to_invoice
                layer_price_unit = layer.value / layer.quantity

                returned_move = layer.stock_move_id.origin_returned_move_id
                if returned_move and returned_move._is_out() and returned_move._is_returned(valued_type='out'):
                    # Odd case! The user receives a product, then returns it. The returns are processed as classic
                    # output, so the value of the returned product can be different from the initial one. The user
                    # then receives again the returned product (that's where we are here) -> the SVL is based on
                    # the returned one, the accounting entries are already compensated, and we don't want to impact
                    # the stock valuation. So, let's fake the layer price unit with the POL one as everything is
                    # already ok
                    layer_price_unit = po_line.currency_id._convert(
                        po_line._get_gross_price_unit(),
                        layer.currency_id,
                        layer.company_id,
                        layer.create_date.date(),
                        round=False
                    )

                aml = self

            aml_gross_price_unit = aml._get_gross_unit_price()
            # convert from aml currency to company currency
            aml_price_unit = aml_gross_price_unit / aml.currency_rate
            aml_price_unit = aml.product_uom_id._compute_price(aml_price_unit, product_uom)

            unit_valuation_difference = aml_price_unit - layer_price_unit

            # Generate the AML values for the already out quantities
            # convert from company currency to aml currency
            unit_valuation_difference_curr = unit_valuation_difference * self.currency_rate
            unit_valuation_difference_curr = product_uom._compute_price(unit_valuation_difference_curr, self.product_uom_id)
            out_qty_to_invoice = product_uom._compute_quantity(out_qty_to_invoice, self.product_uom_id)
            if (
                not self.currency_id.is_zero(unit_valuation_difference_curr * out_qty_to_invoice) and
                self.product_id.valuation == 'real_time'
            ):
                aml_vals_list += self._prepare_pdiff_aml_vals(out_qty_to_invoice, unit_valuation_difference_curr)

            # Generate the SVL values for the on hand quantities (and impact the parent layer)
            po_pu_curr = po_line.currency_id._convert(po_line.price_unit, self.currency_id, self.company_id, self.move_id.invoice_date or self.date or fields.Date.context_today(self), round=False)
            price_difference_curr = po_pu_curr - aml_gross_price_unit
            if not float_is_zero(unit_valuation_difference * qty_to_correct, precision_rounding=self.company_id.currency_id.rounding):
                svl_vals = self._prepare_pdiff_svl_vals(layer, sign * qty_to_correct, unit_valuation_difference, price_difference_curr)
                layer.remaining_value += svl_vals['value']
                svl_vals_list.append(svl_vals)
        return svl_vals_list, aml_vals_list

    def _prepare_pdiff_aml_vals(self, qty, unit_valuation_difference):
        self.ensure_one()
        vals_list = []

        sign = self.move_id.direction_sign
        expense_account = self._get_price_diff_account()
        if not expense_account:
            return vals_list

        for price, account in [
            (unit_valuation_difference, expense_account),
            (-unit_valuation_difference, self.account_id),
        ]:
            vals_list.append({
                'name': self.name[:64],
                'move_id': self.move_id.id,
                'partner_id': self.partner_id.id or self.move_id.commercial_partner_id.id,
                'currency_id': self.currency_id.id,
                'product_id': self.product_id.id,
                'product_uom_id': self.product_uom_id.id,
                'balance': self.company_id.currency_id.round((qty * price * sign) / self.currency_rate),
                'account_id': account.id,
                'analytic_distribution': self.analytic_distribution,
                'display_type': 'cogs',
            })
        return vals_list

    def _prepare_pdiff_svl_vals(self, corrected_layer, quantity, unit_cost, pdiff):
        self.ensure_one()
        common_svl_vals = {
            'account_move_id': self.move_id.id,
            'account_move_line_id': self.id,
            'company_id': self.company_id.id,
            'product_id': self.product_id.id,
            'quantity': 0,
            'unit_cost': 0,
            'remaining_qty': 0,
            'remaining_value': 0,
            'description': self.move_id.name and '%s - %s' % (self.move_id.name, self.product_id.name) or self.product_id.name,
        }
        return {
            **self.product_id._prepare_in_svl_vals(quantity, unit_cost),
            **common_svl_vals,
            'stock_valuation_layer_id': corrected_layer.id,
            'price_diff_value': self.currency_id.round(pdiff * quantity),
        }

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


class ProductTemplate(models.Model):
    _name = 'product.template'
    _inherit = 'product.template'

    @api.model
    def _get_buy_route(self):
        buy_route = self.env.ref('purchase_stock.route_warehouse0_buy', raise_if_not_found=False)
        if buy_route:
            return self.env['stock.route'].search([('id', '=', buy_route.id)]).ids
        return []

    route_ids = fields.Many2many(default=lambda self: self._get_buy_route())


class ProductProduct(models.Model):
    _name = 'product.product'
    _inherit = 'product.product'

    purchase_order_line_ids = fields.One2many('purchase.order.line', 'product_id', string="PO Lines") # used to compute quantities

    def _get_quantity_in_progress(self, location_ids=False, warehouse_ids=False):
        if not location_ids:
            location_ids = []
        if not warehouse_ids:
            warehouse_ids = []

        qty_by_product_location, qty_by_product_wh = super()._get_quantity_in_progress(location_ids, warehouse_ids)
        domain = self._get_lines_domain(location_ids, warehouse_ids)
        groups = self.env['purchase.order.line']._read_group(domain,
            ['product_id', 'product_qty', 'order_id', 'product_uom', 'orderpoint_id'],
            ['order_id', 'product_id', 'product_uom', 'orderpoint_id'], lazy=False)
        for group in groups:
            if group.get('orderpoint_id'):
                location = self.env['stock.warehouse.orderpoint'].browse(group['orderpoint_id'][:1]).location_id
            else:
                order = self.env['purchase.order'].browse(group['order_id'][0])
                location = order.picking_type_id.default_location_dest_id
            product = self.env['product.product'].browse(group['product_id'][0])
            uom = self.env['uom.uom'].browse(group['product_uom'][0])
            product_qty = uom._compute_quantity(group['product_qty'], product.uom_id, round=False)
            qty_by_product_location[(product.id, location.id)] += product_qty
            qty_by_product_wh[(product.id, location.warehouse_id.id)] += product_qty
        return qty_by_product_location, qty_by_product_wh

    def _get_lines_domain(self, location_ids=False, warehouse_ids=False):
        domain = []
        rfq_domain = [
            ('state', 'in', ('draft', 'sent', 'to approve')),
            ('product_id', 'in', self.ids)
        ]
        if location_ids:
            domain = expression.AND([rfq_domain, [
                '|',
                    ('order_id.picking_type_id.default_location_dest_id', 'in', location_ids),
                    '&',
                        ('move_dest_ids', '=', False),
                        ('orderpoint_id.location_id', 'in', location_ids)
            ]])
        if warehouse_ids:
            wh_domain = expression.AND([rfq_domain, [
                '|',
                    ('order_id.picking_type_id.warehouse_id', 'in', warehouse_ids),
                    '&',
                        ('move_dest_ids', '=', False),
                        ('orderpoint_id.warehouse_id', 'in', warehouse_ids)
            ]])
            domain = expression.OR([domain, wh_domain])
        return domain


class SupplierInfo(models.Model):
    _inherit = 'product.supplierinfo'

    last_purchase_date = fields.Date('Last Purchase', compute='_compute_last_purchase_date')
    show_set_supplier_button = fields.Boolean(
        'Show Set Supplier Button', compute='_compute_show_set_supplier_button')

    def _compute_last_purchase_date(self):
        self.last_purchase_date = False
        purchases = self.env['purchase.order'].search([
            ('state', 'in', ('purchase', 'done')),
            ('order_line.product_id', 'in',
             self.product_tmpl_id.product_variant_ids.ids),
            ('partner_id', 'in', self.partner_id.ids),
        ], order='date_order desc')
        for supplier in self:
            products = supplier.product_tmpl_id.product_variant_ids
            for purchase in purchases:
                if purchase.partner_id != supplier.partner_id:
                    continue
                if not (products & purchase.order_line.product_id):
                    continue
                supplier.last_purchase_date = purchase.date_order
                break

    def _compute_show_set_supplier_button(self):
        self.show_set_supplier_button = True
        orderpoint_id = self.env.context.get('default_orderpoint_id')
        orderpoint = self.env['stock.warehouse.orderpoint'].browse(orderpoint_id)
        if orderpoint_id:
            self.filtered(
                lambda s: s.id == orderpoint.supplier_id.id
            ).show_set_supplier_button = False

    def action_set_supplier(self):
        self.ensure_one()
        orderpoint_id = self.env.context.get('orderpoint_id')
        orderpoint = self.env['stock.warehouse.orderpoint'].browse(orderpoint_id)
        if not orderpoint:
            return
        if 'buy' not in orderpoint.route_id.rule_ids.mapped('action'):
            orderpoint.route_id = self.env['stock.rule'].search([('action', '=', 'buy')], limit=1).route_id.id
        orderpoint.supplier_id = self
        supplier_min_qty = self.product_uom._compute_quantity(self.min_qty, orderpoint.product_id.uom_id)
        if orderpoint.qty_to_order < supplier_min_qty:
            orderpoint.qty_to_order = supplier_min_qty

```

## File: models\purchase.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from markupsafe import Markup
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.tools.float_utils import float_compare, float_is_zero, float_round
from odoo.exceptions import UserError

from odoo.addons.purchase.models.purchase import PurchaseOrder as Purchase
from odoo.tools.misc import OrderedSet


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    @api.model
    def _default_picking_type(self):
        return self._get_picking_type(self.env.context.get('company_id') or self.env.company.id)

    incoterm_id = fields.Many2one('account.incoterms', 'Incoterm', states={'done': [('readonly', True)]}, help="International Commercial Terms are a series of predefined commercial terms used in international transactions.")
    incoterm_location = fields.Char(string='Incoterm Location', states={'done': [('readonly', True)]})
    incoming_picking_count = fields.Integer("Incoming Shipment count", compute='_compute_incoming_picking_count')
    picking_ids = fields.Many2many('stock.picking', compute='_compute_picking_ids', string='Receptions', copy=False, store=True)
    dest_address_id = fields.Many2one('res.partner', compute='_compute_dest_address_id', store=True, readonly=False)
    picking_type_id = fields.Many2one('stock.picking.type', 'Deliver To', states=Purchase.READONLY_STATES, required=True, default=_default_picking_type, domain="['|', ('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]",
        help="This will determine operation type of incoming shipment")
    default_location_dest_id_usage = fields.Selection(related='picking_type_id.default_location_dest_id.usage', string='Destination Location Type',
        help="Technical field used to display the Drop Ship Address", readonly=True)
    group_id = fields.Many2one('procurement.group', string="Procurement Group", copy=False)
    is_shipped = fields.Boolean(compute="_compute_is_shipped")
    effective_date = fields.Datetime("Arrival", compute='_compute_effective_date', store=True, copy=False,
        help="Completion date of the first receipt order.")
    on_time_rate = fields.Float(related='partner_id.on_time_rate', compute_sudo=False)
    receipt_status = fields.Selection([
        ('pending', 'Not Received'),
        ('partial', 'Partially Received'),
        ('full', 'Fully Received'),
    ], string='Receipt Status', compute='_compute_receipt_status', store=True)

    @api.depends('order_line.move_ids.picking_id')
    def _compute_picking_ids(self):
        for order in self:
            order.picking_ids = order.order_line.move_ids.picking_id

    @api.depends('picking_ids')
    def _compute_incoming_picking_count(self):
        for order in self:
            order.incoming_picking_count = len(order.picking_ids)

    @api.depends('picking_ids.date_done')
    def _compute_effective_date(self):
        for order in self:
            pickings = order.picking_ids.filtered(lambda x: x.state == 'done' and x.location_dest_id.usage != 'supplier' and x.date_done)
            order.effective_date = min(pickings.mapped('date_done'), default=False)

    @api.depends('picking_ids', 'picking_ids.state')
    def _compute_is_shipped(self):
        for order in self:
            if order.picking_ids and all(x.state in ['done', 'cancel'] for x in order.picking_ids):
                order.is_shipped = True
            else:
                order.is_shipped = False

    @api.depends('picking_ids', 'picking_ids.state')
    def _compute_receipt_status(self):
        for order in self:
            if not order.picking_ids or all(p.state == 'cancel' for p in order.picking_ids):
                order.receipt_status = False
            elif all(p.state in ['done', 'cancel'] for p in order.picking_ids):
                order.receipt_status = 'full'
            elif any(p.state == 'done' for p in order.picking_ids):
                order.receipt_status = 'partial'
            else:
                order.receipt_status = 'pending'

    @api.depends('picking_type_id')
    def _compute_dest_address_id(self):
        self.filtered(lambda po: po.picking_type_id.default_location_dest_id.usage != 'customer').dest_address_id = False

    @api.onchange('company_id')
    def _onchange_company_id(self):
        p_type = self.picking_type_id
        if not(p_type and p_type.code == 'incoming' and (p_type.warehouse_id.company_id == self.company_id or not p_type.warehouse_id)):
            self.picking_type_id = self._get_picking_type(self.company_id.id)

    # --------------------------------------------------
    # CRUD
    # --------------------------------------------------

    def write(self, vals):
        if vals.get('order_line') and self.state == 'purchase':
            for order in self:
                pre_order_line_qty = {order_line: order_line.product_qty for order_line in order.mapped('order_line')}
        res = super(PurchaseOrder, self).write(vals)
        if vals.get('order_line') and self.state == 'purchase':
            for order in self:
                to_log = {}
                for order_line in order.order_line:
                    if pre_order_line_qty.get(order_line, False) and float_compare(pre_order_line_qty[order_line], order_line.product_qty, precision_rounding=order_line.product_uom.rounding) > 0:
                        to_log[order_line] = (order_line.product_qty, pre_order_line_qty[order_line])
                if to_log:
                    order._log_decrease_ordered_quantity(to_log)
        return res

    # --------------------------------------------------
    # Actions
    # --------------------------------------------------

    def button_approve(self, force=False):
        result = super(PurchaseOrder, self).button_approve(force=force)
        self._create_picking()
        return result

    def button_cancel(self):
        order_lines_ids = OrderedSet()
        pickings_to_cancel_ids = OrderedSet()
        for order in self:
            for move in order.order_line.mapped('move_ids'):
                if move.state == 'done':
                    raise UserError(_('Unable to cancel purchase order %s as some receptions have already been done.') % (order.name))
            # If the product is MTO, change the procure_method of the closest move to purchase to MTS.
            # The purpose is to link the po that the user will manually generate to the existing moves's chain.
            if order.state in ('draft', 'sent', 'to approve', 'purchase'):
                order_lines_ids.update(order.order_line.ids)

            pickings_to_cancel_ids.update(order.picking_ids.filtered(lambda r: r.state != 'cancel').ids)

        order_lines = self.env['purchase.order.line'].browse(order_lines_ids)

        moves_to_cancel_ids = OrderedSet()
        moves_to_recompute_ids = OrderedSet()
        for order_line in order_lines:
            moves_to_cancel_ids.update(order_line.move_ids.ids)
            if order_line.move_dest_ids:
                move_dest_ids = order_line.move_dest_ids.filtered(lambda move: move.state != 'done' and not move.scrapped)
                if order_line.propagate_cancel:
                    moves_to_cancel_ids.update(move_dest_ids.ids)
                else:
                    moves_to_recompute_ids.update(move_dest_ids.ids)

        if moves_to_cancel_ids:
            moves_to_cancel = self.env['stock.move'].browse(moves_to_cancel_ids)
            moves_to_cancel._action_cancel()

        if moves_to_recompute_ids:
            moves_to_recompute = self.env['stock.move'].browse(moves_to_recompute_ids)
            moves_to_recompute.write({'procure_method': 'make_to_stock'})
            moves_to_recompute._recompute_state()

        if pickings_to_cancel_ids:
            pikings_to_cancel = self.env['stock.picking'].browse(pickings_to_cancel_ids)
            pikings_to_cancel.action_cancel()

        if order_lines:
            order_lines.write({'move_dest_ids': [(5, 0, 0)]})

        return super(PurchaseOrder, self).button_cancel()

    def action_view_picking(self):
        return self._get_action_view_picking(self.picking_ids)

    def _get_action_view_picking(self, pickings):
        """ This function returns an action that display existing picking orders of given purchase order ids. When only one found, show the picking immediately.
        """
        self.ensure_one()
        result = self.env["ir.actions.actions"]._for_xml_id('stock.action_picking_tree_all')
        # override the context to get rid of the default filtering on operation type
        result['context'] = {'default_partner_id': self.partner_id.id, 'default_origin': self.name, 'default_picking_type_id': self.picking_type_id.id}
        # choose the view_mode accordingly
        if not pickings or len(pickings) > 1:
            result['domain'] = [('id', 'in', pickings.ids)]
        elif len(pickings) == 1:
            res = self.env.ref('stock.view_picking_form', False)
            form_view = [(res and res.id or False, 'form')]
            result['views'] = form_view + [(state, view) for state, view in result.get('views', []) if view != 'form']
            result['res_id'] = pickings.id
        return result

    def _prepare_invoice(self):
        invoice_vals = super()._prepare_invoice()
        invoice_vals['invoice_incoterm_id'] = self.incoterm_id.id
        return invoice_vals

    # --------------------------------------------------
    # Business methods
    # --------------------------------------------------

    def _log_decrease_ordered_quantity(self, purchase_order_lines_quantities):

        def _keys_in_groupby(move):
            """ group by picking and the responsible for the product the
            move.
            """
            return (move.picking_id, move.product_id.responsible_id)

        def _render_note_exception_quantity_po(order_exceptions):
            order_line_ids = self.env['purchase.order.line'].browse([order_line.id for order in order_exceptions.values() for order_line in order[0]])
            purchase_order_ids = order_line_ids.mapped('order_id')
            move_ids = self.env['stock.move'].concat(*rendering_context.keys())
            impacted_pickings = move_ids.mapped('picking_id')._get_impacted_pickings(move_ids) - move_ids.mapped('picking_id')
            values = {
                'purchase_order_ids': purchase_order_ids,
                'order_exceptions': order_exceptions.values(),
                'impacted_pickings': impacted_pickings,
            }
            return self.env['ir.qweb']._render('purchase_stock.exception_on_po', values)

        documents = self.env['stock.picking']._log_activity_get_documents(purchase_order_lines_quantities, 'move_ids', 'DOWN', _keys_in_groupby)
        filtered_documents = {}
        for (parent, responsible), rendering_context in documents.items():
            if parent._name == 'stock.picking':
                if parent.state in ['cancel', 'done']:
                    continue
            filtered_documents[(parent, responsible)] = rendering_context
        self.env['stock.picking']._log_activity(_render_note_exception_quantity_po, filtered_documents)

    def _get_destination_location(self):
        self.ensure_one()
        if self.dest_address_id:
            return self.dest_address_id.property_stock_customer.id
        return self.picking_type_id.default_location_dest_id.id

    @api.model
    def _get_picking_type(self, company_id):
        picking_type = self.env['stock.picking.type'].search([('code', '=', 'incoming'), ('warehouse_id.company_id', '=', company_id)])
        if not picking_type:
            picking_type = self.env['stock.picking.type'].search([('code', '=', 'incoming'), ('warehouse_id', '=', False)])
        return picking_type[:1]

    def _prepare_picking(self):
        if not self.group_id:
            self.group_id = self.group_id.create({
                'name': self.name,
                'partner_id': self.partner_id.id
            })
        if not self.partner_id.property_stock_supplier.id:
            raise UserError(_("You must set a Vendor Location for this partner %s", self.partner_id.name))
        return {
            'picking_type_id': self.picking_type_id.id,
            'partner_id': self.partner_id.id,
            'user_id': False,
            'date': self.date_order,
            'origin': self.name,
            'location_dest_id': self._get_destination_location(),
            'location_id': self.partner_id.property_stock_supplier.id,
            'company_id': self.company_id.id,
        }

    def _create_picking(self):
        StockPicking = self.env['stock.picking']
        for order in self.filtered(lambda po: po.state in ('purchase', 'done')):
            if any(product.type in ['product', 'consu'] for product in order.order_line.product_id):
                order = order.with_company(order.company_id)
                pickings = order.picking_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
                if not pickings:
                    res = order._prepare_picking()
                    picking = StockPicking.with_user(SUPERUSER_ID).create(res)
                    pickings = picking
                else:
                    picking = pickings[0]
                moves = order.order_line._create_stock_moves(picking)
                moves = moves.filtered(lambda x: x.state not in ('done', 'cancel'))._action_confirm()
                seq = 0
                for move in sorted(moves, key=lambda move: move.date):
                    seq += 5
                    move.sequence = seq
                moves._action_assign()
                # Get following pickings (created by push rules) to confirm them as well.
                forward_pickings = self.env['stock.picking']._get_impacted_pickings(moves)
                (pickings | forward_pickings).action_confirm()
                picking.message_post_with_view('mail.message_origin_link',
                    values={'self': picking, 'origin': order},
                    subtype_id=self.env.ref('mail.mt_note').id)
        return True

    def _add_picking_info(self, activity):
        """Helper method to add picking info to the Date Updated activity when
        vender updates date_planned of the po lines.
        """
        validated_picking = self.picking_ids.filtered(lambda p: p.state == 'done')
        if validated_picking:
            message = _("Those dates couldn’t be modified accordingly on the receipt %s which had already been validated.", validated_picking[0].name)
        elif not self.picking_ids:
            message = _("Corresponding receipt not found.")
        else:
            message = _("Those dates have been updated accordingly on the receipt %s.", self.picking_ids[0].name)
        activity.note += Markup('<p>{}</p>').format(message)

    def _create_update_date_activity(self, updated_dates):
        activity = super()._create_update_date_activity(updated_dates)
        self._add_picking_info(activity)

    def _update_update_date_activity(self, updated_dates, activity):
        # remove old picking info to update it
        note_lines = activity.note.split('<p>')
        note_lines.pop()
        activity.note = Markup('<p>').join(note_lines)
        super()._update_update_date_activity(updated_dates, activity)
        self._add_picking_info(activity)

    @api.model
    def _get_orders_to_remind(self):
        """When auto sending reminder mails, don't send for purchase order with
        validated receipts."""
        return super()._get_orders_to_remind().filtered(lambda p: not p.effective_date)


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    def _ondelete_stock_moves(self):
        modified_fields = ['qty_received_manual', 'qty_received_method']
        self.flush_recordset(fnames=['qty_received', *modified_fields])
        self.invalidate_recordset(fnames=modified_fields, flush=False)
        query = f'''
            UPDATE {self._table}
            SET qty_received_manual = qty_received, qty_received_method = 'manual'
            WHERE id IN %(ids)s
        '''
        self.env.cr.execute(query, {'ids': self._ids or (None,)})
        self.modified(modified_fields)

    qty_received_method = fields.Selection(selection_add=[('stock_moves', 'Stock Moves')],
                                           ondelete={'stock_moves': _ondelete_stock_moves})

    move_ids = fields.One2many('stock.move', 'purchase_line_id', string='Reservation', readonly=True, copy=False)
    orderpoint_id = fields.Many2one('stock.warehouse.orderpoint', 'Orderpoint', copy=False, index='btree_not_null')
    move_dest_ids = fields.One2many('stock.move', 'created_purchase_line_id', 'Downstream Moves')
    product_description_variants = fields.Char('Custom Description')
    propagate_cancel = fields.Boolean('Propagate cancellation', default=True)
    forecasted_issue = fields.Boolean(compute='_compute_forecasted_issue')

    def _compute_qty_received_method(self):
        super(PurchaseOrderLine, self)._compute_qty_received_method()
        for line in self.filtered(lambda l: not l.display_type):
            if line.product_id.type in ['consu', 'product']:
                line.qty_received_method = 'stock_moves'

    def _get_po_line_moves(self):
        self.ensure_one()
        moves = self.move_ids.filtered(lambda m: m.product_id == self.product_id)
        if self._context.get('accrual_entry_date'):
            moves = moves.filtered(lambda r: fields.Date.context_today(r, r.date) <= self._context['accrual_entry_date'])
        return moves

    def _get_po_line_invoice_lines_su(self):
        #TODO remove in master: un-used
        return self.sudo().invoice_lines

    @api.depends('move_ids.state', 'move_ids.product_uom_qty', 'move_ids.product_uom')
    def _compute_qty_received(self):
        from_stock_lines = self.filtered(lambda order_line: order_line.qty_received_method == 'stock_moves')
        super(PurchaseOrderLine, self - from_stock_lines)._compute_qty_received()
        for line in self:
            if line.qty_received_method == 'stock_moves':
                total = 0.0
                # In case of a BOM in kit, the products delivered do not correspond to the products in
                # the PO. Therefore, we can skip them since they will be handled later on.
                for move in line._get_po_line_moves():
                    if move.state == 'done':
                        if move._is_purchase_return():
                            if move.to_refund:
                                total -= move.product_uom._compute_quantity(move.product_uom_qty, line.product_uom, rounding_method='HALF-UP')
                        elif move.origin_returned_move_id and move.origin_returned_move_id._is_dropshipped() and not move._is_dropshipped_returned():
                            # Edge case: the dropship is returned to the stock, no to the supplier.
                            # In this case, the received quantity on the PO is set although we didn't
                            # receive the product physically in our stock. To avoid counting the
                            # quantity twice, we do nothing.
                            pass
                        elif move.origin_returned_move_id and move.origin_returned_move_id._is_purchase_return() and not move.to_refund:
                            pass
                        else:
                            total += move.product_uom._compute_quantity(move.product_uom_qty, line.product_uom, rounding_method='HALF-UP')
                line._track_qty_received(total)
                line.qty_received = total

    @api.depends('product_uom_qty', 'date_planned')
    def _compute_forecasted_issue(self):
        for line in self:
            warehouse = line.order_id.picking_type_id.warehouse_id
            line.forecasted_issue = False
            if line.product_id:
                virtual_available = line.product_id.with_context(warehouse=warehouse.id, to_date=line.date_planned).virtual_available
                if line.state == 'draft':
                    virtual_available += line.product_uom_qty
                if virtual_available < 0:
                    line.forecasted_issue = True

    @api.model_create_multi
    def create(self, vals_list):
        lines = super(PurchaseOrderLine, self).create(vals_list)
        lines.filtered(lambda l: l.order_id.state == 'purchase')._create_or_update_picking()
        return lines

    def write(self, values):
        if values.get('date_planned'):
            new_date = fields.Datetime.to_datetime(values['date_planned'])
            self.filtered(lambda l: not l.display_type)._update_move_date_deadline(new_date)
        lines = self.filtered(lambda l: l.order_id.state == 'purchase'
                                        and not l.display_type)

        if 'product_packaging_id' in values:
            self.move_ids.filtered(
                lambda m: m.state not in ['cancel', 'done']
            ).product_packaging_id = values['product_packaging_id']

        previous_product_uom_qty = {line.id: line.product_uom_qty for line in lines}
        previous_product_qty = {line.id: line.product_qty for line in lines}
        result = super(PurchaseOrderLine, self).write(values)
        if 'price_unit' in values:
            for line in lines:
                # Avoid updating kit components' stock.move
                moves = line.move_ids.filtered(lambda s: s.state not in ('cancel', 'done') and s.product_id == line.product_id)
                moves.write({'price_unit': line._get_stock_move_price_unit()})
        if 'product_qty' in values:
            lines = lines.filtered(lambda l: float_compare(previous_product_qty[l.id], l.product_qty, precision_rounding=l.product_uom.rounding) != 0)
            lines.with_context(previous_product_qty=previous_product_uom_qty)._create_or_update_picking()
        return result

    def action_product_forecast_report(self):
        self.ensure_one()
        action = self.product_id.action_product_forecast_report()
        action['context'] = {
            'active_id': self.product_id.id,
            'active_model': 'product.product',
            'move_to_match_ids': self.move_ids.filtered(lambda m: m.product_id == self.product_id).ids,
            'purchase_line_to_match_id': self.id,
        }
        warehouse = self.order_id.picking_type_id.warehouse_id
        if warehouse:
            action['context']['warehouse'] = warehouse.id
        return action

    def unlink(self):
        self.move_ids._action_cancel()

        ppg_cancel_lines = self.filtered(lambda line: line.propagate_cancel)
        ppg_cancel_lines.move_dest_ids._action_cancel()

        not_ppg_cancel_lines = self.filtered(lambda line: not line.propagate_cancel)
        not_ppg_cancel_lines.move_dest_ids.write({'procure_method': 'make_to_stock'})
        not_ppg_cancel_lines.move_dest_ids._recompute_state()

        return super().unlink()

    # --------------------------------------------------
    # Business methods
    # --------------------------------------------------

    def _update_move_date_deadline(self, new_date):
        """ Updates corresponding move picking line deadline dates that are not yet completed. """
        moves_to_update = self.move_ids.filtered(lambda m: m.state not in ('done', 'cancel'))
        if not moves_to_update:
            moves_to_update = self.move_dest_ids.filtered(lambda m: m.state not in ('done', 'cancel'))
        for move in moves_to_update:
            move.date_deadline = new_date

    def _create_or_update_picking(self):
        for line in self:
            if line.product_id and line.product_id.type in ('product', 'consu'):
                rounding = line.product_uom.rounding
                # Prevent decreasing below received quantity
                if float_compare(line.product_qty, line.qty_received, precision_rounding=rounding) < 0:
                    raise UserError(_('You cannot decrease the ordered quantity below the received quantity.\n'
                                      'Create a return first.'))

                if float_compare(line.product_qty, line.qty_invoiced, precision_rounding=rounding) < 0 and line.invoice_lines:
                    # If the quantity is now below the invoiced quantity, create an activity on the vendor bill
                    # inviting the user to create a refund.
                    line.invoice_lines[0].move_id.activity_schedule(
                        'mail.mail_activity_data_warning',
                        note=_('The quantities on your purchase order indicate less than billed. You should ask for a refund.'))

                # If the user increased quantity of existing line or created a new line
                # Give priority to the pickings related to the line
                line_pickings = line.move_ids.picking_id.filtered(lambda p: p.state not in ('done', 'cancel') and p.location_dest_id.usage in ('internal', 'transit', 'customer'))
                if line_pickings:
                    picking = line_pickings[0]
                else:
                    pickings = line.order_id.picking_ids.filtered(lambda x: x.state not in ('done', 'cancel') and x.location_dest_id.usage in ('internal', 'transit', 'customer'))
                    picking = pickings and pickings[0] or False
                if not picking:
                    if not line.product_qty > line.qty_received:
                        continue
                    res = line.order_id._prepare_picking()
                    picking = self.env['stock.picking'].create(res)

                moves = line._create_stock_moves(picking)
                moves._action_confirm()._action_assign()

    def _get_stock_move_price_unit(self):
        self.ensure_one()
        order = self.order_id
        price_unit = self._convert_to_tax_base_line_dict()['price_unit']
        price_unit_prec = self.env['decimal.precision'].precision_get('Product Price')
        if self.taxes_id:
            qty = self.product_qty or 1
            price_unit = self.taxes_id.with_context(round=False).compute_all(
                price_unit, currency=self.order_id.currency_id, quantity=qty, product=self.product_id, partner=self.order_id.partner_id
            )['total_void']
            price_unit = price_unit / qty
        if self.product_uom.id != self.product_id.uom_id.id:
            price_unit *= self.product_uom.factor / self.product_id.uom_id.factor
        if order.currency_id != order.company_id.currency_id:
            price_unit = order.currency_id._convert(
                price_unit, order.company_id.currency_id, self.company_id, self.date_order or fields.Date.today(), round=False)
        return float_round(price_unit, precision_digits=price_unit_prec)

    def _prepare_stock_moves(self, picking):
        """ Prepare the stock moves data for one order line. This function returns a list of
        dictionary ready to be used in stock.move's create()
        """
        self.ensure_one()
        res = []
        if self.product_id.type not in ['product', 'consu']:
            return res

        price_unit = self._get_stock_move_price_unit()
        qty = self._get_qty_procurement()

        move_dests = self.move_dest_ids or self.move_ids.move_dest_ids
        move_dests = move_dests.filtered(lambda m: m.state != 'cancel' and not m._is_purchase_return())

        if not move_dests:
            qty_to_attach = 0
            qty_to_push = self.product_qty - qty
        else:
            move_dests_initial_demand = self.product_id.uom_id._compute_quantity(
                sum(move_dests.filtered(lambda m: m.state != 'cancel' and not m.location_dest_id.usage == 'supplier').mapped('product_qty')),
                self.product_uom, rounding_method='HALF-UP')
            qty_to_attach = move_dests_initial_demand - qty
            qty_to_push = self.product_qty - move_dests_initial_demand

        if float_compare(qty_to_attach, 0.0, precision_rounding=self.product_uom.rounding) > 0:
            product_uom_qty, product_uom = self.product_uom._adjust_uom_quantities(qty_to_attach, self.product_id.uom_id)
            res.append(self._prepare_stock_move_vals(picking, price_unit, product_uom_qty, product_uom))
        if not float_is_zero(qty_to_push, precision_rounding=self.product_uom.rounding):
            product_uom_qty, product_uom = self.product_uom._adjust_uom_quantities(qty_to_push, self.product_id.uom_id)
            extra_move_vals = self._prepare_stock_move_vals(picking, price_unit, product_uom_qty, product_uom)
            extra_move_vals['move_dest_ids'] = False  # don't attach
            res.append(extra_move_vals)
        return res

    def _get_qty_procurement(self):
        self.ensure_one()
        qty = 0.0
        outgoing_moves, incoming_moves = self._get_outgoing_incoming_moves()
        for move in outgoing_moves:
            qty -= move.product_uom._compute_quantity(move.product_uom_qty, self.product_uom, rounding_method='HALF-UP')
        for move in incoming_moves:
            qty += move.product_uom._compute_quantity(move.product_uom_qty, self.product_uom, rounding_method='HALF-UP')
        return qty

    def _check_orderpoint_picking_type(self):
        warehouse_loc = self.order_id.picking_type_id.warehouse_id.view_location_id
        dest_loc = self.move_dest_ids.location_id or self.orderpoint_id.location_id
        if warehouse_loc and dest_loc and dest_loc.warehouse_id and not warehouse_loc.parent_path in dest_loc[0].parent_path:
            raise UserError(_('For the product %s, the warehouse of the operation type (%s) is inconsistent with the location (%s) of the reordering rule (%s). Change the operation type or cancel the request for quotation.',
                              self.product_id.display_name, self.order_id.picking_type_id.display_name, self.orderpoint_id.location_id.display_name, self.orderpoint_id.display_name))

    def _prepare_stock_move_vals(self, picking, price_unit, product_uom_qty, product_uom):
        self.ensure_one()
        self._check_orderpoint_picking_type()
        product = self.product_id.with_context(lang=self.order_id.dest_address_id.lang or self.env.user.lang)
        date_planned = self.date_planned or self.order_id.date_planned
        return {
            # truncate to 2000 to avoid triggering index limit error
            # TODO: remove index in master?
            'name': (self.product_id.display_name or '')[:2000],
            'product_id': self.product_id.id,
            'date': date_planned,
            'date_deadline': date_planned,
            'location_id': self.order_id.partner_id.property_stock_supplier.id,
            'location_dest_id': (self.orderpoint_id and not (self.move_ids | self.move_dest_ids)
                and (picking.location_dest_id.parent_path in self.orderpoint_id.location_id.parent_path))
                and self.orderpoint_id.location_id.id or self.order_id._get_destination_location(),
            'picking_id': picking.id,
            'partner_id': self.order_id.dest_address_id.id,
            'move_dest_ids': [(4, x) for x in self.move_dest_ids.ids],
            'state': 'draft',
            'purchase_line_id': self.id,
            'company_id': self.order_id.company_id.id,
            'price_unit': price_unit,
            'picking_type_id': self.order_id.picking_type_id.id,
            'group_id': self.order_id.group_id.id,
            'origin': self.order_id.name,
            'description_picking': product.description_pickingin or self.name,
            'propagate_cancel': self.propagate_cancel,
            'warehouse_id': self.order_id.picking_type_id.warehouse_id.id,
            'product_uom_qty': product_uom_qty,
            'product_uom': product_uom.id,
            'product_packaging_id': self.product_packaging_id.id,
            'sequence': self.sequence,
        }

    @api.model
    def _prepare_purchase_order_line_from_procurement(self, product_id, product_qty, product_uom, company_id, values, po):
        line_description = ''
        if values.get('product_description_variants'):
            line_description = values['product_description_variants']
        supplier = values.get('supplier')
        res = self._prepare_purchase_order_line(product_id, product_qty, product_uom, company_id, supplier, po)
        # We need to keep the vendor name set in _prepare_purchase_order_line. To avoid redundancy
        # in the line name, we add the line_description only if different from the product name.
        # This way, we shoud not lose any valuable information.
        if line_description and product_id.name != line_description:
            res['name'] += '\n' + line_description
        res['date_planned'] = values.get('date_planned')
        res['move_dest_ids'] = [(4, x.id) for x in values.get('move_dest_ids', [])]
        res['orderpoint_id'] = values.get('orderpoint_id', False) and values.get('orderpoint_id').id
        res['propagate_cancel'] = values.get('propagate_cancel')
        res['product_description_variants'] = values.get('product_description_variants')
        return res

    def _create_stock_moves(self, picking):
        values = []
        for line in self.filtered(lambda l: not l.display_type):
            for val in line._prepare_stock_moves(picking):
                values.append(val)
            line.move_dest_ids.created_purchase_line_id = False

        return self.env['stock.move'].create(values)

    def _find_candidate(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values):
        """ Return the record in self where the procument with values passed as
        args can be merged. If it returns an empty record then a new line will
        be created.
        """
        description_picking = ''
        if values.get('product_description_variants'):
            description_picking = values['product_description_variants']
        lines = self.filtered(
            lambda l: l.propagate_cancel == values['propagate_cancel']
            and (l.orderpoint_id == values['orderpoint_id'] if values['orderpoint_id'] and not values['move_dest_ids'] else True)
        )

        # In case 'product_description_variants' is in the values, we also filter on the PO line
        # name. This way, we can merge lines with the same description. To do so, we need the
        # product name in the context of the PO partner.
        if lines and values.get('product_description_variants'):
            partner = self.mapped('order_id.partner_id')[:1]
            product_lang = product_id.with_context(
                lang=partner.lang,
                partner_id=partner.id,
            )
            name = product_lang.display_name
            if product_lang.description_purchase:
                name += '\n' + product_lang.description_purchase
            lines = lines.filtered(lambda l: l.name == name + '\n' + description_picking)
            if lines:
                return lines[0]

        return lines and lines[0] or self.env['purchase.order.line']

    def _get_outgoing_incoming_moves(self):
        outgoing_moves = self.env['stock.move']
        incoming_moves = self.env['stock.move']

        for move in self.move_ids.filtered(lambda r: r.state != 'cancel' and not r.scrapped and self.product_id == r.product_id):
            if move._is_purchase_return() and move.to_refund:
                outgoing_moves |= move
            elif move.location_dest_id.usage != "supplier":
                if not move.origin_returned_move_id or (move.origin_returned_move_id and move.to_refund):
                    incoming_moves |= move

        return outgoing_moves, incoming_moves

    def _update_date_planned(self, updated_date):
        move_to_update = self.move_ids.filtered(lambda m: m.state not in ['done', 'cancel'])
        if not self.move_ids or move_to_update:  # Only change the date if there is no move done or none
            super()._update_date_planned(updated_date)
        if move_to_update:
            self._update_move_date_deadline(updated_date)

    @api.model
    def _update_qty_received_method(self):
        """Update qty_received_method for old PO before install this module."""
        self.search(['!', ('state', 'in', ['purchase', 'done'])])._compute_qty_received_method()

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    days_to_purchase = fields.Float(
        string='Days to Purchase',
        help="Days needed to confirm a PO, define when a PO should be validated")


    def _get_security_by_rule_action(self):
        res = super()._get_security_by_rule_action()
        res['buy'] = self.po_lead
        return res

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_stock_dropshipping = fields.Boolean("Dropshipping")
    days_to_purchase = fields.Float(
        related='company_id.days_to_purchase', readonly=False)
    is_installed_sale = fields.Boolean(string="Is the Sale Module Installed")

    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        res.update(
            is_installed_sale=self.env['ir.module.module'].search([('name', '=', 'sale'), ('state', '=', 'installed')]).id,
        )
        return res

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta, datetime, time
from collections import defaultdict

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    purchase_line_ids = fields.One2many('purchase.order.line', 'partner_id', string="Purchase Lines")
    on_time_rate = fields.Float(
        "On-Time Delivery Rate", compute='_compute_on_time_rate',
        help="Over the past x days; the number of products received on time divided by the number of ordered products."\
            "x is either the System Parameter purchase_stock.on_time_delivery_days or the default 365")

    @api.depends('purchase_line_ids')
    def _compute_on_time_rate(self):
        date_order_days_delta = int(self.env['ir.config_parameter'].sudo().get_param('purchase_stock.on_time_delivery_days', default='365'))
        order_lines = self.env['purchase.order.line'].search([
            ('partner_id', 'in', self.ids),
            ('date_order', '>', fields.Date.today() - timedelta(date_order_days_delta)),
            ('qty_received', '!=', 0),
            ('order_id.state', 'in', ['done', 'purchase']),
            ('product_id', 'in', self.env['product.product'].sudo()._search([('type', '!=', 'service')]))
        ])
        lines_qty_done = defaultdict(lambda: 0)
        moves = self.env['stock.move'].search([
            ('purchase_line_id', 'in', order_lines.ids),
            ('state', '=', 'done')])
        # Fetch fields from db and put them in cache.
        order_lines.read(['date_planned', 'partner_id', 'product_uom_qty'], load='')
        moves.read(['purchase_line_id', 'date'], load='')
        moves = moves.filtered(lambda m: m.date.date() <= m.purchase_line_id.date_planned.date())
        for move, qty_done in zip(moves, moves.mapped('quantity_done')):
            lines_qty_done[move.purchase_line_id.id] += qty_done
        partner_dict = {}
        for line in order_lines:
            on_time, ordered = partner_dict.get(line.partner_id, (0, 0))
            ordered += line.product_uom_qty
            on_time += lines_qty_done[line.id]
            partner_dict[line.partner_id] = (on_time, ordered)
        seen_partner = self.env['res.partner']
        for partner, numbers in partner_dict.items():
            seen_partner |= partner
            on_time, ordered = numbers
            partner.on_time_rate = on_time / ordered * 100 if ordered else -1   # use negative number to indicate no data
        (self - seen_partner).on_time_rate = -1

```

## File: models\stock.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.osv.expression import AND


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    purchase_id = fields.Many2one(
        'purchase.order', related='move_ids.purchase_line_id.order_id',
        string="Purchase Orders", readonly=True)


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    buy_to_resupply = fields.Boolean('Buy to Resupply', default=True,
                                     help="When products are bought, they can be delivered to this warehouse")
    buy_pull_id = fields.Many2one('stock.rule', 'Buy rule')

    def _get_global_route_rules_values(self):
        rules = super(StockWarehouse, self)._get_global_route_rules_values()
        location_id = self.in_type_id.default_location_dest_id
        rules.update({
            'buy_pull_id': {
                'depends': ['reception_steps', 'buy_to_resupply'],
                'create_values': {
                    'action': 'buy',
                    'picking_type_id': self.in_type_id.id,
                    'group_propagation_option': 'none',
                    'company_id': self.company_id.id,
                    'route_id': self._find_or_create_global_route('purchase_stock.route_warehouse0_buy', _('Buy')).id,
                    'propagate_cancel': self.reception_steps != 'one_step',
                },
                'update_values': {
                    'active': self.buy_to_resupply,
                    'name': self._format_rulename(location_id, False, 'Buy'),
                    'location_dest_id': location_id.id,
                    'propagate_cancel': self.reception_steps != 'one_step',
                }
            }
        })
        return rules

    def _get_all_routes(self):
        routes = super(StockWarehouse, self)._get_all_routes()
        routes |= self.filtered(lambda self: self.buy_to_resupply and self.buy_pull_id and self.buy_pull_id.route_id).mapped('buy_pull_id').mapped('route_id')
        return routes

    def get_rules_dict(self):
        result = super(StockWarehouse, self).get_rules_dict()
        for warehouse in self:
            result[warehouse.id].update(warehouse._get_receive_rules_dict())
        return result

    def _get_routes_values(self):
        routes = super(StockWarehouse, self)._get_routes_values()
        routes.update(self._get_receive_routes_values('buy_to_resupply'))
        return routes

    def _update_name_and_code(self, name=False, code=False):
        res = super(StockWarehouse, self)._update_name_and_code(name, code)
        warehouse = self[0]
        #change the buy stock rule name
        if warehouse.buy_pull_id and name:
            warehouse.buy_pull_id.write({'name': warehouse.buy_pull_id.name.replace(warehouse.name, name, 1)})
        return res


class ReturnPicking(models.TransientModel):
    _inherit = "stock.return.picking"

    def _prepare_move_default_values(self, return_line, new_picking):
        vals = super(ReturnPicking, self)._prepare_move_default_values(return_line, new_picking)
        vals['purchase_line_id'] = return_line.move_id.purchase_line_id.id
        return vals


class Orderpoint(models.Model):
    _inherit = "stock.warehouse.orderpoint"

    show_supplier = fields.Boolean('Show supplier column', compute='_compute_show_suppplier')
    supplier_id = fields.Many2one(
        'product.supplierinfo', string='Product Supplier', check_company=True,
        domain="['|', ('product_id', '=', product_id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product_tmpl_id)]")
    vendor_id = fields.Many2one(related='supplier_id.partner_id', string="Vendor", store=True)
    purchase_visibility_days = fields.Float(default=0.0, help="Visibility Days applied on the purchase routes.")

    @api.depends('product_id.purchase_order_line_ids.product_qty', 'product_id.purchase_order_line_ids.state')
    def _compute_qty(self):
        """ Extend to add more depends values """
        return super()._compute_qty()

    @api.depends('product_id.purchase_order_line_ids.product_qty', 'product_id.purchase_order_line_ids.state')
    def _compute_qty_to_order(self):
        """ Extend to add more depends values """
        return super()._compute_qty_to_order()

    @api.depends('supplier_id')
    def _compute_lead_days(self):
        return super()._compute_lead_days()

    def _compute_visibility_days(self):
        res = super()._compute_visibility_days()
        for orderpoint in self:
            if 'buy' in orderpoint.rule_ids.mapped('action'):
                orderpoint.visibility_days = orderpoint.purchase_visibility_days
        return res

    def _set_visibility_days(self):
        res = super()._set_visibility_days()
        for orderpoint in self:
            if 'buy' in orderpoint.rule_ids.mapped('action'):
                orderpoint.purchase_visibility_days = orderpoint.visibility_days
        return res

    def _compute_days_to_order(self):
        res = super()._compute_days_to_order()
        for orderpoint in self:
            if 'buy' in orderpoint.rule_ids.mapped('action'):
                orderpoint.days_to_order = orderpoint.company_id.days_to_purchase
        return res

    @api.depends('route_id')
    def _compute_show_suppplier(self):
        buy_route = []
        for res in self.env['stock.rule'].search_read([('action', '=', 'buy')], ['route_id']):
            buy_route.append(res['route_id'][0])
        for orderpoint in self:
            orderpoint.show_supplier = orderpoint.route_id.id in buy_route

    def action_view_purchase(self):
        """ This function returns an action that display existing
        purchase orders of given orderpoint.
        """
        result = self.env['ir.actions.act_window']._for_xml_id('purchase.purchase_rfq')

        # Remvove the context since the action basically display RFQ and not PO.
        result['context'] = {}
        order_line_ids = self.env['purchase.order.line'].search([('orderpoint_id', '=', self.id)])
        purchase_ids = order_line_ids.mapped('order_id')

        result['domain'] = "[('id','in',%s)]" % (purchase_ids.ids)

        return result

    def _get_lead_days_values(self):
        values = super()._get_lead_days_values()
        if self.supplier_id:
            values['supplierinfo'] = self.supplier_id
        return values

    def _get_replenishment_order_notification(self):
        self.ensure_one()
        domain = [('orderpoint_id', 'in', self.ids)]
        if self.env.context.get('written_after'):
            domain = AND([domain, [('write_date', '>', self.env.context.get('written_after'))]])
        order = self.env['purchase.order.line'].search(domain, limit=1).order_id
        if order:
            action = self.env.ref('purchase.action_rfq_form')
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'title': _('The following replenishment order has been generated'),
                    'message': '%s',
                    'links': [{
                        'label': order.display_name,
                        'url': f'#action={action.id}&id={order.id}&model=purchase.order',
                    }],
                    'sticky': False,
                    'next': {'type': 'ir.actions.act_window_close'},
                }
            }
        return super()._get_replenishment_order_notification()

    def _prepare_procurement_values(self, date=False, group=False):
        values = super()._prepare_procurement_values(date=date, group=group)
        values['supplierinfo_id'] = self.supplier_id
        return values

    def _quantity_in_progress(self):
        res = super()._quantity_in_progress()
        qty_by_product_location, dummy = self.product_id._get_quantity_in_progress(self.location_id.ids)
        for orderpoint in self:
            product_qty = qty_by_product_location.get((orderpoint.product_id.id, orderpoint.location_id.id), 0.0)
            product_uom_qty = orderpoint.product_id.uom_id._compute_quantity(product_qty, orderpoint.product_uom, round=False)
            res[orderpoint.id] += product_uom_qty
        return res

    def _set_default_route_id(self):
        route_ids = self.env['stock.rule'].search([
            ('action', '=', 'buy')
        ]).route_id
        for orderpoint in self:
            route_id = orderpoint.rule_ids.route_id & route_ids
            if not orderpoint.product_id.seller_ids:
                continue
            if not route_id:
                continue
            orderpoint.route_id = route_id[0].id
        return super()._set_default_route_id()


class StockLot(models.Model):
    _inherit = 'stock.lot'

    purchase_order_ids = fields.Many2many('purchase.order', string="Purchase Orders", compute='_compute_purchase_order_ids', readonly=True, store=False)
    purchase_order_count = fields.Integer('Purchase order count', compute='_compute_purchase_order_ids')

    @api.depends('name')
    def _compute_purchase_order_ids(self):
        for lot in self:
            stock_moves = self.env['stock.move.line'].search([
                ('lot_id', '=', lot.id),
                ('state', '=', 'done')
            ]).mapped('move_id')
            stock_moves = stock_moves.search([('id', 'in', stock_moves.ids)]).filtered(
                lambda move: move.picking_id.location_id.usage == 'supplier' and move.state == 'done')
            lot.purchase_order_ids = stock_moves.mapped('purchase_line_id.order_id')
            lot.purchase_order_count = len(lot.purchase_order_ids)

    def action_view_po(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("purchase.purchase_form_action")
        action['domain'] = [('id', 'in', self.mapped('purchase_order_ids.id'))]
        action['context'] = dict(self._context, create=False)
        return action


class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    @api.model
    def run(self, procurements, raise_user_error=True):
        wh_by_comp = dict()
        for procurement in procurements:
            routes = procurement.values.get('route_ids')
            if routes and any(r.action == 'buy' for r in routes.rule_ids):
                company = procurement.company_id
                if company not in wh_by_comp:
                    wh_by_comp[company] = self.env['stock.warehouse'].search([('company_id', '=', company.id)])
                wh = wh_by_comp[company]
                procurement.values['route_ids'] |= wh.reception_route_id
        return super().run(procurements, raise_user_error=raise_user_error)

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools.float_utils import float_round, float_is_zero, float_compare
from odoo.exceptions import UserError


class StockMove(models.Model):
    _inherit = 'stock.move'

    purchase_line_id = fields.Many2one(
        'purchase.order.line', 'Purchase Order Line',
        ondelete='set null', index='btree_not_null', readonly=True)
    created_purchase_line_id = fields.Many2one(
        'purchase.order.line', 'Created Purchase Order Line',
        ondelete='set null', index='btree_not_null', readonly=True, copy=False)

    @api.model
    def _prepare_merge_moves_distinct_fields(self):
        distinct_fields = super(StockMove, self)._prepare_merge_moves_distinct_fields()
        distinct_fields += ['purchase_line_id', 'created_purchase_line_id']
        return distinct_fields

    @api.model
    def _prepare_merge_negative_moves_excluded_distinct_fields(self):
        excluded_fields = super()._prepare_merge_negative_moves_excluded_distinct_fields() + ['created_purchase_line_id']
        if self.env['ir.config_parameter'].sudo().get_param('purchase_stock.merge_different_procurement'):
            excluded_fields += ['procure_method']
        return excluded_fields

    def _should_ignore_pol_price(self):
        self.ensure_one()
        return self.origin_returned_move_id or not self.purchase_line_id or not self.product_id.id

    def _get_price_unit(self):
        """ Returns the unit price for the move"""
        self.ensure_one()
        if self._should_ignore_pol_price():
            return super(StockMove, self)._get_price_unit()
        price_unit_prec = self.env['decimal.precision'].precision_get('Product Price')
        line = self.purchase_line_id
        order = line.order_id
        received_qty = line.qty_received
        if self.state == 'done':
            received_qty -= self.product_uom._compute_quantity(self.quantity_done, line.product_uom, rounding_method='HALF-UP')
        if line.product_id.purchase_method == 'purchase' and float_compare(line.qty_invoiced, received_qty, precision_rounding=line.product_uom.rounding) > 0:
            move_layer = line.move_ids.sudo().stock_valuation_layer_ids
            invoiced_layer = line.sudo().invoice_lines.stock_valuation_layer_ids
            # value on valuation layer is in company's currency, while value on invoice line is in order's currency
            receipt_value = 0
            for layer in move_layer:
                if not layer._should_impact_price_unit_receipt_value():
                    continue
                receipt_value += layer.currency_id._convert(
                    layer.value, order.currency_id, order.company_id, layer.create_date, round=False)
            if invoiced_layer:
                receipt_value += sum(invoiced_layer.mapped(lambda l: l.currency_id._convert(
                    l.value, order.currency_id, order.company_id, l.create_date, round=False)))
            total_invoiced_value = 0
            invoiced_qty = 0
            for invoice_line in line.sudo().invoice_lines:
                if invoice_line.move_id.state != 'posted':
                    continue
                # Discount applied on bill prior to reception
                if invoice_line.discount and not move_layer:
                    price_unit = invoice_line.price_subtotal / invoice_line.quantity
                else:
                    price_unit = invoice_line.price_unit
                if invoice_line.tax_ids:
                    invoice_line_value = invoice_line.tax_ids.with_context(round=False).compute_all(
                        price_unit, currency=invoice_line.currency_id, quantity=invoice_line.quantity)['total_void']
                else:
                    invoice_line_value = price_unit * invoice_line.quantity
                total_invoiced_value += invoice_line.currency_id._convert(
                        invoice_line_value, order.currency_id, order.company_id, invoice_line.move_id.invoice_date, round=False)
                invoiced_qty += invoice_line.product_uom_id._compute_quantity(invoice_line.quantity, line.product_id.uom_id)
            # TODO currency check
            remaining_value = total_invoiced_value - receipt_value
            # TODO qty_received in product uom
            remaining_qty = invoiced_qty - line.product_uom._compute_quantity(received_qty, line.product_id.uom_id)
            if order.currency_id != order.company_id.currency_id and remaining_value and remaining_qty:
                # will be rounded during currency conversion
                price_unit = remaining_value / remaining_qty
            elif remaining_value and remaining_qty:
                price_unit = float_round(remaining_value / remaining_qty, precision_digits=price_unit_prec)
            else:
                price_unit = line._get_gross_price_unit()
        else:
            price_unit = line._get_gross_price_unit()
        if order.currency_id != order.company_id.currency_id:
            # The date must be today, and not the date of the move since the move move is still
            # in assigned state. However, the move date is the scheduled date until move is
            # done, then date of actual move processing. See:
            # https://github.com/odoo/odoo/blob/2f789b6863407e63f90b3a2d4cc3be09815f7002/addons/stock/models/stock_move.py#L36
            convert_date = fields.Date.context_today(self)
            # use currency rate at bill date when invoice before receipt
            if float_compare(line.qty_invoiced, received_qty, precision_rounding=line.product_uom.rounding) > 0:
                convert_date = max(line.sudo().invoice_lines.move_id.filtered(lambda m: m.state == 'posted').mapped('invoice_date'), default=convert_date)
            price_unit = order.currency_id._convert(
                price_unit, order.company_id.currency_id, order.company_id, convert_date, round=False)
        return price_unit

    def _generate_valuation_lines_data(self, partner_id, qty, debit_value, credit_value, debit_account_id, credit_account_id, svl_id, description):
        """ Overridden from stock_account to support amount_currency on valuation lines generated from po
        """
        self.ensure_one()

        rslt = super(StockMove, self)._generate_valuation_lines_data(partner_id, qty, debit_value, credit_value, debit_account_id, credit_account_id, svl_id, description)
        purchase_currency = self.purchase_line_id.currency_id
        company_currency = self.company_id.currency_id
        if not self.purchase_line_id or purchase_currency == company_currency:
            return rslt
        svl = self.env['stock.valuation.layer'].browse(svl_id)
        if not svl.account_move_line_id:
            rslt['credit_line_vals']['amount_currency'] = company_currency._convert(
                rslt['credit_line_vals']['balance'],
                purchase_currency,
                self.company_id,
                self.date
            )
            rslt['debit_line_vals']['amount_currency'] = company_currency._convert(
                rslt['debit_line_vals']['balance'],
                purchase_currency,
                self.company_id,
                self.date
            )
            rslt['debit_line_vals']['currency_id'] = purchase_currency.id
            rslt['credit_line_vals']['currency_id'] = purchase_currency.id
        else:
            rslt['credit_line_vals']['amount_currency'] = 0
            rslt['debit_line_vals']['amount_currency'] = 0
            rslt['debit_line_vals']['currency_id'] = purchase_currency.id
            rslt['credit_line_vals']['currency_id'] = purchase_currency.id
            if not svl.price_diff_value:
                return rslt
            # The idea is to force using the company currency during the reconciliation process
            rslt['debit_line_vals_curr'] = {
                'name': _("Currency exchange rate difference"),
                'product_id': self.product_id.id,
                'quantity': 0,
                'product_uom_id': self.product_id.uom_id.id,
                'partner_id': partner_id,
                'balance': 0,
                'account_id': debit_account_id,
                'currency_id': purchase_currency.id,
                'amount_currency': -svl.price_diff_value,
            }
            rslt['credit_line_vals_curr'] = {
                'name': _("Currency exchange rate difference"),
                'product_id': self.product_id.id,
                'quantity': 0,
                'product_uom_id': self.product_id.uom_id.id,
                'partner_id': partner_id,
                'balance': 0,
                'account_id': credit_account_id,
                'currency_id': purchase_currency.id,
                'amount_currency': svl.price_diff_value,
            }
        return rslt

    def _account_entry_move(self, qty, description, svl_id, cost):
        """
        In case of a PO return, if the value of the returned product is
        different from the purchased one, we need to empty the stock_in account
        with the difference
        """
        am_vals_list = super()._account_entry_move(qty, description, svl_id, cost)
        returned_move = self.origin_returned_move_id
        move = (self | returned_move).with_prefetch(self._prefetch_ids)
        pdiff_exists = bool(move.stock_valuation_layer_ids.stock_valuation_layer_ids.account_move_line_id)

        if not am_vals_list or not self.purchase_line_id or pdiff_exists or float_is_zero(qty, precision_rounding=self.product_id.uom_id.rounding):
            return am_vals_list

        layer = self.env['stock.valuation.layer'].browse(svl_id)

        if returned_move and self._is_out() and self._is_returned(valued_type='out'):
            returned_layer = returned_move.stock_valuation_layer_ids.filtered(lambda svl: not svl.stock_valuation_layer_id)[:1]
            returned_unit_cost = returned_layer.value / returned_layer.quantity
            layer_unit_cost = layer.value / layer.quantity
            unit_diff = layer_unit_cost - returned_unit_cost
        elif returned_move and returned_move._is_out() and returned_move._is_returned(valued_type='out'):
            returned_layer = returned_move.stock_valuation_layer_ids.filtered(lambda svl: not svl.stock_valuation_layer_id)[:1]
            unit_diff = returned_layer.unit_cost - self.purchase_line_id._get_gross_price_unit()
        else:
            return am_vals_list

        diff = unit_diff * qty
        company = self.purchase_line_id.company_id
        if company.currency_id.is_zero(diff):
            return am_vals_list

        sm = self.with_company(company).with_context(is_returned=True)
        accounts = sm.product_id.product_tmpl_id.get_product_accounts()
        acc_exp_id = accounts['expense'].id
        acc_stock_in_id = accounts['stock_input'].id
        journal_id = accounts['stock_journal'].id
        vals = sm._prepare_account_move_vals(acc_exp_id, acc_stock_in_id, journal_id, qty, description, False, diff)
        am_vals_list.append(vals)

        return am_vals_list

    def _prepare_extra_move_vals(self, qty):
        vals = super(StockMove, self)._prepare_extra_move_vals(qty)
        vals['purchase_line_id'] = self.purchase_line_id.id
        return vals

    def _prepare_move_split_vals(self, uom_qty):
        vals = super(StockMove, self)._prepare_move_split_vals(uom_qty)
        vals['purchase_line_id'] = self.purchase_line_id.id
        return vals

    def _clean_merged(self):
        super(StockMove, self)._clean_merged()
        self.write({'created_purchase_line_id': False})

    def _get_upstream_documents_and_responsibles(self, visited):
        if self.created_purchase_line_id and self.created_purchase_line_id.state not in ('done', 'cancel') \
                and (self.created_purchase_line_id.state != 'draft' or self._context.get('include_draft_documents')):
            return [(self.created_purchase_line_id.order_id, self.created_purchase_line_id.order_id.user_id, visited)]
        elif self.purchase_line_id and self.purchase_line_id.state not in ('done', 'cancel'):
            return[(self.purchase_line_id.order_id, self.purchase_line_id.order_id.user_id, visited)]
        else:
            return super(StockMove, self)._get_upstream_documents_and_responsibles(visited)

    def _get_related_invoices(self):
        """ Overridden to return the vendor bills related to this stock move.
        """
        rslt = super(StockMove, self)._get_related_invoices()
        purchase_ids = self.env['purchase.order'].search([('picking_ids', 'in', self.picking_id.ids)])
        rslt += purchase_ids.invoice_ids.filtered(lambda x: x.state == 'posted')
        return rslt

    def _get_source_document(self):
        res = super()._get_source_document()
        return self.purchase_line_id.order_id or res

    def _get_valuation_price_and_qty(self, related_aml, to_curr):
        valuation_price_unit_total = 0
        valuation_total_qty = 0
        for val_stock_move in self:
            # In case val_stock_move is a return move, its valuation entries have been made with the
            # currency rate corresponding to the original stock move
            valuation_date = val_stock_move.origin_returned_move_id.date or val_stock_move.date
            svl = val_stock_move.with_context(active_test=False).mapped('stock_valuation_layer_ids').filtered(
                lambda l: l.quantity)
            layers_qty = sum(svl.mapped('quantity'))
            layers_values = sum(svl.mapped('value'))
            valuation_price_unit_total += related_aml.company_currency_id._convert(
                layers_values, to_curr, related_aml.company_id, valuation_date, round=False,
            )
            valuation_total_qty += layers_qty
        if float_is_zero(valuation_total_qty, precision_rounding=related_aml.product_uom_id.rounding or related_aml.product_id.uom_id.rounding):
            raise UserError(
                _('Odoo is not able to generate the anglo saxon entries. The total valuation of %s is zero.') % related_aml.product_id.display_name)
        return valuation_price_unit_total, valuation_total_qty

    def _is_purchase_return(self):
        self.ensure_one()
        return self.location_dest_id.usage == "supplier" or self.location_dest_id == self.env.ref('stock.stock_location_inter_wh', raise_if_not_found=False)

    def _get_all_related_aml(self):
        # The back and for between account_move and account_move_line is necessary to catch the
        # additional lines from a cogs correction
        return super()._get_all_related_aml() | self.purchase_line_id.invoice_lines.move_id.line_ids.filtered(
            lambda aml: aml.product_id == self.purchase_line_id.product_id)

    def _get_all_related_sm(self, product):
        return super()._get_all_related_sm(product) | self.filtered(lambda m: m.purchase_line_id.product_id == product)

```

## File: models\stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import datetime
from dateutil.relativedelta import relativedelta
from odoo.tools import float_compare

from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.addons.stock.models.stock_rule import ProcurementException
from odoo.tools import groupby


class StockRule(models.Model):
    _inherit = 'stock.rule'

    action = fields.Selection(selection_add=[
        ('buy', 'Buy')
    ], ondelete={'buy': 'cascade'})

    def _get_message_dict(self):
        message_dict = super(StockRule, self)._get_message_dict()
        dummy, destination, dummy = self._get_message_values()
        message_dict.update({
            'buy': _('When products are needed in <b>%s</b>, <br/> '
                     'a request for quotation is created to fulfill the need.<br/>'
                     'Note: This rule will be used in combination with the rules<br/>'
                     'of the reception route(s)') % (destination)
        })
        return message_dict

    @api.depends('action')
    def _compute_picking_type_code_domain(self):
        remaining = self.browse()
        for rule in self:
            if rule.action == 'buy':
                rule.picking_type_code_domain = 'incoming'
            else:
                remaining |= rule
        super(StockRule, remaining)._compute_picking_type_code_domain()

    @api.onchange('action')
    def _onchange_action(self):
        if self.action == 'buy':
            self.location_src_id = False

    @api.model
    def _run_buy(self, procurements):
        procurements_by_po_domain = defaultdict(list)
        errors = []
        for procurement, rule in procurements:

            # Get the schedule date in order to find a valid seller
            procurement_date_planned = fields.Datetime.from_string(procurement.values['date_planned'])

            supplier = False
            if procurement.values.get('supplierinfo_id'):
                supplier = procurement.values['supplierinfo_id']
            elif procurement.values.get('orderpoint_id') and procurement.values['orderpoint_id'].supplier_id:
                supplier = procurement.values['orderpoint_id'].supplier_id
            else:
                supplier = procurement.product_id.with_company(procurement.company_id.id)._select_seller(
                    partner_id=procurement.values.get("supplierinfo_name"),
                    quantity=procurement.product_qty,
                    date=max(procurement_date_planned.date(), fields.Date.today()),
                    uom_id=procurement.product_uom)

            # Fall back on a supplier for which no price may be defined. Not ideal, but better than
            # blocking the user.
            supplier = supplier or procurement.product_id._prepare_sellers(False).filtered(
                lambda s: not s.company_id or s.company_id == procurement.company_id
            )[:1]

            if not supplier:
                msg = _('There is no matching vendor price to generate the purchase order for product %s (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.') % (procurement.product_id.display_name)
                errors.append((procurement, msg))

            partner = supplier.partner_id
            # we put `supplier_info` in values for extensibility purposes
            procurement.values['supplier'] = supplier
            procurement.values['propagate_cancel'] = rule.propagate_cancel

            domain = rule._make_po_get_domain(procurement.company_id, procurement.values, partner)
            procurements_by_po_domain[domain].append((procurement, rule))

        if errors:
            raise ProcurementException(errors)

        for domain, procurements_rules in procurements_by_po_domain.items():
            # Get the procurements for the current domain.
            # Get the rules for the current domain. Their only use is to create
            # the PO if it does not exist.
            procurements, rules = zip(*procurements_rules)

            # Get the set of procurement origin for the current domain.
            origins = set([p.origin for p in procurements])
            # Check if a PO exists for the current domain.
            po = self.env['purchase.order'].sudo().search([dom for dom in domain], limit=1)
            company_id = procurements[0].company_id
            if not po:
                positive_values = [p.values for p in procurements if float_compare(p.product_qty, 0.0, precision_rounding=p.product_uom.rounding) >= 0]
                if positive_values:
                    # We need a rule to generate the PO. However the rule generated
                    # the same domain for PO and the _prepare_purchase_order method
                    # should only uses the common rules's fields.
                    vals = rules[0]._prepare_purchase_order(company_id, origins, positive_values)
                    # The company_id is the same for all procurements since
                    # _make_po_get_domain add the company in the domain.
                    # We use SUPERUSER_ID since we don't want the current user to be follower of the PO.
                    # Indeed, the current user may be a user without access to Purchase, or even be a portal user.
                    po = self.env['purchase.order'].with_company(company_id).with_user(SUPERUSER_ID).create(vals)
            else:
                # If a purchase order is found, adapt its `origin` field.
                if po.origin:
                    missing_origins = origins - set(po.origin.split(', '))
                    if missing_origins:
                        po.write({'origin': po.origin + ', ' + ', '.join(missing_origins)})
                else:
                    po.write({'origin': ', '.join(origins)})

            procurements_to_merge = self._get_procurements_to_merge(procurements)
            procurements = self._merge_procurements(procurements_to_merge)

            po_lines_by_product = {}
            grouped_po_lines = groupby(po.order_line.filtered(lambda l: not l.display_type and l.product_uom == l.product_id.uom_po_id), key=lambda l: l.product_id.id)
            for product, po_lines in grouped_po_lines:
                po_lines_by_product[product] = self.env['purchase.order.line'].concat(*po_lines)
            po_line_values = []
            for procurement in procurements:
                po_lines = po_lines_by_product.get(procurement.product_id.id, self.env['purchase.order.line'])
                po_line = po_lines._find_candidate(*procurement)

                if po_line:
                    # If the procurement can be merge in an existing line. Directly
                    # write the new values on it.
                    vals = self._update_purchase_order_line(procurement.product_id,
                        procurement.product_qty, procurement.product_uom, company_id,
                        procurement.values, po_line)
                    po_line.sudo().write(vals)
                else:
                    if float_compare(procurement.product_qty, 0, precision_rounding=procurement.product_uom.rounding) <= 0:
                        # If procurement contains negative quantity, don't create a new line that would contain negative qty
                        continue
                    # If it does not exist a PO line for current procurement.
                    # Generate the create values for it and add it to a list in
                    # order to create it in batch.
                    partner = procurement.values['supplier'].partner_id
                    po_line_values.append(self.env['purchase.order.line']._prepare_purchase_order_line_from_procurement(
                        procurement.product_id, procurement.product_qty,
                        procurement.product_uom, procurement.company_id,
                        procurement.values, po))
                    # Check if we need to advance the order date for the new line
                    order_date_planned = procurement.values['date_planned'] - relativedelta(
                        days=procurement.values['supplier'].delay)
                    if fields.Date.to_date(order_date_planned) < fields.Date.to_date(po.date_order):
                        po.date_order = order_date_planned
            self.env['purchase.order.line'].sudo().create(po_line_values)

    def _get_lead_days(self, product, **values):
        """Add the company security lead time and the supplier delay to the cumulative delay
        and cumulative description. The company lead time is always displayed for onboarding
        purpose in order to indicate that those options are available.
        """
        delay, delay_description = super()._get_lead_days(product, **values)
        bypass_delay_description = self.env.context.get('bypass_delay_description')
        buy_rule = self.filtered(lambda r: r.action == 'buy')
        seller = 'supplierinfo' in values and values['supplierinfo'] or product.with_company(buy_rule.company_id)._select_seller(quantity=None)
        if not buy_rule or not seller:
            return delay, delay_description
        buy_rule.ensure_one()
        supplier_delay = seller[0].delay
        if supplier_delay and not bypass_delay_description:
            delay_description.append((_('Vendor Lead Time'), _('+ %d day(s)', supplier_delay)))
        security_delay = buy_rule.picking_type_id.company_id.po_lead
        if not bypass_delay_description:
            delay_description.append((_('Purchase Security Lead Time'), _('+ %d day(s)', security_delay)))
        days_to_order = values.get('days_to_order', buy_rule.company_id.days_to_purchase)
        if not bypass_delay_description:
            delay_description.append((_('Days to Purchase'), _('+ %d day(s)', days_to_order)))
        return delay + supplier_delay + security_delay + days_to_order, delay_description

    @api.model
    def _get_procurements_to_merge_groupby(self, procurement):
        # Do not group procument from different orderpoint. 1. _quantity_in_progress
        # directly depends from the orderpoint_id on the line. 2. The stock move
        # generated from the order line has the orderpoint's location as
        # destination location. In case of move_dest_ids those two points are not
        # necessary anymore since those values are taken from destination moves.
        return procurement.product_id, procurement.product_uom, procurement.values['propagate_cancel'],\
            procurement.values.get('product_description_variants'),\
            (procurement.values.get('orderpoint_id') and not procurement.values.get('move_dest_ids')) and procurement.values['orderpoint_id']

    @api.model
    def _get_procurements_to_merge(self, procurements):
        """ Get a list of procurements values and create groups of procurements
        that would use the same purchase order line.
        params procurements_list list: procurements requests (not ordered nor
        sorted).
        return list: procurements requests grouped by their product_id.
        """
        return [pro_g for __, pro_g in groupby(procurements, key=self._get_procurements_to_merge_groupby)]

    @api.model
    def _merge_procurements(self, procurements_to_merge):
        """ Merge the quantity for procurements requests that could use the same
        order line.
        params similar_procurements list: list of procurements that have been
        marked as 'alike' from _get_procurements_to_merge method.
        return a list of procurements values where values of similar_procurements
        list have been merged.
        """
        merged_procurements = []
        for procurements in procurements_to_merge:
            quantity = 0
            move_dest_ids = self.env['stock.move']
            orderpoint_id = self.env['stock.warehouse.orderpoint']
            for procurement in procurements:
                if procurement.values.get('move_dest_ids'):
                    move_dest_ids |= procurement.values['move_dest_ids']
                if not orderpoint_id and procurement.values.get('orderpoint_id'):
                    orderpoint_id = procurement.values['orderpoint_id']
                quantity += procurement.product_qty
            # The merged procurement can be build from an arbitrary procurement
            # since they were mark as similar before. Only the quantity and
            # some keys in values are updated.
            values = dict(procurement.values)
            values.update({
                'move_dest_ids': move_dest_ids,
                'orderpoint_id': orderpoint_id,
            })
            merged_procurement = self.env['procurement.group'].Procurement(
                procurement.product_id, quantity, procurement.product_uom,
                procurement.location_id, procurement.name, procurement.origin,
                procurement.company_id, values
            )
            merged_procurements.append(merged_procurement)
        return merged_procurements

    def _update_purchase_order_line(self, product_id, product_qty, product_uom, company_id, values, line):
        partner = values['supplier'].partner_id
        procurement_uom_po_qty = product_uom._compute_quantity(product_qty, product_id.uom_po_id, rounding_method='HALF-UP')
        seller = product_id.with_company(company_id)._select_seller(
            partner_id=partner,
            quantity=line.product_qty + procurement_uom_po_qty,
            date=line.order_id.date_order and line.order_id.date_order.date(),
            uom_id=product_id.uom_po_id)

        price_unit = self.env['account.tax']._fix_tax_included_price_company(seller.price, line.product_id.supplier_taxes_id, line.sudo().taxes_id, company_id) if seller else 0.0
        if price_unit and seller and line.order_id.currency_id and seller.currency_id != line.order_id.currency_id:
            price_unit = seller.currency_id._convert(
                price_unit, line.order_id.currency_id, line.order_id.company_id, fields.Date.today())

        res = {
            'product_qty': line.product_qty + procurement_uom_po_qty,
            'price_unit': price_unit,
            'move_dest_ids': [(4, x.id) for x in values.get('move_dest_ids', [])]
        }
        orderpoint_id = values.get('orderpoint_id')
        if orderpoint_id:
            res['orderpoint_id'] = orderpoint_id.id
        return res

    def _prepare_purchase_order(self, company_id, origins, values):
        """ Create a purchase order for procuremets that share the same domain
        returned by _make_po_get_domain.
        params values: values of procurements
        params origins: procuremets origins to write on the PO
        """
        purchase_date = min([fields.Datetime.from_string(value['date_planned']) - relativedelta(days=int(value['supplier'].delay)) for value in values])

        # Since the procurements are grouped if they share the same domain for
        # PO but the PO does not exist. In this case it will create the PO from
        # the common procurements values. The common values are taken from an
        # arbitrary procurement. In this case the first.
        values = values[0]
        partner = values['supplier'].partner_id

        fpos = self.env['account.fiscal.position'].with_company(company_id)._get_fiscal_position(partner)

        gpo = self.group_propagation_option
        group = (gpo == 'fixed' and self.group_id.id) or \
                (gpo == 'propagate' and values.get('group_id') and values['group_id'].id) or False

        return {
            'partner_id': partner.id,
            'user_id': False,
            'picking_type_id': self.picking_type_id.id,
            'company_id': company_id.id,
            'currency_id': partner.with_company(company_id).property_purchase_currency_id.id or company_id.currency_id.id,
            'dest_address_id': values.get('partner_id', False),
            'origin': ', '.join(origins),
            'payment_term_id': partner.with_company(company_id).property_supplier_payment_term_id.id,
            'date_order': purchase_date,
            'fiscal_position_id': fpos.id,
            'group_id': group
        }

    def _make_po_get_domain(self, company_id, values, partner):
        gpo = self.group_propagation_option
        group = (gpo == 'fixed' and self.group_id) or \
                (gpo == 'propagate' and 'group_id' in values and values['group_id']) or False

        domain = (
            ('partner_id', '=', partner.id),
            ('state', '=', 'draft'),
            ('picking_type_id', '=', self.picking_type_id.id),
            ('company_id', '=', company_id.id),
            ('user_id', '=', False),
        )
        delta_days = self.env['ir.config_parameter'].sudo().get_param('purchase_stock.delta_days_merge')
        if values.get('orderpoint_id') and delta_days is not False:
            procurement_date = fields.Date.to_date(values['date_planned']) - relativedelta(days=int(values['supplier'].delay))
            delta_days = int(delta_days)
            domain += (
                ('date_order', '<=', datetime.combine(procurement_date + relativedelta(days=delta_days), datetime.max.time())),
                ('date_order', '>=', datetime.combine(procurement_date - relativedelta(days=delta_days), datetime.min.time()))
            )
        if group:
            domain += (('group_id', '=', group.id),)
        return domain

    def _push_prepare_move_copy_values(self, move_to_copy, new_date):
        res = super(StockRule, self)._push_prepare_move_copy_values(move_to_copy, new_date)
        res['purchase_line_id'] = None
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_invoice
from . import account_move_line
from . import product
from . import purchase
from . import res_config_settings
from . import res_partner
from . import res_company
from . import stock
from . import stock_move
from . import stock_rule

```

## File: populate\purchase_stock.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import models
from odoo.tools import populate, groupby

_logger = logging.getLogger(__name__)


class PurchaseOrder(models.Model):
    _inherit = "purchase.order"

    _populate_dependencies = ["res.partner", "stock.picking.type"]

    def _populate_factories(self):
        res = super()._populate_factories()
        picking_types = self.env['stock.picking.type'].search([('code', '=', 'incoming')])

        picking_types_by_company = dict(groupby(picking_types, key=lambda par: par.company_id.id))
        picking_types_inter_company = self.env['stock.picking.type'].concat(*picking_types_by_company.get(False, []))
        picking_types_by_company = {com: self.env['stock.picking.type'].concat(*pt) | picking_types_inter_company for com, pt in picking_types_by_company.items() if com}

        def get_picking_type_id(values=None, random=None, **kwargs):
            return random.choice(picking_types_by_company[values["company_id"]]).id

        return res + [
            ("picking_type_id", populate.compute(get_picking_type_id))
        ]

```

## File: populate\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_stock

```

## File: report\purchase_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import api, fields, models
from odoo.exceptions import UserError
from odoo.osv.expression import expression


class PurchaseReport(models.Model):
    _inherit = "purchase.report"

    picking_type_id = fields.Many2one('stock.warehouse', 'Warehouse', readonly=True)
    avg_receipt_delay = fields.Float(
        'Average Receipt Delay', digits=(16, 2), readonly=True, store=False,  # needs store=False to prevent showing up as a 'measure' option
        help="Amount of time between expected and effective receipt date. Due to a hack needed to calculate this, \
              every record will show the same average value, therefore only use this as an aggregated value with group_operator=avg")
    effective_date = fields.Datetime(string="Effective Date")

    def _select(self):
        return super(PurchaseReport, self)._select() + ", spt.warehouse_id as picking_type_id, po.effective_date as effective_date"

    def _from(self):
        return super(PurchaseReport, self)._from() + " left join stock_picking_type spt on (spt.id=po.picking_type_id)"

    def _group_by(self):
        return super(PurchaseReport, self)._group_by() + ", spt.warehouse_id, effective_date"

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        """ This is a hack to allow us to correctly calculate the average of PO specific date values since
            the normal report query result will duplicate PO values across its PO lines during joins and
            lead to incorrect aggregation values.

            Only the AVG operator is supported for avg_receipt_delay.
        """
        avg_receipt_delay = next((field for field in fields if re.search(r'\bavg_receipt_delay\b', field)), False)

        if avg_receipt_delay:
            fields.remove(avg_receipt_delay)
            if any(field.split(':')[1].split('(')[0] != 'avg' for field in [avg_receipt_delay] if field):
                raise UserError("Value: 'avg_receipt_delay' should only be used to show an average. If you are seeing this message then it is being accessed incorrectly.")

        res = []
        if fields:
            res = super(PurchaseReport, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

        if not res and avg_receipt_delay:
            res = [{}]

        if avg_receipt_delay:
            query = """ SELECT AVG(receipt_delay.po_receipt_delay)::decimal(16,2) AS avg_receipt_delay
                          FROM (
                              SELECT extract(epoch from age(po.effective_date, po.date_planned))/(24*60*60) AS po_receipt_delay
                              FROM purchase_order po
                              WHERE po.id IN (
                                  SELECT "purchase_report"."order_id" FROM %s WHERE %s)
                              ) AS receipt_delay
                    """

            subdomain = domain + [('company_id', '=', self.env.company.id), ('effective_date', '!=', False)]
            subtables, subwhere, subparams = expression(subdomain, self).query.get_sql()

            self.env.cr.execute(query % (subtables, subwhere), subparams)
            res[0].update({
                '__count': 1,
                avg_receipt_delay.split(':')[0]: self.env.cr.fetchall()[0][0],
            })
        return res

```

## File: report\purchase_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="report_purchaseorder_document" inherit_id="purchase.report_purchaseorder_document">
        <xpath expr="//t[@t-if='o.dest_address_id']" position="after">
            <t t-else="">
                <t t-set="information_block">
                    <strong>Shipping address:</strong>
                    <div t-if="o.picking_type_id and o.picking_type_id.warehouse_id">
                        <span t-field="o.picking_type_id.warehouse_id.name"/>
                        <div t-field="o.picking_type_id.warehouse_id.partner_id" t-options='{"widget": "contact", "fields": ["address", "phone"], "no_marker": True, "phone_icons": True}'/>
                    </div>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[@t-elif='o.date_order']" position="after">
            <div t-if="o.incoterm_id" class="col-3 bm-2">
                <strong>Incoterm:</strong>
                <p t-if="o.incoterm_location" t-out="'%s %s' % (o.incoterm_id.code, o.incoterm_location)" class="m-0"/>
                <p t-else="" t-field="o.incoterm_id.code" class="m-0"/>
            </div>
        </xpath>
    </template>

    <template id="report_purchasequotation_document" inherit_id="purchase.report_purchasequotation_document">
        <xpath expr="//t[@t-if='o.dest_address_id']" position="after">
            <t t-else="">
                <t t-set="information_block">
                    <strong>Shipping address:</strong>
                    <div t-if="o.picking_type_id and o.picking_type_id.warehouse_id">
                        <span t-field="o.picking_type_id.warehouse_id.name"/>
                        <div t-field="o.picking_type_id.warehouse_id.partner_id" t-options='{"widget": "contact", "fields": ["address", "phone"], "no_marker": True, "phone_icons": True}'/>
                    </div>
                </t>
            </t>
        </xpath>
        <xpath expr="//span[@t-field='o.name']/.." position="after">
            <div id="informations" class="row mt16 mb16">
                <div t-if="o.incoterm_id" class="col-3 bm-2">
                    <strong>Incoterm:</strong>
                    <p t-if="o.incoterm_location" t-out="'%s %s' % (o.incoterm_id.code, o.incoterm_location)" class="m-0"/>
                    <p t-else="" t-field="o.incoterm_id.code" class="m-0"/>
                </div>
            </div>
        </xpath>
    </template>

</odoo>

```

## File: report\purchase_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="purchase_report_view_search" model="ir.ui.view">
        <field name="name">purchase.report.search.stock</field>
        <field name="model">purchase.report</field>
        <field name="inherit_id" ref="purchase.view_purchase_order_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="picking_type_id"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: report\report_stock_forecasted.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="purchase_report_product_product_replenishment" inherit_id="stock.report_product_product_replenishment">
    </template>
</odoo>

```

## File: report\report_stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ReportStockRule(models.AbstractModel):
    _inherit = 'report.stock.report_stock_rule'

    @api.model
    def _get_rule_loc(self, rule, product_id):
        """ We override this method to handle buy rules which do not have a location_src_id.
        """
        res = super(ReportStockRule, self)._get_rule_loc(rule, product_id)
        if rule.action == 'buy':
            res['source'] = self.env.ref('stock.stock_location_suppliers')
        return res

```

## File: report\report_stock_rule.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
    <template id="purchase_report_stock_rule" inherit_id="stock.report_stock_rule">
        <xpath expr="//div[hasclass('o_report_stock_rule_rule')]/t" position="before">
            <t t-if="rule[0].action == 'buy'">
                <t t-if="rule[1] == 'origin'">
                    <t t-call="stock.report_stock_rule_left_arrow"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_rule')]/t[last()]" position="after">
            <t t-if="rule[0].action == 'buy'">
                <t t-if="rule[1] == 'destination'">
                    <t t-call="stock.report_stock_rule_right_arrow"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_rule_name')]/span" position="before">
            <t t-if="rule[0].action == 'buy'">
                <i class="fa fa-credit-card fa-fw" t-attf-style="color: #{color};"/>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_legend')]" position="inside">
            <div class="o_report_stock_rule_legend_line">
                <div class="o_report_stock_rule_legend_label">Buy</div>
                <div class="o_report_stock_rule_legend_symbol">
                    <div class="fa fa-credit-card fa-fw" t-attf-style="color: #{color};"/>
                </div>
            </div>
        </xpath>
    </template>
</odoo>
```

## File: report\stock_forecasted.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ReplenishmentReport(models.AbstractModel):
    _inherit = 'report.stock.report_product_product_replenishment'

    def _serialize_docs(self, docs, product_template_ids=False, product_variant_ids=False):
        res = super()._serialize_docs(docs, product_template_ids, product_variant_ids)
        res['draft_purchase_orders'] = docs['draft_purchase_orders'].read(fields=['id', 'name'])
        return res

    def _compute_draft_quantity_count(self, product_template_ids, product_variant_ids, wh_location_ids):
        res = super()._compute_draft_quantity_count(product_template_ids, product_variant_ids, wh_location_ids)
        domain = [('state', 'in', ['draft', 'sent', 'to approve'])]
        domain += self._product_purchase_domain(product_template_ids, product_variant_ids)
        warehouse_id = self.env['stock.warehouse']._get_warehouse_id_from_context()
        if warehouse_id:
            domain += [('order_id.picking_type_id.warehouse_id', '=', warehouse_id)]
        po_lines = self.env['purchase.order.line'].search(domain)
        in_sum = sum(po_lines.mapped('product_uom_qty'))
        res['draft_purchase_qty'] = in_sum
        res['draft_purchase_orders'] = po_lines.mapped("order_id").sorted(key=lambda po: po.name)
        res['draft_purchase_orders_matched'] = self.env.context.get('purchase_line_to_match_id') in po_lines.ids
        res['qty']['in'] += in_sum
        return res

    def _product_purchase_domain(self, product_template_ids, product_variant_ids):
        if product_variant_ids:
            return [('product_id', 'in', product_variant_ids)]
        elif product_template_ids:
            products = self.env['product.product'].search_read(
                [('product_tmpl_id', 'in', product_template_ids)], ['id']
            )
            product_ids = [product['id'] for product in products]
            return [('product_id', 'in', product_ids)]

```

## File: report\vendor_delay_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools
from odoo.exceptions import UserError
from odoo.osv.expression import expression


class VendorDelayReport(models.Model):
    _name = "vendor.delay.report"
    _description = "Vendor Delay Report"
    _auto = False

    partner_id = fields.Many2one('res.partner', 'Vendor', readonly=True)
    product_id = fields.Many2one('product.product', 'Product', readonly=True)
    category_id = fields.Many2one('product.category', 'Product Category', readonly=True)
    date = fields.Datetime('Effective Date', readonly=True)
    qty_total = fields.Float('Total Quantity', readonly=True)
    qty_on_time = fields.Float('On-Time Quantity', readonly=True)
    on_time_rate = fields.Float('On-Time Delivery Rate', readonly=True)

    def init(self):
        tools.drop_view_if_exists(self.env.cr, 'vendor_delay_report')
        self.env.cr.execute("""
CREATE OR replace VIEW vendor_delay_report AS(
SELECT m.id                     AS id,
       m.date                   AS date,
       m.purchase_line_id       AS purchase_line_id,
       m.product_id             AS product_id,
       Min(pc.id)               AS category_id,
       Min(po.partner_id)       AS partner_id,
       Min(m.product_qty)       AS qty_total,
       Sum(CASE
             WHEN (m.state = 'done' and pol.date_planned::date >= m.date::date) THEN (ml.qty_done / ml_uom.factor * pt_uom.factor)
             ELSE 0
           END)                 AS qty_on_time
FROM   stock_move m
       JOIN purchase_order_line pol
         ON pol.id = m.purchase_line_id
       JOIN purchase_order po
         ON po.id = pol.order_id
       JOIN product_product p
         ON p.id = m.product_id
       JOIN product_template pt
         ON pt.id = p.product_tmpl_id
       JOIN uom_uom pt_uom
         ON pt_uom.id = pt.uom_id
       JOIN product_category pc
         ON pc.id = pt.categ_id
       LEFT JOIN stock_move_line ml
         ON ml.move_id = m.id
       LEFT JOIN uom_uom ml_uom
         ON ml_uom.id = ml.product_uom_id
GROUP  BY m.id
)""")

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if all('on_time_rate' not in field for field in fields):
            res = super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)
            return res

        for field in fields:
            if 'on_time_rate' not in field:
                continue

            fields.remove(field)

            agg = field.split(':')[1:]
            if agg and agg[0] != 'sum':
                raise NotImplementedError('Aggregate functions other than \':sum\' are not allowed.')

            qty_total = field.replace('on_time_rate', 'qty_total')
            if qty_total not in fields:
                fields.append(qty_total)
            qty_on_time = field.replace('on_time_rate', 'qty_on_time')
            if qty_on_time not in fields:
                fields.append(qty_on_time)
            break

        res = super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

        for group in res:
            if group['qty_total'] == 0:
                on_time_rate = 100
            else:
                on_time_rate = group['qty_on_time'] / group['qty_total'] * 100
            group.update({'on_time_rate': on_time_rate})

        return res

```

## File: report\vendor_delay_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="vendor_delay_report_filter" model="ir.ui.view">
        <field name="name">vendor.delay.report.search</field>
        <field name="model">vendor.delay.report</field>
        <field name="arch" type="xml">
            <search string="On-time Delivery">
                <field name="partner_id"/>
                <field name="product_id"/>
                <filter string="Effective Date Last Year" name="later_than_a_year_ago"  domain="[('date', '&gt;=', ((context_today()-relativedelta(years=1)).strftime('%Y-%m-%d')))]"/>
            </search>
            </field>
    </record>

    <record id="vendor_delay_report_view_graph" model="ir.ui.view">
        <field name="name">vendor.delay.report.view.graph</field>
        <field name="model">vendor.delay.report</field>
        <field name="arch" type="xml">
            <graph string="On-Time Delivery" sample="1" disable_linking="1">
                <field name="product_id"/>
                <field name="on_time_rate" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="action_purchase_vendor_delay_report" model="ir.actions.act_window">
        <field name="name">On-time Delivery</field>
        <field name="res_model">vendor.delay.report</field>
        <field name="view_mode">graph</field>
        <field name="search_view_id" ref="vendor_delay_report_filter"/>
        <field name="help">Vendor On-time Delivery analysis</field>
        <field name="target">current</field>
        <field name="context">{'search_default_later_than_a_year_ago':1}</field>
    </record>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_report
from . import stock_forecasted
from . import report_stock_rule
from . import vendor_delay_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_purchase_order_stock_worker,purchase.order,model_purchase_order,stock.group_stock_user,1,0,0,0
access_purchase_order_line_stock_worker,purchase.order.line,model_purchase_order_line,stock.group_stock_user,1,0,0,0
access_stock_location_purchase_user,stock.location,stock.model_stock_location,purchase.group_purchase_user,1,0,0,0
access_stock_warehouse_purchase_user,stock.warehouse,stock.model_stock_warehouse,purchase.group_purchase_user,1,0,0,0
access_stock_picking_purchase_user,stock.picking,stock.model_stock_picking,purchase.group_purchase_user,1,1,1,1
access_stock_move_purchase_user,stock.move,stock.model_stock_move,purchase.group_purchase_user,1,1,1,0
access_stock_location_purchase_user_manager,stock.location,stock.model_stock_location,purchase.group_purchase_manager,1,0,0,0
access_stock_warehouse_purchase_user_manager,stock.warehouse,stock.model_stock_warehouse,purchase.group_purchase_manager,1,0,0,0
access_stock_picking_purchase_user_manager,stock.picking,stock.model_stock_picking,purchase.group_purchase_manager,1,1,1,1
access_stock_move_purchase_user_manager,stock.move,stock.model_stock_move,purchase.group_purchase_manager,1,1,1,1
access_stock_warehouse_orderpoint_manager,stock.warehouse.orderpoint,stock.model_stock_warehouse_orderpoint,purchase.group_purchase_manager,1,0,0,0
access_stock_warehouse_orderpoint_user,stock.warehouse.orderpoint,stock.model_stock_warehouse_orderpoint,purchase.group_purchase_user,1,0,0,0
access_report_purchase_order,vendor.delay.report,model_vendor_delay_report,purchase.group_purchase_manager,1,0,0,0
access_report_purchase_order_user,vendor.delay.report user,model_vendor_delay_report,purchase.group_purchase_user,1,0,0,0

```

## File: static\src\js\tours\purchase_stock.js

```javascript
odoo.define('purchase_stock.purchase_steps', function (require) {
"use strict";

var core = require('web.core');

var _t = core._t;
var PurchaseAdditionalTourSteps = require('purchase.purchase_steps');

PurchaseAdditionalTourSteps.include({

    init: function() {
        this._super.apply(this, arguments);
    },

    _get_purchase_stock_steps: function () {
        this._super.apply(this, arguments);
        return [{
            trigger: ".oe_button_box button[name='action_view_picking']",
            extra_trigger: ".oe_button_box button[name='action_view_picking']",
            content: _t("Receive the ordered products."),
            position: "bottom",
            run: 'click',
        }, {
            trigger: ".o_statusbar_buttons button[name='button_validate']",
            content: _t("Validate the receipt of all ordered products."),
            position: "bottom",
            run: 'click',
        }, {
            trigger: ".modal-footer .btn-primary",
            extra_trigger: ".modal-dialog",
            content: _t("Process all the receipt quantities."),
            position: "bottom",
        }, {
            trigger: ".o_back_button a, .breadcrumb-item:not('.active'):last",
            content: _t('Go back to the purchase order to generate the vendor bill.'),
            position: 'bottom',
        }, {
            trigger: ".o_statusbar_buttons button[name='action_create_invoice']",
            content: _t("Generate the draft vendor bill."),
            position: "bottom",
            run: 'click',
        }
        ];
    }
});

return PurchaseAdditionalTourSteps;

});

```

## File: static\src\purchase_stock_forecasted\forecasted_details.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="purchase_stock.ForecastedDetails" owl="1" t-inherit="stock.ForecastedDetails" t-inherit-mode="extension">
         <xpath expr="//tr[@name='draft_picking_in']" position="after">
            <tr t-if="props.docs.draft_purchase_qty" name="draft_po_in" t-attf-class="#{props.docs.draft_purchase_orders_matched and 'o_grid_match table-info'}">
                <td colspan="2">Requests for quotation</td>
                <td t-out="_formatFloat(props.docs.draft_purchase_qty)" class="text-end"/>
                <td>
                    <t t-foreach="props.docs.draft_purchase_orders" t-as="purchase_order" t-key="purchase_order_index">
                        <t t-if="purchase_order_index > 0"> | </t>
                        <a t-attf-href="#" t-out="purchase_order.name"
                            class="fw-bold"
                            t-on-click.prevent="() => this.props.openView('purchase.order', 'form', purchase_order.id)"/>
                    </t>
                </td>
            </tr>
        </xpath>
    </t>
</templates>

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="purchase_order_view_form_inherit" model="ir.ui.view">
        <field name="name">purchase.order.form.inherit</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header/button[@name='action_rfq_send']" position="after">
                <button name="action_view_picking" 
                    string="Receive Products" class="oe_highlight" type="object" 
                    attrs="{'invisible': ['|', '|' , ('is_shipped', '=', True), ('state','not in', ('purchase','done')), ('incoming_picking_count', '=', 0)]}" 
                    data-hotkey="y" groups="stock.group_stock_user"/>
            </xpath>
            <xpath expr="//header/button[@name='confirm_reminder_mail']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', '|', '|', ('state', 'not in', ('purchase', 'done')), ('mail_reminder_confirmed', '=', True), ('date_planned', '=', False), ('effective_date', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button type="object"
                    name="action_view_picking"
                    class="oe_stat_button"
                    icon="fa-truck" attrs="{'invisible':[('incoming_picking_count','=', 0)]}" groups="stock.group_stock_user">
                    <field name="incoming_picking_count" widget="statinfo" string="Receipt" help="Incoming Shipments"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='currency_id']" position="after">
                <field name="is_shipped" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/tree//field[@name='date_planned']" position="after">
                <field name="move_dest_ids" invisible="1"/>
            </xpath>
            <xpath expr="//page/field[@name='order_line']/tree/field[@name='product_qty']" position="after">
                <field name="forecasted_issue" invisible="1"/>
                <button type="object" name="action_product_forecast_report" title="Forecast Report" icon="fa-area-chart" attrs="{'invisible': ['|', '|', ('id', '=', False), ('forecasted_issue', '=', False), ('product_type', '!=', 'product')]}" class="text-danger"/>
                <button type="object" name="action_product_forecast_report" title="Forecast Report" icon="fa-area-chart" attrs="{'invisible': ['|', '|', ('id', '=', False), ('forecasted_issue', '=', True), ('product_type', '!=', 'product')]}"/>
            </xpath>
            <xpath expr="//div[@name='date_planned_div']" position="inside">
                <button name="%(action_purchase_vendor_delay_report)d" class="oe_link" type="action" context="{'search_default_partner_id': partner_id}" attrs="{'invisible': ['|', ('state', 'in', ['purchase', 'done']), ('partner_id', '=', False)]}">
                    <span attrs="{'invisible': [('on_time_rate', '&lt;', 0)]}"><field name="on_time_rate" widget="integer" class="oe_inline"/>% On-Time Delivery</span>
                    <span attrs="{'invisible': [('on_time_rate', '&gt;=', 0)]}">No On-time Delivery Data</span>
                </button>
            </xpath>
            <xpath expr="//label[@for='receipt_reminder_email']" position="attributes">
                <attribute name="attrs">{'invisible': [('effective_date', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//div[@name='reminder']" position="attributes">
                <attribute name="attrs">{'invisible': [('effective_date', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//div[@name='reminder']" position="after">
                <field name="effective_date" attrs="{'invisible': [('effective_date', '=', False)]}"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/form//field[@name='invoice_lines']" position="after">
                <field name="move_ids"/>
            </xpath>
            <field name="invoice_status" position="before">
                <field name="receipt_status" attrs="{'invisible': [('state', 'not in', ('purchase', 'done'))]}"/>
            </field>
            <xpath expr="//field[@name='order_line']/form//field[@name='analytic_distribution']" position="before">
                <field name="propagate_cancel" groups="base.group_no_one"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/tree//field[@name='analytic_distribution']" position="before">
                <field name="propagate_cancel" groups="base.group_no_one" optional="hide"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/tree//field[@name='qty_received']" position="attributes">
                <attribute name="attrs">{'column_invisible': [('parent.state', 'not in', ('purchase', 'done'))], 'readonly': [('product_type', 'in', ('consu', 'product'))]}</attribute>
            </xpath>
            <xpath expr="//page[@name='purchase_delivery_invoice']/group/group" position="inside">
                <field name="default_location_dest_id_usage" invisible="1"/>
                <field name="incoterm_id"/>
                <field name="incoterm_location" />
            </xpath>
            <xpath expr="//div[@name='reminder']" position="after">
                <field name="picking_type_id" domain="[('code','=','incoming'), '|', ('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                <field name="dest_address_id" groups="stock.group_stock_multi_locations" attrs="{'invisible': [('default_location_dest_id_usage', '!=', 'customer')], 'required': [('default_location_dest_id_usage', '=', 'customer')]}"/>
            </xpath>
        </field>
    </record>

    <record id="purchase_order_line_view_form_inherit" model="ir.ui.view">
        <field name="name">purchase.order.line.form.inherit</field>
        <field name="model">purchase.order.line</field>
        <field name="inherit_id" ref="purchase.purchase_order_line_form2"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='invoice_lines']" position="after">
                <separator string="Stock Moves"/>
                <field name="move_ids"/>
            </xpath>
        </field>
    </record>

    <record id="purchase_order_view_tree_inherit" model="ir.ui.view">
        <field name="name">purchase.order.tree.inherit</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_view_tree"/>
        <field name="arch" type="xml">
            <field name="invoice_status" position="before">
                <field name="effective_date" invisible="1"/>
                <field name="receipt_status" optional="hide" widget="badge"
                decoration-success="date_planned and (date_planned &gt; datetime.datetime.combine(datetime.date.today(), datetime.time(23,59,59)).to_utc().strftime('%Y-%m-%d %H:%M:%S') or receipt_status=='full' and effective_date &lt;= date_planned)"
                decoration-danger="date_planned and (date_planned &lt; effective_date or date_planned &lt; datetime.datetime.combine(datetime.date.today(), datetime.time(0,0,0)).to_utc().strftime('%Y-%m-%d %H:%M:%S') and receipt_status!='full' and effective_date &lt;= date_planned)"
                decoration-warning="date_planned and date_planned &gt;= datetime.datetime.combine(datetime.date.today(), datetime.time(0,0,0)).to_utc().strftime('%Y-%m-%d %H:%M:%S') and date_planned &lt;= datetime.datetime.combine(datetime.date.today(), datetime.time(23,59,59)).to_utc().strftime('%Y-%m-%d %H:%M:%S') and receipt_status!='full'"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

	<record id="res_config_settings_view_form_purchase" model="ir.ui.view">
		<field name="name">res.config.settings.view.form.inherit.purchase</field>
		<field name="model">res.config.settings</field>
		<field name="priority" eval="25"/>
		<field name="inherit_id" ref="purchase.res_config_settings_view_form_purchase"/>
		<field name="arch" type="xml">
			<xpath expr="//div[@data-key='purchase']" position="inside">
				<field name="is_installed_sale" invisible="1"/>
				<h2 attrs="{'invisible': [('is_installed_sale', '=', False)]}">Logistics</h2>
				<div class="row mt16 o_settings_container" name="request_vendor_setting_container">
					<div class="col-12 col-lg-6 o_setting_box" attrs="{'invisible': [('is_installed_sale', '=', False)]}" title="This adds a dropshipping route to apply on products in order to request your vendors to deliver to your customers. A product to dropship will generate a purchase request for quotation once the sales order confirmed. This is a on-demand flow. The requested delivery address will be the customer delivery address and not your warehouse.">
						<div class="o_setting_left_pane">
							<field name="module_stock_dropshipping"/>
						</div>
						<div class="o_setting_right_pane">
							<label for="module_stock_dropshipping"/>
							<a href="https://www.odoo.com/documentation/16.0/applications/inventory_and_mrp/inventory/shipping/operation/dropshipping.html" title="Documentation" class="o_doc_link" target="_blank"></a>
							<div class="text-muted">
								Request your vendors to deliver to your customers
							</div>
						</div>
					</div>
				</div>
			</xpath>
		</field>
	</record>

	<record id="res_config_settings_view_form_stock" model="ir.ui.view">
		<field name="name">res.config.settings.view.form.inherit.purchase.stock</field>
		<field name="model">res.config.settings</field>
		<field name="inherit_id" ref="stock.res_config_settings_view_form"/>
		<field name="arch" type="xml">
			<xpath expr="//h2[@id='schedule_info']" position="attributes">
				<attribute name="invisible">0</attribute>
			</xpath>
			<div id="purchase_po_lead" position="replace">
				<div class="col-12 col-lg-6 o_setting_box"
                                    title="Margin of error for vendor lead times. When the system generates Purchase Orders for reordering products,they will be scheduled that many days earlier to cope with unexpected vendor delays."
                                    name="schedule_receivings_setting_container">
					<div class="o_setting_left_pane">
						<field name="use_po_lead"/>
					</div>
					<div class="o_setting_right_pane">
						<label for="use_po_lead"/>
						<a href="https://www.odoo.com/documentation/16.0/applications/inventory_and_mrp/inventory/management/planning/scheduled_dates.html" title="Documentation" class="me-2 o_doc_link" target="_blank"></a>
						<span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
						<div class="text-muted">
							Schedule request for quotations earlier to avoid delays
						</div>
						<div class="content-group">
							<div class="mt16" attrs="{'invisible': [('use_po_lead','=',False)]}">
								<span>Move forward expected request creation date by <field name="po_lead" class="oe_inline"/> days</span>
							</div>
						</div>
					</div>
				</div>
				<div class="col-12 col-lg-6 o_setting_box">
					<div class="o_setting_right_pane">
						<label for="days_to_purchase"/>
						<span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
						<div class="text-muted">
							Days needed to confirm a PO
						</div>
						<div class="content-group">
							<div class="mt16">
								<span><field name="days_to_purchase" class="oe_inline"/> days</span>
							</div>
						</div>
					</div>
				</div>
			</div>
		</field>
	</record>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_partner_view_purchase_buttons_inherit" model="ir.ui.view">
            <field name="name">res.partner.purchase.stock.form.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="purchase.res_partner_view_purchase_buttons"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='purchase_order_count']/.." position="after">
                    <button class="oe_stat_button" name="%(action_purchase_vendor_delay_report)d" type="action"
                        groups="purchase.group_purchase_user"
                        icon="fa-truck"
                        context="{'search_default_partner_id': id}">
                        <div class="o_form_field o_stat_info">
                            <div class="o_row" attrs="{'invisible': [('on_time_rate', '&lt;', 0)]}">
                                <span class="o_stat_value">
                                    <field string="On-time Rate" name="on_time_rate" widget="integer"/>
                                </span>
                                <span class="o_stat_value">%</span>
                            </div>
                            <div class="o_stat_value" attrs="{'invisible': [('on_time_rate', '&gt;=', 0)]}">
                                No data yet
                            </div>
                            <span class="o_stat_text">On-time Rate</span>
                        </div>
                    </button>
                </xpath>
            </field>
    </record>
</odoo>

```

## File: views\stock_lot_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_production_lot_view_form" model="ir.ui.view">
        <field name="name">stock.production.lot.view.form</field>
        <field name="model">stock.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]/button" position="before">
                <button class="oe_stat_button" name="action_view_po"
                        type="object" icon="fa-credit-card" help="Purchase Orders"
                        attrs="{'invisible': ['|', ('purchase_order_count', '=', 0), ('display_complete', '=', False)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="purchase_order_count" widget="statinfo" nolabel="1" class="mr4"/>
                        </span>
                        <span class="o_stat_text">Purchases</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//group[@name='main_group']" position="after">
                <group>
                    <field name="purchase_order_ids" widget="many2many" readonly="True"
                           attrs="{'invisible': ['|', ('purchase_order_ids', '=', []), ('display_complete', '=', True)]}">
                        <tree>
                            <field name="name"/>
                            <field name="partner_id"/>
                            <field name="date_order"/>
                            <field name="state" invisible="1"/>
                        </tree>
                    </field>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_rule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Stock rules -->
        <record id="view_stock_rule_form_stock_inherit_purchase_stock" model="ir.ui.view">
            <field name="name">stock.rule.form.stock.inherit.purchase_stock</field>
            <field name="model">stock.rule</field>
            <field name="inherit_id" ref="stock.view_stock_rule_form"/>
            <field name="arch" type="xml">
                <field name="location_src_id" position="attributes">
                    <attribute name="attrs">{'required': [('action', 'in', ['pull', 'push', 'pull_push'])], 'invisible': [('action', '=', 'buy')]}</attribute>
                </field>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\stock_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="stock_move_purchase" model="ir.ui.view">
            <field name="name">stock.move.form</field>
            <field name="model">stock.move</field>
            <field name="inherit_id" ref="stock.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='origin_grp']" position="inside">
                    <field name="purchase_line_id"/>
                </xpath>
            </field>
        </record>

        <record id="view_warehouse_inherited" model="ir.ui.view">
            <field name="name">Stock Warehouse Inherited</field>
            <field name="model">stock.warehouse</field>
            <field name="inherit_id" ref="stock.view_warehouse"/>
            <field name="arch" type="xml">
                 <xpath expr="//field[@name='resupply_wh_ids']" position="before">
                    <field name="buy_to_resupply"/>
                </xpath>
                <xpath expr="//group[@name='group_resupply']" position="attributes">
                    <attribute name="groups">stock.group_adv_location,stock.group_stock_multi_warehouses</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_warehouse_orderpoint_tree_editable_inherited_mrp" model="ir.ui.view">
            <field name="name">stock.warehouse.orderpoint.tree.editable.inherit.mrp</field>
            <field name="model">stock.warehouse.orderpoint</field>
            <field name="inherit_id" ref="stock.view_warehouse_orderpoint_tree_editable"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_stock_replenishment_info']" position="before">
                    <field name="show_supplier" invisible="1"/>
                    <field name="supplier_id" string="Vendor" optional="show" attrs="{'invisible': [('show_supplier', '=', False)]}" options="{'no_create': True}"/>
                </xpath>
            </field>
        </record>

</odoo>

```

## File: wizard\stock_replenishment_info.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockReplenishmentInfo(models.TransientModel):
    _inherit = 'stock.replenishment.info'
    _description = 'Stock supplier replenishment information'

    supplierinfo_id = fields.Many2one(related='orderpoint_id.supplier_id')
    supplierinfo_ids = fields.Many2many(
        'product.supplierinfo', compute='_compute_supplierinfo_ids',
        store=True)

    @api.depends('orderpoint_id')
    def _compute_supplierinfo_ids(self):
        for replenishment_info in self:
            replenishment_info.supplierinfo_ids = replenishment_info.product_id.seller_ids

```

## File: wizard\stock_replenishment_info.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <record id="product_supplierinfo_replenishment_tree_view" model="ir.ui.view">
        <field name="name">product.supplierinfo.replenishment.tree.view</field>
        <field name="model">product.supplierinfo</field>
        <field name="inherit_id" ref="product.product_supplierinfo_tree_view"/>
        <field name="mode">primary</field>
        <field name="priority">100</field>
        <field name="arch" type="xml">
            <field name="delay" position="after">
                <field name="show_set_supplier_button" invisible="1"/>
                <field name="last_purchase_date"/>
                <button name="action_set_supplier" type="object" string="Set as Supplier" class="btn btn-link"
                        context="{'orderpoint_id': parent.orderpoint_id, 'stock_replenishment_info_id': parent.id}"
                        attrs="{'invisible': [('show_set_supplier_button', '=', False)]}"/>
            </field>
            <xpath expr="//tree" position="attributes">
                <attribute name="decoration-bf">parent.supplierinfo_id == id</attribute>
            </xpath>
            <field name="min_qty" position="attributes">
                <attribute name="decoration-danger">min_qty &gt; parent.qty_to_order</attribute>
            </field>
            <field name="company_id" position="attributes">
                <attribute name="optional">hide</attribute>
            </field>
            <field name="delay" position="attributes">
                <attribute name="optional">show</attribute>
            </field>
        </field>
    </record>

    <record id="view_stock_replenishment_info_stock_purchase_inherit" model="ir.ui.view">
        <field name="name">stock.replenishment.information.purchase.stock.inherit</field>
        <field name="model">stock.replenishment.info</field>
        <field name="inherit_id" ref="stock.view_stock_replenishment_info"/>
        <field name="arch" type="xml">
            <xpath expr="//page" position="before">
                <page string="Vendors">
                    <field name="supplierinfo_id" invisible="1"/>
                    <field name="supplierinfo_ids" readonly="1" context="{'tree_view_ref': 'purchase_stock.product_supplierinfo_replenishment_tree_view'}"/>
                </page>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_replenishment_info

```

