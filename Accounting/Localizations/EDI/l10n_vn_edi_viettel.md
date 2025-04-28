# Odoo Module: l10n_vn_edi_viettel

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": """Vietnam - E-invoicing""",
    'icon': '/account/static/description/l10n.png',
    "version": "1.0",
    'countries': ['vn'],
    "category": "Accounting/Localizations/EDI",
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/vietnam.html',
    "depends": [
        "l10n_vn",
    ],
    "summary": "E-invoicing using SInvoice by Viettel",
    "description": """
Vietnam - E-invoicing
=====================
Using SInvoice by Viettel
    """,
    "data": [
        'security/ir.model.access.csv',
        "views/account_move_views.xml",
        "views/res_config_settings_views.xml",
        "views/res_partner_views.xml",
        "views/sinvoice_views.xml",
        "wizard/account_move_reversal_view.xml",
        "wizard/l10n_vn_edi_cancellation_request_views.xml",
    ],
    "installable": True,
    "license": "LGPL-3",
}

```

## File: data\neutralize.sql

```sql
-- disable l10n_vn_edi integration
UPDATE res_company
   SET l10n_vn_edi_username = '',
       l10n_vn_edi_password = '';

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import io
import re
import time
import uuid
import zipfile
from datetime import datetime, timedelta

import requests
from requests import RequestException

from odoo import _, api, fields, models
from odoo.exceptions import UserError

SINVOICE_API_URL = 'https://api-vinvoice.viettel.vn/services/einvoiceapplication/api/'
SINVOICE_TIMEOUT = 60  # They recommend between 60 and 90 seconds, but 60s is already quite long.


def _l10n_vn_edi_send_request(method, url, json_data=None, params=None, headers=None, cookies=None):
    """ Send a request to the API based on the given parameters. In case of errors, the error message is returned. """
    try:
        response = requests.request(method, url, json=json_data, params=params, headers=headers, cookies=cookies, timeout=SINVOICE_TIMEOUT)
        resp_json = response.json()
        error = None
        if resp_json.get('code') or resp_json.get('error'):
            data = resp_json.get('data') or resp_json.get('error')
            error = _('Error when contacting SInvoice: %s.', data)
        return resp_json, error
    except (RequestException, ValueError) as err:
        return {}, _('Something went wrong, please try again later: %s', err)


class AccountMove(models.Model):
    _inherit = 'account.move'

    # EDI values
    l10n_vn_edi_invoice_state = fields.Selection(
        string='Sinvoice Status',
        selection=[
            ('ready_to_send', 'Ready to send'),
            ('sent', 'Sent'),
            # Set when we write on the payment status
            ('payment_state_to_update', 'Payment status to update'),
            ('canceled', 'Canceled'),
            ('adjusted', 'Adjusted'),
            ('replaced', 'Replaced'),
        ],
        copy=False,
        compute='_compute_l10n_vn_edi_invoice_state',
        store=True,
        readonly=False,
    )
    # This id is important when sending by batches in order to recognize individual invoices.
    l10n_vn_edi_invoice_transaction_id = fields.Char(
        string='SInvoice Transaction ID',
        help='Technical field to store the transaction ID if needed',
        export_string_translation=False,
        copy=False,
    )
    l10n_vn_edi_invoice_symbol = fields.Many2one(
        string='Invoice Symbol',
        comodel_name='l10n_vn_edi_viettel.sinvoice.symbol',
        compute='_compute_l10n_vn_edi_invoice_symbol',
        readonly=False,
        store=True,
    )
    l10n_vn_edi_invoice_number = fields.Char(
        string='SInvoice Number',
        help='Invoice Number as appearing on SInvoice.',
        copy=False,
        readonly=True,
    )
    l10n_vn_edi_reservation_code = fields.Char(
        string='Secret Code',
        help='Secret code that can be used by a customer to lookup an invoice on SInvoice.',
        copy=False,
        readonly=True,
    )
    l10n_vn_edi_issue_date = fields.Datetime(
        string='Issue Date',
        help='Date of issue of the invoice on the e-invoicing system.',
        copy=False,
        readonly=True,
    )
    l10n_vn_edi_sinvoice_file_id = fields.Many2one(
        comodel_name='ir.attachment',
        compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_file_id', 'l10n_vn_edi_sinvoice_file'),
        depends=['l10n_vn_edi_sinvoice_file'],
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_vn_edi_sinvoice_file = fields.Binary(
        string='SInvoice json File',
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_vn_edi_sinvoice_xml_file_id = fields.Many2one(
        comodel_name='ir.attachment',
        compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_xml_file_id', 'l10n_vn_edi_sinvoice_xml_file'),
        depends=['l10n_vn_edi_sinvoice_xml_file'],
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_vn_edi_sinvoice_xml_file = fields.Binary(
        string='SInvoice xml File',
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_vn_edi_sinvoice_pdf_file_id = fields.Many2one(
        comodel_name='ir.attachment',
        compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_pdf_file_id', 'l10n_vn_edi_sinvoice_pdf_file'),
        depends=['l10n_vn_edi_sinvoice_pdf_file'],
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_vn_edi_sinvoice_pdf_file = fields.Binary(
        string='SInvoice pdf File',
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    # Replacement/Adjustment fields
    l10n_vn_edi_agreement_document_name = fields.Char(
        string='Agreement Name',
        copy=False,
    )
    l10n_vn_edi_agreement_document_date = fields.Datetime(
        string='Agreement Date',
        copy=False,
    )
    l10n_vn_edi_adjustment_type = fields.Selection(
        string='Adjustment type',
        selection=[
            ('1', 'Money adjustment'),
            ('2', 'Information adjustment'),
        ],
        copy=False,
    )
    # Only used in case of replacement invoice.
    l10n_vn_edi_replacement_origin_id = fields.Many2one(
        comodel_name='account.move',
        string='Replacement of',
        copy=False,
        readonly=True,
        check_company=True,
        export_string_translation=False,
    )
    l10n_vn_edi_reversed_entry_invoice_number = fields.Char(
        string='Revered Entry SInvoice Number',  # Need string here to avoid same label warning
        related='reversed_entry_id.l10n_vn_edi_invoice_number',
        export_string_translation=False,
    )

    @api.depends('l10n_vn_edi_invoice_state')
    def _compute_show_reset_to_draft_button(self):
        # EXTEND 'account'
        super()._compute_show_reset_to_draft_button()
        self.filtered(lambda m: m._l10n_vn_need_cancel_request()).show_reset_to_draft_button = False

    @api.depends('l10n_vn_edi_invoice_state')
    def _compute_need_cancel_request(self):
        # EXTEND 'account' to add dependencies
        return super()._compute_need_cancel_request()

    @api.depends('payment_state')
    def _compute_l10n_vn_edi_invoice_state(self):
        """ Automatically set the state to payment_state_to_update when the payment state is updated.

        This is a bit simplistic, as it can be wrongly set (for example, no need to send when going from in_payment to paid)
        But this shouldn't be an issue since the logic to send the update will check if anything need to change.
        """
        for invoice in self:
            if invoice.country_code == 'VN' and invoice.l10n_vn_edi_invoice_state == 'sent':
                invoice.l10n_vn_edi_invoice_state = 'payment_state_to_update'
            else:
                invoice.l10n_vn_edi_invoice_state = invoice.l10n_vn_edi_invoice_state

    @api.depends('company_id', 'partner_id')
    def _compute_l10n_vn_edi_invoice_symbol(self):
        """ Use the property l10n_vn_edi_symbol to set a default invoice symbol. """
        for invoice in self:
            if invoice.country_code == 'VN':
                # Even if there was a value already set, we assume that it should be updated if the partner is changed.
                invoice.l10n_vn_edi_invoice_symbol = invoice.partner_id.l10n_vn_edi_symbol
            else:
                invoice.l10n_vn_edi_invoice_symbol = False

    def button_request_cancel(self):
        # EXTEND 'account'
        if self._l10n_vn_need_cancel_request():
            return {
                'name': _('Invoice Cancellation'),
                'type': 'ir.actions.act_window',
                'view_type': 'form',
                'view_mode': 'form',
                'res_model': 'l10n_vn_edi_viettel.cancellation',
                'target': 'new',
                'context': {'default_invoice_id': self.id},
            }

        return super().button_request_cancel()

    def _get_fields_to_detach(self):
        # EXTENDS account
        fields_list = super()._get_fields_to_detach()
        fields_list.extend(['l10n_vn_edi_sinvoice_file', 'l10n_vn_edi_sinvoice_xml_file','l10n_vn_edi_sinvoice_pdf_file'])
        return fields_list

    def _l10n_vn_edi_fetch_invoice_file_data(self, file_format):
        """ Helper to try fetching a few time in case the files are not yet ready. """
        self.ensure_one()
        files_data, error_message = self._l10n_vn_edi_try_fetch_invoice_file_data(file_format)

        if error_message:
            return '', error_message

        # Sometimes the documents are not available right away. This is quite rare, but I saw it happen a few times.
        # To handle that we will try up to three time to fetch the document => The impact should be negligible.
        threshold = 1
        while not files_data['fileToBytes'] and threshold < 3:
            time.sleep(0.125 * threshold)
            files_data, error_message = self._l10n_vn_edi_try_fetch_invoice_file_data(file_format)
            threshold += 1
        return files_data, error_message

    def _l10n_vn_edi_try_fetch_invoice_file_data(self, file_format):
        """
        Query sinvoice in order to fetch the data representation of the invoice, either zip or pdf.
        """
        self.ensure_one()
        if not self._l10n_vn_edi_is_sent():
            return {}, _("In order to download the invoice's PDF file, you must first send it to SInvoice")

        # == Lock ==
        self.env['res.company']._with_locked_records(self)

        access_token, error = self._l10n_vn_edi_get_access_token()
        if error:
            return {}, error

        return _l10n_vn_edi_send_request(
            method='POST',
            url=f'{SINVOICE_API_URL}InvoiceAPI/InvoiceUtilsWS/getInvoiceRepresentationFile',
            json_data={
                'supplierTaxCode': self.company_id.vat,
                'templateCode': self.l10n_vn_edi_invoice_symbol.invoice_template_id.name,
                'invoiceNo': self.l10n_vn_edi_invoice_number,
                'strIssueDate': self._l10n_vn_edi_format_date(self.l10n_vn_edi_issue_date),
                'transactionUuid': self.l10n_vn_edi_invoice_transaction_id,
                'fileType': file_format,
            },
            cookies={'access_token': access_token},
        )

    def _l10n_vn_edi_fetch_invoice_xml_file_data(self):
        """
        Query sinvoice in order to fetch the xsl and xml data representation of the invoice.

        Returns a list of tuple with both file names, mimetype, content and the field it should be stored in.
        """
        self.ensure_one()
        files_data, error_message = self._l10n_vn_edi_fetch_invoice_file_data('ZIP')
        if error_message:
            return files_data, error_message

        file_bytes = base64.b64decode(files_data['fileToBytes'])

        # For some reason, request_response['fileToBytes'] is a zip file containing the other zip file.
        # The content of the inner zip is a xsl file as well as a xml file.
        # In our case the xsl file is not important, so we can simply ignore it.
        with zipfile.ZipFile(io.BytesIO(file_bytes)) as zip_file:
            inner_zip_bytes = zip_file.read(zip_file.filelist[0])
            with zipfile.ZipFile(io.BytesIO(inner_zip_bytes)) as inner_zip:
                for file in inner_zip.filelist:
                    if file.filename.endswith('.xml'):
                        return {
                            'name': file.filename,
                            'mimetype': 'application/xml',
                            'raw': inner_zip.read(file),
                            'res_field': 'l10n_vn_edi_sinvoice_xml_file',
                        }, ""

    def _l10n_vn_edi_fetch_invoice_pdf_file_data(self):
        """
        Query sinvoice in order to fetch the pdf data representation of the invoice.

        Returns a tuple with the pdf name, mimetype, content and field.
        """
        self.ensure_one()
        file_data, error_message = self._l10n_vn_edi_fetch_invoice_file_data('PDF')
        if error_message:
            return file_data, error_message

        file_bytes = base64.b64decode(file_data['fileToBytes'])

        return {
            'name': file_data['fileName'],
            'mimetype': 'application/pdf',
            'raw': file_bytes,
            'res_field': 'l10n_vn_edi_sinvoice_pdf_file',
        }, ""

    def action_l10n_vn_edi_update_payment_status(self):
        """ Send a request to update the payment status of the invoice. """

        invoices = self.filtered(lambda i: i.l10n_vn_edi_invoice_state == 'payment_state_to_update')
        if not invoices:
            return

        # == Lock ==
        self.env['res.company']._with_locked_records(invoices)

        for invoice in invoices:
            sinvoice_status = 'unpaid'

            # SInvoice will return a NOT_FOUND_DATA error if the status in Odoo matches the one on their side.
            # Because of that we wouldn't be able to differentiate a real issue (invoice on our side not matching theirs)
            # With simply a status already up to date. So we need to check the status first to see if we need to update.
            invoice_lookup, error_message = invoice._l10n_vn_edi_lookup_invoice()
            if error_message:
                raise UserError(error_message)

            if 'result' in invoice_lookup:
                invoice_data = invoice_lookup['result'][0]
                if invoice_data['status'] == 'Chưa thanh toán':  # Vietnamese for 'unpaid'
                    sinvoice_status = 'unpaid'
                else:
                    sinvoice_status = 'paid'

            params = {
                'supplierTaxCode': invoice.company_id.vat,
                'invoiceNo': invoice.l10n_vn_edi_invoice_number,
                'strIssueDate': invoice._l10n_vn_edi_format_date(invoice.l10n_vn_edi_issue_date),
            }

            if invoice.payment_state in {'in_payment', 'paid'} and sinvoice_status == 'unpaid':
                # Mark the invoice as paid
                endpoint = f'{SINVOICE_API_URL}InvoiceAPI/InvoiceWS/updatePaymentStatus'
                params['templateCode'] = invoice.l10n_vn_edi_invoice_symbol.invoice_template_id.name
            elif invoice.payment_state not in {'in_payment', 'paid'} and sinvoice_status == 'paid':
                # Mark the invoice as not paid
                endpoint = f'{SINVOICE_API_URL}InvoiceAPI/InvoiceWS/cancelPaymentStatus'
            else:
                continue

            access_token, error = self._l10n_vn_edi_get_access_token()
            if error:
                raise UserError(error)

            _request_response, error_message = _l10n_vn_edi_send_request(
                method='POST',
                url=endpoint,
                params=params,
                headers={
                    'Content-Type': 'application/x-www-form-urlencoded;',
                },
                cookies={'access_token': access_token},
            )

            if error_message:
                raise UserError(error_message)

            # Revert back to the sent state as the status is up-to-date.
            invoice.l10n_vn_edi_invoice_state = 'sent'

            if self._can_commit():
                self._cr.commit()

    def _l10n_vn_need_cancel_request(self):
        return self._l10n_vn_edi_is_sent() and self.l10n_vn_edi_invoice_state != 'canceled'

    def _need_cancel_request(self):
        # EXTEND 'account'
        return super()._need_cancel_request() or self._l10n_vn_need_cancel_request()

    def _post(self, soft=True):
        # EXTEND 'account'
        posted = super()._post(soft=soft)

        # Ensure to tag the move as 'Ready to send' upon posting if it makes sense.
        posted.filtered(
            lambda invoice:
                invoice.country_code == 'VN'
                and invoice.is_sale_document()
                and not invoice._l10n_vn_edi_is_sent()
        ).l10n_vn_edi_invoice_state = 'ready_to_send'

        return posted

    # -------------------------------------------------------------------------
    # API METHODS
    # -------------------------------------------------------------------------

    def _l10n_vn_edi_check_invoice_configuration(self):
        """ Some checks that are used to avoid common errors before sending the invoice. """
        self.ensure_one()
        company = self.company_id
        commercial_partner = self.commercial_partner_id
        errors = []
        if not company.l10n_vn_edi_username or not company.l10n_vn_edi_password:
            errors.append(_('Sinvoice credentials are missing on company %s.', company.display_name))
        if not company.vat:
            errors.append(_('VAT number is missing on company %s.', company.display_name))
        company_phone = company.phone and self._l10n_vn_edi_format_phone_number(company.phone)
        if company_phone and not company_phone.isdecimal():
            errors.append(_('Phone number for company %s must only contain digits or +.', company.display_name))
        commercial_partner_phone = commercial_partner.phone and self._l10n_vn_edi_format_phone_number(commercial_partner.phone)
        if commercial_partner_phone and not commercial_partner_phone.isdecimal():
            errors.append(_('Phone number for partner %s must only contain digits or +.', commercial_partner.display_name))
        if not self.l10n_vn_edi_invoice_symbol:
            errors.append(_('The invoice symbol must be provided.'))
        if self.l10n_vn_edi_invoice_symbol and not self.l10n_vn_edi_invoice_symbol.invoice_template_id:
            errors.append(_("The invoice symbol's template must be provided."))
        if self.move_type == 'out_refund' and (not self.reversed_entry_id or not self.reversed_entry_id._l10n_vn_edi_is_sent()):
            errors.append(_('You can only send a credit note linked to a previously sent invoice.'))
        if not company.street or not company.state_id or not company.country_id:
            errors.append(_('The street, state and country of company %s must be provided.', company.display_name))
        if self.company_currency_id.name != 'VND':
            vnd = self.env.ref('base.VND')
            rate = vnd.with_context(date=self.invoice_date or self.date).rate
            if not vnd.active or rate == 1:
                errors.append(_('Please make sure that the VND currency is enabled, and that the exchange rates are set.'))
        return errors

    def _l10n_vn_edi_send_invoice(self, invoice_json_data):
        """ Send an invoice to the SInvoice system.

        Handles lookup on the system in order to ensure that the invoice was not sent successfully yet in case of
        timeout or other unforeseen error.
        """
        self.ensure_one()

        # == Lock ==
        self.env['res.company']._with_locked_records(self)

        invoice_data = {}
        # If the request was sent but ended up failing, there is still the possibility that the invoice was saved
        # on their system (timeout, for example)
        if self.l10n_vn_edi_invoice_transaction_id:
            invoice_lookup, error_message = self._l10n_vn_edi_lookup_invoice()
            if 'result' in invoice_lookup:
                invoice_data = invoice_lookup['result'][0]
            # note: We do not catch errors on this endpoint for simplicity, as it should not be required.
        else:
            # We do not store the transaction id on the move right away so that we can avoid the above api call.
            # When sending for the first time, we'll get the id from the file data, which we generated earlier in the flow.
            self.l10n_vn_edi_invoice_transaction_id = invoice_json_data['generalInvoiceInfo']['transactionUuid']

        # if the above request did not return data, we can assume that the invoice has failed to be created, or was never sent
        if not invoice_data:
            # Send the invoice to the system
            access_token, error = self._l10n_vn_edi_get_access_token()
            if error:
                return [error]

            request_response, error_message = _l10n_vn_edi_send_request(
                method='POST',
                url=f'{SINVOICE_API_URL}InvoiceAPI/InvoiceWS/createInvoice/{self.company_id.vat}',
                json_data=invoice_json_data,
                cookies={'access_token': access_token},
            )

            if error_message:
                return [error_message]

            invoice_data = request_response['result']

        self.write({
            'l10n_vn_edi_reservation_code': invoice_data['reservationCode'],
            'l10n_vn_edi_invoice_number': invoice_data['invoiceNo'],
            'l10n_vn_edi_invoice_state': 'sent',
        })

        if self._can_commit():
            self._cr.commit()

    def _l10n_vn_edi_cancel_invoice(self, reason, agreement_document_name, agreement_document_date):
        """ Send a request to cancel the invoice. """
        self.ensure_one()

        # == Lock ==
        self.env['res.company']._with_locked_records(self)

        # If no error raised, we try to cancel it on the EDI.
        access_token, error = self._l10n_vn_edi_get_access_token()
        if error:
            raise UserError(error)

        _request_response, error_message = _l10n_vn_edi_send_request(
            method='POST',
            url=f'{SINVOICE_API_URL}InvoiceAPI/InvoiceWS/cancelTransactionInvoice',
            params={
                'supplierTaxCode': self.company_id.vat,
                'templateCode': self.l10n_vn_edi_invoice_symbol.invoice_template_id.name,
                'invoiceNo': self.l10n_vn_edi_invoice_number,
                'strIssueDate': self._l10n_vn_edi_format_date(self.l10n_vn_edi_issue_date),
                'additionalReferenceDesc': agreement_document_name,
                'additionalReferenceDate': self._l10n_vn_edi_format_date(agreement_document_date),
                'reasonDelete': reason,
            },
            headers={
                'Content-Type': 'application/x-www-form-urlencoded;',
            },
            cookies={'access_token': access_token},
        )

        if error_message:
            raise UserError(error_message)

        self.l10n_vn_edi_invoice_state = 'canceled'

        try:
            self._check_fiscal_lock_dates()
            self.line_ids._check_tax_lock_date()

            self.button_cancel()

            self.with_context(no_new_invoice=True).message_post(
                body=_('The invoice has been canceled for reason: %(reason)s', reason=reason),
            )
        except UserError as e:
            self.with_context(no_new_invoice=True).message_post(
                body=_('The invoice has been canceled on sinvoice for reason: %(reason)s'
                       'But the cancellation in Odoo failed with error: %(error)s', reason=reason, error=e),
            )

        if self._can_commit():
            self._cr.commit()

    def button_draft(self):
        # EXTEND account
        # When going from canceled => draft, we ensure to clear the edi fields so that the invoice can be resent if required.
        cancelled_sinvoices = self.filtered(
            lambda i: i.country_code == 'VN' and i.l10n_vn_edi_invoice_state == 'canceled' and i.state == 'cancel'
        )
        res = super().button_draft()
        cancelled_sinvoices.write({
            'l10n_vn_edi_invoice_transaction_id': False,
            'l10n_vn_edi_invoice_number': False,
            'l10n_vn_edi_reservation_code': False,
            'l10n_vn_edi_issue_date': False,
            'l10n_vn_edi_invoice_state': False,
        })
        # Cleanup the files as well. They will still be available in the chatter.
        cancelled_sinvoices.l10n_vn_edi_sinvoice_xml_file_id.unlink()
        cancelled_sinvoices.l10n_vn_edi_sinvoice_pdf_file_id.unlink()
        cancelled_sinvoices.l10n_vn_edi_sinvoice_file_id.unlink()
        return res

    def _l10n_vn_edi_generate_invoice_json(self):
        """ Return the dict of data that will be sent to the api in order to create the invoice. """
        # We leave the summarized information computation to SInvoice.
        self.ensure_one()
        # This MUST match chronologically with the sequence they generate on their system, which is why it is set to now.
        self.l10n_vn_edi_issue_date = fields.Datetime.now()
        json_values = {}
        self._l10n_vn_edi_add_general_invoice_information(json_values)
        self._l10n_vn_edi_add_buyer_information(json_values)
        self._l10n_vn_edi_add_seller_information(json_values)
        self._l10n_vn_edi_add_payment_information(json_values)
        self._l10n_vn_edi_add_item_information(json_values)
        self._l10n_vn_edi_add_tax_breakdowns(json_values)
        return json_values

    def _l10n_vn_edi_add_general_invoice_information(self, json_values):
        """ General invoice information, such as the model number, invoice symbol, type, date of issues, ... """
        self.ensure_one()
        invoice_data = {
            'transactionUuid': str(uuid.uuid4()),
            'invoiceType': self.l10n_vn_edi_invoice_symbol.invoice_template_id.template_invoice_type,
            'templateCode': self.l10n_vn_edi_invoice_symbol.invoice_template_id.name,
            'invoiceSeries': self.l10n_vn_edi_invoice_symbol.name,
            # This timestamp is important as it is used to check the chronological order of Invoice Numbers.
            # Since this xml is generated upon posting, just like the invoice number, using now() should keep that order
            # correct in most case.
            'invoiceIssuedDate': self._l10n_vn_edi_format_date(self.l10n_vn_edi_issue_date),
            'currencyCode': self.currency_id.name,
            'adjustmentType': '1',  # 1 for original invoice, which is the case during first issuance.
            'paymentStatus': self.payment_state in {'in_payment', 'paid'},
            'cusGetInvoiceRight': True,  # Set to true, allowing the customer to see the invoice.
            'validation': 1,  # Set to 1, SInvoice will validate tax information while processing the invoice.
        }

        # When invoicing in a foreign currency, we need to provide the rate, or it will default to 1.
        if self.currency_id.name != 'VND':
            invoice_data['exchangeRate'] = self.env['res.currency']._get_conversion_rate(
                from_currency=self.currency_id,
                to_currency=self.env.ref('base.VND'),
                company=self.company_id,
                date=self.invoice_date or self.date,
            )

        adjustment_origin_invoice = None
        if self.move_type == 'out_refund':  # Credit note are used to adjust an existing invoice
            adjustment_origin_invoice = self.reversed_entry_id
        elif self.l10n_vn_edi_replacement_origin_id:  # 'Reverse and create invoice' is used to issue a replacement invoice
            adjustment_origin_invoice = self.l10n_vn_edi_replacement_origin_id

        if adjustment_origin_invoice:
            invoice_data.update({
                'adjustmentType': '5' if self.move_type == 'out_refund' else '3',  # Adjustment or replacement
                'adjustmentInvoiceType': self.l10n_vn_edi_adjustment_type or '',
                'originalInvoiceId': adjustment_origin_invoice.l10n_vn_edi_invoice_number,
                'originalInvoiceIssueDate': self._l10n_vn_edi_format_date(adjustment_origin_invoice.l10n_vn_edi_issue_date),
                'originalTemplateCode': adjustment_origin_invoice.l10n_vn_edi_invoice_symbol.invoice_template_id.name,
                'additionalReferenceDesc': self.l10n_vn_edi_agreement_document_name,
                'additionalReferenceDate': self._l10n_vn_edi_format_date(self.l10n_vn_edi_agreement_document_date),
            })

        json_values['generalInvoiceInfo'] = invoice_data

    def _l10n_vn_edi_add_buyer_information(self, json_values):
        """ Create and return the buyer information for the current invoice. """
        self.ensure_one()

        commercial_partner_phone = self.commercial_partner_id.phone and self._l10n_vn_edi_format_phone_number(self.commercial_partner_id.phone)
        buyer_information = {
            'buyerName': self.partner_id.name,
            'buyerLegalName': self.commercial_partner_id.name,
            'buyerTaxCode': self.commercial_partner_id.vat or '',
            'buyerAddressLine': self.partner_id.street,
            'buyerPhoneNumber': commercial_partner_phone or '',
            'buyerEmail': self.commercial_partner_id.email or '',
            'buyerDistrictName': self.partner_id.state_id.name,
            'buyerCityName': self.partner_id.city,
            'buyerCountryCode': self.partner_id.country_id.code,
            'buyerNotGetInvoice': 0,  # Set to 1 to no send the invoice to the buyer.
        }

        if self.partner_bank_id:
            buyer_information.update({
                'buyerBankName': self.partner_bank_id.bank_name,
                'buyerBankAccount': self.partner_bank_id.acc_number,
            })

        json_values['buyerInfo'] = buyer_information

    def _l10n_vn_edi_add_seller_information(self, json_values):
        """ Create and return the seller information for the current invoice. """
        self.ensure_one()
        company_phone = self.company_id.phone and self._l10n_vn_edi_format_phone_number(self.company_id.phone)
        seller_information = {
            'sellerLegalName': self.company_id.name,
            'sellerTaxCode': self.company_id.vat,
            'sellerAddressLine': self.company_id.street,
            'sellerPhoneNumber': company_phone or '',
            'sellerEmail': self.company_id.email,
            'sellerDistrictName': self.company_id.state_id.name,
            'sellerCountryCode': self.company_id.country_id.code,
            'sellerWebsite': self.company_id.website,
        }

        if self.partner_bank_id:
            seller_information.update({
                'sellerBankName': self.partner_bank_id.bank_name,
                'sellerBankAccount': self.partner_bank_id.acc_number,
            })

            if self.partner_bank_id.proxy_type == 'merchant_id':
                seller_information.update({
                    'merchantCode': self.partner_bank_id.proxy_value,
                    'merchantName': self.company_id.name,
                    'merchantCity': self.company_id.city,
                })

        json_values['sellerInfo'] = seller_information

    def _l10n_vn_edi_add_payment_information(self, json_values):
        """ Create and return the payment information for the current invoice. Not fully supported. """
        self.ensure_one()
        json_values['payments'] = [{
            # We need to provide a value but when we send the invoice, we may not have this information.
            # According to VN laws, if the payment method has not been determined, we can fill in TM/CK.
            # TM is for bank transfer, CK is for cash payment.
            'paymentMethodName': 'TM/CK',
        }]

    def _l10n_vn_edi_add_item_information(self, json_values):
        """ Create and return the items information for the current invoice. """
        self.ensure_one()
        items_information = []
        code_map = {
            'product': 1,
            'line_note': 2,
            'discount': 3,
        }
        for line in self.invoice_line_ids.filtered(lambda ln: ln.display_type == 'product'):
            # For credit notes amount, we send negative values (reduces the amount of the original invoice)
            sign = 1 if self.move_type == 'out_invoice' else -1
            item_information = {
                'itemCode': line.product_id.code,
                'itemName': line.product_id.name,
                'unitName': line.product_uom_id.name,
                'unitPrice': line.price_unit * sign,
                'quantity': line.quantity,
                # This amount should be without discount applied.
                'itemTotalAmountWithoutTax': line.currency_id.round(line.price_unit * line.quantity) * sign,
                # In Vietnam a line will always have only one tax.
                # Values are either: -2 (no tax), -1 (not declaring/paying taxes), 0,5,8,10 (the tax %)
                # Most use cases will be -2 or a tax percentage, so we limit the support to these.
                'taxPercentage': line.tax_ids and line.tax_ids[0].amount or -2,
                'taxAmount': (line.price_total - line.price_subtotal) * sign,
                'discount': line.discount,
                'itemTotalAmountAfterDiscount': line.price_subtotal * sign,
                'itemTotalAmountWithTax': line.price_total * sign,
            }
            if line.display_type in code_map:
                item_information['selection'] = code_map[line.display_type]
            if line.display_type == 'discount':
                item_information['isIncreaseItem'] = False
            if self.move_type == 'out_refund':
                item_information.update({
                    'adjustmentTaxAmount': item_information['taxAmount'],
                    'isIncreaseItem': False,
                })
            items_information.append(item_information)

        json_values['itemInfo'] = items_information

    def _l10n_vn_edi_add_tax_breakdowns(self, json_values):
        """ Create and return the tax breakdown of the current invoice. """
        self.ensure_one()

        def grouping_key_generator(base_line, tax_data):
            # Requirement is to generate a tax breakdown per taxPercentage
            return {'tax_percentage': tax_data['tax'].amount or -2}

        tax_breakdowns = []

        tax_details_grouped = self._prepare_invoice_aggregated_taxes(grouping_key_generator=grouping_key_generator)
        for tax_percentage, tax_percentage_values in tax_details_grouped['tax_details'].items():
            tax_breakdowns.append({
                'taxPercentage': tax_percentage['tax_percentage'],
                'taxableAmount': tax_percentage_values['base_amount_currency'],
                'taxAmount': tax_percentage_values['tax_amount_currency'],
                'taxableAmountPos': self.move_type == 'out_invoice',  # For adjustment invoice, the amount should be considered as negative.
                'taxAmountPos': self.move_type == 'out_invoice',  # Same
            })

        json_values['taxBreakdowns'] = tax_breakdowns

    def _l10n_vn_edi_lookup_invoice(self):
        """ Lookup on invoice, returning its current details on SInvoice. """
        self.ensure_one()
        access_token, error = self._l10n_vn_edi_get_access_token()
        if error:
            return {}, error

        invoice_data, error_message = _l10n_vn_edi_send_request(
            method='POST',
            url=f'{SINVOICE_API_URL}InvoiceAPI/InvoiceWS/searchInvoiceByTransactionUuid',
            params={
                'supplierTaxCode': self.company_id.vat,
                'transactionUuid': self.l10n_vn_edi_invoice_transaction_id,
            },
            headers={
                'Content-Type': 'application/x-www-form-urlencoded;',
            },
            cookies={'access_token': access_token},
        )
        return invoice_data, error_message

    def _l10n_vn_edi_get_access_token(self):
        """ Return an access token to be used to contact the API. Either take a valid stored one or get a new one. """
        self.ensure_one()
        credentials_company = self._l10n_vn_edi_get_credentials_company()
        # First, check if we have a token stored and if it is still valid.
        if credentials_company.l10n_vn_edi_token and credentials_company.l10n_vn_edi_token_expiry > datetime.now():
            return credentials_company.l10n_vn_edi_token, ""

        data = {'username': credentials_company.l10n_vn_edi_username, 'password': credentials_company.l10n_vn_edi_password}
        request_response, error_message = _l10n_vn_edi_send_request(
            method='POST',
            url='https://api-vinvoice.viettel.vn/auth/login',  # This one is special and uses another base address.
            json_data=data
        )
        if error_message:
            return "", error_message
        if 'access_token' not in request_response:  # Just in case something else go wrong and it's missing the token
            return "", _('Connection to the API failed, please try again later.')

        access_token = request_response['access_token']

        try:
            access_token_expiry = datetime.now() + timedelta(seconds=int(request_response['expires_in']))
        except ValueError:  # Simple security measure in case we don't get the expected format in the response.
            return "", _('Error while parsing API answer. Please try again later.')

        # Tokens are valid for 5 minutes. Storing it helps reduce api calls and speed up things a little bit.
        credentials_company.write({
            'l10n_vn_edi_token': access_token,
            'l10n_vn_edi_token_expiry': access_token_expiry,
        })

        return request_response['access_token'], ""

    def _l10n_vn_edi_get_credentials_company(self):
        """ The company holding the credentials could be one of the parent companies.
        We need to ensure that:
            - We use the credentials of the parent company, if no credentials are set on the child one.
            - We store the access token on the appropriate company, based on which holds the credentials.
        """
        if self.company_id.l10n_vn_edi_username and self.company_id.l10n_vn_edi_password:
            return self.company_id

        return self.company_id.sudo().parent_ids.filtered(
            lambda c: c.l10n_vn_edi_username and c.l10n_vn_edi_password
        )[-1:]

    # -------------------------------------------------------------------------
    # HELPERS
    # -------------------------------------------------------------------------

    @api.model
    def _l10n_vn_edi_format_date(self, date):
        """
        All APIs for Sinvoice uses the same time format, being the current hour, minutes and seconds converted into
        seconds since unix epoch, but formatting like milliseconds since unix epoch.
        It means that the time will end in 000 for the milliseconds as they are not as of today used by the system.
        """
        return int(date.timestamp()) * 1000 if date else 0

    @api.model
    def _l10n_vn_edi_format_phone_number(self, number):
        """
        Simple helper that takes in a phone number and try to format it to fit sinvoice format.
        SInvoice only allows digits, so we will remove any (, ), -, + characters.
        """
        # We first replace + by 00, then we remove all non digit characters.
        number = number.replace('+', '00')
        return re.sub(r'[^0-9]+', '', number)

    def _l10n_vn_edi_is_sent(self):
        """ Small helper that returns true if self has been sent to sinvoice. """
        self.ensure_one()
        sent_statuses = {'sent', 'payment_state_to_update', 'canceled', 'adjusted', 'replaced'}
        return self.l10n_vn_edi_invoice_state in sent_statuses

    def _get_mail_thread_data_attachments(self):
        res = super()._get_mail_thread_data_attachments()
        # attachments with 'res_field' are excluded, and we want this in the chatter for audit/... purposes.
        return res | self.l10n_vn_edi_sinvoice_file_id

```

## File: models\account_move_send.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json

from odoo import SUPERUSER_ID, _, api, models


class AccountMoveSend(models.AbstractModel):
    _inherit = 'account.move.send'

    @api.model
    def _is_vn_edi_applicable(self, move):
        return bool(move.l10n_vn_edi_invoice_state == 'ready_to_send' and move._l10n_vn_edi_get_credentials_company())

    def _get_all_extra_edis(self) -> dict:
        # EXTENDS 'account'
        res = super()._get_all_extra_edis()
        res.update({'vn_sinvoice_send': {'label': _("Send to SInvoice"), 'is_applicable': self._is_vn_edi_applicable}})
        return res

    # -------------------------------------------------------------------------
    # ATTACHMENTS
    # -------------------------------------------------------------------------

    @api.model
    def _get_invoice_extra_attachments(self, move):
        # EXTENDS 'account'
        # we require these to be downloadable for a better UX. It was also said that the xml and pdf files are
        # important files that needs to be shared with the customer.
        return (
            super()._get_invoice_extra_attachments(move)
            + move.l10n_vn_edi_sinvoice_xml_file_id
            + move.l10n_vn_edi_sinvoice_pdf_file_id
        )

    def _get_placeholder_mail_attachments_data(self, move, invoice_edi_format=None, extra_edis=None):
        # EXTENDS 'account'
        results = super()._get_placeholder_mail_attachments_data(move, invoice_edi_format=invoice_edi_format, extra_edis=extra_edis)
        if invoice_edi_format == 'vn_sinvoice' and move._l10n_vn_edi_get_credentials_company():
            results.extend([{
                'id': 'placeholder_sinvoice.pdf',
                'name': f'{move.company_id.vat}-{move.l10n_vn_edi_invoice_symbol.name}101.pdf',
                'mimetype': 'application/pdf',
                'placeholder': True,
            }, {
                'id': 'placeholder_sinvoice.xml',
                'name': f'{move.company_id.vat}-{move.l10n_vn_edi_invoice_symbol.name}101.xml',
                'mimetype': 'application/xml',
                'placeholder': True,
            }])

        return results

    # -------------------------------------------------------------------------
    # SENDING METHODS
    # -------------------------------------------------------------------------

    @api.model
    def _generate_sinvoice_file_date(self, invoice, invoice_data):
        # Ensure that we still generate the file if 'generate' is ul10n_vn_edi_invoice_transaction_id-checked but send it.
        need_file = (
            (invoice_data['invoice_edi_format'] == 'vn_sinvoice' and invoice._l10n_vn_edi_get_credentials_company())
            or 'vn_sinvoice_send' in invoice_data['extra_edis']
        )
        # In case we already have a json file existing on the invoice, we skip regenerating it.
        if need_file and not invoice.l10n_vn_edi_sinvoice_file:
            errors = invoice._l10n_vn_edi_check_invoice_configuration()
            if not errors:
                json_data = invoice._l10n_vn_edi_generate_invoice_json()
                invoice_data['sinvoice_attachments'] = [{
                    'name': f'{invoice.name.replace("/", "_")}_sinvoice.json',
                    'raw': json.dumps(json_data, ensure_ascii=False).encode('utf8'),
                    'mimetype': 'application/json',
                    'res_model': invoice._name,
                    'res_id': invoice.id,
                    'res_field': 'l10n_vn_edi_sinvoice_file',  # Binary field
                }]
            else:
                invoice_data['error'] = {
                    'error_title': _('Error when generating SInvoice file.'),
                    'errors': errors,
                }

    def _hook_invoice_document_before_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_before_pdf_report_render(invoice, invoice_data)
        self._generate_sinvoice_file_date(invoice, invoice_data)

    def _call_web_service_before_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_before_invoice_pdf_render(invoices_data)
        for invoice, invoice_data in invoices_data.items():
            if 'vn_sinvoice_send' in invoice_data['extra_edis']:
                errors = invoice._l10n_vn_edi_check_invoice_configuration()
                if not errors:
                    if 'sinvoice_attachments' in invoice_data:
                        json_data = json.loads(invoice_data['sinvoice_attachments'][0]['raw'].decode('utf-8'))
                    # If the invoice was downloaded but not sent, the json file could already be there.
                    elif invoice.l10n_vn_edi_sinvoice_file:
                        json_data = json.loads(base64.b64decode(invoice.l10n_vn_edi_sinvoice_file).decode('utf-8'))
                    # If we don't have the file data and the file, we will regenerate it.
                    else:
                        self._generate_sinvoice_file_date(invoice, invoice_data)
                        # In case the above call ended in an error, we skip setting json_data
                        if 'sinvoice_attachments' not in invoice_data:
                            continue
                        json_data = json.loads(invoice_data['sinvoice_attachments'][0]['raw'].decode('utf-8'))
                    if json_data:
                        errors = invoice._l10n_vn_edi_send_invoice(json_data)

                if errors:
                    invoice_data['error'] = {
                        'error_title': _('Error when sending to SInvoice'),
                        'errors': errors,
                    }

                if self._can_commit():
                    self._cr.commit()

    def _call_web_service_after_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_after_invoice_pdf_render(invoices_data)
        for invoice, invoice_data in invoices_data.items():
            # Handle the json file, and create it if it does not yet exist. This can be done without sending to the EDI.
            json_file_data = [file for file in invoice_data.get('sinvoice_attachments', []) if file['mimetype'] == 'application/json']

            if not invoice.l10n_vn_edi_sinvoice_file and json_file_data:
                self.env['ir.attachment'].with_user(SUPERUSER_ID).create(json_file_data)
                invoice.invalidate_recordset(fnames=[
                    'l10n_vn_edi_sinvoice_file_id',
                    'l10n_vn_edi_sinvoice_file',
                ])

            if invoice.l10n_vn_edi_invoice_state != 'sent':
                continue

            # Download SInvoice documents in order to attach them to the email we sent to the customer.
            # If the email is not being sent, we will still get the files and attach them to the invoice.
            xml_data, xml_error_message = invoice._l10n_vn_edi_fetch_invoice_xml_file_data()
            pdf_data, pdf_error_message = invoice._l10n_vn_edi_fetch_invoice_pdf_file_data()
            if xml_error_message or pdf_error_message:
                invoice_data['error'] = {
                    'error_title': _('Error when receiving SInvoice files.'),
                    'errors': [error_message for error_message in [xml_error_message, pdf_error_message] if error_message],
                }

            # Not using _link_invoice_documents for these because it depends on _need_invoice_document and I can't get it to work
            # well while allowing users to download the files before sending.
            attachments_data = []
            for file, error in [(xml_data, xml_error_message), (pdf_data, pdf_error_message)]:
                if error:
                    continue

                attachments_data.append({
                    'name': file['name'],
                    'raw': file['raw'],
                    'mimetype': file['mimetype'],
                    'res_model': invoice._name,
                    'res_id': invoice.id,
                    'res_field': file['res_field'],  # Binary field
                })

            if attachments_data:
                attachments = self.env['ir.attachment'].with_user(SUPERUSER_ID).create(attachments_data)
                invoice.invalidate_recordset(fnames=[
                    'l10n_vn_edi_sinvoice_xml_file_id',
                    'l10n_vn_edi_sinvoice_xml_file',
                    'l10n_vn_edi_sinvoice_pdf_file_id',
                    'l10n_vn_edi_sinvoice_pdf_file',
                ])

                # Log the new attachment in the chatter for reference. Make sure to add the JSON file.
                invoice.with_context(no_new_invoice=True).message_post(
                    body=_('Invoice sent to SInvoice'),
                    attachment_ids=attachments.ids + invoice.l10n_vn_edi_sinvoice_file_id.ids,
                )

            if self._can_commit():
                self._cr.commit()

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_vn_edi_username = fields.Char(
        string='SInvoice Username',
        groups='base.group_system',
    )
    l10n_vn_edi_password = fields.Char(
        string='Sinvoice Password',
        groups='base.group_system',
    )
    l10n_vn_edi_token = fields.Char(
        string='Sinvoice Access Token',
        groups='base.group_system',
        readonly=True,
    )
    l10n_vn_edi_token_expiry = fields.Datetime(
        string='Sinvoice Access Token Expiration Date',
        groups='base.group_system',
        readonly=True,
    )

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_vn_edi_username = fields.Char(
        related='company_id.l10n_vn_edi_username',
        readonly=False,
    )
    l10n_vn_edi_password = fields.Char(
        related='company_id.l10n_vn_edi_password',
        readonly=False,
    )
    l10n_vn_edi_default_symbol = fields.Many2one(
        comodel_name='l10n_vn_edi_viettel.sinvoice.symbol',
        string='Default Symbol',
        groups='base.group_system',
        help='This is the symbol that will be used on partners that do not have a specific symbol on them.',
        compute='_compute_l10n_vn_edi_default_symbol',
        inverse='_inverse_l10n_vn_edi_default_symbol',
    )

    @api.depends('company_id')
    def _compute_l10n_vn_edi_default_symbol(self):
        ResPartner = self.env['res.partner']
        l10n_vn_edi_symbol_field = ResPartner._fields['l10n_vn_edi_symbol']
        self.l10n_vn_edi_default_symbol = l10n_vn_edi_symbol_field.get_company_dependent_fallback(ResPartner)

    def _inverse_l10n_vn_edi_default_symbol(self):
        for setting in self:
            self.env['ir.default'].set(
                'res.partner',
                'l10n_vn_edi_symbol',
                setting.l10n_vn_edi_default_symbol.id,
                company_id=setting.company_id.id
            )

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    invoice_edi_format = fields.Selection(selection_add=[('vn_sinvoice', 'SInvoice file')])
    l10n_vn_edi_symbol = fields.Many2one(
        comodel_name='l10n_vn_edi_viettel.sinvoice.symbol',
        string='Default Symbol',
        help='If set, this symbol will be used as the default symbol for all invoices of this customer.',
        company_dependent=True,
        copy=False,
    )

```

## File: models\sinvoice.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import _, api, fields, models
from odoo.exceptions import UserError


# Invoice template that needs to be passed to Sinvoice and will determine the format of the resulting
# invoice pdf on their system
class SInvoiceTemplate(models.Model):
    _name = 'l10n_vn_edi_viettel.sinvoice.template'
    _description = 'SInvoice template'

    name = fields.Char(
        string='Template Code',
        required=True,
    )
    template_invoice_type = fields.Selection(
        selection=[
            # Circular 32 being deprecated, only display types according to the Circular 78
            ('1', '1 - Value-added invoice'),
            ('2', '2 - Sales invoice'),
            ('3', '3 - Public assets sales'),
            ('4', '4 - National reserve sales'),
            ('5', '5 - Invoice for national reserve sales'),
            ('6', '6 - Warehouse release note'),
        ],
        required=True,
    )
    invoice_symbols_ids = fields.One2many(
        comodel_name='l10n_vn_edi_viettel.sinvoice.symbol',
        inverse_name='invoice_template_id',
    )

    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'The template code must be unique!')
    ]

    @api.constrains('name', 'template_invoice_type')
    def _constrains_changes(self):
        """
        Multiple API endpoints will use these data, we should thus not allow changing them if they have been used
        for any invoices sent to sinvoice.
        """
        # The conditions are the same. If any symbols of the template are being used, we shouldn't allow editing it.
        self.invoice_symbols_ids._constrains_changes()


# Invoice symbol that needs to be passed to Sinvoice and will determine the prefix of the
# invoice number on their system
class SInvoiceSymbol(models.Model):
    _name = 'l10n_vn_edi_viettel.sinvoice.symbol'
    _description = 'SInvoice symbol'
    """
    The invoice symbols are made of multiple parts.
    The symbols characters have meanings that will influence the resulting invoices:

    The first character is either C (invoice with code from tax department) or K (without)
    It is then followed by two number which represent the current year (24 in 2024)
    Next, a single character representing additional invoice information:

        T: E-invoices registered with tax authorities by enterprises, organizations or household businesses;
        D: E-invoices used for sale of public property or national reserve goods or special e-invoices that do not have some mandatory contents of the “T” invoices;
        L: E-invoices issued separately by tax authorities;
        M: E-invoices generated by cash registers;
        N: Electronic delivery and internal consignment notes;
        B: Electronic delivery notes for goods sent to sales agents;
        G: VAT invoices in the form of electronic stamps, tickets or cards;
        H: Sales invoices in the form of electronic stamps, tickets or cards.

    Finally, it ends with two characters that shall be decided by the seller for their management requirements.
    """

    name = fields.Char(
        string='Symbol',
        required=True,
    )
    invoice_template_id = fields.Many2one(
        comodel_name='l10n_vn_edi_viettel.sinvoice.template',
        required=True,
    )

    _sql_constraints = [
        ('name_template_uniq', 'unique (name, invoice_template_id)', 'The combination symbol/template must be unique!')
    ]

    @api.constrains('name', 'invoice_template_id')
    def _constrains_changes(self):
        """
        Multiple API endpoints will use these data, we should thus not allow changing them if they have been used
        for any invoices sent to sinvoice.
        """
        invoice_counts = self.env['account.move']._read_group(
            domain=[
                ('l10n_vn_edi_invoice_symbol', 'in', self.ids),
                ('l10n_vn_edi_invoice_state', 'not in', ('ready_to_send', False)),  # Only matches sent invoices.
            ],
            groupby=['l10n_vn_edi_invoice_symbol'],
            aggregates=['__count'],
        )
        invoices_per_symbol = defaultdict(int)
        for symbol, count in invoice_counts:
            invoices_per_symbol[symbol.id] = count

        for record in self:
            if invoices_per_symbol[record.id] > 0:
                raise UserError(_('You cannot change the symbol value or template of the symbol %s because it has '
                                  'already been used to send invoices.', record.name))

    @api.depends('name', 'invoice_template_id')
    def _compute_display_name(self):
        """ As we allow multiple of the same symbol name, we need to also display the template to differentiate. """
        for symbol in self:
            symbol.display_name = f'{symbol.name} ({symbol.invoice_template_id.name})'

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import account_move_send
from . import res_company
from . import res_config_settings
from . import res_partner
from . import sinvoice

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_vn_edi_viettel.access_l10n_vn_edi_viettel_sinvoice_template,access_sinvoice_template,l10n_vn_edi_viettel.model_l10n_vn_edi_viettel_sinvoice_template,account.group_account_invoice,1,0,0,0
l10n_vn_edi_viettel.access_l10n_vn_edi_viettel_sinvoice_symbol,access_sinvoice_symbol,l10n_vn_edi_viettel.model_l10n_vn_edi_viettel_sinvoice_symbol,account.group_account_invoice,1,0,0,0
l10n_vn_edi_viettel.access_l10n_vn_edi_viettel_sinvoice_template_manager,access_sinvoice_template,l10n_vn_edi_viettel.model_l10n_vn_edi_viettel_sinvoice_template,account.group_account_manager,1,1,1,1
l10n_vn_edi_viettel.access_l10n_vn_edi_viettel_sinvoice_symbol_manager,access_sinvoice_symbol,l10n_vn_edi_viettel.model_l10n_vn_edi_viettel_sinvoice_symbol,account.group_account_manager,1,1,1,1
l10n_vn_edi_viettel.access_l10n_vn_edi_viettel_cancellation,access_l10n_vn_edi_viettel_cancellation,l10n_vn_edi_viettel.model_l10n_vn_edi_viettel_cancellation,account.group_account_invoice,1,1,1,1

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_invoice_form_inherit_l10n_vn_edi" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n_vn_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button string="Send Payment Status"
                        name="action_l10n_vn_edi_update_payment_status"
                        type="object"
                        groups="account.group_account_invoice"
                        invisible="not l10n_vn_edi_invoice_number or l10n_vn_edi_invoice_state != 'payment_state_to_update'"/>
            </xpath>
            <xpath expr="//page[@id='other_tab_entry']" position="after">
                <page id="l10n_vn_edi"
                      string="SInvoice"
                      invisible="move_type not in ['out_invoice', 'out_refund'] or country_code != 'VN'">
                    <group>
                        <group>
                            <field name="l10n_vn_edi_invoice_symbol" readonly="l10n_vn_edi_invoice_number"/>
                            <field name="l10n_vn_edi_issue_date" invisible="not l10n_vn_edi_invoice_number"/>
                        </group>
                        <group>
                            <field name="l10n_vn_edi_invoice_state"/>
                            <field name="l10n_vn_edi_invoice_number" invisible="not l10n_vn_edi_invoice_number"/>
                            <field name="l10n_vn_edi_reservation_code" invisible="not l10n_vn_edi_invoice_number"/>
                        </group>
                        <group string="Agreement"
                               invisible="not l10n_vn_edi_replacement_origin_id
                               and (not reversed_entry_id or not l10n_vn_edi_reversed_entry_invoice_number)">
                            <field name="l10n_vn_edi_adjustment_type" readonly="l10n_vn_edi_invoice_number"/>
                            <field name="l10n_vn_edi_agreement_document_name" readonly="l10n_vn_edi_invoice_number"/>
                            <field name="l10n_vn_edi_agreement_document_date" readonly="l10n_vn_edi_invoice_number"/>
                        </group>
                    </group>
                </page>
            </xpath>
        </field>
    </record>

    <record id="view_account_invoice_filter_inherit_l10n_vn_edi" model="ir.ui.view">
        <field name="name">account.invoice.select.inherit.l10n_vn_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <filter name="late" position="after">
                <separator/>
                <field name="l10n_vn_edi_invoice_state"/>
                <filter string="Need payment state update" name="pay_state_to_update" domain="[('l10n_vn_edi_invoice_state', '=', 'payment_state_to_update')]"/>
            </filter>
        </field>
    </record>

    <record id="view_invoice_tree_inherit_l10n_vn_edi" model="ir.ui.view">
        <field name="name">account.invoice.list.inherit.l10n_vn_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_invoice_tree"/>
        <field name="arch" type="xml">
            <field name="status_in_payment" position="after">
                <field name="l10n_vn_edi_invoice_state"
                       widget="badge"
                       decoration-success="l10n_vn_edi_invoice_state == 'sent'"
                       decoration-warning="l10n_vn_edi_invoice_state == 'ready_to_send'"
                       decoration-info="l10n_vn_edi_invoice_state == 'payment_state_to_update'"
                       optional="show"/>
            </field>
        </field>
    </record>

    <record id="l10n_vn_edi_send_invoice_payment_status" model="ir.actions.server">
        <field name="name">Send payment status to SInvoice</field>
        <field name="model_id" ref="account.model_account_move"/>
        <field name="binding_model_id" ref="account.model_account_move"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
            if records:
                action = records.action_l10n_vn_edi_update_payment_status()
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form_inherit_l10n_vn_edi" model="ir.ui.view">
        <field name="name">res.config.settings.form.inherit.l10n_vn_edi</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//app[@name='account']" position="inside">
                <block title="Vietnamese Integration" id="vn_localization" invisible="country_code != 'VN'">
                    <setting id="sinvoice_setting" string="Setup SInvoice" help="Check the documentation to get credentials" documentation="/applications/finance/fiscal_localizations/vietnam.html" company_dependent="1" >
                        <div class="content-group">
                            <div class="mt16 row">
                                <label for="l10n_vn_edi_username" string="Username" class="col-3 col-lg-3 o_light_label"/>
                                <field name="l10n_vn_edi_username" nolabel="1"/>
                            </div>
                            <div class="row">
                                <label for="l10n_vn_edi_password" string="Password" class="col-3 col-lg-3 o_light_label"/>
                                <field name="l10n_vn_edi_password" password="True" nolabel="1"/>
                            </div>
                            <div class="row">
                                <label for="l10n_vn_edi_default_symbol" class="col-3 col-lg-3 o_light_label"/>
                                <field name="l10n_vn_edi_default_symbol" nolabel="1"/>
                            </div>
                        </div>
                    </setting>
                </block>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <record id="res_partner_view_from_inherit_l10n_vn_edi" model="ir.ui.view" >
        <field name="name">res.partner.form.inherit.l10n_vn_edi</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <group name="accounting_entries" position="after">
                <group string="SInvoice" name="sinvoice" invisible="'VN' not in fiscal_country_codes">
                    <field name="l10n_vn_edi_symbol"/>
                </group>
            </group>
        </field>
    </record>
</odoo>

```

## File: views\sinvoice_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="action_sinvoice_template" model="ir.actions.act_window">
        <field name="name">Templates</field>
        <field name="res_model">l10n_vn_edi_viettel.sinvoice.template</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a template
            </p>
            <p>
                A SInvoice template is used when sending invoices to SInvoice in order to know which
                invoice template to use when rendering the invoice.
            </p>
        </field>
    </record>

    <record id="action_sinvoice_symbol" model="ir.actions.act_window">
        <field name="name">Symbols</field>
        <field name="res_model">l10n_vn_edi_viettel.sinvoice.symbol</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a symbol
            </p>
            <p>
                A SInvoice symbol is used when sending invoices to SInvoice in order to generate the
                invoice sequence.
            </p>
        </field>
    </record>

    <menuitem id="l10n_vn_edi_sinvoice_menu" name="SInvoice" groups="account.group_account_manager"
              parent="account.menu_finance_configuration" sequence="99">
        <menuitem id="l10n_vn_edi_sinvoice_templates" action="action_sinvoice_template"/>
        <menuitem id="l10n_vn_edi_sinvoice_symbols" action="action_sinvoice_symbol"/>
    </menuitem>
</odoo>

```

## File: wizard\account_move_reversal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.exceptions import UserError


class AccountMoveReversal(models.TransientModel):
    _inherit = 'account.move.reversal'

    l10n_vn_edi_adjustment_type = fields.Selection(
        selection=[
            ('1', 'Money adjustment'),
            ('2', 'Information adjustment'),
        ],
        string='Adjustment type',
        required=True,
        default='1',
    )
    l10n_vn_edi_agreement_document_name = fields.Char(
        string='Agreement Name',
    )
    l10n_vn_edi_agreement_document_date = fields.Datetime(
        string='Agreement Date',
    )

    def _prepare_default_reversal(self, move):
        # EXTEND 'account'
        values = super()._prepare_default_reversal(move)
        # This information is required when sending an adjustment invoice to sinvoice.
        # This is not needed when sending a replacement (as we will be sending the invoice, not the CN) but it doesn't hurt to log it.
        if move._l10n_vn_edi_is_sent():
            values.update({
                'l10n_vn_edi_agreement_document_name': self.l10n_vn_edi_agreement_document_name or 'NA',
                'l10n_vn_edi_agreement_document_date': self.l10n_vn_edi_agreement_document_date or fields.Datetime.now(),
                'l10n_vn_edi_adjustment_type': self.l10n_vn_edi_adjustment_type,
            })
        return values

    def _modify_default_reverse_values(self, origin_move):
        # EXTEND 'account'
        values = super()._modify_default_reverse_values(origin_move)
        # This information is REQUIRED on the new invoice that will be sent to sinvoice, if we are creating one.
        if origin_move.l10n_vn_edi_invoice_state not in {False, 'ready_to_send'}:
            values.update({
                'l10n_vn_edi_agreement_document_name': self.l10n_vn_edi_agreement_document_name or 'NA',
                'l10n_vn_edi_agreement_document_date': self.l10n_vn_edi_agreement_document_date or fields.Datetime.now(),
                'l10n_vn_edi_adjustment_type': self.l10n_vn_edi_adjustment_type,
                'l10n_vn_edi_replacement_origin_id': origin_move.id,
            })
        return values

    def reverse_moves(self, is_modify=False):
        # EXTEND 'account'
        for move in self.move_ids.filtered(lambda m: m._l10n_vn_edi_is_sent()):
            # If an invoice has a tax code (symbol starts with C) and the code has not been approved by the tax authorities, you cannot adjust/reverse it.
            if move.l10n_vn_edi_invoice_symbol.name.startswith('C'):
                invoice_lookup, _error_message = move._l10n_vn_edi_lookup_invoice()
                if 'result' in invoice_lookup and invoice_lookup['result'][0].get('exchangeStatus') != 'INVOICE_HAS_CODE_APPROVED':
                    raise UserError(_('You cannot adjust/replace invoice %s, it has not been approved by the tax authorities.\n'
                                      'Please cancel/reverse it and create a new invoice instead.', move.name))

            # Makes sure to keep the original status up to date by tagging them by either replaced, or adjusted.
            if is_modify:
                move.l10n_vn_edi_invoice_state = 'replaced'
            else:
                move.l10n_vn_edi_invoice_state = 'adjusted'

        return super().reverse_moves(is_modify)

```

## File: wizard\account_move_reversal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_move_reversal_inherit_l10n_vn_edi" model="ir.ui.view">
            <field name="name">account.move.reversal.form.inherit.l10n_vn_edi</field>
            <field name="model">account.move.reversal</field>
            <field name="inherit_id" ref="account.view_account_move_reversal"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='reason']" position="after">
                    <field name="l10n_vn_edi_adjustment_type" placeholder="Adjustment type..." invisible="country_code != 'VN'" required="country_code == 'VN'"/>
                    <field name="l10n_vn_edi_agreement_document_name" invisible="country_code != 'VN'" placeholder="Agreement document name..."/>
                    <field name="l10n_vn_edi_agreement_document_date" invisible="country_code != 'VN'" placeholder="Agreement document date..."/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\l10n_vn_edi_cancellation_request.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class L10nVNEdiCancellation(models.TransientModel):
    _name = 'l10n_vn_edi_viettel.cancellation'
    _description = 'E-invoice cancellation wizard'

    invoice_id = fields.Many2one(
        comodel_name='account.move',
        string='Invoice to cancel',
    )
    reason = fields.Char(
        string='Reason',
        required=True,
    )
    agreement_document_name = fields.Char(
        string='Agreement Name',
    )
    agreement_document_date = fields.Datetime(
        string='Agreement Date',
    )

    def button_request_cancel(self):
        self.invoice_id._l10n_vn_edi_cancel_invoice(
            self.reason,
            self.agreement_document_name or 'NA',
            self.agreement_document_date or fields.Datetime.now(),
        )

```

## File: wizard\l10n_vn_edi_cancellation_request_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_vn_edi_cancellation_form" model="ir.ui.view">
        <field name="name">l10n_vn_edi_viettel.cancellation.form</field>
        <field name="model">l10n_vn_edi_viettel.cancellation</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="reason" placeholder="Reason..."/>
                    <field name="agreement_document_name" placeholder="Agreement document name..."/>
                    <field name="agreement_document_date" placeholder="Agreement document date..."/>
                </group>
                <footer>
                    <button string="Request Cancellation" name="button_request_cancel" type="object" default_focus="1" class="btn-primary"/>
                    <button string="Close" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_reversal
from . import l10n_vn_edi_cancellation_request

```

