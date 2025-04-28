# Odoo Module: l10n_ec_website_sale

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
    'name': 'Ecuadorian Website',
    'version': '1.0',
    'category': 'Accounting/Localizations/Website',
    'description': """Make ecommerce work for Ecuador.""",
    'depends': [
        'website_sale',
        'l10n_ec',
    ],
    'data': [
        'data/ir_model_fields.xml',
        'data/payment_method_data.xml',

        'views/portal_templates.xml',
        'views/website_sale_templates.xml',
        'views/payment_method_views.xml',
    ],
    'demo': [
        'demo/website_demo.xml',
    ],
    'assets': {
        'web.assets_tests': [
            'l10n_ec_website_sale/static/tests/tours/*.js',
        ],
    },
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.http import request


class L10nECWebsiteSale(WebsiteSale):

    def _get_mandatory_billing_address_fields(self, country_sudo):
        mandatory_fields = super()._get_mandatory_billing_address_fields(country_sudo)
        if request.website.sudo().company_id.country_id.code != 'EC':
            return mandatory_fields

        # For Peruvian company, the VAT is required for all the partners
        mandatory_fields.add('vat')
        mandatory_fields.add('l10n_latam_identification_type_id')
        return mandatory_fields

    def _prepare_address_form_values(self, *args, address_type, use_delivery_as_billing, **kwargs):
        rendering_values = super()._prepare_address_form_values(
            *args,
            address_type=address_type,
            use_delivery_as_billing=use_delivery_as_billing,
            **kwargs,
        )
        if (
            (address_type == 'billing' or use_delivery_as_billing)
            and request.website.sudo().company_id.country_id.code == 'EC'
        ):
            can_edit_vat = rendering_values['can_edit_vat']
            LatamIdentificationType = request.env['l10n_latam.identification.type'].sudo()
            rendering_values.update({
                'identification_types': LatamIdentificationType.search([
                    '|', ('country_id', '=', False), ('country_id.code', '=', 'EC')
                ]) if can_edit_vat else LatamIdentificationType,
                'vat_label': request.env._("Identification Number"),
            })

        return rendering_values

    def _get_vat_validation_fields(self):
        fnames = super()._get_vat_validation_fields()
        if request.website.sudo().company_id.account_fiscal_country_id.code == 'EC':
            fnames.add('l10n_latam_identification_type_id')
            fnames.add('name')
        return fnames

    def _get_shop_payment_values(self, order, **kwargs):
        payment_values = super()._get_shop_payment_values(order, **kwargs)
        company = order.company_id
        # Do not show payment methods without l10n_ec_sri_payment_id.
        # Payment methods without this fields could cause issues since we require a l10n_ec_sri_payment_id to post a move.
        if company.account_fiscal_country_id.code == 'EC':
            payment_methods = payment_values['payment_methods_sudo'].filtered(lambda pm: bool(pm.l10n_ec_sri_payment_id))
            payment_values['payment_methods_sudo'] = payment_methods
        return payment_values

```

## File: controllers\portal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.portal.controllers.portal import CustomerPortal
from odoo.http import request


class CustomerPortalEcuador(CustomerPortal):

    def _is_ecuador_company(self):
        return request.env.company.country_code == 'EC'

    def _get_mandatory_fields(self):
        # EXTEND 'portal'
        mandatory_fields = super()._get_mandatory_fields()

        if self._is_ecuador_company():
            mandatory_fields.extend(('l10n_latam_identification_type_id', 'vat'))

        return mandatory_fields

    def _prepare_portal_layout_values(self):
        # EXTEND 'portal'
        portal_layout_values = super()._prepare_portal_layout_values()

        if self._is_ecuador_company():
            partner = request.env.user.partner_id
            portal_layout_values.update({
                'identification': partner.l10n_latam_identification_type_id,
                'identification_types': request.env['l10n_latam.identification.type'].search(
                    ['|', ('country_id', '=', False), ('country_id.code', '=', 'EC')]),
            })

        return portal_layout_values

    def details_form_validate(self, data, partner_creation=False):
        # EXTEND 'portal'
        error, error_message = super().details_form_validate(data, partner_creation)

        # sanitize identification value to make sure it's correctly written on the partner
        if self._is_ecuador_company() and data.get('l10n_latam_identification_type_id'):
            data['l10n_latam_identification_type_id'] = int(data['l10n_latam_identification_type_id'])

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
            'l10n_latam_identification_type_id', 'l10n_latam_identification_type_id',
        ]"/>
    </function>

</odoo>

```

## File: data\payment_method_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment.payment_method_card" model="payment.method">
        <field name="l10n_ec_sri_payment_id" eval="ref('l10n_ec.P19')"/>
    </record>

    <record id="payment.payment_method_bank_transfer" model="payment.method">
        <field name="l10n_ec_sri_payment_id" eval="ref('l10n_ec.P20')"/>
    </record>

    <record id="payment.payment_method_bank_account" model="payment.method">
        <field name="l10n_ec_sri_payment_id" eval="ref('l10n_ec.P20')"/>
    </record>

</odoo>

```

## File: models\payment_method.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class PaymentMethod(models.Model):
    _inherit = 'payment.method'

    def _get_fiscal_country_codes(self):
        return ','.join(self.env.companies.mapped('account_fiscal_country_id.code'))

    l10n_ec_sri_payment_id = fields.Many2one(
        comodel_name="l10n_ec.sri.payment",
        string="SRI Payment Method",
    )

    fiscal_country_codes = fields.Char(store=False, default=_get_fiscal_country_codes)

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _create_invoices(self, grouped=False, final=False, date=None):
        """ Create invoice(s) for the given Sales Order(s).

        :param bool grouped: if True, invoices are grouped by SO id.
            If False, invoices are grouped by keys returned by :meth:`_get_invoice_grouping_keys`
        :param bool final: if True, refunds will be generated if necessary
        :param date: unused parameter
        :returns: created invoices
        :rtype: `account.move` recordset
        :raises: UserError if one of the orders has no invoiceable lines.
        """
        moves = super()._create_invoices(grouped=grouped, final=final, date=date)
        for move in moves:
            if move.transaction_ids:
                sri_payment_methods = move.transaction_ids.mapped('payment_method_id.l10n_ec_sri_payment_id')
                if len(sri_payment_methods) == 1:
                    move.l10n_ec_sri_payment_id = sri_payment_methods
        return moves

```

## File: models\website.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Website(models.Model):
    _inherit = "website"

    def _display_partner_b2b_fields(self):
        """Ecuadorian localization must always display b2b fields"""
        self.ensure_one()
        return self.company_id.country_id.code == "EC" or super()._display_partner_b2b_fields()

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_method
from . import sale_order
from . import website

```

## File: views\payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_method_form" model="ir.ui.view">
        <field name="name">l10n_ec_website_sale.payment.method.form</field>
        <field name="model">payment.method</field>
        <field name="inherit_id" ref="payment.payment_method_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='supported_currency_ids']/.." position="after">
                <field name="fiscal_country_codes" invisible="True"/>
                <label for="l10n_ec_sri_payment_id"
                       invisible="'EC' not in fiscal_country_codes"/>
                <div invisible="'EC' not in fiscal_country_codes">
                    <field name="l10n_ec_sri_payment_id"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- FIXME JCO should be in the base loca module, it's portal content, not eCommerce -->
    <template id="partner_info" name="Ecuadorian partner">

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

    <template id="portal_my_details_fields" name="portal_my_details_fields" inherit_id="portal.portal_my_details_fields">
        <xpath expr="//input[@name='vat']/.." position="before">
            <t t-if="res_company.country_code == 'EC'">
                <div class="clearfix"/>
                <t t-call="l10n_ec_website_sale.partner_info"/>
            </t>
        </xpath>

        <label for="vat" position="replace">
            <t t-if="res_company.country_id.code != 'EC'">$0</t>
            <t t-else="">
                <label class="col-form-label label-optional" for="vat">
                    Identification Number
                </label>
            </t>
        </label>
    </template>

</odoo>

```

## File: views\website_sale_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="address" inherit_id="website_sale.address">
        <div id="div_vat" position="before">
            <t t-if="res_company.country_id.code == 'EC'">
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
    </template>

</odoo>

```

