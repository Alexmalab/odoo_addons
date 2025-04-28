# Odoo Module: auth_totp

Category: Extra Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
{
    'name': 'Two-Factor Authentication (TOTP)',
    'description': """
Two-Factor Authentication (TOTP)
================================
Allows users to configure two-factor authentication on their user account
for extra security, using time-based one-time passwords (TOTP).

Once enabled, the user will need to enter a 6-digit code as provided
by their authenticator app before being granted access to the system.
All popular authenticator apps are supported.

Note: logically, two-factor prevents password-based RPC access for users
where it is enabled. In order to be able to execute RPC scripts, the user
can setup API keys to replace their main password.
    """,
    'depends': ['web'],
    'category': 'Extra Tools',
    'auto_install': True,
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'data/ir_action_data.xml',
        'views/res_users_views.xml',
        'views/templates.xml',
        'wizard/auth_totp_wizard_views.xml',
    ],
    'assets': {
        'web.assets_tests': [
            'auth_totp/static/tests/**/*',
        ],
        'web.assets_backend': [
            'auth_totp/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\home.py

```python
# -*- coding: utf-8 -*-
import re

from odoo import http, _
from odoo.exceptions import AccessDenied
from odoo.http import request
from odoo.addons.web.controllers import home as web_home

TRUSTED_DEVICE_COOKIE = 'td_id'
TRUSTED_DEVICE_AGE = 90*86400 # 90 days expiration


class Home(web_home.Home):
    @http.route(
        '/web/login/totp',
        type='http', auth='public', methods=['GET', 'POST'], sitemap=False,
        website=True, multilang=False # website breaks the login layout...
    )
    def web_totp(self, redirect=None, **kwargs):
        if request.session.uid:
            return request.redirect(self._login_redirect(request.session.uid, redirect=redirect))

        if not request.session.pre_uid:
            return request.redirect('/web/login')

        error = None

        user = request.env['res.users'].browse(request.session.pre_uid)
        if user and request.httprequest.method == 'GET':
            cookies = request.httprequest.cookies
            key = cookies.get(TRUSTED_DEVICE_COOKIE)
            if key:
                user_match = request.env['auth_totp.device']._check_credentials_for_uid(
                    scope="browser", key=key, uid=user.id)
                if user_match:
                    request.session.finalize(request.env)
                    request.update_env(user=request.session.uid)
                    request.update_context(**request.session.context)
                    return request.redirect(self._login_redirect(request.session.uid, redirect=redirect))

        elif user and request.httprequest.method == 'POST' and kwargs.get('totp_token'):
            try:
                with user._assert_can_auth(user=user.id):
                    user._totp_check(int(re.sub(r'\s', '', kwargs['totp_token'])))
            except AccessDenied as e:
                error = str(e)
            except ValueError:
                error = _("Invalid authentication code format.")
            else:
                request.session.finalize(request.env)
                request.update_env(user=request.session.uid)
                request.update_context(**request.session.context)
                response = request.redirect(self._login_redirect(request.session.uid, redirect=redirect))
                if kwargs.get('remember'):
                    name = _("%(browser)s on %(platform)s",
                        browser=request.httprequest.user_agent.browser.capitalize(),
                        platform=request.httprequest.user_agent.platform.capitalize(),
                    )

                    if request.geoip.city.name:
                        name += f" ({request.geoip.city.name}, {request.geoip.country_name})"

                    key = request.env['auth_totp.device']._generate("browser", name)
                    response.set_cookie(
                        key=TRUSTED_DEVICE_COOKIE,
                        value=key,
                        max_age=TRUSTED_DEVICE_AGE,
                        httponly=True,
                        samesite='Lax'
                    )
                # Crapy workaround for unupdatable Odoo Mobile App iOS (Thanks Apple :@)
                request.session.touch()
                return response

        # Crapy workaround for unupdatable Odoo Mobile App iOS (Thanks Apple :@)
        request.session.touch()
        return request.render('auth_totp.auth_totp_form', {
            'user': user,
            'error': error,
            'redirect': redirect,
        })

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import home

```

## File: data\ir_action_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Action called from the contextual menu -->
    <record model="ir.actions.server" id="action_disable_totp">
        <field name="name">Disable two-factor authentication</field>
        <field name="model_id" ref="base.model_res_users"/>
        <field name="binding_model_id" ref="base.model_res_users"/>
        <field name="state">code</field>
        <field name="code">
            action = records.action_totp_disable()
        </field>
        <field name="groups_id" eval="[(4, ref('base.group_erp_manager'))]"/>
    </record>
</odoo>

```

## File: models\auth_totp.py

```python
# -*- coding: utf-8 -*-
from odoo import api, models
from odoo.addons.auth_totp.controllers.home import TRUSTED_DEVICE_AGE

import logging
_logger = logging.getLogger(__name__)


class AuthTotpDevice(models.Model):

    # init is overriden in res.users.apikeys to create a secret column 'key'
    # use a different model to benefit from the secured methods while not mixing
    # two different concepts

    _name = "auth_totp.device"
    _inherit = "res.users.apikeys"
    _description = "Authentication Device"
    _auto = False

    def _check_credentials_for_uid(self, *, scope, key, uid):
        """Return True if device key matches given `scope` for user ID `uid`"""
        assert uid, "uid is required"
        return self._check_credentials(scope=scope, key=key) == uid

    @api.autovacuum
    def _gc_device(self):
        self._cr.execute("""
            DELETE FROM auth_totp_device
            WHERE create_date < (NOW() AT TIME ZONE 'UTC' - INTERVAL '%s SECONDS')
        """, [TRUSTED_DEVICE_AGE])
        _logger.info("GC'd %d totp devices entries", self._cr.rowcount)

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
from odoo import models
from odoo.http import request

class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        info = super().session_info()
        # because frontend session_info uses this key and is embedded in
        # the view source
        info["user_id"] = request.session.uid,
        return info

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import functools
import logging
import os
import re

from odoo import _, api, fields, models
from odoo.addons.base.models.res_users import check_identity
from odoo.exceptions import AccessDenied, UserError
from odoo.http import request
from odoo.tools import sql

from odoo.addons.auth_totp.models.totp import TOTP, TOTP_SECRET_SIZE

_logger = logging.getLogger(__name__)

compress = functools.partial(re.sub, r'\s', '')
class Users(models.Model):
    _inherit = 'res.users'

    totp_secret = fields.Char(copy=False, groups=fields.NO_ACCESS, compute='_compute_totp_secret', inverse='_inverse_token')
    totp_enabled = fields.Boolean(string="Two-factor authentication", compute='_compute_totp_enabled', search='_totp_enable_search')
    totp_trusted_device_ids = fields.One2many('auth_totp.device', 'user_id', string="Trusted Devices")

    def init(self):
        super().init()
        if not sql.column_exists(self.env.cr, self._table, "totp_secret"):
            self.env.cr.execute("ALTER TABLE res_users ADD COLUMN totp_secret varchar")

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['totp_enabled', 'totp_trusted_device_ids']

    def _mfa_type(self):
        r = super()._mfa_type()
        if r is not None:
            return r
        if self.totp_enabled:
            return 'totp'

    def _should_alert_new_device(self):
        """ Determine if an alert should be sent to the user regarding a new device
        - 2FA enabled -> only for new device
        - Not enabled -> no alert

        To be overriden if needs to be disabled for other 2FA providers
        """
        if request and self._mfa_type():
            key = request.httprequest.cookies.get('td_id')
            if key:
                if request.env['auth_totp.device']._check_credentials_for_uid(
                    scope="browser", key=key, uid=self.id):
                    # the device is known
                    return False
            # 2FA enabled but not a trusted device
            return True
        return super()._should_alert_new_device()

    def _mfa_url(self):
        r = super()._mfa_url()
        if r is not None:
            return r
        if self._mfa_type() == 'totp':
            return '/web/login/totp'

    @api.depends('totp_secret')
    def _compute_totp_enabled(self):
        for r, v in zip(self, self.sudo()):
            r.totp_enabled = bool(v.totp_secret)

    def _rpc_api_keys_only(self):
        # 2FA enabled means we can't allow password-based RPC
        self.ensure_one()
        return self.totp_enabled or super()._rpc_api_keys_only()

    def _get_session_token_fields(self):
        return super()._get_session_token_fields() | {'totp_secret'}

    def _totp_check(self, code):
        sudo = self.sudo()
        key = base64.b32decode(sudo.totp_secret)
        match = TOTP(key).match(code)
        if match is None:
            _logger.info("2FA check: FAIL for %s %r", self, sudo.login)
            raise AccessDenied(_("Verification failed, please double-check the 6-digit code"))
        _logger.info("2FA check: SUCCESS for %s %r", self, sudo.login)

    def _totp_try_setting(self, secret, code):
        if self.totp_enabled or self != self.env.user:
            _logger.info("2FA enable: REJECT for %s %r", self, self.login)
            return False

        secret = compress(secret).upper()
        match = TOTP(base64.b32decode(secret)).match(code)
        if match is None:
            _logger.info("2FA enable: REJECT CODE for %s %r", self, self.login)
            return False

        self.sudo().totp_secret = secret
        if request:
            self.env.flush_all()
            # update session token so the user does not get logged out (cache cleared by change)
            new_token = self.env.user._compute_session_token(request.session.sid)
            request.session.session_token = new_token

        _logger.info("2FA enable: SUCCESS for %s %r", self, self.login)
        return True

    @check_identity
    def action_totp_disable(self):
        logins = ', '.join(map(repr, self.mapped('login')))
        if not (self == self.env.user or self.env.user._is_admin() or self.env.su):
            _logger.info("2FA disable: REJECT for %s (%s) by uid #%s", self, logins, self.env.user.id)
            return False

        self.revoke_all_devices()
        self.sudo().write({'totp_secret': False})

        if request and self == self.env.user:
            self.env.flush_all()
            # update session token so the user does not get logged out (cache cleared by change)
            new_token = self.env.user._compute_session_token(request.session.sid)
            request.session.session_token = new_token

        _logger.info("2FA disable: SUCCESS for %s (%s) by uid #%s", self, logins, self.env.user.id)
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'warning',
                'message': _("Two-factor authentication disabled for the following user(s): %s", ', '.join(self.mapped('name'))),
                'next': {'type': 'ir.actions.act_window_close'},
            }
        }

    @check_identity
    def action_totp_enable_wizard(self):
        if self.env.user != self:
            raise UserError(_("Two-factor authentication can only be enabled for yourself"))

        if self.totp_enabled:
            raise UserError(_("Two-factor authentication already enabled"))

        secret_bytes_count = TOTP_SECRET_SIZE // 8
        secret = base64.b32encode(os.urandom(secret_bytes_count)).decode()
        # format secret in groups of 4 characters for readability
        secret = ' '.join(map(''.join, zip(*[iter(secret)]*4)))
        w = self.env['auth_totp.wizard'].create({
            'user_id': self.id,
            'secret': secret,
        })
        return {
            'type': 'ir.actions.act_window',
            'target': 'new',
            'res_model': 'auth_totp.wizard',
            'name': _("Two-Factor Authentication Activation"),
            'res_id': w.id,
            'views': [(False, 'form')],
            'context': self.env.context,
        }

    @check_identity
    def revoke_all_devices(self):
        self._revoke_all_devices()

    def _revoke_all_devices(self):
        self.totp_trusted_device_ids._remove()

    @api.model
    def change_password(self, old_passwd, new_passwd):
        self.env.user._revoke_all_devices()
        return super().change_password(old_passwd, new_passwd)

    def _compute_totp_secret(self):
        for user in self:
            self.env.cr.execute('SELECT totp_secret FROM res_users WHERE id=%s', (user.id,))
            user.totp_secret = self.env.cr.fetchone()[0]

    def _inverse_token(self):
        for user in self:
            secret = user.totp_secret if user.totp_secret else None
            self.env.cr.execute('UPDATE res_users SET totp_secret = %s WHERE id=%s', (secret, user.id))

    def _totp_enable_search(self, operator, value):
        value = not value if operator == '!=' else value
        if value:
            self.env.cr.execute("SELECT id FROM res_users WHERE totp_secret IS NOT NULL")
        else:
            self.env.cr.execute("SELECT id FROM res_users WHERE totp_secret IS NULL OR totp_secret='false'")
        result = self.env.cr.fetchall()
        return [('id', 'in', [x[0] for x in result])]

```

## File: models\totp.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hmac
import struct
import time

# 160 bits, as recommended by HOTP RFC 4226, section 4, R6.
# Google Auth uses 80 bits by default but supports 160.
TOTP_SECRET_SIZE = 160

# The algorithm (and key URI format) allows customising these parameters but
# google authenticator doesn't support it
# https://github.com/google/google-authenticator/wiki/Key-Uri-Format
ALGORITHM = 'sha1'
DIGITS = 6
TIMESTEP = 30

class TOTP:
    def __init__(self, key):
        self._key = key

    def match(self, code, t=None, window=TIMESTEP, timestep=TIMESTEP):
        """
        :param code: authenticator code to check against this key
        :param int t: current timestamp (seconds)
        :param int window: fuzz window to account for slow fingers, network
                           latency, desynchronised clocks, ..., every code
                           valid between t-window an t+window is considered
                           valid
        """
        if t is None:
            t = time.time()

        low = int((t - window) / timestep)
        high = int((t + window) / timestep) + 1

        return next((
            counter for counter in range(low, high)
            if hotp(self._key, counter) == code
        ), None)

def hotp(secret, counter):
    # C is the 64b counter encoded in big-endian
    C = struct.pack(">Q", counter)
    mac = hmac.new(secret, msg=C, digestmod=ALGORITHM).digest()
    # the data offset is the last nibble of the hash
    offset = mac[-1] & 0xF
    # code is the 4 bytes at the offset interpreted as a 31b big-endian uint
    # (31b to avoid sign concerns). This effectively limits digits to 9 and
    # hard-limits it to 10: each digit is normally worth 3.32 bits but the
    # 10th is only worth 1.1 (9 digits encode 29.9 bits).
    code = struct.unpack_from('>I', mac, offset)[0] & 0x7FFFFFFF
    r = code % (10 ** DIGITS)
    # NOTE: use text / bytes instead of int?
    return r

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import auth_totp
from . import ir_http
from . import res_users
from . import totp

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_auth_totp_device_access_employee","TOTP Device access employees","model_auth_totp_device","base.group_user",1,0,0,0
"access_auth_totp_device_access_portal","TOTP Device access portal","model_auth_totp_device","base.group_portal",1,0,0,0

```

## File: security\security.xml

```xml
<odoo>
    <record model="ir.model.access" id="access_auth_totp_wizard">
        <field name="name">auth_totp wizard access rules</field>
        <field name="model_id" ref="model_auth_totp_wizard"/>
        <field name="group_id" ref="base.group_user"/>
        <field name="perm_read">1</field>
        <field name="perm_write">1</field>
        <field name="perm_create">1</field>
        <field name="perm_unlink">1</field>
    </record>
    <record model="ir.rule" id="rule_auth_totp_wizard">
        <field name="name">Users can only access their own wizard</field>
        <field name="model_id" ref="model_auth_totp_wizard"/>
        <field name="domain_force">[('user_id', '=', user.id)]</field>
    </record>

        <!-- rules for API token -->
        <record id="api_key_public" model="ir.rule">
            <field name="name">Public users can't interact with keys at all</field>
            <field name="model_id" ref="model_auth_totp_device"/>
            <field name="domain_force">[(0, '=', 1)]</field>
            <field name="groups" eval="[Command.link(ref('base.group_public'))]"/>
        </record>
        <record id="api_key_user" model="ir.rule">
            <field name="name">Users can read and delete their own keys</field>
            <field name="model_id" ref="model_auth_totp_device"/>
            <field name="domain_force">[('user_id', '=', user.id)]</field>
            <field name="groups" eval="[
                Command.link(ref('base.group_portal')),
                Command.link(ref('base.group_user')),
            ]"/>
        </record>
        <record id="api_key_admin" model="ir.rule">
            <field name="name">Administrators can view user keys to revoke them</field>
            <field name="model_id" ref="model_auth_totp_device"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[Command.link(ref('base.group_system'))]"/>
        </record>
</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="res_users_view_search">
        <field name="name">res.users.view.search.inherit.auth.totp</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_search" />
        <field name="arch" type="xml">
            <xpath expr="//search" position="inside">
                <separator/>
                <filter name="totp_enabled" string="Two-factor authentication Enabled" domain="[('totp_enabled','!=',False)]"/>
                <filter name="totp_disabled" string="Two-factor authentication Disabled" domain="[('totp_enabled','=',False)]"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="view_totp_form">
        <field name="name">user form: add totp status</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='preferences']" position="after">
                <page string="Account Security" name="security" invisible="not id">
                    <field name="totp_enabled" invisible="1"/>
                    <!-- For own user, allow to activate the two-factor Authentication -->
                    <div>
                        <div class="o_horizontal_separator d-flex align-items-center mt-2 mb-4 text-uppercase fw-bolder small">Two-factor Authentication
                                <div invisible="totp_enabled">
                                    <button invisible="id == uid" name="action_totp_enable_wizard"
                                            disabled="1" type="object" class="fa fa-toggle-off o_auth_2fa_btn disabled" aria-label="Enable 2FA"></button>
                                            <button invisible="id != uid" name="action_totp_enable_wizard"
                                            type="object" class="fa fa-toggle-off o_auth_2fa_btn disabled pe-auto" aria-label="Enable 2FA"></button>
                                </div>
                                <button invisible="not totp_enabled" name="action_totp_disable" type="object"
                                class="fa fa-toggle-on o_auth_2fa_btn text-primary enabled" aria-label="Disable 2FA"></button>
                            </div>
                            <span invisible="totp_enabled" class="text-muted">
                                Two-factor Authentication ("2FA") is a system of double authentication.
                                The first one is done with your password and the second one with a code you get from a dedicated mobile app.
                                Popular ones include Authy, Google Authenticator or the Microsoft Authenticator.
                                <a href="https://www.odoo.com/documentation/17.0/applications/general/auth/2fa.html" title="Learn More" target="_blank">
                                    <i title="Documentation" class="fa fa-fw o_button_icon fa-info-circle"></i>
                                    Learn More
                                </a>
                                </span>
                                <span invisible="not totp_enabled" class="text-muted">This account is protected!</span>
                            </div>
                        <group name="auth_devices" string="Trusted Devices" invisible="not totp_trusted_device_ids">
                            <div colspan="2">
                                <field name="totp_trusted_device_ids" nolabel="1" colspan="4" readonly="1">
                                    <tree create="false" delete="false">
                                        <field name="name" string="Device"/>
                                        <field name="create_date" string="Added On"/>
                                        <button type="object" name="remove"
                                                title="Revoke" icon="fa-trash"/>
                                    </tree>
                                </field>
                                <button name="revoke_all_devices" string="Revoke All" type="object" class="btn btn-secondary"
                                        confirm="Are you sure? The user may be asked to enter two-factor codes again on those devices"/>
                            </div>
                        </group>
                </page>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="view_totp_field">
        <field name="name">users preference: totp</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form_simple_modif"/>
        <field name="arch" type="xml">
            <group name="auth" position="after">
                <field name="totp_enabled" invisible="1"/>
                <div>
                    <div class="o_horizontal_separator mt-2 mb-4 text-uppercase fw-bolder small">Two-factor Authentication
                        <button invisible="totp_enabled" name="action_totp_enable_wizard"
                            type="object" class="fa fa-toggle-off o_auth_2fa_btn" aria-label="Enable 2FA"/>
                        <button invisible="not totp_enabled" name="action_totp_disable"
                            type="object" class="fa fa-toggle-on o_auth_2fa_btn text-primary" aria-label="Disable 2FA"/>
                    </div>
                    <span invisible="not totp_enabled" class="text-muted">Your account is protected!</span>
                    <span invisible="totp_enabled" class="text-muted">
                        Two-factor Authentication ("2FA") is a system of double authentication.
                        The first one is done with your password and the second one with a code you get from a dedicated mobile app.
                        Popular ones include Authy, Google Authenticator or the Microsoft Authenticator.
                        <a href="https://www.odoo.com/documentation/17.0/applications/general/auth/2fa.html" title="Learn More" target="_blank">
                            <i title="Documentation" class="fa fa-fw o_button_icon fa-info-circle"></i>
                            Learn More
                        </a>
                    </span>
                    <group name="auth_devices" string="Trusted Devices" invisible="not totp_trusted_device_ids">
                        <div colspan="2">
                            <field name="totp_trusted_device_ids" nolabel="1" colspan="4" readonly="1">
                                <tree create="false" delete="false">
                                    <field name="name" string="Device"/>
                                    <field name="create_date" string="Added On"/>
                                    <button type="object" name="remove"
                                            title="Revoke" icon="fa-trash"/>
                                </tree>
                            </field>
                            <button name="revoke_all_devices" string="Revoke All" type="object" class="btn btn-secondary"
                                    confirm="Are you sure? You may be asked to enter two-factor codes again on those devices"/>
                        </div>
                    </group>
                </div>
            </group>
        </field>
    </record>
</odoo>

```

## File: views\templates.xml

```xml
<odoo>

    <template id="auth_totp_form" name="Two-Factor Authentication">
        <t t-call="web.login_layout">
            <t t-set="disable_footer">1</t>
            <div class="oe_login_form">
                <h5 class="card-title">Two-factor Authentication</h5>
                <form method="POST" action="" class="">
                    <div class="mb-2 mt-2 text-muted">
                        To login, enter below the six-digit authentication code provided by your Authenticator app.
                        <br/>
                        <a href="https://www.odoo.com/documentation/17.0/applications/general/auth/2fa.html"
                           title="Learn More" target="_blank">Learn More</a>
                    </div>
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <input type="hidden" name="redirect" t-att-value="redirect"/>
                    <div class="mb-3">
                        <label class="fw-bold" for="totp_token">Authentication Code</label>
                        <input id="totp_token" name="totp_token" class="form-control mb-2"
                                autocomplete="off" inputmode="numeric" autofocus="autofocus" required="required" placeholder="e.g. 123456"/>
                    </div>
                    <div class="text-center py-2 border-top">
                        <p class="alert alert-danger" t-if="error" role="alert">
                            <t t-esc="error"/>
                        </p>
                    <div class="mb-2 mt-2 text-muted">
                        <input type="checkbox" name="remember" id="switch-remember" value="1"/>
                        <label for="switch-remember">Don't ask again on this device</label>
                    </div>
                        <div t-attf-class="clearfix oe_login_buttons text-center d-grid mb-1">
                            <button type="submit" class="btn btn-primary">
                                Log in
                            </button>
                        </div>

                    </div>
                </form>

                <form method="POST" action="/web/session/logout">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <div class="w-100 text-center">
                        <button type="submit" class="btn btn-link btn-sm mb-2">
                            Cancel
                        </button>
                    </div>
                </form>
            </div>
        </t>
    </template>
</odoo>

```

## File: wizard\auth_totp_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import functools
import io
import qrcode
import re
import werkzeug.urls

from odoo import _, api, fields, models
from odoo.addons.base.models.res_users import check_identity
from odoo.exceptions import UserError
from odoo.http import request

from odoo.addons.auth_totp.models.totp import ALGORITHM, DIGITS, TIMESTEP

compress = functools.partial(re.sub, r'\s', '')

class TOTPWizard(models.TransientModel):
    _name = 'auth_totp.wizard'
    _description = "2-Factor Setup Wizard"

    user_id = fields.Many2one('res.users', required=True, readonly=True)
    secret = fields.Char(required=True, readonly=True)
    url = fields.Char(store=True, readonly=True, compute='_compute_qrcode')
    qrcode = fields.Binary(
        attachment=False, store=True, readonly=True,
        compute='_compute_qrcode',
    )
    code = fields.Char(string="Verification Code", size=7)

    @api.depends('user_id.login', 'user_id.company_id.display_name', 'secret')
    def _compute_qrcode(self):
        # TODO: make "issuer" configurable through config parameter?
        global_issuer = request and request.httprequest.host.split(':', 1)[0]
        for w in self:
            issuer = global_issuer or w.user_id.company_id.display_name
            w.url = url = werkzeug.urls.url_unparse((
                'otpauth', 'totp',
                werkzeug.urls.url_quote(f'{issuer}:{w.user_id.login}', safe=':'),
                werkzeug.urls.url_encode({
                    'secret': compress(w.secret),
                    'issuer': issuer,
                    # apparently a lowercase hash name is anathema to google
                    # authenticator (error) and passlib (no token)
                    'algorithm': ALGORITHM.upper(),
                    'digits': DIGITS,
                    'period': TIMESTEP,
                }), ''
            ))

            data = io.BytesIO()
            qrcode.make(url.encode(), box_size=4).save(data, optimise=True, format='PNG')
            w.qrcode = base64.b64encode(data.getvalue()).decode()

    @check_identity
    def enable(self):
        try:
            c = int(compress(self.code))
        except ValueError:
            raise UserError(_("The verification code should only contain numbers"))
        if self.user_id._totp_try_setting(self.secret, c):
            self.secret = '' # empty it, because why keep it until GC?
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'type': 'success',
                    'message': _("2-Factor authentication is now enabled."),
                    'next': {'type': 'ir.actions.act_window_close'},
                }
            }
        raise UserError(_('Verification failed, please double-check the 6-digit code'))

    def create(self, vals_list):
        rule = self.env.ref('auth_totp.rule_auth_totp_wizard', raise_if_not_found=False)
        if rule and rule.sudo().groups:
            rule.sudo().groups = False
        return super().create(vals_list)

```

## File: wizard\auth_totp_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="view_totp_wizard">
        <field name="name">auth_totp wizard</field>
        <field name="model">auth_totp.wizard</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <div class="o_auth_totp_enable_2FA container">
                        <div class="mb-3 w-100">
                            <h3 class="fw-bold">Authenticator App Setup</h3>
                            <ul>
                                <div class="d-md-none d-block">
                                    <li>
                                        <field class="text-wrap" name="url" widget="url" options="{'website_path': True}"
                                        text="Click on this link to open your authenticator app"/></li>
                                </div>
                                <li>
                                    <div class="d-flex align-items-center flex-wrap">
                                        <span class="d-md-none d-block">Or install an authenticator app</span>
                                        <span class="d-none d-md-block">Install an authenticator app on your mobile device</span>
                                        <div class="d-block d-md-none">
                                            <a href="https://play.google.com/store/search?q=authenticator&amp;c=apps" class="mx-2" target="blank">
                                                <img alt="On Google Play" style="width: 24px;" src="/base_setup/static/src/img/logo_google_play.png"/>
                                            </a>
                                            <a href="http://appstore.com/2fa" class="mx-2" target="blank">
                                                <img alt="On Apple Store" style="width: 24px;" src="/base_setup/static/src/img/logo_apple_store.png"/>
                                            </a>
                                        </div>
                                    </div>
                                </li>

                                <span class="text-muted">Popular ones include Authy, Google Authenticator or the Microsoft Authenticator.</span>
                                <li>Look for an "Add an account" button</li>
                                <li>
                                    <span class="d-none d-md-block">When requested to do so, scan the barcode below</span>
                                    <span class="d-block d-md-none">When requested to do so, copy the key below</span>
                                </li>
                            </ul>

                            <!-- Desktop version -->
                            <div class="text-center d-none d-md-block">
                                <field name="qrcode" readonly="True" widget="image" options="{'reload': false }"/>

                                <h3 class="fw-bold"><a data-bs-toggle="collapse"
                                   href="#collapseTotpSecret" role="button" aria-expanded="false"
                                   aria-controls="collapseTotpSecret">Cannot scan it?</a></h3>
                                <div class="collapse" id="collapseTotpSecret">
                                  <field name="secret" widget="CopyClipboardChar" readonly="1" class="mb-3 ps-3"/>
                                </div>
                            </div>

                            <!-- Mobile Version -->
                            <div class="text-center d-block d-md-none">
                                <field name="secret" widget="CopyClipboardChar" readonly="1" class="mb-3 ps-3"/>
                            </div>

                            <h3 class="fw-bold">Enter your six-digit code below</h3>
                            <div class="mt-2">
                                <label for="code" class="px-0">Verification Code</label>
                                <div class="d-flex align-items-center">
                                    <field required="True" name="code" autocomplete="off" class="o_field_highlight px-0 me-2" placeholder="e.g. 123456"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </sheet>
                <footer>
                    <button type="object" name="enable" class="btn btn-primary"
                            string="Activate" data-hotkey="q"/>
                    <button string="Cancel" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import auth_totp_wizard

```

