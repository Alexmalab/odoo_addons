# Odoo Module: l10n_it_edi_sale

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Italy - Sale E-invoicing',
    'countries': ['it'],
    'version': '1.0',
    'depends': [
        'l10n_it_edi',
        'sale',
    ],
    'description': 'Sale modifications for Italy E-invoicing',
    'category': 'Accounting/Localizations/EDI',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/italy.html',
    'data': [
        'views/sale_order_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\sale_order.py

```python
from odoo import api, fields, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    l10n_it_origin_document_type = fields.Selection(
        string="Origin Document Type",
        selection=[('purchase_order', 'Purchase Order'), ('contract', 'Contract'), ('agreement', 'Agreement')],
        copy=False,
    )
    l10n_it_origin_document_name = fields.Char(
        string="Origin Document Name",
        copy=False,
    )
    l10n_it_origin_document_date = fields.Date(
        string="Origin Document Date",
        copy=False,
    )
    l10n_it_cig = fields.Char(
        string="CIG",
        copy=False,
        help="Tender Unique Identifier",
    )
    l10n_it_cup = fields.Char(
        string="CUP",
        copy=False,
        help="Public Investment Unique Identifier",
    )
    # Technical field for showing the above fields or not
    l10n_it_partner_pa = fields.Boolean(compute='_compute_l10n_it_partner_pa')

    @api.depends('partner_id.commercial_partner_id.l10n_it_pa_index', 'company_id')
    def _compute_l10n_it_partner_pa(self):
        for order in self:
            partner = order.partner_id.commercial_partner_id
            order.l10n_it_partner_pa = partner and (partner._l10n_it_edi_is_public_administration() or len(partner.l10n_it_pa_index or '') == 7)

    def _prepare_invoice(self):
        res = super()._prepare_invoice()
        has_origin_document_fields_filled = any([
            self.l10n_it_origin_document_type,
            self.l10n_it_origin_document_name,
            self.l10n_it_origin_document_date
        ])
        has_cup_or_cig_fields_filled = self.l10n_it_cig or self.l10n_it_cup
        # If at least one of the origin_document fields is filled, we do not fill missing values with the sale order
        # values to avoid having mismatched origin_document information (e.g. user-entered doc name but SO date)
        if has_origin_document_fields_filled:
            res.update({
                "l10n_it_origin_document_type": self.l10n_it_origin_document_type,
                "l10n_it_origin_document_name": self.l10n_it_origin_document_name,
                "l10n_it_origin_document_date": self.l10n_it_origin_document_date,
                "l10n_it_cig": self.l10n_it_cig,
                "l10n_it_cup": self.l10n_it_cup,
            })
        # Otherwise, if the CUP and/or CIG are filled but origin_document fields are not, pass SO values to invoice
        elif has_cup_or_cig_fields_filled:
            res.update({
                "l10n_it_origin_document_type": "purchase_order",
                "l10n_it_origin_document_name": self.name,
                "l10n_it_origin_document_date": self.date_order,
                "l10n_it_cig": self.l10n_it_cig,
                "l10n_it_cup": self.l10n_it_cup,
            })
        return res

```

## File: models\__init__.py

```python
from . import sale_order

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_order_form_inherit_l10n_it_edi_sale" model="ir.ui.view">
        <field name="name">order.form.inherit.l10n.it.edi.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='sale_reporting']" position="after">
                <field name="l10n_it_partner_pa" invisible="1"/>
                <group name="it_edi_sale_order"
                       string="Italian Electronic Invoicing"
                       invisible="country_code != 'IT' or not l10n_it_partner_pa">
                    <field name="l10n_it_origin_document_type"/>
                    <field name="l10n_it_origin_document_name"/>
                    <field name="l10n_it_origin_document_date"/>
                    <field name="l10n_it_cig"/>
                    <field name="l10n_it_cup"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

