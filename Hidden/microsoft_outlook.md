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
    "version": "1.1",
    "category": "Hidden",
    "description": "Outlook support for incoming / outgoing mail servers",
    "depends": [
        "mail",
    ],
    "data": [
        "views/fetchmail_server_views.xml",
        "views/ir_mail_server_views.xml",
        "views/res_config_settings_views.xml",
        "views/templates.xml",
    ],
    "license": "LGPL-3",
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

        if not isinstance(model, request.env.registry['microsoft.outlook.mixin']):
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
                'error': str(e),
                'model_name': model_name,
                'rec_id': rec_id,
            })

        record.write({
            'microsoft_outlook_refresh_token': refresh_token,
            'microsoft_outlook_access_token': access_token,
            'microsoft_outlook_access_token_expiration': expiration,
        })

        return request.redirect(f'/odoo/{model_name}/{rec_id}')

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
from odoo.exceptions import UserError


class FetchmailServer(models.Model):
    """Add the Outlook OAuth authentication on the incoming mail servers."""

    _name = 'fetchmail.server'
    _inherit = ['fetchmail.server', 'microsoft.outlook.mixin']

    _OUTLOOK_SCOPE = 'https://outlook.office.com/IMAP.AccessAsUser.All'

    server_type = fields.Selection(selection_add=[('outlook', 'Outlook OAuth Authentication')], ondelete={'outlook': 'set default'})

    def _compute_server_type_info(self):
        outlook_servers = self.filtered(lambda server: server.server_type == 'outlook')
        outlook_servers.server_type_info = _(
            'Connect your personal Outlook account using OAuth. \n'
            'You will be redirected to the Outlook login page to accept '
            'the permissions.')
        super(FetchmailServer, self - outlook_servers)._compute_server_type_info()

    @api.depends('server_type')
    def _compute_is_microsoft_outlook_configured(self):
        outlook_servers = self.filtered(lambda server: server.server_type == 'outlook')
        (self - outlook_servers).is_microsoft_outlook_configured = False
        super(FetchmailServer, outlook_servers)._compute_is_microsoft_outlook_configured()

    @api.constrains('server_type', 'is_ssl')
    def _check_use_microsoft_outlook_service(self):
        for server in self:
            if server.server_type == 'outlook' and not server.is_ssl:
                raise UserError(_('SSL is required for server “%s”.', server.name))

    @api.onchange('server_type')
    def onchange_server_type(self):
        """Set the default configuration for a IMAP Outlook server."""
        if self.server_type == 'outlook':
            self.server = 'imap.outlook.com'
            self.is_ssl = True
            self.port = 993
        else:
            self.microsoft_outlook_refresh_token = False
            self.microsoft_outlook_access_token = False
            self.microsoft_outlook_access_token_expiration = False
            super(FetchmailServer, self).onchange_server_type()

    def _imap_login(self, connection):
        """Authenticate the IMAP connection.

        If the mail server is Outlook, we use the OAuth2 authentication protocol.
        """
        self.ensure_one()
        if self.server_type == 'outlook':
            auth_string = self._generate_outlook_oauth2_string(self.user)
            connection.authenticate('XOAUTH2', lambda x: auth_string)
            connection.select('INBOX')
        else:
            super()._imap_login(connection)

    def _get_connection_type(self):
        """Return which connection must be used for this mail server (IMAP or POP).
        The Outlook mail server used an IMAP connection.
        """
        self.ensure_one()
        return 'imap' if self.server_type == 'outlook' else super()._get_connection_type()

```

## File: models\ir_mail_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class IrMailServer(models.Model):
    """Add the Outlook OAuth authentication on the outgoing mail servers."""

    _name = 'ir.mail_server'
    _inherit = ['ir.mail_server', 'microsoft.outlook.mixin']

    _OUTLOOK_SCOPE = 'https://outlook.office.com/SMTP.Send'

    smtp_authentication = fields.Selection(
        selection_add=[('outlook', 'Outlook OAuth Authentication')],
        ondelete={'outlook': 'set default'})

    @api.depends('smtp_authentication')
    def _compute_is_microsoft_outlook_configured(self):
        outlook_servers = self.filtered(lambda server: server.smtp_authentication == 'outlook')
        (self - outlook_servers).is_microsoft_outlook_configured = False
        super(IrMailServer, outlook_servers)._compute_is_microsoft_outlook_configured()

    def _compute_smtp_authentication_info(self):
        outlook_servers = self.filtered(lambda server: server.smtp_authentication == 'outlook')
        outlook_servers.smtp_authentication_info = _(
            'Connect your Outlook account with the OAuth Authentication process.  \n'
            'By default, only a user with a matching email address will be able to use this server. '
            'To extend its use, you should set a "mail.default.from" system parameter.')
        super(IrMailServer, self - outlook_servers)._compute_smtp_authentication_info()

    @api.constrains('smtp_authentication', 'smtp_pass', 'smtp_encryption', 'smtp_user')
    def _check_use_microsoft_outlook_service(self):
        outlook_servers = self.filtered(lambda server: server.smtp_authentication == 'outlook')
        for server in outlook_servers:
            if server.smtp_pass:
                raise UserError(_(
                    'Please leave the password field empty for Outlook mail server “%s”. '
                    'The OAuth process does not require it', server.name))

            if server.smtp_encryption != 'starttls':
                raise UserError(_(
                    'Incorrect Connection Security for Outlook mail server “%s”. '
                    'Please set it to "TLS (STARTTLS)".', server.name))

            if not server.smtp_user:
                raise UserError(_(
                            'Please fill the "Username" field with your Outlook/Office365 username (your email address). '
                            'This should be the same account as the one used for the Outlook OAuthentication Token.'))

    @api.onchange('smtp_encryption')
    def _onchange_encryption(self):
        """Do not change the SMTP configuration if it's a Outlook server

        (e.g. the port which is already set)"""
        if self.smtp_authentication != 'outlook':
            super()._onchange_encryption()

    @api.onchange('smtp_authentication')
    def _onchange_smtp_authentication_outlook(self):
        if self.smtp_authentication == 'outlook':
            self.smtp_host = 'smtp.outlook.com'
            self.smtp_encryption = 'starttls'
            self.smtp_port = 587
        else:
            self.microsoft_outlook_refresh_token = False
            self.microsoft_outlook_access_token = False
            self.microsoft_outlook_access_token_expiration = False

    @api.onchange('smtp_user', 'smtp_authentication')
    def _on_change_smtp_user_outlook(self):
        """The Outlook mail servers can only be used for the user personal email address."""
        if self.smtp_authentication == 'outlook':
            self.from_filter = self.smtp_user

    def _smtp_login(self, connection, smtp_user, smtp_password):
        if len(self) == 1 and self.smtp_authentication == 'outlook':
            auth_string = self._generate_outlook_oauth2_string(smtp_user)
            oauth_param = base64.b64encode(auth_string.encode()).decode()
            connection.ehlo()
            connection.docmd('AUTH', f'XOAUTH2 {oauth_param}')
        else:
            super()._smtp_login(connection, smtp_user, smtp_password)

```

## File: models\microsoft_outlook_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import time
import requests

from werkzeug.urls import url_encode, url_join

from odoo import _, api, fields, models
from odoo.exceptions import AccessError, UserError
from odoo.tools.misc import hmac

_logger = logging.getLogger(__name__)


class MicrosoftOutlookMixin(models.AbstractModel):

    _name = 'microsoft.outlook.mixin'
    _description = 'Microsoft Outlook Mixin'

    _OUTLOOK_SCOPE = None

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

    def _compute_is_microsoft_outlook_configured(self):
        Config = self.env['ir.config_parameter'].sudo()
        microsoft_outlook_client_id = Config.get_param('microsoft_outlook_client_id')
        microsoft_outlook_client_secret = Config.get_param('microsoft_outlook_client_secret')
        self.is_microsoft_outlook_configured = microsoft_outlook_client_id and microsoft_outlook_client_secret

    @api.depends('is_microsoft_outlook_configured')
    def _compute_outlook_uri(self):
        Config = self.env['ir.config_parameter'].sudo()
        base_url = self.get_base_url()
        microsoft_outlook_client_id = Config.get_param('microsoft_outlook_client_id')

        for record in self:
            if not record.id or not record.is_microsoft_outlook_configured:
                record.microsoft_outlook_uri = False
                continue

            record.microsoft_outlook_uri = url_join(self._get_microsoft_endpoint(), 'authorize?%s' % url_encode({
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
        We need him to save the form so the current mail server record exist in DB and
        we can include the record ID in the URL.
        """
        self.ensure_one()

        if not self.env.user.has_group('base.group_system'):
            raise AccessError(_('Only the administrator can link an Outlook mail server.'))

        if not self.is_microsoft_outlook_configured:
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
            int(time.time()) + int(response['expires_in']),
        )

    def _fetch_outlook_access_token(self, refresh_token):
        """Refresh the access token thanks to the refresh token.

        :return:
            access_token, access_token_expiration
        """
        response = self._fetch_outlook_token('refresh_token', refresh_token=refresh_token)
        return (
            response['refresh_token'],
            response['access_token'],
            int(time.time()) + int(response['expires_in']),
        )

    def _fetch_outlook_token(self, grant_type, **values):
        """Generic method to request an access token or a refresh token.

        Return the JSON response of the Outlook API and manage the errors which can occur.

        :param grant_type: Depends the action we want to do (refresh_token or authorization_code)
        :param values: Additional parameters that will be given to the Outlook endpoint
        """
        Config = self.env['ir.config_parameter'].sudo()
        base_url = self.get_base_url()
        microsoft_outlook_client_id = Config.get_param('microsoft_outlook_client_id')
        microsoft_outlook_client_secret = Config.get_param('microsoft_outlook_client_secret')

        response = requests.post(
            url_join(self._get_microsoft_endpoint(), 'token'),
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
            raise UserError(_('An error occurred when fetching the access token. %s', error_description))

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
                raise UserError(_('Please connect with your Outlook account before using it.'))
            (
                self.microsoft_outlook_refresh_token,
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
        return hmac(
            env=self.env(su=True),
            scope='microsoft_outlook_oauth',
            message=(self._name, self.id),
        )

    @api.model
    def _get_microsoft_endpoint(self):
        return self.env["ir.config_parameter"].sudo().get_param(
            'microsoft_outlook.endpoint',
            'https://login.microsoftonline.com/common/oauth2/v2.0/',
        )

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

from . import fetchmail_server
from . import ir_mail_server
from . import res_config_settings

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M46 26.475a.936.936 0 0 0-.448-.804h-.005l-.017-.01-14.554-8.6a1.956 1.956 0 0 0-2.182 0l-14.553 8.6-.018.01a.946.946 0 0 0 .023 1.621l14.553 8.6a1.956 1.956 0 0 0 2.182 0l14.554-8.6a.935.935 0 0 0 .465-.817Z" fill="#0A2767"/><path d="M15.938 20.733h9.55v8.74h-9.55v-8.74Zm28.109-8.883V7.852A1.813 1.813 0 0 0 42.276 6H17.492a1.813 1.813 0 0 0-1.77 1.852v3.998l14.65 3.9 13.675-3.9Z" fill="#0364B8"/><path d="M15.72 11.85h9.768v8.775h-9.767V11.85Z" fill="#0078D4"/><path d="M35.256 11.85h-9.768v8.775l9.768 8.775h8.79v-8.775l-8.79-8.775Z" fill="#28A8EA"/><path d="M25.488 20.625h9.768V29.4h-9.768v-8.775Z" fill="#0078D4"/><path d="M25.488 29.4h9.768v8.775h-9.768V29.4Z" fill="#0364B8"/><path d="M15.938 29.472h9.55v7.944h-9.55v-7.944Z" fill="#14447D"/><path d="M35.256 29.4h8.79v8.775h-8.79V29.4Z" fill="#0078D4"/><path d="m45.553 27.238-.019.01-14.553 8.17a2.032 2.032 0 0 1-.984.304l-.796-.463a1.99 1.99 0 0 1-.195-.112l-14.75-8.403h-.006l-.482-.269v16.54A2 2 0 0 0 15.783 45h28.233c.017 0 .032-.008.05-.008.233-.015.463-.063.683-.142.095-.04.187-.088.274-.142.066-.038.178-.118.178-.118.5-.37.797-.954.8-1.575v-16.54a.877.877 0 0 1-.448.763Z" fill="url(#o_icon_microsoft_outlook__a)"/><path opacity=".5" d="M45.219 26.41v1.014L30 37.882 14.246 26.751a.01.01 0 0 0-.01-.01l-1.445-.868v-.73l.596-.01 1.26.72.03.01.107.069s14.807 8.434 14.846 8.453l.567.332c.048-.02.097-.04.156-.059.03-.02 14.7-8.258 14.7-8.258l.166.01Z" fill="#0A2767"/><path d="m45.553 27.238-.019.011-14.553 8.17a2.044 2.044 0 0 1-2.182 0l-14.554-8.17-.017-.01a.877.877 0 0 1-.46-.764v16.54A2 2 0 0 0 15.78 45h28.205A2 2 0 0 0 46 43.015v-16.54a.877.877 0 0 1-.447.763Z" fill="#1490DF"/><path opacity=".1" d="m31.193 35.299-.218.122a2.025 2.025 0 0 1-.963.313l5.537 6.536 9.659 2.323c.265-.2.475-.462.612-.763L31.193 35.3Z"/><path opacity=".05" d="m32.18 34.745-1.205.676a2.027 2.027 0 0 1-.963.313l2.594 7.14L45.21 44.59a1.97 1.97 0 0 0 .79-1.575v-.214l-13.82-8.056Z"/><path d="M15.809 45h28.174c.434.002.857-.134 1.206-.39L29.2 35.26a1.977 1.977 0 0 1-.195-.111l-14.75-8.403h-.006l-.481-.27v16.482A2.042 2.042 0 0 0 15.809 45Z" fill="#28A8EA"/><path opacity=".1" d="M27.442 15.587v20.797a1.792 1.792 0 0 1-1.123 1.657c-.21.09-.436.137-.665.137H13.768V14.775h1.953V13.8h9.934a1.793 1.793 0 0 1 1.787 1.787Z"/><path opacity=".2" d="M26.465 16.562V37.36c.003.235-.047.468-.146.682a1.78 1.78 0 0 1-1.641 1.109h-10.91V14.775h10.91c.283-.003.563.068.81.205.6.3.977.913.977 1.582Z"/><path opacity=".2" d="M26.465 16.562V35.41a1.801 1.801 0 0 1-1.787 1.79h-10.91V14.776h10.91c.283-.003.563.068.81.205.6.3.977.913.977 1.582Z"/><path opacity=".2" d="M25.488 16.562V35.41a1.795 1.795 0 0 1-1.787 1.79h-9.933V14.776H23.7c.988 0 1.788.8 1.787 1.786v.001Z"/><path d="M5.79 14.775h17.908c.989 0 1.79.8 1.79 1.787v17.876c0 .987-.801 1.787-1.79 1.787H5.79c-.988 0-1.79-.8-1.79-1.787V16.562c0-.987.802-1.787 1.79-1.787Z" fill="url(#o_icon_microsoft_outlook__b)"/><path d="M9.596 22.27a5.202 5.202 0 0 1 2.045-2.255 6.191 6.191 0 0 1 3.25-.813 5.762 5.762 0 0 1 3.007.771 5.16 5.16 0 0 1 1.99 2.155 6.94 6.94 0 0 1 .697 3.169 7.33 7.33 0 0 1-.718 3.315 5.28 5.28 0 0 1-2.05 2.23 5.992 5.992 0 0 1-3.12.792 5.89 5.89 0 0 1-3.074-.78 5.236 5.236 0 0 1-2.016-2.16 6.779 6.779 0 0 1-.705-3.13 7.528 7.528 0 0 1 .694-3.293Zm2.18 5.295a3.375 3.375 0 0 0 1.15 1.484 3.01 3.01 0 0 0 1.798.54 3.152 3.152 0 0 0 1.918-.558c.51-.375.899-.89 1.118-1.484a5.75 5.75 0 0 0 .356-2.07 6.289 6.289 0 0 0-.336-2.096 3.314 3.314 0 0 0-1.082-1.546 2.975 2.975 0 0 0-1.902-.585 3.104 3.104 0 0 0-1.839.545 3.404 3.404 0 0 0-1.172 1.496 5.937 5.937 0 0 0-.008 4.276v-.002Z" fill="#fff"/><path d="M35.256 11.85h8.79v8.775h-8.79V11.85Z" fill="#50D9FF"/><defs><linearGradient id="o_icon_microsoft_outlook__a" x1="29.884" y1="26.475" x2="29.884" y2="45" gradientUnits="userSpaceOnUse"><stop stop-color="#35B8F1"/><stop offset="1" stop-color="#28A8EA"/></linearGradient><linearGradient id="o_icon_microsoft_outlook__b" x1="7.733" y1="13.378" x2="21.718" y2="37.643" gradientUnits="userSpaceOnUse"><stop stop-color="#1784D9"/><stop offset=".5" stop-color="#107AD5"/><stop offset="1" stop-color="#0A63C9"/></linearGradient></defs></svg>

```

## File: views\fetchmail_server_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fetchmail_server_view_form" model="ir.ui.view">
        <field name="name">fetchmail.server.view.form.inherit.outlook</field>
        <field name="model">fetchmail.server</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="mail.view_email_server_form"/>
        <field name="arch" type="xml">
            <field name="user" position="after">
                <div invisible="server_type != 'outlook'"
                    class="d-flex flex-row align-items-center" colspan="2"
                    groups="base.group_system">
                    <span invisible="server_type != 'outlook' or not microsoft_outlook_refresh_token"
                        class="badge text-bg-success">
                        Outlook Token Valid
                    </span>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0"
                        invisible="not is_microsoft_outlook_configured or server_type != 'outlook' or microsoft_outlook_refresh_token">
                        <i class="oi oi-arrow-right"/>
                        Connect your Outlook account
                    </button>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0 ms-2"
                        invisible="not is_microsoft_outlook_configured or server_type != 'outlook' or not microsoft_outlook_refresh_token">
                        <i class="fa fa-cog" title="Edit Settings"/>
                    </button>
                    <button class="alert alert-warning d-block mt-2 text-start"
                        icon="oi-arrow-right" type="action" role="alert"
                        name="%(base.res_config_setting_act_window)d"
                        invisible="is_microsoft_outlook_configured or server_type != 'outlook'">
                        Setup your Outlook API credentials in the general settings to link a Outlook account.
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
        <field name="name">ir.mail_server.view.form.inherit.outlook</field>
        <field name="model">ir.mail_server</field>
        <field name="inherit_id" ref="base.ir_mail_server_form"/>
        <field name="arch" type="xml">
            <field name="smtp_user" position="after">
                <div invisible="smtp_authentication != 'outlook'"
                    class="d-flex flex-row align-items-center" colspan="8"
                    groups="base.group_system">
                    <span invisible="smtp_authentication != 'outlook' or not microsoft_outlook_refresh_token"
                        class="badge text-bg-success">
                        Outlook Token Valid
                    </span>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0"
                        invisible="not is_microsoft_outlook_configured or smtp_authentication != 'outlook' or microsoft_outlook_refresh_token">
                        <i class="oi oi-arrow-right"/>
                        Connect your Outlook account
                    </button>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0 ms-2"
                        invisible="not is_microsoft_outlook_configured or smtp_authentication != 'outlook' or not microsoft_outlook_refresh_token">
                        <i class="fa fa-cog" title="Edit Settings"/>
                    </button>
                    <button class="alert alert-warning d-block mt-2 text-start"
                        icon="oi-arrow-right" type="action" role="alert"
                        name="%(base.res_config_setting_act_window)d"
                        invisible="is_microsoft_outlook_configured or smtp_authentication != 'outlook'">
                        Setup your Outlook API credentials in the general settings to link a Outlook account.
                    </button>
                </div>
            </field>
            <field name="smtp_authentication_info" position="after">
                <widget invisible="smtp_authentication != 'outlook'" name="documentation_link" path="/applications/general/email_communication/email_servers.html?highlight=outgoing email server#use-a-default-from-email-address" label="Read More"/>
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
                <div id="msg_module_microsoft_outlook" position="replace">
                    <div class="row mt16" id="outlook_client_identifier">
                        <label string="ID" for="microsoft_outlook_client_identifier"
                            class="col-lg-3 o_light_label"/>
                        <field name="microsoft_outlook_client_identifier" class="ms-2"
                            placeholder="ID of your Outlook app"/>
                    </div>
                    <div class="row mt16" id="outlook_client_secret">
                        <label string="Secret" for="microsoft_outlook_client_secret"
                            class="col-lg-3 o_light_label"/>
                        <field name="microsoft_outlook_client_secret" password="True" class="ms-2"
                            placeholder="Secret of your Outlook app"/>
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
                <a t-attf-href="/odoo/{{model_name}}/{{rec_id}}">
                    Go back to your mail server
                </a>
            </div>
        </div>
    </template>
</odoo>

```

