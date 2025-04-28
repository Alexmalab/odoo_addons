# Odoo Module: l10n_us

Category: Accounting/Localizations/Account Charts

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
    'name': 'United States - Localizations',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['us'],
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
    """,
    'depends': ['base'],
    'data': [
        'data/res_company_data.xml',
        'views/res_partner_bank_views.xml'
    ],
    'license': 'LGPL-3',
}

```

## File: data\res_company_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
<record model="res.company" id="base.main_company">
    <field name="paperformat_id" ref="base.paperformat_us"/>
</record>
</odoo>

```

## File: models\res_partner_bank.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from odoo import fields, models, api, _
from odoo.exceptions import ValidationError


class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    aba_routing = fields.Char(string="ABA/Routing", help="American Bankers Association Routing Number")

    @api.constrains('aba_routing')
    def _check_aba_routing(self):
        for bank in self:
            if bank.aba_routing and not re.match(r'^\d{1,9}$', bank.aba_routing):
                raise ValidationError(_('ABA/Routing should only contains numbers (maximum 9 digits).'))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner_bank

```

## File: views\res_partner_bank_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_bank_form_inherit_l10n_us" model="ir.ui.view">
        <field name="name">res.partner.bank.form.inherit</field>
        <field name="model">res.partner.bank</field>
        <field name="inherit_id" ref="base.view_partner_bank_form"/>
        <field name="arch" type="xml">
            <field name="bank_id" position="after">
                <field name="aba_routing" invisible="acc_type == 'iban'"/>
            </field>
        </field>
    </record>
</odoo>

```

