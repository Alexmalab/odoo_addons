# Odoo Module: website_payment_paypal

Category: Accounting/Payment Providers

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
    'name': 'Website - Payment Paypal',
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 365,
    'summary': 'Website - Payment Paypal',
    'depends': ['website_payment', 'payment_paypal'],
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    paypal_email_account = fields.Char()

    @api.model
    def get_values(self):
        res = super().get_values()
        paypal = self.env.ref('payment.payment_provider_paypal', raise_if_not_found=False)
        if paypal:
            res['paypal_email_account'] = paypal.sudo().paypal_email_account
        return res

    def set_values(self):
        super().set_values()
        paypal = self.env.ref('payment.payment_provider_paypal', raise_if_not_found=False)
        if paypal and paypal.sudo().paypal_email_account != self.paypal_email_account:
            paypal.sudo().paypal_email_account = self.paypal_email_account

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--NO LONGER USED-->
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.payment.paypal</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="20"/>
        <field name="inherit_id" ref="website_payment.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="website_payment_right_pane" position="inside">
                <div class="content-group">
                    <div class="row mt8 ms-4">
                        <label class="col-lg-3" string="Email" for="paypal_email_account"/>
                        <field name="paypal_email_account"/>
                    </div>
                    <div class="mt8 text-muted">
                        After your first sale, Paypal will email you to link your Paypal account, or setup a new business account, to claim your funds, and setup payment methods.
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

