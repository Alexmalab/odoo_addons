# Odoo Module: cloud_storage_azure

Category: Technical Settings

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


def uninstall_hook(env):
    ICP = env['ir.config_parameter']
    if ICP.get_param('cloud_storage_provider') == 'azure':
        env['res.config.settings']._check_cloud_storage_uninstallable()
        ICP.set_param('cloud_storage_provider', False)
    ICP.search([('key', 'in', [
        'cloud_storage_azure_container_name',
        'cloud_storage_azure_account_name',
        'cloud_storage_azure_tenant_id',
        'cloud_storage_azure_client_id',
        'cloud_storage_azure_client_secret',
    ])]).unlink()

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": "Cloud Storage Azure",
    "summary": """Store chatter attachments in the Azure cloud""",
    "category": "Technical Settings",
    "version": "1.0",
    "depends": ["cloud_storage"],
    "data": [
        "views/settings.xml",
    ],
    "uninstall_hook": "uninstall_hook",
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
DELETE
FROM ir_config_parameter
WHERE key IN ('cloud_storage_azure_account_name',
              'cloud_storage_azure_container_name',
              'cloud_storage_azure_tenant_id',
              'cloud_storage_azure_client_id',
              'cloud_storage_azure_client_secret')

```

## File: models\ir_attachment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from datetime import datetime, timedelta, timezone
from urllib.parse import unquote, quote

from odoo import models
from odoo.exceptions import ValidationError

from ..utils.cloud_storage_azure_utils import generate_blob_sas, get_user_delegation_key, ClientAuthenticationError

CloudStorageAzureUserDelegationKeys = {}  # {db_name: (config, user_delegation_key or exception)}


def get_cloud_storage_azure_user_delegation_key(env):
    """
    Generate a UserDelegationKey used for generating SAS tokens.

    The cached UserDelegationKey is refreshed every 6 days. If the account
    information expires, a ClientAuthenticationError will also be cached to
    prevent the server from repeatedly sending requests to the cloud
    storage provider to get the user delegation key. Note that the cached
    values are not invalidated when the ORM cache is invalidated. To
    invalidate these cached values, you must update the cloud storage
    configuration or the cloud_storage_azure_user_delegation_key_sequence.

    :return: A valid and unexpired UserDelegationKey which is compatible
            with azure.storage.blob.UserDelegationKey
    """
    cached_config, cached_user_delegation_key = CloudStorageAzureUserDelegationKeys.get(env.registry.db_name, (None, None))
    db_config = env['res.config.settings']._get_cloud_storage_configuration()
    db_config.pop('container_name')
    ICP = env['ir.config_parameter'].sudo()
    db_config['sequence'] = int(ICP.get_param('cloud_storage_azure_user_delegation_key_sequence', 0))
    if db_config == cached_config:
        if isinstance(cached_user_delegation_key, Exception):
            raise cached_user_delegation_key
        if cached_user_delegation_key:
            expiry = datetime.strptime(cached_user_delegation_key.signed_expiry, '%Y-%m-%dT%H:%M:%SZ').replace(tzinfo=timezone.utc)
            if expiry > datetime.now(timezone.utc) + timedelta(days=1):
                return cached_user_delegation_key
    key_start_time = datetime.now(timezone.utc)
    key_expiry_time = key_start_time + timedelta(days=7)
    try:
        user_delegation_key = get_user_delegation_key(
            tenant_id=db_config['tenant_id'],
            client_id=db_config['client_id'],
            client_secret=db_config['client_secret'],
            account_name=db_config['account_name'],
            key_start_time=key_start_time,
            key_expiry_time=key_expiry_time,
        )
        CloudStorageAzureUserDelegationKeys[env.registry.db_name] = (db_config, user_delegation_key)
    except ClientAuthenticationError as e:
        ve = ValidationError(e)
        CloudStorageAzureUserDelegationKeys[env.registry.db_name] = (db_config, ve)
        raise ve
    return user_delegation_key


class IrAttachment(models.Model):
    _inherit = 'ir.attachment'
    _cloud_storage_azure_url_pattern = re.compile(r'https://(?P<account_name>[\w]+).blob.core.windows.net/(?P<container_name>[\w]+)/(?P<blob_name>[^?]+)')

    def _get_cloud_storage_azure_info(self):
        match = self._cloud_storage_azure_url_pattern.match(self.url)
        if not match:
            raise ValidationError('%s is not a valid Azure Blob Storage URL.', self.url)
        return {
            'account_name': match['account_name'],
            'container_name': match['container_name'],
            'blob_name': unquote(match['blob_name']),
        }

    def _generate_cloud_storage_azure_url(self, blob_name):
        ICP = self.env['ir.config_parameter'].sudo()
        account_name = ICP.get_param('cloud_storage_azure_account_name')
        container_name = ICP.get_param('cloud_storage_azure_container_name')
        return f"https://{account_name}.blob.core.windows.net/{container_name}/{quote(blob_name)}"

    def _generate_cloud_storage_azure_sas_url(self, **kwargs):
        token = generate_blob_sas(user_delegation_key=get_cloud_storage_azure_user_delegation_key(self.env), **kwargs)
        return f"{self._generate_cloud_storage_azure_url(kwargs['blob_name'])}?{token}"

    # OVERRIDES
    def _generate_cloud_storage_url(self):
        if self.env['ir.config_parameter'].sudo().get_param('cloud_storage_provider') != 'azure':
            return super()._generate_cloud_storage_url()
        blob_name = self._generate_cloud_storage_blob_name()
        return self._generate_cloud_storage_azure_url(blob_name)

    def _generate_cloud_storage_download_info(self):
        if self.env['ir.config_parameter'].sudo().get_param('cloud_storage_provider') != 'azure':
            return super()._generate_cloud_storage_download_info()
        info = self._get_cloud_storage_azure_info()
        expiry = datetime.now(timezone.utc) + timedelta(seconds=self._cloud_storage_download_url_time_to_expiry)
        return {
            'url': self._generate_cloud_storage_azure_sas_url(**info, permission='r', expiry=expiry, cache_control=f'private, max-age={self._cloud_storage_download_url_time_to_expiry}'),
            'time_to_expiry': self._cloud_storage_download_url_time_to_expiry,
        }

    def _generate_cloud_storage_upload_info(self):
        if self.env['ir.config_parameter'].sudo().get_param('cloud_storage_provider') != 'azure':
            return super()._generate_cloud_storage_upload_info()
        info = self._get_cloud_storage_azure_info()
        expiry = datetime.now(timezone.utc) + timedelta(seconds=self._cloud_storage_upload_url_time_to_expiry)
        url = self._generate_cloud_storage_azure_sas_url(**info, permission='c', expiry=expiry)
        return {
            'url': url,
            'method': 'PUT',
            'headers': {
                'x-ms-blob-type': 'BlockBlob',
            },
            'response_status': 201,
        }

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import requests
from datetime import datetime, timedelta, timezone

from odoo import models, fields, _
from odoo.exceptions import ValidationError, UserError


class CloudStorageSettings(models.TransientModel):
    """
    Instructions:
    cloud_storage_azure_account_name, cloud_storage_azure_container_name:
        if changed and old container names are still in use, you should
        promise the current application registration has the permission
        to access all old containers.
    cloud_storage_azure_invalidate_user_delegation_key:
        invalidate the cached value for
        get_cloud_storage_azure_user_delegation_key
    """
    _inherit = 'res.config.settings'

    cloud_storage_provider = fields.Selection(selection_add=[('azure', 'Azure Cloud Storage')])

    cloud_storage_azure_account_name = fields.Char(
        string='Azure Account Name',
        config_parameter='cloud_storage_azure_account_name')
    cloud_storage_azure_container_name = fields.Char(
        string='Azure Container Name',
        config_parameter='cloud_storage_azure_container_name')
    # Application Registry Info
    cloud_storage_azure_tenant_id = fields.Char(
        string='Azure Tenant ID',
        config_parameter='cloud_storage_azure_tenant_id')
    cloud_storage_azure_client_id = fields.Char(
        string='Azure Client ID',
        config_parameter='cloud_storage_azure_client_id')
    cloud_storage_azure_client_secret = fields.Char(
        string='Azure Client Secret',
        config_parameter='cloud_storage_azure_client_secret')
    cloud_storage_azure_invalidate_user_delegation_key = fields.Boolean(
        string='Invalidate Cached Azure User Delegation Key',
    )

    def _get_cloud_storage_configuration(self):
        ICP = self.env['ir.config_parameter'].sudo()
        if ICP.get_param('cloud_storage_provider') != 'azure':
            return super()._get_cloud_storage_configuration
        configuration = {
            'container_name': ICP.get_param('cloud_storage_azure_container_name'),
            'account_name': ICP.get_param('cloud_storage_azure_account_name'),
            'tenant_id': ICP.get_param('cloud_storage_azure_tenant_id'),
            'client_id': ICP.get_param('cloud_storage_azure_client_id'),
            'client_secret': ICP.get_param('cloud_storage_azure_client_secret'),
        }
        return configuration if all(configuration.values()) else {}

    def _setup_cloud_storage_provider(self):
        ICP = self.env['ir.config_parameter'].sudo()
        if ICP.get_param('cloud_storage_provider') != 'azure':
            return super()._setup_cloud_storage_provider()
        blob_info = {
            'account_name': ICP.get_param('cloud_storage_azure_account_name'),
            'container_name': ICP.get_param('cloud_storage_azure_container_name'),
            # use different blob names in case the credentials are allowed to
            # overwrite an existing blob created by previous tests
            'blob_name': f'0/{datetime.now(timezone.utc)}.txt',
        }

        # check blob create permission
        upload_expiry = datetime.now(timezone.utc) + timedelta(seconds=self.env['ir.attachment']._cloud_storage_upload_url_time_to_expiry)
        upload_url = self.env['ir.attachment']._generate_cloud_storage_azure_sas_url(**blob_info, permission='c', expiry=upload_expiry)
        upload_response = requests.put(upload_url, data=b'', headers={'x-ms-blob-type': 'BlockBlob'}, timeout=5)
        if upload_response.status_code != 201:
            raise ValidationError(_('The connection string is not allowed to upload blobs to the container.\n%s', str(upload_response.text)))

        # check blob read permission
        download_expiry = datetime.now(timezone.utc) + timedelta(seconds=self.env['ir.attachment']._cloud_storage_download_url_time_to_expiry)
        download_url = self.env['ir.attachment']._generate_cloud_storage_azure_sas_url(**blob_info, permission='r', expiry=download_expiry)
        download_response = requests.get(download_url, timeout=5)
        if download_response.status_code != 200:
            raise ValidationError(_('The connection string is not allowed to download blobs from the container.\n%s', str(download_response.text)))

    def _check_cloud_storage_uninstallable(self):
        if self.env['ir.config_parameter'].get_param('cloud_storage_provider') != 'azure':
            return super()._check_cloud_storage_uninstallable()
        cr = self.env.cr
        cr.execute(
            """
                SELECT 1
                FROM ir_attachment
                WHERE type = 'cloud_storage'
                AND url LIKE 'https://%.blob.core.windows.net/%'
                LIMIT 1
            """,
        )
        if cr.fetchone():
            raise UserError(_('Some Azure attachments are in use, please migrate their cloud storages before disable this module'))

    def set_values(self):
        super().set_values()
        if self.cloud_storage_azure_invalidate_user_delegation_key:
            ICP = self.env['ir.config_parameter']
            old_seq = int(ICP.get_param('cloud_storage_azure_user_delegation_key_sequence', 0))
            ICP.set_param('cloud_storage_azure_user_delegation_key_sequence', old_seq + 1)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_attachment
from . import res_config_settings

```

## File: utils\cleanup_cloud_storage_azure.py

```python
import logging
import requests
import xmlrpc.client

from concurrent.futures import ThreadPoolExecutor
from itertools import islice
from lxml import etree
from urllib.parse import quote

# This is a script to manually clean up unused azure blobs.

# +--------+ 3. search_read ir.attachments with cloud urls +---------+
# |        | --------------------------------------------> |         |
# |        | <-------------------------------------------- |  Odoo   |
# |        | 4. used urls                                  |         |
# |        |                                               |         |
# | Script |                                               +---------+
# |        |                                               +---------+
# |        |                            1. list all blobs  |  Cloud  |
# |        | --------------------------------------------> | Storage |
# |        | <-------------------------------------------- |         |
# |        | 2. blobs names                                |         |
# |        |                                               |         |
# |        |                        5. delete unused blobs |         |
# |        | --------------------------------------------> |         |
# |        | <-------------------------------------------- |         |
# +--------+ 6. 202: Accepted                              +---------+
#
#
# 1, 2, 3, 4 are done in batch
# 5, 6 are done with threadpool

# odoo
odoo_url = 'http://localhost:8069'
odoo_db = 'odoo_db'
odoo_username = 'admin'
odoo_password = 'admin'

# Azure
X_MS_VERSION = '2023-11-03'
azure_container_name = 'container_name'
azure_account_name = 'account_name'
azure_tenant_id = 'tenant_id'
azure_client_id = 'client_id'
azure_client_secret = 'client_secret'

# Get Azure OAuth token
azure_token_url = f"https://login.microsoftonline.com/{azure_tenant_id}/oauth2/token"
azure_token_data = {
    'grant_type': 'client_credentials',
    'client_id': azure_client_id,
    'client_secret': azure_client_secret,
    'resource': 'https://storage.azure.com/'
}
azure_token_response = requests.post(azure_token_url, data=azure_token_data, timeout=5)
azure_token = azure_token_response.json()['access_token']

_logger = logging.getLogger()


def list_blob_urls(container_name, batch_size=1000):
    cloud_storage_blobs_num = 0
    url = f"https://{azure_account_name}.blob.core.windows.net/{container_name}?restype=container&comp=list"
    headers = {
        'Authorization': f'Bearer {azure_token}',
        'x-ms-version': X_MS_VERSION,
        'Content-Type': 'application/xml'
    }
    params = {
        'maxresults': batch_size,
    }

    while True:
        response = requests.get(url, headers=headers, params=params, timeout=5)
        response_xml = etree.fromstring(response.content)
        for blob in response_xml.findall('.//Blob'):
            cloud_storage_blobs_num += 1
            yield f"https://{azure_account_name}.blob.core.windows.net/{container_name}/{quote(blob.find('Name').text)}"

        params['marker'] = response_xml.find('NextMarker').text
        if params['marker'] is None:
            break

    logging.info('The cloud storage container has %d blobs', cloud_storage_blobs_num)


def split_every(n, iterable, piece_maker=tuple):
    iterator = iter(iterable)
    piece = piece_maker(islice(iterator, n))
    while piece:
        yield piece
        piece = piece_maker(islice(iterator, n))


def get_blobs_to_be_deleted(blob_urls, batch_size=1000):
    common = xmlrpc.client.ServerProxy(f'{odoo_url}/xmlrpc/2/common')
    uid = common.authenticate(odoo_db, odoo_username, odoo_password, {})
    models = xmlrpc.client.ServerProxy(f'{odoo_url}/xmlrpc/2/object')
    for blob_urls_ in split_every(batch_size, blob_urls):
        blob_urls_ = list(blob_urls_)
        attachments = models.execute_kw(odoo_db, uid, odoo_password, 'ir.attachment', 'search_read', [
            [('type', '=', 'cloud_storage'), ('url', 'in', blob_urls_)],
            ['url']
        ])
        used_urls_ = {attachment['url'] for attachment in attachments}
        for blob_url in blob_urls_:
            if blob_url not in used_urls_:
                yield blob_url


def delete_blobs(blob_urls, max_worker=None):
    headers = {
        'Authorization': f'Bearer {azure_token}',
        'x-ms-version': X_MS_VERSION,
        'Content-Type': 'application/xml'
    }
    deleted_cloud_storage_blobs_num = 0

    def delete_blob_(blob_url):
        nonlocal deleted_cloud_storage_blobs_num
        delete_response = requests.delete(blob_url, headers=headers, timeout=5)
        if delete_response.status_code == 202:
            deleted_cloud_storage_blobs_num += 1
            _logger.info('%s is deleted', blob_url)
        elif delete_response.status_code == 404:
            _logger.debug('%s has already been deleted', blob_url)
        else:
            _logger.warning('%s cannot be deleted:\n%s', blob_url, delete_response.text)

    with ThreadPoolExecutor(max_workers=max_worker) as executor:
        executor.map(delete_blob_, blob_urls)

    logging.info('%d blobs are deleted by the script', deleted_cloud_storage_blobs_num)


if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    all_blob_urls = list_blob_urls(container_name=azure_container_name, batch_size=1000)
    to_delete_blob_urls = get_blobs_to_be_deleted(all_blob_urls, batch_size=1000)
    delete_blobs(to_delete_blob_urls)

```

## File: utils\cloud_storage_azure_utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
"""
This module is used to provide an azure.storage.blob.generate_blob_sas
compatible function for generating sas URLs for Azure Storage without
importing the azure.storage.blob library.
"""

import base64
import hashlib
import hmac

import requests
from urllib.parse import quote
from datetime import date
from lxml import etree
from odoo.exceptions import ValidationError

X_MS_VERSION = '2023-11-03'


def sign_string(key, string_to_sign):
    key = base64.b64decode(key.encode())
    string_to_sign = string_to_sign.encode()
    signed_hmac_sha256 = hmac.HMAC(key, string_to_sign, hashlib.sha256)
    digest = signed_hmac_sha256.digest()
    encoded_digest = base64.b64encode(digest).decode()
    return encoded_digest


def _to_utc_datetime(value):
    return value.strftime('%Y-%m-%dT%H:%M:%SZ')


class QueryStringConstants:
    SIGNED_SIGNATURE = 'sig'
    SIGNED_PERMISSION = 'sp'
    SIGNED_START = 'st'
    SIGNED_EXPIRY = 'se'
    SIGNED_RESOURCE = 'sr'
    SIGNED_IDENTIFIER = 'si'
    SIGNED_IP = 'sip'
    SIGNED_PROTOCOL = 'spr'
    SIGNED_VERSION = 'sv'
    SIGNED_CACHE_CONTROL = 'rscc'
    SIGNED_CONTENT_DISPOSITION = 'rscd'
    SIGNED_CONTENT_ENCODING = 'rsce'
    SIGNED_CONTENT_LANGUAGE = 'rscl'
    SIGNED_CONTENT_TYPE = 'rsct'
    SIGNED_OID = 'skoid'
    SIGNED_TID = 'sktid'
    SIGNED_KEY_START = 'skt'
    SIGNED_KEY_EXPIRY = 'ske'
    SIGNED_KEY_SERVICE = 'sks'
    SIGNED_KEY_VERSION = 'skv'
    SIGNED_ENCRYPTION_SCOPE = 'ses'

    # for ADLS
    SIGNED_AUTHORIZED_OID = 'saoid'
    SIGNED_UNAUTHORIZED_OID = 'suoid'
    SIGNED_CORRELATION_ID = 'scid'


class BlobQueryStringConstants:
    SIGNED_TIMESTAMP = 'snapshot'


class _BlobSharedAccessHelper:
    def __init__(self):
        self.query_dict = {}

    def _add_query(self, name, val):
        if val:
            self.query_dict[name] = str(val)

    def get_value_to_append(self, query):
        return_value = self.query_dict.get(query) or ''
        return return_value + '\n'

    def add_base(self, permission, expiry, start, ip, protocol, x_ms_version):
        if isinstance(start, date):
            start = _to_utc_datetime(start)

        if isinstance(expiry, date):
            expiry = _to_utc_datetime(expiry)

        self._add_query(QueryStringConstants.SIGNED_START, start)
        self._add_query(QueryStringConstants.SIGNED_EXPIRY, expiry)
        self._add_query(QueryStringConstants.SIGNED_PERMISSION, permission)
        self._add_query(QueryStringConstants.SIGNED_IP, ip)
        self._add_query(QueryStringConstants.SIGNED_PROTOCOL, protocol)
        self._add_query(QueryStringConstants.SIGNED_VERSION, x_ms_version)

    def add_resource(self, resource):
        self._add_query(QueryStringConstants.SIGNED_RESOURCE, resource)

    def add_id(self, policy_id):
        self._add_query(QueryStringConstants.SIGNED_IDENTIFIER, policy_id)

    def add_override_response_headers(self, cache_control,
                                      content_disposition,
                                      content_encoding,
                                      content_language,
                                      content_type):
        self._add_query(QueryStringConstants.SIGNED_CACHE_CONTROL, cache_control)
        self._add_query(QueryStringConstants.SIGNED_CONTENT_DISPOSITION, content_disposition)
        self._add_query(QueryStringConstants.SIGNED_CONTENT_ENCODING, content_encoding)
        self._add_query(QueryStringConstants.SIGNED_CONTENT_LANGUAGE, content_language)
        self._add_query(QueryStringConstants.SIGNED_CONTENT_TYPE, content_type)

    def add_resource_signature(self, account_name, account_key, path, user_delegation_key=None):
        # pylint: disable = no-member
        if path[0] != '/':
            path = '/' + path

        canonicalized_resource = '/blob/' + account_name + path + '\n'

        # Form the string to sign from shared_access_policy and canonicalized
        # resource. The order of values is important.
        string_to_sign = \
            (self.get_value_to_append(QueryStringConstants.SIGNED_PERMISSION) +
             self.get_value_to_append(QueryStringConstants.SIGNED_START) +
             self.get_value_to_append(QueryStringConstants.SIGNED_EXPIRY) +
             canonicalized_resource)

        if user_delegation_key is not None:
            self._add_query(QueryStringConstants.SIGNED_OID, user_delegation_key.signed_oid)
            self._add_query(QueryStringConstants.SIGNED_TID, user_delegation_key.signed_tid)
            self._add_query(QueryStringConstants.SIGNED_KEY_START, user_delegation_key.signed_start)
            self._add_query(QueryStringConstants.SIGNED_KEY_EXPIRY, user_delegation_key.signed_expiry)
            self._add_query(QueryStringConstants.SIGNED_KEY_SERVICE, user_delegation_key.signed_service)
            self._add_query(QueryStringConstants.SIGNED_KEY_VERSION, user_delegation_key.signed_version)

            string_to_sign += \
                (self.get_value_to_append(QueryStringConstants.SIGNED_OID) +
                 self.get_value_to_append(QueryStringConstants.SIGNED_TID) +
                 self.get_value_to_append(QueryStringConstants.SIGNED_KEY_START) +
                 self.get_value_to_append(QueryStringConstants.SIGNED_KEY_EXPIRY) +
                 self.get_value_to_append(QueryStringConstants.SIGNED_KEY_SERVICE) +
                 self.get_value_to_append(QueryStringConstants.SIGNED_KEY_VERSION) +
                 self.get_value_to_append(QueryStringConstants.SIGNED_AUTHORIZED_OID) +
                 self.get_value_to_append(QueryStringConstants.SIGNED_UNAUTHORIZED_OID) +
                 self.get_value_to_append(QueryStringConstants.SIGNED_CORRELATION_ID))
        else:
            string_to_sign += self.get_value_to_append(QueryStringConstants.SIGNED_IDENTIFIER)

        string_to_sign += \
            (self.get_value_to_append(QueryStringConstants.SIGNED_IP) +
             self.get_value_to_append(QueryStringConstants.SIGNED_PROTOCOL) +
             self.get_value_to_append(QueryStringConstants.SIGNED_VERSION) +
             self.get_value_to_append(QueryStringConstants.SIGNED_RESOURCE) +
             self.get_value_to_append(BlobQueryStringConstants.SIGNED_TIMESTAMP) +
             self.get_value_to_append(QueryStringConstants.SIGNED_ENCRYPTION_SCOPE) +
             self.get_value_to_append(QueryStringConstants.SIGNED_CACHE_CONTROL) +
             self.get_value_to_append(QueryStringConstants.SIGNED_CONTENT_DISPOSITION) +
             self.get_value_to_append(QueryStringConstants.SIGNED_CONTENT_ENCODING) +
             self.get_value_to_append(QueryStringConstants.SIGNED_CONTENT_LANGUAGE) +
             self.get_value_to_append(QueryStringConstants.SIGNED_CONTENT_TYPE))

        # remove the trailing newline
        if string_to_sign[-1] == '\n':
            string_to_sign = string_to_sign[:-1]

        self._add_query(QueryStringConstants.SIGNED_SIGNATURE,
                        sign_string(account_key if user_delegation_key is None else user_delegation_key.value,
                                    string_to_sign))

    def get_token(self):
        # a conscious decision was made to exclude the timestamp in the generated token
        # this is to avoid having two snapshot ids in the query parameters when the user appends the snapshot timestamp
        exclude = [BlobQueryStringConstants.SIGNED_TIMESTAMP]
        return '&'.join([f'{n}={quote(v)}' for n, v in self.query_dict.items() if v is not None and n not in exclude])


class UserDelegationKey:
    """
    Represents a user delegation key, provided to the user by Azure Storage
    based on their Azure Active Directory access token.

    The fields are saved as simple strings since the user does not have to interact with this object;
    to generate an identify SAS, the user can simply pass it to the right API.

    :ivar str signed_oid:
        Object ID of this token.
    :ivar str signed_tid:
        Tenant ID of the tenant that issued this token.
    :ivar str signed_start:
        The datetime this token becomes valid.
    :ivar str signed_expiry:
        The datetime this token expires.
    :ivar str signed_service:
        What service this key is valid for.
    :ivar str signed_version:
        The version identifier of the REST service that created this token.
    :ivar str value:
        The user delegation key.
    """
    def __init__(self):
        self.signed_oid = None
        self.signed_tid = None
        self.signed_start = None
        self.signed_expiry = None
        self.signed_service = None
        self.signed_version = None
        self.value = None


def generate_blob_sas(
        account_name,
        container_name,
        blob_name,
        account_key=None,
        user_delegation_key=None,
        permission=None,
        expiry=None,
        start=None,
        policy_id=None,
        ip=None,
        protocol=None,
        cache_control=None,
        content_disposition=None,
        content_encoding=None,
        content_language=None,
        content_type=None,
    ):
    """Generates a shared access signature for a blob.

    This function is a simplified version of the azure.storage.blob.generate_blob_sas
    without supporting parameters: snapshot and some **kwargs
    for simplicity. And permission can only be str for simplicity.

    Use the returned signature with the credential parameter of any BlobServiceClient,
    ContainerClient or BlobClient.

    :param str account_name:
        The storage account name used to generate the shared access signature.
    :param str container_name:
        The name of the container.
    :param str blob_name:
        The name of the blob.
    :param str account_key:
        The account key, also called shared key or access key, to generate the shared access signature.
        Either `account_key` or `user_delegation_key` must be specified.
    :param ~azure.storage.blob.UserDelegationKey user_delegation_key:
        Instead of an account shared key, the user could pass in a user delegation key.
        A user delegation key can be obtained from the service by authenticating with an AAD identity;
        this can be accomplished by calling :func:`~azure.storage.blob.BlobServiceClient.get_user_delegation_key`.
        When present, the SAS is signed with the user delegation key instead.
    :param permission:
        The permissions associated with the shared access signature. The
        user is restricted to operations allowed by the permissions.
        Permissions must be ordered racwdxytmei.
        Required unless an id is given referencing a stored access policy
        which contains this field. This field must be omitted if it has been
        specified in an associated stored access policy.
    :type permission: str
    :param expiry:
        The time at which the shared access signature becomes invalid.
        Required unless an id is given referencing a stored access policy
        which contains this field. This field must be omitted if it has
        been specified in an associated stored access policy. Azure will always
        convert values to UTC. If a date is passed in without timezone info, it
        is assumed to be UTC.
    :type expiry: ~datetime.datetime or str
    :param start:
        The time at which the shared access signature becomes valid. If
        omitted, start time for this call is assumed to be the time when the
        storage service receives the request. Azure will always convert values
        to UTC. If a date is passed in without timezone info, it is assumed to
        be UTC.
    :type start: ~datetime.datetime or str
    :param str policy_id:
        A unique value up to 64 characters in length that correlates to a
        stored access policy. To create a stored access policy, use
        :func:`~azure.storage.blob.ContainerClient.set_container_access_policy`.
    :param str ip:
        Specifies an IP address or a range of IP addresses from which to accept requests.
        If the IP address from which the request originates does not match the IP address
        or address range specified on the SAS token, the request is not authenticated.
        For example, specifying ip=168.1.5.65 or ip=168.1.5.60-168.1.5.70 on the SAS
        restricts the request to those IP addresses.
    :keyword str protocol:
        Specifies the protocol permitted for a request made. The default value is https.
    :keyword str cache_control:
        Response header value for Cache-Control when resource is accessed
        using this shared access signature.
    :keyword str content_disposition:
        Response header value for Content-Disposition when resource is accessed
        using this shared access signature.
    :keyword str content_encoding:
        Response header value for Content-Encoding when resource is accessed
        using this shared access signature.
    :keyword str content_language:
        Response header value for Content-Language when resource is accessed
        using this shared access signature.
    :keyword str content_type:
        Response header value for Content-Type when resource is accessed
        using this shared access signature.
    :return: A Shared Access Signature (sas) token.
    :rtype: str
    """
    if not policy_id:
        if not expiry:
            raise ValueError("'expiry' parameter must be provided when not using a stored access policy.")
        if not permission:
            raise ValueError("'permission' parameter must be provided when not using a stored access policy.")
    if not user_delegation_key and not account_key:
        raise ValueError("Either user_delegation_key or account_key must be provided.")
    if isinstance(account_key, UserDelegationKey):
        user_delegation_key = account_key

    resource_path = container_name + '/' + blob_name

    sas = _BlobSharedAccessHelper()
    sas.add_base(permission, expiry, start, ip, protocol, X_MS_VERSION)
    sas.add_id(policy_id)

    resource = 'b'
    sas.add_resource(resource)
    sas.add_override_response_headers(cache_control, content_disposition,
                                      content_encoding, content_language,
                                      content_type)
    sas.add_resource_signature(account_name, account_key, resource_path, user_delegation_key=user_delegation_key)

    return sas.get_token()


class ClientAuthenticationError(Exception):
    pass


def get_user_delegation_key(
        tenant_id,
        client_id,
        client_secret,
        account_name,
        key_start_time,
        key_expiry_time,
):
    """
    logically equivalent to the following code for azure library
    ```
    credential = ClientSecretCredential(tenant_id, client_id, client_secret)
    service_client = BlobServiceClient(account_url=f"https://{account_name}.blob.core.windows.net", credential=credential)
    delegation_key = service_client.get_user_delegation_key(key_start_time=key_start_time, key_expiry_time=key_expiry_time)
    ```
    """

    # Get OAuth 2.0 access token using client credentials flow
    token_url = f'https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token'
    token_data = {
        'client_id': client_id,
        'client_secret': client_secret,
        'scope': f'https://{account_name}.blob.core.windows.net/.default',  # https://storage.azure.com/.default
        'grant_type': 'client_credentials'
    }
    token_response = requests.post(token_url, data=token_data, timeout=5)
    if token_response.status_code in (401, 403):
        raise ClientAuthenticationError(f"Failed to get access token: {token_response.content}")
    if token_response.status_code != 200:
        raise ValidationError(f"Failed to get access token: {token_response.content}")
    access_token = token_response.json()['access_token']

    # Generate User Delegation Key using Azure Storage Blob Service REST API
    key_data = f"""<?xml version='1.0' encoding='utf-8'?>
    <KeyInfo><Start>{_to_utc_datetime(key_start_time)}</Start><Expiry>{_to_utc_datetime(key_expiry_time)}</Expiry></KeyInfo>"""
    key_request_url = f'https://{account_name}.blob.core.windows.net/?restype=service&comp=userdelegationkey'
    headers = {
        'Authorization': f'Bearer {access_token}',
        'x-ms-version': X_MS_VERSION,
        'Content-Type': 'application/xml'
    }

    try:
        key_response = requests.post(key_request_url, data=key_data, headers=headers, timeout=5)
    except requests.exceptions.ConnectionError:
        raise ValidationError("Failed to get user delegation key: the account name may be incorrect")
    if key_response.status_code in (401, 403):
        raise ClientAuthenticationError(f"Failed to get user delegation key: {key_response.content}")
    if key_response.status_code != 200:
        raise ValidationError(f"Failed to get user delegation key: {key_response.content}")

    # Parse the user delegation key from the response
    key_response_xml = etree.fromstring(key_response.content)
    user_delegation_key = UserDelegationKey()
    user_delegation_key.signed_oid = key_response_xml.findtext('SignedOid')
    user_delegation_key.signed_tid = key_response_xml.findtext('SignedTid')
    user_delegation_key.signed_start = key_response_xml.findtext('SignedStart')
    user_delegation_key.signed_expiry = key_response_xml.findtext('SignedExpiry')
    user_delegation_key.signed_service = key_response_xml.findtext('SignedService')
    user_delegation_key.signed_version = key_response_xml.findtext('SignedVersion')
    user_delegation_key.value = key_response_xml.findtext('Value')

    return user_delegation_key

```

## File: utils\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import cloud_storage_azure_utils

```

## File: views\settings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="cloud_storage_config_settings_view_form" model="ir.ui.view">
        <field name="name">cloud_storage_config_settings_view_form</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="70"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='cloud_storage_provider']" position="inside">
                <div class="content-group mt16" invisible="cloud_storage_provider != 'azure'">
                    <label for="cloud_storage_azure_account_name" class="o_light_label mr8"/>
                    <field name="cloud_storage_azure_account_name"/>
                    <br/>
                    <label for="cloud_storage_azure_container_name" class="o_light_label mr8"/>
                    <field name="cloud_storage_azure_container_name"/>
                    <br/>
                    <label for="cloud_storage_azure_tenant_id" class="o_light_label mr8"/>
                    <field name="cloud_storage_azure_tenant_id"/>
                    <br/>
                    <label for="cloud_storage_azure_client_id" class="o_light_label mr8"/>
                    <field name="cloud_storage_azure_client_id"/>
                    <br/>
                    <label for="cloud_storage_azure_client_secret" class="o_light_label mr8"/>
                    <field name="cloud_storage_azure_client_secret"/>
                    <label for="cloud_storage_azure_invalidate_user_delegation_key" class="o_light_label mr8"/>
                    <field name="cloud_storage_azure_invalidate_user_delegation_key"/>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

