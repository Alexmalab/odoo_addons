# Odoo Module: l10n_in_sale

Category: Accounting/Accounting

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
    'category': 'Accounting/Accounting',
    'depends': [
        'l10n_in',
        'sale',
    ],
    'data': [
        'views/report_sale_order.xml',
        'views/sale_views.xml'
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

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    l10n_in_reseller_partner_id = fields.Many2one('res.partner',
        string='Reseller', domain="[('vat', '!=', False), '|', ('company_id', '=', False), ('company_id', '=', company_id)]", states={'posted': [('readonly', True)]})
    l10n_in_journal_id = fields.Many2one('account.journal', string="Journal", states={'posted': [('readonly', True)]})

    def _prepare_invoice(self):
        invoice_vals = super(SaleOrder, self)._prepare_invoice()
        invoice_vals['l10n_in_reseller_partner_id'] = self.l10n_in_reseller_partner_id.id
        if self.l10n_in_journal_id:
            invoice_vals['journal_id'] = self.l10n_in_journal_id.id
        return invoice_vals

    @api.onchange('company_id')
    def l10n_in_onchange_company_id(self):
        domain = [('company_id', '=', self.company_id.id), ('type', '=', 'sale')]

        journal = self.env['account.journal'].search(domain, limit=1)
        if journal:
            self.l10n_in_journal_id = journal.id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order

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
                <field name="l10n_in_reseller_partner_id" groups="l10n_in.group_l10n_in_reseller"/>
            </xpath>
            <xpath expr="//group[@name='sale_info']//field[@name='invoice_status']" position="after">
                <field name="l10n_in_journal_id" domain="[('company_id', '=', company_id), ('type','=','sale')]" options="{'no_create': True}"/>
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
        return res

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_make_invoice_advance

```

