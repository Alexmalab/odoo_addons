# Odoo Module: website_event_booth_sale

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Online Event Booth Sale',
    'category': 'Marketing/Events',
    'version': '1.0',
    'summary': 'Events, sell your booths online',
    'description': """
Use the e-commerce to sell your event booths.
    """,
    'depends': ['event_booth_sale', 'website_event_booth', 'website_sale'],
    'data': [
        'views/event_booth_registration_templates.xml',
        'views/event_booth_templates.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            '/website_event_booth_sale/static/src/js/booth_register.js',
        ],
        'web.assets_tests': [
            '/website_event_booth_sale/static/tests/tours/**/*.js'
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\event_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo.http import request, route
from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEventBoothController(WebsiteEventController):

    @route()
    def event_booth_registration_confirm(self, event, booth_category_id, event_booth_ids, **kwargs):
        """Override: Doesn't call the parent method because we go through the checkout
        process which will confirm the booths when receiving the payment."""
        booths = self._get_requested_booths(event, event_booth_ids)
        booth_category = request.env['event.booth.category'].sudo().browse(int(booth_category_id))
        error_code = self._check_booth_registration_values(
            booths,
            kwargs['contact_email'],
            booth_category=booth_category)
        if error_code:
            return json.dumps({'error': error_code})

        booth_values = self._prepare_booth_registration_values(event, kwargs)
        order_sudo = request.website.sale_get_order(force_create=True)
        order_sudo._cart_update(
            product_id=booth_category.product_id.id,
            set_qty=1,
            event_booth_pending_ids=booths.ids,
            registration_values=booth_values,
        )
        if order_sudo.amount_total:
            if request.env.user._is_public():
                order_sudo.partner_id = booth_values['partner_id']
            return json.dumps({'redirect': '/shop/cart'})
        elif order_sudo:
            order_sudo.action_confirm()
            request.website.sale_reset()

            return self._prepare_booth_registration_success_values(event.name, booth_values)

    def _prepare_booth_contact_form_values(self, event, booth_ids, booth_category_id):
        values = super()._prepare_booth_contact_form_values(event, booth_ids, booth_category_id)
        values['has_payment_step'] = request.website.sale_get_order().amount_total or \
            values.get('booth_category', request.env['event.booth.category']).price
        return values

    def _prepare_booth_main_values(self, event, booth_category_id=False, booth_ids=False):
        values = super()._prepare_booth_main_values(event, booth_category_id=booth_category_id, booth_ids=booth_ids)
        values['has_payment_step'] = request.website.sale_get_order().amount_total or \
            any(booth_category.price for booth_category in values.get('available_booth_category_ids', request.env['event.booth.category']))
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_booth

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _is_add_to_cart_allowed(self):
        # `event_booth_registration_confirm` calls `_cart_update` with specific products, allow those aswell.
        return super()._is_add_to_cart_allowed() or\
                self.env['event.booth.category'].sudo().search_count([('product_id', '=', self.id)])

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    @api.model
    def _get_product_types_allow_zero_price(self):
        return super()._get_product_types_allow_zero_price() + ["event_booth"]

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import Command
from odoo import models, _


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _cart_find_product_line(
        self, product_id=None, line_id=None,
        event_booth_pending_ids=None, **kwargs
    ):
        """Check if there is another sale order line which already contains the requested event_booth_pending_ids
        to overwrite it with the newly requested booths to avoid having multiple so_line related to the same booths"""
        lines = super()._cart_find_product_line(product_id, line_id, **kwargs)

        if not event_booth_pending_ids or line_id:
            return lines

        return lines.filtered(
            lambda line: any(booth.id in event_booth_pending_ids for booth in line.event_booth_pending_ids)
        )

    def _verify_updated_quantity(self, order_line, product_id, new_qty, **kwargs):
        """Forbid quantity updates on event booth lines."""
        product = self.env['product.product'].browse(product_id)
        if product.detailed_type == 'event_booth' and new_qty > 1:
            return 1, _('You cannot manually change the quantity of an Event Booth product.')
        return super()._verify_updated_quantity(order_line, product_id, new_qty, **kwargs)

    def _prepare_order_line_values(
        self, product_id, quantity, event_booth_pending_ids=False, registration_values=None,
        **kwargs
    ):
        """Add corresponding event to the SOline creation values (if booths are provided)."""
        values = super()._prepare_order_line_values(product_id, quantity, **kwargs)

        if not event_booth_pending_ids:
            return values

        booths = self.env['event.booth'].browse(event_booth_pending_ids)

        values['event_id'] = booths.event_id.id
        values['event_booth_registration_ids'] = [
            Command.create({
                'event_booth_id': booth.id,
                **registration_values,
            }) for booth in booths
        ]

        return values

    # FIXME VFE investigate if it ever happens.
    # Probably not
    def _prepare_order_line_update_values(
        self, order_line, quantity, event_booth_pending_ids=False, registration_values=None,
        **kwargs
    ):
        """Delete existing booth registrations and create new ones with the update values."""
        values = super()._prepare_order_line_update_values(order_line, quantity, **kwargs)

        if not event_booth_pending_ids:
            return values

        booths = self.env['event.booth'].browse(event_booth_pending_ids)
        values['event_booth_registration_ids'] = [
            Command.delete(registration.id)
            for registration in order_line.event_booth_registration_ids
        ] + [
            Command.create({
                'event_booth_id': booth.id,
                **registration_values,
            }) for booth in booths
        ]

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Website(models.Model):
    _inherit = 'website'

    def sale_product_domain(self):
        return ['&'] + super(Website, self).sale_product_domain() + [('detailed_type', '!=', 'event_booth')]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_product
from . import product_template
from . import sale_order
from . import website

```

## File: static\src\js\booth_register.js

```javascript
/** @odoo-module **/

import BoothRegistration from "@website_event_booth/js/booth_register";

/**
 * This class changes the displayed price after selecting the requested booths.
 */
BoothRegistration.include({

    //--------------------------------------------------------------------------
    // Overrides
    //--------------------------------------------------------------------------

    start() {
        return this._super.apply(this, arguments).then(() => {
            this.categoryPrice = this.selectedBoothCategory ? this.selectedBoothCategory.dataset.price : undefined;
        });
    },

    _onChangeBoothType(ev) {
        this.categoryPrice = parseFloat($(ev.currentTarget).data('price'));
        return this._super.apply(this, arguments);
    },

    /**
     * Updates the displayed total price after selecting the requested booths
     * @param boothCount
     * @private
     */
    _updateUiAfterBoothChange(boothCount) {
        this._super.apply(this, arguments);
        let $elem = this.$('.o_wbooth_booth_total_price');
        $elem.toggleClass('d-none', !boothCount || !this.categoryPrice);
        this._updatePrice(boothCount);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _updatePrice(boothsCount) {
        let $elem = this.$('.o_wbooth_booth_total_price .oe_currency_value');
        $elem.text(boothsCount * this.categoryPrice);
    },

});

```

## File: views\event_booth_registration_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <template id="event_booth_registration_details" inherit_id="website_event_booth.event_booth_registration_details">
        <xpath expr="//button[hasclass('o_wbooth_registration_confirm')]//span" position="replace">
            <t t-if="has_payment_step">Go to Payment</t>
            <t t-else="">Book my Booth(s)</t>
        </xpath>
    </template>

</data></odoo>

```

## File: views\event_booth_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>

    <template id="event_booth_registration" inherit_id="website_event_booth.event_booth_registration">
        <xpath expr="//label//span[hasclass('booth_category_price')]" position="replace">
            <t t-if="booth_category.price">
                <t t-if="(booth_category.price - booth_category.price_reduce) &gt; 0 and website.pricelist_id.discount_policy == 'without_discount'">
                    <del t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'"
                         class="text-danger me-1"
                         t-field="booth_category.price"
                         t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                    <del t-else=""
                         class="text-danger me-1"
                         t-field="booth_category.price_incl"
                         t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                </t>
                <h5 t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'"
                      t-field="booth_category.price_reduce"
                      t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                <h5 t-else=""
                      t-field="booth_category.price_reduce_taxinc"
                      t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
            </t>
            <h5 t-else="">Free</h5>
        </xpath>
        <xpath expr="//input[@name='booth_category_id']" position="attributes">
            <attribute name="t-att-data-price">
                event.company_id.sudo().currency_id._convert(
                    booth_category.price_reduce_taxinc if website.show_line_subtotals_tax_selection == 'tax_included' else booth_category.price_reduce,
                    website.currency_id,
                    event.company_id,
                    datetime.date.today()
                ) or '0'
            </attribute>
        </xpath>
        <xpath expr="//button[hasclass('o_wbooth_registration_submit')]" position="before">
            <div class="o_wbooth_booth_total_price d-none text-end">
                <div class="fw-bold">
                    <span>Total</span>
                    <span t-out="float(0)"
                          t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                </div>
            </div>
        </xpath>
    </template>

    <template id="event_booth_order_progress" inherit_id="website_event_booth.event_booth_order_progress">
        <xpath expr="//li[last()]//span" position="replace">
            <t t-if="has_payment_step">Payment</t>
            <t t-else="">Confirmed</t>
        </xpath>
    </template>

</odoo>

```

