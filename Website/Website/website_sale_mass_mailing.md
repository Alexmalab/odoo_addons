# Odoo Module: website_sale_mass_mailing

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Checkout Newsletter",
    'summary': "Let new customers sign up for a newsletter during checkout",
    'description': """
        Allows anonymous shoppers of your eCommerce to sign up for a newsletter during the checkout
        process.
    """,
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale', 'website_mass_mailing'],
    'data': [
        'views/res_config_settings_views.xml',
        'views/templates.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request

from odoo.addons.website_mass_mailing.controllers.main import MassMailController
from odoo.addons.website_sale.controllers.main import WebsiteSale as WebsiteSaleController


class WebsiteSale(WebsiteSaleController):

    def _handle_extra_form_data(self, extra_form_data, address_values):
        super()._handle_extra_form_data(extra_form_data, address_values)
        if extra_form_data.get('newsletter') and address_values.get('email'):
            MassMailController.subscribe_to_newsletter(
                subscription_type='email',
                value=address_values['email'],
                list_id=request.website.newsletter_id,
                fname='email',
                address_name=address_values['name'],
            )

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    is_newsletter_enabled = fields.Boolean()
    newsletter_id = fields.Many2one(related='website_id.newsletter_id', readonly=False)

    # === CRUD METHODS ===#

    @api.model
    def get_values(self):
        res = super().get_values()
        res['is_newsletter_enabled'] = self.env.ref('website_sale_mass_mailing.newsletter').active
        return res

    def set_values(self):
        super().set_values()
        newsletter_view = self.env.ref('website_sale_mass_mailing.newsletter')
        if newsletter_view.active != self.is_newsletter_enabled:
            newsletter_view.active = self.is_newsletter_enabled

```

## File: models\website.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Website(models.Model):
    _inherit = 'website'

    newsletter_id = fields.Many2one(string="Newsletter List", comodel_name='mailing.list')

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import website

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale.mass.mailing</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="website_checkout_registration" position="after">
                <setting
                    id="newsletter_address"
                    string="Newsletter"
                    help="Show a checkbox to sign up for the selected newsletter to guest users"
                    groups="mass_mailing.group_mass_mailing_user"
                >
                    <field name="is_newsletter_enabled"/>
                    <div class="content-group" invisible="not is_newsletter_enabled">
                        <label for="newsletter_id" class="o_light_label me-2"/>
                        <field
                            name="newsletter_id"
                            class="oe_inline"
                            required="is_newsletter_enabled"
                        />
                    </div>
                </setting>
            </setting>
        </field>
    </record>

</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template
        id="website_sale_mass_mailing.newsletter"
        inherit_id="website_sale.address"
        name="Newsletter"
        active="False"
    >
        <div id="div_email_public" position="after">
            <div class="form-check mt-2">
                <label>
                    <input
                        type="checkbox"
                        name="newsletter"
                        class="form-check-input"
                    /> Be the first to find out all the latest news, products and trends
                </label>
            </div>
        </div>
    </template>

</odoo>

```

