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
        'views/assets.xml',
        'views/res_users.xml',
        'views/res_config_settings_views.xml',
    ],
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

    minlength = fields.Integer("Minimum Password Length", help="Minimum number of characters passwords must contain, set to 0 to disable.")

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()

        res['minlength'] = int(self.env['ir.config_parameter'].sudo().get_param('auth_password_policy.minlength', default=0))

        return res

    @api.model
    def set_values(self):
        self.env['ir.config_parameter'].sudo().set_param('auth_password_policy.minlength', self.minlength)

        super(ResConfigSettings, self).set_values()

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

## File: static\src\js\change_password.js

```javascript
odoo.define('auth_password_policy.ChangePassword', function (require) {
"use strict";
var ChangePassword = require('web.ChangePassword');
var policy = require('auth_password_policy');
var Meter = require('auth_password_policy.Meter');

ChangePassword.include({
    events: {
        'input input[name=new_password]': function (e) {
            this._meter.update(e.target.value);
        }
    },
    willStart: function () {
        var _this = this;
        var getPolicy = this._rpc({
            model: 'res.users',
            method: 'get_password_policy'
        }).then(function (p) {
            _this._meter = new Meter(_this, new policy.Policy(p), policy.recommendations);
        });
        return Promise.all([
            this._super.apply(this, arguments),
            getPolicy
        ]);
    },
    start: function () {
        return Promise.all([
            this._meter.insertAfter(this.$('input[name=new_password]')),
            this._super()
        ]);
    }
})
});

```

## File: static\src\js\password_field.js

```javascript
/**
 * Defines a proper password field (rather than just an InputField option) to
 * provide a "password strength" meter based on the database's current
 * policy & the 2word16 password policy recommended by Shay (2016) "Designing
 * Password Policies for Strength and Usability".
 */
odoo.define('auth_password_policy.PasswordField', function (require) {
"use strict";
var fields = require('web.basic_fields');
var registry = require('web.field_registry');
var policy = require('auth_password_policy');
var Meter = require('auth_password_policy.Meter');
var _formatValue = require('web.AbstractField').prototype._formatValue;

var PasswordField = fields.InputField.extend({
    className: 'o_field_password',

    init: function () {
        this._super.apply(this, arguments);
        this.nodeOptions.isPassword = true;
        this._meter = new Meter(this, new policy.Policy({}), policy.recommendations);
    },
    willStart: function () {
        var _this = this;
        var getPolicy = this._rpc({
            model: 'res.users',
            method: 'get_password_policy',
        }).then(function (p) {
            _this._meter = new Meter(_this, new policy.Policy(p), policy.recommendations);
        });
        return Promise.all([
            this._super.apply(this, arguments),
            getPolicy
        ]);
    },
    /**
     * Add a <meter> next to the input (TODO: move to template?)
     *
     * @override
     * @private
     */
    _renderEdit: function () {
        var _this = this;
        var meter = this._meter;
        return Promise.resolve(this._super.apply(this, arguments)).then(function () {
            return meter._widgetRenderAndInsert(function (t) {
                // insertAfter doesn't work and appendTo means the meter is
                // ignored (as this.$el is an input[type=password])
                _this.$el = t.add(meter.$el);
            }, _this.$el);
        }).then(function () {
            // initial meter update when re-editing
            meter.update(_this._getValue());
        });
    },
    /**
     * disable formatting for this widget, or the value gets replaced by
     * **** before being written back into the widget when switching from
     * readonly to editable (so input -> readonly -> input more, the 1st input
     * is all replaced by *s not just in display but in actual storage)
     *
     * @override
     * @private
     */
    _formatValue: function (value) { return value || ''; },
    /**
     * @override
     * @private
     */
    _renderReadonly: function () {
        this.$el.text(_formatValue.call(this, this.value));
    },
    /**
     * Update meter value on the fly on value change
     *
     * @override
     * @private
     */
    _onInput: function () {
        this._super();
        this._meter.update(this._getValue());
    }
});

registry.add("password_meter", PasswordField);
return PasswordField;
});

```

## File: static\src\js\password_gauge.js

```javascript
odoo.define('auth_password_policy', function (require) {
"use strict";
var core = require('web.core');
var _t = core._t;

var Policy = core.Class.extend({
    /**
     *
     * @param {Object} info
     * @param {Number} [info.minlength=0]
     * @param {Number} [info.minwords=0]
     * @param {Number} [info.minclasses=0]
     */
    init: function (info) {
        this._minlength = info.minlength || 1;
        this._minwords = info.minwords || 1;
        this._minclasses = info.minclasses || 1;
    },
    toString: function () {
        var msgs = [];
        if (this._minlength > 1) {
            msgs.push(_.str.sprintf(_t("at least %d characters"), this._minlength));
        }
        if (this._minwords > 1) {
            msgs.push(_.str.sprintf(_t("at least %d words"), this._minwords));
        }
        if (this._minclasses > 1) {
            msgs.push(_.str.sprintf(_t("at least %d character classes"), this._minclasses));
        }
        return msgs.join(', ')
    },
    score: function (password) {
        var lengthscore = Math.min(
            password.length / this._minlength,
            1.0);
        // we want the number of "words". Splitting on no-words doesn't work
        // because JS will add an empty string when matching a leading or
        // trailing pattern e.g. " foo ".split(/\W+/) will return ['', 'foo', '']
        // by splitting on the words, we should always get wordscount + 1
        var wordscore =  Math.min(
            // \w includes _ which we don't want, so combine \W and _ then
            // invert it to know what "word" is
            //
            // Sadly JS is absolute garbage, so this splitting is basically
            // solely ascii-based unless we want to include cset
            // (http://inimino.org/~inimino/blog/javascript_cset) which can
            // generate non-trivial character-class-set-based regex patterns
            // for us. We could generate the regex statically but they're huge
            // and gnarly as hell.
            (password.split(/[^\W_]+/).length - 1) / this._minwords,
            1.0
        );
        // See above for issues pertaining to character classification:
        // we'll classify using the ascii range because that's basically our
        // only option
        var classes =
              ((/[a-z]/.test(password)) ? 1 : 0)
            + ((/[A-Z]/.test(password)) ? 1 : 0)
            + ((/\d/.test(password)) ? 1 : 0)
            + ((/[^A-Za-z\d]/.test(password)) ? 1 : 0);
        var classesscore = Math.min(classes / this._minclasses, 1.0);

        return lengthscore * wordscore * classesscore;
    },
});

return {
    /**
     * Computes the password's score, should be roughly continuous, under 0.5
     * if the requirements don't pass and at 1 if the recommendations are
     * exceeded
     */
    computeScore: function (password, requirements, recommendations) {
        var req = requirements.score(password);
        var rec = recommendations.score(password);
        return Math.pow(req, 4) * (0.5 + Math.pow(rec, 2) / 2);
    },
    Policy: Policy,
    // Recommendations from Shay (2016):
    // Our research has shown that there are other policies that are more
    // usable and more secure. We found three policies (2class12, 3class12,
    // and 2word16) that we can directly recommend over comp8
    //
    // Since 2class12 is a superset of 3class12 and 2word16, either pick it or
    // pick the other two (and get the highest score of the two). We're
    // picking the other two.
    recommendations: {
        score: function (password) {
            return _.max(_.invoke(this.policies, 'score', password));
        },
        policies: [
            new Policy({minlength: 16, minwords: 2}),
            new Policy({minlength: 12, minclasses: 3})
        ]
    }
}
});

odoo.define('auth_password_policy.Meter', function (require) {
"use strict";
var core = require('web.core');
var policy = require('auth_password_policy');
var Widget = require('web.Widget');
var _t = core._t;

var PasswordPolicyMeter = Widget.extend({
    tagName: 'meter',
    className: 'o_password_meter',
    attributes: {
        min: 0,
        low: 0.5,
        high: 0.99,
        max: 1,
        value: 0,
        optimum: 1,
    },
    init: function (parent, required, recommended) {
        this._super(parent);
        this._required = required;
        this._recommended = recommended;
    },
    start: function () {
        var helpMessage = _t("Required: %s.\n\nHint: increase length, use multiple words and use non-letter characters to increase your password's strength.");
        this.el.setAttribute(
            'title', _.str.sprintf(helpMessage, String(this._required) || _t("no requirements")));
        return this._super().then(function () {
        });
    },
    /**
     * Updates the meter with the information of the new password: computes
     * the (required x recommended) score and sets the widget's value as that
     *
     * @param {String} password
     */
    update: function (password) {
        this.el.value = policy.computeScore(password, this._required, this._recommended);
    }
});
return PasswordPolicyMeter;
});

```

## File: views\assets.xml

```xml
<odoo>
    <template id="assets_backend" inherit_id="web.assets_backend"
              name="Password Policy Backend Assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/auth_password_policy/static/src/js/password_field.js"></script>
            <script type="text/javascript" src="/auth_password_policy/static/src/js/change_password.js"></script>
        </xpath>
    </template>
    <template id="assets_common" inherit_id="web.assets_common"
              name="Password Policy Common Assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/auth_password_policy/static/src/js/password_gauge.js"></script>
            <link rel="stylesheet" type="text/css" href="/auth_password_policy/static/src/css/password_field.css"/>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.form.auth_password_policy</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <!-- add before the Access Rights section -->
            <xpath expr="//div[@id='allow_import']" position="before">
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_left_pane"/>
                    <div class="o_setting_right_pane">
                        <label for="minlength"/>
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

