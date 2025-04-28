# Odoo Module: website_sale_gelato

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "eCommerce/Gelato bridge",
    'category': 'Website/Website',
    'depends': ['sale_gelato', 'website_sale'],
    'data': [
        'data/delivery_carrier_data.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\delivery_carrier_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Prevent recreating delivery if it was deleted before installing website_sale_gelato -->
    <record id="sale_gelato.standard_delivery" model="delivery.carrier" forcecreate="0">
        <field name="is_published">True</field>
    </record>

    <!-- Prevent recreating delivery if it was deleted before installing website_sale_gelato -->
    <record id="sale_gelato.express_delivery" model="delivery.carrier" forcecreate="0">
        <field name="is_published">True</field>
    </record>

</odoo>

```

## File: models\product_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.exceptions import ValidationError


class ProductDocument(models.Model):
    _inherit = 'product.document'

    # === CONSTRAINT METHODS === #

    @api.constrains('datas')
    def _check_product_is_unpublished_before_removing_print_images(self):
        for print_image in self.filtered(lambda i: i.is_gelato):
            template = self.env['product.template'].browse(print_image.res_id)
            if template.is_published and not print_image.datas:
                raise ValidationError(
                    _("Products must be unpublished before print images can be removed.")
                )

```

## File: models\product_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.exceptions import ValidationError


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    # === CONSTRAINT METHODS === #

    @api.constrains('is_published')
    def _check_print_images_are_set_before_publishing(self):
        for product in self.filtered('gelato_template_ref'):
            if product.is_published and product.gelato_missing_images:
                raise ValidationError(
                    _("Print images must be set on products before they can be published.")
                )

    # === ACTION METHODS === #

    def action_create_product_variants_from_gelato_template(self):
        """ Override of `sale_gelato` to unpublish products for which the synchronization with
        Gelato led to new print images being created. """
        image_count_before_sync = len(self.gelato_image_ids)
        res = super().action_create_product_variants_from_gelato_template()
        if image_count_before_sync < len(self.gelato_image_ids):
            self.is_published = False
        return res

    # === BUSINESS METHODS === #

    def _create_attributes_from_gelato_info(self, template_info):
        """ Override of `sale_gelato` to set the eCommerce description. """
        self.description_ecommerce = template_info['description']
        return super()._create_attributes_from_gelato_info(template_info)

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _verify_updated_quantity(self, order_line, product_id, new_qty, **kwargs):
        """ Override of `website_sale` to prevent mixing Gelato and non-Gelato products in the cart.

        This check is not redundant with the constraint on `sale.order` in `sale_gelato` because the
        constraint would only be enforced at the end of the checkout for eCommerce carts, and would
        not mention the specific product that caused the issue nor display the warning message in a
        user-friendly way.

        :param sale.order.line order_line: The order line to update.
        :param int product_id: The ID of the product to update.
        :param int new_qty: The new quantity of the product.
        :param kwargs: Additional keyword arguments.
        :return: The new quantity and an optional warning message.
        :rtype: tuple[int, str]
        """
        product = self.env['product.product'].browse(product_id)
        mixing_products = product.type != 'service' and any(
            (product.gelato_product_uid and not line.product_id.gelato_product_uid)
            or (not product.gelato_product_uid and line.product_id.gelato_product_uid)
            for line in self.order_line.filtered(lambda l: l.product_id.type != 'service')
        )  # Whether Gelato and non-Gelato products that require delivery are mixed.
        if mixing_products:
            return 0, _(
                "The product %(product_name)s cannot be added to the cart as it requires separate"
                " shipping. Please place your order for the current cart first.",
                product_name=product.name,
            )
        return super()._verify_updated_quantity(order_line, product_id, new_qty, **kwargs)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_document
from . import product_template
from . import sale_order

```

