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
        'views/event_booth_templates.xml',
    ],
    'demo': [],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            '/website_event_booth_sale/static/src/js/booth_register.js',
        ],
        'web.assets_tests': [
            '/website_event_booth_sale/static/tests/tours/**/**.js'
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\event_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug

from odoo import http
from odoo.http import request
from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEventBoothController(WebsiteEventController):

    @http.route()
    def event_booth_main(self, event):
        pricelist = request.website.get_current_pricelist()
        if pricelist:
            event = event.with_context(pricelist=pricelist.id)
        return super(WebsiteEventBoothController, self).event_booth_main(event)

    @http.route()
    def event_booth_registration_confirm(self, event, booth_category_id, event_booth_ids, **kwargs):
        booths = self._get_requested_booths(event, event_booth_ids)
        booth_category = request.env['event.booth.category'].sudo().browse(int(booth_category_id))
        order = request.website.sale_get_order(force_create=1)
        order._cart_update(
            product_id=booth_category.product_id.id,
            set_qty=1,
            event_booth_pending_ids=booths.ids,
            registration_values=self._prepare_booth_registration_values(event, kwargs),
        )
        if order.amount_total:
            return request.redirect('/shop/checkout')
        elif order:
            order.action_confirm()
            request.website.sale_reset()

        return request.redirect(('/event/%s/booth/success?' % event.id) + werkzeug.urls.url_encode({
            'booths': ','.join([str(id) for id in booths.ids]),
        }))

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

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import Command
from odoo import fields, models, _


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _cart_find_product_line(self, product_id=None, line_id=None, **kwargs):
        """Check if there is another sale order line which already contains the requested event_booth_pending_ids
        to overwrite it with the newly requested booths to avoid having multiple so_line related to the same booths"""
        self.ensure_one()
        lines = super(SaleOrder, self)._cart_find_product_line(product_id, line_id, **kwargs)
        if line_id:
            return lines
        event_booth_pending_ids = kwargs.get('event_booth_pending_ids')
        if event_booth_pending_ids:
            lines = lines.filtered(
                lambda line: any(booth.id in event_booth_pending_ids for booth in line.event_booth_pending_ids)
            )
        return lines

    def _website_product_id_change(self, order_id, product_id, qty=0, **kwargs):
        values = super(SaleOrder, self)._website_product_id_change(order_id, product_id, qty=qty, **kwargs)
        event_booth_pending_ids = kwargs.get('event_booth_pending_ids')
        if event_booth_pending_ids:
            order_line = self.env['sale.order.line'].sudo().search([
                ('id', 'in', self.order_line.ids),
                ('event_booth_pending_ids', 'in', event_booth_pending_ids)])
            booths = self.env['event.booth'].browse(event_booth_pending_ids).with_context(pricelist=self.pricelist_id.id)
            if order_line.event_booth_pending_ids.ids != event_booth_pending_ids:
                new_registrations_commands = [Command.create({
                                            'event_booth_id': booth.id,
                                            **kwargs.get('registration_values'),
                                        }) for booth in booths]
                if order_line:
                    event_booth_registrations_command = [Command.delete(reg.id) for reg in
                                                         order_line.event_booth_registration_ids] + new_registrations_commands
                else:
                    event_booth_registrations_command = new_registrations_commands
                values['event_booth_registration_ids'] = event_booth_registrations_command

            discount = 0
            order = self.env['sale.order'].sudo().browse(order_id)
            booth_currency = booths.product_id.currency_id
            pricelist_currency = order.pricelist_id.currency_id
            price_reduce = sum(booth.booth_category_id.price_reduce for booth in booths)
            if booth_currency != pricelist_currency:
                price_reduce = booth_currency._convert(
                    price_reduce,
                    pricelist_currency,
                    order.company_id,
                    order.date_order or fields.Datetime.now()
                )
            if order.pricelist_id.discount_policy == 'without_discount':
                price = sum(booth.booth_category_id.price for booth in booths)
                if price != 0:
                    if booth_currency != pricelist_currency:
                        price = booth_currency._convert(
                            price,
                            pricelist_currency,
                            order.company_id,
                            order.date_order or fields.Datetime.now()
                        )
                    discount = (price - price_reduce) / price * 100
                    price_unit = price
                    if discount < 0:
                        discount = 0
                        price_unit = price_reduce
                else:
                    price_unit = price_reduce

            else:
                price_unit = price_reduce

            if order.pricelist_id and order.partner_id:
                order_line = order._cart_find_product_line(booths.product_id.id)
                if order_line:
                    price_unit = self.env['account.tax']._fix_tax_included_price_company(price_unit, booths.product_id.taxes_id, order_line[0].tax_id, self.company_id)

            values.update(
                event_id=booths.event_id.id,
                discount=discount,
                price_unit=price_unit,
                name=booths._get_booth_multiline_description(),
            )

        return values

    def _cart_update(self, product_id=None, line_id=None, add_qty=0, set_qty=0, **kwargs):
        values = {}
        product = self.env['product.product'].browse(product_id)
        if product.detailed_type == 'event_booth' and line_id:
            if set_qty > 1:
                set_qty = 1
                values['warning'] = _('You cannot manually change the quantity of an Event Booth product.')
            if add_qty == 0 and not kwargs.get('event_booth_pending_ids'):
                # when updating the pricelist, the website_sale module call this method without the 'event.booth' ids
                # -> we manually set the argument to make sure the price is updated in '_website_product_id_change'
                kwargs['event_booth_pending_ids'] = self.env['sale.order.line'].browse(line_id).event_booth_pending_ids.ids
        values.update(super(SaleOrder, self)._cart_update(product_id, line_id, add_qty, set_qty, **kwargs))
        return values

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
from . import sale_order
from . import website

```

## File: static\src\js\booth_register.js

```javascript
odoo.define('website_event_booth_sale.booth_registration', function (require) {
'use strict';

const BoothRegistration = require('website_event_booth.booth_registration');

/**
 * This class changes the displayed price after selecting the requested booths.
 */
BoothRegistration.include({

    //--------------------------------------------------------------------------
    // Overrides
    //--------------------------------------------------------------------------

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

});

```

## File: views\event_booth_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <template id="event_booth_registration" inherit_id="website_event_booth.event_booth_registration">
        <xpath expr="//h5[@name='booth_category_name']" position="after">
            <t t-if="booth_category.price">
                <t t-if="(booth_category.price - booth_category.price_reduce) &gt; 1 and website.get_current_pricelist().discount_policy == 'without_discount'">
                    <del class="text-danger mr-1"
                         t-field="booth_category.price"
                         t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.get_current_pricelist().currency_id}"/>
                </t>
                <span t-field="booth_category.price_reduce" class="font-weight-normal text-muted"
                      t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"
                      groups="account.group_show_line_subtotals_tax_excluded"/>
                <span t-field="booth_category.price_reduce_taxinc" class="font-weight-normal text-muted"
                      t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"
                      groups="account.group_show_line_subtotals_tax_included"/>
            </t>
            <span t-else="" class="font-weight-normal text-muted">Free</span>
        </xpath>
        <xpath expr="//input[@name='booth_category_id']" position="attributes">
            <attribute name="t-att-data-price">
                event.company_id.sudo().currency_id._convert(
                    booth_category.price_reduce_taxinc if env.user.has_group('account.group_show_line_subtotals_tax_included') else booth_category.price_reduce,
                    website.get_current_pricelist().currency_id,
                    event.company_id,
                    datetime.date.today()
                ) or '0'
            </attribute>
        </xpath>
        <xpath expr="//div[@name='booth_registration_submit']" position="before">
            <div class="row o_wbooth_booth_total_price d-none">
                <div class="col-sm-2 offset-sm-1">
                    <span class="font-weight-bold">Total</span>
                </div>
                <div class="col-sm-6">
                    <span class="font-weight-bold" t-esc="float(0)"
                          t-options="{'widget': 'monetary', 'display_currency': website.pricelist_id.currency_id}"/>
                </div>
            </div>
        </xpath>
    </template>

</data></odoo>

```

