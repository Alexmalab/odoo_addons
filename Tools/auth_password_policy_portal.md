# Odoo Module: auth_password_policy_portal

Category: Tools

This file contains the source code of the Odoo module.

## File: controllers.py

```python
# -*- coding: utf-8 -*-

from odoo.http import request
from odoo.addons.portal.controllers.portal import CustomerPortal

class CustomerPortalPasswordPolicy(CustomerPortal):
    def _prepare_portal_layout_values(self):
        d = super()._prepare_portal_layout_values()
        d['password_minimum_length'] = request.env['ir.config_parameter'].sudo().get_param('auth_password_policy.minlength')
        return d

```

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import controllers

```

## File: __manifest__.py

```python
{
    'name': "Password Policy support for Signup",
    'depends': ['auth_password_policy', 'portal'],
    'category': 'Tools',
    'auto_install': True,
    'data': ['views/templates.xml'],
    'assets': {
        'web.assets_frontend': [
            'auth_password_policy_portal/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: views\templates.xml

```xml
<odoo>
    <template id="portal_my_security" inherit_id="portal.portal_my_security"
              name="Password policy data for portal">
        <xpath expr="//div[hasclass('form-group')][input[@name='new1']]" position="attributes">
            <attribute name="id">new-password-group</attribute>
        </xpath>
        <!-- only put meter on first "new password" field since both must be identical -->
        <xpath expr="//input[@name='new1']" position="attributes">
            <attribute name="t-att-minlength">password_minimum_length</attribute>
        </xpath>
    </template>
</odoo>

```

