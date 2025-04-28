# Odoo Module: payment_adyen_paybylink

Category: Accounting/Payment

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Endpoints of the API.
# See https://docs.adyen.com/api-explorer/#/CheckoutService/v68/overview for Checkout API
API_ENDPOINT_VERSIONS = {
    '/paymentLinks': 68,          # Checkout API
}

```

## File: utils.py

```python
import re

from odoo import _
from odoo.exceptions import UserError


def format_partner_address(partner):
    """ Format the partner address to comply with the payload structure of the API request.
    :param res.partner partner: The partner making the payment.
    :return: The formatted partner address.
    :rtype: dict
    """
    STREET_FORMAT = '%(street_number)s/%(street_number2)s %(street_name)s'
    street_data = split_street_with_params(partner.street, STREET_FORMAT)
    return {
        'city': partner.city,
        'country': partner.country_id.code or 'ZZ',  # 'ZZ' if the country is not known.
        'stateOrProvince': partner.state_id.code,
        'postalCode': partner.zip,
        'street': street_data['street_name'],
        'houseNumberOrName': street_data['street_number'],
    }


# The method is copy-pasted from `base_address_extended` with small modifications.
def split_street_with_params(street_raw, street_format):
    street_fields = ['street_name', 'street_number', 'street_number2']
    vals = {}
    previous_pos = 0
    field_name = None
    # iter on fields in street_format, detected as '%(<field_name>)s'
    for re_match in re.finditer(r'%\(\w+\)s', street_format):
        field_pos = re_match.start()
        if not field_name:
            #first iteration: remove the heading chars
            street_raw = street_raw[field_pos:]

        # get the substring between 2 fields, to be used as separator
        separator = street_format[previous_pos:field_pos]
        field_value = None
        if separator and field_name:
            #maxsplit set to 1 to unpack only the first element and let the rest untouched
            tmp = street_raw.split(separator, 1)
            if previous_greedy in vals:
                # attach part before space to preceding greedy field
                append_previous, sep, tmp[0] = tmp[0].rpartition(' ')
                street_raw = separator.join(tmp)
                vals[previous_greedy] += sep + append_previous
            if len(tmp) == 2:
                field_value, street_raw = tmp
                vals[field_name] = field_value
        if field_value or not field_name:
            previous_greedy = None
            if field_name == 'street_name' and separator == ' ':
                previous_greedy = field_name
            # select next field to find (first pass OR field found)
            # [2:-2] is used to remove the extra chars '%(' and ')s'
            field_name = re_match.group()[2:-2]
        else:
            # value not found: keep looking for the same field
            pass
        if field_name not in street_fields:
            raise UserError(_("Unrecognized field %s in street format.", field_name))
        previous_pos = re_match.end()

    # last field value is what remains in street_raw minus trailing chars in street_format
    trailing_chars = street_format[previous_pos:]
    if trailing_chars and street_raw.endswith(trailing_chars):
        vals[field_name] = street_raw[:-len(trailing_chars)]
    else:
        vals[field_name] = street_raw
    return vals

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

from . import models
from . import controllers

def _post_init_hook(cr, registry):
    """ Disable the acquirer because new mandatory fields are added in this module. """
    env = api.Environment(cr, SUPERUSER_ID, {})
    acquirers = env['payment.acquirer'].search([
        ('provider', '=', 'adyen'),
        ('state', '!=', 'disabled'),
    ])
    acquirers.write({
        'state': 'disabled',
    })

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Adyen Payment Acquirer/Pay by Link Patch",
    'category': 'Accounting/Payment',
    'summary': "Payment Acquirer: Adyen Pay by Link Patch",
    'version': '1.0',
    'description': """
This module migrates the Adyen implementation from the Hosted Payment Pages API to the Pay by Link
API.
    """,
    'depends': ['payment_adyen'],
    'data': [
        'views/payment_views.xml',
    ],
    'post_init_hook': '_post_init_hook',
    'auto_install': True,
    'license': 'LGPL-3'
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import binascii
import hashlib
import hmac
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo.exceptions import ValidationError
from odoo.http import request, route

from odoo.addons.payment_adyen.controllers.main import AdyenController

_logger = logging.getLogger(__name__)


class AdyenPayByLinkController(AdyenController):

    @route()
    def adyen_notification(self, **post):
        """ Process the data sent by Adyen to the webhook based on the event code.

        See https://docs.adyen.com/development-resources/webhooks/understand-notifications for the
        exhaustive list of event codes.

        :return: The '[accepted]' string to acknowledge the notification
        :rtype: str
        """
        _logger.info(
            "notification received from Adyen with data:\n%s", pprint.pformat(post)
        )
        try:
            # Check the integrity of the notification
            tx_sudo = request.env['payment.transaction'].sudo()._adyen_form_get_tx_from_data(post)
            self._verify_notification_signature(post, tx_sudo)

            # Check whether the event of the notification succeeded and reshape the notification
            # data for parsing
            event_code = post['eventCode']
            if event_code == 'AUTHORISATION' and post['success'] == 'true':
                post['authResult'] = 'AUTHORISED'

                # Handle the notification data
                request.env['payment.transaction'].sudo().form_feedback(post, 'adyen')
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.exception("unable to handle the notification data; skipping to acknowledge")

        return '[accepted]'  # Acknowledge the notification

    @staticmethod
    def _verify_notification_signature(notification_data, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict notification_data: The notification payload containing the received signature
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the signatures don't match
        """
        # Retrieve the received signature from the payload
        received_signature = notification_data.get('additionalData.hmacSignature')
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the payload
        hmac_key = tx_sudo.acquirer_id.adyen_hmac_key
        expected_signature = AdyenPayByLinkController._compute_signature(
            notification_data, hmac_key
        )
        if not hmac.compare_digest(received_signature, expected_signature):
            _logger.warning("received notification with invalid signature")
            raise Forbidden()

    @staticmethod
    def _compute_signature(payload, hmac_key):
        """ Compute the signature from the payload.

        See https://docs.adyen.com/development-resources/webhooks/verify-hmac-signatures

        :param dict payload: The notification payload
        :param str hmac_key: The HMAC key of the acquirer handling the transaction
        :return: The computed signature
        :rtype: str
        """
        def _flatten_dict(_value, _path_base='', _separator='.'):
            """ Recursively generate a flat representation of a dict.

            :param Object _value: The value to flatten. A dict or an already flat value
            :param str _path_base: They base path for keys of _value, including preceding separators
            :param str _separator: The string to use as a separator in the key path
            """
            if isinstance(_value, dict):  # The inner value is a dict, flatten it
                _path_base = _path_base if not _path_base else _path_base + _separator
                for _key in _value:
                    yield from _flatten_dict(_value[_key], _path_base + str(_key))
            else:  # The inner value cannot be flattened, yield it
                yield _path_base, _value

        def _to_escaped_string(_value):
            """ Escape payload values that are using illegal symbols and cast them to string.

            String values containing `\\` or `:` are prefixed with `\\`.
            Empty values (`None`) are replaced by an empty string.

            :param Object _value: The value to escape
            :return: The escaped value
            :rtype: string
            """
            if isinstance(_value, str):
                return _value.replace('\\', '\\\\').replace(':', '\\:')
            elif _value is None:
                return ''
            else:
                return str(_value)

        signature_keys = [
            'pspReference', 'originalReference', 'merchantAccountCode', 'merchantReference',
            'value', 'currency', 'eventCode', 'success'
        ]
        # Build the list of signature values as per the list of required signature keys
        signature_values = [payload.get(key) for key in signature_keys]
        # Escape values using forbidden symbols
        escaped_values = [_to_escaped_string(value) for value in signature_values]
        # Concatenate values together with ':' as delimiter
        signing_string = ':'.join(escaped_values)
        # Convert the HMAC key to the binary representation
        binary_hmac_key = binascii.a2b_hex(hmac_key.encode('ascii'))
        # Calculate the HMAC with the binary representation of the signing string with SHA-256
        binary_hmac = hmac.new(binary_hmac_key, signing_string.encode('utf-8'), hashlib.sha256)
        # Calculate the signature by encoding the result with Base64
        return base64.b64encode(binary_hmac.digest()).decode()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import re
import requests

from werkzeug import urls

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_adyen_paybylink import utils as adyen_utils
from odoo.addons.payment_adyen_paybylink.const import API_ENDPOINT_VERSIONS


_logger = logging.getLogger(__name__)


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    adyen_api_key = fields.Char(
        string="API Key",
        help="The API key of the webservice user",
        required_if_provider='adyen',
        groups='base.group_user',
    )
    adyen_hmac_key = fields.Char(
        string="HMAC Key",
        help="The HMAC key of the webhook",
        required_if_provider='adyen',
        groups='base.group_user',
    )
    adyen_checkout_api_url = fields.Char(
        string="Checkout API URL",
        help="The base URL for the Checkout API endpoints",
        required_if_provider='adyen',
    )
    # We set a default for the now unused key fields rather than making them not required to avoid
    # the error log at DB init when the ORM tries to set the 'NOT NULL' constraint on those fields.
    adyen_skin_code = fields.Char(default="Do not use this field")
    adyen_skin_hmac_key = fields.Char(default="Do not use this field")

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            self._adyen_trim_api_urls(values)
        return super().create(values_list)

    def write(self, values):
        self._adyen_trim_api_urls(values)
        # We set a default for the now unused key fields rather than making them not required to
        # avoid the error log at DB init when the ORM tries to set the 'NOT NULL' constraint on
        # those fields.
        values.update(
            adyen_skin_code="Do not use this field",
            adyen_skin_hmac_key="Do not use this field",
        )
        return super().write(values)

    @api.model
    def _adyen_trim_api_urls(self, values):
        """ Remove the version and the endpoint from the url of Adyen API fields.

        :param dict values: The create or write values
        :return: None
        """
        # Test the value in case we're duplicating an acquirer
        if values.get('adyen_checkout_api_url'):
            values['adyen_checkout_api_url'] = re.sub(
                r'[vV]\d+(/.*)?', '', values['adyen_checkout_api_url']
            )

    def adyen_form_generate_values(self, values):
        base_url = self.get_base_url()

        payment_amount = self._adyen_convert_amount(values['amount'], values['currency'])
        values['adyen_paybylink_data'] = {
            'reference': values['reference'],
            'amount': {
                'value': '%d' % payment_amount,
                'currency': values['currency'] and values['currency'].name or '',
            },
            'merchantAccount': self.adyen_merchant_account,
            'shopperLocale': values.get('partner_lang', ''),
            'returnUrl': urls.url_join(base_url, '/payment/process'),
            'shopperEmail': values.get('partner_email') or values.get('billing_partner_email', ''),
            'shopperReference': self._adyen_compute_shopper_reference(values.get('partner_id')),
            'shopperName': {
                'firstName': values.get('partner_first_name'),
                'lastName': values.get('partner_last_name'),
            },
            'telephoneNumber': values.get('partner_phone'),
            'billingAddress': adyen_utils.format_partner_address(values.get('billing_partner')),
            'deliveryAddress': adyen_utils.format_partner_address(values.get('partner')),
        }

        return values

    def adyen_get_form_action_url(self):
        """ Override of adyen_get_form_action_url """
        form_action_url_values = self._context.get('form_action_url_values')
        if form_action_url_values:
            return self._adyen_get_paybylink(form_action_url_values['adyen_paybylink_data'])
        return False

    def _adyen_get_paybylink(self, data):
        paybylink_response = self._adyen_make_request(
            url_field_name='adyen_checkout_api_url',
            endpoint='/paymentLinks',
            payload=data,
        )
        return paybylink_response['url']

    def _adyen_make_request(
        self, url_field_name, endpoint, endpoint_param=None, payload=None, method='POST'
    ):
        """ Make a request to Adyen API at the specified endpoint.

        Note: self.ensure_one()

        :param str url_field_name: The name of the field holding the base URL for the request
        :param str endpoint: The endpoint to be reached by the request
        :param str endpoint_param: A variable required by some endpoints which are interpolated with
                                   it if provided. For example, the acquirer reference of the source
                                   transaction for the '/payments/{}/refunds' endpoint.
        :param dict payload: The payload of the request
        :param str method: The HTTP method of the request
        :return: The JSON-formatted content of the response
        :rtype: dict
        :raise: ValidationError if an HTTP error occurs
        """

        def _build_url(_base_url, _version, _endpoint):
            """ Build an API URL by appending the version and endpoint to a base URL.

            The final URL follows this pattern: `<_base>/V<_version>/<_endpoint>`.

            :param str _base_url: The base of the url prefixed with `https://`
            :param int _version: The version of the endpoint
            :param str _endpoint: The endpoint of the URL.
            :return: The final URL
            :rtype: str
            """
            _base = _base_url.rstrip('/')  # Remove potential trailing slash
            _endpoint = _endpoint.lstrip('/')  # Remove potential leading slash
            return f'{_base}/V{_version}/{_endpoint}'

        self.ensure_one()

        base_url = self[url_field_name]  # Restrict request URL to the stored API URL fields
        version = API_ENDPOINT_VERSIONS[endpoint]
        endpoint = endpoint if not endpoint_param else endpoint.format(endpoint_param)
        url = _build_url(base_url, version, endpoint)
        headers = {'X-API-Key': self.adyen_api_key}
        try:
            response = requests.request(method, url, json=payload, headers=headers, timeout=60)
            response.raise_for_status()
        except requests.exceptions.ConnectionError:
            _logger.exception("unable to reach endpoint at %s", url)
            raise ValidationError("Adyen: " + _("Could not establish the connection to the API."))
        except requests.exceptions.HTTPError as error:
            _logger.exception(
                "invalid API request at %s with data %s: %s", url, payload, error.response.text
            )
            raise ValidationError("Adyen: " + _("The communication with the API failed."))
        return response.json()

    def _adyen_compute_shopper_reference(self, partner_id):
        """ Compute a unique reference of the partner for Adyen.

        This is used for the `shopperReference` field in communications with Adyen and stored in the
        `adyen_shopper_reference` field on `payment.token` if the payment method is tokenized.

        :param recordset partner_id: The partner making the transaction, as a `res.partner` id
        :return: The unique reference for the partner
        :rtype: str
        """
        return 'ODOO_PARTNER_{partner_id}'.format(partner_id=partner_id)

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def adyen_create(self, values):
        """
        When the customer lands on the `/payment/process` route, `/payment/process/poll` try to find
        the transaction whose `date` field is between yesterday and now.

        Since the `date` field is only set when the state of the transaction is changed, if the
        customer comes back before the webhook, he will see a "transaction not found" page because
        the value of the `date` field would be `False`.
        """
        return dict(date=fields.Datetime.now())

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    @api.model
    def _adyen_form_get_tx_from_data(self, data):
        """ Override of _adyen_form_get_tx_from_data """
        reference, psp_reference = data.get('merchantReference'), data.get('pspReference')
        if not reference or not psp_reference:
            error_msg = _(
                "Adyen: received data with missing reference (%s) or missing pspReference (%s)"
            ) % (reference, psp_reference)
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        tx = self.env['payment.transaction'].search([
            ('reference', '=', reference), ('provider', '=', 'adyen')
        ])
        if not tx or len(tx) > 1:
            error_msg = _("Adyen: received data for reference %s") % reference
            if not tx:
                error_msg += _("; no order found")
            else:
                error_msg += _("; multiple order found")
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        return tx

    def _adyen_form_get_invalid_parameters(self, data):
        """ Override of _adyen_form_get_invalid_parameters to disable this method.

        The pay-by-link implementation doesn't need or want to check for invalid parameters.
        """
        return []

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_acquirer
from . import payment_transaction

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="acquirer_form_adyen" model="ir.ui.view">
            <field name="name">acquirer.form.adyen</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr='//field[@name="adyen_merchant_account"]' position='after'>
                    <field name="adyen_api_key"
                           attrs="{'required':[('provider', '=', 'adyen'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                    <field name="adyen_hmac_key"
                           attrs="{'required':[('provider', '=', 'adyen'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                    <field name="adyen_checkout_api_url"
                           attrs="{'required':[('provider', '=', 'adyen'), ('state', '!=', 'disabled')]}"/>
                </xpath>
                <xpath expr="//field[@name='adyen_skin_code']" position='replace'></xpath>
                <xpath expr="//field[@name='adyen_skin_hmac_key']" position='replace'></xpath>
            </field>
        </record>

    </data>
</odoo>

```

