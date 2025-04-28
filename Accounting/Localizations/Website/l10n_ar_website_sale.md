# Odoo Module: l10n_ar_website_sale

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
    'name': 'Argentinean eCommerce',
    'version': '1.0',
    'category': 'Accounting/Localizations/Website',
    'sequence': 14,
    'author': 'Odoo, ADHOC SA',
    'description': """Be able to see Identification Type and AFIP Responsibility in ecommerce checkout form.""",
    'depends': [
        'website_sale',
        'l10n_ar',
    ],
    'data': [
        'data/ir_model_fields.xml',
        'views/templates.xml',
    ],
    'demo': [
        'demo/website_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'application': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _
from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.http import request, route


class L10nARWebsiteSale(WebsiteSale):

    def _get_mandatory_fields_billing(self, country_id=False):
        """Extend mandatory fields to add new identification and responsibility fields when company is argentina"""
        res = super()._get_mandatory_fields_billing(country_id)
        if request.website.sudo().company_id.country_id.code == "AR":
            res += ["l10n_latam_identification_type_id", "l10n_ar_afip_responsibility_type_id", "vat"]
        return res

    def _get_country_related_render_values(self, kw, render_values):
        res = super()._get_country_related_render_values(kw, render_values)
        if request.website.sudo().company_id.country_id.code == "AR":
            res.update({'identification': kw.get('l10n_latam_identification_type_id'),
                        'responsibility': kw.get('l10n_ar_afip_responsibility_type_id'),
                        'responsibility_types': request.env['l10n_ar.afip.responsibility.type'].search([]),
                        'identification_types': request.env['l10n_latam.identification.type'].search(
                            ['|', ('country_id', '=', False), ('country_id.code', '=', 'AR')])})
        return res

    def _get_vat_validation_fields(self, data):
        res = super()._get_vat_validation_fields(data)
        if request.website.sudo().company_id.country_id.code == "AR":
            res.update({'l10n_latam_identification_type_id': int(data['l10n_latam_identification_type_id'])
                                                             if data.get('l10n_latam_identification_type_id') else False})
            res.update({'name': data['name'] if data.get('name') else False})
        return res

    def checkout_form_validate(self, mode, all_form_values, data):
        """ We extend the method to add a new validation. If AFIP Resposibility is:

        * Final Consumer or Foreign Customer: then it can select any identification type.
        * Any other (Monotributista, RI, etc): should select always "CUIT" identification type"""
        error, error_message = super().checkout_form_validate(mode, all_form_values, data)

        # Identification type and AFIP Responsibility Combination
        if request.website.sudo().company_id.country_id.code == "AR":
            if mode[1] == 'billing':
                if error and any(field in error for field in ['l10n_latam_identification_type_id', 'l10n_ar_afip_responsibility_type_id']):
                    return error, error_message
                id_type_id = data.get("l10n_latam_identification_type_id")
                afip_resp_id = data.get("l10n_ar_afip_responsibility_type_id")

                id_type = request.env['l10n_latam.identification.type'].browse(id_type_id) if id_type_id else False
                afip_resp = request.env['l10n_ar.afip.responsibility.type'].browse(afip_resp_id) if afip_resp_id else False
                cuit_id_type = request.env.ref('l10n_ar.it_cuit')

                # Check if the AFIP responsibility is different from Final Consumer or Foreign Customer,
                # and if the identification type is different from CUIT
                if afip_resp.code not in ['5', '9'] and id_type != cuit_id_type:
                    error["l10n_latam_identification_type_id"] = 'error'
                    error_message.append(_('For the selected AFIP Responsibility you will need to set CUIT Identification Type'))

        return error, error_message

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import main

```

## File: data\ir_model_fields.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <function model="ir.model.fields" name="formbuilder_whitelist">
        <value>res.partner</value>
        <value eval="[
            'l10n_ar_afip_responsibility_type_id', 'l10n_latam_identification_type_id',
        ]"/>
    </function>

</odoo>

```

## File: models\ir_ui_view.py

```python
# -*- coding: utf-8 -*-
from odoo import _, api, models
from odoo.exceptions import ValidationError


class View(models.Model):
    _inherit = 'ir.ui.view'

    @api.constrains('active', 'key', 'website_id')
    def _check_active(self):
        for record in self:
            if record.key == 'website_sale.address_b2b' and record.website_id:
                if record.website_id.company_id.country_id.code == "AR" and not record.active:
                    raise ValidationError(_("B2B fields must always be displayed with Argentinian website."))

```

## File: models\__init__.py

```python
from . import ir_ui_view

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="partner_info" name="Argentinean partner">

        <!-- show afip responsibility -->
        <div t-attf-class="form-group #{error.get('l10n_ar_afip_responsibility_type_id') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="l10n_ar_afip_responsibility_type_id">AFIP Responsibility</label>
            <t t-if="partner.can_edit_vat()">
                <select name="l10n_ar_afip_responsibility_type_id" t-attf-class="form-control #{error.get('l10n_ar_afip_responsibility_type_id') and 'is-invalid' or ''}">
                    <option value="">AFIP Responsibility...</option>
                    <t t-foreach="responsibility_types or []" t-as="resp_type">
                        <option t-att-value="resp_type.id" t-att-selected="resp_type.id == int(responsibility) if responsibility else resp_type.id == partner.l10n_ar_afip_responsibility_type_id.id">
                            <t t-esc="resp_type.name"/>
                        </option>
                    </t>
                </select>
            </t>
            <t t-else="">
                <p class="form-control" t-esc="partner.l10n_ar_afip_responsibility_type_id.name" readonly="1" title="Changing AFIP Responsibility type is not allowed once document(s) have been issued for your account. Please contact us directly for this operation."/>
                <input name="l10n_ar_afip_responsibility_type_id" class="form-control" t-att-value="partner.l10n_ar_afip_responsibility_type_id.id" type='hidden'/>
            </t>
        </div>

        <!-- show identification type -->
        <div t-attf-class="form-group #{error.get('l10n_latam_identification_type_id') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="l10n_latam_identification_type_id">Identification Type</label>
            <t t-if="partner.can_edit_vat()">
                <select name="l10n_latam_identification_type_id" t-attf-class="form-control #{error.get('l10n_latam_identification_type_id') and 'is-invalid' or ''}">
                    <option value="">Identification Type...</option>
                    <t t-foreach="identification_types or []" t-as="id_type">
                        <option t-att-value="id_type.id" t-att-selected="id_type.id == int(identification) if identification else id_type.id == partner.l10n_latam_identification_type_id.id">
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

    </template>

    <template id="address_b2b" inherit_id="website_sale.address_b2b">
        <xpath expr="//input[@name='vat']/.." position="before">
            <t t-if="mode[1] == 'billing'" positon="inside">
                <t t-if="res_company.country_id.code == 'AR'">
                    <t t-set="partner" t-value="website_sale_order.partner_id"/>
                    <t t-call="l10n_ar_website_sale.partner_info"/>
                </t>
            </t>
        </xpath>
        <label for="vat" position="attributes">
            <attribute name="t-if">res_company.country_id.code != 'AR'</attribute>
        </label>
        <label for="vat" position="after">
            <label t-if="res_company.country_id.code == 'AR'" class="col-form-label label-optional" for="vat">Number</label>
        </label>
    </template>

</odoo>

```

