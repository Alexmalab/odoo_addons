# Odoo Module: sale_loyalty_delivery

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
    'name': 'Sale Loyalty - Delivery',
    'summary': 'Adds free shipping mechanism in sales orders',
    'description': 'Integrate free shipping in sales orders.',
    'category': 'Sales/Sales',
    'data': [
        'views/loyalty_reward_views.xml',
    ],
    'depends': ['sale_loyalty', 'delivery'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\loyalty_program.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models


class LoyaltyProgram(models.Model):
    _inherit = 'loyalty.program'

    @api.model
    def _program_type_default_values(self):
        res = super()._program_type_default_values()
        # Add a loyalty reward for free shipping
        if 'loyalty' in res:
            res['loyalty']['reward_ids'].append((0, 0, {
                'reward_type': 'shipping',
                'required_points': 100,
            }))
        return res

    @api.model
    def get_program_templates(self):
        # Override 'promotion' template to say free shipping
        res = super().get_program_templates()
        if 'promotion' in res:
            res['promotion']['description'] = _("Automatic promotion: free shipping on orders higher than $50")
        return res

    @api.model
    def _get_template_values(self):
        res = super()._get_template_values()
        if 'promotion' in res:
            res['promotion']['reward_ids'] = [(5, 0, 0), (0, 0, {
                'reward_type': 'shipping',
            })]
        return res

```

## File: models\loyalty_reward.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models


class LoyaltyReward(models.Model):
    _inherit = 'loyalty.reward'

    reward_type = fields.Selection(
        selection_add=[('shipping', 'Free Shipping')],
        ondelete={'shipping': 'set default'})

    def _compute_description(self):
        shipping_rewards = self.filtered(lambda r: r.reward_type == 'shipping')
        super(LoyaltyReward, self - shipping_rewards)._compute_description()
        shipping_rewards.description = _('Free shipping')
        for reward in shipping_rewards:
            if reward.discount_max_amount:
                format_string = '%(amount)g %(symbol)s'
                if reward.currency_id.position == 'before':
                    format_string = '%(symbol)s %(amount)g'
                formatted_amount = format_string % {'amount': reward.discount_max_amount, 'symbol': reward.currency_id.symbol}
                reward.description += _(' (Max %s)', formatted_amount)

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.fields import Command


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    # delivery overrides

    def _compute_amount_total_without_delivery(self):
        res = super()._compute_amount_total_without_delivery()
        return res - sum(
            self.order_line.filtered(
                lambda l: l.coupon_id and l.coupon_id.program_type in ['ewallet', 'gift_card']
            ).mapped('price_unit')
        )

    # sale_loyalty overrides

    def _get_no_effect_on_threshold_lines(self):
        res = super()._get_no_effect_on_threshold_lines()
        return res + self.order_line.filtered(
            lambda line: line.is_delivery or line.reward_id.reward_type == 'shipping')

    def _get_not_rewarded_order_lines(self):
        """Exclude delivery lines from consideration for reward points."""
        order_line = super()._get_not_rewarded_order_lines()
        return order_line.filtered(lambda line: not line.is_delivery)

    def _get_reward_values_free_shipping(self, reward, coupon, **kwargs):
        delivery_line = self.order_line.filtered(lambda l: l.is_delivery)[:1]
        taxes = delivery_line.product_id.taxes_id._filter_taxes_by_company(self.company_id)
        taxes = self.fiscal_position_id.map_tax(taxes)
        max_discount = reward.discount_max_amount or float('inf')
        return [{
            'name': _('Free Shipping - %s', reward.description),
            'reward_id': reward.id,
            'coupon_id': coupon.id,
            'points_cost': reward.required_points if not reward.clear_wallet else self._get_real_points_for_coupon(coupon),
            'product_id': reward.discount_line_product_id.id,
            'price_unit': -min(max_discount, delivery_line.price_unit or 0),
            'product_uom_qty': 1,
            'product_uom': reward.discount_line_product_id.uom_id.id,
            'order_id': self.id,
            'is_reward_line': True,
            'sequence': max(self.order_line.filtered(lambda x: not x.is_reward_line).mapped('sequence'), default=0) + 1,
            'tax_id': [(Command.CLEAR, 0, 0)] + [(Command.LINK, tax.id, False) for tax in taxes],
        }]

    def _get_reward_line_values(self, reward, coupon, **kwargs):
        self.ensure_one()
        if reward.reward_type == 'shipping':
            self = self.with_context(lang=self._get_lang())
            reward = reward.with_context(lang=self._get_lang())
            return self._get_reward_values_free_shipping(reward, coupon, **kwargs)
        return super()._get_reward_line_values(reward, coupon, **kwargs)

    def _get_claimable_rewards(self, forced_coupons=None):
        res = super()._get_claimable_rewards(forced_coupons)
        if any(reward.reward_type == 'shipping' for reward in self.order_line.reward_id):
            # Allow only one reward of type shipping at the same time
            filtered_res = {}
            for coupon, rewards in res.items():
                filtered_rewards = rewards.filtered(lambda r: r.reward_type != 'shipping')
                if filtered_rewards:
                    filtered_res[coupon] = filtered_rewards
            res = filtered_res
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import loyalty_program
from . import loyalty_reward
from . import sale_order

```

## File: views\loyalty_reward_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="loyalty_reward_view_form_inherit_loyalty_delivery" model="ir.ui.view">
        <field name="name">loyalty.reward.view.form.inherit.loyalty.delivery</field>
        <field name="model">loyalty.reward</field>
        <field name="inherit_id" ref="loyalty.loyalty_reward_view_form"/>
        <field name="arch" type="xml">
            <group name="reward_type_group" position="after">
                <group id="shipping" string="Free shipping" invisible="reward_type != 'shipping'">
                    <field name="discount_max_amount"/>
                </group>
            </group>
        </field>
    </record>

    <record id="loyalty_reward_view_kanban_inherit_loyalty_delivery" model="ir.ui.view">
        <field name="name">loyalty.reward.view.kanban.inherit.loyalty.delivery</field>
        <field name="model">loyalty.reward</field>
        <field name="inherit_id" ref="loyalty.loyalty_reward_view_kanban"/>
        <field name="arch" type="xml">
            <div name="reward_info" position="inside">
                <t t-elif="record.reward_type.raw_value === 'shipping'">
                    Free shipping <t t-if="record.discount_max_amount.raw_value > 0">( Max <field name="discount_max_amount"/> )</t>
                </t>
            </div>
        </field>
    </record>

</odoo>

```

