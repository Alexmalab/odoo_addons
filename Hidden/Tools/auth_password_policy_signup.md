# Odoo Module: auth_password_policy_signup

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: controllers.py

```python
# -*- coding: utf-8 -*-

from odoo.http import request
from odoo.addons.auth_signup.controllers.main import AuthSignupHome

class AddPolicyData(AuthSignupHome):
    def get_auth_signup_config(self):
        d = super(AddPolicyData, self).get_auth_signup_config()
        d['password_minimum_length'] = request.env['ir.config_parameter'].sudo().get_param('auth_password_policy.minlength')
        return d

```

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import controllers
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Password Policy support for Signup",
    'depends': ['auth_password_policy', 'auth_signup'],
    'category': 'Hidden/Tools',
    'auto_install': True,
    'data': [
        'views/signup_templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'auth_password_policy_signup/static/src/public/**/*',
            'auth_password_policy/static/src/password_meter.js',
            'auth_password_policy/static/src/password_policy.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\ir_http.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super()._get_translation_frontend_modules_name()
        return mods + ['auth_password_policy']

```

## File: models\__init__.py

```python
from . import ir_http

```

## File: static\src\public\components\password_meter\password_meter.js

```javascript
/** @odoo-module */

import { Meter } from "@auth_password_policy/password_meter";
import { ConcretePolicy, recommendations } from "@auth_password_policy/password_policy";
import { Component, useExternalListener, useState, xml } from "@odoo/owl";
import { registry } from "@web/core/registry";

class PasswordMeter extends Component {
    static template = xml`
        <Meter t-if="hasMinlength"
            password="state.password"
            required="required"
            recommended="recommended"/>`;
    static components = { Meter };
    static props = {
        selector: String,
    };

    setup() {
        const inputEl = document.querySelector(this.props.selector);
        useExternalListener(inputEl, "input", (e) => {
            this.state.password = e.target.value || "";
        });

        const minlength = Number(inputEl.getAttribute("minlength"));
        this.hasMinlength = !isNaN(minlength);
        this.state = useState({
            password: inputEl.value || "",
        });
        this.required = new ConcretePolicy({ minlength });
        this.recommended = recommendations;
    }
}

registry.category("public_components").add("password_meter", PasswordMeter);

```

## File: static\src\public\components\password_meter\password_meter.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="auth_password_policy_signup.PasswordMeter">
        <Meter t-if="hasMinlength" password="state.password" required="required" recommended="recommended"/>
    </t>
</templates>

```

## File: views\signup_templates.xml

```xml
<odoo>
    <template id="fields" inherit_id="auth_signup.fields"
              name="Password policy data for auth_signup">
        <xpath expr="//input[@name='password']" position="after">
            <owl-component name="password_meter" props='{"selector": "input[name=password]"}'/>
        </xpath>
        <xpath expr="//input[@name='password']" position="attributes">
            <attribute name="t-att-minlength">password_minimum_length</attribute>
        </xpath>
    </template>
</odoo>

```

