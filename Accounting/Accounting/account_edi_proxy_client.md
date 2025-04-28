# Odoo Module: account_edi_proxy_client

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

def _create_demo_config_param(env):
    env['ir.config_parameter'].set_param('account_edi_proxy_client.demo', 'demo')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Proxy features for account_edi',
    'description': """
This module adds generic features to register an Odoo DB on the proxy responsible for receiving data (via requests from web-services).
- An edi_proxy_user has a unique identification on a specific proxy type (e.g. l10n_it_edi, peppol) which
allows to identify him when receiving a document addressed to him. It is linked to a specific company on a specific
Odoo database.
- Encryption features allows to decrypt all the user's data when receiving it from the proxy.
- Authentication offers an additionnal level of security to avoid impersonification, in case someone gains to the user's database.
    """,
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'depends': ['account', 'certificate'],
    'external_dependencies': {
        'python': ['cryptography']
    },
    'data': [
        'security/ir.model.access.csv',
        'security/account_edi_proxy_client_security.xml',
        'views/account_edi_proxy_user_views.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
    'post_init_hook': '_create_demo_config_param',
}

```

## File: data\neutralize.sql

```sql
-- disable edi connections in general, and the Italian one (l10n_it_edi_sdicoop) in particular
-- for malaysian edi, this script can cause issue as you could have both a demo and prod user, in which case it breaks the unique constrain. Another neutralize in the malaysian module disable the clients instead.
UPDATE account_edi_proxy_client_user
SET edi_mode = CASE
                    WHEN proxy_type = 'l10n_it_edi' THEN 'demo'
                    ELSE 'test'
               END
WHERE proxy_type != 'l10n_my_edi';

```

## File: i18n_extra\account_edi_proxy_client.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* account_edi_proxy_client
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 14.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2021-06-03 13:44+0000\n"
"PO-Revision-Date: 2021-06-03 13:44+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: account_edi_proxy_client
#: model:ir.model,name:account_edi_proxy_client.model_account_edi_proxy_client_user
msgid "Account EDI proxy user"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_res_company__account_edi_proxy_client_ids
msgid "Account Edi Proxy Client"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__active
msgid "Active"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__edi_format_code
msgid "Code"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model,name:account_edi_proxy_client.model_res_company
msgid "Companies"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__company_id
msgid "Company"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__create_uid
msgid "Created by"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__create_date
msgid "Created on"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_format__display_name
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__display_name
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_res_company__display_name
msgid "Display Name"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model,name:account_edi_proxy_client.model_account_edi_format
msgid "EDI format"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__edi_format_id
msgid "Edi Format"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__edi_identification
msgid "Edi Identification"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_format__id
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__id
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_res_company__id
msgid "ID"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__id_client
msgid "Id Client"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_format____last_update
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user____last_update
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_res_company____last_update
msgid "Last Modified on"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__write_uid
msgid "Last Updated by"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__write_date
msgid "Last Updated on"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__private_key
msgid "Private Key"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,field_description:account_edi_proxy_client.field_account_edi_proxy_client_user__refresh_token
msgid "Refresh Token"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,help:account_edi_proxy_client.field_account_edi_proxy_client_user__private_key
msgid "The key to encrypt all the user's data"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.fields,help:account_edi_proxy_client.field_account_edi_proxy_client_user__edi_identification
msgid ""
"The unique id that identifies this user for on the edi format, typically the"
" vat"
msgstr ""

#. module: account_edi_proxy_client
#: code:addons/account_edi_proxy_client/models/account_edi_proxy_user.py:0
msgid ""
"The url that this service does not exist. The url it tried to contact was %s"
msgstr ""

#. module: account_edi_proxy_client
#: code:addons/account_edi_proxy_client/models/account_edi_proxy_user.py:0
msgid ""
"The url that this service requested returned an error. The url it tried to "
"contact was %s"
msgstr ""

#. module: account_edi_proxy_client
#: code:addons/account_edi_proxy_client/models/account_edi_proxy_user.py:0
msgid ""
"The url that this service requested returned an error. The url it tried to "
"contact was %s. %s"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.constraint,message:account_edi_proxy_client.constraint_account_edi_proxy_client_user_unique_edi_identification_per_format
msgid "This edi identification is already assigned to a user"
msgstr ""

#. module: account_edi_proxy_client
#: model:ir.model.constraint,message:account_edi_proxy_client.constraint_account_edi_proxy_client_user_unique_id_client
msgid "This id_client is already used on another user."
msgstr ""

```

## File: models\account_edi_proxy_auth.py

```python
import base64
import hashlib
import hmac
import json
import requests
import time
import werkzeug.urls


class OdooEdiProxyAuth(requests.auth.AuthBase):
    """ For routes that needs to be authenticated and verified for access.
        Allows:
        1) to preserve the integrity of the message between the endpoints.
        2) to check user access rights and account validity
        3) to avoid that multiple database use the same credentials, via a refresh_token that expire after 24h.
    """

    def __init__(self, user=False):
        self.id_client = user and user.id_client or False
        self.refresh_token = user and user.sudo().refresh_token or False

    def __call__(self, request):
        # We don't sign request that still don't have a id_client/refresh_token
        if not self.id_client or not self.refresh_token:
            return request
        # craft the message (timestamp|url path|id_client|query params|body content)
        msg_timestamp = int(time.time())
        parsed_url = werkzeug.urls.url_parse(request.path_url)

        body = request.body
        if isinstance(body, bytes):
            body = body.decode()
        body = json.loads(body)

        message = '%s|%s|%s|%s|%s' % (
            msg_timestamp,  # timestamp
            parsed_url.path,  # url path
            self.id_client,
            json.dumps(werkzeug.urls.url_decode(parsed_url.query), sort_keys=True),  # url query params sorted by key
            json.dumps(body, sort_keys=True))  # http request body
        h = hmac.new(base64.b64decode(self.refresh_token), message.encode(), digestmod=hashlib.sha256)

        request.headers.update({
            'odoo-edi-client-id': self.id_client,
            'odoo-edi-signature': h.hexdigest(),
            'odoo-edi-timestamp': msg_timestamp,
        })
        return request

```

## File: models\account_edi_proxy_user.py

```python
import base64
import logging
import uuid

import psycopg2.errors
import requests

from odoo import _, fields, models
from odoo.exceptions import UserError
from odoo.tools import index_exists
from .account_edi_proxy_auth import OdooEdiProxyAuth

_logger = logging.getLogger(__name__)

TIMEOUT = 30


class AccountEdiProxyError(Exception):

    def __init__(self, code, message=False):
        self.code = code
        self.message = message
        super().__init__(message or code)


class AccountEdiProxyClientUser(models.Model):
    """Represents a user of the proxy for an electronic invoicing format.
    An edi_proxy_user has a unique identification on a specific format (for example, the vat for Peppol) which
    allows to identify him when receiving a document addressed to him. It is linked to a specific company on a specific
    Odoo database.
    It also owns a key with which each file should be decrypted with (the proxy encrypt all the files with the public key).
    """
    _name = 'account_edi_proxy_client.user'
    _description = 'Account EDI proxy user'

    active = fields.Boolean(default=True)
    id_client = fields.Char(required=True)
    company_id = fields.Many2one('res.company', string='Company', required=True,
        default=lambda self: self.env.company)
    edi_identification = fields.Char(required=True, help="The unique id that identifies this user, typically the vat")
    private_key_id = fields.Many2one(
        string='Private Key',
        comodel_name='certificate.key',
        required=True,
        domain=[('public', '=', False)],
        help="The key to encrypt all the user's data",
    )
    refresh_token = fields.Char(groups="base.group_system")
    proxy_type = fields.Selection(selection=[], required=True)
    edi_mode = fields.Selection(
        selection=[
            ('prod', 'Production mode'),
            ('test', 'Test mode'),
            ('demo', 'Demo mode'),
        ],
        string='EDI operating mode',
    )

    _sql_constraints = [
        ('unique_id_client', 'unique(id_client)', 'This id_client is already used on another user.'),
        ('unique_active_edi_identification', '', 'This edi identification is already assigned to an active user'),
        ('unique_active_company_proxy', '', 'This company has an active user already created for this EDI type'),
    ]

    def _auto_init(self):
        super()._auto_init()
        if not index_exists(self.env.cr, 'account_edi_proxy_client_user_unique_active_edi_identification'):
            self.env.cr.execute("""
                CREATE UNIQUE INDEX account_edi_proxy_client_user_unique_active_edi_identification
                                 ON account_edi_proxy_client_user(edi_identification, proxy_type, edi_mode)
                              WHERE (active = True)
            """)
        if not index_exists(self.env.cr, 'account_edi_proxy_client_user_unique_active_company_proxy'):
            self.env.cr.execute("""
                CREATE UNIQUE INDEX account_edi_proxy_client_user_unique_active_company_proxy
                                 ON account_edi_proxy_client_user(company_id, proxy_type, edi_mode)
                              WHERE (active = True)
            """)

    def _get_proxy_urls(self):
        # To extend
        return {}

    def _get_server_url(self, proxy_type=None, edi_mode=None):
        proxy_type = proxy_type or self.proxy_type
        edi_mode = edi_mode or self.edi_mode
        proxy_urls = self._get_proxy_urls()
        # letting this traceback in case of a KeyError, as that would mean something's wrong with the code
        return proxy_urls[proxy_type][edi_mode]

    def _get_proxy_users(self, company, proxy_type):
        '''Returns proxy users associated with the given company and proxy type.
        '''
        return company.account_edi_proxy_client_ids.filtered(lambda u: u.proxy_type == proxy_type)

    def _get_proxy_identification(self, company, proxy_type):
        '''Returns the key that will identify company uniquely
        within a specific proxy type and edi operating mode.
        or raises a UserError (if the user didn't fill the related field).
        TO OVERRIDE
        '''
        return False

    def _make_request(self, url, params=False):
        ''' Make a request to proxy and handle the generic elements of the reponse (errors, new refresh token).
        '''
        payload = {
            'jsonrpc': '2.0',
            'method': 'call',
            'params': params or {},
            'id': uuid.uuid4().hex,
        }

        # Last barrier : in case the demo mode is not handled by the caller, we block access.
        if self.edi_mode == 'demo':
            raise AccountEdiProxyError("block_demo_mode", "Can't access the proxy in demo mode")

        try:
            response = requests.post(
                url,
                json=payload,
                timeout=TIMEOUT,
                headers={'content-type': 'application/json'},
                auth=OdooEdiProxyAuth(user=self)).json()
        except (ValueError, requests.exceptions.ConnectionError, requests.exceptions.MissingSchema, requests.exceptions.Timeout, requests.exceptions.HTTPError):
            raise AccountEdiProxyError('connection_error',
                _('The url that this service requested returned an error. The url it tried to contact was %s', url))

        if 'error' in response:
            message = _('The url that this service requested returned an error. The url it tried to contact was %(url)s. %(error_message)s', url=url, error_message=response['error']['message'])
            if response['error']['code'] == 404:
                message = _('The url that this service tried to contact does not exist. The url was “%s”', url)
            raise AccountEdiProxyError('connection_error', message)

        proxy_error = response['result'].pop('proxy_error', False)
        if proxy_error:
            error_code = proxy_error['code']
            if error_code == 'refresh_token_expired':
                self._renew_token()
                self.env.cr.commit() # We do not want to lose it if in the _make_request below something goes wrong
                return self._make_request(url, params)
            if error_code == 'no_such_user':
                # This error is also raised if the user didn't exchange data and someone else claimed the edi_identificaiton.
                self.sudo().active = False
            if error_code == 'invalid_signature':
                raise AccountEdiProxyError(
                    error_code,
                    _("Invalid signature for request. This might be due to another connection to odoo Access Point "
                      "server. It can occur if you have duplicated your database. \n\n"
                      "If you are not sure how to fix this, please contact our support."),
                )
            raise AccountEdiProxyError(error_code, proxy_error['message'] or False)

        return response['result']

    def _register_proxy_user(self, company, proxy_type, edi_mode):
        ''' Generate the public_key/private_key that will be used to encrypt the file, send a request to the proxy
        to register the user with the public key and create the user with the private key.

        :param company: the company of the user.
        '''
        private_key_sudo = self.env['certificate.key'].sudo()._generate_rsa_private_key(
            company,
            name=f"{proxy_type}_{edi_mode}_{company.id}.key",
        )
        edi_identification = self._get_proxy_identification(company, proxy_type)
        if edi_mode == 'demo':
            # simulate registration
            response = {'id_client': f'demo{company.id}{proxy_type}', 'refresh_token': 'demo'}
        else:
            try:
                # b64encode returns a bytestring, we need it as a string
                response = self._make_request(self._get_server_url(proxy_type, edi_mode) + '/iap/account_edi/2/create_user', params={
                    'dbuuid': company.env['ir.config_parameter'].get_param('database.uuid'),
                    'company_id': company.id,
                    'edi_identification': edi_identification,
                    'public_key': private_key_sudo._get_public_key_bytes(encoding='pem').decode(),
                    'proxy_type': proxy_type,
                })
            except AccountEdiProxyError as e:
                raise UserError(e.message)
            if 'error' in response:
                if response['error'] == 'A user already exists with this identification.':
                    # Note: Peppol IAP errors weren't made properly with error code that are then translated on
                    # Odoo side. We are for now forced to check the error message.
                    raise UserError(_('A user already exists with theses credentials on our server. Please check your information.'))
                raise UserError(response['error'])

        return self.create({
            'id_client': response['id_client'],
            'company_id': company.id,
            'proxy_type': proxy_type,
            'edi_mode': edi_mode,
            'edi_identification': edi_identification,
            'private_key_id': private_key_sudo.id,
            'refresh_token': response['refresh_token'],
        })

    def _renew_token(self):
        ''' Request the proxy for a new refresh token.

        Request to the proxy should be made with a refresh token that expire after 24h to avoid
        that multiple database use the same credentials. When receiving an error for an expired refresh_token,
        This method makes a request to get a new refresh token.
        '''
        try:
            with self.env.cr.savepoint(flush=False):
                self.env.cr.execute('SELECT * FROM account_edi_proxy_client_user WHERE id IN %s FOR UPDATE NOWAIT', [tuple(self.ids)])
        except psycopg2.errors.LockNotAvailable:
            return
        response = self._make_request(self._get_server_url() + '/iap/account_edi/1/renew_token')
        if 'error' in response:
            # can happen if the database was duplicated and the refresh_token was refreshed by the other database.
            # we don't want two database to be able to query the proxy with the same user
            # because it could lead to not inconsistent data.
            _logger.error(response['error'])
        self.sudo().refresh_token = response['refresh_token']

    def _decrypt_data(self, data, symmetric_key):
        ''' Decrypt the data. Note that the data is encrypted with a symmetric key, which is encrypted with an asymmetric key.
        We must therefore decrypt the symmetric key.

        :param data:            The data to decrypt.
        :param symmetric_key:   The symmetric_key encrypted with self.private_key_id.public_key()
        '''
        decrypted_key = self.sudo().private_key_id._decrypt(base64.b64decode(symmetric_key))
        return self.env['certificate.key']._account_edi_fernet_decrypt(decrypted_key, base64.b64decode(data))

```

## File: models\key.py

```python
from cryptography.fernet import Fernet

from odoo import api, models


class Key(models.Model):
    _inherit = 'certificate.key'

    @api.model
    def _account_edi_fernet_decrypt(self, key, message):
        key = Fernet(key)
        return key.decrypt(message)

```

## File: models\res_company.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    account_edi_proxy_client_ids = fields.One2many('account_edi_proxy_client.user', inverse_name='company_id', context={'active_test': True})

```

## File: models\__init__.py

```python
from . import account_edi_proxy_user
from . import key
from . import res_company

```

## File: security\account_edi_proxy_client_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="0">
    <record id="account_edi_proxy_client_user_comp_rule" model="ir.rule">
        <field name="name">Account EDI Proxy Client User</field>
        <field name="model_id" ref="model_account_edi_proxy_client_user"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>
</data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_account_edi_proxy_manager","access_account_edi_proxy_user","model_account_edi_proxy_client_user","base.group_system",1,1,1,1
"access_account_edi_proxy_user","access_account_edi_proxy_user","model_account_edi_proxy_client_user","account.group_account_invoice",1,0,0,0

```

## File: views\account_edi_proxy_user_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_form_account_edi_proxy_client_user" model="ir.ui.view">
            <field name="name">EDI Proxy User</field>
            <field name="model">account_edi_proxy_client.user</field>
            <field name="arch" type="xml">
                <form string="Account Journal">
                    <sheet>
                        <group>
                            <group>
                                <field name="company_id" groups="base.group_multi_company" invisible="1"/>
                                <field name="proxy_type" readonly="1" options="{'no_open': True}"/>
                                <field name="id_client" readonly="1"/>
                                <field name="edi_identification" readonly="1"/>
                                <field name="private_key_id" readonly="1"/>
                                <field name="refresh_token" readonly="1"/>
                            </group>
                            <group/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_tree_account_edi_proxy_client_user" model="ir.ui.view">
            <field name="name">EDI Proxy Users</field>
            <field name="model">account_edi_proxy_client.user</field>
            <field name="arch" type="xml">
                <list create="false" delete="false" edit="false">
                    <field name="company_id" groups="base.group_multi_company" column_invisible="True"/>
                    <field name="id_client" readonly="1"/>
                    <field name="edi_identification" readonly="1"/>
                    <field name="private_key_id" readonly="1"/>
                    <field name="refresh_token" readonly="1"/>
                </list>
            </field>
        </record>

        <record id="action_tree_account_edi_proxy_client_user" model="ir.actions.act_window">
            <field name="name">EDI Proxy User</field>
            <field name="res_model">account_edi_proxy_client.user</field>
            <field name="view_mode">list,form</field>
            <field name="view_id" ref="view_tree_account_edi_proxy_client_user"/>
        </record>

        <menuitem name="EDI Proxy Users"
                  sequence="2"
                  parent="account.menu_finance_configuration"
                  id="menu_account_proxy_client_user"
                  action="action_tree_account_edi_proxy_client_user"
                  groups="base.group_no_one"/>

    </data>
</odoo>

```

