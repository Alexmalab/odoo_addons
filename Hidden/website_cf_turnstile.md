# Odoo Module: website_cf_turnstile

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
    'name': 'Cloudflare Turnstile',
    'category': 'Hidden',
    'version': '1.0',
    'description': """
This module implements Cloudflare Turnstile so that you can prevent bot spam on your forms.
    """,
    'depends': ['website'],
    'data': [
        'views/res_config_settings_view.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_cf_turnstile/static/src/js/turnstile.js',
            'website_cf_turnstile/static/src/js/error_handler.js',
            'website_cf_turnstile/static/src/xml/turnstile.xml',
        ],
    },
    'license': 'LGPL-3',
    'installable': True,
}

```

## File: data\neutralize.sql

```sql
-- disable cf turnstile 
UPDATE ir_config_parameter
SET value = ''
WHERE key IN ('cf.turnstile_site_key','cf.turnstile_secret_key');

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

    @api.model
    def get_frontend_session_info(self):
        """Add the Turnstile public key to the given session_info object"""
        session = super().get_frontend_session_info()

        site_key = self.env['ir.config_parameter'].sudo().get_param('cf.turnstile_site_key')
        if site_key:
            session['turnstile_site_key'] = site_key

        return session

    @api.model
    def _verify_request_recaptcha_token(self, action):
        """ Verify the recaptcha token for the current request.
            If no recaptcha private key is set the recaptcha verification
            is considered inactive and this method will return True.
        """
        res = super()._verify_request_recaptcha_token(action)

        if not res:  # check result of google_recaptcha
            return res

        ip_addr = request.httprequest.remote_addr
        token = request.params.pop('turnstile_captcha', False)
        turnstile_result = request.env['ir.http']._verify_turnstile_token(ip_addr, token, action)
        if turnstile_result in ['is_human', 'no_secret']:
            return True
        if turnstile_result == 'wrong_secret':
            raise ValidationError(_("The Cloudflare turnstile private key is invalid."))
        elif turnstile_result == 'wrong_token':
            raise ValidationError(_("The CloudFlare human validation failed."))
        elif turnstile_result == 'timeout':
            raise UserError(_("Your request has timed out, please retry."))
        elif turnstile_result == 'bad_request':
            raise UserError(_("The request is invalid or malformed."))
        else:  # wrong_action e.g.
            return False

    @api.model
    def _verify_turnstile_token(self, ip_addr, token, action=False):
        """
            Verify a turnstile token and returns the result as a string.
            Turnstile verify DOC: https://developers.cloudflare.com/turnstile/get-started/server-side-validation/

            :return: The result of the call to the cloudflare API:
                     is_human: The token is valid and the user trustworthy.
                     is_bot: The user is not trustworthy and most likely a bot.
                     no_secret: No private key in settings.
                     wrong_action: the action performed to obtain the token does not match the one we are verifying.
                     wrong_token: The token provided is invalid or empty.
                     wrong_secret: The private key provided in settings is invalid.
                     timeout: The request has timout or the token provided is too old.
                     bad_request: The request is invalid or malformed.
                     internal-error: The request failed.
            :rtype: str
        """
        private_key = request.env['ir.config_parameter'].sudo().get_param('cf.turnstile_secret_key')
        if not private_key:
            return 'no_secret'
        try:
            r = requests.post('https://challenges.cloudflare.com/turnstile/v0/siteverify', {
                'secret': private_key,
                'response': token,
                'remoteip': ip_addr,
            }, timeout=3.05)
            result = r.json()
            res_success = result['success']
            res_action = res_success and action and result['action']
        except requests.exceptions.Timeout:
            logger.error("Turnstile verification timeout for ip address %s", ip_addr)
            return 'timeout'
        except Exception:
            logger.error("Turnstile verification bad request response")
            return 'bad_request'

        if res_success:
            if res_action and res_action != action:
                logger.warning("Turnstile verification for ip address %s failed with action %f, expected: %s.", ip_addr, res_action, action)
                return 'wrong_action'
            logger.info("Turnstile verification for ip address %s succeeded", ip_addr)
            return 'is_human'
        errors = result.get('error-codes', [])
        logger.warning("Turnstile verification for ip address %s failed error codes %r. token was: [%s]", ip_addr, errors, token)
        for error in errors:
            if error in ['missing-input-secret', 'invalid-input-secret']:
                return 'wrong_secret'
            if error in ['missing-input-response', 'invalid-input-response']:
                return 'wrong_token'
            if error in ('timeout-or-duplicate', 'internal-error'):
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

    turnstile_site_key = fields.Char("CF Site Key", config_parameter='cf.turnstile_site_key', groups='base.group_system')
    turnstile_secret_key = fields.Char("CF Secret Key", config_parameter='cf.turnstile_secret_key', groups='base.group_system')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import res_config_settings

```

## File: static\src\js\error_handler.js

```javascript
/** @odoo-module */

import { ErrorDialog } from "@web/core/errors/error_dialogs";
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";

function turnstileErrorHandler(env, error) {
    if (error.message.includes("Turnstile Error")) {
        env.services.dialog.add(ErrorDialog, {
            name: _t("Cloudflare Turnstile Error"),
            traceback: _t(
                `There was an error with Cloudflare Turnstile, the captcha system.\n` +
                `Please make sure your credentials for this service are properly set up.\n` +
                `The error code is: %s.\n` +
                `You can find more information about this error code here: https://developers.cloudflare.com/turnstile/reference/errors.`,
                error.event.error.code
            ),
        });
        return true;
    }
}

registry.category("error_handlers").add("turnstile_error_handler", turnstileErrorHandler);

```

## File: static\src\js\turnstile.js

```javascript
/** @odoo-module **/

import "@website/snippets/s_website_form/000";  // force deps
import publicWidget from '@web/legacy/js/public/public_widget';
import { renderToElement } from "@web/core/utils/render";
import { session } from "@web/session";

export const turnStile = {
    addTurnstile: function (action) {
        if (!this.isEditable) {
            const mode = new URLSearchParams(window.location.search).get('cf') == 'show' ? 'always' : 'interaction-only';
            const turnstileContainer = renderToElement("website_cf_turnstile.turnstile_container", {
                action: action,
                appearance: mode,
                additionalClasses: "float-end",
                beforeInteractiveGlobalCallback: "turnstileBecomeVisible",
                errorGlobalCallback: "throwTurnstileErrorCode",
                executeGlobalCallback: "turnstileSuccess",
                expiredCallback: "turnstileExpired",
                sitekey: session.turnstile_site_key,
                style: "display: none;",
            });
            let toInsert = $(turnstileContainer);

            // Rethrow the error, or we only will catch a "Script error" without any info
            // because of the script api.js originating from a different domain.
            globalThis.throwTurnstileErrorCode = function (code) {
                const error = new Error("Turnstile Error");
                error.code = code;
                throw error;
            };
            const toggleSpinner = (turnstileContainer, show) => {
                const form = turnstileContainer.parentElement;
                const spinner = form.querySelector("i.turnstile-spinner");
                const button = spinner.parentElement;
                button.disabled = show;
                button.classList.toggle("disabled", show);
                spinner.classList.toggle("d-none", !show);
            };
            // `this` is bound to the turnstile widget calling the callback
            globalThis.turnstileSuccess = function () {
                toggleSpinner(this.wrapper.parentElement, false);
            };
            globalThis.turnstileExpired = function () {
                toggleSpinner(this.wrapper.parentElement, true);
            };
            globalThis.turnstileBecomeVisible = function () {
                const turnstileContainer = this.wrapper.parentElement;
                turnstileContainer.style.display = "";
            };

            // on first load of the remote script, all turnstile containers are rendered
            // if render=explicit is not set in the script url.
            // For subsequent insertion of turnstile containers, we need to call turnstile.render on the container
            // see `renderTurnstile`.
            if (!window.turnstile?.render) {
                const turnstileScript = renderToElement("website_cf_turnstile.turnstile_remote_script");
                toInsert = toInsert.add($(turnstileScript));
            }

            return toInsert;
        }
    },

    /**
     * Remove potential existing loaded script/token
     */
    cleanTurnstile: function () {
        if (this.$(".s_turnstile").length) {
            this.$(".s_turnstile").remove();
        }
    },

    /**
     * @override
     * Discard all library changes to reset the state of the Html.
     */
    destroy: function () {
        this.cleanTurnstile();
        this._super(...arguments);
    },

    /**
     * Take the result of `addTurnstile` and render it if needed
     */
    renderTurnstile: function (turnstileNodes) {
        turnstileNodes = turnstileNodes.toArray();
        const turnstileContainer = turnstileNodes.filter((node) =>
            node.classList.contains("s_turnstile_container")
        )[0];
        const turnstileScript = turnstileNodes.filter(
            (node) => node.id === "s_turnstile_remote_script"
        )[0];
        // there should only be a remote script if it was loaded for the first time
        if (turnstileScript) {
            return;
        }
        if (
            window.turnstile?.render &&
            turnstileContainer &&
            !turnstileContainer.querySelector("iframe")
        ) {
            window.turnstile.render(turnstileContainer);
        }
    },

    _createSpinner() {
        const spinner = document.createElement("i");
        spinner.classList.add("fa", "fa-refresh", "fa-spin", "turnstile-spinner");
        return spinner;
    },

    /**
     * same as addSpinner but does not set innerText
     */
    addSpinnerNoMangle(button) {
        const spinner = this._createSpinner();
        spinner.classList.add("me-1");
        button.disabled = true;
        button.classList.add("disabled");
        button.prepend(spinner);
    },

    addSpinner(button) {
        const spinner = this._createSpinner();
        // avoids double-spacing if the button already contains a space
        button.innerText = " " + button.innerText;
        button.disabled = true;
        button.classList.add("disabled");
        button.prepend(spinner);
    },
};

const signupTurnStile = {
    ...turnStile,

    async willStart() {
        this._super(...arguments);
        if (!session.turnstile_site_key) {
            return;
        }
        const button = this.el.querySelector('button[type="submit"]');
        this.addSpinner(button);
        this.cleanTurnstile();
        const turnstileNodes = this.addTurnstile(this.action);
        turnstileNodes?.insertBefore(button);
        this.renderTurnstile(turnstileNodes);
    },
};

publicWidget.registry.s_website_form.include({
    ...turnStile,

    /**
     * @override
     */
    start: function () {
        const res = this._super(...arguments);
        if (session.turnstile_site_key) {
            const button = this.el.querySelector(".s_website_form_send, .o_website_form_send");
            this.addSpinner(button);
            this.cleanTurnstile();
            const turnstileNodes = this.addTurnstile("website_form");
            turnstileNodes?.insertAfter(button);
            this.renderTurnstile(turnstileNodes);
        }
        return res;
    },
});

publicWidget.registry.turnstileCaptchaSignup = publicWidget.Widget.extend({
    ...signupTurnStile,
    selector: ".oe_signup_form",
    action: "signup",
});

publicWidget.registry.turnstileCaptchaPasswordReset = publicWidget.Widget.extend({
    ...signupTurnStile,
    selector: ".oe_reset_password_form",
    action: "password_reset",
});

```

## File: static\src\xml\turnstile.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="website_cf_turnstile.turnstile_container">
        <div
            t-attf-class="s_turnstile s_turnstile_container cf-turnstile {{additionalClasses}}"
            t-att-data-action="action"
            t-att-data-appearance="appeareance || 'interaction-only'"
            t-att-data-before-interactive-callback="beforeInteractiveGlobalCallback || '() => {}'"
            t-att-data-callback="executeGlobalCallback || '() => {}'"
            t-att-data-expired-callback="expiredCallback || '() => {}'"
            t-att-data-error-callback="errorGlobalCallback || '() => {}'"
            data-response-field-name="turnstile_captcha"
            t-att-data-sitekey="sitekey"
            t-att-style="style"
        ></div>
    </t>
    <t t-name="website_cf_turnstile.turnstile_remote_script">
        <script id="s_turnstile_remote_script" class="s_turnstile" src="https://challenges.cloudflare.com/turnstile/v0/api.js"></script>
    </t>
</templates>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.web.turnstile</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="turnstile_warning" position="replace">
                <div class="content-group" id="cfturnstile_configuration_settings">
                    <span class="o_form_label" for="">Cloudflare Turnstile</span>
                    <div class="mt16 row">
                        <label for="turnstile_site_key" class="col-3 o_light_label"/>
                        <field name="turnstile_site_key"/>
                    </div>
                    <div class="mt16 row">
                        <label for="turnstile_secret_key" class="col-3 o_light_label"/>
                        <field name="turnstile_secret_key"/>
                    </div>
                    <div>
                        <a href="https://blog.cloudflare.com/turnstile-private-captcha-alternative/" class="oe_link" target="_blank">
                            <i class="oi oi-arrow-right"/> More info
                        </a>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

