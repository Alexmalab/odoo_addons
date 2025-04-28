# Odoo Module: l10n_dk_bookkeeping

Category: Uncategorized

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Denmark - Bookkeeping Act',
    'version': '1.0',
    'description': """
This module contains all that is needed for the Bookkeeping Act
    """,
    'summary': "Bookkeeping Act",
    'countries': ['dk'],
    'depends': [
        'l10n_dk',
    ],
    'data': [
        'views/account_move_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_dk_currency_rate_at_transaction = fields.Float(
        string='Rate',
        compute='_compute_currency_rate_at_transaction', readonly=True, store=True,
        digits=0,
    )
    l10n_dk_show_currency_rate = fields.Boolean(compute='_compute_show_currency_rate')

    @api.depends('line_ids')
    def _compute_currency_rate_at_transaction(self):
        for record in self:
            if record.line_ids:
                record.l10n_dk_currency_rate_at_transaction = record.line_ids[0].currency_rate

    @api.depends('country_code', 'company_currency_id', 'currency_id', 'line_ids')
    def _compute_show_currency_rate(self):
        for record in self:
            record.l10n_dk_show_currency_rate = record.country_code == 'DK' and record.company_currency_id != record.currency_id and record.line_ids

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_form_l10n_dk" model="ir.ui.view">
        <field name="name">account.move.form.l10n_dk</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='journal_div']" position="after">
                <field name="l10n_dk_show_currency_rate" invisible="1"/>
                <field name="l10n_dk_currency_rate_at_transaction" invisible="not l10n_dk_show_currency_rate"/>
            </xpath>
        </field>
    </record>

    <record id="view_move_line_tree_l10n_dk" model="ir.ui.view">
        <field name="name">account.move.line.tree.l10n_dk</field>
        <field name="model">account.move.line</field>
        <field name="inherit_id" ref="account.view_move_line_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='credit']" position="after">
                <field name="currency_rate" string="Rate" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>

```

