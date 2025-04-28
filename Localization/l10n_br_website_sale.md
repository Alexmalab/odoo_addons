# Odoo Module: l10n_br_website_sale

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

def _set_tax_included_on_website_sale(env):
    # On the installation of the module, we want every brazilian companies' websites to have the show_line_subtotals_tax_selection set to 'tax_included'
    websites = env['website'].search([('company_id', '!=', 'False')])
    for website in websites:
        if website.company_id.country_id.code == 'BR':
            website.show_line_subtotals_tax_selection = 'tax_included'

def _l10n_br_website_sale_post_init_hook(env):
    _set_tax_included_on_website_sale(env)

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Brazil - Website Sale',
    'version': '1.0',
    'description': 'Bridge Website Sale for Brazil',
    'category': 'Localization',
    'depends': [
        'l10n_br',
        'website_sale',
    ],
    'data': [
        'data/ir_model_fields.xml',

        'views/portal.xml',
        'views/templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'l10n_br_website_sale/static/src/**/*',
        ],
        'web.assets_tests': [
            'l10n_br_website_sale/static/tests/**/*',
        ]
    },
    'installable': True,
    'auto_install': True,
    'post_init_hook': '_l10n_br_website_sale_post_init_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _lt
from odoo.http import request

from odoo.addons.website_sale.controllers.main import WebsiteSale


class L10nBRWebsiteSale(WebsiteSale):

    def _get_mandatory_delivery_address_fields(self, country_sudo):
        mandatory_fields = super()._get_mandatory_delivery_address_fields(country_sudo)
        if (
            country_sudo.code == 'BR'
            and request.website.sudo().company_id.account_fiscal_country_id.code == 'BR'
        ):
            mandatory_fields |= {
                'street_name', 'street2', 'street_number', 'zip', 'city_id', 'state_id', 'country_id'
            }
            mandatory_fields -= {'street', 'city'}  # Brazil uses the base_extended_address fields added above

        return mandatory_fields

    def _get_mandatory_billing_address_fields(self, country_sudo):
        """Extend mandatory fields to add the vat in case the website and the customer are from brazil"""
        mandatory_fields = super()._get_mandatory_billing_address_fields(country_sudo)

        if (
            country_sudo.code == 'BR'
            and request.website.sudo().company_id.account_fiscal_country_id.code == 'BR'
        ):
            mandatory_fields |= {
                'vat', 'l10n_latam_identification_type_id', 'street_name', 'street2', 'street_number', 'zip', 'city_id', 'state_id', 'country_id'
            }
            mandatory_fields -= {'street', 'city'}  # Brazil uses the base_extended_address fields added above

        if 'vat' in mandatory_fields:
            mandatory_fields -= {'vat', 'l10n_latam_identification_type_id'}

        return mandatory_fields

    def _prepare_address_form_values(self, order_sudo, partner_sudo, *args, address_type, **kwargs):
        rendering_values = super()._prepare_address_form_values(
            order_sudo, partner_sudo, *args, address_type=address_type, **kwargs
        )
        if request.website.sudo().company_id.account_fiscal_country_id.code == 'BR':
            rendering_values['city_sudo'] = partner_sudo.city_id
            rendering_values['cities_sudo'] = request.env['res.city'].sudo().search([('country_id.code', '=', 'BR')])

            if (kwargs.get('use_delivery_as_billing') and address_type == 'delivery') or address_type == 'billing':
                can_edit_vat = rendering_values['can_edit_vat']
                LatamIdentificationType = request.env['l10n_latam.identification.type'].sudo()
                rendering_values.update({
                    'identification_types': LatamIdentificationType.search([
                        '|', ('country_id', '=', False), ('country_id.code', '=', 'BR'),
                    ]) if can_edit_vat else LatamIdentificationType,
                    'vat_label': _lt('Number'),
                })
        return rendering_values

```

## File: controllers\portal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.controllers.portal import CustomerPortal
from odoo.http import request

class CustomerPortalBr(CustomerPortal):

    def _get_mandatory_fields(self):
        """Extend mandatory fields to add the vat in case the website and the customer are from brazil"""
        mandatory_fields = super()._get_mandatory_fields()

        if request.params.get('country_id'):
            country = request.env['res.country'].browse(int(request.params['country_id']))
            if request.website.sudo().company_id.country_id.code == "BR" and country.code == "BR" and "vat" not in mandatory_fields:
                mandatory_fields += ['vat']

        return mandatory_fields

    def _get_optional_fields(self):
        """Extend optional fields to add the identification type to avoid having the unknown field error"""
        optional_fields = super()._get_optional_fields()
        if request.website.sudo().company_id.country_id.code == "BR" and 'l10n_latam_identification_type_id' not in optional_fields:
            optional_fields += ['l10n_latam_identification_type_id']
        return optional_fields

    def details_form_validate(self, data, partner_creation=False):
        error, error_message = super().details_form_validate(data, partner_creation)

        website = request.env['website'].get_current_website()
        # This is needed so that the field is correctly write on the partner
        if data.get('l10n_latam_identification_type_id') and website.company_id.country_code == 'BR':
            data['l10n_latam_identification_type_id'] = int(data['l10n_latam_identification_type_id'])
        return error, error_message

    def _prepare_portal_layout_values(self):
        portal_layout_values = super()._prepare_portal_layout_values()
        website = request.env['website'].get_current_website()
        if website.company_id.country_code == 'BR':
            portal_layout_values['identification_types'] = request.env['l10n_latam.identification.type'].search(['|', ('country_id', '=', False), ('country_id.code', '=', 'BR')])
        return portal_layout_values

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import portal
from . import main

```

## File: data\ir_model_fields.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <function model="ir.model.fields" name="formbuilder_whitelist">
        <value>res.partner</value>
        <value eval="[
            'l10n_latam_identification_type_id', 'city_id', 'street_name', 'street_number', 'street_number2',
        ]"/>
    </function>

</odoo>

```

## File: models\website.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Website(models.Model):
    _inherit = 'website'

    @api.model_create_multi
    def create(self, vals_list):
        for website in vals_list:
            if website.get('company_id') and self.env['res.company'].browse(website['company_id']).country_code == "BR":
                website.setdefault('show_line_subtotals_tax_selection', 'tax_included')
        return super().create(vals_list)

    def _display_partner_b2b_fields(self):
        """ Brazil localization must always display b2b fields. """
        return self.company_id.country_id.code == 'BR' or super()._display_partner_b2b_fields()

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import website

```

## File: static\src\js\address.js

```javascript
/** @odoo-module **/

import websiteSaleAddress from '@website_sale/js/address';

websiteSaleAddress.include({
    events: Object.assign(
        {},
        websiteSaleAddress.prototype.events,
        {
            'input input[name="zip"]': '_onChangeZip',
        }
    ),

    _selectState: function(id) {
        this.addressForm.querySelector(`select[name="state_id"] > option[value="${id}"]`).selected = 'selected';
    },

    _onChangeZip: function() {
        if (this.countryCode !== 'BR') {
            return;
        }

        const newZip = this.addressForm.zip.value.padEnd(5, '0');

        for (let option of this.addressForm.querySelectorAll('select[name="city_id"]:not(.d-none) > option')) {
            const ranges = option.getAttribute('zip-ranges');
            if (ranges) {
                // Parse the l10n_br_zip_ranges field (e.g. "[01000-001 05999-999] [08000-000 08499-999]").
                // Loop over each range that is enclosed in [] (e.g. "[01000-001 05999-999]" followed by "[08000-000 08499-999]").
                for (let range of ranges.matchAll(/\[[^\[]+\]/g)) {
                    // Remove square brackets (after this, range is e.g. "01000-001 05999-999")
                    range = range[0].replace(/[\[\]]/g, '');

                    let [start, end] = range.split(' ');

                    // Rely on lexicographical order to figure out if the new zip is in this range.
                    if (newZip >= start && newZip <= end) {
                        option.selected = 'selected';
                        this._selectState(option.getAttribute('state-id'));
                        return;
                    }
                }
            }
        }
    },

    _setVisibility(selector, should_show) {
        this.addressForm.querySelectorAll(selector).forEach(el => {
            if (should_show) {
                el.classList.remove('d-none');
            } else {
                el.classList.add('d-none');
            }

            // Disable hidden inputs to avoid sending back e.g. an empty street when street_name and street_number is
            // filled. It causes street_name and street_number to be lost.
            if (el.tagName === 'INPUT') {
                el.disabled = !should_show;
            }

            el.querySelectorAll('input').forEach(input => input.disabled = !should_show);
        })
    },

    async _changeCountry(ev) {
        const res = await this._super(...arguments);
        if (this.countryCode !== 'BR') {
            return res;
        }

        const countryOption = this.addressForm.country_id;
        const selectedCountryCode = countryOption.value ? countryOption.selectedOptions[0].getAttribute('code') : '';

        if (selectedCountryCode === 'BR') {
            this._setVisibility('.o_standard_address', false); // hide
            this._setVisibility('.o_extended_address', true); // show
            this._onChangeZip();
        } else {
            this._setVisibility('.o_standard_address', true); // show
            this._setVisibility('.o_extended_address', false); // hide
        }

        return res;
    }
});

```

## File: views\portal.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- FIXME JCO should be in the base loca module, it's portal content, not eCommerce -->
    <template id="portal_my_details_fields" inherit_id="portal.portal_my_details_fields">
        <xpath expr="//input[@name='vat']/.." position="before">
            <t t-if="res_company.country_id.code == 'BR'">
                <div class="mb-1 col-xl-6"/> <!-- Empty div to put the vat and identification type on the same line -->

                <div t-attf-class="mb-1 #{error.get('l10n_ar_afip_responsibility_type_id') and 'o_has_error' or ''} col-xl-6">
                    <label class="col-form-label" for="l10n_latam_identification_type_id">Identification Type</label>
                    <t t-if="partner_can_edit_vat">
                        <select name="l10n_latam_identification_type_id" t-attf-class="form-select #{error.get('l10n_latam_identification_type_id') and 'is-invalid' or ''}">
                            <option value="">Identification Type...</option>
                            <t t-foreach="identification_types or []" t-as="id_type">
                                <option t-att-value="id_type.id" t-att-selected="id_type.id == partner.l10n_latam_identification_type_id.id">
                                    <t t-esc="id_type.name"/>
                                </option>
                            </t>
                        </select>
                    </t>
                    <t t-else="">
                        <p class="form-control" t-esc="partner.l10n_latam_identification_type_id.name" readonly="1" title="Changing Identification type is not allowed once document(s) have been issued for your account. Please contact us directly for this operation."/>
                        <input name="l10n_latam_identification_type_id" class="form-control" t-att-value="partner.l10n_latam_identification_type_id.id" type='hidden'/>
                    </t>
                </div>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="address" inherit_id="website_sale.address">
        <div id="div_vat" position="before">
            <t t-if="(use_delivery_as_billing and address_type == 'delivery' or address_type == 'billing') and res_company.country_id.code == 'BR'">
                <!-- Break the line to put Identification Type and Number (vat) on the same line -->
                <div class="clearfix"/>

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
            </t>
        </div>
        <!-- o_city must remain in DOM, otherwise the standard website_sale js breaks. -->
        <input id="o_city" position="attributes">
            <attribute name="class" separator=" " add="o_standard_address"/>
        </input>
        <input id="o_city" position="after">
            <select t-if="res_company.account_fiscal_country_id.code == 'BR'" id="o_city_id" name="city_id" class="form-select o_extended_address">
                <option value="">City...</option>
                <t t-foreach="cities_sudo" t-as="c">
                    <option t-att-value="c.id" t-att-selected="c.id == city_sudo.id" t-att-code="c.id" t-att-state-id="c.state_id.id" t-att-zip-ranges="c.l10n_br_zip_ranges">
                        <t t-esc="c.name" />
                    </option>
                </t>
            </select>
        </input>
        <!-- put base_address_extended fields separately to be more user-friendly -->
        <div id="div_street" position="attributes">
            <attribute name="class" separator=" " add="o_standard_address"/>
        </div>
        <div id="div_street" position="after">
            <t t-if="res_company.account_fiscal_country_id.code == 'BR'">
                <div id="div_street_name" t-attf-class="col-lg-8 mb-2 o_extended_address">
                    <label class="col-form-label" for="o_street_name">Street</label>
                    <input id="o_street_name" type="text" name="street_name" class="form-control" t-att-value="partner_sudo.street_name"/>
                </div>
                <div id="div_street_number" t-attf-class="col-lg-4 mb-2 o_extended_address">
                    <label class="col-form-label" for="o_street_number">Street Number</label>
                    <input id="o_street_number" type="text" name="street_number" class="form-control" t-att-value="partner_sudo.street_number"/>
                </div>
                <div class="w-100"/>
                <div id="div_street_number2" t-attf-class="col-lg-6 mb-2 o_extended_address">
                    <label class="col-form-label" for="o_street_number2">Complement</label>
                    <input id="o_street_number2" type="text" name="street_number2" class="form-control" t-att-value="partner_sudo.street_number2"/>
                </div>
                <div id="div_street2" position="move"/>
            </t>
        </div>
        <!-- street2 is used for neighborhood in Brazil, change the default label -->
        <label for="o_street2" position="attributes">
            <attribute name="t-attf-class" separator=" " add="col-form-label label-optional o_standard_address"/>
        </label>
        <label for="o_street2" position="after">
            <label t-if="res_company.account_fiscal_country_id.code == 'BR'" t-attf-class="col-form-label label-optional o_extended_address" for="o_street2">Neighborhood</label>
        </label>
    </template>

    <template id="total" inherit_id="website_sale.total">
        <tr id="order_total_untaxed" position="attributes">
            <attribute name="t-attf-class" add="#{'d-none' if website.company_id.country_code == 'BR' else ''}" separator=" "/>
        </tr>
        <tr id="order_total_taxes" position="attributes">
            <attribute name="t-attf-class" add="#{'d-none' if website.company_id.country_code == 'BR' else ''}" separator=" "/>
        </tr>
        <tr id="order_total" position="attributes">
            <attribute name="t-attf-class" add="#{'border-top' if website.company_id.country_code != 'BR' else ''}" separator=" "/>
        </tr>
    </template>
</odoo>

```

