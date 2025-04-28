# Odoo Module: auth_totp_mail_enforce

Category: Extra Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import tests

```

## File: __manifest__.py

```python
{
    'name': '2FA by mail',
    'description': """
2FA by mail
===============
Two-Factor authentication by sending a code to the user email inbox
when the 2FA using an authenticator app is not configured.
To enforce users to use a two-factor authentication by default,
and encourage users to configure their 2FA using an authenticator app.
    """,
    'depends': ['auth_totp', 'mail'],
    'category': 'Extra Tools',
    'data': [
        'data/mail_template_data.xml',
        'security/ir.model.access.csv',
        'views/res_config_settings_views.xml',
        'views/templates.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\home.py

```python
# -*- coding: utf-8 -*-
import logging
import odoo.addons.auth_totp.controllers.home

from odoo import http
from odoo.exceptions import AccessDenied, UserError
from odoo.http import request

_logger = logging.getLogger(__name__)


class Home(odoo.addons.auth_totp.controllers.home.Home):
    @http.route()
    def web_totp(self, redirect=None, **kwargs):
        response = super().web_totp(redirect=redirect, **kwargs)
        if response.status_code != 200 or response.qcontext['user']._mfa_type() != 'totp_mail':
            # In case the response from the super is a redirection
            # or the user has another TOTP method, we return the response from the call to super.
            return response
        assert request.session.pre_uid and not request.session.uid, \
            "The user must still be in the pre-authentication phase"

        # Send the email containing the code to the user inbox
        try:
            response.qcontext['user']._send_totp_mail_code()
        except (AccessDenied, UserError) as e:
            response.qcontext['error'] = str(e)
        except Exception as e:
            _logger.exception('Unable to send TOTP email')
            response.qcontext['error'] = str(e)
        return response

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import home

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_totp_mail_code" model="mail.template">
            <field name="name">Settings: 2Fa New Login</field>
            <field name="model_id" ref="base.model_res_users" />
            <field name="subject">Your two-factor authentication code</field>
            <field name="email_to">{{ object.email_formatted }}</field>
            <field name="email_from">{{ (object.company_id.email_formatted or user.email_formatted) }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    Dear <t t-out="object.partner_id.name or ''"></t><br/><br/>
    <p>Someone is trying to log in into your account with a new device.</p>
    <ul>
        <t t-set="not_available">N/A</t>
        <li>Location: <t t-out="ctx.get('location') or not_available"/></li>
        <li>Device: <t t-out="ctx.get('device') or not_available"/></li>
        <li>Browser: <t t-out="ctx.get('browser') or not_available"/></li>
        <li>IP address: <t t-out="ctx.get('ip') or not_available"/></li>
    </ul>
    <p>If this is you, please enter the following code to complete the login:</p>
    <t t-set="code_expiration" t-value="object._get_totp_mail_code()"/>
    <t t-set="code" t-value="code_expiration[0]"/>
    <t t-set="expiration" t-value="code_expiration[1]"/>
    <div style="margin: 16px 0px 16px 0px; text-align: center;">
        <span t-out="code" style="background-color:#faf9fa; border: 1px solid #dad8de; padding: 8px 16px 8px 16px; font-size: 24px; color: #875A7B; border-radius: 5px;"/>
    </div>
    <small>Please note that this code expires in <t t-out="expiration"/>.</small>

    <p style="margin: 16px 0px 16px 0px;">
        If you did NOT initiate this log-in,
        you should immediately change your password to ensure account security.
    </p>

    <p style="margin: 16px 0px 16px 0px;">
        We also strongly recommend enabling the two-factor authentication using an authenticator app to help secure your account.
    </p>

    <p style="margin: 16px 0px 16px 0px; text-align: center;">
        <a t-att-href="object.get_totp_invite_url()"
            style="background-color:#875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px;">
            Activate my two-factor authentication
        </a>
    </p>
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: models\auth_totp_rate_limit_log.py

```python
from odoo import fields, models


class AuthTotpRateLimitLog(models.TransientModel):
    _name = 'auth.totp.rate.limit.log'
    _description = 'TOTP rate limit logs'

    def init(self):
        self.env.cr.execute("""
            CREATE INDEX IF NOT EXISTS auth_totp_rate_limit_log_user_id_limit_type_create_date_idx
            ON auth_totp_rate_limit_log(user_id, limit_type, create_date);
        """)

    user_id = fields.Many2one('res.users', required=True, readonly=True)
    scope = fields.Char(readonly=True)
    ip = fields.Char(readonly=True)
    limit_type = fields.Selection([
        ('send_email', 'Send Email'),
        ('code_check', 'Code Checking'),
    ], readonly=True)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    auth_totp_enforce = fields.Boolean(
        string="Enforce two-factor authentication",
    )
    auth_totp_policy = fields.Selection([
        ('employee_required', 'Employees only'),
        ('all_required', 'All users')
    ],
        string="Two-factor authentication enforcing policy",
        config_parameter='auth_totp.policy',
    )

    @api.onchange('auth_totp_enforce')
    def _onchange_auth_totp_enforce(self):
        if self.auth_totp_enforce:
            self.auth_totp_policy = self.auth_totp_policy or 'employee_required'
        else:
            self.auth_totp_policy = False

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        res['auth_totp_enforce'] = bool(self.env['ir.config_parameter'].sudo().get_param('auth_totp.policy'))
        return res

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import babel.dates
import logging

from datetime import datetime, timedelta

from odoo import _, models
from odoo.exceptions import AccessDenied, UserError
from odoo.http import request
from odoo.tools.misc import babel_locale_parse, hmac

from odoo.addons.auth_totp.models.totp import hotp, TOTP

_logger = logging.getLogger(__name__)

TOTP_RATE_LIMITS = {
    'send_email': (10, 3600),
    'code_check': (10, 3600),
}


class Users(models.Model):
    _inherit = 'res.users'

    def _mfa_type(self):
        r = super()._mfa_type()
        if r is not None:
            return r
        ICP = self.env['ir.config_parameter'].sudo()
        otp_required = False
        if ICP.get_param('auth_totp.policy') == 'all_required':
            otp_required = True
        elif ICP.get_param('auth_totp.policy') == 'employee_required' and self._is_internal():
            otp_required = True
        if otp_required:
            return 'totp_mail'

    def _mfa_url(self):
        r = super()._mfa_url()
        if r is not None:
            return r
        if self._mfa_type() == 'totp_mail':
            return '/web/login/totp'

    def _totp_check(self, code):
        self._totp_rate_limit('code_check')
        user = self.sudo()
        if user._mfa_type() != 'totp_mail':
            return super()._totp_check(code)

        key = user._get_totp_mail_key()
        match = TOTP(key).match(code, window=3600, timestep=3600)
        if match is None:
            _logger.info("2FA check (mail): FAIL for %s %r", user, user.login)
            raise AccessDenied(_("Verification failed, please double-check the 6-digit code"))
        _logger.info("2FA check(mail): SUCCESS for %s %r", user, user.login)
        self._totp_rate_limit_purge('code_check')
        self._totp_rate_limit_purge('send_email')
        return True

    def _get_totp_mail_key(self):
        self.ensure_one()
        return hmac(self.env(su=True), 'auth_totp_mail-code', (self.id, self.login, self.login_date)).encode()

    def _get_totp_mail_code(self):
        self.ensure_one()

        key = self._get_totp_mail_key()

        now = datetime.now()
        counter = int(datetime.timestamp(now) / 3600)

        code = hotp(key, counter)
        expiration = timedelta(seconds=3600)
        lang = babel_locale_parse(self.env.context.get('lang') or self.lang)
        expiration = babel.dates.format_timedelta(expiration, locale=lang)

        return str(code).zfill(6), expiration

    def _send_totp_mail_code(self):
        self.ensure_one()
        self._totp_rate_limit('send_email')

        if not self.email:
            raise UserError(_("Cannot send email: user %s has no email address.", self.name))

        template = self.env.ref('auth_totp_mail_enforce.mail_template_totp_mail_code').sudo()
        context = {}
        if request:
            device = request.httprequest.user_agent.platform
            browser = request.httprequest.user_agent.browser
            context.update({
                'location': None,
                'device': device and device.capitalize() or None,
                'browser': browser and browser.capitalize() or None,
                'ip': request.httprequest.environ['REMOTE_ADDR'],
            })
            if request.geoip.city.name:
                context['location'] = f"{request.geoip.city.name}, {request.geoip.country_name}"

        email_values = {
            'email_to': self.email,
            'email_cc': False,
            'auto_delete': True,
            'recipient_ids': [],
            'partner_ids': [],
            'scheduled_date': False,
        }
        with self.env.cr.savepoint():
            template.with_context(**context).send_mail(
                self.id, force_send=True, raise_exception=True, email_values=email_values, email_layout_xmlid='mail.mail_notification_light'
            )

    def _totp_rate_limit(self, limit_type):
        self.ensure_one()
        assert request, "A request is required to be able to rate limit TOTP related actions"
        limit, interval = TOTP_RATE_LIMITS.get(limit_type)
        RateLimitLog = self.env['auth.totp.rate.limit.log'].sudo()
        ip = request.httprequest.environ['REMOTE_ADDR']
        domain = [
            ('user_id', '=', self.id),
            ('create_date', '>=', datetime.now() - timedelta(seconds=interval)),
            ('limit_type', '=', limit_type),
            ('ip', '=', ip),
        ]
        count = RateLimitLog.search_count(domain)
        if count >= limit:
            descriptions = {
                'send_email': _('You reached the limit of authentication mails sent for your account'),
                'code_check': _('You reached the limit of code verifications for your account'),
            }
            description = descriptions.get(limit_type)
            raise AccessDenied(description)
        RateLimitLog.create({
            'user_id': self.id,
            'ip': ip,
            'limit_type': limit_type,
        })

    def _totp_rate_limit_purge(self, limit_type):
        self.ensure_one()
        assert request, "A request is required to be able to rate limit TOTP related actions"
        ip = request.httprequest.environ['REMOTE_ADDR']
        RateLimitLog = self.env['auth.totp.rate.limit.log'].sudo()
        RateLimitLog.search([
            ('user_id', '=', self.id),
            ('limit_type', '=', limit_type),
            ('ip', '=', ip),
        ]).unlink()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import auth_totp_rate_limit_log
from . import res_config_settings
from . import res_users

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_auth_totp_rate_limit_log","access_auth_totp_rate_limit_log","model_auth_totp_rate_limit_log","base.group_user",0,0,0,0

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.auth_totp_mail_enforce</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="40"/>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//setting[@id='allow_import']" position="before">
                    <setting id="auth_totp_policy" help="Enforce the two-factor authentication by email for employees or for all users (including portal users) if they didn't enable any other two-factor authentication method.">
                        <field name="auth_totp_enforce" />
                        <div class="mt16" invisible="not auth_totp_enforce">
                            <field name="auth_totp_policy" class="o_light_label" widget="radio"/>
                        </div>
                    </setting>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\templates.xml

```xml
<odoo>
    <template id="auth_totp_mail_form" inherit_id="auth_totp.auth_totp_form">
        <xpath expr="//form/div[1]" position="attributes">
            <attribute name="t-if">user._mfa_type() == 'totp'</attribute>
        </xpath>
        <xpath expr="//form/div[1]" position="after">
            <div t-if="user._mfa_type() == 'totp_mail'" class="mb-2 mt-2 text-muted">
                <i class="fa fa-envelope-o"/>
                To login, enter below the six-digit authentication code just sent via email to <t t-out="user.email"/>.
                <br/>
            </div>
        </xpath>
        <xpath expr="//form[1]" position="after">
            <form method="POST" t-if="user._mfa_type() == 'totp_mail'">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <input type="hidden" name="send_email" value="1"/>
                <button type="submit" class="btn btn-secondary btn-block">Re-send email</button>
            </form>
        </xpath>
        <xpath expr="//div[hasclass('border-top')]" position="before">
            <div class="mb-2" t-if="user._mfa_type() == 'totp_mail'">
                We strongly recommend enabling the two-factor authentication using an authenticator app to help secure your account.
                <br/>
                <a href="https://www.odoo.com/documentation/17.0/applications/general/auth/2fa.html" title="Learn More" target="_blank">Learn More</a>
            </div>
        </xpath>
    </template>
</odoo>

```

