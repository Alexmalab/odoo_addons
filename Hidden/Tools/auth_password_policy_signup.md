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
            'auth_password_policy_signup/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: static\src\js\signup_policy.js

```javascript
odoo.define('auth_password_policy_signup.policy', function (require) {
"use strict";

require('web.dom_ready');
var policy = require('auth_password_policy');
var PasswordMeter = require('auth_password_policy.Meter');

var $signupForm = $('.oe_signup_form, .oe_reset_password_form');
if (!$signupForm.length) { return; }

// hook in password strength meter
// * requirement is the password field's minlength
// * recommendations are from the module
var $password = $('[type=password][minlength]');
var minlength = Number($password.attr('minlength'));
if (isNaN(minlength)) { return; }

var meter = new PasswordMeter(null, new policy.Policy({minlength: minlength}), policy.recommendations);
meter.insertAfter($password);
$password.on('input', function () {
    meter.update($password.val());
});
});

```

## File: views\signup_templates.xml

```xml
<odoo>
    <template id="fields" inherit_id="auth_signup.fields"
              name="Password policy data for auth_signup">
        <xpath expr="//input[@name='password']" position="attributes">
            <attribute name="t-att-minlength">password_minimum_length</attribute>
        </xpath>
    </template>
</odoo>

```

