# Odoo Module: auth_oauth

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'OAuth2 Authentication',
    'category': 'Hidden/Tools',
    'description': """
Allow users to login through OAuth2 Provider.
=============================================
""",
    'maintainer': 'Odoo S.A.',
    'depends': ['base', 'web', 'base_setup', 'auth_signup'],
    'data': [
        'data/auth_oauth_data.xml',
        'views/auth_oauth_views.xml',
        'views/res_users_views.xml',
        'views/res_config_settings_views.xml',
        'views/auth_oauth_templates.xml',
        'security/ir.model.access.csv',
    ],
    'assets': {
        'web.assets_frontend': [
            'auth_oauth/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import functools
import json
import logging
import os

import werkzeug.urls
import werkzeug.utils
from werkzeug.exceptions import BadRequest

from odoo import api, http, SUPERUSER_ID, _
from odoo.exceptions import AccessDenied
from odoo.http import request
from odoo import registry as registry_get
from odoo.tools.misc import clean_context

from odoo.addons.auth_signup.controllers.main import AuthSignupHome as Home
from odoo.addons.web.controllers.main import db_monodb, ensure_db, set_cookie_and_redirect, login_and_redirect


_logger = logging.getLogger(__name__)


#----------------------------------------------------------
# helpers
#----------------------------------------------------------
def fragment_to_query_string(func):
    @functools.wraps(func)
    def wrapper(self, *a, **kw):
        kw.pop('debug', False)
        if not kw:
            return """<html><head><script>
                var l = window.location;
                var q = l.hash.substring(1);
                var r = l.pathname + l.search;
                if(q.length !== 0) {
                    var s = l.search ? (l.search === '?' ? '' : '&') : '?';
                    r = l.pathname + l.search + s + q;
                }
                if (r == l.pathname) {
                    r = '/';
                }
                window.location = r;
            </script></head><body></body></html>"""
        return func(self, *a, **kw)
    return wrapper


#----------------------------------------------------------
# Controller
#----------------------------------------------------------
class OAuthLogin(Home):
    def list_providers(self):
        try:
            providers = request.env['auth.oauth.provider'].sudo().search_read([('enabled', '=', True)])
        except Exception:
            providers = []
        for provider in providers:
            return_url = request.httprequest.url_root + 'auth_oauth/signin'
            state = self.get_state(provider)
            params = dict(
                response_type='token',
                client_id=provider['client_id'],
                redirect_uri=return_url,
                scope=provider['scope'],
                state=json.dumps(state),
                # nonce=base64.urlsafe_b64encode(os.urandom(16)),
            )
            provider['auth_link'] = "%s?%s" % (provider['auth_endpoint'], werkzeug.urls.url_encode(params))
        return providers

    def get_state(self, provider):
        redirect = request.params.get('redirect') or 'web'
        if not redirect.startswith(('//', 'http://', 'https://')):
            redirect = '%s%s' % (request.httprequest.url_root, redirect[1:] if redirect[0] == '/' else redirect)
        state = dict(
            d=request.session.db,
            p=provider['id'],
            r=werkzeug.urls.url_quote_plus(redirect),
        )
        token = request.params.get('token')
        if token:
            state['t'] = token
        return state

    @http.route()
    def web_login(self, *args, **kw):
        ensure_db()
        if request.httprequest.method == 'GET' and request.session.uid and request.params.get('redirect'):
            # Redirect if already logged in and redirect param is present
            return request.redirect(request.params.get('redirect'))
        providers = self.list_providers()

        response = super(OAuthLogin, self).web_login(*args, **kw)
        if response.is_qweb:
            error = request.params.get('oauth_error')
            if error == '1':
                error = _("Sign up is not allowed on this database.")
            elif error == '2':
                error = _("Access Denied")
            elif error == '3':
                error = _("You do not have access to this database or your invitation has expired. Please ask for an invitation and be sure to follow the link in your invitation email.")
            else:
                error = None

            response.qcontext['providers'] = providers
            if error:
                response.qcontext['error'] = error

        return response

    def get_auth_signup_qcontext(self):
        result = super(OAuthLogin, self).get_auth_signup_qcontext()
        result["providers"] = self.list_providers()
        return result


class OAuthController(http.Controller):

    @http.route('/auth_oauth/signin', type='http', auth='none')
    @fragment_to_query_string
    def signin(self, **kw):
        state = json.loads(kw['state'])
        dbname = state['d']
        if not http.db_filter([dbname]):
            return BadRequest()
        provider = state['p']
        context = clean_context(state.get('c', {}))
        registry = registry_get(dbname)
        with registry.cursor() as cr:
            try:
                env = api.Environment(cr, SUPERUSER_ID, context)
                credentials = env['res.users'].sudo().auth_oauth(provider, kw)
                cr.commit()
                action = state.get('a')
                menu = state.get('m')
                redirect = werkzeug.urls.url_unquote_plus(state['r']) if state.get('r') else False
                url = '/web'
                if redirect:
                    url = redirect
                elif action:
                    url = '/web#action=%s' % action
                elif menu:
                    url = '/web#menu_id=%s' % menu
                resp = login_and_redirect(*credentials, redirect_url=url)
                # Since /web is hardcoded, verify user has right to land on it
                if werkzeug.urls.url_parse(resp.location).path == '/web' and not request.env.user.has_group('base.group_user'):
                    resp.location = '/'
                return resp
            except AttributeError:
                # auth_signup is not installed
                _logger.error("auth_signup not installed on database %s: oauth sign up cancelled." % (dbname,))
                url = "/web/login?oauth_error=1"
            except AccessDenied:
                # oauth credentials not valid, user could be on a temporary session
                _logger.info('OAuth2: access denied, redirect to main page in case a valid session exists, without setting cookies')
                url = "/web/login?oauth_error=3"
                redirect = request.redirect(url, 303)
                redirect.autocorrect_location_header = False
                return redirect
            except Exception as e:
                # signup error
                _logger.exception("OAuth2: %s" % str(e))
                url = "/web/login?oauth_error=2"

        return set_cookie_and_redirect(url)

    @http.route('/auth_oauth/oea', type='http', auth='none')
    def oea(self, **kw):
        """login user via Odoo Account provider"""
        dbname = kw.pop('db', None)
        if not dbname:
            dbname = db_monodb()
        if not dbname:
            return BadRequest()
        if not http.db_filter([dbname]):
            return BadRequest()

        registry = registry_get(dbname)
        with registry.cursor() as cr:
            try:
                env = api.Environment(cr, SUPERUSER_ID, {})
                provider = env.ref('auth_oauth.provider_openerp')
            except ValueError:
                return set_cookie_and_redirect('/web?db=%s' % dbname)
            assert provider._name == 'auth.oauth.provider'

        state = {
            'd': dbname,
            'p': provider.id,
            'c': {'no_user_creation': True},
        }

        kw['state'] = json.dumps(state)
        return self.signin(**kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\auth_oauth_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="provider_openerp" model="auth.oauth.provider">
            <field name="name">Odoo.com Accounts</field>
            <field name="auth_endpoint">https://accounts.odoo.com/oauth2/auth</field>
            <field name="scope">userinfo</field>
            <field name="validation_endpoint">https://accounts.odoo.com/oauth2/tokeninfo</field>
            <field name="css_class">fa fa-fw o_custom_icon</field>
            <field name="body">Log in with Odoo.com</field>
            <field name="enabled" eval="True"/>
        </record>
        <record id="provider_facebook" model="auth.oauth.provider">
            <field name="name">Facebook Graph</field>
            <field name="auth_endpoint">https://www.facebook.com/dialog/oauth</field>
            <field name="scope">public_profile,email</field>
            <field name="validation_endpoint">https://graph.facebook.com/me</field>
            <field name="data_endpoint">https://graph.facebook.com/me?fields=id,name,email</field>
            <field name="css_class">fa fa-fw fa-facebook-square</field>
            <field name="body">Log in with Facebook</field>
        </record>
        <record id="provider_google" model="auth.oauth.provider">
            <field name="name">Google OAuth2</field>
            <field name="auth_endpoint">https://accounts.google.com/o/oauth2/auth</field>
            <field name="scope">openid profile email</field>
            <field name="validation_endpoint">https://www.googleapis.com/oauth2/v3/userinfo</field>
            <field name="css_class">fa fa-fw fa-google</field>
            <field name="body">Log in with Google</field>
        </record>

        <!-- Use database uuid as client_id for OpenERP oauth provider -->
        <function model="auth.oauth.provider" name="write">
            <value eval="[ref('auth_oauth.provider_openerp')]"/>
            <value model="ir.config_parameter" eval="{
                'client_id': obj().env['ir.config_parameter'].get_param('database.uuid'),
            }"/>
        </function>
    </data>
</odoo>

```

## File: models\auth_oauth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AuthOAuthProvider(models.Model):
    """Class defining the configuration values of an OAuth2 provider"""

    _name = 'auth.oauth.provider'
    _description = 'OAuth2 provider'
    _order = 'sequence, name'

    name = fields.Char(string='Provider name', required=True)  # Name of the OAuth2 entity, Google, etc
    client_id = fields.Char(string='Client ID')  # Our identifier
    auth_endpoint = fields.Char(string='Authorization URL', required=True)  # OAuth provider URL to authenticate users
    scope = fields.Char(default='openid profile email')  # OAUth user data desired to access
    validation_endpoint = fields.Char(string='UserInfo URL', required=True)  # OAuth provider URL to get user information
    data_endpoint = fields.Char()
    enabled = fields.Boolean(string='Allowed')
    css_class = fields.Char(string='CSS class', default='fa fa-fw fa-sign-in text-primary')
    body = fields.Char(required=True, string="Login button label", help='Link text in Login Dialog', translate=True)
    sequence = fields.Integer(default=10)

```

## File: models\ir_config_parameter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrConfigParameter(models.Model):
    _inherit = 'ir.config_parameter'

    def init(self, force=False):
        super(IrConfigParameter, self).init(force=force)
        if force:
            oauth_oe = self.env.ref('auth_oauth.provider_openerp')
            if not oauth_oe:
                return
            dbuuid = self.sudo().get_param('database.uuid')
            oauth_oe.write({'client_id': dbuuid})

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models

_logger = logging.getLogger(__name__)


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    @api.model
    def get_uri(self):
        return "%s/auth_oauth/signin" % (self.env['ir.config_parameter'].get_param('web.base.url'))

    auth_oauth_google_enabled = fields.Boolean(string='Allow users to sign in with Google')
    auth_oauth_google_client_id = fields.Char(string='Client ID')
    server_uri_google = fields.Char(string='Server uri')

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        google_provider = self.env.ref('auth_oauth.provider_google', False)
        google_provider and res.update(
            auth_oauth_google_enabled=google_provider.enabled,
            auth_oauth_google_client_id=google_provider.client_id,
            server_uri_google=self.get_uri(),
        )
        return res

    def set_values(self):
        super(ResConfigSettings, self).set_values()
        google_provider = self.env.ref('auth_oauth.provider_google', False)
        google_provider and google_provider.write({
            'enabled': self.auth_oauth_google_enabled,
            'client_id': self.auth_oauth_google_client_id,
        })

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

import requests
import werkzeug.http

from odoo import api, fields, models
from odoo.exceptions import AccessDenied, UserError
from odoo.addons.auth_signup.models.res_users import SignupError

from odoo.addons import base
base.models.res_users.USER_PRIVATE_FIELDS.append('oauth_access_token')

class ResUsers(models.Model):
    _inherit = 'res.users'

    oauth_provider_id = fields.Many2one('auth.oauth.provider', string='OAuth Provider')
    oauth_uid = fields.Char(string='OAuth User ID', help="Oauth Provider user_id", copy=False)
    oauth_access_token = fields.Char(string='OAuth Access Token', readonly=True, copy=False, prefetch=False)

    _sql_constraints = [
        ('uniq_users_oauth_provider_oauth_uid', 'unique(oauth_provider_id, oauth_uid)', 'OAuth UID must be unique per provider'),
    ]

    def _auth_oauth_rpc(self, endpoint, access_token):
        if self.env['ir.config_parameter'].sudo().get_param('auth_oauth.authorization_header'):
            response = requests.get(endpoint, headers={'Authorization': 'Bearer %s' % access_token}, timeout=10)
        else:
            response = requests.get(endpoint, params={'access_token': access_token}, timeout=10)

        if response.ok: # nb: could be a successful failure
            return response.json()

        auth_challenge = werkzeug.http.parse_www_authenticate_header(
            response.headers.get('WWW-Authenticate'))
        if auth_challenge.type == 'bearer' and 'error' in auth_challenge:
            return dict(auth_challenge)

        return {'error': 'invalid_request'}

    @api.model
    def _auth_oauth_validate(self, provider, access_token):
        """ return the validation data corresponding to the access token """
        oauth_provider = self.env['auth.oauth.provider'].browse(provider)
        validation = self._auth_oauth_rpc(oauth_provider.validation_endpoint, access_token)
        if validation.get("error"):
            raise Exception(validation['error'])
        if oauth_provider.data_endpoint:
            data = self._auth_oauth_rpc(oauth_provider.data_endpoint, access_token)
            validation.update(data)
        # unify subject key, pop all possible and get most sensible. When this
        # is reworked, BC should be dropped and only the `sub` key should be
        # used (here, in _generate_signup_values, and in _auth_oauth_signin)
        subject = next(filter(None, [
            validation.pop(key, None)
            for key in [
                'sub', # standard
                'id', # google v1 userinfo, facebook opengraph
                'user_id', # google tokeninfo, odoo (tokeninfo)
            ]
        ]), None)
        if not subject:
            raise AccessDenied('Missing subject identity')
        validation['user_id'] = subject

        return validation

    @api.model
    def _generate_signup_values(self, provider, validation, params):
        oauth_uid = validation['user_id']
        email = validation.get('email', 'provider_%s_user_%s' % (provider, oauth_uid))
        name = validation.get('name', email)
        return {
            'name': name,
            'login': email,
            'email': email,
            'oauth_provider_id': provider,
            'oauth_uid': oauth_uid,
            'oauth_access_token': params['access_token'],
            'active': True,
        }

    @api.model
    def _auth_oauth_signin(self, provider, validation, params):
        """ retrieve and sign in the user corresponding to provider and validated access token
            :param provider: oauth provider id (int)
            :param validation: result of validation of access token (dict)
            :param params: oauth parameters (dict)
            :return: user login (str)
            :raise: AccessDenied if signin failed

            This method can be overridden to add alternative signin methods.
        """
        oauth_uid = validation['user_id']
        try:
            oauth_user = self.search([("oauth_uid", "=", oauth_uid), ('oauth_provider_id', '=', provider)])
            if not oauth_user:
                raise AccessDenied()
            assert len(oauth_user) == 1
            oauth_user.write({'oauth_access_token': params['access_token']})
            return oauth_user.login
        except AccessDenied as access_denied_exception:
            if self.env.context.get('no_user_creation'):
                return None
            state = json.loads(params['state'])
            token = state.get('t')
            values = self._generate_signup_values(provider, validation, params)
            try:
                _, login, _ = self.signup(values, token)
                return login
            except (SignupError, UserError):
                raise access_denied_exception

    @api.model
    def auth_oauth(self, provider, params):
        # Advice by Google (to avoid Confused Deputy Problem)
        # if validation.audience != OUR_CLIENT_ID:
        #   abort()
        # else:
        #   continue with the process
        access_token = params.get('access_token')
        validation = self._auth_oauth_validate(provider, access_token)

        # retrieve and sign in user
        login = self._auth_oauth_signin(provider, validation, params)
        if not login:
            raise AccessDenied()
        # return user credentials
        return (self.env.cr.dbname, login, access_token)

    def _check_credentials(self, password, env):
        try:
            return super(ResUsers, self)._check_credentials(password, env)
        except AccessDenied:
            passwd_allowed = env['interactive'] or not self.env.user._rpc_api_keys_only()
            if passwd_allowed and self.env.user.active:
                res = self.sudo().search([('id', '=', self.env.uid), ('oauth_access_token', '=', password)])
                if res:
                    return
            raise

    def _get_session_token_fields(self):
        return super(ResUsers, self)._get_session_token_fields() | {'oauth_access_token'}

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import auth_oauth
from . import res_config_settings
from . import ir_config_parameter
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_auth_oauth_provider,auth_oauth_provider,model_auth_oauth_provider,base.group_system,1,1,1,1

```

## File: views\auth_oauth_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="providers" name="OAuth Providers">
            <t t-if="len(providers) &gt; 0">
                <em t-attf-class="d-block text-center text-muted small my-#{len(providers) if len(providers) &lt; 3 else 3}">- or -</em>
                <div class="o_auth_oauth_providers list-group mt-1 mb-1 text-left">
                    <a t-foreach="providers" t-as="p" class="list-group-item list-group-item-action py-2" t-att-href="p['auth_link']">
                        <i t-att-class="p['css_class']"/>
                        <t t-esc="p['body']"/>
                    </a>
                </div>
            </t>
        </template>

        <template id="login" inherit_id="web.login" name="OAuth Login buttons">
            <xpath expr="//form" position="before">
                <t t-set="form_small" t-value="True" t-if="len(providers) &gt; 2"/>
            </xpath>
            <xpath expr="//div[hasclass('o_login_auth')]" position="inside">
                <t t-call="auth_oauth.providers"/>
            </xpath>
        </template>

        <template id="signup" inherit_id="auth_signup.signup" name="OAuth Signup buttons">
            <xpath expr="//form" position="before">
                <t t-set="form_small" t-value="True"/>
            </xpath>
            <xpath expr="//div[hasclass('o_login_auth')]" position="inside">
                <t t-call="auth_oauth.providers"/>
            </xpath>
        </template>

        <template id="reset_password" inherit_id="auth_signup.reset_password" name="OAuth Reset Password buttons">
            <xpath expr="//div[hasclass('o_login_auth')]" position="inside">
                <t t-call="auth_oauth.providers"/>
            </xpath>
        </template>
</odoo>

```

## File: views\auth_oauth_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="view_oauth_provider_form" model="ir.ui.view">
            <field name="name">auth.oauth.provider.form</field>
            <field name="model">auth.oauth.provider</field>
            <field name="arch" type="xml">
                <form string="arch">
                    <sheet>
                        <group>
                            <field name="name" />
                            <field name="client_id" />
                            <field name="enabled" />
                            <field name="body" />
                            <field name="css_class" groups="base.group_no_one" />
                        </group>
                        <group>
                            <field name="auth_endpoint" />
                            <field name="scope" />
                            <field name="validation_endpoint" />
                            <field name="data_endpoint" />
                        </group>
                    </sheet>
                </form>
            </field>
        </record>        
        <record id="view_oauth_provider_tree" model="ir.ui.view">
            <field name="name">auth.oauth.provider.tree</field>
            <field name="model">auth.oauth.provider</field>
            <field name="arch" type="xml">
                <tree string="arch">
                    <field name="sequence" widget="handle"/>
                    <field name="name" />
                    <field name="client_id" />
                    <field name="enabled" />
                </tree>
            </field>
        </record>
        <record id="action_oauth_provider" model="ir.actions.act_window">
            <field name="name">Providers</field>
            <field name="res_model">auth.oauth.provider</field>
            <field name="view_mode">tree,form</field>
        </record>
        <menuitem id="menu_oauth_providers" name="OAuth Providers"
            parent="base.menu_users" sequence="30"
            action="action_oauth_provider" groups="base.group_no_one"/>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.auth.oauth</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <div id="msg_module_auth_oauth" position="replace">
                    <div class="content-group" attrs="{'invisible': [('module_auth_oauth','=',False)]}">
                        <div class="mt8">
                            <button type="action" name="%(auth_oauth.action_oauth_provider)d" string="OAuth Providers" icon="fa-arrow-right" class="btn-link"/>
                        </div>
                    </div>
                </div>
                <div id="module_auth_oauth" position="after">
                    <div class="col-12 col-lg-6 o_setting_box"
                        id="signin_google_setting"
                        attrs="{'invisible': [('module_auth_oauth','=',False)]}">
                        <div class="o_setting_left_pane">
                            <field name="auth_oauth_google_enabled"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label string="Google Authentication" for="auth_oauth_google_enabled"/>
                            <a href="https://www.odoo.com/documentation/15.0/applications/general/auth/google.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                            <div class="text-muted">
                                Allow users to sign in with their Google account
                            </div>
                            <div class="content-group" attrs="{'invisible': [('auth_oauth_google_enabled','=',False)]}">
                                <div class="row mt16">
                                    <label for="auth_oauth_google_client_id" string="Client ID:" class="col-lg-3 o_light_label"/>
                                    <field name="auth_oauth_google_client_id" placeholder="e.g. 1234-xyz.apps.googleusercontent.com"/>
                                </div>
                                <a href="https://www.odoo.com/documentation/15.0/applications/general/auth/google.html" target="_blank"><i class="fa fa-fw fa-arrow-right"/>Tutorial</a>
                            </div>
                        </div>
                    </div>
                </div>
            </field>
        </record>
</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_users_form" model="ir.ui.view">
            <field name="name">res.users.form.inherit</field>
            <field name="model">res.users</field>
            <field name="type">form</field>
            <field name="inherit_id" ref="base.view_users_form"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='preferences']" position="after">
                    <page string="Oauth" name="oauth">
                        <group>
                            <field name="oauth_provider_id"/>
                            <field name="oauth_uid"/>
                            <field name="oauth_access_token"/>
                        </group>
                    </page>
                </xpath>
            </field>
        </record>
</odoo>

```

