# Odoo Module: website_payment_authorize

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
    'name': 'Website - Payment Authorize',
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 365,
    'summary': 'Website - Payment Authorize',
    'depends': ['website_payment', 'payment_authorize'],
    'data': [
        'views/res_config_settings_views.xml'
    ],
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

    authorize_capture_method = fields.Selection(
        string='Authorize.net: Payment Capture Method',
        selection=[
            ('auto', 'Automatically Capture Payment'),
            ('manual', 'Manually Charge Later'),
        ])

    @api.model
    def get_values(self):
        res = super().get_values()
        authorize = self.env.ref('payment.payment_provider_authorize').sudo()
        res['authorize_capture_method'] = 'manual' if authorize.capture_manually else 'auto'
        return res

    def set_values(self):
        super().set_values()
        authorize = self.env.ref('payment.payment_provider_authorize').sudo()
        capture_manually = self.authorize_capture_method == 'manual'
        if authorize.capture_manually != capture_manually:
            authorize.capture_manually = capture_manually

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
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.payment.authorize</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="20"/>
        <field name="inherit_id" ref="website_payment.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="website_payment" position="after">
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_right_pane">
                        <label for="authorize_capture_method"/>
                        <div class="text-muted">
                            Charge order directly or authorize at the order and capture the payment later on, manually.
                        </div>
                        <div class="content-group">
                            <div class="content-group">
                                <div class="row mt16 ms-4">
                                    <field name="authorize_capture_method" class="w-75" widget="radio"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

