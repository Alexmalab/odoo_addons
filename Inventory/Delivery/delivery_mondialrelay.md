# Odoo Module: delivery_mondialrelay

Category: Inventory/Delivery

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "delivery_mondialrelay",
    'summary': """ Let's choose a Point Relais® as shipping address """,

    'description': """
This module allow your customer to choose a Point Relais® and use it as shipping address.
This module doesn't implement the WebService. It is only the integration of the widget.

Delivery price pre-configured is an example, you need to adapt the pricing's rules.
    """,
    'category': 'Inventory/Delivery',
    'version': '0.1',
    'depends': ['stock_delivery'],
    'data': [
        'data/data.xml',
        'views/views.xml',
        'wizard/choose_delivery_carrier_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'delivery_mondialrelay/static/src/components/**/*.js',
            'delivery_mondialrelay/static/src/scss/mondialrelay.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="product_product_delivery_mondialrelay" model="product.product">
            <field name="name">Mondial Relay</field>
            <field name="default_code">MR</field>
            <field name="type">service</field>
            <field name="categ_id" ref="delivery.product_category_deliveries"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="list_price">5</field>
            <field name="invoice_policy">order</field>
        </record>

        <record id="delivery_carrier_mondialrelay_be_lu" model="delivery.carrier">
            <field name="name">Mondial Relay</field>
            <field name="sequence">30</field>
            <field name="delivery_type">base_on_rule</field>
            <field name="integration_level">rate</field>
            <field name="country_ids" eval="[(6, 0, [ref('base.be'), ref('base.lu')])]"/>
            <field name="product_id" ref="delivery_mondialrelay.product_product_delivery_mondialrelay"/>
        </record>
        <record id="delivery_price_rule1" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier_mondialrelay_be_lu"/>
            <field name="max_value" eval="10"/>
            <field name="list_base_price" eval="4"/>
        </record>
        <record id="delivery_price_rule2" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier_mondialrelay_be_lu"/>
            <field name="operator">&gt;=</field>
            <field name="max_value" eval="10"/>
            <field name="list_base_price" eval="5"/>
        </record>

        <record id="delivery_carrier_mondialrelay_fr_nl" model="delivery.carrier">
            <field name="name">Mondial Relay</field>
            <field name="sequence">31</field>
            <field name="delivery_type">base_on_rule</field>
            <field name="integration_level">rate</field>
            <field name="country_ids" eval="[(6, 0, [ref('base.fr'), ref('base.nl')])]"/>
            <field name="product_id" ref="delivery_mondialrelay.product_product_delivery_mondialrelay"/>
        </record>

        <record id="delivery_price_rule_fr_nl_1" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier_mondialrelay_fr_nl"/>
            <field name="max_value" eval="1"/>
            <field name="list_base_price" eval="6"/>
        </record>
        <record id="delivery_price_rule_fr_nl_2" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier_mondialrelay_fr_nl"/>
            <field name="max_value" eval="10"/>
            <field name="list_base_price" eval="10"/>
        </record>
        <record id="delivery_price_rule_fr_nl_3" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier_mondialrelay_fr_nl"/>
            <field name="operator">&gt;=</field>
            <field name="max_value" eval="10"/>
            <field name="list_base_price" eval="20"/>
        </record>

        <record id="delivery_carrier_mondialrelay_es" model="delivery.carrier">
            <field name="name">Mondial Relay</field>
            <field name="sequence">32</field>
            <field name="delivery_type">base_on_rule</field>
            <field name="integration_level">rate</field>
            <field name="country_ids" eval="[(6, 0, [ref('base.es')])]"/>
            <field name="product_id" ref="delivery_mondialrelay.product_product_delivery_mondialrelay"/>
        </record>

        <record id="delivery_price_rule_es_1" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier_mondialrelay_es"/>
            <field name="max_value" eval="3"/>
            <field name="list_base_price" eval="10"/>
        </record>
        <record id="delivery_price_rule_es_2" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier_mondialrelay_es"/>
            <field name="max_value" eval="7"/>
            <field name="list_base_price" eval="15"/>
        </record>
        <record id="delivery_price_rule_es_3" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier_mondialrelay_es"/>
            <field name="operator">&gt;=</field>
            <field name="max_value" eval="10"/>
            <field name="list_base_price" eval="28"/>
        </record>
    </data>
</odoo>

```

## File: data\neutralize.sql

```sql
-- disable mondialrelay
UPDATE delivery_carrier
SET mondialrelay_brand = 'BDTEST  ';

```

## File: models\delivery_carrier.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models, api
from odoo.exceptions import UserError


class DeliveryCarrierMondialRelay(models.Model):
    _inherit = 'delivery.carrier'

    is_mondialrelay = fields.Boolean(compute='_compute_is_mondialrelay', search='_search_is_mondialrelay')
    mondialrelay_brand = fields.Char(string='Brand Code', default='BDTEST  ')
    mondialrelay_packagetype = fields.Char(default="24R", groups="base.group_system")  # Advanced

    @api.depends('product_id.default_code')
    def _compute_is_mondialrelay(self):
        for c in self:
            c.is_mondialrelay = c.product_id.default_code == "MR"

    def _search_is_mondialrelay(self, operator, value):
        if operator not in ('=', '!=') or not isinstance(value, bool):
            raise UserError(_("Operation not supported"))
        if not value:
            operator = '!=' if operator == '=' else '='
        return [('product_id.default_code', operator, 'MR')]

    def fixed_get_tracking_link(self, picking):
        return self.base_on_rule_get_tracking_link(picking)

    def base_on_rule_get_tracking_link(self, picking):
        if self.is_mondialrelay:
            return 'https://www.mondialrelay.com/public/permanent/tracking.aspx?ens=%(brand)s&exp=%(track)s&language=%(lang)s' % {
                'brand': picking.carrier_id.mondialrelay_brand,
                'track': picking.carrier_tracking_ref,
                'lang': (picking.partner_id.lang or 'fr').split('_')[0],
            }
        return super().base_on_rule_get_tracking_link(picking)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResPartnerMondialRelay(models.Model):
    _inherit = 'res.partner'

    is_mondialrelay = fields.Boolean(compute='_compute_is_mondialrelay')

    @api.depends('ref')
    def _compute_is_mondialrelay(self):
        for p in self:
            p.is_mondialrelay = p.ref and p.ref.startswith('MR#')

    @api.model
    def _mondialrelay_search_or_create(self, data):
        ref = 'MR#%s' % data['id']
        partner = self.search([
            ('id', 'child_of', self.commercial_partner_id.ids),
            ('ref', '=', ref),
            # fast check that address always the same
            ('street', '=', data['street']),
            ('zip', '=', data['zip']),
        ])
        if not partner:
            partner = self.create({
                'ref': ref,
                'name': data['name'],
                'street': data['street'],
                'street2': data['street2'],
                'zip': data['zip'],
                'city': data['city'],
                'country_id': self.env.ref('base.%s' % data['country_code']).id,
                'type': 'delivery',
                'parent_id': self.id,
            })
        return partner

    def _avatar_get_placeholder_path(self):
        if self.is_mondialrelay:
            return "delivery_mondialrelay/static/src/img/truck_mr.png"
        return super()._avatar_get_placeholder_path()

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.exceptions import UserError


class SaleOrderMondialRelay(models.Model):
    _inherit = 'sale.order'

    def action_confirm(self):
        unmatch = self.filtered(lambda so: so.carrier_id.is_mondialrelay != so.partner_shipping_id.is_mondialrelay)
        if unmatch:
            error = _('Mondial Relay mismatching between delivery method and shipping address.')
            if len(self) > 1:
                error += ' (%s)' % ','.join(unmatch.mapped('name'))
            raise UserError(error)
        return super().action_confirm()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import delivery_carrier
from . import res_partner
from . import sale_order

```

## File: static\src\components\mondialrelay_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { loadJS } from "@web/core/assets";

// temporary for OnNoResultReturned bug
import { UncaughtCorsError } from "@web/core/errors/error_service";
const errorHandlerRegistry = registry.category("error_handlers");
import { Component, onWillRender, useEffect, useRef, useState, xml } from "@odoo/owl";

const MONDIALRELAY_SCRIPT_URL = "https://widget.mondialrelay.com/parcelshop-picker/jquery.plugin.mondialrelay.parcelshoppicker.min.js"

function corsIgnoredErrorHandler(env, error) {
    if (error instanceof UncaughtCorsError) {
        return true;
    }
}

export class MondialRelayField extends Component {
    setup() {
        this.root = useRef("root");
        this.state = useState({
            libLoaded: false, // Whether the library is loaded or not
        });
        onWillRender(() => {
            // Do nothing if the record is not of type mondial_relay
            if (!this.enabled || this.state.libLoaded) {
                return;
            }
            loadJS(MONDIALRELAY_SCRIPT_URL).then(() => {this.state.libLoaded = true});
        });

        useEffect(
            (el) => {
                if (!el) {
                    return;
                }
                this.insertWidget($(el));
            },
            () => [this.state.libLoaded && this.root.el],
        )
    }

    get enabled() {
        return this.props.record.data.is_mondialrelay;
    }

    insertWidget($el) {
        const params = {
            Target: "", // required but handled by OnParcelShopSelected
            Brand: this.props.record.data.mondialrelay_brand,
            ColLivMod: this.props.record.data.mondial_realy_colLivMod,
            AllowedCountries: this.props.record.data.mondialrelay_allowed_countries,
            PostCode: this.props.record.data.shipping_zip || '',
            Country: this.props.record.data.shipping_country_code  || '',
            Responsive: true,
            ShowResultsOnMap: true,
            AutoSelect: this.props.record.data.mondialrelay_last_selected_id,
            OnParcelShopSelected: (RelaySelected) => {
                const values = JSON.stringify({
                    'id': RelaySelected.ID,
                    'name': RelaySelected.Nom,
                    'street': RelaySelected.Adresse1,
                    'street2': RelaySelected.Adresse2,
                    'zip': RelaySelected.CP,
                    'city': RelaySelected.Ville,
                    'country': RelaySelected.Pays,
                });
                this.props.record.update({ [this.props.name]: values });
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
        $el.show();
        $el.MR_ParcelShopPicker(params);
        $el.trigger("MR_RebindMap");
    }
}
MondialRelayField.template = xml`<div t-if="enabled" t-ref="root"/>`;

export const mondialRelayField = {
    component: MondialRelayField,
};

registry.category("fields").add("mondialrelay_relay", mondialRelayField);

```

## File: views\views.xml

```xml
<odoo>
  <data>

    <record id="view_delivery_carrier_form_provider_mondialrelay" model="ir.ui.view">
        <field name="name">delivery.carrier.form.provider.mondialrelay</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_form"/>
        <field name="arch" type="xml">
            <field name="product_id" position="after">
                <field name="is_mondialrelay" invisible="1" />
                <field name="mondialrelay_brand" invisible="not is_mondialrelay" required="is_mondialrelay" />
            </field>
        </field>
    </record>

    <record id="view_delivery_carrier_tree_provider_mondialrelay" model="ir.ui.view">
        <field name="name">delivery.carrier.tree.provider.mondialrelay</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_tree"/>
        <field name="arch" type="xml">
            <field name="country_ids" position="attributes">
                <attribute name="optional">show</attribute>
            </field>
        </field>
    </record>

  </data>
</odoo>

```

## File: wizard\choose_delivery_carrier.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.tools.json import scriptsafe as json_safe
from odoo.exceptions import ValidationError


class ChooseDeliveryCarrier(models.TransientModel):
    _inherit = 'choose.delivery.carrier'

    shipping_zip = fields.Char(related='order_id.partner_shipping_id.zip')
    shipping_country_code = fields.Char(related='order_id.partner_shipping_id.country_id.code')

    is_mondialrelay = fields.Boolean(compute='_compute_is_mondialrelay')
    mondialrelay_last_selected = fields.Char(string="Last Relay Selected")
    mondialrelay_last_selected_id = fields.Char(compute='_compute_mr_last_selected_id')
    mondialrelay_brand = fields.Char(related='carrier_id.mondialrelay_brand')
    mondialrelay_colLivMod = fields.Char(related='carrier_id.mondialrelay_packagetype')
    mondialrelay_allowed_countries = fields.Char(compute='_compute_mr_allowed_countries')

    @api.depends('carrier_id')
    def _compute_is_mondialrelay(self):
        self.ensure_one()
        self.is_mondialrelay = self.carrier_id.product_id.default_code == "MR"

    @api.depends('carrier_id', 'order_id.partner_shipping_id')
    def _compute_mr_last_selected_id(self):
        self.ensure_one()
        if self.order_id.partner_shipping_id.is_mondialrelay:
            self.mondialrelay_last_selected_id = '%s-%s' % (
                self.shipping_country_code,
                self.order_id.partner_shipping_id.ref.lstrip('MR#'),
            )
        else:
            self.mondialrelay_last_selected_id = ''

    @api.depends('carrier_id')
    def _compute_mr_allowed_countries(self):
        self.ensure_one()
        self.mondialrelay_allowed_countries = ','.join(self.carrier_id.country_ids.mapped('code')).upper() or ''

    def button_confirm(self):
        if self.carrier_id.is_mondialrelay:
            if not self.mondialrelay_last_selected:
                raise ValidationError(_('Please, choose a Parcel Point'))
            data = json_safe.loads(self.mondialrelay_last_selected)
            partner_shipping = self.order_id.partner_id._mondialrelay_search_or_create({
                'id': data['id'],
                'name': data['name'],
                'street': data['street'],
                'street2': data['street2'],
                'zip': data['zip'],
                'city': data['city'],
                'country_code': data['country'][:2].lower(),
            })
            if partner_shipping != self.order_id.partner_shipping_id:
                self.order_id.partner_shipping_id = partner_shipping

        return super().button_confirm()

```

## File: wizard\choose_delivery_carrier_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="choose_delivery_carrier_view_form" model="ir.ui.view">
        <field name="name">choose.delivery.carrier.form</field>
        <field name="model">choose.delivery.carrier</field>
        <field name="inherit_id" ref="delivery.choose_delivery_carrier_view_form"/>
        <field name="arch" type="xml">
            <form position="inside">
                <field name="is_mondialrelay" invisible="1"/>
                <div class="o_zone_widget" invisible="not is_mondialrelay"/>
                <field name="mondialrelay_last_selected" widget="mondialrelay_relay"/>
                <field name="mondialrelay_last_selected_id" invisible="1"/>
                <field name="mondialrelay_brand" invisible="1"/>
                <field name="mondialrelay_colLivMod" invisible="1"/>
                <field name="mondialrelay_allowed_countries" invisible="1"/>
                <field name="shipping_zip" invisible="1"/>
                <field name="shipping_country_code" invisible="1"/>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import choose_delivery_carrier

```

