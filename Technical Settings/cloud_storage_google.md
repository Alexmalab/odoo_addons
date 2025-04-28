# Odoo Module: cloud_storage_google

Category: Technical Settings

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


def uninstall_hook(env):
    ICP = env['ir.config_parameter']
    if ICP.get_param('cloud_storage_provider') == 'google':
        env['res.config.settings']._check_cloud_storage_uninstallable()
        ICP.set_param('cloud_storage_provider', False)
    ICP.search([('key', 'in', [
        'cloud_storage_google_bucket_name',
        'cloud_storage_google_account_info',
    ])]).unlink()

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": "Cloud Storage Google",
    "summary": """Store chatter attachments in the Google cloud""",
    "category": "Technical Settings",
    "version": "1.0",
    "depends": ["cloud_storage"],
    "data": [
        "views/settings.xml",
    ],
    'assets': {
        'web.assets_backend': [
            'cloud_storage_google/static/src/**/*',
        ],
    },
    "uninstall_hook": "uninstall_hook",
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
DELETE
FROM ir_config_parameter
WHERE key = 'cloud_storage_google_bucket_name'
OR key = 'cloud_storage_google_account_info';

```

## File: models\ir_attachment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import re
from urllib.parse import unquote, quote

try:
    from google.oauth2 import service_account
    from google.auth.transport.requests import Request
except ImportError:
    service_account = Request = None

from odoo import models
from odoo.exceptions import ValidationError

from ..utils.cloud_storage_google_utils import generate_signed_url_v4

CloudStorageGoogleCredentials = {}  # {db_name: (account_info, credential)}


def get_cloud_storage_google_credential(env):
    """ Get the credentials object of currently used account info.
    This method is cached to because from_service_account_info is slow.
    """
    cached_account_info, cached_credential = CloudStorageGoogleCredentials.get(env.registry.db_name, (None, None))
    account_info = json.loads(env['ir.config_parameter'].sudo().get_param('cloud_storage_google_account_info'))
    if cached_account_info == account_info:
        return cached_credential
    credential = service_account.Credentials.from_service_account_info(account_info)
    CloudStorageGoogleCredentials[env.registry.db_name] = (account_info, credential)
    return credential


class IrAttachment(models.Model):
    _inherit = 'ir.attachment'
    _cloud_storage_google_url_pattern = re.compile(r'https://storage\.googleapis\.com/(?P<bucket_name>[\w\-.]+)/(?P<blob_name>[^?]+)')

    def _get_cloud_storage_google_info(self):
        match = self._cloud_storage_google_url_pattern.match(self.url)
        if not match:
            raise ValidationError('%s is not a valid Google Cloud Storage URL.', self.url)
        return {
            'bucket_name': match['bucket_name'],
            'blob_name': unquote(match['blob_name']),
        }

    def _generate_cloud_storage_google_url(self, blob_name):
        bucket_name = self.env['ir.config_parameter'].get_param('cloud_storage_google_bucket_name')
        return f"https://storage.googleapis.com/{bucket_name}/{quote(blob_name)}"

    def _generate_cloud_storage_google_signed_url(self, bucket_name, blob_name, **kwargs):
        quote_blob_name = quote(blob_name)
        return generate_signed_url_v4(
            credentials=get_cloud_storage_google_credential(self.env),
            resource=f'/{bucket_name}/{quote_blob_name}',
            **kwargs,
        )

    # OVERRIDES
    def _generate_cloud_storage_url(self):
        if self.env['ir.config_parameter'].sudo().get_param('cloud_storage_provider') != 'google':
            return super()._generate_cloud_storage_url()
        blob_name = self._generate_cloud_storage_blob_name()
        return self._generate_cloud_storage_google_url(blob_name)

    def _generate_cloud_storage_download_info(self):
        if self.env['ir.config_parameter'].sudo().get_param('cloud_storage_provider') != 'google':
            return super()._generate_cloud_storage_download_info()
        info = self._get_cloud_storage_google_info()
        return {
            'url': self._generate_cloud_storage_google_signed_url(info['bucket_name'], info['blob_name'], method='GET', expiration=self._cloud_storage_download_url_time_to_expiry),
            'time_to_expiry': self._cloud_storage_download_url_time_to_expiry,
        }

    def _generate_cloud_storage_upload_info(self):
        if self.env['ir.config_parameter'].sudo().get_param('cloud_storage_provider') != 'google':
            return super()._generate_cloud_storage_upload_info()
        info = self._get_cloud_storage_google_info()
        return {
            'url': self._generate_cloud_storage_google_signed_url(info['bucket_name'], info['blob_name'], method='PUT', expiration=self._cloud_storage_upload_url_time_to_expiry),
            'method': 'PUT',
            'response_status': 200,
        }

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import requests
from datetime import datetime, timezone

try:
    from google.oauth2 import service_account
    from google.auth.transport.requests import Request
except ImportError:
    service_account = Request = None

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError, UserError

from .ir_attachment import get_cloud_storage_google_credential


class CloudStorageSettings(models.TransientModel):
    """
    Instructions:
    cloud_storage_google_bucket_name: if changed and the old bucket name
        are still in use, you should promise the current service account
        has the permission to access the old bucket.
    """
    _inherit = 'res.config.settings'

    cloud_storage_provider = fields.Selection(selection_add=[('google', 'Google Cloud Storage')])

    cloud_storage_google_bucket_name = fields.Char(
        string='Google Bucket Name',
        config_parameter='cloud_storage_google_bucket_name')
    # Google Service Account Key in JSON format
    cloud_storage_google_service_account_key = fields.Binary(
        string='Google Service Account Key', store=False
    )
    cloud_storage_google_account_info = fields.Char(
        string='Google Service Account Info',
        compute='_compute_cloud_storage_google_account_info',
        store=True,
        readonly=False,
        config_parameter='cloud_storage_google_account_info',
    )

    def get_values(self):
        res = super().get_values()
        if account_info := self.env['ir.config_parameter'].get_param('cloud_storage_google_account_info'):
            res['cloud_storage_google_service_account_key'] = base64.b64encode(account_info.encode())
        return res

    @api.onchange('cloud_storage_google_service_account_key')
    def _compute_cloud_storage_google_account_info(self):
        for setting in self:
            key = setting.with_context(bin_size=False).cloud_storage_google_service_account_key
            setting.cloud_storage_google_account_info = base64.b64decode(key) if key else False

    def _setup_cloud_storage_provider(self):
        ICP = self.env['ir.config_parameter']
        if ICP.get_param('cloud_storage_provider') != 'google':
            return super()._setup_cloud_storage_provider()
        # check bucket access
        bucket_name = ICP.get_param('cloud_storage_google_bucket_name')
        # use different blob names in case the credentials are allowed to
        # overwrite an existing blob created by previous tests
        blob_name = f'0/{datetime.now(timezone.utc)}.txt'

        IrAttachment = self.env['ir.attachment']
        # check blob create permission
        upload_url = IrAttachment._generate_cloud_storage_google_signed_url(bucket_name, blob_name, method='PUT', expiration=IrAttachment._cloud_storage_upload_url_time_to_expiry)
        upload_response = requests.put(upload_url, data=b'', timeout=5)
        if upload_response.status_code != 200:
            raise ValidationError(_('The account info is not allowed to upload blobs to the bucket.\n%s', str(upload_response.text)))

        # check blob read permission
        download_url = IrAttachment._generate_cloud_storage_google_signed_url(bucket_name, blob_name, method='GET', expiration=IrAttachment._cloud_storage_download_url_time_to_expiry)
        download_response = requests.get(download_url, timeout=5)
        if download_response.status_code != 200:
            raise ValidationError(_('The account info is not allowed to download blobs from the bucket.\n%s', str(upload_response.text)))

        # CORS management is not allowed in the Google Cloud console.
        # configure CORS on bucket to allow .pdf preview and direct upload
        cors = [{
            'origin': ['*'],
            'method': ['GET', 'PUT'],
            'responseHeader': ['Content-Type'],
            'maxAgeSeconds': IrAttachment._cloud_storage_download_url_time_to_expiry,
        }]
        credential = get_cloud_storage_google_credential(self.env).with_scopes(['https://www.googleapis.com/auth/devstorage.full_control'])
        credential.refresh(Request())
        url = f"https://storage.googleapis.com/storage/v1/b/{bucket_name}?fields=cors"
        headers = {
            'Authorization': f'Bearer {credential.token}',
            'Content-Type': 'application/json'
        }
        data = json.dumps({'cors': cors})
        patch_response = requests.patch(url, data=data, headers=headers, timeout=5)
        if patch_response.status_code != 200:
            raise ValidationError(_("The account info is not allowed to set the bucket's CORS.\n%s", str(patch_response.text)))

    def _get_cloud_storage_configuration(self):
        ICP = self.env['ir.config_parameter'].sudo()
        if ICP.get_param('cloud_storage_provider') != 'google':
            return super()._get_cloud_storage_configuration()
        configuration = {
            'bucket_name': ICP.get_param('cloud_storage_google_bucket_name'),
            'account_info': ICP.get_param('cloud_storage_google_account_info'),
        }
        return configuration if all(configuration.values()) else {}

    def _check_cloud_storage_uninstallable(self):
        if self.env['ir.config_parameter'].get_param('cloud_storage_provider') != 'google':
            return super()._check_cloud_storage_uninstallable()
        cr = self.env.cr
        cr.execute(
            """
                SELECT type
                FROM ir_attachment
                WHERE type = 'cloud_storage'
                AND url LIKE 'https://storage.googleapis.com/%'
                LIMIT 1
            """
        )
        if cr.fetchone():
            raise UserError(_('Some Google attachments are in use, please migrate cloud storages before disable the provider'))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_attachment
from . import res_config_settings

```

## File: utils\cleanup_cloud_storage_google.py

```python
import json
import logging
import requests
import xmlrpc.client

from concurrent.futures import ThreadPoolExecutor
from google.oauth2 import service_account
from google.auth.transport.requests import Request
from itertools import islice
from urllib.parse import quote

# This is a script to manually clean up unused google blobs.

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
# +--------+ 6. 204: No Content                            +---------+
#
#
# 1, 2, 3, 4 are done in batch
# 5, 6 are done with threadpool

# Odoo
odoo_url = 'http://localhost:8069'
odoo_db = 'odoo_db'
odoo_username = 'admin'
odoo_password = 'admin'

# Google service account
GOOGLE_CLOUD_STORAGE_ENDPOINT = 'https://storage.googleapis.com'
google_cloud_bucket_name = 'bucket_name'
google_cloud_account_info = r"""account_info"""
google_cloud_account_info = json.loads(google_cloud_account_info)

# Get Google credentials
credentials = service_account.Credentials.from_service_account_info(google_cloud_account_info).with_scopes(
    ['https://www.googleapis.com/auth/devstorage.full_control'])
credentials.refresh(Request())

_logger = logging.getLogger(__name__)


def list_blob_urls(bucket_name, batch_size=1000):
    cloud_storage_blobs_num = 0
    url = f"https://www.googleapis.com/storage/v1/b/{bucket_name}/o"
    params = {
        'maxResults': batch_size,
        'fields': 'items(name), nextPageToken',
    }
    headers = {
        'Authorization': f"Bearer {credentials.token}"
    }

    while True:
        response = requests.get(url, params=params, headers=headers, timeout=5)
        data = response.json()
        for blob in data.get('items', []):
            cloud_storage_blobs_num += 1
            yield f'{GOOGLE_CLOUD_STORAGE_ENDPOINT}/{bucket_name}/{quote(blob["name"])}'

        params['pageToken'] = data.get('nextPageToken')
        if params['pageToken'] is None:
            break

    _logger.info('The cloud storage container has %d blobs.', cloud_storage_blobs_num)


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


def delete_blobs(blob_urls, max_workers=None):
    headers = {
        'Authorization': f"Bearer {credentials.token}"
    }
    deleted_cloud_storage_blobs_num = 0

    def delete_blob_(blob_url):
        nonlocal deleted_cloud_storage_blobs_num
        delete_response = requests.delete(blob_url, headers=headers, timeout=5)
        if delete_response.status_code == 204:
            deleted_cloud_storage_blobs_num += 1
            _logger.info('%s is deleted', blob_url)
        elif delete_response.status_code == 404:
            _logger.debug('%s has been deleted', blob_url)
        else:
            _logger.warning('%s cannot be deleted:\n%s', blob_url, delete_response.text)

    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        executor.map(delete_blob_, blob_urls)

    _logger.info(' %d blobs are deleted by the script', deleted_cloud_storage_blobs_num)


if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    all_blob_urls = list_blob_urls(bucket_name=google_cloud_bucket_name, batch_size=1000)
    to_delete_blob_urls = get_blobs_to_be_deleted(all_blob_urls, batch_size=1000)
    delete_blobs(to_delete_blob_urls)

```

## File: utils\cloud_storage_google_utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
"""
This module is used to provide a google.cloud.storage._signing.generate_signed_url_v4
compatible function for generating signed URLs for Google Cloud Storage without
importing the google-cloud-storage library.
"""

import binascii
import collections
import datetime
import hashlib
import urllib

try:
    from google.oauth2 import service_account
except ImportError:
    service_account = None

DEFAULT_ENDPOINT = "https://storage.googleapis.com"
SEVEN_DAYS = 7 * 24 * 60 * 60  # max age for V4 signed URLs.
_EXPIRATION_TYPES = (int, datetime.datetime, datetime.timedelta)


def get_expiration_seconds_v4(expiration):
    """Convert 'expiration' to a number of seconds offset from the current time.

    :type expiration: Union[Integer, datetime.datetime, datetime.timedelta]
    :param expiration: Point in time when the signed URL should expire. If
                       a ``datetime`` instance is passed without an explicit
                       ``tzinfo`` set,  it will be assumed to be ``UTC``.

    :raises: :exc:`TypeError` when expiration is not a valid type.
    :raises: :exc:`ValueError` when expiration is too large.
    :rtype: Integer
    :returns: seconds in the future when the signed URL will expire
    """
    if not isinstance(expiration, _EXPIRATION_TYPES):
        raise TypeError(
            "Expected an integer timestamp, datetime, or "
            "timedelta. Got %s" % type(expiration)
        )

    now = datetime.datetime.now(datetime.timezone.utc)

    if isinstance(expiration, int):
        seconds = expiration

    if isinstance(expiration, datetime.datetime):
        if expiration.tzinfo is None:
            expiration = expiration.replace(tzinfo=datetime.timezone.utc)

        expiration = expiration - now

    if isinstance(expiration, datetime.timedelta):
        seconds = int(expiration.total_seconds())

    if seconds > SEVEN_DAYS:
        raise ValueError(f"Max allowed expiration interval is seven days {SEVEN_DAYS}")

    return seconds


def get_v4_now_dtstamps():
    """Get current timestamp and datestamp in V4 valid format.

    :rtype: str, str
    :returns: Current timestamp, datestamp.
    """
    now = datetime.datetime.now(datetime.timezone.utc)
    timestamp = now.strftime("%Y%m%dT%H%M%SZ")
    datestamp = now.date().strftime("%Y%m%d")
    return timestamp, datestamp


def get_canonical_headers(headers):
    """Canonicalize headers for signing.

    See:
    https://cloud.google.com/storage/docs/access-control/signed-urls#about-canonical-extension-headers

    :type headers: Union[dict|List(Tuple(str,str))]
    :param headers:
        (Optional) Additional HTTP headers to be included as part of the
        signed URLs.  See:
        https://cloud.google.com/storage/docs/xml-api/reference-headers
        Requests using the signed URL *must* pass the specified header
        (name and value) with each request for the URL.

    :rtype: str
    :returns: List of headers, normalized / sortted per the URL refernced above.
    """
    if headers is None:
        headers = []
    elif isinstance(headers, dict):
        headers = list(headers.items())

    if not headers:
        return [], []

    normalized = collections.defaultdict(list)
    for key, val in headers:
        key = key.lower().strip()
        val = " ".join(val.split())
        normalized[key].append(val)

    ordered_headers = sorted((key, ",".join(val)) for key, val in normalized.items())

    canonical_headers = [f"{key}:{val}" for key, val in ordered_headers]
    return canonical_headers, ordered_headers


def generate_signed_url_v4(
        credentials,
        resource,
        expiration,
        api_access_endpoint=DEFAULT_ENDPOINT,
        method="GET",
        content_md5=None,
        content_type=None,
        response_type=None,
        response_disposition=None,
        generation=None,
        headers=None,
        query_parameters=None,
):
    """Generate a V4 signed URL to provide query-string auth'n to a resource.

    This function is a simplified version of the google.cloud.storage._signing.generate_signed_url_v4
    without supporting parameters: service_account_email, access_token and _request_timestamp

    See headers [reference](https://cloud.google.com/storage/docs/reference-headers)
    for more details on optional arguments.

    :type credentials: :class:`google.auth.credentials.Signing`
    :param credentials: Credentials object with an associated private key to
                        sign text. That credentials must provide signer_email
                        only if service_account_email and access_token are not
                        passed.

    :type resource: str
    :param resource: A pointer to a specific resource
                     (typically, ``/bucket-name/path/to/blob.txt``).
                     Caller should have already URL-encoded the value.

    :type expiration: Union[Integer, datetime.datetime, datetime.timedelta]
    :param expiration: Point in time when the signed URL should expire. If
                       a ``datetime`` instance is passed without an explicit
                       ``tzinfo`` set,  it will be assumed to be ``UTC``.

    :type api_access_endpoint: str
    :param api_access_endpoint: (Optional) URI base. Defaults to
                                "https://storage.googleapis.com/"

    :type method: str
    :param method: The HTTP verb that will be used when requesting the URL.
                   Defaults to ``'GET'``. If method is ``'RESUMABLE'`` then the
                   signature will additionally contain the `x-goog-resumable`
                   header, and the method changed to POST. See the signed URL
                   docs regarding this flow:
                   https://cloud.google.com/storage/docs/access-control/signed-urls


    :type content_md5: str
    :param content_md5: (Optional) The MD5 hash of the object referenced by
                        ``resource``.

    :type content_type: str
    :param content_type: (Optional) The content type of the object referenced
                         by ``resource``.

    :type response_type: str
    :param response_type: (Optional) Content type of responses to requests for
                          the signed URL. Ignored if content_type is set on
                          object/blob metadata.

    :type response_disposition: str
    :param response_disposition: (Optional) Content disposition of responses to
                                 requests for the signed URL.

    :type generation: str
    :param generation: (Optional) A value that indicates which generation of
                       the resource to fetch.

    :type headers: dict
    :param headers:
        (Optional) Additional HTTP headers to be included as part of the
        signed URLs.  See:
        https://cloud.google.com/storage/docs/xml-api/reference-headers
        Requests using the signed URL *must* pass the specified header
        (name and value) with each request for the URL.

    :type query_parameters: dict
    :param query_parameters:
        (Optional) Additional query parameters to be included as part of the
        signed URLs.  See:
        https://cloud.google.com/storage/docs/xml-api/reference-headers#query

    :raises: :exc:`TypeError` when expiration is not a valid type.
    :raises: :exc:`AttributeError` if credentials is not an instance
            of :class:`google.auth.credentials.Signing`.

    :rtype: str
    :returns: A signed URL you can use to access the resource
              until expiration.
    """
    expiration_seconds = get_expiration_seconds_v4(expiration)

    request_timestamp, datestamp = get_v4_now_dtstamps()

    client_email = credentials.signer_email

    credential_scope = f"{datestamp}/auto/storage/goog4_request"
    credential = f"{client_email}/{credential_scope}"

    if headers is None:
        headers = {}

    if content_type is not None:
        headers["Content-Type"] = content_type

    if content_md5 is not None:
        headers["Content-MD5"] = content_md5

    header_names = [key.lower() for key in headers]
    if "host" not in header_names:
        headers["Host"] = urllib.parse.urlparse(api_access_endpoint).netloc

    if method.upper() == "RESUMABLE":
        method = "POST"
        headers["x-goog-resumable"] = "start"

    canonical_headers, ordered_headers = get_canonical_headers(headers)
    canonical_header_string = (
            "\n".join(canonical_headers) + "\n"
    )  # Yes, Virginia, the extra newline is part of the spec.
    signed_headers = ";".join([key for key, _ in ordered_headers])

    if query_parameters is None:
        query_parameters = {}
    else:
        query_parameters = {key: value or "" for key, value in query_parameters.items()}

    query_parameters["X-Goog-Algorithm"] = "GOOG4-RSA-SHA256"
    query_parameters["X-Goog-Credential"] = credential
    query_parameters["X-Goog-Date"] = request_timestamp
    query_parameters["X-Goog-Expires"] = expiration_seconds
    query_parameters["X-Goog-SignedHeaders"] = signed_headers

    if response_type is not None:
        query_parameters["response-content-type"] = response_type

    if response_disposition is not None:
        query_parameters["response-content-disposition"] = response_disposition

    if generation is not None:
        query_parameters["generation"] = generation

    canonical_query_string = urllib.parse.urlencode(query_parameters, quote_via=urllib.parse.quote)

    lowercased_headers = dict(ordered_headers)

    payload = lowercased_headers.get("x-goog-content-sha256", "UNSIGNED-PAYLOAD")

    canonical_elements = [
        method,
        resource,
        canonical_query_string,
        canonical_header_string,
        signed_headers,
        payload,
    ]
    canonical_request = "\n".join(canonical_elements)

    canonical_request_hash = hashlib.sha256(
        canonical_request.encode("ascii")
    ).hexdigest()

    string_elements = [
        "GOOG4-RSA-SHA256",
        request_timestamp,
        credential_scope,
        canonical_request_hash,
    ]
    string_to_sign = "\n".join(string_elements)

    signature_bytes = credentials.sign_bytes(string_to_sign.encode("ascii"))
    signature = binascii.hexlify(signature_bytes).decode("ascii")

    return f"{api_access_endpoint}{resource}?{canonical_query_string}&X-Goog-Signature={signature}"

```

## File: utils\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import cloud_storage_google_utils

```

## File: views\settings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="cloud_storage_google_config_settings_view_form" model="ir.ui.view">
        <field name="name">cloud_storage_google_config_settings_view_form</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="70"/>
        <field name="inherit_id" ref="cloud_storage.cloud_storage_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='cloud_storage_provider']" position="inside">
                <div class="content-group mt16" invisible="cloud_storage_provider != 'google'">
                    <label for="cloud_storage_google_bucket_name" class="o_light_label mr8"/>
                    <field name="cloud_storage_google_bucket_name"/>
                    <br/>
                    <label for="cloud_storage_google_service_account_key" class="o_light_label mr8"/>
                    <!-- hide the download button because it is a computed value without a stored attachment -->
                    <field name="cloud_storage_google_service_account_key"
                           widget="binary"
                           class="o_field_binary_hide_download"
                           options="{'accepted_file_extensions': '.json'}"/>
                    <field name="cloud_storage_google_account_info" widget="text" class="w-100"/>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

