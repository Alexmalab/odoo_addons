# Odoo Module: account_edi_proxy_client

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from odoo import api, SUPERUSER_ID

def _create_demo_config_param(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env['ir.config_parameter'].set_param('account_edi_proxy_client.demo', 'demo')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Proxy features for account_edi',
    'description': """
This module adds generic features to register an Odoo DB on the proxy responsible for receiving data (via requests from web-services).
- An edi_proxy_user has a unique identification on a specific format (for example, the vat for Peppol) which
allows to identify him when receiving a document addressed to him. It is linked to a specific company on a specific
Odoo database.
- Encryption features allows to decrypt all the user's data when receiving it from the proxy.
- Authentication offers an additionnal level of security to avoid impersonification, in case someone gains to the user's database.
    """,
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'depends': ['account_edi'],
    'external_dependencies': {
        'python': ['cryptography']
    },
    'data': [
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'application': False,
    'license': 'LGPL-3',
    'post_init_hook': '_create_demo_config_param',
}

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
#, python-format
msgid ""
"The url that this service does not exist. The url it tried to contact was %s"
msgstr ""

#. module: account_edi_proxy_client
#: code:addons/account_edi_proxy_client/models/account_edi_proxy_user.py:0
#, python-format
msgid ""
"The url that this service requested returned an error. The url it tried to "
"contact was %s"
msgstr ""

#. module: account_edi_proxy_client
#: code:addons/account_edi_proxy_client/models/account_edi_proxy_user.py:0
#, python-format
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

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-
from odoo import models


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    # -------------------------------------------------------------------------
    # Helpers
    # -------------------------------------------------------------------------

    def _get_proxy_user(self, company):
        '''Returns the proxy_user associated with this edi format.
        '''
        self.ensure_one()
        return company.account_edi_proxy_client_ids.filtered(lambda u: u.edi_format_id == self)

    # -------------------------------------------------------------------------
    # To override
    # -------------------------------------------------------------------------

    def _get_proxy_identification(self, company):
        '''Returns the key that will identify company uniquely for this edi format (for example, the vat)
        or raises a UserError (if the user didn't fill the related field).
        TO OVERRIDE
        '''
        return False

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
from odoo import models, fields, _
from odoo.exceptions import UserError
from .account_edi_proxy_auth import OdooEdiProxyAuth

from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.fernet import Fernet
from psycopg2 import OperationalError
import requests
import uuid
import base64
import logging


_logger = logging.getLogger(__name__)


DEFAULT_SERVER_URL = 'https://l10n-it-edi.api.odoo.com'
DEFAULT_TEST_SERVER_URL = 'https://iap-services-test.odoo.com'
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
    id_client = fields.Char(required=True, index=True)
    company_id = fields.Many2one('res.company', string='Company', required=True,
        default=lambda self: self.env.company)
    edi_format_id = fields.Many2one('account.edi.format', required=True)
    edi_format_code = fields.Char(related='edi_format_id.code', readonly=True)
    edi_identification = fields.Char(required=True, help="The unique id that identifies this user for on the edi format, typically the vat")
    private_key = fields.Binary(required=True, attachment=False, groups="base.group_system", help="The key to encrypt all the user's data")
    refresh_token = fields.Char(groups="base.group_system")

    _sql_constraints = [
        ('unique_id_client', 'unique(id_client)', 'This id_client is already used on another user.'),
        ('unique_edi_identification_per_format', 'unique(edi_identification, edi_format_id)', 'This edi identification is already assigned to a user'),
    ]

    def _get_demo_state(self):
        demo_state = self.env['ir.config_parameter'].sudo().get_param('account_edi_proxy_client.demo', False)
        return 'prod' if demo_state in ['prod', False] else 'test' if demo_state == 'test' else 'demo'

    def _get_server_url(self):
        return DEFAULT_TEST_SERVER_URL if self._get_demo_state() == 'test' else self.env['ir.config_parameter'].sudo().get_param('account_edi_proxy_client.edi_server_url', DEFAULT_SERVER_URL)

    def _make_request(self, url, params=False):
        ''' Make a request to proxy and handle the generic elements of the reponse (errors, new refresh token).
        '''
        payload = {
            'jsonrpc': '2.0',
            'method': 'call',
            'params': params or {},
            'id': uuid.uuid4().hex,
        }

        if self._get_demo_state() == 'demo':
            # Last barrier : in case the demo mode is not handled by the caller, we block access.
            raise Exception("Can't access the proxy in demo mode")

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
            message = _('The url that this service requested returned an error. The url it tried to contact was %s. %s', url, response['error']['message'])
            if response['error']['code'] == 404:
                message = _('The url that this service does not exist. The url it tried to contact was %s', url)
            raise AccountEdiProxyError('connection_error', message)

        proxy_error = response['result'].pop('proxy_error', False)
        if proxy_error:
            error_code = proxy_error['code']
            if error_code == 'refresh_token_expired':
                self._renew_token()
                if not self.env.context.get('test_skip_commit'):
                    self.env.cr.commit() # We do not want to lose it if in the _make_request below something goes wrong
                return self._make_request(url, params)
            if error_code == 'no_such_user':
                # This error is also raised if the user didn't exchange data and someone else claimed the edi_identificaiton.
                self.sudo().active = False
            raise AccountEdiProxyError(error_code, proxy_error['message'] or False)

        return response['result']

    def _register_proxy_user(self, company, edi_format, edi_identification):
        ''' Generate the public_key/private_key that will be used to encrypt the file, send a request to the proxy
        to register the user with the public key and create the user with the private key.

        :param company: the company of the user.
        :param edi_identification: The unique ID that identifies this user on this edi network and to which the files will be addressed.
                                   Typically the vat.
        '''
        # public_exponent=65537 is a default value that should be used most of the time, as per the documentation of cryptography.
        # key_size=2048 is considered a reasonable default key size, as per the documentation of cryptography.
        # see https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa/
        private_key = rsa.generate_private_key(
            public_exponent=65537,
            key_size=2048,
            backend=default_backend()
        )
        private_pem = private_key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.PKCS8,
            encryption_algorithm=serialization.NoEncryption()
        )
        public_key = private_key.public_key()
        public_pem = public_key.public_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PublicFormat.SubjectPublicKeyInfo
        )
        if self._get_demo_state() == 'demo':
            # simulate registration
            response = {'id_client': f'demo{company.id}', 'refresh_token': 'demo'}
        else:
            try:
                # b64encode returns a bytestring, we need it as a string
                response = self._make_request(self._get_server_url() + '/iap/account_edi/1/create_user', params={
                    'dbuuid': company.env['ir.config_parameter'].get_param('database.uuid'),
                    'company_id': company.id,
                    'edi_format_code': edi_format.code,
                    'edi_identification': edi_identification,
                    'public_key': base64.b64encode(public_pem).decode()
                })
            except AccountEdiProxyError as e:
                raise UserError(e.message)
            if 'error' in response:
                raise UserError(response['error'])

        self.create({
            'id_client': response['id_client'],
            'company_id': company.id,
            'edi_format_id': edi_format.id,
            'edi_identification': edi_identification,
            'private_key': base64.b64encode(private_pem),
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
        except OperationalError as e:
            if e.pgcode == '55P03':
                return
            raise e
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
        :param symmetric_key:   The symmetric_key encrypted with self.private_key.public_key()
        '''
        private_key = serialization.load_pem_private_key(
            base64.b64decode(self.sudo().private_key),
            password=None,
            backend=default_backend()
        )
        key = private_key.decrypt(
            base64.b64decode(symmetric_key),
            padding.OAEP(
                mgf=padding.MGF1(algorithm=hashes.SHA256()),
                algorithm=hashes.SHA256(),
                label=None
            )
        )
        f = Fernet(key)
        return f.decrypt(base64.b64decode(data))

```

## File: models\res_company.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    account_edi_proxy_client_ids = fields.One2many('account_edi_proxy_client.user', inverse_name='company_id')

```

## File: models\__init__.py

```python
from . import account_edi_format
from . import account_edi_proxy_user
from . import res_company

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_account_edi_proxy_manager","access_account_edi_proxy_user","model_account_edi_proxy_client_user","base.group_system",1,1,1,1
"access_account_edi_proxy_user","access_account_edi_proxy_user","model_account_edi_proxy_client_user","account.group_account_invoice",1,0,0,0

```

