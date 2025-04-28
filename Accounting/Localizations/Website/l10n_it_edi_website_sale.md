# Odoo Module: l10n_it_edi_website_sale

Category: Accounting/Localizations/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Italy eCommerce eInvoicing",
    'version': "1.0",
    'category': 'Accounting/Localizations/Website',
    'summary': "Features for Italian eCommerce eInvoicing",
    'description': """
Contains features for Italian eCommerce eInvoicing
    """,
    'depends': ['l10n_it_edi', 'website_sale'],
    'data': [
        'views/templates.xml',
        'data/data.xml'
    ],
    'installable': True,
    'auto_install': True,
    'license': 'OEEL-1',
    'assets': {
        'web.assets_frontend': [
            '/l10n_it_edi_website_sale/static/src/js/l10n_it_edi_website_sale.js',
        ],
        'web.assets_tests': [
            'l10n_it_edi_website_sale/static/tests/**/*',
        ],
    },
}

```

## File: controllers\main.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.http import request
from odoo.exceptions import UserError
from odoo import _

class ItalyWebsiteSaleForm(WebsiteSale):
    def checkout_form_validate(self, mode, all_form_values, data):
        error, error_message = super().checkout_form_validate(mode, all_form_values, data)
        Partner = request.env['res.partner']
        if data.get('l10n_it_codice_fiscale'):
            partner_dummy = Partner.new({
                'l10n_it_codice_fiscale': data.get('l10n_it_codice_fiscale')
            })
            try:
                partner_dummy.validate_codice_fiscale()
            except UserError as e:
                error['l10n_it_codice_fiscale'] = 'error'
                error_message.append(e.name)
        pa_index = data.get('l10n_it_pa_index')
        if pa_index:
            if len(pa_index) < 6 or len(pa_index) > 7:
                error['l10n_it_pa_index'] = 'error'
                error_message.append(_('Destination Code (SDI) must have between 6 and 7 characters'))
        return error, error_message

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>res.partner</value>
            <value eval="[
                'l10n_it_codice_fiscale',
                'l10n_it_pa_index',
            ]"/>
        </function>
    </data>
</odoo>

```

## File: static\src\js\l10n_it_edi_website_sale.js

```javascript
/** @odoo-module **/
import { WebsiteSale } from "@website_sale/js/website_sale";

WebsiteSale.include({
    events: Object.assign(WebsiteSale.prototype.events, {
        "change input[name='vat'], select[name='country_id']": "computeCodiceFiscale",
    }),

    computeCodiceFiscale: function() {
        const vatValue = this.$('input[name="vat"]').val();
        const countryValue = this.$('select[name="country_id"]').find(':selected').attr('code');
        const l10nItCodiceFiscaleInput = this.$('input[name="l10n_it_codice_fiscale"]');

        if (vatValue && (vatValue.startsWith('IT') || countryValue === 'IT')) {
            if (/^IT[0-9]{11}$/.test(vatValue)) {
                l10nItCodiceFiscaleInput.val(vatValue.slice(2, 13));
            }
            else {
                l10nItCodiceFiscaleInput.val(vatValue);
            }
        }
    },
});

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="address" inherit_id="website_sale.address" name="Show l10n_it fields" customize_show="True">
            <xpath expr="//div[contains(@t-attf-class, 'div_vat')]" position="after">
                <t t-if="res_company.account_fiscal_country_id.code == 'IT'">
                    <div class="w-100"/>
                    <div t-attf-class="mb-3 #{error.get('vat') and 'o_has_error' or ''} col-lg-6 div_l10n_it_codice_fiscale mb-0" id="div_l10n_it_codice_fiscale">
                        <label class="col-form-label fw-normal label-optional" for="l10n_it_codice_fiscale">Codice Fiscale</label>
                        <input type="text" name="l10n_it_codice_fiscale" t-attf-class="form-control text-uppercase #{error.get('l10n_it_codice_fiscale') and 'is-invalid' or ''}" t-att-value="'l10n_it_codice_fiscale' in checkout and checkout['l10n_it_codice_fiscale']"/>
                    </div>
                    <div t-attf-class="mb-3 #{error.get('vat') and 'o_has_error' or ''} col-lg-6 div_l10n_it_pa_index mb-0" id="div_l10n_it_pa_index">
                        <label class="col-form-label fw-normal label-optional" for="l10n_it_pa_index">Destination Code (SDI)</label>
                        <input type="text" name="l10n_it_pa_index" t-attf-class="form-control text-uppercase #{error.get('l10n_it_pa_index') and 'is-invalid' or ''}" t-att-value="'l10n_it_pa_index' in checkout and checkout['l10n_it_pa_index']"/>
                    </div>
                </t>
            </xpath>
            <!-- Sets the country code for every country option -->
            <xpath expr="//t[@t-foreach='countries']//option" position="attributes">
                <attribute name="t-att-code">c.code</attribute>
            </xpath>
        </template>
    </data>
</odoo>

```

