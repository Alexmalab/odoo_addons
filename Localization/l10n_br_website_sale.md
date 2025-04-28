# Odoo Module: l10n_br_website_sale

Category: Localization

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
    'name': 'Brazil - Website Sale',
    'version': '1.0',
    'description': 'Bridge Website Sale for Brazil',
    'category': 'Localization',
    'depends': [
        'l10n_br',
        'website_sale',
    ],
    'data': [
        'views/portal.xml',
        'views/templates.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.http import request


class WebsiteSaleBr(WebsiteSale):

    def _get_mandatory_fields_billing(self, country_id=False):
        """Extend mandatory fields to add the vat in case the website and the customer are from brazil"""
        mandatory_fields = super()._get_mandatory_fields_billing(country_id)

        if request.params.get('country_id'):
            country = request.env['res.country'].browse(int(request.params['country_id']))
            if request.website.sudo().company_id.country_id.code == "BR" and country.code == "BR" and "vat" not in mandatory_fields:
                mandatory_fields += ['vat']
            # Needed because the user could put brazil and then change to another country, we don't want the field to stay mandatory
            elif 'vat' in mandatory_fields and country.code != 'BR':
                mandatory_fields.remove('vat')
        return mandatory_fields

    def values_postprocess(self, order, mode, values, errors, error_msg):
        post, errors, error_msg = super().values_postprocess(order, mode, values, errors, error_msg)
        website = request.env['website'].get_current_website()
        # This is needed so that the field is correctly write on the partner
        if values.get('l10n_latam_identification_type_id') and website.company_id.country_code == 'BR':
            post['l10n_latam_identification_type_id'] = values['l10n_latam_identification_type_id']

        return post, errors, error_msg

    def _get_country_related_render_values(self, kw, render_values):
        country_related_values = super()._get_country_related_render_values(kw, render_values)
        website = request.env['website'].get_current_website()
        if website.company_id.country_code == 'BR':
            country_related_values['identification_types'] = request.env['l10n_latam.identification.type'].search(['|', ('country_id', '=', False), ('country_id.code', '=', 'BR')])
        return country_related_values

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

## File: models\website.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Website(models.Model):
    _inherit = 'website'

    def _display_partner_b2b_fields(self):
        """ Brazil localization must always display b2b fields. """
        return self.company_id.country_id.code == 'BR' or super()._display_partner_b2b_fields()

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website

```

## File: views\portal.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="portal_my_details_fields" inherit_id="portal.portal_my_details_fields">
        <xpath expr="//input[@name='vat']/.." position="before">
            <t t-if="res_company.country_id.code == 'BR'">
                <div class="mb-1 col-xl-6"/> <!-- Empty div to put the vat and identification type on the same line -->

                <div t-attf-class="mb-1 #{error.get('l10n_ar_afip_responsibility_type_id') and 'o_has_error' or ''} col-xl-6">
                    <label class="col-form-label" for="l10n_latam_identification_type_id">Identification Type</label>
                    <t t-if="partner.can_edit_vat()">
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
        <xpath expr="//input[@name='vat']/.." position="before">
            <t t-if="mode[1] == 'billing'" positon="inside">
                <t t-if="res_company.country_id.code == 'BR'">
                    <t t-set="partner" t-value="website_sale_order.partner_id"/>
                    <div class="col-lg-6 mb-2"/> <!-- Empty div to put the vat and identification type on the same line -->

                    <div t-attf-class="mb-3 #{error.get('l10n_ar_afip_responsibility_type_id') and 'o_has_error' or ''} col-xl-6">
                        <label class="col-form-label" for="l10n_latam_identification_type_id">Identification Type</label>
                        <t t-if="partner.can_edit_vat()">
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
            </t>
        </xpath>
    </template>
</odoo>

```

