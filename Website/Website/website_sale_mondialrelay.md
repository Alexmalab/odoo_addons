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
        'views/delivery_form_templates.xml',
        'views/res_config_settings_views.xml',
        'views/templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_mondialrelay/static/src/**/*',
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
from odoo.addons.website_sale.controllers.delivery import Delivery

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
            'phone': order.partner_id.phone,
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

    def _prepare_address_update(self, *args, **kwargs):
        """Updates of mondialrelay addresses are forbidden"""
        partner_sudo, _address_type = super()._prepare_address_update(*args, **kwargs)

        if partner_sudo and partner_sudo.is_mondialrelay:
            raise UserError(_('You cannot edit the address of a Point Relais®.'))

        return partner_sudo, _address_type

    def _check_delivery_address(self, partner_sudo):
        # skip check for mondialrelay partners as the customer can not edit them
        if partner_sudo.is_mondialrelay:
            return True
        return super()._check_delivery_address(partner_sudo)


class WebsiteSaleDeliveryMondialrelay(Delivery):

    def _order_summary_values(self, order, **post):
        res = super()._order_summary_values(order, **post)
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

    def _can_be_edited_by_current_customer(self, *args, **kwargs):
        return super()._can_be_edited_by_current_customer(*args, **kwargs) and not self.is_mondialrelay

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

    def _compute_partner_shipping_id(self):
        super()._compute_partner_shipping_id()
        ecommerce_orders = self.filtered('website_id')
        for order in ecommerce_orders:
            if order.partner_shipping_id.is_mondialrelay and not order.carrier_id.is_mondialrelay:
                order.partner_shipping_id = order.partner_id

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import sale_order

```

## File: static\src\js\checkout.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import "@website_sale/js/checkout";
import { rpc } from "@web/core/network/rpc";
import { renderToElement } from "@web/core/utils/render";

const WebsiteSaleCheckout = publicWidget.registry.WebsiteSaleCheckout;

// temporary for OnNoResultReturned bug
import {registry} from "@web/core/registry";
import {ThirdPartyScriptError} from "@web/core/errors/error_service";
const errorHandlerRegistry = registry.category("error_handlers");

function corsIgnoredErrorHandler(env, error) {
    if (error instanceof ThirdPartyScriptError) {
        return true;
    }
}

WebsiteSaleCheckout.include({
    events: Object.assign({
        "click #btn_confirm_relay": "_onClickBtnConfirmRelay",
    }, WebsiteSaleCheckout.prototype.events),

    /**
     * Do not allow use same as delivery if delivery method mondialrelay or mondialrelay address is
     * selected.
     *
     * @override of `website_sale`
     */
    async start() {
        await this._super(...arguments);
        this.$('#use_delivery_as_billing_label')?.tooltip();
        this._adaptUseDeliveryAsBillingToggle();
    },

    /**
     * Loads Mondial Relay modal when method is selected and disable `use_delivery_as_billing` if
     * not available.
     *
     * @override
     */
    async _selectDeliveryMethod(ev) {
        const checkedRadio = ev.currentTarget;
        await this._super(...arguments);
        if (checkedRadio.dataset.isMondialrelay) {
            if (this.use_delivery_as_billing_toggle?.checked) {
                // Uncheck use same as delivery and show the billing address row.
                this.use_delivery_as_billing_toggle.dispatchEvent(new MouseEvent('click'));
            }
            // Fetch delivery method data.
            const result = await this._setDeliveryMethod(checkedRadio.dataset.dmId);
            // Show mondialrelay modal.
            if (!$('#modal_mondialrelay').length) {
                this._loadMondialRelayModal(result);
            } else {
                this.$modal_mondialrelay.find('#btn_confirm_relay').toggleClass(
                    'disabled', !result.mondial_relay.current
                );
                this.$modal_mondialrelay.modal('show');
            }
        }
        this._adaptUseDeliveryAsBillingToggle();
    },

    /**
     * If mondialrelay address is chosen uncheck `use billing as delivery` and show billing address
     * row. Mondialrelay addresses are not allowed to be selected as billing.
     *
     * @override of `website_sale`
     */
    async _changeAddress(ev) {
        const newAddress = ev.currentTarget;
        if (newAddress.dataset.isMondialrelay && this.use_delivery_as_billing_toggle?.checked) {
            // Uncheck use same as delivery and show the billing address row.
            this.use_delivery_as_billing_toggle.dispatchEvent(new MouseEvent('click'));
        }
        await this._super(...arguments);
        this._adaptUseDeliveryAsBillingToggle();
    },

    /**
     * Disable use same as delivery when delivery method mondialrelay or mondialrelay address is
     * selected, otherwise enable it.
     *
     * @private
     * @return {void}
     */
    _adaptUseDeliveryAsBillingToggle() {
        if (this.use_delivery_as_billing_toggle) {
            const checkedRadio = document.querySelector('input[name="o_delivery_radio"]:checked');
            const selectedDeliveryAddress = this._getSelectedAddress('delivery');
            const requireSeparateBillingAddress = (
                checkedRadio?.dataset.isMondialrelay
                || selectedDeliveryAddress?.dataset.isMondialrelay
            );
            this.use_delivery_as_billing_toggle.disabled = requireSeparateBillingAddress;
            this.$('#use_delivery_as_billing_label').tooltip(
                requireSeparateBillingAddress ? 'enable' : 'disable'
            );
        }
    },

    /**
     * This method render the modal, and inject it in dom with the Modial Relay Widgets script.
     * Once script loaded, it initialize the widget pre-configured with the information of result
     *
     * @private
     *
     * @param {Object} result: dict returned by call of _order_summary_values (python)
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
        rpc('/website_sale_mondialrelay/update_shipping', {
            ...this.lastRelaySelected,
        }).then(() => {
            location.reload(); // Update the addresses.
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

## File: views\delivery_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="mondialrelay_delivery_method" inherit_id="website_sale.delivery_method">
        <input name="o_delivery_radio" position="attributes">
            <attribute name="t-att-data-is-mondialrelay">dm.is_mondialrelay</attribute>
        </input>
        <label class="o_delivery_carrier_label" position="after">
            <small t-if="dm.is_mondialrelay" class="text-muted my-auto">
                Click to choose a pickup point
            </small>
        </label>
    </template>

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

    <template
        id="website_sale_mondialrelay_billing_address_row"
        inherit_id="website_sale.billing_address_row"
    >
        <label id="use_delivery_as_billing_label" position="attributes">
            <attribute name="title">Unavailable with Mondial Relay</attribute>
            <attribute name="data-bs-toggle">tooltip</attribute>
            <attribute name="data-bs-trigger">hover focus</attribute>
        </label>
    </template>

    <template id="website_sale_mondialrelay_address_kanban" inherit_id="website_sale.address_kanban">
        <xpath expr="//t[@t-esc='contact']" position="before">
            <t t-if="contact.is_mondialrelay">
                <img class="float-end" title="Mondial Relay" height="20px" src="/website_sale_mondialrelay/static/src/img/logo.png" />
            </t>
        </xpath>
        <div name="address_card" position="attributes">
            <attribute name="t-att-data-is-mondialrelay">contact.is_mondialrelay</attribute>
        </div>
    </template>

    <template id="website_sale_mondialrelay_address_on_payment" inherit_id="website_sale.address_on_payment">
        <xpath expr="//span[@t-out='order.partner_shipping_id']" position="before">
            <t t-if="order.partner_shipping_id.is_mondialrelay" >
                <img src="/website_sale_mondialrelay/static/src/img/logo.png" title="Mondial Relay" height="20px" />
            </t>
        </xpath>
    </template>

</odoo>

```

