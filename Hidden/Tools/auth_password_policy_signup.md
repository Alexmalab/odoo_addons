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

## File: static\src\js\password_meter.js

```javascript
/** @odoo-module */

import { _t } from "web.core";
import Widget from "web.Widget";
import { computeScore } from "@auth_password_policy/password_policy";

export default Widget.extend({
    tagName: "meter",
    className: "o_password_meter",
    attributes: {
        min: 0,
        low: 0.5,
        high: 0.99,
        max: 1,
        length: 0,
        value: 0,
        optimum: 1,
    },
    init(parent, required, recommended) {
        this._super(parent);
        this._required = required;
        this._recommended = recommended;
    },
    start() {
        var helpMessage = _t(
            "Required: %s.\n\nHint: increase length, use multiple words and use non-letter characters to increase your password's strength."
        );
        this.el.setAttribute(
            "title",
            _.str.sprintf(helpMessage, String(this._required) || _t("no requirements"))
        );
        return this._super().then(function () {});
    },
    /**
     * Updates the meter with the information of the new password: computes
     * the (required x recommended) score and sets the widget's value as that
     *
     * @param {String} password
     */
    update(password) {
        this.el.setAttribute("length", password.length);
        this.el.value = computeScore(password, this._required, this._recommended);
    },
});

```

## File: static\src\js\signup_policy.js

```javascript
/** @odoo-module */

import "web.dom_ready";
import { ConcretePolicy, recommendations } from "@auth_password_policy/password_policy";
import PasswordMeter from "@auth_password_policy_signup/js/password_meter";

const signupForm = document.querySelector('.oe_signup_form, .oe_reset_password_form');
if (signupForm) {
    const password = document.querySelector("[type=password][minlength]");
    const minlength = password ? Number(password.getAttribute("minlength")) : NaN;
    if (!isNaN(minlength)) {
        const meter = new PasswordMeter(null, new ConcretePolicy({minlength}), recommendations);
        meter.insertAfter(password);
        password.addEventListener("input", (e) => {
            meter.update(e.target.value);
        });
    }
}

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

