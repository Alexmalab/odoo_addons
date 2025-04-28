# Odoo Module: auth_password_policy

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Password Policy",
    "summary": "Implement basic password policy configuration & check",
    'category': 'Hidden/Tools',
    'depends': ['base_setup', 'web'],
    'data': [
        'data/defaults.xml',
        'views/res_users.xml',
        'views/res_config_settings_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'auth_password_policy/static/src/password_meter.js',
            'auth_password_policy/static/src/password_field.js',
        ],
        'web.assets_frontend': [
            'auth_password_policy/static/src/css/password_field.css',
            'auth_password_policy/static/src/password_policy.js',
        ],
        'web.assets_common': [
            'auth_password_policy/static/src/css/password_field.css',
            'auth_password_policy/static/src/password_policy.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\defaults.xml

```xml
<odoo>
<data noupdate="1">
    <record model="ir.config_parameter" id="minlength" forcecreate="True">
        <field name="key">auth_password_policy.minlength</field>
        <field name="value" type="int">8</field>
    </record>
</data>
</odoo>

```

## File: models\res_config_settings.py

```python
from odoo import api, fields, models, _


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    minlength = fields.Integer(
        "Minimum Password Length", config_parameter="auth_password_policy.minlength", default=0,
        help="Minimum number of characters passwords must contain, set to 0 to disable.")

    @api.onchange('minlength')
    def _on_change_mins(self):
        """ Password lower bounds must be naturals
        """
        self.minlength = max(0, self.minlength or 0)

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
from odoo import api, models, _
from odoo.exceptions import UserError


class ResUsers(models.Model):
    _inherit = 'res.users'

    @api.model
    def get_password_policy(self):
        params = self.env['ir.config_parameter'].sudo()
        return {
            'minlength': int(params.get_param('auth_password_policy.minlength', default=0)),
        }

    def _set_password(self):
        self._check_password_policy(self.mapped('password'))

        super(ResUsers, self)._set_password()

    def _check_password_policy(self, passwords):
        failures = []
        params = self.env['ir.config_parameter'].sudo()

        minlength = int(params.get_param('auth_password_policy.minlength', default=0))
        for password in passwords:
            if not password:
                continue
            if len(password) < minlength:
                failures.append(_(u"Passwords must have at least %d characters, got %d.") % (minlength, len(password)))

        if failures:
            raise UserError(u'\n\n '.join(failures))

```

## File: models\__init__.py

```python
from . import res_config_settings
from . import res_users

```

## File: static\src\password_field.js

```javascript
/** @odoo-module **/

import { _lt } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { standardFieldProps } from "@web/views/fields/standard_field_props";
import { useInputField } from "@web/views/fields/input_field_hook";

import { recommendations, ConcretePolicy } from "./password_policy";
import { Meter } from "./password_meter";

const { Component, xml, onWillStart, useState } = owl;

export class PasswordField extends Component {
    setup() {
        this.state = useState({
            required: new ConcretePolicy({}),
            value: "",
        });

        useInputField({
            getValue: () => this.props.value || "",
        });

        const orm = useService("orm");
        onWillStart(async () => {
            const policy = await orm.call("res.users", "get_password_policy");
            this.state.required = new ConcretePolicy(policy);
        });
        this.recommendations = recommendations;
    }
}
PasswordField.displayName = _lt("Password");
PasswordField.supportedTypes = ["char"];
PasswordField.props = standardFieldProps;
PasswordField.components = { Meter };
PasswordField.template = xml`
<span t-if="props.readonly" t-out="props.value and '*'.repeat(props.value.length)"/>
<t t-else="">
    <input class="o_input o_field_password" type="password"
           t-att-id="props.id" t-ref="input" placeholder=" "
           t-on-input="ev => this.state.value = ev.target.value"/>
    <Meter password="state.value"
           required="state.required"
           recommended="recommendations"/>
</t>
`;

registry.category("fields").add("password_meter", PasswordField);

```

## File: static\src\password_meter.js

```javascript
/** @odoo-module **/

import { sprintf } from "@web/core/utils/strings";
import { computeScore } from "./password_policy";

const { Component, xml } = owl;

export class Meter extends Component {
    get title() {
        return sprintf(
            this.env._t(
                "Required: %s\n\nHint: to increase password strength, increase length, use multiple words, and use non-letter characters."
            ),
            String(this.props.required) || this.env._t("no requirements")
        );
    }

    get value() {
        return computeScore(this.props.password, this.props.required, this.props.recommended);
    }
}
Meter.template = xml`
<meter class="o_password_meter"
       min="0" low="0.5" high="0.99" max="1" optimum="1"
       t-att-title="title" t-att-value="value"/>
`;
Meter.props = {
    password: { type: String },
    required: Object,
    recommended: Object,
};

```

## File: static\src\password_policy.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { sprintf } from "@web/core/utils/strings";

export class Policy {
    /**
     * @param {String} password
     * @returns {number}
     */
    score(password) {}
}

export class ConcretePolicy extends Policy {
    /**
     * @param {Object} info
     * @param {Number} [info.minlength=0]
     * @param {Number} [info.minwords=0]
     * @param {Number} [info.minclasses=0]
     */
    constructor({ minlength, minwords, minclasses }) {
        super();
        this.minlength = minlength || 0;
        this.minwords = minwords || 0;
        this.minclasses = minclasses || 0;
    }
    toString() {
        const msgs = [];
        if (this.minlength > 1) {
            msgs.push(sprintf(_t("at least %s characters"), this.minlength));
        }
        if (this.minwords > 1) {
            msgs.push(sprintf(_t("at least %s words"), this.minwords));
        }
        if (this.minclasses > 1) {
            msgs.push(sprintf(_t("at least %s character classes"), this.minclasses));
        }
        return msgs.join(", ");
    }

    score(password) {
        if (!password) {
            return 0;
        }
        const lengthscore = Math.min(password.length / this.minlength, 1.0);
        // we want the number of "words". Splitting on no-words doesn't work
        // because JS will add an empty string when matching a leading or
        // trailing pattern e.g. " foo ".split(/\W+/) will return ['', 'foo', '']
        // by splitting on the words, we should always get wordscount + 1

        // \w includes _ which we don't want, so combine \W and _ then
        // invert it to know what "word" is
        //
        // Sadly JS is absolute garbage, so this splitting is basically
        // solely ascii-based unless we want to include cset
        // (http://inimino.org/~inimino/blog/javascript_cset) which can
        // generate non-trivial character-class-set-based regex patterns
        // for us. We could generate the regex statically but they're huge
        // and gnarly as hell.
        const wordCount = password.split(/[^\W_]+/).length - 1;
        const wordscore = this.minwords !== 0 ? Math.min(wordCount / this.minwords, 1.0) : 1.0;
        // See above for issues pertaining to character classification:
        // we'll classify using the ascii range because that's basically our
        // only option
        const classes =
            (/[a-z]/.test(password) ? 1 : 0) +
            (/[A-Z]/.test(password) ? 1 : 0) +
            (/\d/.test(password) ? 1 : 0) +
            (/[^A-Za-z\d]/.test(password) ? 1 : 0);
        const classesscore = Math.min(classes / this.minclasses, 1.0);

        return lengthscore * wordscore * classesscore;
    }
}

/**
 * Computes the password's score, should be roughly continuous, under 0.5
 * if the requirements don't pass and at 1 if the recommendations are
 * exceeded
 *
 * @param {String} password
 * @param {Policy} requirements
 * @param {Policy} recommendations
 */
export function computeScore(password, requirements, recommendations = recommendations) {
    const req = requirements.score(password);
    const rec = recommendations.score(password);
    return Math.pow(req, 4) * (0.5 + Math.pow(rec, 2) / 2);
}

/**
 * Recommendations from Shay (2016):
 *
 * > Our research has shown that there are other policies that are more usable
 * > and more secure. We found three policies (2class12, 3class12, and 2word16)
 * > that we can directly recommend over comp8
 *
 * Since 2class12 is a superset of 3class12 and 2word16, either pick it or
 * pick the other two (and get the highest score of the two). We're
 * picking the other two.
 *
 * @type Policy
 */
export const recommendations = {
    score(password) {
        return Math.max(...this.policies.map((p) => p.score(password)));
    },
    policies: [
        new ConcretePolicy({ minlength: 16, minwords: 2 }),
        new ConcretePolicy({ minlength: 12, minclasses: 3 }),
    ],
};

```

## File: views\res_config_settings_views.xml

```xml
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.form.auth_password_policy</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval ="20"/>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <!-- add before the Access Rights section -->
            <xpath expr="//div[@id='allow_import']" position="before">
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_left_pane"/>
                    <div class="o_setting_right_pane">
                        <label for="minlength" class="mr8"/>
                        <field name="minlength"/>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_users.xml

```xml
<odoo>
    <record id="change_password" model="ir.ui.view">
        <field name="name">Enable password meter on own password wizard</field>
        <field name="inherit_id" ref="base.change_password_own_form"/>
        <field name="model">change.password.own</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='new_password']" position="attributes">
                <attribute name="widget">password_meter</attribute>
            </xpath>
        </field>
    </record>
    <record id="change_password_multi" model="ir.ui.view">
        <field name="name">Enable password meter on multi passwords wizard</field>
        <field name="inherit_id" ref="base.change_password_wizard_user_tree_view"/>
        <field name="model">change.password.user</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='new_passwd']" position="attributes">
                <attribute name="widget">password_meter</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

