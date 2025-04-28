# Odoo Module: l10n_in_sale

Category: Accounting/Localizations/Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
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
        'views/res_partner_views.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'installable': True,
    'application': False,
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

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountMove(models.Model):
    _inherit = "account.move"

    def _l10n_in_get_shipping_partner(self):
        shipping_partner = super()._l10n_in_get_shipping_partner()
        return self.partner_shipping_id or shipping_partner

    @api.model
    def _l10n_in_get_shipping_partner_gstin(self, shipping_partner):
        return shipping_partner.l10n_in_shipping_gstin

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_in_shipping_gstin = fields.Char("Shipping GSTIN")

    @api.constrains('l10n_in_shipping_gstin')
    def _check_l10n_in_shipping_gstin(self):
        check_vat_in = self.env['res.partner'].check_vat_in
        wrong_shipping_gstin_partner = self.filtered(lambda p: p.l10n_in_shipping_gstin and not check_vat_in(p.l10n_in_shipping_gstin))
        if wrong_shipping_gstin_partner:
            raise ValidationError(_("The shipping GSTIN number [%s] does not seem to be valid") %(",".join(p.l10n_in_shipping_gstin for p in wrong_shipping_gstin_partner)))

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
        ], string="GST Treatment", readonly=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]}, compute="_compute_l10n_in_gst_treatment", store=True)
    l10n_in_company_country_code = fields.Char(related='company_id.country_id.code', string="Country code")

    @api.depends('partner_id')
    def _compute_l10n_in_gst_treatment(self):
        for order in self:
            # set default value as False so CacheMiss error never occurs for this field.
            order.l10n_in_gst_treatment = False
            if order.l10n_in_company_country_code == 'IN':
                l10n_in_gst_treatment = order.partner_id.l10n_in_gst_treatment
                if not l10n_in_gst_treatment and order.partner_id.country_id and order.partner_id.country_id.code != 'IN':
                    l10n_in_gst_treatment = 'overseas'
                if not l10n_in_gst_treatment:
                    l10n_in_gst_treatment = order.partner_id.vat and 'regular' or 'consumer'
                order.l10n_in_gst_treatment = l10n_in_gst_treatment

    @api.depends('company_id')
    def _compute_l10n_in_journal_id(self):
        for order in self:
            # set default value as False so CacheMiss error never occurs for this field.
            order.l10n_in_journal_id = False
            if order.l10n_in_company_country_code == 'IN':
                domain = [('company_id', '=', order.company_id.id), ('type', '=', 'sale')]
                journal = self.env['account.journal'].search(domain, limit=1)
                if journal:
                    order.l10n_in_journal_id = journal.id


    def _prepare_invoice(self):
        invoice_vals = super(SaleOrder, self)._prepare_invoice()
        if self.l10n_in_company_country_code == 'IN':
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

from . import res_partner
from . import sale_order
from . import account_move

```

## File: report\account_invoice_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class L10nInAccountInvoiceReport(models.Model):
    _inherit = "l10n_in.account.invoice.report"

    def _from(self):
        from_str = super(L10nInAccountInvoiceReport, self)._from()
        return from_str.replace(
            "LEFT JOIN res_country_state ps ON ps.id = p.state_id",
            """
            LEFT JOIN res_partner dp ON dp.id = am.partner_shipping_id
            LEFT JOIN res_country_state ps ON ps.id = dp.state_id
            """
        )

    def _where(self):
        where_str = super(L10nInAccountInvoiceReport, self)._where()
        where_str += """ AND (aml.product_id IS NULL or aml.product_id != COALESCE(
            (SELECT value from ir_config_parameter where key = 'sale.default_deposit_product_id'), '0')::int)
            """
        return where_str

```

## File: report\exempted_gst_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class L10nInExemptedReport(models.Model):
    _inherit = "l10n_in.exempted.report"

    def _from(self):
        from_str = super(L10nInExemptedReport, self)._from()
        from_str += """ AND aml.product_id != COALESCE(
            (SELECT value from ir_config_parameter where key = 'sale.default_deposit_product_id'), '0')::int
            """
        return from_str

```

## File: report\hsn_gst_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class L10nInProductHsnReport(models.Model):
    _inherit = "l10n_in.product.hsn.report"

    def _from(self):
        from_str = super(L10nInProductHsnReport, self)._from()
        from_str += """ AND aml.product_id != COALESCE(
            (SELECT value from ir_config_parameter where key = 'sale.default_deposit_product_id'), '0')::int
            """
        return from_str

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_invoice_report
from . import exempted_gst_report
from . import hsn_gst_report

```

## File: views\report_sale_order.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="gst_report_saleorder_document_inherit" inherit_id="sale.report_saleorder_document">
        <xpath expr="//span[@t-field='line.name']" position="after">
            <t t-if="line.product_id.l10n_in_hsn_code"><h6><strong class="ml16">HSN/SAC Code:</strong> <span t-field="line.product_id.l10n_in_hsn_code"/></h6></t>
        </xpath>
    </template>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_in_view_partner_form" model="ir.ui.view">
        <field name="name">l10n.in.res.partner.vat.inherit</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="100"/>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_in_shipping_gstin" groups="sale.group_delivery_invoice_address" attrs="{'invisible': ['|',('type', '!=', 'delivery'),('parent_id', '=', False)]}"/>
            </xpath>
            <xpath expr="//field[@name='child_ids']/form//field[@name='comment']" position="before">
                <field name="l10n_in_shipping_gstin" groups="sale.group_delivery_invoice_address" attrs="{'invisible': [('type', '!=', 'delivery')]}"/>
            </xpath>
        </field>
    </record>
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
                <field name="l10n_in_company_country_code" invisible="1"/>
                <field name="l10n_in_reseller_partner_id" groups="l10n_in.group_l10n_in_reseller"
                    attrs="{'invisible': [('l10n_in_company_country_code','!=', 'IN')]}"/>
                <field name="l10n_in_gst_treatment"
                    attrs="{'invisible':[('l10n_in_company_country_code','!=','IN')],'required':[('l10n_in_company_country_code','=','IN')]}"/>
            </xpath>
            <xpath expr="//group[@name='sale_info']//field[@name='invoice_status']" position="after">
                <field name="l10n_in_journal_id" domain="[('company_id', '=', company_id), ('type','=','sale')]" options="{'no_create': True}" attrs="{'invisible': [('l10n_in_company_country_code','!=', 'IN')]}"/>
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

    def _prepare_invoice_values(self, order, name, amount, so_line):
        res = super()._prepare_invoice_values(order, name, amount, so_line)
        if order.l10n_in_journal_id:
            res['journal_id'] = order.l10n_in_journal_id.id
        if order.l10n_in_company_country_code == 'IN':
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

