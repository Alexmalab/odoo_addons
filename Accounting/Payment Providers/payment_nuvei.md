# Odoo Module: payment_nuvei

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The currencies supported by Nuvei, in ISO 4217 format.
SUPPORTED_CURRENCIES = [
    'ARS',
    'BRL',
    'CAD',
    'CLP',
    'COP',
    'MXN',
    'PEN',
    'USD',
    'UYU',
]

# The codes of the payment methods to activate when Nuvei is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    # Primary payment methods.
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
    'tarjeta_mercadopago',
    'naranja',
}

# Some payment methods require no decimal points no matter the currency
INTEGER_METHODS = [
    'webpay',
]

# Some payment methods require first and last name on customers to work.
FULL_NAME_METHODS = [
    'boleto',
]

# Mapping of payment method codes to Nuvei codes.
PAYMENT_METHODS_MAPPING = {
    'astropay': 'apmgw_Astropay_TEF',
    'boleto': 'apmgw_BOLETO',
    'card': 'cc_card',
    'nuvei_local': 'apmgw_Local_Payments',
    'oxxopay': 'apmgw_OXXO_PAY',
    'pix': 'apmgw_PIX',
    'pse': 'apmgw_PSE',
    'spei': 'apmgw_SPEI',
    'webpay': 'apmgw_Webpay',
}

# The keys of the values to use in the calculation of the signature.
SIGNATURE_KEYS = [
    'totalAmount',
    'currency',
    'responseTimeStamp',
    'PPP_TransactionID',
    'Status',
    'productId',
]

# Mapping of transaction states to Nuvei payment statuses.
PAYMENT_STATUS_MAPPING = {
    'pending': ('pending',),
    'done': ('approved', 'ok',),
    'error': ('declined', 'error', 'fail',),
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import reset_payment_provider, setup_provider


def post_init_hook(env):
    setup_provider(env, 'nuvei')


def uninstall_hook(env):
    reset_payment_provider(env, 'nuvei')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Nuvei",
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A payment provider covering Latin America.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_nuvei_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools import consteq

from odoo.addons.payment import utils as payment_utils


_logger = logging.getLogger(__name__)


class NuveiController(http.Controller):
    _return_url = '/payment/nuvei/return'
    _webhook_url = '/payment/nuvei/webhook'

    @http.route(_return_url, type='http', auth='public', methods=['GET'])
    def nuvei_return_from_checkout(self, tx_ref=None, error_access_token=None, **data):
        """ Process the notification data sent by Nuvei after redirection.

        :param str tx_ref: The optional reference of the transaction having been canceled/errored.
        :param str error_access_token: The optional access token to verify the authenticity of
                                       requests for errored payments.
        :param dict data: The notification data.
        """
        _logger.info("Handling redirection from Nuvei with data:\n%s", pprint.pformat(data))
        if tx_ref and error_access_token:
            _logger.warning("Nuvei errored on transaction with reference: %s", tx_ref)

        tx_data = data or {'invoice_id': tx_ref}
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'nuvei', tx_data
        )
        self._verify_notification_signature(tx_sudo, data, error_access_token=error_access_token)

        # Handle the notification data if there is any.
        tx_sudo._handle_notification_data('nuvei', data)
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def nuvei_webhook(self, **data):
        """ Process the notification data sent by Nuvei to the webhook.

        See https://docs.nuvei.com/documentation/integration/webhooks/payment-dmns/.

        :param dict data: The notification data.
        :return: The 'OK' string to acknowledge the notification.
        :rtype: str
        """
        _logger.info("Notification received from Nuvei with data:\n%s", pprint.pformat(data))
        try:
            # Check the integrity of the notification.
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'nuvei', data
            )
            self._verify_notification_signature(tx_sudo, data)

            # Handle the notification data.
            tx_sudo._handle_notification_data('nuvei', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed.
            _logger.exception("Unable to handle the notification data; skipping to acknowledge.")

        return 'OK'  # Acknowledge the notification.

    @staticmethod
    def _verify_notification_signature(tx_sudo, notification_data, error_access_token=None):
        """ Check that the received signature matches the expected one.

        :param payment.transaction tx_sudo: The sudoed transaction referenced by the notification
                                            data.
        :param dict notification_data: The notification data.
        :param str error_access_token: The optional access token for verifying errored payments.
        :return: None
        :raise Forbidden: If the signatures don't match.
        """
        if error_access_token:  # The access token is not included when the payment goes through.
            # Verify the request based on the provided access token.
            ref = tx_sudo.reference
            if not payment_utils.check_access_token(error_access_token, ref):
                _logger.warning("Received cancel/error with invalid access token.")
                raise Forbidden()
        else:  # The payment went through.
            received_signature = notification_data.get('advanceResponseChecksum')
            if not received_signature:
                _logger.warning("received notification with missing signature")
                raise Forbidden()

            # Compare the received signature with the expected signature computed from the data.
            expected_signature = tx_sudo.provider_id._nuvei_calculate_signature(
                notification_data, incoming=True,
            )
            if not consteq(received_signature, expected_signature):
                _logger.warning("received notification with invalid signature")
                raise Forbidden()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable nuvei payment provider
UPDATE payment_provider
   SET nuvei_merchant_identifier = NULL,
       nuvei_site_identifier = NULL,
       nuvei_secret_key = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_nuvei" model="payment.provider">
        <field name="code">nuvei</field>
        <field name="redirect_form_view_id" ref="payment_nuvei.redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import logging

from odoo import fields, models

from odoo.addons.payment_nuvei import const


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('nuvei', "Nuvei")], ondelete={'nuvei': 'set default'}
    )
    nuvei_merchant_identifier = fields.Char(
        string="Nuvei Merchant Identifier",
        help="The code of the merchant account to use with this provider.",
        required_if_provider='nuvei',
    )
    nuvei_site_identifier = fields.Char(
        string="Nuvei Site Identifier",
        help="The site identifier code associated with the merchant account.",
        required_if_provider='nuvei',
        groups='base.group_system',
    )
    nuvei_secret_key = fields.Char(
        string="Nuvei Secret Key",
        required_if_provider='nuvei',
        groups='base.group_system',
    )

    # === BUSINESS METHODS === #

    def _nuvei_get_api_url(self):
        if self.state == 'enabled':
            return 'https://secure.safecharge.com/ppp/purchase.do'
        else:  # 'test'
            return 'https://ppp-test.safecharge.com/ppp/purchase.do'

    def _nuvei_calculate_signature(self, data, incoming=True):
        """ Compute the signature for the provided data according to the Nuvei documentation.

        :param dict data: The data to sign.
        :param bool incoming: If the signature must be generated for an incoming (Nuvei to Odoo) or
                              outgoing (Odoo to Nuvei) communication.
        :return: The calculated signature.
        :rtype: str
        """
        self.ensure_one()
        signature_keys = const.SIGNATURE_KEYS if incoming else data.keys()
        sign_data = ''.join([str(data.get(k, '')) for k in signature_keys])
        key = self.nuvei_secret_key
        signing_string = f'{key}{sign_data}'
        return hashlib.sha256(signing_string.encode()).hexdigest()

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'nuvei':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'nuvei':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from urllib.parse import urlencode
from uuid import uuid4

from odoo import _, models
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_round

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_nuvei import const
from odoo.addons.payment_nuvei.controllers.main import NuveiController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of `payment` to return Nuvei-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the
                                       transaction.
        :return: The dict of provider-specific rendering values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'nuvei':
            return res

        first_name, last_name = payment_utils.split_partner_name(self.partner_name)
        if self.payment_method_code in const.FULL_NAME_METHODS and not (first_name and last_name):
            raise UserError(
                "Nuvei: " + _(
                    "%(payment_method)s requires both a first and last name.",
                    payment_method=self.payment_method_id.name,
                )
            )

        # Some payment methods don't support float values, even for currencies that does. Therefore,
        # we must round them.
        is_mandatory_integer_pm = self.payment_method_code in const.INTEGER_METHODS
        rounding = 0 if is_mandatory_integer_pm else self.currency_id.decimal_places
        rounded_amount = float_round(self.amount, rounding, rounding_method='DOWN')

        # Phone numbers need to be standardized and validated.
        phone_number = self.partner_phone and self._phone_format(
            number=self.partner_phone, country=self.partner_country_id, raise_exception=False
        )

        # When a parsing error occurs with Nuvei or the user cancels the order, they do not send the
        # checksum back, as such we need to pass an access token token in the url.
        base_url = self.provider_id.get_base_url()
        return_url = base_url + NuveiController._return_url
        cancel_error_url_params = {
            'tx_ref': self.reference,
            'error_access_token': payment_utils.generate_access_token(self.reference),
        }
        cancel_error_url = f'{return_url}?{urlencode(cancel_error_url_params)}'

        url_params = {
            'address1': self.partner_address or '',
            'city': self.partner_city or '',
            'country': self.partner_country_id.code,
            'currency': self.currency_id.name,
            'email': self.partner_email or '',
            'encoding': 'UTF-8',
            'first_name': first_name,
            'item_amount_1': rounded_amount,
            'item_name_1': self.reference,
            'item_quantity_1': 1,
            'invoice_id': self.reference,
            'last_name': last_name,
            'merchantLocale': self.partner_lang,
            'merchant_id': self.provider_id.nuvei_merchant_identifier,
            'merchant_site_id': self.provider_id.nuvei_site_identifier,
            'payment_method_mode': 'filter',
            'payment_method': const.PAYMENT_METHODS_MAPPING.get(
                self.payment_method_code, self.payment_method_code
            ),
            'phone1': phone_number or '',
            'state': self.partner_state_id.code or '',
            'user_token_id': uuid4(),  # Random string due to some PMs requiring it but not used.
            'time_stamp': self.create_date.strftime('%Y-%m-%d.%H:%M:%S'),
            'total_amount': rounded_amount,
            'version': '4.0.0',
            'zip': self.partner_zip or '',
            'back_url': cancel_error_url,
            'error_url': cancel_error_url,
            'notify_url': base_url + NuveiController._webhook_url,
            'pending_url': return_url,
            'success_url': return_url,
        }

        checksum = self.provider_id._nuvei_calculate_signature(url_params, incoming=False)
        rendering_values = {
            'api_url': self.provider_id._nuvei_get_api_url(),
            'checksum': checksum,
            'url_params': url_params,
        }
        return rendering_values

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on Nuvei data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: payment.transaction
        :raise ValidationError: If inconsistent data are received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'nuvei' or len(tx) == 1:
            return tx

        reference = notification_data.get('invoice_id')
        if not reference:
            raise ValidationError(
                "Nuvei: " + _("Received data with missing reference.")
            )

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'nuvei')])
        if not tx:
            raise ValidationError(
                "Nuvei: " + _("No transaction found matching reference %(ref)s.", ref=reference)
            )

        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment` to process the transaction based on Nuvei data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data are received.
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'nuvei':
            return

        if not notification_data:
            self._set_canceled(state_message=_("The customer left the payment page."))
            return

        # Update the provider reference.
        self.provider_reference = notification_data.get('TransactionID')

        # Update the payment method.
        payment_option = notification_data.get('payment_method', '')
        payment_method = self.env['payment.method']._get_from_code(
            payment_option.lower(), mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        status = notification_data.get('Status') or notification_data.get('ppp_status')
        if not status:
            raise ValidationError("Nuvei: " + _("Received data with missing payment state."))
        status = status.lower()
        if status in const.PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending()
        elif status in const.PAYMENT_STATUS_MAPPING['done']:
            self._set_done()
        elif status in const.PAYMENT_STATUS_MAPPING['error']:
            failure_reason = notification_data.get('Reason') or notification_data.get('message')
            self._set_error(_(
                "An error occurred during the processing of your payment (%(reason)s). Please try"
                " again.", reason=failure_reason,
            ))
        else:  # Classify unsupported payment states as the `error` tx state.
            status_description = notification_data.get('Reason')
            _logger.info(
                "Received data with invalid payment status (%(status)s) and reason '%(reason)s' "
                "for transaction with reference %(ref)s",
                {'status': status, 'reason': status_description, 'ref': self.reference},
            )
            self._set_error("Nuvei: " + _(
                "Received invalid transaction status %(status)s and reason '%(reason)s'.",
                status=status, reason=status_description
            ))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M 43.105 4 L 44.077 4 L 44.077 0.818 L 45.185 0.818 L 45.185 0 L 42 0 L 42 0.818 L 43.105 0.818 L 43.105 4 Z M 45.88 4 L 46.738 4 L 46.738 1.453 L 46.791 1.453 L 47.663 4 L 48.218 4 L 49.089 1.453 L 49.145 1.453 L 49.145 4 L 50 4 L 50 0 L 48.892 0 L 47.968 2.714 L 47.918 2.714 L 46.99 0 L 45.88 0 L 45.88 4 Z" fill="#D1D5DB" id="object-1"/><g clip-path="url(#clip0_4771_14794)" transform="matrix(0.192255, 0, 0, 0.192255, -29.263975, 1.410311)" style="" id="object-0"><path d="M194.3 131.3C194.3 124.6 190.8 122.2 185.3 122.2C180.1 122.2 176.6 125 174.6 127.6V163.2H157V108H174.6V112.8L172.4 116.8C175.7 112.9 184.4 106.6 193.6 106.6C206.1 106.6 211.8 113.9 211.8 124V163H194.3V131.3Z" fill="#081F2C"/><path d="M254.4 154.9C251 158.8 242.8 164.4 233.6 164.4C221.1 164.4 215.5 157.3 215.5 147.2V108H233V140C233 146.6 236.4 148.9 242.1 148.9C247.1 148.9 250.5 146.2 252.6 143.5V108H270.2V163.1H252.6V159L254.4 154.9Z" fill="#081F2C"/><path d="M356.997 106.6C373.297 106.6 385.197 118.6 385.197 137.3V141.1H347.297L345.797 139.1C345.797 139.8 345.897 140.7 346.097 141.5C347.397 146.4 351.997 150.9 359.997 150.9C364.897 150.9 370.397 149 373.497 146.2L380.897 157.1C375.397 162 366.297 164.4 357.897 164.4C340.897 164.4 327.797 153.3 327.797 135.4C327.697 119.5 339.797 106.6 356.997 106.6ZM345.697 129.7H368.397C367.897 125.8 365.097 120.2 356.997 120.2C349.397 120.2 346.397 125.7 345.697 129.7Z" fill="#081F2C"/><path d="M388.703 108H406.303V163.1H388.703V108Z" fill="#081F2C"/><path d="M313.603 108L301.803 140.8L302.203 145.9L288.703 108H273.703V116.9L291.903 163.1H310.603L332.203 108H313.603Z" fill="#081F2C"/><path d="M387.5 91C387.5 85.4 391.9 81 397.5 81C403.1 81 407.5 85.4 407.5 91C407.5 96.6 403.1 101 397.5 101C391.9 101.1 387.5 96.6 387.5 91Z" fill="#E40946"/></g></svg>

```

## File: views\payment_nuvei_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="checksum" t-att-value="checksum"/>
            <t t-foreach="url_params" t-as="param">
                <input type="hidden" t-att-name="param" t-att-value="url_params[param]" />
            </t>
        </form>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Nuvei Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group invisible="code != 'nuvei'">
                    <field
                        name="nuvei_merchant_identifier"
                        string="Merchant Identifier"
                        required="code == 'nuvei' and state != 'disabled'"
                    />
                    <field
                        name="nuvei_site_identifier"
                        string="Site Identifier"
                        required="code == 'nuvei' and state != 'disabled'"
                    />
                    <field
                        name="nuvei_secret_key"
                        string="Secret Key"
                        required="code == 'nuvei' and state != 'disabled'"
                        password="True"
                    />
                </group>
            </group>
        </field>
    </record>

</odoo>

```

