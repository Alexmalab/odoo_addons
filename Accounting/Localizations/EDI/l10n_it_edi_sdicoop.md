# Odoo Module: l10n_it_edi_sdicoop

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

from odoo import api, SUPERUSER_ID


def _disable_pec_mail_post_init(cr, registry):
    ''' Pec mail cannot be used in conjunction with SdiCoop, so disable the Pec fetchmail servers.
    '''
    env = api.Environment(cr, SUPERUSER_ID, {})

    env['fetchmail.server'].search([('l10n_it_is_pec', '=', True)]).l10n_it_is_pec = False
    env['res.company'].search([('l10n_it_mail_pec_server_id', '!=', None)]).l10n_it_mail_pec_server_id = None

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Italy - E-invoicing (SdiCoop)',
    'version': '0.3',
    'depends': [
        'l10n_it_edi',
        'account_edi',
        'account_edi_proxy_client',
    ],
    'author': 'Odoo',
    'description': """
E-invoice implementation for Italy with the web-service. Ability to send and receive document from SdiCoop. Files sent by SdiCoop are first stored on the proxy
and then fetched by this module.
    """,
    'category': 'Accounting/Localizations/EDI',
    'website': 'http://www.odoo.com/',
    'data': [
        'data/cron.xml',
        'views/l10n_it_view.xml',
        'views/res_config_settings_views.xml',
    ],
    'post_init_hook': '_disable_pec_mail_post_init',
    'license': 'LGPL-3',
}

```

## File: data\cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_receive_fattura_pa_invoice" model="ir.cron">
        <field name="name">FatturaPA: Receive invoices from the exchange system</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="model_id" ref="account_edi.model_account_edi_format"/>
        <field name="code">model._cron_receive_fattura_pa()</field>
        <field name="doall" eval="False"/>
        <field name="state">code</field>
    </record>
</odoo>

```

## File: i18n_extra\l10n_it_edi_sdicoop.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_it_edi_sdicoop
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 14.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2022-03-30 10:14+0000\n"
"PO-Revision-Date: 2022-03-30 10:14+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_it_edi_sdicoop
#: model_terms:ir.ui.view,arch_db:l10n_it_edi_sdicoop.res_config_settings_view_form
msgid ""
"<span class=\"o_form_label\">\n"
"                                        Fattura Elettronica mode\n"
"                                    </span>"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model_terms:ir.ui.view,arch_db:l10n_it_edi_sdicoop.res_config_settings_view_form
msgid "<span class=\"o_form_label\">Allow Odoo to process invoices</span>"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model_terms:ir.ui.view,arch_db:l10n_it_edi_sdicoop.res_config_settings_view_form
msgid "A Demo service is in use."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model_terms:ir.ui.view,arch_db:l10n_it_edi_sdicoop.res_config_settings_view_form
msgid "An Official or Test service has been registered."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid "Attached file is empty"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model_terms:ir.ui.view,arch_db:l10n_it_edi_sdicoop.res_config_settings_view_form
msgid "By checking this box, I accept that Odoo may process my invoices."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model,name:l10n_it_edi_sdicoop.model_res_config_settings
msgid "Config Settings"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields.selection,name:l10n_it_edi_sdicoop.selection__res_config_settings__l10n_it_edi_sdicoop_demo_mode__demo
msgid "Demo"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_edi_format__display_name
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_move__display_name
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_res_config_settings__display_name
msgid "Display Name"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model,name:l10n_it_edi_sdicoop.model_account_edi_format
msgid "EDI format"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model_terms:ir.ui.view,arch_db:l10n_it_edi_sdicoop.res_config_settings_view_form
msgid "Electronic Document Invoicing"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_bank_statement_line__l10n_it_edi_attachment_id
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_move__l10n_it_edi_attachment_id
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_payment__l10n_it_edi_attachment_id
msgid "FatturaPA Attachment"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_bank_statement_line__l10n_it_edi_transaction
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_move__l10n_it_edi_transaction
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_payment__l10n_it_edi_transaction
msgid "FatturaPA Transaction"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.actions.server,name:l10n_it_edi_sdicoop.ir_cron_receive_fattura_pa_invoice_ir_actions_server
#: model:ir.cron,cron_name:l10n_it_edi_sdicoop.ir_cron_receive_fattura_pa_invoice
#: model:ir.cron,name:l10n_it_edi_sdicoop.ir_cron_receive_fattura_pa_invoice
msgid "FatturaPA: Receive invoices from the exchange system"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_edi_format__id
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_move__id
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_res_config_settings__id
msgid "ID"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model_terms:ir.ui.view,arch_db:l10n_it_edi_sdicoop.res_config_settings_view_form
msgid ""
"In demo mode Odoo will just simulate the sending of invoices to the government.<br/>\n"
"                                        In test mode (experimental) Odoo will send the invoices to a non-production service.\n"
"                                        Saving this change will direct all companies on this database to this use this configuration.\n"
"                                        Once registered for testing or official, the mode cannot be changed."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"Invoices for PA are not managed by Odoo, you can download the document and "
"send it on your own."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_res_config_settings__is_edi_proxy_active
msgid "Is Edi Proxy Active"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid "Italian invoice: %s"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model,name:l10n_it_edi_sdicoop.model_account_move
msgid "Journal Entry"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_res_config_settings__l10n_it_edi_proxy_current_state
msgid "L10N It Edi Proxy Current State"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_res_config_settings__l10n_it_edi_sdicoop_demo_mode
msgid "L10N It Edi Sdicoop Demo Mode"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_res_config_settings__l10n_it_edi_sdicoop_register
msgid "L10N It Edi Sdicoop Register"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_edi_format____last_update
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_account_move____last_update
#: model:ir.model.fields,field_description:l10n_it_edi_sdicoop.field_res_config_settings____last_update
msgid "Last Modified on"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields.selection,name:l10n_it_edi_sdicoop.selection__res_config_settings__l10n_it_edi_sdicoop_demo_mode__prod
msgid "Official"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"Please fill your codice fiscale to be able to receive invoices from "
"FatturaPA"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid "Service momentarily unavailable"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: model:ir.model.fields.selection,name:l10n_it_edi_sdicoop.selection__res_config_settings__l10n_it_edi_sdicoop_demo_mode__test
msgid "Test (experimental)"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/res_config_settings.py:0
#, python-format
msgid ""
"The company has already registered with the service as 'Test' or 'Official',"
" it cannot change."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"The filename is duplicated. Try again (or adjust the FatturaPA Filename "
"sequence). Original message from the SDI: %s"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"The invoice has been issued, but the delivery to the Addressee has failed. "
"You will be required to send a courtesy copy of the invoice to your customer"
" through another channel, outside of the Exchange System, and promptly "
"notify him that the original is deposited in his personal area on the portal"
" \"Invoices and Fees\" of the Revenue Agency."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"The invoice has been issued, but the delivery to the Public Administration "
"has failed. The Exchange System will contact them to report the problem and "
"request that they provide a solution. During the following 10 days, the "
"Exchange System will try to forward the FatturaPA file to the Public "
"Administration in question again. Should this also fail, the System will "
"notify Odoo of the failed delivery, and you will be required to send the "
"invoice to the Administration through another channel, outside of the "
"Exchange System."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid "The invoice has been refused by the Exchange System"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"The invoice has been succesfully transmitted. The addressee has 15 days to "
"accept or reject it."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid "The invoice was refused by the addressee."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"The invoice was sent to FatturaPA, but we are still awaiting a response. "
"Click the link above to check for an update."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"The invoice was successfully transmitted to the Public Administration and we"
" are waiting for confirmation"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"The invoice was successfully transmitted to the Public Administration and we"
" are waiting for confirmation."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"This invoice number had already been submitted to the SdI, so it is set as Sent. Please verify that the system is correctly configured, because the correct flow does not need to send the same invoice twice for any reason.\n"
" Original message from the SDI: %s"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid "Unauthorized user"
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid "You are not allowed to check the status of this invoice."
msgstr ""

#. module: l10n_it_edi_sdicoop
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#: code:addons/l10n_it_edi_sdicoop/models/account_edi_format.py:0
#, python-format
msgid ""
"You must accept the terms and conditions in the settings to use FatturaPA."
msgstr ""

```

## File: models\account_edi_format.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _, _lt
from odoo.exceptions import UserError
from odoo.addons.account_edi_proxy_client.models.account_edi_proxy_user import AccountEdiProxyError

from lxml import etree
import base64
import logging

_logger = logging.getLogger(__name__)


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    # -------------------------------------------------------------------------
    # Import
    # -------------------------------------------------------------------------

    def _cron_receive_fattura_pa(self):
        ''' Check the proxy for incoming invoices.
        '''
        proxy_users = self.env['account_edi_proxy_client.user'].search([('edi_format_id', '=', self.env.ref('l10n_it_edi.edi_fatturaPA').id)])

        if proxy_users._get_demo_state() == 'demo':
            return

        for proxy_user in proxy_users:
            company = proxy_user.company_id
            try:
                res = proxy_user._make_request(proxy_user._get_server_url() + '/api/l10n_it_edi/1/in/RicezioneInvoice')

            except AccountEdiProxyError as e:
                res = {}
                _logger.error('Error while receiving file from SdiCoop: %s', e)

            proxy_acks = []
            retrigger = False
            for id_transaction, fattura in res.items():

                # The server has a maximum number of documents it can send at a time
                # If that maximum is reached, then we search for more
                # by re-triggering the download cron, avoiding the timeout.
                current_num, max_num = fattura.get('current_num', 0), fattura.get('max_num', 0)
                retrigger = retrigger or current_num == max_num > 0

                if self.env['ir.attachment'].search([('name', '=', fattura['filename']), ('res_model', '=', 'account.move')], limit=1):
                    # name should be unique, the invoice already exists
                    _logger.info('E-invoice already exist: %s', fattura['filename'])
                    proxy_acks.append(id_transaction)
                    continue

                file = proxy_user._decrypt_data(fattura['file'], fattura['key'])

                try:
                    tree = etree.fromstring(file)
                except Exception:
                    # should not happen as the file has been checked by SdiCoop
                    _logger.info('Received file badly formatted, skipping: \n %s', file)
                    continue
                invoice = self.env['account.move'].with_company(company).create({'move_type': 'in_invoice'})
                attachment = self.env['ir.attachment'].create({
                    'name': fattura['filename'],
                    'raw': file,
                    'type': 'binary',
                    'res_model': 'account.move',
                    'res_id': invoice.id
                })
                if not self.env.context.get('test_skip_commit'):
                    self.env.cr.commit() #In case something fails after, we still have the attachment
                # So that we don't delete the attachment when deleting the invoice
                attachment.res_id = False
                attachment.res_model = False
                invoice.unlink()
                invoice = self.env.ref('l10n_it_edi.edi_fatturaPA')._create_invoice_from_xml_tree(fattura['filename'], tree)
                attachment.write({'res_model': 'account.move',
                                  'res_id': invoice.id})
                proxy_acks.append(id_transaction)
                if not self.env.context.get('test_skip_commit'):
                    self.env.cr.commit()

            if proxy_acks:
                try:
                    proxy_user._make_request(proxy_user._get_server_url() + '/api/l10n_it_edi/1/ack',
                                            params={'transaction_ids': proxy_acks})
                except AccountEdiProxyError as e:
                    _logger.error('Error while receiving file from SdiCoop: %s', e)

            if retrigger:
                _logger.info('Retriggering "Receive invoices from the exchange system"...')
                self.env.ref('l10n_it_edi_sdicoop.ir_cron_receive_fattura_pa_invoice')._trigger()

    # -------------------------------------------------------------------------
    # Export
    # -------------------------------------------------------------------------

    def _get_invoice_edi_content(self, move):
        #OVERRIDE
        if self.code != 'fattura_pa':
            return super()._get_invoice_edi_content(move)
        return move._export_as_xml()

    def _check_move_configuration(self, move):
        # OVERRIDE
        res = super()._check_move_configuration(move)
        if self.code != 'fattura_pa':
            return res

        res.extend(self._l10n_it_edi_check_invoice_configuration(move))

        if not self._get_proxy_user(move.company_id):
            res.append(_("You must accept the terms and conditions in the settings to use FatturaPA."))

        return res

    def _needs_web_services(self):
        self.ensure_one()
        return self.code == 'fattura_pa' or super()._needs_web_services()

    def _l10n_it_edi_is_required_for_invoice(self, invoice):
        """ _is_required_for_invoice for SdiCoop.
            OVERRIDE
        """
        is_self_invoice = self._l10n_it_edi_is_self_invoice(invoice)
        return (
            (invoice.is_sale_document() or (is_self_invoice and invoice.is_purchase_document()))
            and invoice.l10n_it_send_state not in ('sent', 'delivered', 'delivered_accepted')
            and invoice.country_code == 'IT'
        )

    def _support_batching(self, move=None, state=None, company=None):
        # OVERRIDE
        if self.code == 'fattura_pa':
            return state == 'to_send' and move.is_invoice()

        return super()._support_batching(move=move, state=state, company=company)

    def _get_batch_key(self, move, state):
        # OVERRIDE
        if self.code != 'fattura_pa':
            return super()._get_batch_key(move, state)

        return move.move_type, bool(move.l10n_it_edi_transaction)

    def _l10n_it_post_invoices_step_1(self, invoices):
        ''' Send the invoices to the proxy.
        '''
        to_return = {}

        to_send = {}
        for invoice in invoices:
            xml = "<?xml version='1.0' encoding='UTF-8'?>" + str(invoice._export_as_xml())
            filename = self._l10n_it_edi_generate_electronic_invoice_filename(invoice)
            attachment = self.env['ir.attachment'].create({
                'name': filename,
                'res_id': invoice.id,
                'res_model': invoice._name,
                'raw': xml.encode(),
                'description': _('Italian invoice: %s', invoice.move_type),
                'type': 'binary',
            })
            invoice.l10n_it_edi_attachment_id = attachment

            if invoice._is_commercial_partner_pa():
                invoice.message_post(
                    body=(_("Invoices for PA are not managed by Odoo, you can download the document and send it on your own."))
                )
                to_return[invoice] = {'attachment': attachment, 'success': True}
            else:
                to_send[filename] = {
                    'invoice': invoice,
                    'data': {'filename': filename, 'xml': base64.b64encode(xml.encode()).decode()}}

        company = invoices.company_id
        proxy_user = self._get_proxy_user(company)
        if not proxy_user:  # proxy user should exist, because there is a check in _check_move_configuration
            return {invoice: {
                'error': _("You must accept the terms and conditions in the settings to use FatturaPA."),
                'blocking_level': 'error'} for invoice in invoices}

        responses = {}
        if proxy_user._get_demo_state() == 'demo':
            responses = {i['data']['filename']: {'id_transaction': 'demo'} for i in to_send.values()}
        else:
            try:
                responses = self._l10n_it_edi_upload([i['data'] for i in to_send.values()], proxy_user)
            except AccountEdiProxyError as e:
                return {invoice: {'error': e.message, 'blocking_level': 'error'} for invoice in invoices}

        for filename, response in responses.items():
            invoice = to_send[filename]['invoice']
            to_return[invoice] = response
            if 'id_transaction' in response:
                invoice.l10n_it_edi_transaction = response['id_transaction']
                to_return[invoice].update({
                    'error': _('The invoice was sent to FatturaPA, but we are still awaiting a response. Click the link above to check for an update.'),
                    'blocking_level': 'info',
                })
        return to_return

    def _l10n_it_post_invoices_step_2(self, invoices):
        ''' Check if the sent invoices have been processed by FatturaPA.
        '''
        to_check = {i.l10n_it_edi_transaction: i for i in invoices}
        to_return = {}
        company = invoices.company_id

        proxy_user = self._get_proxy_user(company)
        if not proxy_user:  # proxy user should exist, because there is a check in _check_move_configuration
            return {invoice: {
                'error': _("You must accept the terms and conditions in the settings to use FatturaPA."),
                'blocking_level': 'error'} for invoice in invoices}

        if proxy_user._get_demo_state() == 'demo':
            # simulate success and bypass ack
            return {invoice: {'attachment': invoice.l10n_it_edi_attachment_id} for invoice in invoices}
        else:
            try:
                responses = proxy_user._make_request(proxy_user._get_server_url() + '/api/l10n_it_edi/1/in/TrasmissioneFatture',
                                                    params={'ids_transaction': list(to_check.keys())})
            except AccountEdiProxyError as e:
                return {invoice: {'error': e.message, 'blocking_level': 'error'} for invoice in invoices}

        proxy_acks = []
        for id_transaction, response in responses.items():
            invoice = to_check[id_transaction]
            if 'error' in response:
                to_return[invoice] = response
                continue

            state = response['state']
            if state == 'awaiting_outcome':
                to_return[invoice] = {
                    'error': _('The invoice was sent to FatturaPA, but we are still awaiting a response. Click the link above to check for an update.'),
                    'blocking_level': 'info'}

            elif state == 'not_found':
                # Invoice does not exist on proxy. Either it does not belong to this proxy_user or it was not created correctly when
                # it was sent to the proxy.
                to_return[invoice] = {'error': _('You are not allowed to check the status of this invoice.'), 'blocking_level': 'error'}

            elif state == 'ricevutaConsegna':
                if invoice._is_commercial_partner_pa():
                    to_return[invoice] = {'error': _('The invoice has been succesfully transmitted. The addressee has 15 days to accept or reject it.')}
                else:
                    to_return[invoice] = {'attachment': invoice.l10n_it_edi_attachment_id, 'success': True}
                proxy_acks.append(id_transaction)

            elif state == 'notificaMancataConsegna':
                if invoice._is_commercial_partner_pa():
                    to_return[invoice] = {'error': _(
                        'The invoice has been issued, but the delivery to the Public Administration'
                        ' has failed. The Exchange System will contact them to report the problem'
                        ' and request that they provide a solution.'
                        ' During the following 10 days, the Exchange System will try to forward the'
                        ' FatturaPA file to the Public Administration in question again.'
                        ' Should this also fail, the System will notify Odoo of the failed delivery,'
                        ' and you will be required to send the invoice to the Administration'
                        ' through another channel, outside of the Exchange System.')}
                else:
                    to_return[invoice] = {'success': True, 'attachment': invoice.l10n_it_edi_attachment_id}
                    invoice._message_log(body=_(
                        'The invoice has been issued, but the delivery to the Addressee has'
                        ' failed. You will be required to send a courtesy copy of the invoice'
                        ' to your customer through another channel, outside of the Exchange'
                        ' System, and promptly notify him that the original is deposited'
                        ' in his personal area on the portal "Invoices and Fees" of the'
                        ' Revenue Agency.'))
                proxy_acks.append(id_transaction)

            elif state == 'NotificaDecorrenzaTermini':
                # This condition is part of the Public Administration flow
                invoice._message_log(body=_(
                    'The invoice has been correctly issued. The Public Administration recipient'
                    ' had 15 days to either accept or refused this document, but they did not reply,'
                    ' so from now on we consider it accepted.'))
                to_return[invoice] = {'attachment': invoice.l10n_it_edi_attachment_id, 'success': True}
                proxy_acks.append(id_transaction)

            # In the transaction states above, we don't need to read the attachment.
            # In the following cases instead we need to read the information inside
            # about the notification itself, i.e. the error message in case of rejection.
            else:
                attachment_file = response.get('file')
                if not attachment_file: # It means there is no status update, so we can skip it
                    document = invoice.edi_document_ids.filtered(lambda d: d.edi_format_id.code == 'fattura_pa')
                    to_return[invoice] = {'error': document.error, 'blocking_level': document.blocking_level}
                    continue

                xml = proxy_user._decrypt_data(attachment_file, response['key'])
                response_tree = etree.fromstring(xml)

                if state == 'notificaScarto':
                    elements = response_tree.xpath('//Errore')
                    error_codes = [element.find('Codice').text for element in elements]
                    errors = [element.find('Descrizione').text for element in elements]
                    # Duplicated invoice
                    if '00404' in error_codes:
                        idx = error_codes.index('00404')
                        invoice.message_post(body=_(
                            'This invoice number had already been submitted to the SdI, so it is'
                            ' set as Sent. Please verify that the system is correctly configured,'
                            ' because the correct flow does not need to send the same invoice'
                            ' twice for any reason.\n'
                            ' Original message from the SDI: %s', errors[idx]))
                        to_return[invoice] = {'attachment': invoice.l10n_it_edi_attachment_id, 'success': True}
                    else:
                        # Add helpful text if duplicated filename error
                        if '00002' in error_codes:
                            idx = error_codes.index('00002')
                            errors[idx] = _(
                                'The filename is duplicated. Try again (or adjust the FatturaPA Filename sequence).'
                                ' Original message from the SDI: %s', [errors[idx]]
                            )
                        to_return[invoice] = {'error': self._format_error_message(_('The invoice has been refused by the Exchange System'), errors), 'blocking_level': 'error'}
                        invoice.l10n_it_edi_transaction = False
                    proxy_acks.append(id_transaction)

                elif state == 'notificaEsito':
                    outcome = response_tree.find('Esito').text
                    if outcome == 'EC01':
                        to_return[invoice] = {'attachment': invoice.l10n_it_edi_attachment_id, 'success': True}
                    else:  # ECO2
                        to_return[invoice] = {'error': _('The invoice was refused by the addressee.'), 'blocking_level': 'error'}
                    proxy_acks.append(id_transaction)

        if proxy_acks:
            try:
                proxy_user._make_request(proxy_user._get_server_url() + '/api/l10n_it_edi/1/ack',
                                        params={'transaction_ids': proxy_acks})
            except AccountEdiProxyError as e:
                # Will be ignored and acked again next time.
                _logger.error('Error while acking file to SdiCoop: %s', e)

        return to_return

    def _post_fattura_pa(self, invoices):
        # OVERRIDE
        if not invoices[0].l10n_it_edi_transaction:
            return self._l10n_it_post_invoices_step_1(invoices)
        else:
            return self._l10n_it_post_invoices_step_2(invoices)

    # -------------------------------------------------------------------------
    # Proxy methods
    # -------------------------------------------------------------------------

    def _get_proxy_identification(self, company):
        if self.code != 'fattura_pa':
            return super()._get_proxy_identification()

        if not company.l10n_it_codice_fiscale:
            raise UserError(_('Please fill your codice fiscale to be able to receive invoices from FatturaPA'))

        return self.env['res.partner']._l10n_it_normalize_codice_fiscale(company.l10n_it_codice_fiscale)

    def _l10n_it_edi_upload(self, files, proxy_user):
        '''Upload files to fatturapa.

        :param files:    A list of dictionary {filename, base64_xml}.
        :returns:        A dictionary.
        * message:       Message from fatturapa.
        * transactionId: The fatturapa ID of this request.
        * error:         An eventual error.
        * error_level:   Info, warning, error.
        '''
        ERRORS = {
            'EI01': {'error': _lt('Attached file is empty'), 'blocking_level': 'error'},
            'EI02': {'error': _lt('Service momentarily unavailable'), 'blocking_level': 'warning'},
            'EI03': {'error': _lt('Unauthorized user'), 'blocking_level': 'error'},
        }

        if not files:
            return {}

        result = proxy_user._make_request(proxy_user._get_server_url() + '/api/l10n_it_edi/1/out/SdiRiceviFile', params={'files': files})

        # Translate the errors.
        for filename in result.keys():
            if 'error' in result[filename]:
                result[filename] = ERRORS.get(result[filename]['error'], {'error': result[filename]['error'], 'blocking_level': 'error'})

        return result

```

## File: models\account_invoice.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import api, fields, models


_logger = logging.getLogger(__name__)

DEFAULT_FACTUR_ITALIAN_DATE_FORMAT = '%Y-%m-%d'


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_edi_transaction = fields.Char(copy=False, string="FatturaPA Transaction")
    l10n_it_edi_attachment_id = fields.Many2one('ir.attachment', copy=False, string="FatturaPA Attachment", ondelete="restrict")

    def send_pec_mail(self):
        self.ensure_one()
        # OVERRIDE
        # With SdiCoop web-service, no need to send PEC mail.
        # Set the state to 'other' because the invoice should not be managed par l10n_it_edi.
        self.l10n_it_send_state = 'other'

    @api.depends('l10n_it_edi_transaction')
    def _compute_show_reset_to_draft_button(self):
        super(AccountMove, self)._compute_show_reset_to_draft_button()
        for move in self.filtered(lambda m: m.l10n_it_edi_transaction):
            move.show_reset_to_draft_button = False

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models, fields, _
from odoo.exceptions import UserError

class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    is_edi_proxy_active = fields.Boolean(compute='_compute_is_edi_proxy_active')
    l10n_it_edi_proxy_current_state = fields.Char(compute='_compute_l10n_it_edi_proxy_current_state')
    l10n_it_edi_sdicoop_register = fields.Boolean(compute='_compute_l10n_it_edi_sdicoop_register', inverse='_set_l10n_it_edi_sdicoop_register_demo_mode')
    l10n_it_edi_sdicoop_demo_mode = fields.Selection(
        [('demo', 'Demo'),
         ('test', 'Test (experimental)'),
         ('prod', 'Official')],
        compute='_compute_l10n_it_edi_sdicoop_demo_mode',
        inverse='_set_l10n_it_edi_sdicoop_register_demo_mode',
        readonly=False)

    def _create_proxy_user(self, company_id):
        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        edi_identification = fattura_pa._get_proxy_identification(company_id)
        self.env['account_edi_proxy_client.user']._register_proxy_user(company_id, fattura_pa, edi_identification)

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_l10n_it_edi_sdicoop_demo_mode(self):
        for config in self:
            config.l10n_it_edi_sdicoop_demo_mode = self.env['account_edi_proxy_client.user']._get_demo_state()

    def _set_l10n_it_edi_sdicoop_demo_mode(self):
        for config in self:
            self.env['ir.config_parameter'].set_param('account_edi_proxy_client.demo', config.l10n_it_edi_sdicoop_demo_mode)

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_is_edi_proxy_active(self):
        for config in self:
            config.is_edi_proxy_active = config.company_id.account_edi_proxy_client_ids

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_l10n_it_edi_proxy_current_state(self):
        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        for config in self:
            proxy_user = config.company_id.account_edi_proxy_client_ids.search([
                ('company_id', '=', config.company_id.id),
                ('edi_format_id','=', fattura_pa.id),
            ], limit=1)

            config.l10n_it_edi_proxy_current_state = 'inactive' if not proxy_user else 'demo' if proxy_user.id_client[:4] == 'demo' else 'active'

    @api.depends('company_id')
    def _compute_l10n_it_edi_sdicoop_register(self):
        """Needed because it expects a compute"""
        self.l10n_it_edi_sdicoop_register = False

    def button_create_proxy_user(self):
        # For now, only fattura_pa uses the proxy.
        # To use it for more, we have to either make the activation of the proxy on a format basis
        # or create a user per format here (but also when installing new formats)
        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        edi_identification = fattura_pa._get_proxy_identification(self.company_id)
        if not edi_identification:
            return

        self.env['account_edi_proxy_client.user']._register_proxy_user(self.company_id, fattura_pa, edi_identification)

    def _set_l10n_it_edi_sdicoop_register_demo_mode(self):

        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        for config in self:

            proxy_user = self.env['account_edi_proxy_client.user'].search([
                ('company_id', '=', config.company_id.id),
                ('edi_format_id', '=', fattura_pa.id)
            ], limit=1)

            real_proxy_users = self.env['account_edi_proxy_client.user'].sudo().search([
                ('id_client', 'not like', 'demo'),
            ])

            # Update the config as per the selected radio button
            previous_demo_state = proxy_user._get_demo_state()
            self.env['ir.config_parameter'].set_param('account_edi_proxy_client.demo', config.l10n_it_edi_sdicoop_demo_mode)

            # If the user is trying to change from a state in which they have a registered official or testing proxy client
            # to another state, we should stop them
            if real_proxy_users and previous_demo_state != config.l10n_it_edi_sdicoop_demo_mode:
                raise UserError(_("The company has already registered with the service as 'Test' or 'Official', it cannot change."))


            if config.l10n_it_edi_sdicoop_register:
                # There should only be one user at a time, if there are no users, register one
                if not proxy_user:
                    self._create_proxy_user(config.company_id)
                    return

                # If there is a demo user, and we are transitioning from demo to test or production, we should
                # delete all demo users and then create the new user.
                elif proxy_user.id_client[:4] == 'demo' and config.l10n_it_edi_sdicoop_demo_mode != 'demo':
                    self.env['account_edi_proxy_client.user'].search([('id_client', '=like', 'demo%')]).sudo().unlink()
                    self._create_proxy_user(config.company_id)


```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_invoice
from . import account_edi_format
from . import res_config_settings

```

## File: views\l10n_it_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fetchmail_server_form_l10n_it_inherit" model="ir.ui.view">
        <field name="name">fetchmail.server.form.l10n.it.inherit</field>
        <field name="model">fetchmail.server</field>
        <field name="priority">30</field>
        <field name="inherit_id" ref="l10n_it_edi.fetchmail_server_form_l10n_it"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//field[@name='l10n_it_is_pec']" position="replace" />
        </data>
        </field>
    </record>

    <record id="res_company_form_l10n_it_inherit" model="ir.ui.view">
        <field name="name">res.company.form.l10n.it.inherit</field>
        <field name="model">res.company</field>
        <field name="priority">30</field>
        <field name="inherit_id" ref="l10n_it_edi.res_company_form_l10n_it"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//field[@name='l10n_it_mail_pec_server_id']" position="replace"/>
                <xpath expr="//field[@name='l10n_it_address_send_fatturapa']" position="replace"/>
                <xpath expr="//field[@name='l10n_it_address_recipient_fatturapa']" position="replace"/>
            </data>
        </field>
    </record>

    <record id="l10n_it_edi.invoice_supplier_tree_l10n_it" model="ir.ui.view">
        <field name="arch" type="xml">
            <!-- Remove the l10n_state_it field. Doing so
            with xpath might replace other view elements that we shouldn't remove.-->
            <data/>
        </field>
    </record>

    <record id="l10n_it_edi.invoice_kanban_l10n_it" model="ir.ui.view">
        <field name="arch" type="xml">
            <!-- Remove the l10n_state_it field. Doing so
            with xpath might replace other view elements that we shouldn't remove.-->
            <data/>
        </field>
    </record>

    <record id="l10n_it_edi.account_invoice_form_l10n_it" model="ir.ui.view">
        <field name="arch" type="xml">
            <!-- Remove the l10n_state_it field but keep other info tab.
            Doing so with xpath might repace other view elements that we shouldn't remove. -->
            <data>
                <xpath expr="//page[@name='other_info']" position="after">
                    <page string="Electronic Invoicing"
                        name="electronic_invoicing"
                        attrs="{'invisible': [('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund'))]}">
                        <group>
                            <group>
                                <field name="l10n_it_stamp_duty"/>
                                <field name="l10n_it_ddt_id"
                                       attrs="{'invisible': [('move_type', 'not in', ('out_invoice', 'out_refund'))]}"/>
                            </group>
                        </group>
                    </page>
                </xpath>
            </data>
        </field>
    </record>


    <record id="l10n_it_edi.view_account_invoice_filter_l10n_it" model="ir.ui.view">
        <field name="arch" type="xml">
            <!-- Remove the filters.-->
            <data />
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.proxy.user</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='account_vendor_bills']" position="after">
                <div attrs="{'invisible':[('country_code', '!=', 'IT')]}">
                    <h2>Electronic Document Invoicing</h2>
                    <div class="row mt16 o_settings_container" id='account_edi'>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_right_pane">
                                <div class="group-content">
                                    <field name="l10n_it_edi_proxy_current_state" invisible="1"/>
                                    <span class="o_form_label">
                                        Fattura Elettronica mode
                                    </span>
                                    <div class="text-muted">
                                        In demo mode Odoo will just simulate the sending of invoices to the government.<br/>
                                        In test mode (experimental) Odoo will send the invoices to a non-production service.
                                        Saving this change will direct all companies on this database to this use this configuration.
                                        Once registered for testing or official, the mode cannot be changed.
                                    </div>
                                    <field name="l10n_it_edi_sdicoop_demo_mode"
                                           widget="radio"
                                           options="{'horizontal': true}"/>
                                </div>
                                <div class="mt8 content-group" attrs="{'invisible': ['|',('l10n_it_edi_proxy_current_state','=','active'), '&amp;', ('l10n_it_edi_proxy_current_state','=','demo'), ('l10n_it_edi_sdicoop_demo_mode','=','demo')]}">
                                    <span class="o_form_label">Allow Odoo to process invoices</span>
                                    <div class="text-muted">
                                        By checking this box, I accept that Odoo may process my invoices.
                                    </div>
                                    <div class="content-group">
                                        <field name="l10n_it_edi_sdicoop_register"/>
                                    </div>

                                </div>
                                <div class="text-success mt8" attrs="{'invisible': [('l10n_it_edi_proxy_current_state','in', ['inactive', 'demo'])]}">
                                    An Official or Test service has been registered.
                                </div>
                                <div class="text-success mt8" attrs="{'invisible': ['|',('l10n_it_edi_proxy_current_state','!=', 'demo'), ('l10n_it_edi_sdicoop_demo_mode', '!=', 'demo')]}">
                                    A Demo service is in use.
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

