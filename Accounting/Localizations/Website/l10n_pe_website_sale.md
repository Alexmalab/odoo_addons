# Odoo Module: l10n_pe_website_sale

Category: Accounting/Localizations/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import controllers
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "Peruvian eCommerce",
    "version": "0.1",
    "summary": "Be able to see Identification Type in ecommerce checkout form.",
    "category": "Accounting/Localizations/Website",
    "author": "Vauxoo, Odoo",
    "license": "LGPL-3",
    "depends": [
        "website_sale",
        "l10n_pe",
    ],
    "data": [
        "security/ir.model.access.csv",
        "data/ir_model_fields.xml",
        "views/templates.xml",
    ],
    "assets": {
        "web.assets_frontend": [
            "l10n_pe_website_sale/static/src/js/website_sale.js",
        ],
    },
    "installable": True,
    "auto_install": True,
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.http import request, route


class L10nPEWebsiteSale(WebsiteSale):

    def _get_mandatory_billing_address_fields(self, country_sudo):
        mandatory_fields = super()._get_mandatory_billing_address_fields(country_sudo)
        if request.website.sudo().company_id.country_id.code != 'PE':
            return mandatory_fields

        # For Peruvian company, the VAT is required for all the partners
        mandatory_fields.add('vat')
        if country_sudo.code == 'PE':
            mandatory_fields |= {
                'state_id', 'city_id', 'l10n_pe_district', 'l10n_latam_identification_type_id',
            }
            mandatory_fields.remove('city')
        return mandatory_fields

    def _get_mandatory_delivery_address_fields(self, country_sudo):
        mandatory_fields = super()._get_mandatory_delivery_address_fields(country_sudo)
        if request.website.sudo().company_id.country_id.code != 'PE':
            return mandatory_fields

        if country_sudo.code == 'PE':
            mandatory_fields |= {'state_id', 'city_id', 'l10n_pe_district'}
            mandatory_fields.remove('city')
        return mandatory_fields

    def _prepare_address_form_values(self, order_sudo, partner_sudo, address_type, **kwargs):
        rendering_values = super()._prepare_address_form_values(
            order_sudo, partner_sudo, address_type=address_type, **kwargs
        )
        if request.website.sudo().company_id.country_id.code != 'PE':
            return rendering_values

        if kwargs.get('use_delivery_as_billing') and address_type == 'delivery' or address_type == 'billing':
            can_edit_vat = rendering_values['can_edit_vat']
            LatamIdentificationType = request.env['l10n_latam.identification.type'].sudo()
            rendering_values.update({
                'identification_types': LatamIdentificationType.search([
                    '|', ('country_id', '=', False), ('country_id.code', '=', 'PE')
                ]) if can_edit_vat else LatamIdentificationType,
                'vat_label': request.env._("Identification Number"),
            })

        state = request.env['res.country.state'].browse(rendering_values['state_id'])
        city = partner_sudo.city_id
        ResCity = request.env['res.city'].sudo()
        District = request.env['l10n_pe.res.city.district'].sudo()
        rendering_values.update({
            'state': state,
            'state_cities': ResCity.search([('state_id', '=', state.id)]) if state else ResCity,
            'city': city,
            'city_districts': District.search([('city_id', '=', city.id)]) if city else District,
        })
        return rendering_values

    def _get_vat_validation_fields(self):
        fnames = super()._get_vat_validation_fields()
        if request.website.sudo().company_id.account_fiscal_country_id.code == 'PE':
            fnames.add('l10n_latam_identification_type_id')
            fnames.add('name')
        return fnames

    @route(
        '/shop/state_infos/<model("res.country.state"):state>',
        type='json',
        auth='public',
        methods=['POST'],
        website=True,
    )
    def state_infos(self, state, **kw):
        states = request.env['res.city'].sudo().search([('state_id', '=', state.id)])
        return {'cities': [(c.id, c.name, c.l10n_pe_code) for c in states]}

    @route(
        '/shop/city_infos/<model("res.city"):city>',
        type='json',
        auth='public',
        methods=['POST'],
        website=True,
    )
    def city_infos(self, city, **kw):
        districts = request.env['l10n_pe.res.city.district'].sudo().search([('city_id', '=', city.id)])
        return {'districts': [(d.id, d.name, d.code) for d in districts]}

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import main

```

## File: data\ir_model_fields.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <function model="ir.model.fields" name="formbuilder_whitelist">
        <value>res.partner</value>
        <value eval="[
            'city_id',
            'l10n_latam_identification_type_id',
            'l10n_pe_district',
        ]"/>
    </function>

</odoo>

```

## File: models\website.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Website(models.Model):
    _inherit = "website"

    def _display_partner_b2b_fields(self):
        """Peruvian localization must always display b2b fields"""
        self.ensure_one()
        return self.company_id.country_id.code == "PE" or super()._display_partner_b2b_fields()

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_city_public,res_city_group_public,base_address_extended.model_res_city,base.group_public,1,0,0,0
access_city_portal,res_city_group_portal,base_address_extended.model_res_city,base.group_portal,1,0,0,0

```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module **/

import websiteSaleAddress from "@website_sale/js/address";
import { rpc } from "@web/core/network/rpc";

websiteSaleAddress.include({
    events: Object.assign(
        {},
        websiteSaleAddress.prototype.events,
        {
            "change select[name='city_id']": "_onChangeCity",
        }
    ),

    start: function () {
        this._super.apply(this, arguments);

        this.elementCountry = this.addressForm.country_id;
        this.isPeruvianCompany = this.countryCode === 'PE';
        if (this.isPeruvianCompany) {
            this.elementState = this.addressForm.state_id;
            this.elementCities = this.addressForm.city_id;
            this.elementDistricts = this.addressForm.l10n_pe_district;
        }
    },

    _changeOption(selectElement, choices) {
        // empty existing options, only keep the placeholder.
        selectElement.options.length = 1;
        if (choices.length) {
            choices.forEach((item) => {
                let option = new Option(item[1], item[0]);
                option.setAttribute('data-code', item[2]);
                selectElement.appendChild(option);
            });
        }
    },

    async _onChangeState() {
        await this._super(...arguments);
        let selectedCountry = this.elementCountry.value ?
            this.elementCountry.selectedOptions[0].getAttribute('code') : '';
        if (this.isPeruvianCompany && selectedCountry === "PE") {
            const stateId = this.elementState.value;
            let choices = [];
            if (stateId)  {
                const data = await rpc(`/shop/state_infos/${stateId}`, {});
                choices = data.cities;
            }
            this._changeOption(this.elementCities, choices);
            // reset districts input as well
            this._onChangeCity();
        }
    },

    async _onChangeCity() {
        if (this.isPeruvianCompany) {
            const cityId = this.elementCities.value;
            let choices = [];
            if (cityId) {
                const data = await rpc(`/shop/city_infos/${cityId}`, {});
                choices = data.districts;
            }
            this._changeOption(this.elementDistricts, choices);
        }
    },

    async _changeCountry(init=false) {
        await this._super(...arguments);
        if (this.isPeruvianCompany) {
            let selectedCountry = this.elementCountry.value ?
                this.elementCountry.selectedOptions[0].getAttribute('code') : '';
            if (selectedCountry == 'PE') {
                let cityInput = this.addressForm.city;
                if (cityInput.value) {
                    cityInput.value = '';
                }
                this._hideInput('city');
                this._showInput('city_id');
                this._showInput('l10n_pe_district');
            } else {
                this._hideInput('city_id');
                this._hideInput('l10n_pe_district');
                this._showInput('city');
                this.elementCities.value = '';
                this.elementDistricts.value = '';
            }
        }
    },
});

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <template id="partner_info" name="Peruvian partner">
        <div class="col-xl-6 mb-3">
            <label class="col-form-label" for="l10n_latam_identification_type_id">Identification Type</label>
            <t t-if="can_edit_vat">
                <select name="l10n_latam_identification_type_id" class="form-select">
                    <option value="">Identification Type...</option>
                    <t t-foreach="identification_types" t-as="id_type">
                        <option t-att-value="id_type.id"
                            t-att-selected="id_type.id == partner_sudo.l10n_latam_identification_type_id.id">
                            <t t-out="id_type.name"/>
                        </option>
                    </t>
                </select>
            </t>
            <t t-else="">
                <p class="form-control"
                    t-out="partner_sudo.l10n_latam_identification_type_id.name"
                    readonly="1"
                    title="Changing Identification type is not allowed once document(s) have been issued for your account. Please contact us directly for this operation."/>
                <input name="l10n_latam_identification_type_id"
                    class="form-control"
                    t-att-value="partner_sudo.l10n_latam_identification_type_id.id"
                    type="hidden"/>
            </t>
        </div>
    </template>

    <template id="partner_address_info" name="Peruvian partner address">
        <div id="div_city_id" class="col-lg-6 mb-3"
            t-att-style="(country and country.code != 'PE') and 'display:none;'">
            <label class="col-form-label" for="city_id">City</label>
            <select id="city_id" name="city_id" class="form-select" data-init="1">
                <option value="">City...</option>
                <t t-foreach="state_cities" t-as="city">
                    <option t-att-value="city.id"
                        t-att-selected="city.id == partner_sudo.city_id.id">
                        <t t-out="city.name" />
                    </option>
                </t>
            </select>
        </div>

        <!-- show district -->
        <div id="div_district" class="col-lg-6 mb-3"
            t-att-style="((country and country.code != 'PE') or not city) and 'display:none;'">
            <label class="col-form-label" for="l10n_pe_district">District</label>
            <select id="l10n_pe_district" name="l10n_pe_district" class="form-select" data-init="1">
                <option value="">District...</option>
                <t t-foreach="city_districts" t-as="district">
                    <option t-att-value="district.id"
                        t-att-selected="district.id == partner_sudo.l10n_pe_district.id">
                        <t t-out="district.name" />
                    </option>
                </t>
            </select>
        </div>

    </template>

    <template id="address" inherit_id="website_sale.address">
        <div id="div_vat" position="before">
            <t t-if="(use_delivery_as_billing and address_type == 'delivery' or address_type == 'billing') and res_company.country_id.code == 'PE'">
                <div class="w-100" />
                <t t-call="l10n_pe_website_sale.partner_info" />
            </t>
        </div>
        <div id="div_state" position="after">
            <t t-if="res_company.country_id.code == 'PE'">
                <t t-call="l10n_pe_website_sale.partner_address_info" />
            </t>
        </div>
    </template>

</odoo>

```

