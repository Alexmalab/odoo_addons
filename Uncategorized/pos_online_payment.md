# Odoo Module: pos_online_payment

Category: Uncategorized

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
    'name': 'Point of Sale online payment',
    'depends': ['point_of_sale', 'account_payment'],
    'data': [
        'views/payment_transaction_views.xml',
        'views/pos_payment_views.xml',
        'views/pos_payment_method_views.xml',
        'views/payment_portal_templates.xml',
        'views/account_payment_views.xml',
    ],
    'auto_install': True,
    'installable': True,
    'assets': {
        'point_of_sale.assets_prod': [
            'pos_online_payment/static/src/app/**/*',
            'pos_online_payment/static/src/css/**/*',
        ],
        'web.assets_tests': [
            'pos_online_payment/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\payment_portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_encode

from odoo import _, http, tools
from odoo.http import request
from odoo.exceptions import AccessError, ValidationError, UserError
from odoo.addons.payment.controllers import portal as payment_portal


class PaymentPortal(payment_portal.PaymentPortal):

    def _check_order_access(self, pos_order_id, access_token):
        try:
            order_sudo = self._document_check_access(
                'pos.order', pos_order_id, access_token)
        except:
            raise AccessError(
                _("The provided order or access token is invalid."))

        if order_sudo.state == "cancel":
            raise ValidationError(_("The order has been canceled."))
        return order_sudo

    @staticmethod
    def _ensure_session_open(pos_order_sudo):
        if pos_order_sudo.session_id.state != 'opened':
            raise AccessError(_("The POS session is not opened."))

    def _get_partner_sudo(self, user_sudo):
        return user_sudo.partner_id

    def _redirect_login(self):
        return request.redirect('/web/login?' + url_encode({'redirect': request.httprequest.full_path}))

    @staticmethod
    def _get_amount_to_pay(order_to_pay_sudo):
        if order_to_pay_sudo.state in ('paid', 'done', 'invoiced'):
            return 0.0
        amount = order_to_pay_sudo._get_checked_next_online_payment_amount()
        if amount and PaymentPortal._is_valid_amount(amount, order_to_pay_sudo.currency_id):
            return amount
        else:
            return order_to_pay_sudo.get_amount_unpaid()

    @staticmethod
    def _is_valid_amount(amount, currency):
        return isinstance(amount, float) and tools.float_compare(amount, 0.0, precision_rounding=currency.rounding) > 0

    def _get_allowed_providers_sudo(self, pos_order_sudo, partner_id, amount_to_pay):
        payment_method = pos_order_sudo.online_payment_method_id
        if not payment_method:
            raise UserError(_("There is no online payment method configured for this Point of Sale order."))
        compatible_providers_sudo = request.env['payment.provider'].sudo()._get_compatible_providers(
            pos_order_sudo.company_id.id, partner_id, amount_to_pay, currency_id=pos_order_sudo.currency_id.id
        )  # In sudo mode to read the fields of providers and partner (if logged out).
        # Return the payment providers configured in the pos.payment.method that are compatible for the payment API
        return compatible_providers_sudo & payment_method._get_online_payment_providers(pos_order_sudo.config_id.id, error_if_invalid=False)

    @staticmethod
    def _new_url_params(access_token, exit_route=None):
        url_params = {
            'access_token': access_token,
        }
        if exit_route:
            url_params['exit_route'] = exit_route
        return url_params

    @staticmethod
    def _get_pay_route(pos_order_id, access_token, exit_route=None):
        return f'/pos/pay/{pos_order_id}?' + url_encode(PaymentPortal._new_url_params(access_token, exit_route))

    @staticmethod
    def _get_landing_route(pos_order_id, access_token, exit_route=None, tx_id=None):
        url_params = PaymentPortal._new_url_params(access_token, exit_route)
        if tx_id:
            url_params['tx_id'] = tx_id
        return f'/pos/pay/confirmation/{pos_order_id}?' + url_encode(url_params)

    @http.route('/pos/pay/<int:pos_order_id>', type='http', methods=['GET'], auth='public', website=True, sitemap=False)
    def pos_order_pay(self, pos_order_id, access_token=None, exit_route=None):
        """ Behaves like payment.PaymentPortal.payment_pay but for POS online payment.

        :param int pos_order_id: The POS order to pay, as a `pos.order` id
        :param str access_token: The access token used to verify the user
        :param str exit_route: The URL to open to leave the POS online payment flow

        :return: The rendered payment form
        :rtype: str
        :raise: AccessError if the provided order or access token is invalid
        :raise: ValidationError if data on the server prevents the payment
        """
        pos_order_sudo = self._check_order_access(pos_order_id, access_token)
        self._ensure_session_open(pos_order_sudo)

        user_sudo = request.env.user
        if not pos_order_sudo.partner_id:
            user_sudo = request.env.ref('base.public_user')
        logged_in = not user_sudo._is_public()
        partner_sudo = pos_order_sudo.partner_id or self._get_partner_sudo(user_sudo)
        if not partner_sudo:
            return self._redirect_login()

        kwargs = {
            'pos_order_id': pos_order_sudo.id,
        }
        rendering_context = {
            **kwargs,
            'exit_route': exit_route,
            'reference_prefix': request.env['payment.transaction'].sudo()._compute_reference_prefix(provider_code=None, separator='-', **kwargs),
            'partner_id': partner_sudo.id,
            'access_token': access_token,
            'transaction_route': f'/pos/pay/transaction/{pos_order_sudo.id}?' + url_encode(PaymentPortal._new_url_params(access_token, exit_route)),
            'landing_route': self._get_landing_route(pos_order_sudo.id, access_token, exit_route=exit_route),
            **self._get_extra_payment_form_values(**kwargs),
        }

        currency_id = pos_order_sudo.currency_id

        if not currency_id.active:
            rendering_context['currency'] = False
            return self._render_pay(rendering_context)
        rendering_context['currency'] = currency_id

        amount_to_pay = self._get_amount_to_pay(pos_order_sudo)
        if not self._is_valid_amount(amount_to_pay, currency_id):
            rendering_context['amount'] = False
            return self._render_pay(rendering_context)
        rendering_context['amount'] = amount_to_pay

        # Select all the payment methods and tokens that match the payment context.
        providers_sudo = self._get_allowed_providers_sudo(pos_order_sudo, partner_sudo.id, amount_to_pay)
        payment_methods_sudo = request.env['payment.method'].sudo()._get_compatible_payment_methods(
            providers_sudo.ids,
            partner_sudo.id,
            currency_id=currency_id.id,
        )  # In sudo mode to read the fields of providers.
        if logged_in:
            tokens_sudo = request.env['payment.token'].sudo()._get_available_tokens(
                providers_sudo.ids, partner_sudo.id
            )  # In sudo mode to be able to read the fields of providers.
            show_tokenize_input_mapping = self._compute_show_tokenize_input_mapping(
                providers_sudo, **kwargs)
        else:
            tokens_sudo = request.env['payment.token']
            show_tokenize_input_mapping = dict.fromkeys(providers_sudo.ids, False)

        rendering_context.update({
            'providers_sudo': providers_sudo,
            'payment_methods_sudo': payment_methods_sudo,
            'tokens_sudo': tokens_sudo,
            'show_tokenize_input_mapping': show_tokenize_input_mapping,
            **self._get_extra_payment_form_values(**kwargs),
        })
        return self._render_pay(rendering_context)

    def _render_pay(self, rendering_context):
        return request.render('pos_online_payment.pay', rendering_context)

    @http.route('/pos/pay/transaction/<int:pos_order_id>', type='json', auth='public', website=True, sitemap=False)
    def pos_order_pay_transaction(self, pos_order_id, access_token=None, **kwargs):
        """ Behaves like payment.PaymentPortal.payment_transaction but for POS online payment.

        :param int pos_order_id: The POS order to pay, as a `pos.order` id
        :param str access_token: The access token used to verify the user
        :param str exit_route: The URL to open to leave the POS online payment flow
        :param dict kwargs: Data from payment module

        :return: The mandatory values for the processing of the transaction
        :rtype: dict
        :raise: AccessError if the provided order or access token is invalid
        :raise: ValidationError if data on the server prevents the payment
        :raise: UserError if data provided by the user is invalid/missing
        """
        pos_order_sudo = self._check_order_access(pos_order_id, access_token)
        self._ensure_session_open(pos_order_sudo)
        exit_route = request.httprequest.args.get('exit_route')
        user_sudo = request.env.user
        if not pos_order_sudo.partner_id:
            user_sudo = request.env.ref('base.public_user')
        logged_in = not user_sudo._is_public()
        partner_sudo = pos_order_sudo.partner_id or self._get_partner_sudo(user_sudo)
        if not partner_sudo:
            return self._redirect_login()

        self._validate_transaction_kwargs(kwargs)
        if kwargs.get('is_validation'):
            raise UserError(
                _("A validation payment cannot be used for a Point of Sale online payment."))

        if 'partner_id' in kwargs and kwargs['partner_id'] != partner_sudo.id:
            raise UserError(
                _("The provided partner_id is different than expected."))
        # Avoid tokenization for the public user.
        kwargs.update({
            'partner_id': partner_sudo.id,
            'partner_phone': partner_sudo.phone,
            'custom_create_values': {
                'pos_order_id': pos_order_sudo.id,
            },
        })
        if not logged_in:
            if kwargs.get('tokenization_requested') or kwargs.get('flow') == 'token':
                raise UserError(
                    _("Tokenization is not available for logged out customers."))
            kwargs['custom_create_values']['tokenize'] = False

        currency_id = pos_order_sudo.currency_id
        if not currency_id.active:
            raise ValidationError(_("The currency is invalid."))
        # Ignore the currency provided by the customer
        kwargs['currency_id'] = currency_id.id

        amount_to_pay = self._get_amount_to_pay(pos_order_sudo)
        if not self._is_valid_amount(amount_to_pay, currency_id):
            raise ValidationError(_("There is nothing to pay for this order."))
        if tools.float_compare(kwargs['amount'], amount_to_pay, precision_rounding=currency_id.rounding) != 0:
            raise ValidationError(
                _("The amount to pay has changed. Please refresh the page."))

        payment_option_id = kwargs.get('payment_method_id') or kwargs.get('token_id')
        if not payment_option_id:
            raise UserError(_("A payment option must be specified."))
        flow = kwargs.get('flow')
        if not (flow and flow in ['redirect', 'direct', 'token']):
            raise UserError(_("The payment should either be direct, with redirection, or made by a token."))
        providers_sudo = self._get_allowed_providers_sudo(pos_order_sudo, partner_sudo.id, amount_to_pay)
        if flow == 'token':
            tokens_sudo = request.env['payment.token']._get_available_tokens(
                providers_sudo.ids, partner_sudo.id)
            if payment_option_id not in tokens_sudo.ids:
                raise UserError(_("The payment token is invalid."))
        else:
            if kwargs.get('provider_id') not in providers_sudo.ids:
                raise UserError(_("The payment provider is invalid."))

        kwargs['reference_prefix'] = None  # Computed with pos_order_id
        kwargs.pop('pos_order_id', None) # _create_transaction kwargs keys must be different than custom_create_values keys

        tx_sudo = self._create_transaction(**kwargs)
        tx_sudo.landing_route = PaymentPortal._get_landing_route(pos_order_sudo.id, access_token, exit_route=exit_route, tx_id=tx_sudo.id)

        return tx_sudo._get_processing_values()

    @http.route('/pos/pay/confirmation/<int:pos_order_id>', type='http', methods=['GET'], auth='public', website=True, sitemap=False)
    def pos_order_pay_confirmation(self, pos_order_id, tx_id=None, access_token=None, exit_route=None, **kwargs):
        """ Behaves like payment.PaymentPortal.payment_confirm but for POS online payment.

        :param int pos_order_id: The POS order to confirm, as a `pos.order` id
        :param str tx_id: The transaction to confirm, as a `payment.transaction` id
        :param str access_token: The access token used to verify the user
        :param str exit_route: The URL to open to leave the POS online payment flow
        :param dict kwargs: Data from payment module

        :return: The rendered confirmation page
        :rtype: str
        :raise: AccessError if the provided order or access token is invalid
        """
        tx_id = self._cast_as_int(tx_id)
        rendering_context = {
            'state': 'error',
            'exit_route': exit_route,
            'pay_route': self._get_pay_route(pos_order_id, access_token, exit_route)
        }
        if not tx_id or not pos_order_id:
            return self._render_pay_confirmation(rendering_context)

        pos_order_sudo = self._check_order_access(pos_order_id, access_token)

        tx_sudo = request.env['payment.transaction'].sudo().search([('id', '=', tx_id)])
        if tx_sudo.pos_order_id.id != pos_order_sudo.id:
            return self._render_pay_confirmation(rendering_context)

        rendering_context.update(
            pos_order_id=pos_order_sudo.id,
            order_reference=pos_order_sudo.pos_reference,
            tx_reference=tx_sudo.reference,
            amount=tx_sudo.amount,
            currency=tx_sudo.currency_id,
            provider_name=tx_sudo.provider_id.name,
            tx=tx_sudo, # for the payment.transaction_status template
        )

        if tx_sudo.state not in ('authorized', 'done'):
            rendering_context['state'] = 'tx_error'
            return self._render_pay_confirmation(rendering_context)

        tx_sudo._process_pos_online_payment()

        rendering_context['state'] = 'success'
        return self._render_pay_confirmation(rendering_context)

    def _render_pay_confirmation(self, rendering_context):
        return request.render('pos_online_payment.pay_confirmation', rendering_context)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_portal

```

## File: models\account_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models


class AccountPayment(models.Model):
    _inherit = 'account.payment'

    pos_order_id = fields.Many2one('pos.order', string='POS Order', help='The Point of Sale order linked to this payment', readonly=True)

    def action_view_pos_order(self):
        """ Return the action for the view of the pos order linked to the payment.
        """
        self.ensure_one()

        action = {
            'name': _("POS Order"),
            'type': 'ir.actions.act_window',
            'res_model': 'pos.order',
            'target': 'current',
            'res_id': self.pos_order_id.id,
            'view_mode': 'form'
        }
        return action

```

## File: models\payment_transaction.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models, tools
from odoo.exceptions import ValidationError


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    pos_order_id = fields.Many2one('pos.order', string='POS Order', help='The Point of Sale order linked to the payment transaction', readonly=True)

    @api.model
    def _compute_reference_prefix(self, provider_code, separator, **values):
        """ Override of payment to compute the reference prefix based on POS-specific values.

        :return: The computed reference prefix if POS order id is found, the one of `super` otherwise
        :rtype: str
        """
        pos_order_id = values.get('pos_order_id')
        if pos_order_id:
            pos_order = self.env['pos.order'].sudo().browse(pos_order_id).exists()
            if pos_order:
                return pos_order.pos_reference
        return super()._compute_reference_prefix(provider_code, separator, **values)

    def _reconcile_after_done(self):
        """ Override of payment to process POS online payments automatically. """
        super()._reconcile_after_done()
        self._process_pos_online_payment()

    def _process_pos_online_payment(self):
        for tx in self:
            if tx and tx.pos_order_id and tx.state in ('authorized', 'done') and not tx.payment_id.pos_order_id:
                pos_order = tx.pos_order_id
                if tools.float_compare(tx.amount, 0.0, precision_rounding=pos_order.currency_id.rounding) <= 0:
                    raise ValidationError(_('The payment transaction (%d) has a negative amount.', tx.id))

                if not tx.payment_id: # the payment could already have been created by account_payment module
                    tx._create_payment()
                if not tx.payment_id:
                    raise ValidationError(_('The POS online payment (tx.id=%d) could not be saved correctly', tx.id))

                payment_method = pos_order.online_payment_method_id
                if not payment_method:
                    pos_config = pos_order.config_id
                    payment_method = self.env['pos.payment.method'].sudo()._get_or_create_online_payment_method(pos_config.company_id.id, pos_config.id)
                    if not payment_method:
                        raise ValidationError(_('The POS online payment (tx.id=%d) could not be saved correctly because the online payment method could not be found', tx.id))

                pos_order.add_payment({
                    'amount': tx.amount,
                    'payment_date': tx.last_state_change,
                    'payment_method_id': payment_method.id,
                    'online_account_payment_id': tx.payment_id.id,
                    'pos_order_id': pos_order.id,
                })
                tx.payment_id.update({
                    'pos_payment_method_id': payment_method.id,
                    'pos_order_id': pos_order.id,
                    'pos_session_id': pos_order.session_id.id,
                })
                if pos_order.state == 'draft' and pos_order._is_pos_order_paid():
                    pos_order._process_saved_order(False)
                pos_order._send_online_payments_notification_via_bus()

    def action_view_pos_order(self):
        """ Return the action for the view of the pos order linked to the transaction.
        """
        self.ensure_one()

        action = {
            'name': _("POS Order"),
            'type': 'ir.actions.act_window',
            'res_model': 'pos.order',
            'target': 'current',
            'res_id': self.pos_order_id.id,
            'view_mode': 'form'
        }
        return action

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, _
from odoo.exceptions import ValidationError


class PosConfig(models.Model):
    _inherit = 'pos.config'

    @api.constrains('payment_method_ids')
    def _check_online_payment_methods(self):
        """ Checks the journal currency with _get_online_payment_providers(..., error_if_invalid=True)"""
        for config in self:
            opm_amount = 0
            for pm in config.payment_method_ids:
                if pm.is_online_payment:
                    opm_amount += 1
                    if opm_amount > 1:
                        raise ValidationError(_("A POS config cannot have more than one online payment method."))
                    if not pm._get_online_payment_providers(config.id, error_if_invalid=True):
                        raise ValidationError(_("To use an online payment method in a POS config, it must have at least one published payment provider supporting the currency of that POS config."))

    def _get_cashier_online_payment_method(self):
        self.ensure_one()
        return self.payment_method_ids.filtered('is_online_payment')[:1]

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, tools


class PosOrder(models.Model):
    _inherit = 'pos.order'

    online_payment_method_id = fields.Many2one('pos.payment.method', compute="_compute_online_payment_method_id")
    next_online_payment_amount = fields.Float(string='Next online payment amount to pay', digits=0, required=False) # unlimited precision

    @api.depends('config_id.payment_method_ids')
    def _compute_online_payment_method_id(self):
        for order in self:
            order.online_payment_method_id = order.config_id._get_cashier_online_payment_method()

    def get_amount_unpaid(self):
        self.ensure_one()
        return self.currency_id.round(self._get_rounded_amount(self.amount_total) - self.amount_paid)

    def _clean_payment_lines(self):
        self.ensure_one()
        order_payments = self.env['pos.payment'].search(['&', ('pos_order_id', '=', self.id), ('online_account_payment_id', '=', False)])
        order_payments.unlink()

    def get_and_set_online_payments_data(self, next_online_payment_amount=False):
        """ Allows to update the amount to pay for the next online payment and
            get online payments already made and how much remains to be paid.
            If next_online_payment_amount is different than False, updates the
            next online payment amount, otherwise, the next online payment amount
            is unchanged.
            If next_online_payment_amount is 0 and the order has no successful
            online payment, is in draft state, is not a restaurant order and the
            pos.config has no trusted config, then the order is deleted from the
            database, because it was probably added for the online payment flow.
        """
        self.ensure_one()
        is_paid = self.state in ('paid', 'done', 'invoiced')
        if is_paid:
            return {
                'id': self.id,
                'paid_order': self._export_for_ui(self)
            }

        online_payments = self.sudo().env['pos.payment'].search_read(domain=['&', ('pos_order_id', '=', self.id), ('online_account_payment_id', '!=', False)], fields=['payment_method_id', 'amount'], load=False)
        return_data = {
            'id': self.id,
            'online_payments': online_payments,
            'amount_unpaid': self.get_amount_unpaid(),
        }
        if not isinstance(next_online_payment_amount, bool):
            if tools.float_is_zero(next_online_payment_amount, precision_rounding=self.currency_id.rounding) and len(online_payments) == 0 and self.state == 'draft' and not self.config_id.module_pos_restaurant and len(self.config_id.trusted_config_ids) == 0:
                self.sudo()._clean_payment_lines() # Needed to delete the order
                self.sudo().unlink()
                return_data['deleted'] = True
            elif self._check_next_online_payment_amount(next_online_payment_amount):
                self.next_online_payment_amount = next_online_payment_amount

        return return_data

    def _send_online_payments_notification_via_bus(self):
        self.ensure_one()
        # The bus communication is only protected by the name of the channel.
        # Therefore, no sensitive information is sent through it, only a
        # notification to invite the local browser to do a safe RPC to
        # the server to check the new state of the order.
        self.env['bus.bus']._sendone(self.session_id._get_bus_channel_name(), 'ONLINE_PAYMENTS_NOTIFICATION', {'id': self.id})

    def _check_next_online_payment_amount(self, amount):
        self.ensure_one()
        return tools.float_compare(amount, 0.0, precision_rounding=self.currency_id.rounding) >= 0 and tools.float_compare(amount, self.get_amount_unpaid(), precision_rounding=self.currency_id.rounding) <= 0

    def _get_checked_next_online_payment_amount(self):
        self.ensure_one()
        amount = self.next_online_payment_amount
        return amount if self._check_next_online_payment_amount(amount) else False

```

## File: models\pos_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from odoo import api, fields, models, _
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class PosPayment(models.Model):
    _inherit = 'pos.payment'

    online_account_payment_id = fields.Many2one('account.payment', string='Online accounting payment', readonly=True) # One2one

    @api.model_create_multi
    def create(self, vals_list):
        online_account_payments_by_pm = {}
        for vals in vals_list:
            pm_id = vals['payment_method_id']
            if pm_id not in online_account_payments_by_pm:
                online_account_payments_by_pm[pm_id] = set()
            online_account_payments_by_pm[pm_id].add(vals.get('online_account_payment_id'))

        opms_read_id = self.env['pos.payment.method'].search_read(['&', ('id', 'in', list(online_account_payments_by_pm.keys())), ('is_online_payment', '=', True)], ["id"])
        opms_id = {opm_read_id['id'] for opm_read_id in opms_read_id}
        online_account_payments_to_check_id = set()

        for pm_id, oaps_id in online_account_payments_by_pm.items():
            if pm_id in opms_id:
                if None in oaps_id:
                    raise UserError(_("Cannot create a POS online payment without an accounting payment."))
                else:
                    online_account_payments_to_check_id.update(oaps_id)
            elif any(oaps_id):
                raise UserError(_("Cannot create a POS payment with a not online payment method and an online accounting payment."))

        if online_account_payments_to_check_id:
            valid_oap_amount = self.env['account.payment'].search_count([('id', 'in', list(online_account_payments_to_check_id))])
            if valid_oap_amount != len(online_account_payments_to_check_id):
                raise UserError(_("Cannot create a POS online payment without an accounting payment."))

        return super().create(vals_list)

    def write(self, vals):
        if vals.keys() & ('amount', 'payment_date', 'payment_method_id', 'online_account_payment_id', 'pos_order_id') and any(payment.online_account_payment_id or payment.payment_method_id.is_online_payment for payment in self):
            raise UserError(_("Cannot edit a POS online payment essential data."))
        return super().write(vals)

    @api.constrains('payment_method_id')
    def _check_payment_method_id(self):
        bypass_check_payments = self.filtered('payment_method_id.is_online_payment')
        if any(payment.payment_method_id != payment.pos_order_id.online_payment_method_id for payment in bypass_check_payments):
            # An online payment must always be saved for the POS, even if the online payment method is no longer configured/allowed in the pos.config, because in any case it is saved by account_payment and payment modules.
            _logger.warning("Allow to save a POS online payment with an unexpected online payment method")

        super(PosPayment, self - bypass_check_payments)._check_payment_method_id()

```

## File: models\pos_payment_method.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class PosPaymentMethod(models.Model):
    _inherit = "pos.payment.method"

    is_online_payment = fields.Boolean(string="Online Payment", help="Use this payment method for online payments (payments made on a web page with online payment providers)", default=False)
    online_payment_provider_ids = fields.Many2many('payment.provider', string="Allowed Providers", domain="[('is_published', '=', True), ('state', 'in', ['enabled', 'test'])]")
    has_an_online_payment_provider = fields.Boolean(compute='_compute_has_an_online_payment_provider', readonly=True)
    type = fields.Selection(selection_add=[('online', 'Online')])

    @api.depends('is_online_payment')
    def _compute_type(self):
        opm = self.filtered('is_online_payment')
        if opm:
            opm.type = 'online'

        super(PosPaymentMethod, self - opm)._compute_type()

    def _get_online_payment_providers(self, pos_config_id=False, error_if_invalid=True):
        self.ensure_one()
        providers_sudo = self.sudo().online_payment_provider_ids
        if not providers_sudo: # Empty = all published providers
            providers_sudo = self.sudo().env['payment.provider'].search([('is_published', '=', True), ('state', 'in', ['enabled', 'test'])])

        if not pos_config_id:
            return providers_sudo

        config_currency = self.sudo().env['pos.config'].browse(pos_config_id).currency_id
        valid_providers = providers_sudo.filtered(lambda p: not p.journal_id.currency_id or p.journal_id.currency_id == config_currency)
        if error_if_invalid and len(providers_sudo) != len(valid_providers):
            raise ValidationError(_("All payment providers configured for an online payment method must use the same currency as the Sales Journal, or the company currency if that is not set, of the POS config."))
        return valid_providers

    @api.depends('is_online_payment', 'online_payment_provider_ids')
    def _compute_has_an_online_payment_provider(self):
        for pm in self:
            if pm.is_online_payment:
                pm.has_an_online_payment_provider = bool(pm._get_online_payment_providers())
            else:
                pm.has_an_online_payment_provider = False

    def _is_write_forbidden(self, fields):
        return super(PosPaymentMethod, self)._is_write_forbidden(fields - {'online_payment_provider_ids'})

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('is_online_payment', False):
                self._force_online_payment_values(vals)
        return super().create(vals_list)

    def write(self, vals):
        if 'is_online_payment' in vals:
            if vals['is_online_payment']:
                self._force_online_payment_values(vals)
            return super().write(vals)

        opm = self.filtered('is_online_payment')
        not_opm = self - opm

        res = True
        if opm:
            forced_vals = vals.copy()
            self._force_online_payment_values(forced_vals, True)
            res = super(PosPaymentMethod, opm).write(forced_vals) and res
        if not_opm:
            res = super(PosPaymentMethod, not_opm).write(vals) and res

        return res

    @staticmethod
    def _force_online_payment_values(vals, if_present=False):
        if 'type' in vals:
            vals['type'] = 'online'

        disabled_fields_name = ('split_transactions', 'receivable_account_id', 'outstanding_account_id', 'journal_id', 'is_cash_count', 'use_payment_terminal')
        if if_present:
            for name in disabled_fields_name:
                if name in vals:
                    vals[name] = False
        else:
            for name in disabled_fields_name:
                vals[name] = False

    def _get_payment_terminal_selection(self):
        return super(PosPaymentMethod, self)._get_payment_terminal_selection() if not self.is_online_payment else []

    @api.depends('type')
    def _compute_hide_use_payment_terminal(self):
        opm = self.filtered(lambda pm: pm.type == 'online')
        if opm:
            opm.hide_use_payment_terminal = True
        super(PosPaymentMethod, self - opm)._compute_hide_use_payment_terminal()

    @api.model
    def _get_or_create_online_payment_method(self, company_id, pos_config_id):
        """ Get the first online payment method compatible with the provided pos.config.
            If there isn't any, try to find an existing one in the same company and return it without adding the pos.config to it.
            If there is not, create a new one for the company and return it without adding the pos.config to it.
        """
        # Parameters are ids instead of a pos.config record because this method can be called from a web controller or internally
        payment_method_id = self.env['pos.payment.method'].search([('is_online_payment', '=', True), ('company_id', '=', company_id), ('config_ids', 'in', pos_config_id)], limit=1).exists()
        if not payment_method_id:
            payment_method_id = self.env['pos.payment.method'].search([('is_online_payment', '=', True), ('company_id', '=', company_id)], limit=1).exists()
            if not payment_method_id:
                payment_method_id = self.env['pos.payment.method'].create({
                    'name': _('Online Payment'),
                    'is_online_payment': True,
                    'company_id': company_id,
                })
                if not payment_method_id:
                    raise ValidationError(_('Could not create an online payment method (company_id=%d, pos_config_id=%d)', company_id, pos_config_id))
        return payment_method_id

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from odoo import models, tools, _
from odoo.exceptions import UserError


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_pos_payment_method(self):
        result = super()._loader_params_pos_payment_method()
        result['search_params']['fields'].append('is_online_payment')
        return result

    def _accumulate_amounts(self, data):
        data = super()._accumulate_amounts(data)
        amounts = lambda: {'amount': 0.0, 'amount_converted': 0.0}

        split_receivables_online = defaultdict(amounts)
        currency_rounding = self.currency_id.rounding
        for order in self._get_closed_orders():
            for payment in order.payment_ids:
                amount = payment.amount
                if tools.float_is_zero(amount, precision_rounding=currency_rounding):
                    continue
                date = payment.payment_date
                payment_method = payment.payment_method_id
                payment_type = payment_method.type

                if payment_type == 'online':
                    split_receivables_online[payment] = self._update_amounts(split_receivables_online[payment], {'amount': amount}, date)

        data.update({'split_receivables_online': split_receivables_online,})
        return data

    def _create_bank_payment_moves(self, data):
        data = super()._create_bank_payment_moves(data)

        split_receivables_online = data.get('split_receivables_online')
        MoveLine = data.get('MoveLine')

        online_payment_to_receivable_lines = {}

        for payment, amounts in split_receivables_online.items():
            split_receivable_line = MoveLine.create(self._get_split_receivable_op_vals(payment, amounts['amount'], amounts['amount_converted']))
            account_payment = payment.online_account_payment_id
            payment_receivable_line = account_payment.move_id.line_ids.filtered(lambda line: line.account_id == account_payment.destination_account_id)
            online_payment_to_receivable_lines[payment] = split_receivable_line | payment_receivable_line

        data['online_payment_to_receivable_lines'] = online_payment_to_receivable_lines
        return data

    def _get_split_receivable_op_vals(self, payment, amount, amount_converted):
        partner = payment.online_account_payment_id.partner_id
        accounting_partner = self.env["res.partner"]._find_accounting_partner(partner)
        if not accounting_partner:
            raise UserError(_("The partner of the POS online payment (id=%d) could not be found", payment.id))
        partial_vals = {
            'account_id': accounting_partner.property_account_receivable_id.id,
            'move_id': self.move_id.id,
            'partner_id': accounting_partner.id,
            'name': '%s - %s (%s)' % (self.name, payment.payment_method_id.name, payment.online_account_payment_id.payment_method_line_id.payment_provider_id.name),
        }
        return self._debit_amounts(partial_vals, amount, amount_converted)

    def _reconcile_account_move_lines(self, data):
        data = super()._reconcile_account_move_lines(data)
        online_payment_to_receivable_lines = data.get('online_payment_to_receivable_lines')

        for payment, lines in online_payment_to_receivable_lines.items():
            if payment.online_account_payment_id.partner_id.property_account_receivable_id.reconcile:
                lines.filtered(lambda line: not line.reconciled).reconcile()

        return data

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_payment_method
from . import pos_payment
from . import account_payment
from . import payment_transaction
from . import pos_config
from . import pos_order
from . import pos_session

```

## File: static\src\app\bus\pos_bus_service.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PosBus } from "@point_of_sale/app/bus/pos_bus_service";

patch(PosBus.prototype, {
    //@override
    dispatch(message) {
        super.dispatch(...arguments);

        if (message.type === "ONLINE_PAYMENTS_NOTIFICATION") {
            // The bus communication is only protected by the name of the channel.
            // Therefore, no sensitive information is sent through it, only a
            // notification to invite the local browser to do a safe RPC to
            // the server to check the new state of the order.
            const currentOrder = this.pos.get_order();
            if (currentOrder && currentOrder.server_id === message.payload.id) {
                currentOrder.update_online_payments_data_with_server(this.orm, false);
            }
        }
    },
});

```

## File: static\src\app\customer_display\customer_display_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_online_payment.CustomerFacingDisplayHead" t-inherit="point_of_sale.CustomerFacingDisplayHead" t-inherit-mode="extension">
        <xpath expr="//div[@class='resources']" position="inside">
            <link rel="stylesheet" type="text/css" href="/pos_online_payment/static/src/css/customer_facing_display.css" />
        </xpath>
    </t>

    <t t-name="pos_online_payment.CustomerFacingDisplayMainContainer" t-inherit="point_of_sale.CustomerFacingDisplayMainContainer" t-inherit-mode="extension">
        <xpath expr="//t[@t-call='point_of_sale.CustomerFacingDisplayOrderLines']" position="replace">
            <t t-if="order.get_current_screen_data().name === 'PaymentScreen' and order.uiState.PaymentScreen?.onlinePaymentData">
                <div class="online-payment">
                    <div class="instructions">
                        <p>Please scan the QR code to open the payment page</p>
                        <div class="spacer"/>
                        <div class="qr-code" alt="QR Code to pay" t-attf-style="background-image: url('{{order.uiState.PaymentScreen.onlinePaymentData.qrCode}}');" />
                        <div class="spacer"/>
                    </div>
                    <div class="info">
                        <div>
                            <span>Amount: </span>
                            <span class="amount" t-esc="pos.env.utils.formatCurrency(order.uiState.PaymentScreen.onlinePaymentData.amount)" />
                        </div>
                        <div>
                            <span>Order reference: </span>
                            <span t-esc="order.name" />
                        </div>
                        <div>
                            <span>Order id: </span>
                            <span t-esc="order.server_id" />
                        </div>
                    </div>
                </div>
            </t>
            <t t-else="">
                <t t-call="point_of_sale.CustomerFacingDisplayOrderLines" />
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\app\screens\payment_screen\payment_screen.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";
import { OnlinePaymentPopup } from "@pos_online_payment/app/utils/online_payment_popup/online_payment_popup";
import { ConfirmPopup } from "@point_of_sale/app/utils/confirm_popup/confirm_popup";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { floatIsZero } from "@web/core/utils/numbers";
import { qrCodeSrc } from "@point_of_sale/utils";

patch(PaymentScreen.prototype, {
    getRemainingOnlinePaymentLines() {
        return this.paymentLines.filter(
            (line) => line.payment_method.is_online_payment && line.get_payment_status() !== "done"
        );
    },
    checkRemainingOnlinePaymentLines(unpaidAmount) {
        const remainingLines = this.getRemainingOnlinePaymentLines();
        let remainingAmount = 0;
        let amount = 0;
        for (const line of remainingLines) {
            amount = line.get_amount();
            if (amount <= 0) {
                this.popup.add(ErrorPopup, {
                    title: _t("Invalid online payment"),
                    body: _t(
                        "Online payments cannot have a negative amount (%s: %s).",
                        line.payment_method.name,
                        this.env.utils.formatCurrency(amount)
                    ),
                });
                return false;
            }
            remainingAmount += amount;
        }
        if (!floatIsZero(unpaidAmount - remainingAmount, this.pos.currency.decimal_places)) {
            this.popup.add(ErrorPopup, {
                title: _t("Invalid online payments"),
                body: _t(
                    "The total amount of remaining online payments to execute (%s) doesn't correspond to the remaining unpaid amount of the order (%s).",
                    this.env.utils.formatCurrency(remainingAmount),
                    this.env.utils.formatCurrency(unpaidAmount)
                ),
            });
            return false;
        }
        return true;
    },
    //@override
    async _isOrderValid(isForceValidate) {
        if (!await super._isOrderValid(...arguments)) {
            return false;
        }

        if (!this.payment_methods_from_config.some((pm) => pm.is_online_payment)) {
            return true;
        }

        if (this.currentOrder.finalized) {
            this.afterOrderValidation(false);
            return false;
        }

        const onlinePaymentLines = this.getRemainingOnlinePaymentLines();
        if (onlinePaymentLines.length > 0) {
            // Send the order to the server everytime before the online payments process to
            // allow the server to get the data for online payments and link the successful
            // online payments to the order.
            // The validation process will be done by the server directly after a successful
            // online payment that makes the order fully paid.
            this.currentOrder.date_order = luxon.DateTime.now();
            this.currentOrder.save_to_db();
            this.pos.addOrderToUpdateSet();

            try {
                await this.pos.sendDraftToServer();
            } catch (error) {
                // Code from _finalizeValidation():
                if (error.code == 700 || error.code == 701) {
                    this.error = true;
                }

                if ("code" in error) {
                    // We started putting `code` in the rejected object for invoicing error.
                    // We can continue with that convention such that when the error has `code`,
                    // then it is an error when invoicing. Besides, _handlePushOrderError was
                    // introduce to handle invoicing error logic.
                    await this._handlePushOrderError(error);
                }
                this.showSaveOrderOnServerErrorPopup();
                return false;
            }

            if (!this.currentOrder.server_id) {
                this.showSaveOrderOnServerErrorPopup();
                return false;
            }

            if (!this.currentOrder.server_id) {
                this.cancelOnlinePayment(this.currentOrder);
                this.popup.add(ErrorPopup, {
                    title: _t("Online payment unavailable"),
                    body: _t("The QR Code for paying could not be generated."),
                });
                return false;
            }
            const qrCodeImgSrc = qrCodeSrc(`${this.pos.base_url}/pos/pay/${this.currentOrder.server_id}?access_token=${this.currentOrder.access_token}`);

            let prevOnlinePaymentLine = null;
            let lastOrderServerOPData = null;
            for (const onlinePaymentLine of onlinePaymentLines) {
                const onlinePaymentLineAmount = onlinePaymentLine.get_amount();
                // The local state is not aware if the online payment has already been done.
                lastOrderServerOPData = await this.currentOrder.update_online_payments_data_with_server(this.pos.orm, onlinePaymentLineAmount);
                if (!lastOrderServerOPData) {
                    this.popup.add(ErrorPopup, {
                        title: _t("Online payment unavailable"),
                        body: _t("There is a problem with the server. The order online payment status cannot be retrieved."),
                    });
                    return false;
                }
                if (!lastOrderServerOPData.is_paid) {
                    if (lastOrderServerOPData.modified_payment_lines) {
                        this.cancelOnlinePayment(this.currentOrder);
                        this.showModifiedOnlinePaymentsPopup();
                        return false;
                    }
                    if ((prevOnlinePaymentLine && prevOnlinePaymentLine.get_payment_status() !== "done") || !this.checkRemainingOnlinePaymentLines(lastOrderServerOPData.amount_unpaid)) {
                        this.cancelOnlinePayment(this.currentOrder);
                        return false;
                    }

                    onlinePaymentLine.set_payment_status("waiting");
                    this.currentOrder.select_paymentline(onlinePaymentLine);
                    lastOrderServerOPData = await this.showOnlinePaymentQrCode(qrCodeImgSrc, onlinePaymentLineAmount);
                    if (onlinePaymentLine.get_payment_status() === "waiting") {
                        onlinePaymentLine.set_payment_status(undefined);
                    }
                    prevOnlinePaymentLine = onlinePaymentLine;
                }
            }

            if (!lastOrderServerOPData || !lastOrderServerOPData.is_paid) {
                lastOrderServerOPData = await this.currentOrder.update_online_payments_data_with_server(this.pos.orm, 0);
            }
            if (!lastOrderServerOPData || !lastOrderServerOPData.is_paid) {
                return false;
            }

            await this.afterPaidOrderSavedOnServer(lastOrderServerOPData.paid_order);
            return false; // Cancel normal flow because the current order is already saved on the server.
        } else if (this.currentOrder.server_id) {
            const orderServerOPData = await this.currentOrder.update_online_payments_data_with_server(this.pos.orm, 0);
            if (!orderServerOPData) {
                const { confirmed } = await this.popup.add(ConfirmPopup, {
                    title: _t("Online payment unavailable"),
                    body: _t("There is a problem with the server. The order online payment status cannot be retrieved. Are you sure there is no online payment for this order ?"),
                    confirmText: _t("Yes"),
                });
                return confirmed;
            }
            if (orderServerOPData.is_paid) {
                await this.afterPaidOrderSavedOnServer(orderServerOPData.paid_order);
                return false; // Cancel normal flow because the current order is already saved on the server.
            }
            if (orderServerOPData.modified_payment_lines) {
                this.showModifiedOnlinePaymentsPopup();
                return false;
            }
        }

        return true;
    },
    cancelOnlinePayment(order) {
        // Remove the draft order from the server if there is no done online payment
        order.update_online_payments_data_with_server(this.pos.orm, 0);
    },
    showSaveOrderOnServerErrorPopup() {
        this.popup.add(ErrorPopup, {
            title: _t("Online payment unavailable"),
            body: _t("There is a problem with the server. The order cannot be saved and therefore the online payment is not possible."),
        });
    },
    showModifiedOnlinePaymentsPopup() {
        this.popup.add(ErrorPopup, {
            title: _t("Updated online payments"),
            body: _t("There are online payments that were missing in your view."),
        });
    },
    async showOnlinePaymentQrCode(qrCodeData, amount) {
        if (!this.currentOrder.uiState.PaymentScreen) {
            this.currentOrder.uiState.PaymentScreen = {};
        }
        this.currentOrder.uiState.PaymentScreen.onlinePaymentData = {
            amount: amount,
            qrCode: qrCodeData,
            order: this.currentOrder,
        };

        const { confirmed, payload: orderServerOPData } = await this.popup.add(OnlinePaymentPopup, this.currentOrder.uiState.PaymentScreen.onlinePaymentData);

        if (this.currentOrder.uiState.PaymentScreen) {
            delete this.currentOrder.uiState.PaymentScreen.onlinePaymentData;
            if (Object.keys(this.currentOrder.uiState.PaymentScreen).length === 0) {
                delete this.currentOrder.uiState.PaymentScreen;
            }
        }

        return confirmed ? orderServerOPData : null;
    },
    async afterPaidOrderSavedOnServer(orderJSON) {
        if (!orderJSON) {
            this.popup.add(ErrorPopup, {
                title: _t("Server error"),
                body: _t("The saved order could not be retrieved."),
            });
            return;
        }

        // Update the local order with the data from the server, because it's the server
        // that is responsible for saving the final state of an order when there is an
        // online payment in it.
        // This prevents the case where the cashier changes the payment lines after the
        // order is paid with an online payment and the server saves the order as paid.
        // Without that update, the payment lines printed on the receipt ticket would
        // be invalid.
        const isInvoiceRequested = this.currentOrder.is_to_invoice();
        const orderJSONInArray = [orderJSON];
        await this.pos._loadMissingProducts(orderJSONInArray);
        await this.pos._loadMissingPartners(orderJSONInArray);
        const updatedOrder = this.pos.createReactiveOrder(orderJSON);
        if (!updatedOrder || this.currentOrder.server_id !== updatedOrder.backendId) {
            this.popup.add(ErrorPopup, {
                title: _t("Order saving issue"),
                body: _t("The order has not been saved correctly on the server."),
            });
            return;
        }
        this.pos.orders.add(updatedOrder);
        const oldLocalOrder = this.currentOrder;
        this.pos.set_order(updatedOrder);
        this.pos.removeOrder(oldLocalOrder, false);
        this.pos.validated_orders_name_server_id_map[this.currentOrder.name] = this.currentOrder.id;

        // Now, do practically the normal flow
        if ((this.currentOrder.is_paid_with_cash() || this.currentOrder.get_change()) && this.pos.config.iface_cashdrawer) {
            this.hardwareProxy.printer.openCashbox();
        }

        this.currentOrder.finalized = true;

        if (isInvoiceRequested) {
            if (!this.currentOrder.account_move) {
                this.popup.add(ErrorPopup, {
                    title: _t("Invoice could not be generated"),
                    body: _t("The invoice could not be generated."),
                });
            } else {
                await this.report.doAction("account.account_invoices", [
                    this.currentOrder.account_move,
                ]);
            }
        }

        await this.postPushOrderResolve([this.currentOrder.server_id]);

        this.afterOrderValidation(true);
    },
});

```

## File: static\src\app\store\models.js

```javascript
/** @odoo-module */
import { patch } from "@web/core/utils/patch";
import { Order, Payment } from "@point_of_sale/app/store/models";
import { floatIsZero } from "@web/core/utils/numbers";

patch(Order.prototype, {
    async update_online_payments_data_with_server(orm, next_online_payment_amount) {
        if (!this.server_id) {
            return false;
        }
        try {
            const opData = await orm.call("pos.order", "get_and_set_online_payments_data", [this.server_id, next_online_payment_amount]);
            return this.process_online_payments_data_from_server(opData);
        } catch (ex) {
            console.error("update_online_payments_data_with_server failed: ", ex);
            return null;
        }
    },
    async process_online_payments_data_from_server(opData) {
        if (!opData) {
            return false;
        }
        if (opData.id !== this.server_id) {
            console.error("Called process_online_payments_data_from_server on the wrong order.");
        }

        if ("paid_order" in opData) {
            opData.is_paid = true;
            this.uiState.PaymentScreen?.onlinePaymentPopup?.setReceivedOrderServerOPData(opData);
            return opData;
        } else {
            opData.is_paid = false;
        }

        if ("deleted" in opData && opData["deleted"]) {
            // The current order was previously saved on the server in the draft state, and has been deleted.
            this.server_id = false;
        }

        let newDoneOnlinePayment = false;

        const opLinesToUpdate = this.paymentlines.filter(
            (line) => line.payment_method.is_online_payment && ["waiting", "done"].includes(line.get_payment_status())
        );
        for (const op of opData.online_payments) {
            const matchingLineIndex = opLinesToUpdate.findIndex(
                (pl) => pl.payment_method.id === op.payment_method_id && floatIsZero(pl.amount - op.amount, this.pos.currency.decimal_places)
            );
            let opLine = null;
            if (matchingLineIndex > -1) {
                opLine = opLinesToUpdate[matchingLineIndex];

                opLinesToUpdate.splice(matchingLineIndex, 1);
            }
            if (!opLine) {
                opLine = new Payment(
                    {},
                    {
                        order: this,
                        payment_method: this.pos.payment_methods_by_id[op.payment_method_id],
                        pos: this.pos,
                    }
                );
                this.paymentlines.add(opLine);
                opData['modified_payment_lines'] = true;
            }
            opLine.set_amount(op.amount);
            opLine.can_be_reversed = false;
            if (opLine.get_payment_status() !== "done") {
                newDoneOnlinePayment = true;
            }
            opLine.set_payment_status("done");
        }
        for (const missingInServerLine of opLinesToUpdate) {
            if (missingInServerLine.get_payment_status() === "done") {
                this.paymentlines.remove(missingInServerLine);
                opData['modified_payment_lines'] = true;
            }
        }
        if (newDoneOnlinePayment || opData['modified_payment_lines']) {
            this.uiState.PaymentScreen?.onlinePaymentPopup?.cancel();
        }

        return opData;
    },
});

patch(Payment.prototype, {
    //@override
    export_as_JSON() {
        if (this.payment_method.is_online_payment) {
            return null; // It is the role of the server to save the online payment, not the role of the POS session.
        } else {
            return super.export_as_JSON();
        }
    },
    //@override
    canBeAdjusted() {
        if (this.payment_method.is_online_payment) {
            return false;
        } else {
            return super.canBeAdjusted();
        }
    },
});

```

## File: static\src\app\utils\online_payment_popup\online_payment_popup.js

```javascript
/** @odoo-module */

import { AbstractAwaitablePopup } from "@point_of_sale/app/popup/abstract_awaitable_popup";

export class OnlinePaymentPopup extends AbstractAwaitablePopup {
    static template = "pos_online_payment.OnlinePaymentPopup";

    setup() {
        super.setup();
        if (this.props.order.uiState.PaymentScreen) {
            this.props.order.uiState.PaymentScreen.onlinePaymentPopup = this;
        }
    }
    setReceivedOrderServerOPData(opData) {
        this.opData = opData;
        this.confirm();
    }
    async confirm() {
        super.confirm();
        delete this.props.order.uiState.PaymentScreen?.onlinePaymentPopup;
    }
    cancel() {
        super.cancel();
        delete this.props.order.uiState.PaymentScreen?.onlinePaymentPopup;
    }
    async getPayload() {
        return this.opData;
    }
}

```

## File: static\src\app\utils\online_payment_popup\online_payment_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_online_payment.OnlinePaymentPopup">
        <div class="popup online-payment-popup">
            <main class="body">
                <div class="title">
                    Scan to Pay
                </div>
                <div class="instructions">
                    <p>Invite your customer to scan the QR code to pay: </p>
                    <img class="qr-code" t-att-src="props.qrCode" alt="QR Code to pay"/>
                </div>
                <div class="info">
                    <div><span>Amount: </span><span class="amount" t-esc="env.utils.formatCurrency(props.amount)"/></div>
                    <div><span>Order reference: </span><span t-esc="props.order.name"/></div>
                    <div><span>Order id: </span><span t-esc="props.order.server_id"/></div>
                </div>
            </main>
            <footer class="footer modal-footer">
                <div class="button cancel btn btn-lg" t-on-click="cancel">Cancel</div>
            </footer>
        </div>
    </t>
</templates>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="view_account_payment_form_inherit_pos_online_payment" model="ir.ui.view">
        <field name="name">view.account.payment.form.inherit.pos_online_payment</field>
        <field name="model">account.payment</field>
        <field name="inherit_id" ref="account.view_account_payment_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="pos_order_id" invisible="1"/>
                <button name="action_view_pos_order" type="object"
                        class="oe_stat_button" icon="fa-shopping-cart"
                        invisible="not pos_order_id">
                    <field name="pos_order_id" widget="statinfo" string="POS Order"/>
                </button>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\payment_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Display of /pos/pay (the order of the if conditions matters)-->
    <template id="pay">
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment'" />
            <t t-set="additional_title">
                <t t-esc="page_title" />
            </t>
            <t t-set="no_footer" t-value="1"/>
            <t t-set="no_header" t-value="1"/>

            <div class="wrap">
                <div class="d-flex flex-column vh-100">
                    <div class="d-flex justify-content-between p-3 border-bottom border-top bg-light text-center">
                        <span class="order-reference fw-bolder" t-out="reference_prefix"/>
                        <span class="order-id text-muted"> Order ID: <t t-out="pos_order_id"/></span>
                    </div>
                    <div class="d-flex flex-column p-3">
                        <h4 class="d-flex gap-3 align-items-center mb-3 fs-6 small text-uppercase fw-bolder">
                            To Pay
                            <hr class="flex-grow-1 m-0"/>
                        </h4>
                        <span t-if="not currency" class="alert alert-danger m-0">
                            <strong>Error:</strong> The currency is missing or invalid.
                        </span>
                        <span t-elif="not amount" class="alert alert-info m-0">
                            There is nothing to pay.
                        </span>
                        <span t-elif="not payment_methods_sudo and not tokens_sudo" class="alert alert-danger m-0">
                            <strong>No suitable payment method could be found.</strong>
                            <br/>
                            If you believe that it is an error, please contact the website administrator.
                        </span>
                        <span t-else="">
                            <t t-call="pos_online_payment.pay_summary"/>
                            <t t-call="payment.form" />
                        </span>
                    </div>
                    <div class="d-grid p-3" t-if="exit_route">
                        <a role="button" class="btn btn-light btn-lg border py-3" t-att-href="exit_route">
                            Cancel payment
                        </a>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="pay_summary">
        <span id="o_pos_summary_amount"
                t-out="amount"
                t-options="{'widget': 'monetary', 'display_currency': currency}"
                class="fs-1 text-primary fw-bold"
        />
    </template>

    <!-- Display of /pos/pay/confirmation -->
    <template id="pay_confirmation">
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment Confirmation'" />
            <t t-set="additional_title">
                <t t-esc="page_title" />
            </t>
            <t t-set="no_footer" t-value="1"/>
            <t t-set="no_header" t-value="1"/>

            <div class="wrap">
                <div class="container d-flex flex-column vh-100 pb-3">
                    <div class="row flex-grow-1">
                        <div class="col mt-3">
                            <t t-if="state == 'error'">
                                <div class="alert alert-danger">
                                    <strong>Error:</strong> There was a problem during the payment process.
                                </div>
                            </t>
                            <t t-else="">
                                <t t-call="payment.transaction_status"/>
                                <div class="row row-cols-1 row-cols-sm-2">
                                    <t t-if="state == 'success'">
                                        <div class="col fw-bold">Amount</div>
                                        <div class="col mb-2 mb-lg-0 text-start text-sm-end" t-out="amount" t-options="{'widget': 'monetary', 'display_currency': currency}"/>
                                    </t>

                                    <div class="col fw-bold">Order Reference</div>
                                    <div t-out="order_reference" class="col mb-2 mb-lg-0 text-start text-sm-end"/>

                                    <div class="col fw-bold">Transaction Reference</div>
                                    <div t-out="tx_reference" class="col mb-2 mb-lg-0 text-start text-sm-end"/>

                                    <div class="col fw-bold">Order ID</div>
                                    <div t-out="pos_order_id" class="col mb-2 mb-lg-0 text-start text-sm-end"/>
                                </div>
                            </t>
                        </div>
                    </div>
                    <div class="row g-2 row-cols-1 justify-content-between">
                        <div t-if="exit_route" class="col col-sm-auto me-auto">
                            <a role="button" t-att-class="'btn btn-lg w-100 ' + ('btn-primary' if state == 'success' else 'btn-light border')"  t-att-href="exit_route">
                                <t t-esc="'Continue' if state == 'success' else 'Cancel payment'"/>
                            </a>
                        </div>
                        <div t-if="pay_route and state != 'success'" class="col col-sm-auto ms-auto">
                            <a role="button" class="btn btn-primary btn-lg w-100" t-att-href="pay_route"
                                >
                                Try again
                            </a>
                        </div>
                    </div>
                    <div class="row" t-if="state == 'success'">
                        <div class="mb-3 text-muted text-center"> 
                            Processed by 
                            <t t-esc="provider_name"/>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

</odoo>

```

## File: views\payment_transaction_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_transaction_form" model="ir.ui.view">
        <field name="name">payment.transaction.form</field>
        <field name="model">payment.transaction</field>
        <field name="inherit_id" ref="payment.payment_transaction_form"/>
        <field name="arch" type="xml">
            <button name="action_view_refunds" position="before">
                <field name="pos_order_id" invisible="1"/>
                <button name="action_view_pos_order" type="object"
                        class="oe_stat_button" icon="fa-shopping-cart"
                        invisible="not pos_order_id">
                    <field name="pos_order_id" widget="statinfo" string="POS Order"/>
                </button>
            </button>
        </field>
    </record>
</odoo>

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_online_payment" model="ir.ui.view">
        <field name="name">pos.payment.method.form.inherit.pos_online_payment</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//form/sheet" position="before">
                <field name="has_an_online_payment_provider" invisible="1"/>
                <div class="alert alert-danger" role="alert" invisible="not is_online_payment or has_an_online_payment_provider">
                    You have not activated any <bold><a type="action" name="%(payment.action_payment_provider)d" class="alert-link" role="button">payment provider</a></bold> to allow online payments.
                </div>
            </xpath>
            <xpath expr="//form/sheet/group/group/field[@name='split_transactions']" position="before">
                <field name="is_online_payment"/>
            </xpath>
            <xpath expr="//form/sheet/group/group/field[@name='split_transactions']" position="attributes">
                <attribute name="invisible">is_online_payment</attribute>
            </xpath>
            <xpath expr="//form/sheet/group/group/field[@name='journal_id']" position="attributes">
                <attribute name="invisible">is_online_payment</attribute>
                <attribute name="required">not is_online_payment and not split_transactions</attribute>
            </xpath>
            <xpath expr="//form/sheet/group/group/field[@name='receivable_account_id']" position="attributes">
                <attribute name="invisible">is_online_payment or split_transactions</attribute>
            </xpath>
            <xpath expr="//form/sheet/group[@name='Payment methods']/group" position="after">
                <group invisible="not is_online_payment">
                    <div colspan="2">
                        <label for="online_payment_provider_ids"/>
                        <field name="online_payment_provider_ids" widget="many2many_tags" options="{'no_create': True}" placeholder="All available providers" />
                        <button name="%(payment.action_payment_provider)d" icon="fa-arrow-right" type="action" string="Payment Providers" class="btn-link" />
                    </div>
                </group>
            </xpath>
        </field>
    </record>

    <record id="pos_payment_method_view_tree_inherit_pos_online_payment" model="ir.ui.view">
        <field name="name">pos.payment.method.tree.inherit.pos_online_payment</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="decoration-danger">is_online_payment and not has_an_online_payment_provider</attribute>
            </xpath>
            <xpath expr="//tree" position="inside">
                <field name="is_online_payment" optional="hide"/>
                <field name="has_an_online_payment_provider" column_invisible="True"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_pos_payment_form" model="ir.ui.view">
        <field name="name">pos.payment.form</field>
        <field name="model">pos.payment</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_payment_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='transaction_id']" position='after'>
                <field name="online_account_payment_id" readonly="1" invisible="not online_account_payment_id"/>
            </xpath>
        </field>
    </record>
</odoo>

```

