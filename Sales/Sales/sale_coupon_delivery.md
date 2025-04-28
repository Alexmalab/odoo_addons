# Odoo Module: sale_coupon_delivery

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
    'name': "Sale Coupon Delivery",
    'summary': """Allows to offer free shippings in coupon reward""",
    'description': """Integrate coupon mechanism with shipping costs.""",
    'category': 'Sales/Sales',
    'version': '1.0',
    'depends': ['sale_coupon', 'delivery'],
    'data': [
    ],
    'demo': [
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\sale_coupon.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, _


class Coupon(models.Model):
    _inherit = "coupon.coupon"

    def _check_coupon_code(self, order_date, partner_id, **kwargs):
        order = kwargs.get('order', False)
        if order and self.program_id.reward_type == 'free_shipping' and not order.order_line.filtered(lambda line: line.is_delivery):
            return {'error': _('The shipping costs are not in the order lines.')}
        return super(Coupon, self)._check_coupon_code(order_date, partner_id, **kwargs)

```

## File: models\sale_coupon_program.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, _, api


class CouponProgram(models.Model):
    _inherit = "coupon.program"

    def _filter_not_ordered_reward_programs(self, order):
        """
        Returns the programs when the reward is actually in the order lines
        """
        programs = super(CouponProgram, self)._filter_not_ordered_reward_programs(order)
        # Do not filter on free delivery programs. As delivery_unset is called everywhere (which is
        # rather stupid), the delivery line is unliked to be created again instead of writing on it to
        # modify the price_unit. That way, the reward is unlink and is not set back again.
        return programs

    def _check_promo_code(self, order, coupon_code):
        if self.reward_type == 'free_shipping' and not any(line.is_delivery for line in order.order_line):
            return {'error': _('The shipping costs are not in the order lines.')}
        return super(CouponProgram, self)._check_promo_code(order, coupon_code)

    def _get_lines_suitable_for_program(self, order):
        lines = super()._get_lines_suitable_for_program(order)
        return lines.filtered(lambda line: not line.is_delivery)

```

## File: models\sale_coupon_reward.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class CouponReward(models.Model):
    _inherit = 'coupon.reward'
    _description = "Coupon Reward"

    reward_type = fields.Selection(selection_add=[('free_shipping', 'Free Shipping')])

    def name_get(self):
        result = []
        reward_names = super(CouponReward, self).name_get()
        free_shipping_reward_ids = self.filtered(lambda reward: reward.reward_type == 'free_shipping').ids
        for res in reward_names:
            result.append((res[0], res[0] in free_shipping_reward_ids and _("Free Shipping") or res[1]))
        return result

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _

class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _get_no_effect_on_threshold_lines(self):
        self.ensure_one()
        # Do not count shipping and free shipping
        free_delivery_product = self.env['coupon.program'].search([('reward_type', '=', 'free_shipping')]).mapped('discount_line_product_id')
        lines = self.order_line.filtered(lambda line: line.is_delivery or line.product_id in free_delivery_product)
        return lines + super(SaleOrder, self)._get_no_effect_on_threshold_lines()

    def _get_paid_order_lines(self):
        """ Returns the taxes included sale order total amount without the rewards amount"""
        free_reward_product = self.env['coupon.program'].search([('reward_type', '=', 'product')]).mapped('discount_line_product_id')
        return self.order_line.filtered(lambda x: not x._is_not_sellable_line() or x.product_id in free_reward_product)

    def _get_reward_line_values(self, program):
        if program.reward_type == 'free_shipping':
            return [self._get_reward_values_free_shipping(program)]
        else:
            return super(SaleOrder, self)._get_reward_line_values(program)

    def _get_reward_values_free_shipping(self, program):
        delivery_line = self.order_line.filtered(lambda x: x.is_delivery)
        taxes = delivery_line.product_id.taxes_id.filtered(lambda t: t.company_id.id == self.company_id.id)
        taxes = self.fiscal_position_id.map_tax(taxes)
        return {
            'name': _("Discount: %s", program.name),
            'product_id': program.discount_line_product_id.id,
            'price_unit': delivery_line and - delivery_line.price_unit or 0.0,
            'product_uom_qty': 1.0,
            'product_uom': program.discount_line_product_id.uom_id.id,
            'order_id': self.id,
            'is_reward_line': True,
            'tax_id': [(4, tax.id, False) for tax in taxes],
        }

    def _get_cheapest_line(self):
        # Unit prices tax included
        return min(self.order_line.filtered(lambda x: not x._is_not_sellable_line() and x.price_reduce > 0), key=lambda x: x['price_reduce'])

class SalesOrderLine(models.Model):
    _inherit = "sale.order.line"

    def unlink(self):
        # Due to delivery_set and delivery_unset methods that are called everywhere, don't unlink
        # reward lines if it's a free shipping
        self = self.exists()
        orders = self.mapped('order_id')
        applied_programs = orders.mapped('no_code_promo_program_ids') + \
                           orders.mapped('code_promo_program_id') + \
                           orders.mapped('applied_coupon_ids').mapped('program_id')
        free_shipping_products = applied_programs.filtered(
            lambda program: program.reward_type == 'free_shipping'
        ).mapped('discount_line_product_id')
        lines_to_unlink = self.filtered(lambda line: line.product_id not in free_shipping_products)
        # Unless these lines are the last ones
        res = super(SalesOrderLine, lines_to_unlink).unlink()
        only_free_shipping_line_orders = orders.filtered(lambda order: len(order.order_line.ids) == 1 and order.order_line.is_reward_line)
        super(SalesOrderLine, only_free_shipping_line_orders.mapped('order_line')).unlink()
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order
from . import sale_coupon_program
from . import sale_coupon_reward
```

