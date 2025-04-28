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
    'icon': '/l10n_in/static/description/icon.png',
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
            <field name="l10n_in_hsn_code">8303.00.00</field>
            <field name="l10n_in_hsn_description">Armoured or reinforced safes, strong boxes and doors and safe deposit lockers for strong rooms, cash or deed boxes and the like, of base metal
            </field>
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
        string='Reseller', domain="[('vat', '!=', False), '|', ('company_id', '=', False), ('company_id', '=', company_id)]", readonly=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]})
    l10n_in_journal_id = fields.Many2one('account.journal', string="Journal", compute="_compute_l10n_in_journal_id", store=True, readonly=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]})
    l10n_in_gst_treatment = fields.Selection([
            ('regular', 'Registered Business - Regular'),
            ('composition', 'Registered Business - Composition'),
            ('unregistered', 'Unregistered Business'),
            ('consumer', 'Consumer'),
            ('overseas', 'Overseas'),
            ('special_economic_zone', 'Special Economic Zone'),
            ('deemed_export', 'Deemed Export'),
            ('uin_holders', 'UIN Holders'),
        ], string="GST Treatment", readonly=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]}, compute="_compute_l10n_in_gst_treatment", store=True)

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

    @api.depends('company_id')
    def _compute_l10n_in_journal_id(self):
        for order in self:
            if order._origin and order._origin.company_id == order.company_id and order.l10n_in_journal_id.company_id == order.company_id:
                continue
            # set default value as False so CacheMiss error never occurs for this field.
            order.l10n_in_journal_id = False
            if order.country_code == 'IN':
                domain = [('company_id', '=', order.company_id.id), ('type', '=', 'sale')]
                journal = self.env['account.journal'].search(domain, limit=1)
                if journal:
                    order.l10n_in_journal_id = journal.id


    def _prepare_invoice(self):
        invoice_vals = super(SaleOrder, self)._prepare_invoice()
        if self.country_code == 'IN':
            invoice_vals['l10n_in_reseller_partner_id'] = self.l10n_in_reseller_partner_id.id
            if self.l10n_in_journal_id:
                invoice_vals['journal_id'] = self.l10n_in_journal_id.id
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
                <field name="country_code" invisible="1"/>
                <field name="l10n_in_reseller_partner_id" groups="l10n_in.group_l10n_in_reseller"
                    attrs="{'invisible': [('country_code','!=', 'IN')]}"/>
                <field name="l10n_in_gst_treatment"
                    attrs="{'invisible':[('country_code','!=','IN')],'required':[('country_code','=','IN')]}"/>
            </xpath>
            <xpath expr="//group[@name='sale_info']//field[@name='invoice_status']" position="after">
                <field name="l10n_in_journal_id" domain="[('company_id', '=', company_id), ('type','=','sale')]" options="{'no_create': True}" attrs="{'invisible': [('country_code','!=', 'IN')]}"/>
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

    def _prepare_invoice_values(self, order, so_line):
        res = super()._prepare_invoice_values(order, so_line)
        if order.l10n_in_journal_id:
            res['journal_id'] = order.l10n_in_journal_id.id
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

