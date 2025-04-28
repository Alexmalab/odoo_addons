# Odoo Module: l10n_lu_peppol_id

Category: Uncategorized

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
    'name': "Luxembourg - Peppol Identifier",
    'version': '1.0',
    'description': """
    Some Luxembourg public institutions do not have a VAT number but have been assigned an arbitrary number 
    (see: https://pch.gouvernement.lu/fr/peppol.html). Thus, this module adds the Peppol Identifier field on 
    the account.move form view. If this field is set, it is then read when exporting electronic invoicing formats.
    """,
    'depends': ['l10n_lu', 'account_edi_ubl_cii'],
    'data': [
        'views/partner_view.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    country_code = fields.Char(related='country_id.code')
    l10n_lu_peppol_identifier = fields.Char("Peppol Unique Identifier")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner

```

## File: views\partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_partner_form_inherit_l10n_lu_peppol_id" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_lu_peppol_id</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='misc']" position="inside">
                <field name="country_code" invisible="1"/>
                <field name="l10n_lu_peppol_identifier" attrs="{'invisible': [('country_code', '!=', 'LU')]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

