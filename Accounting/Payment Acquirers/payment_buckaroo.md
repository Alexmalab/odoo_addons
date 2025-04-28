# Odoo Module: payment_buckaroo

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Mapping of transaction states to Buckaroo status codes.
# See https://www.pronamic.nl/wp-content/uploads/2013/04/BPE-3.0-Gateway-HTML.1.02.pdf for the
# exhaustive list of status codes.
STATUS_CODES_MAPPING = {
    'pending': (790, 791, 792, 793),
    'done': (190,),
    'cancel': (890, 891),
    'refused': (690,),
    'error': (490, 491, 492,),
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'buckaroo')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Buckaroo Payment Acquirer',
    'version': '2.0',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 355,
    'summary': 'Payment Acquirer: Buckaroo Implementation',
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_buckaroo_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'application': True,
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class BuckarooController(http.Controller):
    _return_url = '/payment/buckaroo/return'

    @http.route(
        _return_url, type='http', auth='public', methods=['POST'], csrf=False, save_session=False
    )
    def buckaroo_return_from_redirect(self, **data):
        """ Process the data returned by Buckaroo after redirection.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict data: The feedback data
        """
        _logger.info("received notification data:\n%s", pprint.pformat(data))
        request.env['payment.transaction'].sudo()._handle_feedback_data('buckaroo', data)
        return request.redirect('/payment/status')

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_acquirer_buckaroo" model="payment.acquirer">
        <field name="provider">buckaroo</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">False</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">False</field>
    </record>

    <record id="payment_method_buckaroo" model="account.payment.method">
        <field name="name">Buckaroo</field>
        <field name="code">buckaroo</field>
        <field name="payment_type">inbound</field>
    </record>

</odoo>

```

## File: models\account_payment_method.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountPaymentMethod(models.Model):
    _inherit = 'account.payment.method'

    @api.model
    def _get_payment_method_information(self):
        res = super()._get_payment_method_information()
        res['buckaroo'] = {'mode': 'electronic', 'domain': [('type', '=', 'bank')]}
        return res

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from hashlib import sha1

from werkzeug import urls

from odoo import fields, models


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(
        selection_add=[('buckaroo', "Buckaroo")], ondelete={'buckaroo': 'set default'})
    buckaroo_website_key = fields.Char(
        string="Website Key", help="The key solely used to identify the website with Buckaroo",
        required_if_provider='buckaroo')
    buckaroo_secret_key = fields.Char(
        string="Buckaroo Secret Key", required_if_provider='buckaroo', groups='base.group_system')

    def _buckaroo_get_api_url(self):
        """ Return the API URL according to the state.

        Note: self.ensure_one()

        :return: The API URL
        :rtype: str
        """
        self.ensure_one()
        if self.state == 'enabled':
            return 'https://checkout.buckaroo.nl/html/'
        else:
            return 'https://testcheckout.buckaroo.nl/html/'

    def _buckaroo_generate_digital_sign(self, values, incoming=True):
        """ Generate the shasign for incoming or outgoing communications.

        :param dict values: The values used to generate the signature
        :param bool incoming: Whether the signature must be generate for an incoming (Buckaroo to
                              Odoo) or outgoing (Odoo to Buckaroo) communication.
        :return: The shasign
        :rtype: str
        """
        if incoming:
            # Remove the signature from the values used to check the signature
            for key in values.keys():
                if key.upper() == 'BRQ_SIGNATURE':  # Keys are case-insensitive
                    del values[key]
                    break
            # Incoming communication values must be URL-decoded before checking the signature
            items = [(k, urls.url_unquote_plus(v)) for k, v in values.items()]
        else:
            # Only use items whose key starts with 'add_', 'brq_', or 'cust_' (case insensitive)
            items = [
                (k, v) for k, v in values.items()
                if any(k.upper().startswith(key_prefix) for key_prefix in ('ADD_', 'BRQ_', 'CUST_'))
            ]
        # Sort parameters by lower-cased key. Not upper- because ord('A') < ord('_') < ord('a').
        sorted_items = sorted(items, key=lambda pair: pair[0].lower())
        # Build the signing string by concatenating all parameters
        sign_string = ''.join(f'{k}={v or ""}' for k, v in sorted_items)
        # Append the pre-shared secret key to the signing string
        sign_string += self.buckaroo_secret_key
        # Calculate the SHA-1 hash over the signing string
        return sha1(sign_string.encode('utf-8')).hexdigest()

    def _get_default_payment_method_id(self):
        self.ensure_one()
        if self.provider != 'buckaroo':
            return super()._get_default_payment_method_id()
        return self.env.ref('payment_buckaroo.payment_method_buckaroo').id

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_buckaroo.const import STATUS_CODES_MAPPING
from odoo.addons.payment_buckaroo.controllers.main import BuckarooController

_logger = logging.getLogger(__name__)


def _normalize_dataset(data):
    """ Set all keys of a dictionary to uppercase.

    As Buckaroo parameters names are case insensitive, we can convert everything to upper case to
    easily detected the presence of a parameter by checking the uppercase key only.

    :param dict data: The dictionary whose keys must be set to uppercase
    :return: A copy of the original data with all keys set to uppercase
    :rtype: dict
    """
    return {key.upper(): val for key, val in data.items()}


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Buckaroo-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of acquirer-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider != 'buckaroo':
            return res

        return_url = urls.url_join(self.acquirer_id.get_base_url(), BuckarooController._return_url)
        rendering_values = {
            'api_url': self.acquirer_id._buckaroo_get_api_url(),
            'Brq_websitekey': self.acquirer_id.buckaroo_website_key,
            'Brq_amount': self.amount,
            'Brq_currency': self.currency_id.name,
            'Brq_invoicenumber': self.reference,
            # Include all 4 URL keys despite they share the same value as they are part of the sig.
            'Brq_return': return_url,
            'Brq_returncancel': return_url,
            'Brq_returnerror': return_url,
            'Brq_returnreject': return_url,
        }
        if self.partner_lang:
            rendering_values['Brq_culture'] = self.partner_lang.replace('_', '-')
        rendering_values['Brq_signature'] = self.acquirer_id._buckaroo_generate_digital_sign(
            rendering_values, incoming=False
        )
        return rendering_values

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on Buckaroo data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        :raise: ValidationError if the data match no transaction
        :raise: ValidationError if the signature can not be verified
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'buckaroo':
            return tx

        normalized_data = _normalize_dataset(data)
        reference = normalized_data.get('BRQ_INVOICENUMBER')
        shasign = normalized_data.get('BRQ_SIGNATURE')
        if not reference or not shasign:
            raise ValidationError(
                "Buckaroo: " + _(
                    "Received data with missing reference (%(ref)s) or shasign (%(sign)s)",
                    ref=reference, sign=shasign
                )
            )

        tx = self.search([('reference', '=', reference), ('provider', '=', 'buckaroo')])
        if not tx:
            raise ValidationError(
                "Buckaroo: " + _("No transaction found matching reference %s.", reference)
            )

        # Verify signature
        shasign_check = tx.acquirer_id._buckaroo_generate_digital_sign(data, incoming=True)
        if shasign_check != shasign:
            raise ValidationError(
                "Buckaroo: " + _(
                    "Invalid shasign: received %(sign)s, computed %(check)s",
                    sign=shasign, check=shasign_check
                )
            )

        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on Buckaroo data.

        Note: self.ensure_one()

        :param dict data: The feedback data sent by the provider
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_feedback_data(data)
        if self.provider != 'buckaroo':
            return

        normalized_data = _normalize_dataset(data)
        transaction_keys = normalized_data.get('BRQ_TRANSACTIONS')
        if not transaction_keys:
            raise ValidationError("Buckaroo: " + _("Received data with missing transaction keys"))
        # BRQ_TRANSACTIONS can hold multiple, comma-separated, tx keys. In practice, it holds only
        # one reference. So we split for semantic correctness and keep the first transaction key.
        self.acquirer_reference = transaction_keys.split(',')[0]

        status_code = int(normalized_data.get('BRQ_STATUSCODE') or 0)
        if status_code in STATUS_CODES_MAPPING['pending']:
            self._set_pending()
        elif status_code in STATUS_CODES_MAPPING['done']:
            self._set_done()
        elif status_code in STATUS_CODES_MAPPING['cancel']:
            self._set_canceled()
        elif status_code in STATUS_CODES_MAPPING['refused']:
            self._set_error(_("Your payment was refused (code %s). Please try again.", status_code))
        elif status_code in STATUS_CODES_MAPPING['error']:
            self._set_error(_("An error occurred during processing of your payment (code %s). Please try again.", status_code))
        else:
            _logger.warning("Buckaroo: received unknown status code: %s", status_code)
            self._set_error("Buckaroo: " + _("Unknown status code: %s", status_code))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_method
from . import payment_acquirer
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CDC484"/><stop offset="100%" stop-color="#B5AA59"/></linearGradient><path id="d" d="M31.757 37.5L34 31h15.036v-3.212a.342.342 0 0 0-.339-.343H35l.92-2.742h13.116c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742 2.258.007 3.907.022 5.137 0h24.31a.342.342 0 0 0 .339-.343V37.5H31.757zM15.46 55.642h-1.036V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.423 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zM13 19h3.688l2.517 7.42h8.184L29.994 19H33l-8.76 26h-1.97L13 19zm7 11l2.964 9L26 30h-6z"/><path id="e" d="M31.757 35.5L34 29h15.036v-3.212a.342.342 0 0 0-.339-.343H35l.92-2.742h13.116c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742 2.258.007 3.907.022 5.137 0h24.31a.342.342 0 0 0 .339-.343V35.5H31.757zM15.46 53.642h-1.036V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.423 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zM13 17h3.688l2.517 7.42h8.184L29.994 17H33l-8.76 26h-1.97L13 17zm7 11l2.964 9L26 28h-6z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916l13.5-16.675L18.889 24l11.172-7 .967 5.703 4.533.036L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_buckaroo_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="Brq_websitekey" t-att-value="Brq_websitekey"/>
            <input type="hidden" name="Brq_amount" t-att-value="Brq_amount"/>
            <input type="hidden" name="Brq_currency" t-att-value="Brq_currency"/>
            <input type="hidden" name="Brq_invoicenumber" t-att-value="Brq_invoicenumber"/>
            <input type="hidden" name="Brq_signature" t-att-value="Brq_signature"/>
            <input t-if="Brq_culture" type="hidden" name="Brq_culture" t-att-value="Brq_culture"/>
            <!-- URLs -->
            <input type="hidden" name="Brq_return" t-att-value="Brq_return"/>
            <input type="hidden" name="Brq_returncancel" t-att-value="Brq_returncancel"/>
            <input type="hidden" name="Brq_returnerror" t-att-value="Brq_returnerror"/>
            <input type="hidden" name="Brq_returnreject" t-att-value="Brq_returnreject"/>
        </form>
    </template>

</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">Buckaroo Acquirer Form</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.payment_acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'buckaroo')]}">
                    <field name="buckaroo_website_key" attrs="{'required':[ ('provider', '=', 'buckaroo'), ('state', '!=', 'disabled')]}"/>
                    <field name="buckaroo_secret_key" string="Secret Key" attrs="{'required':[ ('provider', '=', 'buckaroo'), ('state', '!=', 'disabled')]}"/>
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

