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
                'error': str(e.name),
                'model_name': model_name,
                'rec_id': rec_id,
            })

        record.write({
            'microsoft_outlook_refresh_token': refresh_token,
            'microsoft_outlook_access_token': access_token,
            'microsoft_outlook_access_token_expiration': expiration,
        })

        return request.redirect(f'/web?#id={rec_id}&model={model_name}&view_type=form')

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
                raise UserError(_('SSL is required for the server %r.', server.name))

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
                    'Please leave the password field empty for Outlook mail server %r. '
                    'The OAuth process does not require it', server.name))

            if server.smtp_encryption != 'starttls':
                raise UserError(_(
                    'Incorrect Connection Security for Outlook mail server %r. '
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
<?xml version="1.0" ?><svg id="Capa_1" style="enable-background:new 0 0 128 128;" version="1.1" viewBox="0 0 128 128" xml:space="preserve" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><style type="text/css">
	.st0{fill:#21A365;}
	.st1{fill:#107C41;}
	.st2{fill:#185B37;}
	.st3{fill:#33C481;}
	.st4{fill:#17864C;}
	.st5{fill:#FFFFFF;}
	.st6{fill:#036C70;}
	.st7{fill:#1A9BA1;}
	.st8{fill:#37C6D0;}
	.st9{fill:#04878B;}
	.st10{fill:#4F59CA;}
	.st11{fill:#7B82EA;}
	.st12{fill:#4C53BB;}
	.st13{fill:#0F78D5;}
	.st14{fill:#29A7EB;}
	.st15{fill:#0358A8;}
	.st16{fill:#0F79D6;}
	.st17{fill:#038387;}
	.st18{fill:#048A8E;}
	.st19{fill:#C8421D;}
	.st20{fill:#FF8F6A;}
	.st21{fill:#ED6B47;}
	.st22{fill:#891323;}
	.st23{fill:#AF2131;}
	.st24{fill:#C94E60;}
	.st25{fill:#E08195;}
	.st26{fill:#B42839;}
	.st27{fill:#0464B8;}
	.st28{fill:#0377D4;}
	.st29{fill:#4FD8FF;}
	.st30{fill:#1681D7;}
	.st31{fill:#0178D4;}
	.st32{fill:#042071;}
	.st33{fill:#168FDE;}
	.st34{fill:#CA64EA;}
	.st35{fill:#7E1FAF;}
	.st36{fill:#AE4BD5;}
	.st37{fill:#9332BF;}
	.st38{fill:#7719AA;}
	.st39{fill:#0078D4;}
	.st40{fill:#1490DF;}
	.st41{fill:#0364B8;}
	.st42{fill:#28A8EA;}
	.st43{fill:#41A5ED;}
	.st44{fill:#2C7BD5;}
	.st45{fill:#195ABE;}
	.st46{fill:#103E91;}
	.st47{fill:#2166C3;}
	.st48{opacity:0.2;}
</style><g><path class="st27" d="M120.2,22.8H34.2V9.1C34.2,6.8,36,5,38.2,5h77.9c2.3,0,4.1,1.8,4.1,4.1V22.8z"/><rect class="st28" height="26.6" width="28.7" x="34.2" y="22.8"/><rect class="st14" height="26.6" width="28.7" x="62.9" y="22.8"/><rect class="st29" height="26.6" width="28.7" x="91.6" y="22.8"/><rect class="st30" height="26.6" width="28.7" x="34.2" y="49.4"/><rect class="st31" height="26.6" width="28.7" x="62.9" y="49.4"/><rect class="st14" height="26.6" width="28.7" x="91.6" y="49.4"/><rect class="st27" height="26.6" width="28.7" x="62.9" y="75.9"/><rect class="st30" height="26.6" width="28.7" x="91.6" y="75.9"/></g><path class="st32" d="M126.9,69.1l-6.6,3.9V61.8l6.5,3.6C128.3,66.1,128.3,68.2,126.9,69.1z"/><path class="st33" d="M126.9,69.1l-0.6,0.4l0,0l-0.7,0.4l-5.3,3.1v-0.1l-88.4,50.6h89.1c3.8,0,6.8-3.1,6.8-6.8l0.1-49.4  C128,67.9,127.6,68.7,126.9,69.1z"/><g><path class="st14" d="M122,123.5H32.8c-3.8,0-6.8-3.1-6.8-6.8V68.5L122,123.5z"/></g><path class="st30" d="M59,96.5h-53c-3.5,0-6.4-2.9-6.4-6.4V37.9c0-3.5,2.9-6.4,6.4-6.4h53c3.5,0,6.4,2.9,6.4,6.4v52.2  C65.4,93.6,62.6,96.5,59,96.5z"/><g><path class="st5" d="M32.5,82.9c-10.3,0-16.8-7.8-16.8-18.2c0-11,7.1-18.8,17.3-18.8c10.6,0,16.8,7.9,16.8,18.1   C49.9,76.1,42.5,82.9,32.5,82.9L32.5,82.9z M32.9,77.7c6.4,0,10-5.9,10-13.4c0-6.8-3.4-13.1-10-13.1s-10.1,6.2-10.1,13.4   S26.4,77.7,32.9,77.7L32.9,77.7z"/></g><path class="st48" d="M65.5,37.3c0,0.2,0,0.4,0,0.6v52.2c0,3.5-2.9,6.4-6.4,6.4H26v5.7h38.4c3.5,0,6.4-2.9,6.4-6.4V43.6  C70.8,40.4,68.5,37.7,65.5,37.3z"/></svg>
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
                <field name="is_microsoft_outlook_configured" invisible="1"/>
                <field name="microsoft_outlook_refresh_token" invisible="1"/>
                <field name="microsoft_outlook_access_token" invisible="1"/>
                <field name="microsoft_outlook_access_token_expiration" invisible="1"/>
                <div attrs="{'invisible': [('server_type', '!=', 'outlook')]}"
                    class="d-flex flex-row align-items-center" colspan="2">
                    <span attrs="{'invisible': ['|', ('server_type', '!=', 'outlook'), ('microsoft_outlook_refresh_token', '=', False)]}"
                        class="badge text-bg-success">
                        Outlook Token Valid
                    </span>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', ('is_microsoft_outlook_configured', '=', False), ('server_type', '!=', 'outlook'), ('microsoft_outlook_refresh_token', '!=', False)]}">
                        <i class="fa fa-arrow-right"/>
                        Connect your Outlook account
                    </button>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0 ms-2"
                        attrs="{'invisible': ['|', '|', ('is_microsoft_outlook_configured', '=', False), ('server_type', '!=', 'outlook'), ('microsoft_outlook_refresh_token', '=', False)]}">
                        <i class="fa fa-cog" title="Edit Settings"/>
                    </button>
                    <button class="alert alert-warning d-block mt-2 text-start"
                        icon="fa-arrow-right" type="action" role="alert"
                        name="%(base.res_config_setting_act_window)d"
                        attrs="{'invisible': ['|', ('is_microsoft_outlook_configured', '=', True), ('server_type', '!=', 'outlook')]}">
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
                <field name="is_microsoft_outlook_configured" invisible="1"/>
                <field name="microsoft_outlook_refresh_token" invisible="1"/>
                <field name="microsoft_outlook_access_token" invisible="1"/>
                <field name="microsoft_outlook_access_token_expiration" invisible="1"/>
                <div attrs="{'invisible': [('smtp_authentication', '!=', 'outlook')]}"
                    class="d-flex flex-row align-items-center" colspan="8">
                    <span attrs="{'invisible': ['|', ('smtp_authentication', '!=', 'outlook'), ('microsoft_outlook_refresh_token', '=', False)]}"
                        class="badge text-bg-success">
                        Outlook Token Valid
                    </span>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', ('is_microsoft_outlook_configured', '=', False), ('smtp_authentication', '!=', 'outlook'), ('microsoft_outlook_refresh_token', '!=', False)]}">
                        <i class="fa fa-arrow-right"/>
                        Connect your Outlook account
                    </button>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0 ms-2"
                        attrs="{'invisible': ['|', '|', ('is_microsoft_outlook_configured', '=', False), ('smtp_authentication', '!=', 'outlook'), ('microsoft_outlook_refresh_token', '=', False)]}">
                        <i class="fa fa-cog" title="Edit Settings"/>
                    </button>
                    <button class="alert alert-warning d-block mt-2 text-start"
                        icon="fa-arrow-right" type="action" role="alert"
                        name="%(base.res_config_setting_act_window)d"
                        attrs="{'invisible': ['|', ('is_microsoft_outlook_configured', '=', True), ('smtp_authentication', '!=', 'outlook')]}">
                        Setup your Outlook API credentials in the general settings to link a Outlook account.
                    </button>
                </div>
            </field>
            <field name="smtp_authentication_info" position="after">
                <a href="https://www.odoo.com/documentation/16.0/applications/general/email_communication/email_servers.html?highlight=outgoing email server#use-a-default-from-email-address"
                    attrs="{'invisible': [('smtp_authentication', '!=', 'outlook')]}" target="_blank">
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
                <a t-attf-href="/web?#id=#{rec_id}&amp;model=#{model_name}&amp;view_type=form">
                    Go back to your mail server
                </a>
            </div>
        </div>
    </template>
</odoo>

```

