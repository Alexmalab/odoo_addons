# Odoo Module: website_mass_mailing_sms

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Newsletter Subscribe SMS Template',
    'summary': 'Attract visitors to subscribe to mailing lists',
    'description': """
This module adds a new template to the Newsletter Block to allow 
your visitors to subscribe with their phone number.
    """,
    'version': '1.0',
    'category': 'Website/Website',
    'depends': ['website_mass_mailing', 'mass_mailing_sms'],
    'data': [
        'views/snippets/snippets_templates.xml',
        'data/ir_model_data.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request
from odoo.addons.mass_mailing.controllers import main


class MassMailController(main.MassMailController):

    def _get_value(self, subscription_type):
        value = super(MassMailController, self)._get_value(subscription_type)
        if not value and subscription_type == 'mobile':
            if not request.env.user._is_public():
                value = request.env.user.partner_id.mobile
            elif request.session.get('mass_mailing_mobile'):
                value = request.session['mass_mailing_mobile']
        return value

    def _get_fname(self, subscription_type):
        value_field = super(MassMailController, self)._get_fname(subscription_type)
        if not value_field and subscription_type == 'mobile':
            value_field = 'mobile'
        return value_field

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\ir_model_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>mailing.contact</value>
            <value eval="[
                'mobile',
            ]"/>
        </function>
    </data>
</odoo>

```

## File: views\snippets\snippets_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_newsletter_block_sms_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-8 offset-lg-2 pt24 pb24">
            <h2>Always First.</h2>
            <p>Be the first to find out all the latest news, products, and trends.</p>
            <div class="s_newsletter_subscribe_form s_subscription_list js_subscribe" data-vxml="001" data-list-id="0" data-name="Newsletter Form">
                <div class="input-group">
                    <!-- input name must be an existing 'mailing.contact' field -->
                    <input type="tel" name="mobile" class="js_subscribe_value form-control" placeholder="e.g. +1 555-555-1234"/>
                    <a role="button" href="#" class="btn btn-primary js_subscribe_btn o_submit">Subscribe</a>
                    <a role="button" href="#" class="btn btn-success js_subscribed_btn d-none o_submit" disabled="disabled">Thanks</a>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="newsletter_subscribe_options" name="Newsletter Subscribe Options" inherit_id="website.snippet_options">
    <xpath expr="//div[@data-js='NewsletterLayout']/we-select/we-button[@data-select-data-attribute='email']" position="after">
        <we-button title="SMS Newsletter" string="SMS Subscription"
                   data-select-template="website_mass_mailing_sms.s_newsletter_block_sms_template"
                   data-select-data-attribute="sms" data-name="sms_opt"/>
    </xpath>
</template>

</odoo>

```

