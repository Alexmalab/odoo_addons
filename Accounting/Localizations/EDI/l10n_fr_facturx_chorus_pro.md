# Odoo Module: l10n_fr_facturx_chorus_pro

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'France - Factur-X integration with Chorus Pro',
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'description': """
Add supports to fill three optional fields used when using Chorus Pro, especially when invoicing public services.
""",
    'depends': [
        'account',
        'account_edi_facturx',
        'l10n_fr'
    ],
    'data': [
        'views/account_move_views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
from odoo import fields, models


class AccountMove(models.Model):
    _inherit = "account.move"

    buyer_reference = fields.Char(help="'Service Exécutant' in Chorus PRO.")
    contract_reference = fields.Char(help="'Numéro de Marché' in Chorus PRO.")
    purchase_order_reference = fields.Char(help="'Engagement Juridique' in Chorus PRO.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_move_form_inherit_chorus_pro" model="ir.ui.view">
            <field name="name">account.move.form.inherit.chorus.pro</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='accounting_info_group']" position="after">
                    <group name="accounting_info_group_chorus_pro" string="Chorus Pro" attrs="{'invisible': [('move_type', 'not in', ('out_invoice', 'out_refund'))]}">
                        <field name="buyer_reference"/>
                        <field name="contract_reference"/>
                        <field name="purchase_order_reference"/>
                    </group>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

