# Odoo Module: l10n_in_sale

Category: Accounting/Localizations/Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Indian - Sale Report(GST)',
    'version': '1.0',
    'description': """GST Sale Report""",
    'category': 'Accounting/Localizations/Sale',
    'depends': [
        'l10n_in',
        'sale',
    ],
    'data': [
        'views/report_sale_order.xml',
        'views/sale_views.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\product_demo.xml

```xml
<odoo>
    <data noupdate="1">
        <record id="sale.advance_product_0" model="product.product">
            <field name="l10n_in_hsn_code">83030000</field>
        </record>
    </data>
</odoo>

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    l10n_in_reseller_partner_id = fields.Many2one('res.partner',
        string='Reseller', domain="[('vat', '!=', False), '|', ('company_id', '=', False), ('company_id', '=', company_id)]", readonly=False)
    l10n_in_gst_treatment = fields.Selection([
            ('regular', 'Registered Business - Regular'),
            ('composition', 'Registered Business - Composition'),
            ('unregistered', 'Unregistered Business'),
            ('consumer', 'Consumer'),
            ('overseas', 'Overseas'),
            ('special_economic_zone', 'Special Economic Zone'),
            ('deemed_export', 'Deemed Export'),
            ('uin_holders', 'UIN Holders'),
        ], string="GST Treatment", readonly=False, compute="_compute_l10n_in_gst_treatment", store=True, precompute=True)

    @api.depends('partner_id', 'partner_shipping_id', 'l10n_in_gst_treatment')
    def _compute_fiscal_position_id(self):

        def _get_fiscal_state(order, foreign_state):
            """
            Maps each order to its corresponding fiscal state based on its type,
            fiscal conditions, and the state of the associated partner or company.
            """

            if (
                order.country_code != 'IN'
                # Partner's FP takes precedence through super
                or order.partner_shipping_id.property_account_position_id
                or order.partner_id.property_account_position_id
            ):
                return False
            elif order.l10n_in_gst_treatment == 'special_economic_zone':
                # Special Economic Zone
                return foreign_state
            
            # Computing Place of Supply for particular order
            partner_state = (
                order.partner_id.commercial_partner_id == order.partner_shipping_id.commercial_partner_id
                and order.partner_shipping_id.state_id
                or order.partner_id.state_id
            )
            if not partner_state:
                partner_state = order.partner_id.commercial_partner_id.state_id or order.company_id.state_id
            if partner_state.country_id.code != 'IN':
                partner_state = foreign_state
            return partner_state

        FiscalPosition = self.env['account.fiscal.position']
        foreign_state = self.env['res.country.state'].search([('code', '!=', 'IN')], limit=1)
        for state_id, orders in self.grouped(lambda order: _get_fiscal_state(order, foreign_state)).items():
            if state_id:
                virtual_partner = self.env['res.partner'].new({
                    'state_id': state_id.id,
                    'country_id': state_id.country_id.id,
                })
                # Group orders by company to avoid multi-company conflicts
                for company_id, company_orders in orders.grouped('company_id').items():
                    company_orders.fiscal_position_id = FiscalPosition.with_company(
                        company_id.id
                    )._get_fiscal_position(virtual_partner)
            else:
                super(SaleOrder, orders)._compute_fiscal_position_id()

    @api.depends('partner_id')
    def _compute_l10n_in_gst_treatment(self):
        for order in self:
            # set default value as False so CacheMiss error never occurs for this field.
            order.l10n_in_gst_treatment = False
            if order.country_code == 'IN':
                l10n_in_gst_treatment = order.partner_id.l10n_in_gst_treatment
                if not l10n_in_gst_treatment and order.partner_id.country_id and order.partner_id.country_id.code != 'IN':
                    l10n_in_gst_treatment = 'overseas'
                if not l10n_in_gst_treatment:
                    l10n_in_gst_treatment = order.partner_id.vat and 'regular' or 'consumer'
                order.l10n_in_gst_treatment = l10n_in_gst_treatment

    def _prepare_invoice(self):
        invoice_vals = super(SaleOrder, self)._prepare_invoice()
        if self.country_code == 'IN':
            invoice_vals['l10n_in_reseller_partner_id'] = self.l10n_in_reseller_partner_id.id
            invoice_vals['l10n_in_gst_treatment'] = self.l10n_in_gst_treatment
        return invoice_vals

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order

```

## File: views\report_sale_order.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="gst_report_saleorder_document_inherit" inherit_id="sale.report_saleorder_document">
        <xpath expr="//span[@t-field='line.name']" position="after">
            <t t-if="line.product_id.l10n_in_hsn_code and line.company_id.account_fiscal_country_id.code == 'IN'">
                <h6><strong class="ml16">HSN/SAC Code:</strong> <span t-field="line.product_id.l10n_in_hsn_code"/></h6>
            </t>
        </xpath>
    </template>

</odoo>

```

## File: views\sale_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_order_form_inherit_l10n_in_sale" model="ir.ui.view">
        <field name="name">sale.order.form.inherit.l10n.in.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="l10n_in_reseller_partner_id" groups="l10n_in.group_l10n_in_reseller"
                    invisible="country_code != 'IN'"
                    readonly="state not in ['draft', 'sent']"/>
                <field name="l10n_in_gst_treatment"
                    invisible="country_code != 'IN'"
                    readonly="state not in ['draft', 'sent']"
                    required="country_code == 'IN'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\sale_make_invoice_advance.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleAdvancePaymentInv(models.TransientModel):
    _inherit = "sale.advance.payment.inv"

    def _prepare_invoice_values(self, order, so_line, accounts):
        res = super()._prepare_invoice_values(order, so_line, accounts)
        if order.country_code == 'IN':
            res['l10n_in_gst_treatment'] = order.l10n_in_gst_treatment
        if order.l10n_in_reseller_partner_id:
            res['l10n_in_reseller_partner_id'] = order.l10n_in_reseller_partner_id
        return res

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_make_invoice_advance

```

