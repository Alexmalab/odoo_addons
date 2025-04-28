# Odoo Module: loyalty_delivery

Category: Sales

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
    'name': 'Coupons & Loyalty - Delivery',
    'summary': "Add a free shipping option to your rewards",
    'category': 'Sales',
    'version': '1.0',
    'depends': ['loyalty', 'delivery'],
    'data': [
        'data/loyalty_delivery_data.xml',
        'views/loyalty_reward_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\loyalty_delivery_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="loyalty.gift_card_product_50" model="product.product">
        <field name="tracking">none</field>
    </record>
    <record id="loyalty.ewallet_product_50" model="product.product">
        <field name="tracking">none</field>
    </record>
</odoo>

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

from odoo import models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _compute_amount_total_without_delivery(self):
        self.ensure_one()
        lines = self.order_line.filtered(lambda l: l.coupon_id and l.coupon_id.program_type in ['ewallet', 'gift_card'])
        return super()._compute_amount_total_without_delivery() - sum(lines.mapped('price_unit'))

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
            <xpath expr="//group[@id='reward_type_group']" position="after">
                <group id="shipping" string="Free shipping" attrs="{'invisible': [('reward_type', '!=', 'shipping')]}">
                    <field name="discount_max_amount"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="loyalty_reward_view_kanban_inherit_loyalty_delivery" model="ir.ui.view">
        <field name="name">loyalty.reward.view.kanban.inherit.loyalty.delivery</field>
        <field name="model">loyalty.reward</field>
        <field name="inherit_id" ref="loyalty.loyalty_reward_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='reward_info']" position="inside">
                <t t-if="record.reward_type.raw_value === 'shipping'">

                    <a>Free shipping <t t-if="record.discount_max_amount.raw_value > 0">( Max <field name="discount_max_amount"/> )</t></a>
                    <br/><br/>
                </t>                    
            </xpath>
        </field>
    </record>
</odoo>

```

