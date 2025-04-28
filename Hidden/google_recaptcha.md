# Odoo Module: google_recaptcha

Category: Hidden

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
    'name': 'Google reCAPTCHA integration',
    'category': 'Hidden',
    'version': '1.0',
    'description': """
This module implements reCaptchaV3 so that you can prevent bot spam on your public modules.
    """,
    'depends': ['base_setup'],
    'data': [
        'views/res_config_settings_view.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'google_recaptcha/static/src/scss/recaptcha.scss',
            'google_recaptcha/static/src/js/recaptcha.js',
            'google_recaptcha/static/src/js/signup.js',
        ],
        'web.assets_backend': [
            # TODO we may want to consider moving that file in website instead
            # of here and/or adding it in the "website.assets_wysiwyg" bundle,
            # which is lazy loaded.
            'google_recaptcha/static/src/xml/recaptcha.xml',
            'google_recaptcha/static/src/scss/recaptcha_backend.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import requests

from odoo import api, models, _
from odoo.http import request
from odoo.exceptions import UserError, ValidationError

logger = logging.getLogger(__name__)


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        session_info = super().session_info()
        return self._add_public_key_to_session_info(session_info)

    @api.model
    def get_frontend_session_info(self):
        frontend_session_info = super().get_frontend_session_info()
        return self._add_public_key_to_session_info(frontend_session_info)

    @api.model
    def _add_public_key_to_session_info(self, session_info):
        """Add the ReCaptcha public key to the given session_info object"""
        public_key = self.env['ir.config_parameter'].sudo().get_param('recaptcha_public_key')
        if public_key:
            session_info['recaptcha_public_key'] = public_key
        return session_info

    @api.model
    def _verify_request_recaptcha_token(self, action):
        """ Verify the recaptcha token for the current request.
            If no recaptcha private key is set the recaptcha verification
            is considered inactive and this method will return True.
        """
        ip_addr = request.httprequest.remote_addr
        token = request.params.pop('recaptcha_token_response', False)
        recaptcha_result = request.env['ir.http']._verify_recaptcha_token(ip_addr, token, action)
        if recaptcha_result in ['is_human', 'no_secret']:
            return True
        if recaptcha_result == 'wrong_secret':
            raise ValidationError(_("The reCaptcha private key is invalid."))
        elif recaptcha_result == 'wrong_token':
            raise ValidationError(_("The reCaptcha token is invalid."))
        elif recaptcha_result == 'timeout':
            raise UserError(_("Your request has timed out, please retry."))
        elif recaptcha_result == 'bad_request':
            raise UserError(_("The request is invalid or malformed."))
        else:
            return False

    @api.model
    def _verify_recaptcha_token(self, ip_addr, token, action=False):
        """
            Verify a recaptchaV3 token and returns the result as a string.
            RecaptchaV3 verify DOC: https://developers.google.com/recaptcha/docs/verify

            :return: The result of the call to the google API:
                     is_human: The token is valid and the user trustworthy.
                     is_bot: The user is not trustworthy and most likely a bot.
                     no_secret: No reCaptcha secret set in settings.
                     wrong_action: the action performed to obtain the token does not match the one we are verifying.
                     wrong_token: The token provided is invalid or empty.
                     wrong_secret: The private key provided in settings is invalid.
                     timeout: The request has timout or the token provided is too old.
                     bad_request: The request is invalid or malformed.
            :rtype: str
        """
        private_key = request.env['ir.config_parameter'].sudo().get_param('recaptcha_private_key')
        if not private_key:
            return 'no_secret'
        min_score = request.env['ir.config_parameter'].sudo().get_param('recaptcha_min_score')
        try:
            r = requests.post('https://www.recaptcha.net/recaptcha/api/siteverify', {
                'secret': private_key,
                'response': token,
                'remoteip': ip_addr,
            }, timeout=2)  # it takes ~50ms to retrieve the response
            result = r.json()
            res_success = result['success']
            res_action = res_success and action and result['action']
        except requests.exceptions.Timeout:
            logger.error("Trial captcha verification timeout for ip address %s", ip_addr)
            return 'timeout'
        except Exception:
            logger.error("Trial captcha verification bad request response")
            return 'bad_request'

        if res_success:
            score = result.get('score', False)
            if score < float(min_score):
                logger.warning("Trial captcha verification for ip address %s failed with score %f.", ip_addr, score)
                return 'is_bot'
            if res_action and res_action != action:
                logger.warning("Trial captcha verification for ip address %s failed with action %f, expected: %s.", ip_addr, score, action)
                return 'wrong_action'
            logger.info("Trial captcha verification for ip address %s succeeded with score %f.", ip_addr, score)
            return 'is_human'
        errors = result.get('error-codes', [])
        logger.warning("Trial captcha verification for ip address %s failed error codes %r. token was: [%s]", ip_addr, errors, token)
        for error in errors:
            if error in ['missing-input-secret', 'invalid-input-secret']:
                return 'wrong_secret'
            if error in ['missing-input-response', 'invalid-input-response']:
                return 'wrong_token'
            if error == 'timeout-or-duplicate':
                return 'timeout'
            if error == 'bad-request':
                return 'bad_request'
        return 'is_bot'

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    recaptcha_public_key = fields.Char("Site Key", config_parameter='recaptcha_public_key', groups='base.group_system')
    recaptcha_private_key = fields.Char("Secret Key", config_parameter='recaptcha_private_key', groups='base.group_system')
    recaptcha_min_score = fields.Float(
        "Minimum score",
        config_parameter='recaptcha_min_score',
        groups='base.group_system',
        default="0.7",
        help="By default, should be one of 0.1, 0.3, 0.7, 0.9.\n1.0 is very likely a good interaction, 0.0 is very likely a bot"
    )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import res_config_settings

```

## File: static\src\js\recaptcha.js

```javascript
/** @odoo-module **/

import { session } from "@web/session";
import { loadJS } from "@web/core/assets";
import { _t } from "@web/core/l10n/translation";

export class ReCaptcha {
    /**
     * @override
     */
    constructor() {
        this._publicKey = session.recaptcha_public_key;
    }
    /**
     * Loads the recaptcha libraries.
     *
     * @returns {Promise|boolean} promise if libs are loading else false if the reCaptcha key is empty.
     */
    loadLibs() {
        if (this._publicKey) {
            this._recaptchaReady = loadJS(`https://www.recaptcha.net/recaptcha/api.js?render=${encodeURIComponent(this._publicKey)}`)
                .then(() => new Promise(resolve => window.grecaptcha.ready(() => resolve())));
            return this._recaptchaReady.then(() => !!document.querySelector('.grecaptcha-badge'));
        }
        return false;
    }
    /**
     * Returns an object with the token if reCaptcha call succeeds
     * If no key is set an object with a message is returned
     * If an error occurred an object with the error message is returned
     *
     * @param {string} action
     * @returns {Promise|Object}
     */
    async getToken(action) {
        if (!this._publicKey) {
            return {
                message: _t("No recaptcha site key set."),
            };
        }
        await this._recaptchaReady;
        try {
            return {
                token: await window.grecaptcha.execute(this._publicKey, {action: action})
            };
        } catch {
            return {
                error: _t("The recaptcha site key is invalid."),
            };
        }
    }
}

export default {
    ReCaptcha: ReCaptcha,
};

```

## File: static\src\js\signup.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { ReCaptcha } from "@google_recaptcha/js/recaptcha";

const CaptchaFunctionality = {
    events: {
        submit: "_onSubmit",
    },

    init() {
        this._super(...arguments);
        this._recaptcha = new ReCaptcha();
    },

    async willStart() {
        return this._recaptcha.loadLibs();
    },

    _onSubmit(ev) {
        if (this._recaptcha._publicKey && !this.$el.find("input[name='recaptcha_token_response']").length) {
            ev.preventDefault();
            this._recaptcha.getToken(this.tokenName).then((tokenCaptcha) => {
                this.$el.append(
                    `<input name="recaptcha_token_response" type="hidden" value="${tokenCaptcha.token}"/>`,
                );
                this.$el.submit();
            });
        }
    },
};

publicWidget.registry.SignupCaptcha = publicWidget.Widget.extend({
    ...CaptchaFunctionality,
    selector: ".oe_signup_form",
    tokenName: "signup",
});

publicWidget.registry.ResetPasswordCaptcha = publicWidget.Widget.extend({
    ...CaptchaFunctionality,
    selector: ".oe_reset_password_form",
    tokenName: "password_reset",
});

```

## File: static\src\xml\recaptcha.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<t t-name="google_recaptcha.recaptcha_legal_terms">
    <small class="o_recaptcha_legal_terms">
        Protected by reCAPTCHA,
        <a href="https://policies.google.com/privacy" target="_blank">Privacy Policy</a>
        &amp;
        <a href="https://policies.google.com/terms" target="_blank">Terms of Service</a>
        apply.
    </small>
</t>

</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.web.recaptcha</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='recaptcha']" position="attributes">
                <attribute name="help" add="If no keys are provided, no checks will be done." separator=" " />
            </xpath>
            <div id="recaptcha_warning" position="replace">
                <div class="content-group" id="reacaptcha_configuration_settings" invisible="not module_google_recaptcha">
                    <div class="mt16 row">
                        <label for="recaptcha_public_key" class="col-3 o_light_label"/>
                        <field name="recaptcha_public_key"/>
                    </div>
                    <div class="mt16 row">
                        <label for="recaptcha_private_key" class="col-3 o_light_label"/>
                        <field name="recaptcha_private_key"/>
                    </div>
                    <div class="mt16 row">
                        <label for="recaptcha_min_score" class="col-3 o_light_label"/>
                        <field name="recaptcha_min_score"/>
                    </div>
                    <div>
                        <a href="https://www.google.com/recaptcha/admin/create" class="oe_link" target="_blank">
                            <i class="oi oi-arrow-right"/> Generate reCAPTCHA v3 keys
                        </a>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

