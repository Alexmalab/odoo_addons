# Odoo Module: l10n_my_ubl_pint

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Malaysia - UBL PINT',
    'countries': ['my'],
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'icon': '/account/static/description/l10n.png',
    'description': """
    The UBL PINT e-invoicing format for Malaysia is based on the Peppol International (PINT) model for Billing.
    """,
    'depends': ['account_edi_ubl_cii'],
    'data': [
        'views/report_invoice.xml',
        'views/res_company_view.xml',
        'views/res_partner_view.xml',
    ],
    'installable': True,
    'license': 'LGPL-3'
}

```

## File: models\account_edi_xml_pint_my.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class AccountEdiXmlUBLPINTMY(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_bis3"
    _name = 'account.edi.xml.pint_my'
    _description = "Malaysian implementation of Peppol International (PINT) model for Billing"
    """
    * PINT Official documentation: https://docs.peppol.eu/poac/pint/pint/
    * PINT MY Official documentation: https://docs.peppol.eu/poac/my/pint-my
    """

    def _export_invoice_filename(self, invoice):
        # EXTENDS account_edi_ubl_cii
        return f"{invoice.name.replace('/', '_')}_pint_my.xml"

    def _export_invoice_vals(self, invoice):
        # EXTENDS account_edi_ubl_cii
        vals = super()._export_invoice_vals(invoice)
        vals['vals'].update({
            # see https://docs.peppol.eu/poac/my/pint-my/bis/#profiles
            'customization_id': self._get_customization_ids()['pint_my'],
            'profile_id': 'urn:peppol:bis:billing',
        })
        if invoice.currency_id != invoice.company_id.currency_id:
            # see https://docs.peppol.eu/poac/my/pint-my/bis/#_tax_in_accounting_currency
            vals['vals']['tax_currency_code'] = invoice.company_id.currency_id.name  # accounting currency
        return vals

    def _get_invoice_tax_totals_vals_list(self, invoice, taxes_vals):
        # EXTENDS account_edi_ubl_cii
        vals_list = super()._get_invoice_tax_totals_vals_list(invoice, taxes_vals)
        company_currency = invoice.company_id.currency_id
        if invoice.currency_id != company_currency:
            # see https://docs.peppol.eu/poac/my/pint-my/bis/#_tax_in_accounting_currency
            vals_list.append({
                'currency': company_currency,
                'currency_dp': company_currency.decimal_places,
                'tax_amount': taxes_vals['tax_amount'],
                'tax_subtotal_vals': [],
            })
        return vals_list

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        """ [aligned-ibrp-cl-01-my]-Malaysian invoice tax categories MUST be coded using Malaysian codes. """
        # EXTENDS account_edi_ubl_cii
        tax_scheme_vals_list = super()._get_partner_party_tax_scheme_vals_list(partner, role)
        # See https://docs.peppol.eu/poac/my/pint-my/bis/#_seller_tax_identifier
        tax_scheme_vals_list[0]['company_id'] = partner.sst_registration_number or 'NA'
        if role == 'supplier':
            # TIN
            gst_tax_scheme = tax_scheme_vals_list[0].copy()
            gst_tax_scheme.update({
                'company_id': partner.vat,
                'tax_scheme_vals': {'id': 'GST'},
            })
            tax_scheme_vals_list.append(gst_tax_scheme)

        return tax_scheme_vals_list

    def _get_tax_unece_codes(self, invoice, tax):
        """
        In malaysia, only the following codes can be used: T, E, O
        https://docs.peppol.eu/poac/my/pint-my/bis/#_tax_category_code
        """
        # OVERRIDE account_edi_ubl_cii
        codes = {
            'tax_category_code': False,
            'tax_exemption_reason_code': False,
            'tax_exemption_reason': False,
        }
        # If a business is not registered for SST and/or TTx, the business is not allowed to charge sales tax,
        # service tax or tourism tax in the e-Invoice.
        # In this case, the tax category code should be 'O' (Outside scope of tax).
        # For now, we do not properly support Tourism tax (TTx) due to a lack of clarity on the subject.
        supplier = invoice.company_id.partner_id.commercial_partner_id
        if not supplier.sst_registration_number:
            codes['tax_category_code'] = 'O'
        elif tax.amount != 0:
            codes['tax_category_code'] = 'T'
        else:
            codes['tax_category_code'] = 'E'
        return codes

    def _export_invoice_constraints(self, invoice, vals):
        # EXTENDS account_edi_ubl_cii
        constraints = super()._export_invoice_constraints(invoice, vals)

        # A tax category "Outside of tax cope" can only have an amount of 0.
        for tax_total_val in vals['vals']['tax_total_vals']:
            for tax_subtotal_val in tax_total_val['tax_subtotal_vals']:
                tax_category_vals = tax_subtotal_val['tax_category_vals']
                if tax_category_vals['tax_category_code'] == 'O' and tax_category_vals['percent'] != 0:
                    constraints['peppol_my_sst_registration'] = _(
                        "If your business is registered for SST, please provide your registration number in your company details.\n"
                        "Otherwise, you are not allowed to charge sales or services taxes in the e-Invoice."
                    )
                    break

        # In malaysia, tax on good is paid at the manufacturer level. It is thus common to invoice without taxes,
        # unless invoicing for a service.
        constraints.pop('tax_on_line', '')
        constraints.pop('cen_en16931_tax_line', '')

        return constraints

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class ResCompany(models.Model):
    _inherit = 'res.company'

    sst_registration_number = fields.Char(related='partner_id.sst_registration_number', readonly=False)
    ttx_registration_number = fields.Char(related='partner_id.ttx_registration_number', readonly=False)


class BaseDocumentLayout(models.TransientModel):
    _inherit = 'base.document.layout'

    account_fiscal_country_id = fields.Many2one(related="company_id.account_fiscal_country_id")
    sst_registration_number = fields.Char(related='company_id.sst_registration_number')
    ttx_registration_number = fields.Char(related='company_id.ttx_registration_number')

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api


class ResPartner(models.Model):
    _inherit = 'res.partner'

    ubl_cii_format = fields.Selection(selection_add=[('pint_my', "PINT Malaysia")])
    sst_registration_number = fields.Char(
        string="SST",
        help="Malaysian Sales and Service Tax Number",
    )
    ttx_registration_number = fields.Char(
        string="TTx",
        help="Malaysian Tourism Tax Number",
    )

    def _get_edi_builder(self):
        # EXTENDS 'account_edi_ubl_cii'
        if self.ubl_cii_format == 'pint_my':
            return self.env['account.edi.xml.pint_my']
        return super()._get_edi_builder()

    def _compute_ubl_cii_format(self):
        # EXTENDS 'account_edi_ubl_cii'
        super()._compute_ubl_cii_format()
        for partner in self:
            if partner.country_code == 'MY':
                partner.ubl_cii_format = 'pint_my'

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['sst_registration_number', 'ttx_registration_number']

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_edi_xml_pint_my
from . import res_company
from . import res_partner

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Add the SST and TTx to the company -->
    <template id="l10n_my_ubl_pint_external_layout_standard" inherit_id="web.external_layout_standard">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <li t-if="company.sst_registration_number and company.account_fiscal_country_id.code == 'MY'">
                SST: <span t-field="company.sst_registration_number"/>
            </li>
            <li t-if="company.ttx_registration_number and company.account_fiscal_country_id.code == 'MY'">
                TTx: <span t-field="company.ttx_registration_number"/>
            </li>
        </xpath>
    </template>

    <template id="l10n_my_ubl_pint_external_layout_bold" inherit_id="web.external_layout_bold">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <li t-if="company.sst_registration_number and company.account_fiscal_country_id.code == 'MY'">
                SST: <span t-field="company.sst_registration_number"/>
            </li>
            <li t-if="company.ttx_registration_number and company.account_fiscal_country_id.code == 'MY'">
                TTx: <span t-field="company.ttx_registration_number"/>
            </li>
        </xpath>
    </template>

    <template id="l10n_my_ubl_pint_external_layout_boxed" inherit_id="web.external_layout_boxed">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <li t-if="company.sst_registration_number and company.account_fiscal_country_id.code == 'MY'">
                SST: <span t-field="company.sst_registration_number"/>
            </li>
            <li t-if="company.ttx_registration_number and company.account_fiscal_country_id.code == 'MY'">
                TTx: <span t-field="company.ttx_registration_number"/>
            </li>
        </xpath>
    </template>

    <template id="l10n_my_ubl_pint_external_layout_striped" inherit_id="web.external_layout_striped">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <li t-if="company.sst_registration_number and company.account_fiscal_country_id.code == 'MY'">
                SST: <span t-field="company.sst_registration_number"/>
            </li>
            <li t-if="company.ttx_registration_number and company.account_fiscal_country_id.code == 'MY'">
                TTx: <span t-field="company.ttx_registration_number"/>
            </li>
        </xpath>
    </template>

    <!-- add the SST to the partner -->
    <template id="l10n_my_report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='partner_vat_address_not_same_as_shipping']" position="after">
            <div t-if="o.partner_id.sst_registration_number and o.partner_id.country_code == 'MY'">
                SST: <span t-field="o.partner_id.sst_registration_number"/>
            </div>
        </xpath>

        <xpath expr="//div[@id='partner_vat_address_same_as_shipping']" position="after">
            <div t-if="o.partner_id.sst_registration_number and o.partner_id.country_code == 'MY'">
                SST: <span t-field="o.partner_id.sst_registration_number"/>
            </div>
        </xpath>

        <xpath expr="//div[@id='partner_vat_no_shipping']" position="after">
            <div t-if="o.partner_id.sst_registration_number and o.partner_id.country_code == 'MY'">
                SST: <span t-field="o.partner_id.sst_registration_number"/>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_company_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_company_form_inherit_l10n_my_ubl_pint" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_my_ubl_pint</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='currency_id']" position="after">
                <field name="sst_registration_number" invisible="country_code != 'MY'" placeholder="A01-2345-67891012"/>
                <field name="ttx_registration_number" invisible="country_code != 'MY'" placeholder="123-4567-89012345"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_my_ubl_pint" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_my_ubl_pint</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="sst_registration_number" invisible="'MY' not in fiscal_country_codes" placeholder="A01-2345-67891012" readonly="parent_id"/>
                <field name="ttx_registration_number" invisible="'MY' not in fiscal_country_codes" placeholder="123-4567-89012345" readonly="parent_id"/>
            </xpath>
        </field>
    </record>
</odoo>

```

