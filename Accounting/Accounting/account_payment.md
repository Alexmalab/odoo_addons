# Odoo Module: account_payment

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Payment - Account',
    'category': 'Accounting/Accounting',
    'summary': 'Account and Payment Link and Portal',
    'version': '1.0',
    'description': """Link Account and Payment and add Portal Payment

Provide tools for account-related payment as well as portal options to
enable payment.

 * UPDATE ME
""",
    'depends': ['payment'],
    'data': [
        'views/account_portal_templates.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.exceptions import AccessError, MissingError, ValidationError
from odoo.fields import Command
from odoo.http import request, route

from odoo.addons.payment.controllers import portal as payment_portal


class PaymentPortal(payment_portal.PaymentPortal):

    @route('/invoice/transaction/<int:invoice_id>', type='json', auth='public')
    def invoice_transaction(self, invoice_id, access_token, **kwargs):
        """ Create a draft transaction and return its processing values.

        :param int invoice_id: The invoice to pay, as an `account.move` id
        :param str access_token: The access token used to authenticate the request
        :param dict kwargs: Locally unused data passed to `_create_transaction`
        :return: The mandatory values for the processing of the transaction
        :rtype: dict
        :raise: ValidationError if the invoice id or the access token is invalid
        """
        # Check the invoice id and the access token
        try:
            invoice_sudo = self._document_check_access('account.move', invoice_id, access_token)
        except MissingError as error:
            raise error
        except AccessError:
            raise ValidationError("The access token is invalid.")

        kwargs['reference_prefix'] = None  # Allow the reference to be computed based on the invoice
        logged_in = not request.env.user._is_public()
        partner = request.env.user.partner_id if logged_in else invoice_sudo.partner_id
        kwargs['partner_id'] = partner.id
        kwargs.pop('custom_create_values', None)  # Don't allow passing arbitrary create values
        tx_sudo = self._create_transaction(
            custom_create_values={'invoice_ids': [Command.set([invoice_id])]}, **kwargs,
        )

        return tx_sudo._get_processing_values()

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request

from odoo.addons.account.controllers import portal
from odoo.addons.payment.controllers.portal import PaymentPortal
from odoo.addons.portal.controllers.portal import _build_url_w_params


class PortalAccount(portal.PortalAccount):

    def _invoice_get_page_view_values(self, invoice, access_token, **kwargs):
        values = super()._invoice_get_page_view_values(invoice, access_token, **kwargs)
        logged_in = not request.env.user._is_public()
        # We set partner_id to the partner id of the current user if logged in, otherwise we set it
        # to the invoice partner id. We do this to ensure that payment tokens are assigned to the
        # correct partner and to avoid linking tokens to the public user.
        partner_sudo = request.env.user.partner_id if logged_in else invoice.partner_id
        invoice_company = invoice.company_id or request.env.company

        acquirers_sudo = request.env['payment.acquirer'].sudo()._get_compatible_acquirers(
            invoice_company.id, partner_sudo.id, currency_id=invoice.currency_id.id
        )  # In sudo mode to read the fields of acquirers and partner (if not logged in)
        tokens = request.env['payment.token'].search(
            [('acquirer_id', 'in', acquirers_sudo.ids), ('partner_id', '=', partner_sudo.id)]
        )  # Tokens are cleared at the end if the user is not logged in

        # Make sure that the partner's company matches the invoice's company.
        if not PaymentPortal._can_partner_pay_in_company(partner_sudo, invoice_company):
            acquirers_sudo = request.env['payment.acquirer'].sudo()
            tokens = request.env['payment.token']

        fees_by_acquirer = {
            acq_sudo: acq_sudo._compute_fees(
                invoice.amount_total, invoice.currency_id, invoice.partner_id.country_id
            ) for acq_sudo in acquirers_sudo.filtered('fees_active')
        }
        values.update({
            'acquirers': acquirers_sudo,
            'tokens': tokens,
            'fees_by_acquirer': fees_by_acquirer,
            'show_tokenize_input': logged_in,  # Prevent public partner from saving payment methods
            'amount': invoice.amount_residual,
            'currency': invoice.currency_id,
            'partner_id': partner_sudo.id,
            'access_token': access_token,
            'transaction_route': f'/invoice/transaction/{invoice.id}/',
            'landing_route': _build_url_w_params(invoice.access_url, {'access_token': access_token})
        })
        if not logged_in:
            # Don't display payment tokens of the invoice partner if the user is not logged in, but
            # inform that logging in will make them available.
            values.update({
                'existing_token': bool(tokens),
                'tokens': request.env['payment.token'],
            })
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal
from . import payment
```

## File: views\account_portal_templates.xml

```xml
<odoo>
    <template id="portal_my_invoices_payment" name="Payment on My Invoices" inherit_id="account.portal_my_invoices">
        <xpath expr="//t[@t-call='portal.portal_table']/thead/tr/th[last()]" position="before">
            <th></th>
        </xpath>
        <xpath expr="//t[@t-foreach='invoices']/tr/td[last()]" position="before">
            <td class="text-center">
                <t t-set="tx_ids" t-value="invoice.transaction_ids.filtered(lambda tx: tx.state in ('pending', 'authorized'))"/>
                <t t-set="pending_manual_txs" t-value="tx_ids.filtered(lambda tx: tx.state == 'pending' and tx.provider in ('none', 'transfer'))"/>
                <a t-if="invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and invoice.amount_total and invoice.move_type == 'out_invoice' and (pending_manual_txs or not tx_ids or invoice.amount_paid &lt; invoice.amount_total)"
                    t-att-href="invoice.get_portal_url(anchor='portal_pay')" title="Pay Now" aria-label="Pay now" class="btn btn-sm btn-primary" role="button">
                    <i class="fa fa-arrow-circle-right"/><span class='d-none d-md-inline'> Pay Now</span>
                </a>
            </td>
        </xpath>
        <xpath expr="//t[@t-foreach='invoices']/tr/td[hasclass('tx_status')]" position="replace">
            <t t-set="last_tx" t-value="invoice.get_portal_last_transaction()"/>
            <td class="tx_status text-center">
                <t t-if="invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and (last_tx.state not in ['pending', 'authorized'] or (last_tx.state == 'pending' and last_tx.provider in ('none', 'transfer')))">
                    <span class="badge badge-pill badge-info"><i class="fa fa-fw fa-clock-o"></i><span class="d-none d-md-inline"> Waiting for Payment</span></span>
                </t>
                <t t-if="invoice.state == 'posted' and last_tx.state == 'authorized'">
                    <span class="badge badge-pill badge-primary"><i class="fa fa-fw fa-check"/><span class="d-none d-md-inline"> Authorized</span></span>
                </t>
                <t t-if="invoice.state == 'posted' and last_tx.state == 'pending' and last_tx.provider not in ('none', 'transfer')">
                  <span class="badge badge-pill badge-warning"><span class="d-none d-md-inline"> Pending</span></span>
                </t>
                <t t-if="invoice.state == 'posted' and invoice.payment_state in ('paid', 'in_payment')">
                    <span class="badge badge-pill badge-success"><i class="fa fa-fw fa-check"></i><span class="d-none d-md-inline"> Paid</span></span>
                </t>
                <t t-if="invoice.state == 'posted' and invoice.payment_state == 'reversed'">
                    <span class="badge badge-pill badge-success"><i class="fa fa-fw fa-check"></i><span class="d-none d-md-inline"> Reversed</span></span>
                </t>
                <t t-if="invoice.state == 'cancel'">
                    <span class="badge badge-pill badge-danger"><i class="fa fa-fw fa-remove"></i><span class="d-none d-md-inline"> Cancelled</span></span>
                </t>
            </td>
        </xpath>
    </template>

    <template id="portal_invoice_payment" name="Invoice Payment">
        <div class="row">
            <div class="modal fade" id="pay_with" role="dialog">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h3 class="modal-title">Pay with</h3>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                        </div>
                        <div class="modal-body">
                            <div t-if="acquirers or tokens" id="payment_method" class="text-left col-md-13">
                                <t t-call="payment.checkout"/>
                            </div>
                            <div t-else="" class="alert alert-warning">
                                <strong>No suitable payment option could be found.</strong><br/>
                                If you believe that it is an error, please contact the website administrator.
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="portal_invoice_page_inherit_payment" name="Payment on My Invoices" inherit_id="account.portal_invoice_page">
        <xpath expr="//t[@t-call='portal.portal_record_sidebar']//div[hasclass('o_download_pdf')]" position="before">
            <t t-set="tx_ids" t-value="invoice.transaction_ids.filtered(lambda tx: tx.state in ('pending', 'authorized'))"/>
            <t t-set="pending_manual_txs" t-value="tx_ids.filtered(lambda tx: tx.state == 'pending' and tx.provider in ('none', 'transfer'))"/>
            <div>
                <a href="#" t-if="invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and invoice.amount_total and invoice.move_type == 'out_invoice' and (pending_manual_txs or not tx_ids or invoice.amount_paid &lt; invoice.amount_total)"

                    class="btn btn-primary btn-block mb-2" data-toggle="modal" data-target="#pay_with">
                    <i class="fa fa-fw fa-arrow-circle-right"/> Pay Now
                </a>
                <div t-if="tx_ids and not pending_manual_txs and not invoice.amount_residual and invoice.payment_state not in ('paid', 'in_payment')" class="alert alert-info py-1 mb-2" >
                    <i class="fa fa-fw fa-check-circle"/> Pending
                </div>
                <div t-if="invoice.payment_state in ('paid', 'in_payment')" class="alert alert-success py-1 mb-2" >
                    <i class="fa fa-fw fa-check-circle"/> Paid
                </div>
            </div>
        </xpath>
        <xpath expr="//div[@id='invoice_content']//div[hasclass('o_portal_html_view')]" position="before">
            <t t-set="tx" t-value="invoice.get_portal_last_transaction()"/>
            <div t-if="invoice.transaction_ids and invoice.amount_total and not success and not error and (invoice.payment_state != 'not_paid' or tx.state in ('pending', 'authorized'))"
                 class="o_account_payment_tx_status"
                 t-att-data-invoice-id="invoice.id">
                <t t-call="payment.transaction_status"/>
            </div>
            <t t-set="tx_ids" t-value="invoice.transaction_ids.filtered(lambda tx: tx.state in ('authorized', 'done'))"/>
            <div t-if="(invoice.amount_residual or not tx_ids) and invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and invoice.amount_total" id="portal_pay">
                <t t-call="account_payment.portal_invoice_payment"/>
            </div>
            <div class="panel-body" t-if="existing_token">
                <div class="offset-lg-3 col-lg-6">
                    <i class="fa fa-info"></i> You have credits card registered, you can log-in to be able to use them.
                </div>
            </div>
        </xpath>
    </template>

    <template id="portal_invoice_error" name="Invoice error display: payment errors"
            inherit_id="account.portal_invoice_error">
        <xpath expr="//t[@name='generic']" position="after">
            <t t-if="error == 'pay_invoice_invalid_doc'">
                There was an error processing your payment: invalid invoice.
            </t>
            <t t-if="error == 'pay_invoice_invalid_token'">
                There was en error processing your payment: invalid credit card ID.
            </t>
            <t t-if="error == 'pay_invoice_tx_fail'">
                There was an error processing your payment: transaction failed.<br />
                <t t-set="tx_id" t-value="invoice.get_portal_last_transaction()"/>
                <t t-if="tx_id and tx_id.state_message">
                    <t t-esc="tx_id.state_message"/>
                </t>
            </t>
            <t t-if="error == 'pay_invoice_tx_token'">
                There was an error processing your payment: issue with credit card ID validation.
            </t>
        </xpath>
    </template>

    <template id="portal_invoice_success" name="Invoice success display: payment success"
            inherit_id="account.portal_invoice_success">
        <xpath expr="//a[hasclass('close')]" position="after">
            <t t-if="success == 'pay_invoice'">
                <t t-set="payment_tx_id" t-value="invoice.get_portal_last_transaction()"/>
                <span t-if='payment_tx_id.acquirer_id.sudo().done_msg' t-out="payment_tx_id.acquirer_id.sudo().done_msg"/>
                <div t-if="payment_tx_id.acquirer_id.sudo().pending_msg and payment_tx_id.provider == 'transfer' and invoice.ref">
                    <b>Communication: </b><span t-esc='invoice.ref'/>
                </div>
            </t>
            <t t-if="success == 'pay_invoice' and invoice.payment_state in ('paid', 'in_payment')">
                Done, your online payment has been successfully processed. Thank you for your order.
            </t>
        </xpath>
    </template>
</odoo>

```

