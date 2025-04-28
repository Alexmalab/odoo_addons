# Odoo Module: website_sale_mondialrelay

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
    'name': "eCommerce Mondialrelay Delivery",
    'summary': "Let's choose Point Relais® on your ecommerce",

    'description': """
This module allow your customer to choose a Point Relais® and use it as shipping address.
    """,
    'category': 'Website/Website',
    'version': '0.1',
    'depends': ['website_sale', 'delivery_mondialrelay'],
    'data': [
        'views/delivery_carrier_views.xml',
        'views/res_config_settings_views.xml',
        'views/templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_mondialrelay/static/src/js/website_sale_mondialrelay.js',
            'website_sale_mondialrelay/static/src/xml/website_sale_mondialrelay.xml',
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
from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.addons.website_sale.controllers.delivery import WebsiteSaleDelivery

from odoo.exceptions import AccessDenied, UserError
from odoo.http import request


class MondialRelay(http.Controller):

    @http.route(['/website_sale_mondialrelay/update_shipping'], type='json', auth="public", website=True)
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

    def _check_shipping_partner_mandatory_fields(self, partner_id):
        # skip check for mondialrelay partners as the user can not edit them
        if partner_id.is_mondialrelay:
            return True
        return super()._check_shipping_partner_mandatory_fields(partner_id)


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

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import controllers

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    def _can_be_edited_by_current_customer(self, sale_order, mode):
        return super()._can_be_edited_by_current_customer(sale_order, mode) and not self.is_mondialrelay

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import ValidationError


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _check_cart_is_ready_to_be_paid(self):
        if (
            self.partner_shipping_id.is_mondialrelay and self.delivery_set
            and self.carrier_id and not self.carrier_id.is_mondialrelay
        ):
            raise ValidationError(_(
                "Point Relais® can only be used with the delivery method Mondial Relay."
            ))
        elif not self.partner_shipping_id.is_mondialrelay and self.carrier_id.is_mondialrelay:
            raise ValidationError(_(
                "Delivery method Mondial Relay can only ship to Point Relais®."
            ))
        return super()._check_cart_is_ready_to_be_paid()

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class WebsiteMondialRelay(models.Model):
    _inherit = 'website'

    def _prepare_sale_order_values(self, partner_sudo):
        values = super()._prepare_sale_order_values(partner_sudo)

        # never use Mondial Relay shipping address as default.
        shipping_address = self.env['res.partner'].browse(values['partner_shipping_id'])
        if shipping_address.id != values['partner_invoice_id'] and shipping_address.is_mondialrelay:
            values['partner_shipping_id'] = values['partner_invoice_id']
        return values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import res_partner
from . import sale_order
from . import website

```

## File: static\src\js\website_sale_mondialrelay.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import "@website_sale/js/website_sale_delivery";
import { renderToElement } from "@web/core/utils/render";

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
    events: Object.assign({
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
    _handleCarrierUpdateResult: async function (carrierInput) {
        await this._super(...arguments);
        if (this.result.mondial_relay) {
            if (!$('#modal_mondialrelay').length) {
                this._loadMondialRelayModal(this.result);
            } else {
                this.$modal_mondialrelay.find('#btn_confirm_relay').toggleClass('disabled', !this.result.mondial_relay.current);
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
        $(renderToElement('website_sale_mondialrelay', {})).appendTo('body');
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
        this.rpc('/website_sale_mondialrelay/update_shipping', {
            ...this.lastRelaySelected,
        }).then((o) => {
            $('#address_on_payment').html(o.address);
            this.$modal_mondialrelay.modal('hide');
        });
    },
});

```

## File: static\src\xml\website_sale_mondialrelay.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="website_sale_mondialrelay">
         <div class="modal fade" id="modal_mondialrelay" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <div class="modal-header">
                        <div class="h5 modal-title">Choose your Parcel Point</div>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <div class="w-100" id="o_zone_widget">
                            <!-- here will be instancied the  Widget -->
                        </div>
                    </div>
                    <footer class="modal-footer">
                        <button type="button" id="btn_confirm_relay" class="btn btn-primary disabled" aria-label="Choose this Parcel Point">Send to this Parcel Point</button>
                        <button type="button" class="btn" data-bs-dismiss="modal" aria-label="Cancel">Cancel</button>
                    </footer>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: views\delivery_carrier_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="delivery_carrier_view_search" model="ir.ui.view">
        <field name="name">delivery.carrier.view.search.inherit.website.sale.mondialrelay</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='inactive']" position="before">
                <filter string="Mondial Relay" name="is_mondialrelay" domain="[('is_mondialrelay', '=', True)]"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale.mondialrelay</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="shipping_provider_mondialrelay_setting" position="inside">
                <div class="content-group">
                    <div class="mt8" invisible="not module_delivery_mondialrelay">
                        <button name="%(delivery.action_delivery_carrier_form)d" icon="oi-arrow-right" type="action" string="Mondial Relay Shipping Methods" class="btn-link" context="{'search_default_is_mondialrelay': True}"/>
                    </div>
                </div>
            </setting>
        </field>
    </record>
</odoo>

```

## File: views\templates.xml

```xml
<odoo>

    <template id="website_sale_mondialrelay_address_on_payment" inherit_id="website_sale.address_on_payment">
        <xpath expr="//span[@t-esc='order.partner_shipping_id'] | //span[@id='shipping_on_payment_details']" position="before">
            <t t-if="order.partner_shipping_id.is_mondialrelay" >
                <img src="/website_sale_mondialrelay/static/src/img/logo.png" title="Mondial Relay" height="20px" />
            </t>
        </xpath>
    </template>

    <template id="website_sale_mondialrelay_address_kanban" inherit_id="website_sale.address_kanban">
        <xpath expr="//t[@t-esc='contact']" position="before">
            <t t-if="contact.is_mondialrelay">
                <img class="float-end" title="Mondial Relay" height="20px" src="/website_sale_mondialrelay/static/src/img/logo.png" />
            </t>
        </xpath>
    </template>

</odoo>

```

