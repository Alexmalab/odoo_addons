# Odoo Module: website_sale_delivery_mondialrelay

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "website_sale_delivery_mondialrelay",
    'summary': """ Let's choose Point Relais® on your ecommerce """,

    'description': """
        This module allow your customer to choose a Point Relais® and use it as shipping address.
    """,
    'category': 'Website/Website',
    'version': '0.1',
    'depends': ['website_sale_delivery', 'delivery_mondialrelay'],
    'data': [
        'views/templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_delivery_mondialrelay/static/src/js/website_sale_delivery_mondialrelay.js',
        ],
    },
    'license': 'LGPL-3',
    'auto_install': True,
}

```

## File: controllers\controllers.py

```python
# -*- coding: utf-8 -*-
from odoo import http, _
from odoo.addons.website_sale.controllers.main import WebsiteSale, PaymentPortal
from odoo.addons.website_sale_delivery.controllers.main import WebsiteSaleDelivery

from odoo.exceptions import AccessDenied, ValidationError, UserError
from odoo.http import request


class MondialRelay(http.Controller):

    @http.route(['/website_sale_delivery_mondialrelay/update_shipping'], type='json', auth="public", website=True)
    def mondial_relay_update_shipping(self, **data):
        order = request.website.sale_get_order()

        if order.partner_id == request.website.user_id.sudo().partner_id:
            raise AccessDenied('Customer of the order cannot be the public user at this step.')

        if order.carrier_id.country_ids:
            country_is_allowed = data['Pays'][:2].upper() in order.carrier_id.country_ids.mapped(lambda c: c.code.upper())
            assert country_is_allowed, _("%s is not allowed for this delivery carrier.", data['Pays'])

        partner_shipping = order.partner_id.sudo()._mondialrelay_search_or_create({
            'id': data['ID'],
            'name': data['Nom'],
            'street': data['Adresse1'],
            'street2': data['Adresse2'],
            'zip': data['CP'],
            'city': data['Ville'],
            'country_code': data['Pays'][:2].lower(),
        })
        if order.partner_shipping_id != partner_shipping:
            order.partner_shipping_id = partner_shipping
            order.onchange_partner_shipping_id()

        return {
            'address': request.env['ir.qweb']._render('website_sale.address_on_payment', {
                'order': order,
                'only_services': order and order.only_services,
            }),
            'new_partner_shipping_id': order.partner_shipping_id.id,
        }


class WebsiteSaleMondialrelay(WebsiteSale):

    @http.route()
    def address(self, **kw):
        res = super().address(**kw)
        Partner_sudo = request.env['res.partner'].sudo()
        partner_id = res.qcontext.get('partner_id', 0)
        if partner_id > 0 and Partner_sudo.browse(partner_id).is_mondialrelay:
            raise UserError(_('You cannot edit the address of a Point Relais®.'))
        return res


class WebsiteSaleDeliveryMondialrelay(WebsiteSaleDelivery):

    def _update_website_sale_delivery_return(self, order, **post):
        res = super()._update_website_sale_delivery_return(order, **post)
        if order.carrier_id.is_mondialrelay:
            res['mondial_relay'] = {
                'brand': order.carrier_id.mondialrelay_brand,
                'col_liv_mod': order.carrier_id.mondialrelay_packagetype,
                'partner_zip': order.partner_shipping_id.zip,
                'partner_country_code': order.partner_shipping_id.country_id.code.upper(),
                'allowed_countries': ','.join(order.carrier_id.country_ids.mapped('code')).upper(),
            }
            if order.partner_shipping_id.is_mondialrelay:
                res['mondial_relay']['current'] = '%s-%s' % (
                    res['mondial_relay']['partner_country_code'],
                    order.partner_shipping_id.ref.lstrip('MR#'),
                )

        return res


class PaymentPortalMondialRelay(PaymentPortal):

    @http.route()
    def shop_payment_transaction(self, *args, **kwargs):
        order = request.website.sale_get_order()
        if order.partner_shipping_id.is_mondialrelay and order.carrier_id and not order.carrier_id.is_mondialrelay and order.delivery_set:
            raise ValidationError(_('Point Relais® can only be used with the delivery method Mondial Relay.'))
        elif not order.partner_shipping_id.is_mondialrelay and order.carrier_id.is_mondialrelay:
            raise ValidationError(_('Delivery method Mondial Relay can only ship to Point Relais®.'))
        return super().shop_payment_transaction(*args, **kwargs)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import controllers

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class WebsiteMondialRelay(models.Model):
    _inherit = 'website'

    def _prepare_sale_order_values(self, partner, pricelist):
        self.ensure_one()
        values = super()._prepare_sale_order_values(partner, pricelist)
        # never use Mondial Relay shipping address as default.
        shipping_address = self.env['res.partner'].browse(values['partner_shipping_id'])
        if shipping_address.id != values['partner_invoice_id'] and shipping_address.is_mondialrelay:
            values['partner_shipping_id'] = values['partner_invoice_id']
        return values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import website

```

## File: static\src\js\website_sale_delivery_mondialrelay.js

```javascript
/** @odoo-module **/

import publicWidget from "web.public.widget";
import "website_sale_delivery.checkout";
import {qweb as QWeb} from "web.core";

const WebsiteSaleDeliveryWidget = publicWidget.registry.websiteSaleDelivery;

// temporary for OnNoResultReturned bug
import {registry} from "@web/core/registry";
import {UncaughtCorsError} from "@web/core/errors/error_service";
const errorHandlerRegistry = registry.category("error_handlers");

function corsIgnoredErrorHandler(env, error) {
    if (error instanceof UncaughtCorsError) {
        return true;
    }
}

WebsiteSaleDeliveryWidget.include({
    xmlDependencies: (WebsiteSaleDeliveryWidget.prototype.xmlDependencies || []).concat([
        '/website_sale_delivery_mondialrelay/static/src/xml/website_sale_delivery_mondialrelay.xml',
    ]),
    events: _.extend({
        "click #btn_confirm_relay": "_onClickBtnConfirmRelay",
    }, WebsiteSaleDeliveryWidget.prototype.events),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Loads Mondial Relay the first time, else show it.
     *
     * @override
     */
    _handleCarrierUpdateResult: function (result) {
        this._super(...arguments);
        if (result.mondial_relay) {
            if (!$('#modal_mondialrelay').length) {
                this._loadMondialRelayModal(result);
            } else {
                this.$modal_mondialrelay.find('#btn_confirm_relay').toggleClass('disabled', !result.mondial_relay.current);
                this.$modal_mondialrelay.modal('show');
            }
        }
    },
    /**
     * This method render the modal, and inject it in dom with the Modial Relay Widgets script.
     * Once script loaded, it initialize the widget pre-configured with the information of result
     *
     * @private
     *
     * @param {Object} result: dict returned by call of _update_website_sale_delivery_return (python)
     */
    _loadMondialRelayModal: function (result) {
        // add modal to body and bind 'save' button
        $(QWeb.render('website_sale_delivery_mondialrelay', {})).appendTo('body');
        this.$modal_mondialrelay = $('#modal_mondialrelay');
        this.$modal_mondialrelay.find('#btn_confirm_relay').on('click', this._onClickBtnConfirmRelay.bind(this));

        // load mondial relay script
        const script = document.createElement('script');
        script.src = "https://widget.mondialrelay.com/parcelshop-picker/jquery.plugin.mondialrelay.parcelshoppicker.min.js";
        script.onload = () => {
            // instanciate MondialRelay widget
            const params = {
                Target: "", // required but handled by OnParcelShopSelected
                Brand: result.mondial_relay.brand,
                ColLivMod: result.mondial_relay.col_liv_mod,
                AllowedCountries: result.mondial_relay.allowed_countries,
                Country: result.mondial_relay.partner_country_code,
                PostCode: result.mondial_relay.partner_zip,
                Responsive: true,
                ShowResultsOnMap: true,
                AutoSelect: result.mondial_relay.current,
                OnParcelShopSelected: (RelaySelected) => {
                    this.lastRelaySelected = RelaySelected;
                    this.$modal_mondialrelay.find('#btn_confirm_relay').removeClass('disabled');
                },
                OnNoResultReturned: () => {
                    // HACK while Mondial Relay fix his bug
                    // disable corsErrorHandler for 10 seconds
                    // If code postal not valid, it will crash with Cors Error:
                    // Cannot read property 'on' of undefined at u.MR_FitBounds
                    const randInt = Math.floor(Math.random() * 100);
                    errorHandlerRegistry.add("corsIgnoredErrorHandler" + randInt, corsIgnoredErrorHandler, {sequence: 10});
                    setTimeout(function () {
                        errorHandlerRegistry.remove("corsIgnoredErrorHandler" + randInt);
                    }, 10000);
                },
            };
            this.$modal_mondialrelay.find('#o_zone_widget').MR_ParcelShopPicker(params);
            this.$modal_mondialrelay.modal('show');
            this.$modal_mondialrelay.find('#o_zone_widget').trigger("MR_RebindMap");
        };
        document.body.appendChild(script);

    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------


    /**
     * Update the shipping address on the order and refresh the UI.
     *
     * @private
     *
     */
    _onClickBtnConfirmRelay: function () {
        if (!this.lastRelaySelected) {
            return;
        }
        this._rpc({
            route: '/website_sale_delivery_mondialrelay/update_shipping',
            params: {
                ...this.lastRelaySelected,
            },
        }).then((o) => {
            $('#address_on_payment').html(o.address);
            this.$modal_mondialrelay.modal('hide');
        });
    },
});

```

## File: static\src\xml\website_sale_delivery_mondialrelay.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="website_sale_delivery_mondialrelay">
         <div class="modal fade" id="modal_mondialrelay" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <div class="modal-header">
                        <div class="h5 modal-title">Choose your Parcel Point</div>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span>×</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="w-100" id="o_zone_widget">
                            <!-- here will be instancied the  Widget -->
                        </div>
                    </div>
                    <footer class="modal-footer">
                        <button type="button" id="btn_confirm_relay" class="btn btn-primary disabled" aria-label="Choose this Parcel Point">Send to this Parcel Point</button>
                        <button type="button" class="btn" data-dismiss="modal" aria-label="Cancel">Cancel</button>
                    </footer>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: views\templates.xml

```xml
<odoo>
    <data>
        <template id="website_sale_delivery_mondialrelay_address_on_payment" inherit_id="website_sale.address_on_payment">
            <xpath expr="//span[@t-esc='order.partner_shipping_id']" position="before">
                <t t-if="order.partner_shipping_id.is_mondialrelay" >
                    <img src="/website_sale_delivery_mondialrelay/static/src/img/logo.png" title="Mondial Relay" height="20px" />
                </t>
            </xpath>
        </template>

        <template id="website_sale_delivery_mondialrelay_checkout" inherit_id="website_sale.checkout">
            <xpath expr="//t[@t-set='allow_edit']" position="after">
                <t t-set="allow_edit" t-value="allow_edit and not contact.is_mondialrelay"/>
            </xpath>
        </template>

        <template id="website_sale_delivery_mondialrelay_address_kanban" inherit_id="website_sale.address_kanban">
            <xpath expr="//t[@t-esc='contact']" position="before">
                <t t-if="contact.is_mondialrelay">
                    <img class="float-right" title="Mondial Relay" height="20px" src="/website_sale_delivery_mondialrelay/static/src/img/logo.png" />
                </t>
            </xpath>
        </template>
    </data>
</odoo>

```

