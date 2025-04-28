# Odoo Module: l10n_ar_website_sale

Category: Accounting/Localizations/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import controllers
from . import models
from . import tests

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Argentinean eCommerce',
    'countries': ['ar'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Website',
    'sequence': 14,
    'author': 'Odoo S.A., ADHOC SA',
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
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _
from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.http import request


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

                if not id_type or not afip_resp:
                    # Those two values were not provided and are not required, skip the validation
                    return error, error_message

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

## File: models\sale_order.py

```python
import pytz

from odoo import fields, models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _create_account_invoices(self, invoice_vals_list, final):
        """ EXTENDS 'sale'
        Necessary because if someone creates an invoice after 9 pm Argentina time, if the invoice is created
        automatically, then it is created with the date of the next day (UTC date) instead of today.
        This fix is necessary because it causes problems validating invoices in ARCA (ex AFIP), since when generating
        the invoice with the date of the next day, no more invoices could be generated with today's date.
        We took the same approach that was used in the POS module to set the date, in this case always forcing the
        Argentina timezone """
        invoices = super()._create_account_invoices(invoice_vals_list, final)
        for invoice in invoices:
            if invoice.country_code == 'AR':
                timezone = pytz.timezone('America/Buenos_Aires')
                context_today_ar = fields.Datetime.now().astimezone(timezone).date()
                invoice.invoice_date = context_today_ar
        return invoices

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Website(models.Model):
    _inherit = "website"

    def _display_partner_b2b_fields(self):
        """ Argentinean localization must always display b2b fields """
        self.ensure_one()
        return self.company_id.country_id.code == "AR" or super()._display_partner_b2b_fields()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website
from . import sale_order

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="partner_info" name="Argentinean partner">

        <!-- show afip responsibility -->
        <div t-attf-class="mb-3 #{error.get('l10n_ar_afip_responsibility_type_id') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="l10n_ar_afip_responsibility_type_id">AFIP Responsibility</label>
            <t t-if="partner.can_edit_vat()">
                <select name="l10n_ar_afip_responsibility_type_id" t-attf-class="form-select #{error.get('l10n_ar_afip_responsibility_type_id') and 'is-invalid' or ''}">
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
        <div t-attf-class="mb-3 #{error.get('l10n_latam_identification_type_id') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="l10n_latam_identification_type_id">Identification Type</label>
            <t t-if="partner.can_edit_vat()">
                <select name="l10n_latam_identification_type_id" t-attf-class="form-select #{error.get('l10n_latam_identification_type_id') and 'is-invalid' or ''}">
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

    <template id="address" inherit_id="website_sale.address">
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

