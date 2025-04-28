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

from odoo.http import request

from odoo.addons.website_sale.controllers.main import WebsiteSale


class L10nARWebsiteSale(WebsiteSale):

    def _get_mandatory_billing_address_fields(self, country_sudo):
        """Extend mandatory fields to add new identification and responsibility fields when company is argentina"""
        mandatory_fields = super()._get_mandatory_billing_address_fields(country_sudo)
        if request.website.sudo().company_id.country_id.code == 'AR':
            mandatory_fields |= {
                'l10n_latam_identification_type_id',
                'l10n_ar_afip_responsibility_type_id',
                'vat',
            }
        return mandatory_fields

    def _prepare_address_form_values(self, *args, address_type, **kwargs):
        rendering_values = super()._prepare_address_form_values(
            *args, address_type=address_type, **kwargs
        )
        if (kwargs.get('use_delivery_as_billing') and address_type == 'delivery' or address_type == 'billing') and request.website.sudo().company_id.account_fiscal_country_id.code == 'AR':
            can_edit_vat = rendering_values['can_edit_vat']
            LatamIdentificationType = request.env['l10n_latam.identification.type'].sudo()
            rendering_values.update({
                'responsibility_types': request.env['l10n_ar.afip.responsibility.type'].search([]),
                'identification_types': LatamIdentificationType.search([
                    '|', ('country_id', '=', False), ('country_id.code', '=', 'AR'),
                ]) if can_edit_vat else LatamIdentificationType,
                'vat_label': request.env._("Number"),
            })
        return rendering_values

    def _get_vat_validation_fields(self):
        fnames = super()._get_vat_validation_fields()
        if request.website.sudo().company_id.country_id.code == "AR":
            fnames.add('name')
            fnames.add('l10n_latam_identification_type_id')
        return fnames

    def _validate_address_values(self, address_values, partner_sudo, address_type, *args, **kwargs):
        """ We extend the method to add a new validation. If AFIP Resposibility is:

        * Final Consumer or Foreign Customer: then it can select any identification type.
        * Any other (Monotributista, RI, etc): should select always "CUIT" identification type
        """
        invalid_fields, missing_fields, error_messages = super()._validate_address_values(
            address_values, partner_sudo, address_type, *args, **kwargs
        )

        # Identification type and AFIP Responsibility Combination
        if address_type == 'billing' and request.website.sudo().company_id.country_id.code == 'AR':
            if missing_fields and any(
                fname in missing_fields
                for fname in [
                    'l10n_latam_identification_type_id', 'l10n_ar_afip_responsibility_type_id'
                ]
            ):
                return invalid_fields, missing_fields, error_messages

            afip_resp = request.env['l10n_ar.afip.responsibility.type'].browse(
                address_values.get('l10n_ar_afip_responsibility_type_id')
            )
            id_type = request.env['l10n_latam.identification.type'].browse(
                address_values.get('l10n_latam_identification_type_id')
            )

            if not id_type or not afip_resp:
                # Those two values were not provided and are not required, skip the validation
                return invalid_fields, missing_fields, error_messages

            # Check if the AFIP responsibility is different from Final Consumer or Foreign Customer,
            # and if the identification type is different from CUIT
            if afip_resp.code not in ['5', '9'] and id_type != request.env.ref('l10n_ar.it_cuit'):
                invalid_fields.add('l10n_latam_identification_type_id')
                error_messages.append(request.env._(
                    "For the selected AFIP Responsibility you will need to set CUIT Identification Type"))

        return invalid_fields, missing_fields, error_messages

```

## File: controllers\portal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.portal.controllers.portal import CustomerPortal
from odoo.http import request


class L10nARCustomerPortal(CustomerPortal):

    def _is_argentine_company(self):
        return request.env.company.country_code == 'AR'

    def _get_optional_fields(self):
        # EXTEND 'portal'
        optional_fields = super()._get_optional_fields()

        if self._is_argentine_company():
            optional_fields.extend(('l10n_latam_identification_type_id', 'l10n_ar_afip_responsibility_type_id', 'vat'))

        return optional_fields

    def _prepare_portal_layout_values(self):
        # EXTEND 'portal'
        portal_layout_values = super()._prepare_portal_layout_values()

        if self._is_argentine_company():
            partner = request.env.user.partner_id
            portal_layout_values.update({
                'responsibility': partner.l10n_ar_afip_responsibility_type_id,
                'identification': partner.l10n_latam_identification_type_id,
                'partner_sudo': partner,
                'responsibility_types': request.env['l10n_ar.afip.responsibility.type'].search([]),
                'identification_types': request.env['l10n_latam.identification.type'].search(
                    ['|', ('country_id', '=', False), ('country_id.code', '=', 'AR')]),
            })

        return portal_layout_values

    def details_form_validate(self, data, partner_creation=False):
        # EXTEND 'portal'
        error, error_message = super().details_form_validate(data, partner_creation)

        # sanitize identification values to make sure it's correctly written on the partner
        if self._is_argentine_company():
            for identification_field in ('l10n_latam_identification_type_id', 'l10n_ar_afip_responsibility_type_id'):
                if data.get(identification_field):
                    data[identification_field] = int(data[identification_field])

        return error, error_message

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import main
from . import portal

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
        <div class="col-xl-6 mb-3">
            <label class="col-form-label" for="l10n_ar_afip_responsibility_type_id">AFIP Responsibility</label>
            <t t-if="can_edit_vat">
                <select name="l10n_ar_afip_responsibility_type_id" class="form-select">
                    <option value="">AFIP Responsibility...</option>
                    <t t-foreach="responsibility_types" t-as="resp_type">
                        <option t-att-value="resp_type.id"
                            t-att-selected="resp_type.id == partner_sudo.l10n_ar_afip_responsibility_type_id.id">
                            <t t-out="resp_type.name"/>
                        </option>
                    </t>
                </select>
            </t>
            <t t-else="">
                <p class="form-control"
                    t-out="partner_sudo.l10n_ar_afip_responsibility_type_id.name"
                    readonly="1"
                    title="Changing AFIP Responsibility type is not allowed once document(s) have been issued for your account. Please contact us directly for this operation."/>
                <input name="l10n_ar_afip_responsibility_type_id"
                    class="form-control"
                    t-att-value="partner_sudo.l10n_ar_afip_responsibility_type_id.id"
                    type="hidden"/>
            </t>
        </div>

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

    <template id="address" inherit_id="website_sale.address">
        <div id="div_vat" position="before">
            <t t-if="(use_delivery_as_billing and address_type == 'delivery' or address_type == 'billing') and res_company.country_id.code == 'AR'">
                <t t-call="l10n_ar_website_sale.partner_info"/>
            </t>
        </div>
    </template>

    <template id="portal_my_details_fields" name="portal_my_details_fields" inherit_id="portal.portal_my_details_fields">
        <xpath expr="//input[@name='vat']/.." position="before">
            <t t-if="res_company.country_code == 'AR'">
                <t t-call="l10n_ar_website_sale.partner_info"/>
            </t>
        </xpath>

        <label for="vat" position="replace">
            <t t-if="res_company.country_id.code != 'AR'">$0</t>
            <t t-else="">
                <label class="col-form-label label-optional" for="vat">
                    Identification Number
                </label>
            </t>
        </label>
    </template>

</odoo>

```

