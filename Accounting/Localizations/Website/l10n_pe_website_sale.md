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
    "countries": ["pe"],
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
        'web.assets_tests': [
            'l10n_pe_website_sale/static/tests/tours/website_sale_address.js',
        ],
    },
    "installable": True,
    "auto_install": True,
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import http
from odoo.http import request

from odoo.addons.website_sale.controllers.main import WebsiteSale


class L10nPEWebsiteSale(WebsiteSale):

    def _get_mandatory_fields_billing(self, country_id=False):
        """Extend mandatory fields to add new identification, responsibility
        city_id and district fields when company is Peru"""
        res = super()._get_mandatory_fields_billing(country_id)
        if request.website.sudo().company_id.country_id.code != "PE":
            return res
        # For Peruvian company, the VAT is required for all the partners
        res.append("vat")
        if country_id == request.website.sudo().company_id.country_id.id:
            res += ["city_id", "l10n_pe_district", "l10n_latam_identification_type_id"]
            res.remove("city")
        return res

    def _get_mandatory_fields_shipping(self, country_id=False):
        """Extend mandatory fields to add city_id and district fields when the selected country is Peru"""
        res = super()._get_mandatory_fields_shipping(country_id)
        if request.website.sudo().company_id.country_id.code != "PE":
            return res
        if country_id == request.website.sudo().company_id.country_id.id:
            res += ["city_id", "l10n_pe_district"]
            res.remove("city")
        return res

    def _get_country_related_render_values(self, kw, render_values):
        res = super()._get_country_related_render_values(kw, render_values)

        if request.website.sudo().company_id.country_id.code == "PE":
            values = render_values["checkout"]
            state = "state_id" in values \
                    and values["state_id"] != "" \
                    and request.env["res.country.state"].browse(int(values["state_id"]))
            city = "city_id" in values \
                    and values["city_id"] != "" \
                    and request.env["res.city"].browse(int(values["city_id"]))
            to_include = {
                "identification": kw.get("l10n_latam_identification_type_id"),
                "identification_types": request.env["l10n_latam.identification.type"].sudo().search(
                    ["|", ("country_id", "=", False), ("country_id.code", "=", "PE")]
                ),
            }
            if state:
                to_include["state"] = state
                to_include["state_cities"] = request.env["res.city"].sudo().search([("state_id", "=", state.id)])
            if city:
                to_include["city"] = city
                to_include["city_districts"] = request.env["l10n_pe.res.city.district"].sudo().search([("city_id", "=", city.id)])
            res.update(to_include)
        return res

    def _get_vat_validation_fields(self, data):
        res = super()._get_vat_validation_fields(data)
        if request.website.sudo().company_id.account_fiscal_country_id.code == "PE":
            res.update({
                "l10n_latam_identification_type_id":
                    int(data["l10n_latam_identification_type_id"])
                    if data.get("l10n_latam_identification_type_id") else False,
                "name": data.get("name", False),
            })
        return res

    @http.route(
        ['/shop/state_infos/<model("res.country.state"):state>'],
        type="json",
        auth="public",
        methods=["POST"],
        website=True,
    )
    def state_infos(self, state, **kw):
        states = request.env["res.city"].sudo().search([("state_id", "=", state.id)])
        return {'cities': [(c.id, c.name, c.l10n_pe_code) for c in states]}

    @http.route(
        ['/shop/city_infos/<model("res.city"):city>'], type="json", auth="public", methods=["POST"], website=True
    )
    def city_infos(self, city, **kw):
        districts = request.env["l10n_pe.res.city.district"].sudo().search([("city_id", "=", city.id)])
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
import {WebsiteSale} from "@website_sale/js/website_sale";

WebsiteSale.include({
    events: Object.assign({}, WebsiteSale.prototype.events, {
        "change select[name='city_id']": "_onChangeCity",
    }),
    start: function () {
        this.elementCities = document.querySelector("select[name='city_id']");
        this.elementDistricts = document.querySelector("select[name='l10n_pe_district']");
        this.cityBlock = document.querySelector(".div_city");
        this.autoFormat = document.querySelector(".checkout_autoformat");
        this.elementState = document.querySelector("select[name='state_id']");
        this.elemenCountry = document.querySelector("select[name='country_id']");
        this.isPeruvianCompany = this.elemenCountry?.dataset.company_country_code === 'PE';
        return this._super.apply(this, arguments);
    },
    _changeOption: function (selectCheck, rpcRoute, place, selectElement) {
        if (!selectCheck) {
            return;
        }
        return this.rpc(rpcRoute, {
        }).then((data) => {
            if (this.isPeruvianCompany) {
                if (data[place]?.length) {
                    let previousValue = selectElement.value;
                    selectElement.innerHTML = "";
                    data[place].forEach((item) => {
                        let opt = document.createElement("option");
                        opt.textContent = item[1];
                        opt.value = item[0];
                        opt.setAttribute("data-code", item[2]);
                        selectElement.appendChild(opt);
                    });
                if ([...selectElement.options].some(opt => opt.value === previousValue)) {
                    selectElement.value = previousValue;
                }
                    selectElement.parentElement.style.display = "block";
                } else {
                    selectElement.value = "";
                    selectElement.parentElement.style.display = "none";
                }
            }
        });
    },
    _onChangeState: function (ev) {
        return this._super.apply(this, arguments).then(() => {
            let selectedCountry = this.elemenCountry.options[this.elemenCountry.selectedIndex].getAttribute("code");
            if (this.isPeruvianCompany && selectedCountry === "PE") {
                if (this.elementState.value === "" && this.elemenCountry.value !== '') {
                    this.elementState.options[1].selected = true;
                }
                const state = this.elementState.value;
                const rpcRoute = `/shop/state_infos/${state}`;
                return this.autoFormat.length
                    ? this._changeOption(state, rpcRoute, "cities", this.elementCities).then(() => this._onChangeCity())
                    : undefined;
            }
        });
    },
    _onChangeCity: function () {
        if (this.isPeruvianCompany) {
            const city = this.elementCities.value;
            const rpcRoute = `/shop/city_infos/${city}`;
            return this.autoFormat.length
                ? this._changeOption(city, rpcRoute, "districts", this.elementDistricts)
                : undefined;
        }
    },
    _onChangeCountry: function (ev) {
        return this._super.apply(this, arguments).then(() => {
            if (this.isPeruvianCompany) {
                let selectedCountry = ev.currentTarget.options[ev.currentTarget.selectedIndex].getAttribute("code");
                let cityInput = document.querySelector(".form-control[name='city']");
                if (selectedCountry == "PE") {
                    if (cityInput.value) {
                        cityInput.value = "";
                    }
                    this.cityBlock.classList.add("d-none");
                    return this._onChangeState().then(() => {
                        this._onChangeCity();
                    });
                } else {
                    this.cityBlock.querySelectorAll("input").forEach((input) => {
                        input.value = "";
                    });
                    this.cityBlock.classList.remove("d-none");
                    this.elementCities.value = "";
                    this.elementCities.parentElement.style.display = "none";
                    this.elementDistricts.value = "";
                    this.elementDistricts.parentElement.style.display = "none";
                }
            }
        });
    },
});

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <template id="partner_info" name="Peruvian partner">

        <!-- show identification type -->
        <div t-attf-class="mb-3 #{error.get('l10n_latam_identification_type_id') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="l10n_latam_identification_type_id">Identification Type</label>
            <t t-if="can_edit_vat">
                <select
                    name="l10n_latam_identification_type_id"
                    t-attf-class="form-select #{error.get('l10n_latam_identification_type_id') and 'is-invalid' or ''}">
                    <option value="">Identification Type...</option>
                    <t t-foreach="identification_types or []" t-as="id_type">
                        <option
                            t-att-value="id_type.id"
                            t-att-selected="id_type.id == int(identification) if identification else id_type.id == partner.l10n_latam_identification_type_id.id">
                            <t t-out="id_type.name" />
                        </option>
                    </t>
                </select>
            </t>
            <t t-else="">
                <p class="form-control"
                    t-out="partner.l10n_latam_identification_type_id.name"
                    readonly="1"
                    title="Changing Identification type is not allowed once document(s) have been issued for your account. Please contact us directly for this operation."/>
                <input name="l10n_latam_identification_type_id"
                    class="form-control"
                    t-att-value="partner.l10n_latam_identification_type_id.id"
                    type='hidden'/>
            </t>
        </div>

    </template>

    <template id="partner_address_info" name="Peruvian partner address">

        <!-- show city -->
        <div t-attf-class="mb-3 #{error.get('city_id') and 'o_has_error' or ''} col-lg-6 div_city_id"
            t-attf-style="#{(country and country.code != 'PE') and 'd-none' or ''}">
            <label class="col-form-label" for="city_id">City</label>
            <select id="city_id"
                name="city_id"
                t-attf-class="form-select #{error.get('city_id') and 'is-invalid' or ''}"
                data-init="1">
                <t t-foreach="state_cities" t-as="city">
                    <option t-att-value="city.id"
                        t-att-selected="city.id == ('city_id' in checkout and checkout['city_id'] != '' and int(checkout['city_id']))">
                        <t t-out="city.name" />
                    </option>
                </t>
            </select>
        </div>

        <!-- show district -->
        <div t-attf-class="mb-3 #{error.get('l10n_pe_district') and 'o_has_error' or ''} col-lg-6 div_district"
            t-att-style="not city and 'd-none'">
            <label class="col-form-label" for="l10n_pe_district">District</label>
            <select id="l10n_pe_district"
                name="l10n_pe_district"
                t-attf-class="form-select #{error.get('l10n_pe_district') and 'is-invalid' or ''}"
                data-init="1">
                <t t-foreach="city_districts" t-as="district">
                    <option t-att-value="district.id"
                        t-att-selected="district.id == ('l10n_pe_district' in checkout and checkout['l10n_pe_district'] != '' and int(checkout['l10n_pe_district']))">
                        <t t-out="district.name" />
                    </option>
                </t>
            </select>
        </div>

    </template>

    <template id="address" inherit_id="website_sale.address">
        <xpath expr="//input[@name='vat']/.." position="before">
            <t t-if="mode[1] == 'billing'" position="inside">
                <t t-if="res_company.country_id.code == 'PE'">
                    <t t-set="partner" t-value="website_sale_order.partner_id" />
                    <div class="w-100" />
                    <t t-call="l10n_pe_website_sale.partner_info" />
                </t>
            </t>
        </xpath>
        <label for="vat" position="replace">
            <t t-if="res_company.country_id.code != 'PE'">$0</t>
            <t t-else="">
                <label class="col-form-label label-optional" for="vat">
                    Identification Number
                </label>
            </t>
        </label>
        <xpath expr="//select[@name='country_id']" position="attributes">
            <attribute name="t-att-data-company_country_code">res_company.country_id.code</attribute>
        </xpath>
        <xpath expr="//select[@name='state_id']/.." position="after">
            <t t-if="res_company.country_id.code == 'PE'">
                <t t-call="l10n_pe_website_sale.partner_address_info" />
            </t>
        </xpath>
        <!-- Sets the country code for every country option -->
        <xpath expr="//t[@t-foreach='countries']//option" position="attributes">
            <attribute name="t-att-code">c.code</attribute>
        </xpath>
    </template>

</odoo>

```

