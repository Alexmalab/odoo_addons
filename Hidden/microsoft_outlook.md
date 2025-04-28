# Odoo Module: microsoft_outlook

Category: Hidden

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
    "name": "Microsoft Outlook",
    "version": "1.0",
    "category": "Hidden",
    "description": "Outlook support for outgoing mail servers",
    "depends": [
        "mail",
    ],
    "data": [
        "views/ir_mail_server_views.xml",
        "views/res_config_settings_views.xml",
        "views/templates.xml",
    ],
    "auto_install": False,
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import werkzeug

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import UserError
from odoo.http import request
from odoo.tools import consteq

_logger = logging.getLogger(__name__)


class MicrosoftOutlookController(http.Controller):
    @http.route('/microsoft_outlook/confirm', type='http', auth='user')
    def microsoft_outlook_callback(self, code=None, state=None, error_description=None, **kwargs):
        """Callback URL during the OAuth process.

        Outlook redirects the user browser to this endpoint with the authorization code.
        We will fetch the refresh token and the access token thanks to this authorization
        code and save those values on the given mail server.
        """
        if not request.env.user.has_group('base.group_system'):
            _logger.error('Microsoft Outlook: Non system user try to link an Outlook account.')
            raise Forbidden()

        try:
            state = json.loads(state)
            model_name = state['model']
            rec_id = state['id']
            csrf_token = state['csrf_token']
        except Exception:
            _logger.error('Microsoft Outlook: Wrong state value %r.', state)
            raise Forbidden()

        if error_description:
            return request.render('microsoft_outlook.microsoft_outlook_oauth_error', {
                'error': error_description,
                'model_name': model_name,
                'rec_id': rec_id,
            })

        model = request.env[model_name]

        if not issubclass(type(model), request.env.registry['microsoft.outlook.mixin']):
            # The model must inherits from the "microsoft.outlook.mixin" mixin
            raise Forbidden()

        record = model.browse(rec_id).exists()
        if not record:
            raise Forbidden()

        if not csrf_token or not consteq(csrf_token, record._get_outlook_csrf_token()):
            _logger.error('Microsoft Outlook: Wrong CSRF token during Outlook authentication.')
            raise Forbidden()

        try:
            refresh_token, access_token, expiration = record._fetch_outlook_refresh_token(code)
        except UserError as e:
            return request.render('microsoft_outlook.microsoft_outlook_oauth_error', {
                'error': str(e.name),
                'model_name': model_name,
                'rec_id': rec_id,
            })

        record.write({
            'microsoft_outlook_refresh_token': refresh_token,
            'microsoft_outlook_access_token': access_token,
            'microsoft_outlook_access_token_expiration': expiration,
        })

        return werkzeug.utils.redirect(f'/web?#id={rec_id}&model={model_name}&view_type=form', 303)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\ir_mail_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import _, api, models
from odoo.exceptions import UserError


class IrMailServer(models.Model):
    """Add the Outlook OAuth authentication on the outgoing mail servers."""

    _name = 'ir.mail_server'
    _inherit = ['ir.mail_server', 'microsoft.outlook.mixin']

    _OUTLOOK_SCOPE = 'https://outlook.office.com/SMTP.Send'

    @api.constrains('use_microsoft_outlook_service', 'smtp_pass', 'smtp_encryption')
    def _check_use_microsoft_outlook_service(self):
        for server in self:
            if not server.use_microsoft_outlook_service:
                continue

            if server.smtp_pass:
                raise UserError(_(
                    'Please leave the password field empty for Outlook mail server %r. '
                    'The OAuth process does not require it')
                    % server.name)

            if server.smtp_encryption != 'starttls':
                raise UserError(_(
                    'Incorrect Connection Security for Outlook mail server %r. '
                    'Please set it to "TLS (STARTTLS)".')
                    % server.name)

    @api.onchange('smtp_encryption')
    def _onchange_encryption(self):
        """Do not change the SMTP configuration if it's a Outlook server

        (e.g. the port which is already set)"""
        if not self.use_microsoft_outlook_service:
            super()._onchange_encryption()

    @api.onchange('use_microsoft_outlook_service')
    def _onchange_use_microsoft_outlook_service(self):
        if self.use_microsoft_outlook_service:
            self.smtp_host = 'smtp.outlook.com'
            self.smtp_encryption = 'starttls'
            self.smtp_port = 587
        else:
            self.microsoft_outlook_refresh_token = False
            self.microsoft_outlook_access_token = False
            self.microsoft_outlook_access_token_expiration = False

    def _smtp_login(self, connection, smtp_user, smtp_password):
        if len(self) == 1 and self.use_microsoft_outlook_service:
            auth_string = self._generate_outlook_oauth2_string(smtp_user)
            oauth_param = base64.b64encode(auth_string.encode()).decode()
            connection.ehlo()
            connection.docmd('AUTH', 'XOAUTH2 %s' % oauth_param)
        else:
            super()._smtp_login(connection, smtp_user, smtp_password)

```

## File: models\microsoft_outlook_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import hmac
import json
import logging
import time
import requests

from werkzeug.urls import url_encode, url_join

from odoo import _, api, fields, models
from odoo.exceptions import AccessError, UserError

_logger = logging.getLogger(__name__)


class MicrosoftOutlookMixin(models.AbstractModel):

    _name = 'microsoft.outlook.mixin'
    _description = 'Microsoft Outlook Mixin'

    _OUTLOOK_SCOPE = None
    _OUTLOOK_ENDPOINT = 'https://login.microsoftonline.com/common/oauth2/v2.0/'

    use_microsoft_outlook_service = fields.Boolean('Outlook Authentication')
    is_microsoft_outlook_configured = fields.Boolean('Is Outlook Credential Configured',
        compute='_compute_is_microsoft_outlook_configured')
    microsoft_outlook_refresh_token = fields.Char(string='Outlook Refresh Token',
        groups='base.group_system', copy=False)
    microsoft_outlook_access_token = fields.Char(string='Outlook Access Token',
        groups='base.group_system', copy=False)
    microsoft_outlook_access_token_expiration = fields.Integer(string='Outlook Access Token Expiration Timestamp',
        groups='base.group_system', copy=False)
    microsoft_outlook_uri = fields.Char(compute='_compute_outlook_uri', string='Authentication URI',
        help='The URL to generate the authorization code from Outlook', groups='base.group_system')

    @api.depends('use_microsoft_outlook_service')
    def _compute_is_microsoft_outlook_configured(self):
        Config = self.env['ir.config_parameter'].sudo()
        microsoft_outlook_client_id = Config.get_param('microsoft_outlook_client_id')
        microsoft_outlook_client_secret = Config.get_param('microsoft_outlook_client_secret')
        self.is_microsoft_outlook_configured = microsoft_outlook_client_id and microsoft_outlook_client_secret

    @api.depends('use_microsoft_outlook_service')
    def _compute_outlook_uri(self):
        Config = self.env['ir.config_parameter'].sudo()
        base_url = Config.get_param('web.base.url')
        microsoft_outlook_client_id = Config.get_param('microsoft_outlook_client_id')

        for record in self:
            if not record.id or not record.use_microsoft_outlook_service or not record.is_microsoft_outlook_configured:
                record.microsoft_outlook_uri = False
                continue

            record.microsoft_outlook_uri = url_join(self._OUTLOOK_ENDPOINT, 'authorize?%s' % url_encode({
                'client_id': microsoft_outlook_client_id,
                'response_type': 'code',
                'redirect_uri': url_join(base_url, '/microsoft_outlook/confirm'),
                'response_mode': 'query',
                # offline_access is needed to have the refresh_token
                'scope': 'offline_access %s' % self._OUTLOOK_SCOPE,
                'state': json.dumps({
                    'model': record._name,
                    'id': record.id,
                    'csrf_token': record._get_outlook_csrf_token(),
                })
            }))

    def open_microsoft_outlook_uri(self):
        """Open the URL to accept the Outlook permission.

        This is done with an action, so we can force the user the save the form.
        We need him to save the form so the current mail server record exist in DB, and
        we can include the record ID in the URL.
        """
        self.ensure_one()

        if not self.env.user.has_group('base.group_system'):
            raise AccessError(_('Only the administrator can link an Outlook mail server.'))

        if not self.use_microsoft_outlook_service or not self.is_microsoft_outlook_configured:
            raise UserError(_('Please configure your Outlook credentials.'))

        return {
            'type': 'ir.actions.act_url',
            'url': self.microsoft_outlook_uri,
        }

    def _fetch_outlook_refresh_token(self, authorization_code):
        """Request the refresh token and the initial access token from the authorization code.

        :return:
            refresh_token, access_token, access_token_expiration
        """
        response = self._fetch_outlook_token('authorization_code', code=authorization_code)
        return (
            response['refresh_token'],
            response['access_token'],
            int(time.time()) + response['expires_in'],
        )

    def _fetch_outlook_access_token(self, refresh_token):
        """Refresh the access token thanks to the refresh token.

        :return:
            access_token, access_token_expiration
        """
        response = self._fetch_outlook_token('refresh_token', refresh_token=refresh_token)
        return (
            response['access_token'],
            int(time.time()) + response['expires_in'],
        )

    def _fetch_outlook_token(self, grant_type, **values):
        """Generic method to request an access token or a refresh token.

        Return the JSON response of the Outlook API and manage the errors which can occur.

        :param grant_type: Depends the action we want to do (refresh_token or authorization_code)
        :param values: Additional parameters that will be given to the Outlook endpoint
        """
        Config = self.env['ir.config_parameter'].sudo()
        base_url = Config.get_param('web.base.url')
        microsoft_outlook_client_id = Config.get_param('microsoft_outlook_client_id')
        microsoft_outlook_client_secret = Config.get_param('microsoft_outlook_client_secret')

        response = requests.post(
            url_join(self._OUTLOOK_ENDPOINT, 'token'),
            data={
                'client_id': microsoft_outlook_client_id,
                'client_secret': microsoft_outlook_client_secret,
                'scope': 'offline_access %s' % self._OUTLOOK_SCOPE,
                'redirect_uri': url_join(base_url, '/microsoft_outlook/confirm'),
                'grant_type': grant_type,
                **values,
            },
            timeout=10,
        )

        if not response.ok:
            try:
                error_description = response.json()['error_description']
            except Exception:
                error_description = _('Unknown error.')
            raise UserError(_('An error occurred when fetching the access token. %s') % error_description)

        return response.json()

    def _generate_outlook_oauth2_string(self, login):
        """Generate a OAuth2 string which can be used for authentication.

        :param user: Email address of the Outlook account to authenticate
        :return: The SASL argument for the OAuth2 mechanism.
        """
        self.ensure_one()
        now_timestamp = int(time.time())
        if not self.microsoft_outlook_access_token \
           or not self.microsoft_outlook_access_token_expiration \
           or self.microsoft_outlook_access_token_expiration < now_timestamp:
            if not self.microsoft_outlook_refresh_token:
                raise UserError(_('Please login your Outlook mail server before using it.'))
            (
                self.microsoft_outlook_access_token,
                self.microsoft_outlook_access_token_expiration,
            ) = self._fetch_outlook_access_token(self.microsoft_outlook_refresh_token)
            _logger.info(
                'Microsoft Outlook: fetch new access token. It expires in %i minutes',
                (self.microsoft_outlook_access_token_expiration - now_timestamp) // 60)
        else:
            _logger.info(
                'Microsoft Outlook: reuse existing access token. It expires in %i minutes',
                (self.microsoft_outlook_access_token_expiration - now_timestamp) // 60)

        return 'user=%s\1auth=Bearer %s\1\1' % (login, self.microsoft_outlook_access_token)

    def _get_outlook_csrf_token(self):
        """Generate a CSRF token that will be verified in `microsoft_outlook_callback`.

        This will prevent a malicious person to make an admin user disconnect the mail servers.
        """
        self.ensure_one()
        _logger.info('Microsoft Outlook: generate CSRF token for %s #%i', self._name, self.id)
        secret = self.env['ir.config_parameter'].sudo().get_param('database.secret')
        data = str((self.env.cr.dbname, 'microsoft_outlook_oauth', self._name, self.id))
        return hmac.new(secret.encode('utf-8'), data.encode('utf-8'), hashlib.sha256).hexdigest()

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    microsoft_outlook_client_identifier = fields.Char('Outlook Client Id', config_parameter='microsoft_outlook_client_id')
    microsoft_outlook_client_secret = fields.Char('Outlook Client Secret', config_parameter='microsoft_outlook_client_secret')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import microsoft_outlook_mixin

from . import ir_mail_server
from . import res_config_settings

```

## File: views\ir_mail_server_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_mail_server_view_form" model="ir.ui.view">
        <field name="name">ir.mail_server.view.form.inherit.outlook</field>
        <field name="model">ir.mail_server</field>
        <field name="inherit_id" ref="base.ir_mail_server_form"/>
        <field name="arch" type="xml">
            <field name="smtp_host" position="before">
                <field name="use_microsoft_outlook_service" string="Outlook"/>
            </field>
            <field name="smtp_user" position="after">
                <field name="is_microsoft_outlook_configured" invisible="1"/>
                <field name="microsoft_outlook_refresh_token" invisible="1"/>
                <field name="microsoft_outlook_access_token" invisible="1"/>
                <field name="microsoft_outlook_access_token_expiration" invisible="1"/>
                <div></div>
                <div attrs="{'invisible': [('use_microsoft_outlook_service', '=', False)]}">
                    <span attrs="{'invisible': ['|', ('use_microsoft_outlook_service', '=', False), ('microsoft_outlook_refresh_token', '=', False)]}"
                        class="badge badge-success">
                        Outlook Token Valid
                    </span>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', '|', ('is_microsoft_outlook_configured', '=', False), ('use_microsoft_outlook_service', '=', False), ('microsoft_outlook_refresh_token', '!=', False)]}">
                        <i class="fa fa-arrow-right"/>
                        Connect your Outlook account
                    </button>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', '|', ('is_microsoft_outlook_configured', '=', False), ('use_microsoft_outlook_service', '=', False), ('microsoft_outlook_refresh_token', '=', False)]}">
                        <i class="fa fa-cog"/>
                        Edit Settings
                    </button>
                    <div class="alert alert-warning" role="alert"
                        attrs="{'invisible': ['|', ('is_microsoft_outlook_configured', '=', True), ('use_microsoft_outlook_service', '=', False)]}">
                        Setup your Outlook API credentials in the general settings to link a Outlook account.
                    </div>
                </div>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.microsoft_outlook</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <div id="emails" position="inside">
                    <div class="col-12 col-lg-6 o_setting_box" id="microsoft_outlook_setting"
                        attrs="{'invisible': [('external_email_server_default', '=', False)]}">
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Outlook Credentials</span>
                            <div class="text-muted">
                                Send and receive email with your Outlook account.
                            </div>
                            <div class="content-group">
                                <div class="row mt16" id="outlook_client_identifier">
                                    <label string="Client ID" for="microsoft_outlook_client_identifier"
                                        class="col-lg-3 o_light_label"/>
                                    <field name="microsoft_outlook_client_identifier" class="ml-2"/>
                                </div>
                                <div class="row mt16" id="outlook_client_secret">
                                    <label string="Client Secret" for="microsoft_outlook_client_secret"
                                        class="col-lg-3 o_light_label"/>
                                    <field name="microsoft_outlook_client_secret" password="True" class="ml-2"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="microsoft_outlook_oauth_error">
        <t t-call-assets="web.assets_frontend" t-js="false"/>
        <div class="py-5">
            <div class="alert alert-warning w-50 mx-auto" role="alert">
                <t t-esc="error"/>
                <br/>
                <a t-attf-href="/web?#id=#{rec_id}&amp;model=#{model_name}&amp;view_type=form">
                    Go back to your mail server
                </a>
            </div>
        </div>
    </template>
</odoo>

```

