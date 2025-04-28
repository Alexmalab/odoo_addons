# Odoo Module: pos_cash_rounding

Category: Sales/Point Of Sale

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
    'name': 'Point of Sale Cash Rounding',
    'version': '1.0.0',
    'category': 'Sales/Point Of Sale',
    'sequence': 20,
    'summary': 'Allow specific rounding in pos',
    'description': "",
    'depends': ['point_of_sale'],
    'data': [
        'security/ir.model.access.csv',
        'views/res_config_settings_view.xml',
        'views/pos_config_view.xml',
        'views/account_cash_rounding_view.xml',
        'views/pos_order_view.xml',
        'views/pos_template.xml',
    ],
    'qweb': [
        'static/src/xml/pos.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_cash_rounding.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError

class AccountCashRounding(models.Model):
    _inherit = 'account.cash.rounding'

    loss_account_id = fields.Many2one('account.account', string='Loss Account')

    def _get_loss_account_id(self):
        return self.loss_account_id or super(AccountCashRounding, self)._get_loss_account_id()

    @api.constrains('rounding', 'rounding_method', 'strategy')
    def _check_session_state(self):
        open_session = self.env['pos.session'].search_count([('config_id.rounding_method', '=', self.id), ('state', '!=', 'closed')])
        if open_session:
            raise ValidationError(
                _("You are not allowed to change the cash rounding configuration while a pos session using it is already opened."))

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError


class PosConfig(models.Model):
    _inherit = 'pos.config'

    rounding_method = fields.Many2one('account.cash.rounding', string="Cash rounding", domain=[('strategy', '=', 'add_invoice_line')])
    cash_rounding = fields.Boolean(string="Cash Rounding")
    only_round_cash_method = fields.Boolean(string="Only apply rounding on cash")


    @api.constrains('rounding_method')
    def _check_rounding_method_strategy(self):
        if self.cash_rounding and self.rounding_method.strategy != 'add_invoice_line':
            raise ValidationError(_("Cash rounding strategy must be: 'Add a rounding line'"))

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields
from odoo.tools import float_is_zero, float_round

class PosOrder(models.Model):
    _inherit = "pos.order"


    def _get_rounded_amount(self, amount):
        if self.config_id.cash_rounding and (
                not self.config_id.only_round_cash_method or self.payment_ids.filtered(lambda p: p.payment_method_id.is_cash_count)):
            amount = float_round(amount, precision_rounding=self.config_id.rounding_method.rounding, rounding_method=self.config_id.rounding_method.rounding_method)
        return super(PosOrder, self)._get_rounded_amount(amount)

    def _prepare_invoice_vals(self):
        vals = super(PosOrder, self)._prepare_invoice_vals()
        if self.config_id.cash_rounding and (
                not self.config_id.only_round_cash_method or self.payment_ids.filtered(lambda p: p.payment_method_id.is_cash_count)):
            vals['invoice_cash_rounding_id'] = self.config_id.rounding_method.id
        return vals

    def _create_invoice(self, move_vals):
        new_move = super(PosOrder, self)._create_invoice(move_vals)
        if self.config_id.cash_rounding:
            rounding_applied = float_round(self.amount_paid - self.amount_total, precision_rounding=new_move.currency_id.rounding)
            rounding_line = new_move.line_ids.filtered(lambda line: line.is_rounding_line)
            if rounding_line and rounding_line.debit > 0:
                rounding_line_difference = rounding_line.debit + rounding_applied
            elif rounding_line and rounding_line.credit > 0:
                rounding_line_difference = -rounding_line.credit + rounding_applied
            else:
                rounding_line_difference = rounding_applied
            if rounding_applied:
                if rounding_applied > 0.0:
                    account_id = new_move.invoice_cash_rounding_id._get_loss_account_id().id
                else:
                    account_id = new_move.invoice_cash_rounding_id._get_profit_account_id().id
                if rounding_line:
                    if rounding_line_difference:
                        rounding_line.with_context(check_move_validity=False).write({
                            'debit': rounding_applied < 0.0 and -rounding_applied or 0.0,
                            'credit': rounding_applied > 0.0 and rounding_applied or 0.0,
                            'account_id': account_id,
                            'price_unit': rounding_applied,
                        })
                else:
                    self.env['account.move.line'].with_context(check_move_validity=False).create({
                         'debit': rounding_applied < 0.0 and -rounding_applied or 0.0,
                         'credit': rounding_applied > 0.0 and rounding_applied or 0.0,
                         'quantity': 1.0,
                         'amount_currency': rounding_applied,
                         'partner_id': new_move.partner_id.id,
                         'move_id': new_move.id,
                         'currency_id': new_move.currency_id if new_move.currency_id != new_move.company_id.currency_id else False,
                         'company_id': new_move.company_id.id,
                         'company_currency_id': new_move.company_id.currency_id.id,
                         'is_rounding_line': True,
                         'sequence': 9999,
                         'name': new_move.invoice_cash_rounding_id.name,
                         'account_id': account_id,
                     })
            else:
                if rounding_line:
                    rounding_line.with_context(check_move_validity=False).unlink()
            if rounding_line_difference:
                existing_terms_line = new_move.line_ids.filtered(
                    lambda line: line.account_id.user_type_id.type in ('receivable', 'payable'))
                if existing_terms_line.debit > 0:
                    existing_terms_line_new_val = float_round(
                        existing_terms_line.debit + rounding_line_difference,
                        precision_rounding=new_move.currency_id.rounding)
                else:
                    existing_terms_line_new_val = float_round(
                        -existing_terms_line.credit + rounding_line_difference,
                        precision_rounding=new_move.currency_id.rounding)
                existing_terms_line.write({
                    'debit': existing_terms_line_new_val > 0.0 and existing_terms_line_new_val or 0.0,
                    'credit': existing_terms_line_new_val < 0.0 and -existing_terms_line_new_val or 0.0,
                })

                new_move._recompute_payment_terms_lines()
        return new_move

    def _get_amount_receivable(self):
        if self.config_id.cash_rounding and (
                not self.config_id.only_round_cash_method or self.payment_ids.filtered(lambda p: p.payment_method_id.is_cash_count)):
            return self.amount_paid
        return super(PosOrder, self)._get_amount_receivable()

    def _is_pos_order_paid(self):
        res = super(PosOrder, self)._is_pos_order_paid()
        if not res and self.config_id.cash_rounding:
            currency = self.currency_id
            if self.config_id.rounding_method.rounding_method == "HALF-UP":
                maxDiff = currency.round(self.config_id.rounding_method.rounding / 2)
            else:
                maxDiff = currency.round(self.config_id.rounding_method.rounding)

            diff = currency.round(self.amount_total - self.amount_paid)
            res = abs(diff) <= maxDiff
        return res

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields
from odoo.tools import float_is_zero, float_round, float_compare

class PosSession(models.Model):
    _inherit = "pos.session"

    def _get_rounding_difference_vals(self, amount, amount_converted):
        partial_args = {
            'name': 'Rounding line',
            'move_id': self.move_id.id,
        }
        if amount > 0:    # loss
            partial_args['account_id'] = self.config_id.rounding_method._get_loss_account_id().id
            return self._credit_amounts(partial_args, amount, amount_converted)
        else:   # profit
            partial_args['account_id'] = self.config_id.rounding_method._get_profit_account_id().id
            return self._debit_amounts(partial_args, -amount, -amount_converted)

    def _get_extra_move_lines_vals(self):
        res = super(PosSession, self)._get_extra_move_lines_vals()
        if not self.config_id.cash_rounding:
            return res
        rounding_difference = {'amount': 0.0, 'amount_converted': 0.0}
        rounding_vals = []
        for order in self.order_ids:
            if not order.is_invoiced:
                rounding_difference['amount'] += self.currency_id.round(order.amount_paid - order.amount_total)
        if not self.is_in_company_currency:
            difference = sum(self.move_id.line_ids.mapped('debit')) - sum(self.move_id.line_ids.mapped('credit'))
            rounding_difference['amount_converted'] = self.company_id.currency_id.round(difference)
        else:
            rounding_difference['amount_converted'] = rounding_difference['amount']
        if (
            not float_is_zero(rounding_difference['amount'], precision_rounding=self.currency_id.rounding)
            or not float_is_zero(rounding_difference['amount_converted'], precision_rounding=self.company_id.currency_id.rounding)
        ):
            rounding_vals += [self._get_rounding_difference_vals(rounding_difference['amount'], rounding_difference['amount_converted'])]
        return res + rounding_vals

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_cash_rounding
from . import pos_config
from . import pos_order
from . import pos_session

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_account_cash_rounding,account.cash.rounding,model_account_cash_rounding,point_of_sale.group_pos_user,1,0,0,0

```

## File: static\src\js\pos_cash_rounding.js

```javascript
odoo.define('pos_cash_rounding.cash_rounding', function (require) {
    "use strict";

var models = require('point_of_sale.models');
var rpc = require('web.rpc');
var screens = require('point_of_sale.screens');
var utils = require('web.utils');

var core    = require('web.core');
var _t      = core._t;
var round_pr = utils.round_precision;


models.load_models([{
    model: 'account.cash.rounding',
    fields: ['name', 'rounding', 'rounding_method'],
    domain: function(self){return [['id', '=', self.config.rounding_method[0]]]; },
    loaded: function(self, cash_rounding) {
        self.cash_rounding = cash_rounding;
    }
},
]);

var _super_order = models.Order.prototype;
models.Order = models.Order.extend({
    export_for_printing: function() {
      var result = _super_order.export_for_printing.apply(this,arguments);
      result.total_rounded = this.get_total_with_tax() + this.get_rounding_applied();
      result.rounding_applied = this.get_rounding_applied();
      return result;
    },
    get_due: function(paymentline) {
      var due  = _super_order.get_due.apply(this, arguments);
      due += this.get_rounding_applied();
      return round_pr(due, this.pos.currency.rounding);
    },
    add_paymentline: function(payment_method){
        _super_order.add_paymentline.apply(this, arguments);
        if(this.pos.config.cash_rounding && (!payment_method.is_cash_count || this.pos.config.iface_precompute_cash)){
          this.selected_paymentline.set_amount(0);
          this.selected_paymentline.set_amount(this.get_due());
        }
    },
    get_change_value: function(paymentline) {
      var change  = _super_order.get_change_value.apply(this, arguments);
      change -= this.get_rounding_applied();
      return round_pr(change, this.pos.currency.rounding);
    },
    get_rounding_applied: function() {
        if(this.pos.config.cash_rounding) {
            const only_cash = this.pos.config.only_round_cash_method;
            const paymentlines = this.get_paymentlines();
            const last_line = paymentlines ? paymentlines[paymentlines.length-1]: false;
            const last_line_is_cash = last_line ? last_line.payment_method.is_cash_count == true: false;
            if (!only_cash || (only_cash && last_line_is_cash)) {
                var remaining = this.get_total_with_tax() - this.get_total_paid();
                var total = round_pr(remaining, this.pos.cash_rounding[0].rounding);
                var sign = remaining > 0 ? 1.0 : -1.0;

                var rounding_applied = total - remaining;
                rounding_applied *= sign;
                // because floor and ceil doesn't include decimals in calculation, we reuse the value of the half-up and adapt it.
                if (utils.float_is_zero(rounding_applied, this.pos.currency.decimals)){
                    // https://xkcd.com/217/
                    return 0;
                } else if(this.get_total_with_tax() < this.pos.cash_rounding[0].rounding) {
                    return 0;
                } else if(this.pos.cash_rounding[0].rounding_method === "UP" && rounding_applied < 0 && remaining > 0) {
                    rounding_applied += this.pos.cash_rounding[0].rounding;
                } else if(this.pos.cash_rounding[0].rounding_method === "UP" && rounding_applied > 0 && remaining < 0) {
                    rounding_applied -= this.pos.cash_rounding[0].rounding;
                }  else if(this.pos.cash_rounding[0].rounding_method === "DOWN" && rounding_applied > 0 && remaining > 0){
                    rounding_applied -= this.pos.cash_rounding[0].rounding;
                } else if(this.pos.cash_rounding[0].rounding_method === "DOWN" && rounding_applied < 0 && remaining < 0){
                    rounding_applied += this.pos.cash_rounding[0].rounding;
                }
                return sign * rounding_applied;
            } else {
                return 0;
            }
        }
        return 0;
    },
    check_paymentlines_rounding: function() {
        if(this.pos.config.cash_rounding) {
            var cash_rounding = this.pos.cash_rounding[0].rounding;
            var default_rounding = this.pos.currency.rounding;
            for(var id in this.get_paymentlines()) {
                var line = this.get_paymentlines()[id];
                var diff = round_pr(round_pr(line.amount, cash_rounding) - round_pr(line.amount, default_rounding), default_rounding);
                if(this.get_total_with_tax() < this.pos.cash_rounding[0].rounding)
                    return true;
                if(diff && line.payment_method.is_cash_count) {
                    return false;
                } else if(!this.pos.config.only_round_cash_method && diff) {
                    return false;
                }
            }
            return true;
        }
        return true;
    },
    get_total_balance: function() {
        return this.get_total_with_tax() - this.get_total_paid() + this.get_rounding_applied();
    },
    is_paid: function() {
        var is_paid = _super_order.is_paid.apply(this, arguments);
        return is_paid && this.check_paymentlines_rounding();
    }
});
    screens.PaymentScreenWidget.include({
        validate_order: function(force_validation) {
            if(this.pos.config.cash_rounding) {
                if(!this.pos.get_order().check_paymentlines_rounding()) {
                    this.pos.gui.show_popup('error', {
                        'title': _t("Rounding error in payment lines"),
                        'body': _t("The amount of your payment lines must be rounded to validate the transaction."),
                    });
                    return;
                }
            }
            this._super(event);
        },
    });
});

```

## File: static\src\xml\pos.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-extend="OrderReceipt">
        <t t-jquery='.pos-receipt-amount:first' t-operation='after'>
          <t t-if="receipt.total_rounded != receipt.total_with_tax">
              <div class="pos-receipt-amount">
                  Rounding
                  <span t-esc='widget.format_currency(receipt.rounding_applied)' class="pos-receipt-right-align"/>
              </div>
              <div class="pos-receipt-amount">
                  To Pay
                  <span t-esc='widget.format_currency(receipt.total_rounded)' class="pos-receipt-right-align"/>
              </div>
          </t>
        </t>
    </t>
    <t t-extend="PaymentScreen-Paymentlines">
        <t t-jquery='.paymentlines-empty > .total' t-operation='replace'>
            <div class='total'>
                <t t-esc="widget.format_currency(order.get_total_with_tax())"/>
            </div>
        </t>
    </t>
</templates>

```

## File: views\account_cash_rounding_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_rounding_form_view_inherited" model="ir.ui.view">
        <field name="name">pos.cash.rounding.form.inherited</field>
        <field name="model">account.cash.rounding</field>
        <field name="inherit_id" ref="account.rounding_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_title')]" position="before">
                <div class="o_notification_alert alert alert-warning" role="alert">
                  The Point of Sale only support the "add a rounding line" rounding strategy.
                </div>
            </xpath>
            <xpath expr="//field[@name='account_id']" position="after">
                <field name="loss_account_id" options="{'no_create': True}" attrs="{'invisible': [('strategy', '!=', 'add_invoice_line')]}" domain="[('user_type_id.type', 'not in', ('receivable', 'payable'))]"/>
            </xpath>
            <xpath expr="//field[@name='account_id']" position="attributes">
                  <attribute name="string">Profit Account</attribute>
                  <attribute name="groups"></attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_config_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_config_view_form_inherit_cash_rounding" model="ir.ui.view">
        <field name="name">pos.config.form.inherit.cash_rounding</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='payment_methods_new']" position="after">
                <div class="col-12 col-lg-6 o_setting_box" groups="account.group_cash_rounding">
                    <div class="o_setting_left_pane">
                        <field name="cash_rounding"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="cash_rounding"/>
                        <div class="text-muted">
                            Define the smallest coinage of the currency used to pay
                        </div>
                        <div class="content-group mt16" attrs="{'invisible': [('cash_rounding', '=', False)]}">
                            <div class="row mt16">
                                <label string="Rounding Method" for="rounding_method" class="col-lg-3 o_light_label" />
                                <field name="rounding_method" attrs="{'required' : [('cash_rounding', '=', True)]}" domain="[('company_id', '=', company_id)]"/>
                            </div>
                            <div class="row mt16">
                              <label string="Only on cash methods" for="only_round_cash_method" class="col-lg-3 o_light_label" />
                              <field name="only_round_cash_method"/>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_order_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_order_view_form_inherit_cash_rounding" model="ir.ui.view">
        <field name="name">pos.order.form.inherit.cash_rounding</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='amount_total']" position="after">
                <field name="amount_paid"
                    string="Total Paid (with rounding)"
                    class="oe_subtotal_footer_separator"
                    widget="monetary"
                    attrs="{'invisible': [('amount_paid','=', 'amount_total')]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <template id="assets" inherit_id="point_of_sale.assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/pos_cash_rounding/static/src/js/pos_cash_rounding.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_view_form_inherit_pos_cash_rounding" model="ir.ui.view">
        <field name="name">res.config.form.inherit.pos.cash_rounding</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@data-key='point_of_sale']" position="inside" >
                <h2>Payments</h2>
                <div class="row mt16 o_settings_container">
                    <div class="col-12 col-lg-6 o_setting_box">
                        <div class="o_setting_left_pane">
                            <field name="group_cash_rounding"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="group_cash_rounding"/>
                            <div class="text-muted">
                                Define the smallest coinage of the currency used to pay by cash
                            </div>
                            <div class="mt8">
                                <button name="%(account.rounding_list_action)d" icon="fa-arrow-right"
                                        type="action" string="Cash Roundings" class="btn-link"
                                        attrs="{'invisible': [('group_cash_rounding', '=', False)]}"/>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

