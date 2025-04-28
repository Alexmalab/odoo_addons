# Odoo Module: auth_totp

Category: Extra Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import controllers
from . import models

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
        'views/user_preferences.xml',
        'views/templates.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\home.py

```python
# -*- coding: utf-8 -*-
import re

import odoo.addons.web.controllers.main
from odoo import http, _
from odoo.addons.auth_totp.models.res_users import TRUSTED_DEVICE_SCOPE
from odoo.exceptions import AccessDenied
from odoo.http import request

TRUSTED_DEVICE_COOKIE = 'td_id'
TRUSTED_DEVICE_AGE = 90*86400 # 90 days expiration


class Home(odoo.addons.web.controllers.main.Home):
    @http.route(
        '/web/login/totp',
        type='http', auth='public', methods=['GET', 'POST'], sitemap=False,
        website=True, multilang=False # website breaks the login layout...
    )
    def web_totp(self, redirect=None, **kwargs):
        if request.session.uid:
            return http.redirect_with_hash(self._login_redirect(request.session.uid, redirect=redirect))

        if not request.session.pre_uid:
            return http.redirect_with_hash('/web/login')

        error = None
        user = request.env['res.users'].browse(request.session.pre_uid)
        if user and request.httprequest.method == 'GET':
            cookies = request.httprequest.cookies
            key = cookies.get(TRUSTED_DEVICE_COOKIE)
            if key:
                checked_credentials = request.env['res.users.apikeys']._check_credentials(scope=TRUSTED_DEVICE_SCOPE, key=key)
                if checked_credentials == user.id:
                    request.session.finalize()
                    return http.redirect_with_hash(self._login_redirect(request.session.uid, redirect=redirect))

        elif user and request.httprequest.method == 'POST':
            try:
                with user._assert_can_auth():
                    user._totp_check(int(re.sub(r'\s', '', kwargs['totp_token'])))
            except AccessDenied:
                error = _("Verification failed, please double-check the 6-digit code")
            except ValueError:
                error = _("Invalid authentication code format.")
            else:
                request.session.finalize()
                response = http.redirect_with_hash(self._login_redirect(request.session.uid, redirect=redirect))
                if kwargs.get('remember'):
                    name = _("%(browser)s on %(platform)s",
                        browser=request.httprequest.user_agent.browser.capitalize(),
                        platform=request.httprequest.user_agent.platform.capitalize(),
                    )
                    geoip = request.session.get('geoip')
                    if geoip:
                        name += " (%s, %s)" % (geoip['city'], geoip['country_name'])

                    key = request.env['res.users.apikeys']._generate(TRUSTED_DEVICE_SCOPE, name)
                    response.set_cookie(
                        key=TRUSTED_DEVICE_COOKIE,
                        value=key,
                        max_age=TRUSTED_DEVICE_AGE,
                        httponly=True,
                        samesite='Lax'
                    )
                return response

        return request.render('auth_totp.auth_totp_form', {
            'error': error,
            'redirect': redirect,
        })

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import home

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
import base64
import functools
import hmac
import io
import logging
import os
import re
import struct
import time

import werkzeug.urls

from odoo import _, api, fields, models
from odoo.addons.base.models.res_users import check_identity
from odoo.exceptions import AccessDenied, UserError
from odoo.http import request, db_list
from odoo.tools import sql

_logger = logging.getLogger(__name__)

TRUSTED_DEVICE_SCOPE = '2fa_trusted_device'

compress = functools.partial(re.sub, r'\s', '')
class Users(models.Model):
    _inherit = 'res.users'

    totp_secret = fields.Char(copy=False, groups=fields.NO_ACCESS, compute='_compute_totp_secret', inverse='_inverse_totp_secret')
    totp_enabled = fields.Boolean(string="Two-factor authentication", compute='_compute_totp_enabled', search='_search_totp_enable')
    totp_trusted_device_ids = fields.One2many('res.users.apikeys', 'user_id',
        string="Trusted Devices", domain=[('scope', '=', TRUSTED_DEVICE_SCOPE)])
    api_key_ids = fields.One2many(domain=[('scope', '!=', TRUSTED_DEVICE_SCOPE)])

    def __init__(self, pool, cr):
        init_res = super().__init__(pool, cr)
        if not sql.column_exists(cr, self._table, "totp_secret"):
            cr.execute("ALTER TABLE res_users ADD COLUMN totp_secret varchar")
        pool[self._name].SELF_READABLE_FIELDS = self.SELF_READABLE_FIELDS + ['totp_enabled', 'totp_trusted_device_ids']
        return init_res

    def _mfa_type(self):
        r = super()._mfa_type()
        if r is not None:
            return r
        if self.totp_enabled:
            return 'totp'

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
            raise AccessDenied()
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
            self.flush()
            # update session token so the user does not get logged out (cache cleared by change)
            new_token = self.env.user._compute_session_token(request.session.sid)
            request.session.session_token = new_token

        _logger.info("2FA enable: SUCCESS for %s %r", self, self.login)
        return True

    @check_identity
    def totp_disable(self):
        logins = ', '.join(map(repr, self.mapped('login')))
        if not (self == self.env.user or self.env.user._is_admin() or self.env.su):
            _logger.info("2FA disable: REJECT for %s (%s) by uid #%s", self, logins, self.env.user.id)
            return False

        self.revoke_all_devices()
        self.sudo().write({'totp_secret': False})

        if request and self == self.env.user:
            self.flush()
            # update session token so the user does not get logged out (cache cleared by change)
            new_token = self.env.user._compute_session_token(request.session.sid)
            request.session.session_token = new_token

        _logger.info("2FA disable: SUCCESS for %s (%s) by uid #%s", self, logins, self.env.user.id)
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'warning',
                'message': _("Two-factor authentication disabled for user(s) %s", logins),
                'next': {'type': 'ir.actions.act_window_close'},
            }
        }

    @check_identity
    def totp_enable_wizard(self):
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
            'name': _("Enable Two-Factor Authentication"),
            'res_id': w.id,
            'views': [(False, 'form')],
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
        for user in self.filtered('id'):
            self.env.cr.execute('SELECT totp_secret FROM res_users WHERE id=%s', (user.id,))
            user.totp_secret = self.env.cr.fetchone()[0]

    def _inverse_totp_secret(self):
        for user in self.filtered('id'):
            secret = user.totp_secret if user.totp_secret else None
            self.env.cr.execute('UPDATE res_users SET totp_secret = %s WHERE id=%s', (secret, user.id))

    def _search_totp_enable(self, operator, value):
        value = not value if operator == '!=' else value
        if value:
            self.env.cr.execute("SELECT id FROM res_users WHERE totp_secret IS NOT NULL")
        else:
            self.env.cr.execute("SELECT id FROM res_users WHERE totp_secret IS NULL OR totp_secret='false'")
        result = self.env.cr.fetchall()
        return [('id', 'in', [x[0] for x in result])]

class TOTPWizard(models.TransientModel):
    _name = 'auth_totp.wizard'
    _description = "Two-Factor Setup Wizard"

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
            import qrcode
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
                    'message': _("Two-factor authentication is now enabled."),
                    'next': {'type': 'ir.actions.act_window_close'},
                }
            }
        raise UserError(_('Verification failed, please double-check the 6-digit code'))

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

    def match(self, code, t=None, window=TIMESTEP):
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

        low = int((t - window) / TIMESTEP)
        high = int((t + window) / TIMESTEP) + 1

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
from . import ir_http
from . import res_users

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
</odoo>

```

## File: views\templates.xml

```xml
<odoo>
    <template id="assets_tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/auth_totp/static/tests/totp_flow.js"></script>
        </xpath>
    </template>
    <template id="auth_totp_form" name="Two-Factor Authentication">
        <t t-call="web.login_layout">
            <t t-set="disable_footer">1</t>
            <div class="oe_login_form">
                <h5 class="card-title">Two-factor Authentication</h5>
                <form method="POST" action="" class="">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <input type="hidden" name="redirect" t-att-value="redirect"/>
                    <div class="form-group">
                        <label for="totp_token">Authentication Code (6 digits)</label>
                        <input id="totp_token" name="totp_token" class="form-control mb-2"
                               autofocus="autofocus" required="required"/>
                    </div>
                    <p class="alert alert-danger" t-if="error" role="alert">
                        <t t-esc="error"/>
                    </p>
                    <div class="mb-2 mt-2 text-muted">
                        <input type="checkbox" name="remember" id="switch-remember" value="1"/>
                        <label for="switch-remember">Don't ask again on this device</label>
                    </div>
                    <div t-attf-class="clearfix oe_login_buttons text-center mb-1">
                        <button type="submit" class="btn btn-primary btn-block">
                            Verify
                        </button>
                    </div>
                    <div class="small mb-2 mt-2 text-muted">
                        <i class="fa fa-2x fa-mobile pull-left"/>
                        Open the two-factor authentication app on your
                        device to obtain a code and verify your identity
                    </div>
                </form>
            </div>
            <div class="text-center pb-2 border-top">
                <form method="POST" action="/web/session/logout" class="form-inline">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <button type="submit" class="btn btn-link btn-sm mb-2">
                        Cancel
                    </button>
                </form>
            </div>
        </t>
    </template>
</odoo>

```

## File: views\user_preferences.xml

```xml
<odoo>
    <record model="ir.ui.view" id="view_totp_list">
        <field name="name">users list: add totp status</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_tree"/>
        <field name="arch" type="xml">
            <tree>
                <field name="totp_enabled"/>
            </tree>
        </field>
    </record>
    <record model="ir.ui.view" id="view_totp_form">
        <field name="name">user form: add totp status</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='references']/group[1]" position="before">
                <field name="totp_enabled" invisible="1"/>
                <group attrs="{'invisible': [('totp_enabled', '!=', False)]}">
                    <div>
                        <span class="alert alert-info" role="status">
                            <i class="fa fa-warning"/>
                            Two-factor authentication not enabled
                        </span>
                    </div>
                </group>
                <group attrs="{'invisible': [('totp_enabled', '=', False)]}">
                    <div>
                        <span class="text-success">
                            <i class="fa fa-check-circle"/>
                            Two-factor authentication enabled
                        </span>
                    </div>
                </group>
            </xpath>
        </field>
    </record>

    <record model="ir.actions.server" id="action_disable_totp">
        <field name="name">Disable TOTP on users</field>
        <field name="model_id" ref="base.model_res_users"/>
        <field name="binding_model_id" ref="base.model_res_users"/>
        <field name="state">code</field>
        <field name="code">
            action = records.totp_disable()
        </field>
        <field name="groups_id" eval="[(4, ref('base.group_erp_manager'), 0)]"/>
    </record>

    <record model="ir.ui.view" id="view_totp_wizard">
        <field name="name">auth_totp wizard</field>
        <field name="model">auth_totp.wizard</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <div class="row container">
                        <div class="mb-3">
                            <h3 class="font-weight-bold">Scan this barcode with your app</h3>
                            <div>
                                Scan the image below with the authenticator app on your phone.<br/>
                                If you cannot scan the barcode, here are some alternative options:
                                <ul>
                                    <li><field class="text-wrap" name="url" widget="url"
                                               options="{'website_path': True}"
                                               text="Click on this link to open your authenticator app"/></li>

                                    <li>Or enter the secret code manually:
                                        <a data-toggle="collapse"
                                           href="#collapseTotpSecret" role="button" aria-expanded="false"
                                           aria-controls="collapseTotpSecret">show the code</a>
                                    </li>
                                </ul>
                                <!-- code outside list to have more horiz space on mobile -->
                                <div class="collapse col-12 col-md-6" id="collapseTotpSecret">
                                  <div class="card card-body">
                                    <h3>Your two-factor secret:</h3>
                                    <code class="text-center"><field name="secret"/></code>
                                  </div>
                                </div>
                            </div>

                            <field class="offset-1" name="qrcode" readonly="True" widget="image"/>

                            <h3 class="font-weight-bold">Enter the 6-digit code from your app</h3>
                            <div class="text-justify col-10 col-lg-6 px-0">
                                After scanning the barcode, the app will display a 6-digit code that you
                                should enter below. Don't worry if the code changes in the app,
                                it stays valid a bit longer.
                            </div>
                            <div class="mt-2">
                                <label for="code" class="col-4 col-md-12 px-0">Verification Code</label>
                                <field required="True" name="code" class="col-10 col-md-6 px-0"/>
                            </div>

                        </div>
                    </div>
                </sheet>
                <footer>
                    <button type="object" name="enable" class="btn btn-primary"
                            string="Enable two-factor authentication"/>
                    <button string="Cancel" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="view_totp_field">
        <field name="name">users preference: totp</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form_simple_modif"/>
        <field name="arch" type="xml">
            <button name="preference_change_password" position="after">
                    <field name="totp_enabled" invisible="1"/>
                    <group attrs="{'invisible': [('totp_enabled', '!=', False)]}">
                        <div>
                            <span class="alert alert-info" role="status">
                                <i class="fa fa-warning"/>
                                Two-factor authentication not enabled
                                <a href="https://www.odoo.com/documentation/14.0/applications/general/auth/2fa.html"
                                   title="What is this?" class="o_doc_link" target="_blank"></a>
                            </span>
                            <button name="totp_enable_wizard" type="object" string="Enable two-factor authentication"
                                    class="btn btn-info mx-3"/>
                        </div>
                    </group>
                    <group attrs="{'invisible': [('totp_enabled', '=', False)]}">
                        <div>
                            <span class="text-success">
                                <i class="fa fa-check-circle"/>
                                Two-factor authentication enabled
                                <a href="https://www.odoo.com/documentation/14.0/applications/general/auth/2fa.html"
                                   title="What is this?" class="o_doc_link" target="_blank"></a>
                            </span>
                            <button name="totp_disable" type="object" string="(Disable two-factor authentication)"
                                    class="btn btn-link text-muted"/>
                        </div>
                        <div colspan="2" attrs="{'invisible': [('totp_trusted_device_ids', '=', [])]}">
                            <field name="totp_trusted_device_ids" nolabel="1" colspan="4" readonly="1">

                                <tree create="false" delete="false">
                                    <field name="name" string="Trusted Devices"/>
                                    <field name="create_date" string="Added On"/>
                                    <button type="object" name="remove" icon="fa-trash"/>
                                </tree>
                                <form string="Trusted Device">
                                    <group>
                                        <group>
                                            <field name="name" string="Device Name"/>
                                            <field name="create_date" string="Added On"/>
                                        </group>
                                    </group>
                                    <footer>
                                        <button name="remove" string="Revoke" type="object" icon="fa-trash"/>
                                        <button name="preference_cancel" string="Cancel" special="cancel" class="btn-secondary"/>
                                    </footer>
                                </form>

                            </field>
                            <button name="revoke_all_devices" string="Revoke All" type="object" class="btn btn-secondary"
                                    confirm="Are you sure? Two-factor authentication will be required again on all your devices"/>
                        </div>
                    </group>
            </button>
        </field>
    </record>
</odoo>

```

