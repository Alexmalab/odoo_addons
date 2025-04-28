# Odoo Module: sale_gift_card

Category: Sales/Sales

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
    'name': "Gift Card for sales module",
    'summary': "Use gift card in your sales orders",
    'description': """Integrate gift card mechanism in sales orders.""",
    'category': 'Sales/Sales',
    'version': '1.0',
    'depends': ['gift_card', 'sale'],
    'auto_install': True,
    'data': [
        'data/gift_card_data.xml',
        'data/mail_template_data.xml',
        'views/sale_order_view.xml',
        'views/templates.xml',
        'security/ir.model.access.csv',
    ],
    'license': 'LGPL-3',
}

```

## File: data\gift_card_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- the product used to deduce on sale order -->
        <record id="gift_card.pay_with_gift_card_product" model="product.product">
            <field name="taxes_id" eval="False"/>
            <field name="supplier_taxes_id" eval="False"/>
        </record>

        <record id="gift_card.gift_card_product_50" model="product.product">
            <field name="taxes_id" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <record id="mail_template_gift_card" model="mail.template">
        <field name="name">Gift Card: Send by Email</field>
        <field name="model_id" ref="model_gift_card"/>
        <field name="subject">Your Gift Card</field>
        <field name="partner_to">{{ object.partner_id.id or object.buy_line_id.order_id.partner_id.id }}</field>
        <field name="body_html" type="html">
            <div style="margin:0px; font-size:24px; font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:36px; color:#333333; text-align: center">
                Here is your gift card!
            </div>
            <div style="padding-top:20px; padding-bottom:20px">
                <img src="/gift_card/static/img/gift_card.png" style="display:block; border:0; outline:none; text-decoration:none; margin:auto;" width="300"/>
            </div>
            <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; text-align:center;">
                <h3 style="margin:0px; line-height:48px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:40px; font-style:normal; font-weight:normal; color:#333333; text-align:center">
                    <strong t-out="format_amount(object.initial_amount, object.currency_id) or ''">$ 150.00</strong></h3>
            </div>
            <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; background-color:#efefef; text-align:center;">
                <p style="margin:0px; font-size:14px;font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:21px; color:#333333">
                    <strong>Gift Card Code</strong>
                </p>
                <p style="margin:0px; font-size:25px;font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:38px; color:#A9A9A9" t-out="object.code or ''">4f10-15d6-41b7-b04c-7b3e</p>
            </div>
            <div style="padding:0; margin:0px; padding-top:10px; padding-bottom:10px; text-align:center;">
                <h3 style="margin:0px; line-height:17px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:14px; font-style:normal; font-weight:normal; color:#A9A9A9; text-align:center">Card expires <t t-out="format_date(object.expired_date) or ''">05/05/2021</t></h3>
            </div>
            <div style="padding:20px; margin:0px; text-align:center;">
                <span style="background-color:#999999; display:inline-block; width:auto; border-radius:5px;">
                    <a t-attf-href="{{ object.buy_line_id.order_id.get_base_url() }}/shop" target="_blank" style="text-decoration:none; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:22px; color:#FFFFFF; border-style:solid; border-color:#999999; border-width:20px 30px; display:inline-block; background-color:#999999; border-radius:5px; font-weight:bold; font-style:normal; line-height:26px; width:auto; text-align:center">Use it right now!</a>
                </span>
            </div>
        </field>
        <field name="auto_delete" eval="True"/>
    </record>
</odoo>

```

## File: models\gift_card.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class GiftCard(models.Model):
    _inherit = "gift.card"

    buy_line_id = fields.Many2one("sale.order.line", copy=False, readonly=True,
                                  help="Sale Order line where this gift card has been bought.")
    redeem_line_ids = fields.One2many('sale.order.line', 'gift_card_id', string="Redeems")

    @api.depends("redeem_line_ids")
    def _compute_balance(self):
        super()._compute_balance()
        for record in self:
            confirmed_line = record.redeem_line_ids.filtered(lambda l: l.state in ('sale', 'done'))
            balance = record.balance
            if confirmed_line:
                balance -= sum(confirmed_line.mapped(
                    lambda line: line.currency_id._convert(line.price_unit, record.currency_id, record.env.company, line.create_date) * -1
                ))
            record.balance = balance

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    gift_card_count = fields.Integer(compute="_compute_gift_card_count")

    @api.depends("order_line.generated_gift_card_ids")
    def _compute_gift_card_count(self):
        for record in self:
            record.gift_card_count = len(record.order_line.mapped("generated_gift_card_ids"))

    @api.constrains('state')
    def _constrains_state(self):
        # release gift card amount when order state become canceled
        for record in self.filtered(lambda so: so.state == 'cancel'):
            record.order_line.filtered(lambda ol: ol.gift_card_id).unlink()

        # create and send gift card when order become confirmed
        for record in self.filtered(lambda so: so.state == 'sale'):
            for gift_card_order_line in record.order_line.filtered(lambda ol: ol.product_id.detailed_type == 'gift'):
                gift_card_order_line._create_gift_cards()
            record.sudo()._send_gift_card_mail()

    def _pay_with_gift_card(self, gift_card):
        error = False

        if not gift_card.can_be_used():
            error = _('Invalid or Expired Gift Card.')
        elif gift_card in self.order_line.mapped("gift_card_id"):
            error = _('Gift Card already used.')
        elif gift_card.partner_id and gift_card.partner_id != self.env.user.partner_id:
            error = _('Gift Card are restricted for another user.')

        amount = min(self.amount_total, gift_card.balance_converted(self.currency_id))
        if not error and amount > 0:
            pay_gift_card_id = self.env.ref('gift_card.pay_with_gift_card_product')
            gift_card.redeem_line_ids.filtered(lambda redeem: redeem.state != "sale").unlink()
            self.env["sale.order.line"].create({
                'product_id': pay_gift_card_id.id,
                'price_unit': - amount,
                'product_uom_qty': 1,
                'product_uom': pay_gift_card_id.uom_id.id,
                'gift_card_id': gift_card.id,
                'order_id': self.id
            })
        return error

    def _compute_amount_total_without_delivery(self):
        self.ensure_one()
        # Add back 'payment' rewards from the total without delivery, they should count towards the delivery price goal.
        lines = self.order_line.filtered(lambda l: l.gift_card_id)
        return super()._compute_amount_total_without_delivery() - sum(lines.mapped('price_unit'))

    def _send_gift_card_mail(self):
        template = self.env.ref('sale_gift_card.mail_template_gift_card', raise_if_not_found=False)
        if template and self.gift_card_count:
            for gift in self.order_line.mapped("generated_gift_card_ids"):
                template.send_mail(gift.id, force_send=True, notif_layout='mail.mail_notification_light')

    def _recompute_gift_card_lines(self):
        for record in self:
            lines_to_remove = self.env['sale.order.line']
            lines_to_update = []

            gift_payment_lines = record.order_line.filtered('gift_card_id')
            to_pay = sum((self.order_line - gift_payment_lines).mapped('price_total'))

            # consume older gift card first
            for gift_card_line in gift_payment_lines.sorted(lambda line: line.gift_card_id.expired_date):
                amount = min(to_pay, gift_card_line.gift_card_id.balance_converted(record.currency_id))
                if amount:
                    to_pay -= amount
                    if gift_card_line.price_unit != -amount or gift_card_line.product_uom_qty != 1:
                        lines_to_update.append(
                            fields.Command.update(gift_card_line.id, {'price_unit': -amount, 'product_uom_qty': 1})
                        )
                else:
                    lines_to_remove += gift_card_line
            lines_to_remove.unlink()
            record.update({'order_line': lines_to_update})


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    generated_gift_card_ids = fields.One2many('gift.card', "buy_line_id", string="Bought Gift Card")
    gift_card_id = fields.Many2one('gift.card', help="Deducted from this Gift Card", copy=False)

    def _is_not_sellable_line(self):
        return self.gift_card_id or super()._is_not_sellable_line()

    def _create_gift_cards(self):
        return self.env['gift.card'].create(
            [self._build_gift_card() for _ in range(int(self.product_uom_qty))]
        )

    def _build_gift_card(self):
        return {
            'initial_amount': self.order_id.currency_id._convert(
                self.price_unit,
                self.order_id.env.company.currency_id,
                self.order_id.env.company,
                fields.Date.today()
            ),
            'buy_line_id': self.id,
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gift_card
from . import sale_order

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_gift_card_sales,Gift Card Program Salesman,model_gift_card,sales_team.group_sale_salesman,1,0,0,0
access_gift_card_manager,Gift Card Program Sale Manager,model_gift_card,sales_team.group_sale_manager,1,1,1,1

```

## File: views\sale_order_view.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Actions -->
    <record id="gift_card_sale_order_action" model="ir.actions.act_window">
        <field name="name">Gift Cards</field>
        <field name="res_model">gift.card</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('buy_line_id.order_id', '=', active_id)]</field>
    </record>

    <record id="sale_order_view_extend_gift_card_form" model="ir.ui.view">
        <field name="name">sale.order.view.form.inherit.gift.card</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='preview_sale_order']" position="before">
                <button class="oe_stat_button"
                        name="%(gift_card_sale_order_action)d"
                        attrs="{'invisible': [('gift_card_count', '=', 0)]}"
                        icon="fa-gift"
                        type="action">
                    <field name="gift_card_count" widget="statinfo" string="Gift Cards"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="sale_gift_card_view_form" model="ir.ui.view">
        <field name="name">gift.card.form Website</field>
        <field name="model">gift.card</field>
        <field name="inherit_id" ref="gift_card.gift_card_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='code']" position="after">
                <field name="buy_line_id" options="{'no_create': True}"/>
            </xpath>
            <xpath expr="//group[@name='gift_card']" position="after">
                <group>
                    <field name="redeem_line_ids" options="{'no_create': True}" readonly="1">
                        <tree>
                            <field name="order_id"/>
                        </tree>
                    </field>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" ?>
<odoo>

    <template id="sale_gift_card.used_gift_card">
        <div class="text-muted d-none d-md-block small" t-if="line.gift_card_id">
            <span>
                Code:
                <t t-esc="line.gift_card_id.code[-4:].rjust(14, '&#8902;')"/>
            </span>
            <br/>
            <span>
                Expired Date:
                <t t-esc="line.gift_card_id.expired_date"/>
            </span>
        </div>
    </template>

    <template id="sale_order_portal_content_inherit" name="Gift Card Products Portal" inherit_id="sale.sale_order_portal_content">
        <xpath expr="//section[@id='details']//td[@id='product_name']/*[last()]" position="after">
            <t t-if="line.gift_card_id" t-call="sale_gift_card.used_gift_card"/>
            <t t-if="line.generated_gift_card_ids" t-call="sale_gift_card.sale_purchased_gift_card">
                <t t-set="order" t-value="sale_order"/>
                <t t-set="hide_intro" t-value="1"/>
            </t>
        </xpath>
    </template>

    <template id="sale_purchased_gift_card">
        <div class="card mt-3 " t-if="order.gift_card_count != 0">
            <div class="card-body">
                <span t-if="not hide_intro">You will find below your gift cards code. An email has been sent with it. You can use it starting right now.</span>
                <table class="table text-center table-borderless">
                    <thead>
                        <tr>
                            <th class="font-weight-normal">Gift Card Code</th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="order.order_line" t-as="order_line">
                            <tr t-foreach="order_line.generated_gift_card_ids" t-as="gift_card">
                                <td class="o_purchased_gift_card">
                                    Gift #<t t-esc="gift_card.id"/>
                                    (<span t-field="gift_card.initial_amount" style="white-space: nowrap;" t-options="{'widget': 'monetary', 'display_currency': gift_card.currency_id}"/>)
                                    <strong t-esc="gift_card.code"/>
                                    <button class="btn btn-sm btn-secondary copy-to-clipboard" t-att-data-clipboard-text="gift_card.code">
                                        <span class="fa fa-clipboard"/> Copy
                                    </button>
                                </td>
                            </tr>
                        </t>
                    </tbody>
                </table>
            </div>
        </div>
    </template>
</odoo>

```

