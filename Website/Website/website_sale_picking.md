# Odoo Module: website_sale_picking

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'On site Payment & Picking',
    'version': '1.0',
    'category': 'Website/Website',
    'description': """
Allows customers to pay for their orders at a shop, instead of paying online.
""",
    'depends': ['website_sale_delivery', 'payment_custom'],
    'data': [
        'data/website_sale_picking_data.xml',
        'views/res_config_settings_views.xml',
        'views/templates.xml',
        'views/delivery_view.xml'
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_picking/static/src/js/checkout_form.js'
        ],
        'web.assets_tests': [
            'website_sale_picking/static/tests/tours/**/*.js'
        ]
    },
    'application': False,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.addons.website_sale.controllers.main import PaymentPortal
from odoo.exceptions import ValidationError
from odoo.http import request


class PaymentPortalOnsite(PaymentPortal):

    def _validate_transaction_for_order(self, transaction, sale_order_id):
        """
        Throws a ValidationError if the user tries to pay on site without also using an onsite delivery carrier
        Also sets the sale order's warehouse id to the carrier's if it exists
        """
        super()._validate_transaction_for_order(transaction, sale_order_id)
        sale_order = request.env['sale.order'].browse(sale_order_id).exists().sudo()

        # This should never be triggered unless the user intentionally forges a request.
        if sale_order.carrier_id.delivery_type != 'onsite' and (
            transaction.provider_id.code == 'custom'
            and transaction.provider_id.custom_mode == 'onsite'
        ):
            raise ValidationError(_("You cannot pay onsite if the delivery is not onsite"))

        if sale_order.carrier_id.delivery_type == 'onsite' and sale_order.carrier_id.warehouse_id:
            sale_order.warehouse_id = sale_order.carrier_id.warehouse_id

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\website_sale_picking_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_provider_onsite" model="payment.provider">
        <field name="name">Pay in store when picking the product</field>
        <field name="module_id" ref="base.module_website_sale_picking"/>
        <field name="code">custom</field>
        <field name="state">enabled</field>
        <field name="custom_mode">onsite</field>
        <field name="redirect_form_view_id" ref="payment_custom.redirect_form"/>
        <field name="pending_msg" type="html">
            <p>
                <i>Your order has been saved.</i> Please come to the store to pay for your products
            </p>
        </field>
    </record>

    <record id="onsite_delivery_product" model="product.product">
        <field name="name">On site picking</field>
        <field name="description">Pay in store when picking the product</field>
        <field name="type">service</field>
        <field name="list_price">0</field>
        <field name="purchase_ok">false</field>
        <field name="sale_ok">false</field>
    </record>

    <record model="delivery.carrier" id="website_sale_picking.default_onsite_carrier">
        <field name="name">[On Site Pick] My Shop 1</field>
        <field name="delivery_type">onsite</field>
        <field name="website_published">true</field>
        <field name="product_id" ref="website_sale_picking.onsite_delivery_product"/>
        <field name="website_id" ref="website.default_website"/>
    </record>

</odoo>

```

## File: models\delivery_carrier.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, _, api
from odoo.exceptions import ValidationError


class DeliveryCarrier(models.Model):
    _inherit = 'delivery.carrier'

    # Onsite delivery means the client comes to a physical store to get the products himself.
    delivery_type = fields.Selection(selection_add=[
        ('onsite', 'Pickup in store')
    ], ondelete={'onsite': 'set default'})

    # If set, the sales order shipping address will take this warehouse's address.
    warehouse_id = fields.Many2one('stock.warehouse', 'Warehouse')

    @api.constrains('warehouse_id', 'company_id')
    def _check_warehouse_company(self):
        for carrier in self:
            if carrier.warehouse_id.company_id and carrier.company_id and carrier.company_id != carrier.warehouse_id.company_id:
                raise ValidationError(_("The picking site and warehouse must share the same company"))

    def onsite_rate_shipment(self, order):
        """
        Required to show the price on the checkout page for the onsite delivery type
        """
        return {
            'success': True,
            'price': self.product_id.list_price,
            'error_message': False,
            'warning_message': False
        }

    def onsite_send_shipping(self, pickings):
        return [{
            'exact_price': p.carrier_id.fixed_price,
            'tracking_number': False
        } for p in pickings]

    def onsite_cancel_shipment(self, pickings):
        pass  # No need to communicate to an external service, however the method must exist so that cancel_shipment() works.

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    custom_mode = fields.Selection(
        selection_add=[('onsite', "On Site")]
    )

    @api.model
    def _get_compatible_providers(self, *args, sale_order_id=None, website_id=None, **kwargs):
        """ Override of payment to exclude onsite providers if the delivery doesn't match.

        :param int sale_order_id: The sale order to be paid, if any, as a `sale.order` id
        :param int website_id: The provided website, as a `website` id
        :return: The compatible providers
        :rtype: recordset of `payment.provider`
        """
        compatible_providers = super()._get_compatible_providers(
            *args, sale_order_id=sale_order_id, website_id=website_id, **kwargs)
        # Show on site picking only if delivery carriers onsite exists
        onsite_carriers = self.env['delivery.carrier'].search([
            ('website_published', '=', True),
            ('delivery_type', '=', 'onsite'),
            '|',
                ('website_id', '=?', website_id),
                ('website_id', '=', False)
        ])
        order = self.env['sale.order'].browse(sale_order_id).exists()

        # Show onsite providers only if onsite carriers exists
        # and the order contains physical products
        if not onsite_carriers or not any(
            product.type in ('consu', 'product')
            for product in order.order_line.product_id
        ):
            compatible_providers = compatible_providers.filtered(
                lambda p: p.code != 'custom' or p.custom_mode != 'onsite'
            )

        return compatible_providers

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    picking_site_ids = fields.Many2many(
        'delivery.carrier',
        related='website_id.picking_site_ids',
        readonly=False,
    )

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Website(models.Model):
    _inherit = 'website'

    picking_site_ids = fields.Many2many('delivery.carrier', string='Picking sites',
                                        compute='_compute_picking_sites')

    def _compute_picking_sites(self):
        delivery_carriers = self.env['delivery.carrier'].search([('delivery_type', '=', 'onsite')])
        for website in self:
            website.picking_site_ids = delivery_carriers.filtered_domain([('website_id.id', '=', website.id)])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website
from . import res_config_settings
from . import payment_provider
from . import delivery_carrier

```

## File: static\src\js\checkout_form.js

```javascript
/** @odoo-module */

import publicWidget from 'web.public.widget';
import { _t } from 'web.core';
import 'website_sale_delivery.checkout';

publicWidget.registry.websiteSaleDelivery.include({
    start: function () {
        this.onsiteOptions = document.querySelectorAll('.o_payment_option_card input[type=radio][data-is-onsite="1"]');
        if(this.onsiteOptions.length > 0){ // Falsy evaluation does not work with NodeList
            this.paymentOptions = document.querySelectorAll('.o_payment_option_card input[type=radio]');

            this.warning = document.createElement('p');
            const boldMsg = document.createElement('b');
            boldMsg.innerText = _t('No suitable payment option could be found.');
            this.warning.innerText = _t('If you believe that it is an error, please contact the website administrator.');
            boldMsg.classList.add('d-block');
            this.warning.prepend(boldMsg);
            this.warning.classList.add('alert-warning', 'p-3', 'm-1', 'd-none');

            this.paymentOptionsContainer = document.querySelector('#payment_method');
            this.paymentOptionsContainer.querySelector('div.card').prepend(this.warning);
        }
        return this._super.apply(this, ...arguments);
    },

    /**
     * Hides or shows a payment option card.
     * @param node the input element of the payment option card
     * @param enabled whether to show or hide the card
     * @private
     */
    _setEnablePaymentOption(node, enabled) {
        if (enabled) {
            node.parentNode.parentNode.classList.remove('d-none');
        } else {
            node.parentNode.parentNode.classList.add('d-none');
            node.checked = false;
        }
    },

    /**
     * Checks all payment options and hides them if it is an onsite payment option and the delivery is not onsite.
     * @param {Event} ev the triggered document event
     * @private
     * @override
     */
    _onCarrierClick: function (ev) {
        this._super(...arguments);

        if(this.onsiteOptions.length === 0){ // Falsy evaluation does not work with NodeList
            return;
        }

        this.warning.classList.add('d-none');

        const input = ev.currentTarget.querySelector('input');
        let atLeastOneOptionAvailable = false;
        for (let option of this.paymentOptions) {
            if (option.dataset.isOnsite && input.dataset.deliveryType !== 'onsite') {
                this._setEnablePaymentOption(option, false);
            } else{
                if(option.dataset.isOnsite){
                    this._setEnablePaymentOption(option, true);
                }
                atLeastOneOptionAvailable = true;
            }
        }

        // Jquery because the button does not behave nicely with vanilla dataset.
        let $payButton = $('button[name="o_payment_submit_button"]');
        let disabledReasons = $payButton.data('disabled_reasons') || {};
        disabledReasons.noOptionAvailableOnsite = false;

        if (!atLeastOneOptionAvailable) {
            this.warning.classList.remove('d-none');
            disabledReasons.noOptionAvailableOnsite = true;
        } else if (this.paymentOptions.length === 1) {
            $(this.paymentOptions[0]).click(); // Make sure the option is selected if that's the only one, because the input is hidden in that case.
        }
        $payButton.data('disabled_reasons', disabledReasons);
    }
});

```

## File: views\delivery_view.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
     <record id="view_delivery_carrier_form_with_onsite_picking" model="ir.ui.view">
         <field name="inherit_id" ref="delivery.view_delivery_carrier_form"/>
         <field name="name">Delivery Carrier with Onsite Picking</field>
         <field name="model">delivery.carrier</field>
         <field name="arch" type="xml">
             <xpath expr="//field[@name='integration_level']" position="before">
                 <field attrs="{'invisible': [('delivery_type', '!=', 'onsite')]}" name="warehouse_id" options="{'no_quick_create': True}"
                        domain="[('company_id', '=?', company_id)]"/>
             </xpath>
         </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale.onsite</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_sale.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@id='onsite_payment_setting']/div[hasclass('o_setting_right_pane')]" position="inside">
                <div class="content-group">
                    <label class="col-lg-3" string="Picking sites" for="picking_site_ids"/>
                    <field name="picking_site_ids" domain="[('delivery_type', '=', 'onsite')]" widget="many2many_tags"
                           context="{'default_website_id': website_id, 'default_product_id': %(website_sale_picking.onsite_delivery_product)d, 'default_delivery_type': 'onsite', 'default_website_published': True, 'default_company_id': company_id}"/>
                </div>
                <div class="mt8">
                    <button name="%(delivery.action_delivery_carrier_form)d" icon="fa-arrow-right" type="action" string="Customize Pickup Sites" class="btn-link" context="{'search_default_delivery_type': 'onsite'}"/>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <template id="checkout_delivery" inherit_id="website_sale_delivery.payment_delivery_methods">
        <xpath expr="//input[@name='delivery_type']" position="attributes">
            <attribute name="t-att-data-delivery-type">delivery.delivery_type</attribute>
        </xpath>
    </template>

    <template id="checkout_payment" inherit_id="payment.checkout">
        <xpath expr="//input[@name='o_payment_radio']" position="attributes">
            <attribute name="t-att-data-is-onsite">1 if provider.custom_mode == 'onsite' else 0</attribute>
        </xpath>
    </template>

    <template id="payment_confirmation_status" inherit_id="website_sale.payment_confirmation_status">
        <xpath expr="(//div[hasclass('card-body')])[1]" position="replace">
            <t t-if="payment_tx_id.provider_id.custom_mode == 'onsite'">
                <div class="card-body">
                    <div class="o_header_carrier_message">
                        <b t-out="order.carrier_id.name"/><span class="text-muted"> (On site picking)</span>
                    </div>
                    <div class="o_body_carrier_message">
                        <t t-out="order.carrier_id.website_description"/>
                    </div>
                </div>
            </t>
            <t t-else="">
                <t>$0</t> <!-- Replace by old content -->
            </t>
        </xpath>
    </template>
</odoo>

```

