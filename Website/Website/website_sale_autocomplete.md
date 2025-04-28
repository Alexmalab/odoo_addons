# Odoo Module: website_sale_autocomplete

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Google places autocompletion',
    'category': 'Website/Website',
    'summary': 'Assist your users with automatic completion & suggestions when filling their address during checkout',
    'version': '1.0',
    'description': "Assist your users with automatic completion & suggestions when filling their address during checkout",
    'depends': [
        'website_sale'
    ],
    'data': [
        'views/templates.xml',
        'views/res_config_settings_views.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_autocomplete/static/src/js/address_form.js',
            'website_sale_autocomplete/static/src/xml/autocomplete.xml',
        ],
        'web.assets_tests': [
            'website_sale_autocomplete/static/tests/**/*.js'
        ],
    },
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import requests

from odoo import http
from odoo.http import request
from odoo.tools import html2plaintext

import logging
_logger = logging.getLogger(__name__)


FIELDS_MAPPING = {
    'country': ['country'],
    'street_number': ['number'],
    'locality': ['city'],  # If locality exists, use it instead of the more general administrative area
    'route': ['street'],
    'postal_code': ['zip'],
    'administrative_area_level_1': ['state', 'city'],
    'administrative_area_level_2': ['state', 'country']
}

# If a google fields may correspond to multiple standard fields, the first occurrence in the list will overwrite following entries.
FIELDS_PRIORITY = ['country', 'street_number', 'neighborhood', 'locality', 'route', 'postal_code',
                   'administrative_area_level_1', 'administrative_area_level_2']
GOOGLE_PLACES_ENDPOINT = 'https://maps.googleapis.com/maps/api/place'
TIMEOUT = 2.5


class AutoCompleteController(http.Controller):

    def _translate_google_to_standard(self, google_fields):
        standard_data = {}

        for google_field in google_fields:
            fields_standard = FIELDS_MAPPING[google_field['type']] if google_field['type'] in FIELDS_MAPPING else []

            for field_standard in fields_standard:
                if field_standard in standard_data:  # if a value is already assigned, do not overwrite it.
                    continue
                # Convert state and countries to odoo ids
                if field_standard == 'country':
                    standard_data[field_standard] = request.env['res.country'].search(
                        [('code', '=', google_field['short_name'].upper())])[0].id
                elif field_standard == 'state':
                    state = request.env['res.country.state'].search(
                        [('code', '=', google_field['short_name'].upper()),
                         ('country_id', '=', standard_data['country'])])
                    if len(state) == 1:
                        standard_data[field_standard] = state.id
                else:
                    standard_data[field_standard] = google_field['long_name']
        return standard_data

    def _guess_number_from_input(self, source_input, standard_address):
        """
        Google might not send the house number in case the address
        does not exist in their database.
        We try to guess the number from the user's input to avoid losing the info.
        """
        # Remove other parts from address to make better guesses
        guessed_house_number = source_input \
            .replace(standard_address.get('zip', ''), '') \
            .replace(standard_address.get('street', ''), '') \
            .replace(standard_address.get('city', ''), '')
        guessed_house_number = guessed_house_number.split(',')[0].strip()
        return guessed_house_number

    def _perform_place_search(self, partial_address, api_key=None, session_id=None, language_code=None, country_code=None):
        if len(partial_address) <= 5:
            return {
                'results': [],
                'session_id': session_id
            }

        params = {
            'key': api_key,
            'fields': 'formatted_address,name',
            'inputtype': 'textquery',
            'types': 'address',
            'input': partial_address
        }
        if country_code:
            params['components'] = f'country:{country_code}'
        if language_code:
            params['language'] = language_code
        if session_id:
            params['sessiontoken'] = session_id

        try:
            results = requests.get(f'{GOOGLE_PLACES_ENDPOINT}/autocomplete/json', params=params, timeout=TIMEOUT).json()
        except (TimeoutError, ValueError) as e:
            _logger.error(e)
            return {
                'results': [],
                'session_id': session_id
            }

        if results.get('error_message'):
            _logger.error(results['error_message'])

        results = results.get('predictions', [])

        # Convert google specific format to standard format.
        return {
            'results': [{
                'formatted_address': result['description'],
                'google_place_id': result['place_id'],
            } for result in results],
            'session_id': session_id
        }

    def _perform_complete_place_search(self, address, api_key=None, google_place_id=None, language_code=None, session_id=None):
        params = {
            'key': api_key,
            'place_id': google_place_id,
            'fields': 'address_component,adr_address'
        }

        if language_code:
            params['language'] = language_code
        if session_id:
            params['sessiontoken'] = session_id

        try:
            results = requests.get(f'{GOOGLE_PLACES_ENDPOINT}/details/json', params=params, timeout=TIMEOUT).json()
        except (TimeoutError, ValueError) as e:
            _logger.error(e)
            return {'address': None}

        if results.get('error_message'):
            _logger.error(results['error_message'])

        try:
            html_address = results['result']['adr_address']
            results = results['result']['address_components']  # Get rid of useless extra data
        except KeyError:
            return {'address': None}

        # Keep only the first type from the list of types
        for res in results:
            res['type'] = res.pop('types')[0]

        # Sort the result by their priority.
        results.sort(key=lambda r: FIELDS_PRIORITY.index(r['type']) if r['type'] in FIELDS_PRIORITY else 100)

        standard_address = self._translate_google_to_standard(results)

        if 'number' not in standard_address:
            standard_address['number'] = self._guess_number_from_input(address, standard_address)
            standard_address['formatted_street_number'] = f'{standard_address["number"]} {standard_address.get("street", "")}'
        else:
            formatted_from_html = html2plaintext(html_address.split(',')[0])
            formatted_manually = f'{standard_address["number"]} {standard_address.get("street", "")}'
            # Sometimes, the google api sends back abbreviated data :
            # "52 High Road Street" becomes "52 HR St" for example. We usually take the result from google, but if it's an abbreviation, take our guess instead.
            if len(formatted_from_html) >= len(formatted_manually):
                standard_address['formatted_street_number'] = formatted_from_html
            else:
                standard_address['formatted_street_number'] = formatted_manually
        return standard_address

    @http.route('/autocomplete/address', methods=['POST'], type='json', auth='public', website=True)
    def _autocomplete_address(self, partial_address, session_id=None):
        api_key = request.env['website'].get_current_website().sudo().google_places_api_key
        return self._perform_place_search(partial_address, session_id=session_id, api_key=api_key)

    @http.route('/autocomplete/address_full', methods=['POST'], type='json', auth='public', website=True)
    def _autocomplete_address_full(self, address, session_id=None, google_place_id=None, **kwargs):
        api_key = request.env['website'].get_current_website().sudo().google_places_api_key
        return self._perform_complete_place_search(address, google_place_id=google_place_id,
                                                   session_id=session_id, api_key=api_key, **kwargs)

```

## File: controllers\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable website_sale_autocomplete
UPDATE website
SET google_places_api_key = 'dummy';
```

## File: models\res_config_settings.py

```python
from odoo import models, fields


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    google_places_api_key = fields.Char(
        string='Google Places API Key',
        related='website_id.google_places_api_key',
        readonly=False)

```

## File: models\website.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class Website(models.Model):
    _inherit = 'website'

    google_places_api_key = fields.Char(
        string='Google Places API Key',
        groups="base.group_system")

    def has_google_places_api_key(self):
        return bool(self.sudo().google_places_api_key)

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website
from . import res_config_settings

```

## File: static\src\js\address_form.js

```javascript
/** @odoo-module */

import publicWidget from '@web/legacy/js/public/public_widget';
import { KeepLast } from "@web/core/utils/concurrency";
import { debounce } from "@web/core/utils/timing";
import { renderToElement } from "@web/core/utils/render";

publicWidget.registry.AddressForm = publicWidget.Widget.extend({
    selector: '.oe_cart .checkout_autoformat:has(input[name="street"][data-autocomplete-enabled="1"])',
    events: {
        'input input[name="street"]': '_onChangeStreet',
        'click .js_autocomplete_result': '_onClickAutocompleteResult'
    },
    init: function() {
        this.streetAndNumberInput = document.querySelector('input[name="street"]');
        this.cityInput = document.querySelector('input[name="city"]');
        this.zipInput = document.querySelector('input[name="zip"]');
        this.countrySelect = document.querySelector('select[name="country_id"]');
        this.stateSelect = document.querySelector('select[name="state_id"]');
        this.keepLast = new KeepLast();
        this.sessionId = this._generateUUID();

        this._onChangeStreet = debounce(this._onChangeStreet, 200);
        this._super.apply(this, arguments);

        this.rpc = this.bindService("rpc");
    },

     /**
      * Used to generate a unique session ID for the places API.
      *
      * @private
      */
    _generateUUID: function() {
        return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function (c) {
        const r = (Math.random() * 16) | 0, v = c == "x" ? r : (r & 0x3) | 0x8;
        return v.toString(16);
        });
    },

    _hideAutocomplete: function (inputContainer) {
        const dropdown = inputContainer.querySelector('.dropdown-menu');
        if (dropdown) {
            dropdown.remove();
        }
    },

    _onChangeStreet: async function (ev) {
        const inputContainer = ev.currentTarget.parentNode;
        if (ev.currentTarget.value.length >= 5) {
            this.keepLast.add(
                this.rpc('/autocomplete/address', {
                    partial_address: ev.currentTarget.value,
                    session_id: this.sessionId || null
                })).then((response) => {
                    this._hideAutocomplete(inputContainer);
                    inputContainer.appendChild(renderToElement("website_sale_autocomplete.AutocompleteDropDown", {
                        results: response.results
                    }));
                    if (response.session_id) {
                        this.sessionId = response.session_id;
                    }
                }
            );
        } else {
            this._hideAutocomplete(inputContainer);
        }
    },

    _onClickAutocompleteResult: async function(ev) {
        const dropDown = ev.currentTarget.parentNode;

        const spinner = document.createElement('div');
        dropDown.innerText = '';
        dropDown.classList.add('d-flex', 'justify-content-center', 'align-items-center');
        spinner.classList.add('spinner-border', 'text-warning', 'text-center', 'm-auto');
        dropDown.appendChild(spinner);

        const address = await this.rpc('/autocomplete/address_full', {
            address: ev.currentTarget.innerText,
            google_place_id: ev.currentTarget.dataset.googlePlaceId,
            session_id: this.sessionId || null
        });
        if (address.formatted_street_number) {
            this.streetAndNumberInput.value = address.formatted_street_number;
        }
        // Text fields, empty if no value in order to avoid the user missing old data.
        this.zipInput.value = address.zip || '';
        this.cityInput.value = address.city || '';

        // Selects based on odoo ids
        if (address.country) {
            this.countrySelect.value = address.country;
            // Let the state select know that the country has changed so that it may fetch the correct states or disappear.
            this.countrySelect.dispatchEvent(new Event('change', {bubbles: true}));
        }
        if (address.state) {
            // Waits for the stateSelect to update before setting the state.
            new MutationObserver((entries, observer) => {
                this.stateSelect.value = address.state;
                observer.disconnect();
            }).observe(this.stateSelect, {
                childList: true, // Trigger only if the options change
            });
        }
        dropDown.remove();
    },
});

```

## File: static\src\xml\autocomplete.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<templates>
    <t t-name="website_sale_autocomplete.AutocompleteDropDown">
        <div t-attf-class="dropdown-menu position-relative #{results.length ? 'show' : ''}">
            <a class="dropdown-item js_autocomplete_result"
               t-foreach="results" t-as="result" t-key="result_index"
               t-att-data-google-place-id="result['google_place_id']">
                <t t-out="result['formatted_address']"/>
            </a>
            <img class="ms-auto pe-1" src="/website_sale_autocomplete/static/src/img/powered_by_google_on_white.png" alt="Powered by Google"/>
        </div>
    </t>
</templates>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <record id="res_config_settings_view_form_inherit_autocomplete_googleplaces" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.autocomplete.googleplaces</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='autocomplete_googleplaces_setting']" position="inside">
                <div>
                    <div class="content-group row mt16">
                        <label class="col-4" for="google_places_api_key" string="API Key"/>
                        <field class="col-6" name="google_places_api_key"/>
                    </div>
                    <div class="mt8">
                        <a target="_blank" href="https://console.cloud.google.com/getting-started">
                            <i class="oi oi-arrow-right"/>
                            Create a Google Project and get a key
                        </a>
                        <br/>
                        <a target="_blank" href="https://console.cloud.google.com/billing">
                            <i class="oi oi-arrow-right"/>
                            Enable billing on your Google Project
                        </a>
                    </div>
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
    <template id="website_sale_address_with_autocomplete" inherit_id="website_sale.address">
        <xpath expr="//input[@name='street']" position="attributes">
            <attribute name="t-att-data-autocomplete-enabled">
                1 if website.has_google_places_api_key() else 0
            </attribute>
        </xpath>
    </template>
</odoo>

```

