# Odoo Module: google_gmail

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
    "name": "Google Gmail",
    "version": "1.2",
    "category": "Hidden",
    "description": "Gmail support for incoming / outgoing mail servers",
    "depends": [
        "mail",
    ],
    "data": [
        "views/fetchmail_server_views.xml",
        "views/ir_mail_server_views.xml",
        "views/res_config_settings_views.xml",
    ],
    "auto_install": True,
    "license": "LGPL-3",
    "assets": {
        "web.assets_backend": [
            "google_gmail/static/src/scss/google_gmail.scss",
        ]
    },
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging

from werkzeug.exceptions import Forbidden
from werkzeug.urls import url_encode

from odoo import _, http
from odoo.exceptions import UserError
from odoo.http import request
from odoo.tools import consteq

_logger = logging.getLogger(__name__)


class GoogleGmailController(http.Controller):
    @http.route('/google_gmail/confirm', type='http', auth='user')
    def google_gmail_callback(self, code=None, state=None, error=None, **kwargs):
        """Callback URL during the OAuth process.

        Gmail redirects the user browser to this endpoint with the authorization code.
        We will fetch the refresh token and the access token thanks to this authorization
        code and save those values on the given mail server.
        """
        if not request.env.user.has_group('base.group_system'):
            _logger.error('Google Gmail: non-system user trying to link an Gmail account.')
            raise Forbidden()

        if error:
            return _('An error occur during the authentication process.')

        try:
            state = json.loads(state)
            model_name = state['model']
            rec_id = state['id']
            csrf_token = state['csrf_token']
        except Exception:
            _logger.error('Google Gmail: Wrong state value %r.', state)
            raise Forbidden()

        model = request.env[model_name]

        if not isinstance(model, request.env.registry['google.gmail.mixin']):
            # The model must inherits from the "google.gmail.mixin" mixin
            raise Forbidden()

        record = model.browse(rec_id).exists()
        if not record:
            raise Forbidden()

        if not csrf_token or not consteq(csrf_token, record._get_gmail_csrf_token()):
            _logger.error('Google Gmail: Wrong CSRF token during Gmail authentication.')
            raise Forbidden()

        try:
            refresh_token, access_token, expiration = record._fetch_gmail_refresh_token(code)
        except UserError:
            return _('An error occur during the authentication process.')

        record.write({
            'google_gmail_access_token': access_token,
            'google_gmail_access_token_expiration': expiration,
            'google_gmail_authorization_code': code,
            'google_gmail_refresh_token': refresh_token,
        })

        url_params = {
            'id': rec_id,
            'model': model_name,
            'view_type': 'form'
        }
        url = '/web?#' + url_encode(url_params)
        return request.redirect(url)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\fetchmail_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class FetchmailServer(models.Model):
    _name = 'fetchmail.server'
    _inherit = ['fetchmail.server', 'google.gmail.mixin']

    server_type = fields.Selection(selection_add=[('gmail', 'Gmail OAuth Authentication')], ondelete={'gmail': 'set default'})

    def _compute_server_type_info(self):
        gmail_servers = self.filtered(lambda server: server.server_type == 'gmail')
        gmail_servers.server_type_info = _(
            'Connect your Gmail account with the OAuth Authentication process. \n'
            'You will be redirected to the Gmail login page where you will '
            'need to accept the permission.')
        super(FetchmailServer, self - gmail_servers)._compute_server_type_info()

    @api.onchange('server_type', 'is_ssl', 'object_id')
    def onchange_server_type(self):
        """Set the default configuration for a IMAP Gmail server."""
        if self.server_type == 'gmail':
            self.server = 'imap.gmail.com'
            self.is_ssl = True
            self.port = 993
        else:
            self.google_gmail_authorization_code = False
            self.google_gmail_refresh_token = False
            self.google_gmail_access_token = False
            self.google_gmail_access_token_expiration = False
            super(FetchmailServer, self).onchange_server_type()

    def _imap_login(self, connection):
        """Authenticate the IMAP connection.

        If the mail server is Gmail, we use the OAuth2 authentication protocol.
        """
        self.ensure_one()
        if self.server_type == 'gmail':
            auth_string = self._generate_oauth2_string(self.user, self.google_gmail_refresh_token)
            connection.authenticate('XOAUTH2', lambda x: auth_string)
            connection.select('INBOX')
        else:
            super(FetchmailServer, self)._imap_login(connection)

    def _get_connection_type(self):
        """Return which connection must be used for this mail server (IMAP or POP).
        The Gmail mail server used an IMAP connection.
        """
        self.ensure_one()
        return 'imap' if self.server_type == 'gmail' else super()._get_connection_type()

```

## File: models\google_gmail_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import time
import requests

from werkzeug.urls import url_encode, url_join

from odoo import _, api, fields, models, tools
from odoo.exceptions import AccessError, UserError

_logger = logging.getLogger(__name__)


class GoogleGmailMixin(models.AbstractModel):

    _name = 'google.gmail.mixin'
    _description = 'Google Gmail Mixin'

    _SERVICE_SCOPE = 'https://mail.google.com/'

    google_gmail_authorization_code = fields.Char(string='Authorization Code', groups='base.group_system', copy=False)
    google_gmail_refresh_token = fields.Char(string='Refresh Token', groups='base.group_system', copy=False)
    google_gmail_access_token = fields.Char(string='Access Token', groups='base.group_system', copy=False)
    google_gmail_access_token_expiration = fields.Integer(string='Access Token Expiration Timestamp', groups='base.group_system', copy=False)
    google_gmail_uri = fields.Char(compute='_compute_gmail_uri', string='URI', help='The URL to generate the authorization code from Google', groups='base.group_system')

    @api.depends('google_gmail_authorization_code')
    def _compute_gmail_uri(self):
        Config = self.env['ir.config_parameter'].sudo()
        google_gmail_client_id = Config.get_param('google_gmail_client_id')
        google_gmail_client_secret = Config.get_param('google_gmail_client_secret')
        base_url = self.get_base_url()

        redirect_uri = url_join(base_url, '/google_gmail/confirm')

        if not google_gmail_client_id or not google_gmail_client_secret:
            self.google_gmail_uri = False
        else:
            for record in self:
                google_gmail_uri = 'https://accounts.google.com/o/oauth2/v2/auth?%s' % url_encode({
                    'client_id': google_gmail_client_id,
                    'redirect_uri': redirect_uri,
                    'response_type': 'code',
                    'scope': self._SERVICE_SCOPE,
                    # access_type and prompt needed to get a refresh token
                    'access_type': 'offline',
                    'prompt': 'consent',
                    'state': json.dumps({
                        'model': record._name,
                        'id': record.id or False,
                        'csrf_token': record._get_gmail_csrf_token() if record.id else False,
                    })
                })
                record.google_gmail_uri = google_gmail_uri

    def open_google_gmail_uri(self):
        """Open the URL to accept the Gmail permission.

        This is done with an action, so we can force the user the save the form.
        We need him to save the form so the current mail server record exist in DB, and
        we can include the record ID in the URL.
        """
        self.ensure_one()

        if not self.env.user.has_group('base.group_system'):
            raise AccessError(_('Only the administrator can link a Gmail mail server.'))

        if not self.google_gmail_uri:
            raise UserError(_('Please configure your Gmail credentials.'))

        return {
            'type': 'ir.actions.act_url',
            'url': self.google_gmail_uri,
        }

    def _fetch_gmail_refresh_token(self, authorization_code):
        """Request the refresh token and the initial access token from the authorization code.

        :return:
            refresh_token, access_token, access_token_expiration
        """
        response = self._fetch_gmail_token('authorization_code', code=authorization_code)

        return (
            response['refresh_token'],
            response['access_token'],
            int(time.time()) + response['expires_in'],
        )

    def _fetch_gmail_access_token(self, refresh_token):
        """Refresh the access token thanks to the refresh token.

        :return:
            access_token, access_token_expiration
        """
        response = self._fetch_gmail_token('refresh_token', refresh_token=refresh_token)

        return (
            response['access_token'],
            int(time.time()) + response['expires_in'],
        )

    def _fetch_gmail_token(self, grant_type, **values):
        """Generic method to request an access token or a refresh token.

        Return the JSON response of the GMail API and manage the errors which can occur.

        :param grant_type: Depends the action we want to do (refresh_token or authorization_code)
        :param values: Additional parameters that will be given to the GMail endpoint
        """
        Config = self.env['ir.config_parameter'].sudo()
        google_gmail_client_id = Config.get_param('google_gmail_client_id')
        google_gmail_client_secret = Config.get_param('google_gmail_client_secret')
        base_url = self.get_base_url()
        redirect_uri = url_join(base_url, '/google_gmail/confirm')

        response = requests.post(
            'https://oauth2.googleapis.com/token',
            data={
                'client_id': google_gmail_client_id,
                'client_secret': google_gmail_client_secret,
                'grant_type': grant_type,
                'redirect_uri': redirect_uri,
                **values,
            },
            timeout=5,
        )

        if not response.ok:
            raise UserError(_('An error occurred when fetching the access token.'))

        return response.json()

    def _generate_oauth2_string(self, user, refresh_token):
        """Generate a OAuth2 string which can be used for authentication.

        :param user: Email address of the Gmail account to authenticate
        :param refresh_token: Refresh token for the given Gmail account

        :return: The SASL argument for the OAuth2 mechanism.
        """
        self.ensure_one()
        now_timestamp = int(time.time())
        if not self.google_gmail_access_token \
           or not self.google_gmail_access_token_expiration \
           or self.google_gmail_access_token_expiration < now_timestamp:

            access_token, expiration = self._fetch_gmail_access_token(self.google_gmail_refresh_token)

            self.write({
                'google_gmail_access_token': access_token,
                'google_gmail_access_token_expiration': expiration,
            })

            _logger.info(
                'Google Gmail: fetch new access token. Expires in %i minutes',
                (self.google_gmail_access_token_expiration - now_timestamp) // 60)
        else:
            _logger.info(
                'Google Gmail: reuse existing access token. Expire in %i minutes',
                (self.google_gmail_access_token_expiration - now_timestamp) // 60)

        return 'user=%s\1auth=Bearer %s\1\1' % (user, self.google_gmail_access_token)

    def _get_gmail_csrf_token(self):
        """Generate a CSRF token that will be verified in `google_gmail_callback`.

        This will prevent a malicious person to make an admin user disconnect the mail servers.
        """
        self.ensure_one()
        _logger.info('Google Gmail: generate CSRF token for %s #%i', self._name, self.id)
        return tools.misc.hmac(
            env=self.env(su=True),
            scope='google_gmail_oauth',
            message=(self._name, self.id),
        )

```

## File: models\ir_mail_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import _, fields, models, api
from odoo.exceptions import UserError


class IrMailServer(models.Model):
    """Represents an SMTP server, able to send outgoing emails, with SSL and TLS capabilities."""

    _name = 'ir.mail_server'
    _inherit = ['ir.mail_server', 'google.gmail.mixin']

    smtp_authentication = fields.Selection(
        selection_add=[('gmail', 'Gmail OAuth Authentication')],
        ondelete={'gmail': 'set default'})

    def _compute_smtp_authentication_info(self):
        gmail_servers = self.filtered(lambda server: server.smtp_authentication == 'gmail')
        gmail_servers.smtp_authentication_info = _(
            'Connect your Gmail account with the OAuth Authentication process.  \n'
            'By default, only a user with a matching email address will be able to use this server. '
            'To extend its use, you should set a "mail.default.from" system parameter.')
        super(IrMailServer, self - gmail_servers)._compute_smtp_authentication_info()

    @api.onchange('smtp_encryption')
    def _onchange_encryption(self):
        """Do not change the SMTP configuration if it's a Gmail server
        (e.g. the port which is already set)"""
        if self.smtp_authentication != 'gmail':
            super(IrMailServer, self)._onchange_encryption()

    @api.onchange('smtp_authentication')
    def _onchange_smtp_authentication_gmail(self):
        if self.smtp_authentication == 'gmail':
            self.smtp_host = 'smtp.gmail.com'
            self.smtp_encryption = 'starttls'
            self.smtp_port = 587
        else:
            self.google_gmail_authorization_code = False
            self.google_gmail_refresh_token = False
            self.google_gmail_access_token = False
            self.google_gmail_access_token_expiration = False

    @api.onchange('smtp_user', 'smtp_authentication')
    def _on_change_smtp_user_gmail(self):
        """The Gmail mail servers can only be used for the user personal email address."""
        if self.smtp_authentication == 'gmail':
            self.from_filter = self.smtp_user

    @api.constrains('smtp_authentication', 'smtp_pass', 'smtp_encryption', 'from_filter', 'smtp_user')
    def _check_use_google_gmail_service(self):
        gmail_servers = self.filtered(lambda server: server.smtp_authentication == 'gmail')
        for server in gmail_servers:
            if server.smtp_pass:
                raise UserError(_(
                    'Please leave the password field empty for Gmail mail server %r. '
                    'The OAuth process does not require it', server.name))

            if server.smtp_encryption != 'starttls':
                raise UserError(_(
                    'Incorrect Connection Security for Gmail mail server %r. '
                    'Please set it to "TLS (STARTTLS)".', server.name))

            if not server.smtp_user:
                raise UserError(_(
                    'Please fill the "Username" field with your Gmail username (your email address). '
                    'This should be the same account as the one used for the Gmail OAuthentication Token.'))

    def _smtp_login(self, connection, smtp_user, smtp_password):
        if len(self) == 1 and self.smtp_authentication == 'gmail':
            auth_string = self._generate_oauth2_string(smtp_user, self.google_gmail_refresh_token)
            oauth_param = base64.b64encode(auth_string.encode()).decode()
            connection.ehlo()
            connection.docmd('AUTH', f'XOAUTH2 {oauth_param}')
        else:
            super(IrMailServer, self)._smtp_login(connection, smtp_user, smtp_password)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    google_gmail_client_identifier = fields.Char('Gmail Client Id', config_parameter='google_gmail_client_id')
    google_gmail_client_secret = fields.Char('Gmail Client Secret', config_parameter='google_gmail_client_secret')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import google_gmail_mixin

from . import fetchmail_server
from . import ir_mail_server
from . import res_config_settings

```

## File: static\description\icon.svg

```svg
<?xml version="1.0" ?><svg height="48px" viewBox="0 0 48 48" width="48px" xmlns="http://www.w3.org/2000/svg"><path d="M45,16.2l-5,2.75l-5,4.75L35,40h7c1.657,0,3-1.343,3-3V16.2z" fill="#4caf50"/><path d="M3,16.2l3.614,1.71L13,23.7V40H6c-1.657,0-3-1.343-3-3V16.2z" fill="#1e88e5"/><polygon fill="#e53935" points="35,11.2 24,19.45 13,11.2 12,17 13,23.7 24,31.95 35,23.7 36,17"/><path d="M3,12.298V16.2l10,7.5V11.2L9.876,8.859C9.132,8.301,8.228,8,7.298,8h0C4.924,8,3,9.924,3,12.298z" fill="#c62828"/><path d="M45,12.298V16.2l-10,7.5V11.2l3.124-2.341C38.868,8.301,39.772,8,40.702,8h0 C43.076,8,45,9.924,45,12.298z" fill="#fbc02d"/></svg>
```

## File: views\fetchmail_server_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fetchmail_server_view_form" model="ir.ui.view">
        <field name="name">fetchmail.server.view.form.inherit.gmail</field>
        <field name="model">fetchmail.server</field>
        <field name="priority">100</field>
        <field name="inherit_id" ref="mail.view_email_server_form"/>
        <field name="arch" type="xml">
            <field name="user" position="after">
                <field name="google_gmail_uri" invisible="1"/>
                <field name="google_gmail_refresh_token" invisible="1"/>
                <div attrs="{'invisible': [('server_type', '!=', 'gmail')]}"
                    class="d-flex flex-row align-items-center" colspan="2">
                    <span attrs="{'invisible': ['|', ('server_type', '!=', 'gmail'), ('google_gmail_refresh_token', '=', False)]}"
                        class="badge text-bg-success">
                        Gmail Token Valid
                    </span>
                    <button type="object"
                        name="open_google_gmail_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', ('google_gmail_uri', '=', False), ('server_type', '!=', 'gmail'), ('google_gmail_refresh_token', '!=', False)]}">
                        <i class="fa fa-arrow-right"/>
                        Connect your Gmail account
                    </button>
                    <button type="object"
                        name="open_google_gmail_uri" class="btn-link px-0 ms-2"
                        attrs="{'invisible': ['|', '|', ('google_gmail_uri', '=', False), ('server_type', '!=', 'gmail'), ('google_gmail_refresh_token', '=', False)]}">
                        <i class="fa fa-cog" title="Edit Settings"/>
                    </button>
                    <button class="alert alert-warning d-block mt-2 text-start"
                        icon="fa-arrow-right" type="action" role="alert"
                        name="%(base.res_config_setting_act_window)d"
                        attrs="{'invisible': ['|', ('server_type', '!=', 'gmail'), ('google_gmail_uri', '!=', False)]}">
                        Setup your Gmail API credentials in the general settings to link a Gmail account.
                    </button>
                </div>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\ir_mail_server_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_mail_server_view_form" model="ir.ui.view">
        <field name="name">ir.mail_server.view.form.inherit.gmail</field>
        <field name="model">ir.mail_server</field>
        <field name="inherit_id" ref="base.ir_mail_server_form"/>
        <field name="arch" type="xml">
            <field name="smtp_user" position="after">
                <field name="google_gmail_uri" invisible="1"/>
                <field name="google_gmail_refresh_token" invisible="1"/>
                <div attrs="{'invisible': [('smtp_authentication', '!=', 'gmail')]}"
                    class="d-flex flex-row align-items-center" colspan="8">
                    <span attrs="{'invisible': ['|', ('smtp_authentication', '!=', 'gmail'), ('google_gmail_refresh_token', '=', False)]}"
                        class="badge text-bg-success">
                        Gmail Token Valid
                    </span>
                    <button type="object"
                        name="open_google_gmail_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', ('google_gmail_uri', '=', False), ('smtp_authentication', '!=', 'gmail'), ('google_gmail_refresh_token', '!=', False)]}">
                        <i class="fa fa-arrow-right"/>
                        Connect your Gmail account
                    </button>
                    <button type="object"
                        name="open_google_gmail_uri" class="btn-link px-0 ms-2"
                        attrs="{'invisible': ['|', '|', ('google_gmail_uri', '=', False), ('smtp_authentication', '!=', 'gmail'), ('google_gmail_refresh_token', '=', False)]}">
                        <i class="fa fa-cog" title="Edit Settings"/>
                    </button>
                    <button class="alert alert-warning d-block mt-2 text-start"
                        icon="fa-arrow-right" type="action" role="alert"
                        name="%(base.res_config_setting_act_window)d"
                        attrs="{'invisible': ['|', ('smtp_authentication', '!=', 'gmail'), ('google_gmail_uri', '!=', False)]}">
                        Setup your Gmail API credentials in the general settings to link a Gmail account.
                    </button>
                </div>
            </field>
            <field name="smtp_authentication_info" position="after">
                <a href="https://www.odoo.com/documentation/16.0/applications/general/email_communication/email_servers.html?highlight=outgoing email server#use-a-default-from-email-address"
                    attrs="{'invisible': [('smtp_authentication', '!=', 'gmail')]}" target="_blank">
                    Read More
                </a>
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
            <field name="name">res.config.settings.view.form.inherit.google_gmail</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <div id="msg_module_google_gmail" position="replace">
                    <div class="row mt16" id="gmail_client_identifier">
                        <label string="ID" for="google_gmail_client_identifier" class="col-lg-3 o_light_label"/>
                        <field name="google_gmail_client_identifier" class="ms-2"
                            placeholder="ID of your Google app"/>
                    </div>
                    <div class="row mt16" id="gmail_client_secret">
                        <label string="Secret" for="google_gmail_client_secret" class="col-lg-3 o_light_label"/>
                        <field name="google_gmail_client_secret" password="True" class="ms-2"
                            placeholder="Secret of your Google app"/>
                    </div>
                </div>
            </field>
        </record>
    </data>
</odoo>

```

