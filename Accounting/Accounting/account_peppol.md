# Odoo Module: account_peppol

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard
from . import tools


def _account_peppol_post_init(env):
    for company in env['res.company'].sudo().search([]):
        env['ir.default'].set('res.partner', 'peppol_verification_state', 'not_verified', company_id=company.id)

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Peppol",
    'summary': "This module is used to send/receive documents with PEPPOL",
    'description': """
- Register as a PEPPOL participant
- Send and receive documents via PEPPOL network in Peppol BIS Billing 3.0 format
    """,
    'category': 'Accounting/Accounting',
    'version': '1.1',
    'countries': [
        # !!! KEEP ALIGNED WITH ACCOUNT/MODELS/COMPANY.PEPPOL_DEFAULT_COUNTRIES
        'at', 'be', 'ch', 'cy', 'cz', 'de', 'dk', 'ee', 'es', 'fi',
        'fr', 'gr', 'ie', 'is', 'it', 'lt', 'lu', 'lv', 'mt', 'nl',
        'no', 'pl', 'pt', 'ro', 'se', 'si',
    ],
    'depends': [
        'account_edi_proxy_client',
        'account_edi_ubl_cii',
    ],
    'external_dependencies': {
        'python': ['phonenumbers']
    },
    'data': [
        'data/cron.xml',
        'data/mail_templates_email_layouts.xml',
        'data/res_partner_data.xml',
        'security/ir.model.access.csv',
        'views/account_journal_dashboard_views.xml',
        'views/account_move_views.xml',
        'views/account_portal_templates.xml',
        'views/res_partner_views.xml',
        'views/res_config_settings_views.xml',
        'wizard/peppol_registration_views.xml',
        'wizard/service_wizard.xml',
    ],
    'demo': [
        'demo/account_peppol_demo.xml',
    ],
    'post_init_hook': '_account_peppol_post_init',
    'license': 'LGPL-3',
    'assets': {
        'web.assets_backend': [
            'account_peppol/static/src/components/**/*',
        ],
        'web.assets_frontend': [
            'account_peppol/static/src/js/*',
        ],
    },
    'auto_install': ['account_edi_ubl_cii'],  # auto-install when account_edi_ubl_cii AND one company exists in countries above
}

```

## File: controllers\portal.py

```python
from odoo import _
from odoo.addons.portal.controllers.portal import CustomerPortal
from odoo.addons.account.models.company import PEPPOL_LIST
from odoo.http import request


class PortalAccount(CustomerPortal):

    # ------------------------------------------------------------
    # My Account
    # ------------------------------------------------------------

    def _prepare_portal_layout_values(self):
        # EXTENDS 'portal'
        portal_layout_values = super()._prepare_portal_layout_values()
        can_send = request.env['account_edi_proxy_client.user']._get_can_send_domain()
        if request.env.company.account_peppol_proxy_state in can_send:
            partner = request.env.user.partner_id
            portal_layout_values['invoice_sending_methods'].update({'peppol': _('by Peppol')})
            portal_layout_values.update({
                'peppol_eas_list': dict(partner._fields['peppol_eas'].selection),
            })
        return portal_layout_values

    def _get_mandatory_fields(self):
        # EXTENDS 'portal'
        mandatory_fields = super()._get_mandatory_fields()

        sending_method = request.params.get('invoice_sending_method')
        if sending_method == 'peppol':
            mandatory_fields += ['peppol_eas', 'peppol_endpoint', 'invoice_edi_format']

        return mandatory_fields

    def _get_optional_fields(self):
        # EXTENDS 'portal'
        optional_fields = super()._get_optional_fields()

        sending_method = request.params.get('invoice_sending_method')
        if sending_method and sending_method != 'peppol':
            optional_fields += ['peppol_eas', 'peppol_endpoint']
        return optional_fields

    def details_form_validate(self, data, partner_creation=False):
        # EXTENDS 'portal'
        error, error_message = super().details_form_validate(data, partner_creation=False)

        if data.get('invoice_sending_method') == 'peppol':
            peppol_eas = data.get('peppol_eas')
            peppol_endpoint = data.get('peppol_endpoint')
            edi_format = data.get('invoice_edi_format')
            if request.env['res.country'].browse(int(data.get('country_id'))).code not in PEPPOL_LIST:
                error['country_id'] = 'error'
                error_message.append(_('That country is not available for Peppol.'))
            if endpoint_error_message := request.env['res.partner']._build_error_peppol_endpoint(peppol_eas, peppol_endpoint):
                error['invalid_peppol_endpoint'] = 'error'
                error_message.append(endpoint_error_message)
            if request.env['res.partner']._get_peppol_verification_state(peppol_endpoint, peppol_eas, edi_format) != 'valid':
                error['invalid_peppol_config'] = 'error'
                error_message.append(_('If you want to be invoiced by Peppol, your configuration must be valid.'))

        return error, error_message

```

## File: controllers\__init__.py

```python
from . import portal

```

## File: data\cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_peppol_get_new_documents" model="ir.cron">
        <field name="name">PEPPOL: retrieve new documents</field>
        <field name="interval_number">4</field>
        <field name="interval_type">hours</field>
        <field name="model_id" ref="model_account_edi_proxy_client_user"/>
        <field name="code">model._cron_peppol_get_new_documents()</field>
        <field name="state">code</field>
    </record>

    <record id="ir_cron_peppol_get_message_status" model="ir.cron">
        <field name="name">PEPPOL: update message status</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="model_id" ref="model_account_edi_proxy_client_user"/>
        <field name="code">model._cron_peppol_get_message_status()</field>
        <field name="state">code</field>
    </record>

    <record id="ir_cron_peppol_get_participant_status" model="ir.cron">
        <field name="name">PEPPOL: update participant status</field>
        <field name="interval_number">1</field>
        <field name="interval_type">weeks</field>
        <field name="model_id" ref="model_account_edi_proxy_client_user"/>
        <field name="code">model._cron_peppol_get_participant_status()</field>
        <field name="state">code</field>
    </record>
</odoo>

```

## File: data\mail_templates_email_layouts.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="mail_notification_layout_with_responsible_signature_and_peppol"
                  name="Mail: mail notification layout with responsible signature (user_id of the record) and Peppol advertisement"
                  inherit_id="mail.mail_notification_layout_with_responsible_signature"
                  primary="True">
            <xpath expr="//t[hasclass('o_signature')]" position="after">
                <div id="peppol_advertisement" t-if="peppol_info" style="font-size: 13px;">
                    <t t-if="peppol_info['is_peppol_sent']">
                        <p style="min-width: 590px;">
                            PS: This invoice has also been <b style="color: $o-enterprise-action-color">sent on Peppol</b>.
                        </p>
                    </t>
                    <t t-if="not peppol_info['is_peppol_sent']">
                        <p style="min-width: 590px;">
                            PS: <b style="color: $o-enterprise-action-color;">We did not send your invoice on Peppol.</b>
                            <t t-if="peppol_info['peppol_country'] == 'BE'">
                                In Belgium, electronic invoicing will be <u>mandatory as of January 2026</u>.
                                <a target="_blank" href="https://finance.belgium.be/en/enterprises/vat/e-invoicing/mandatory-use-structured-electronic-invoices-2026" style="text-decoration: none;">
                                    &#x1F517;
                                </a>
                            </t>
                            <br/>
                            If you need a Peppol compliant software, we recommend <a target="_blank" href="https://www.odoo.com/app/invoicing?utm_source=db&amp;utm_medium=email&amp;utm_campaign=einvoicing" style="color: $o-enterprise-color;">Odoo</a>.
                        </p>
                    </t>
                </div>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: data\res_partner_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <function model="ir.default" name="set" eval="('res.partner', 'peppol_verification_state', 'not_verified')"/>
</odoo>
```

## File: models\account_edi_proxy_user.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from datetime import timedelta

from odoo import _, api, fields, models, modules, tools
from odoo.addons.account_edi_proxy_client.models.account_edi_proxy_user import AccountEdiProxyError
from odoo.addons.account_peppol.tools.demo_utils import handle_demo
from odoo.exceptions import UserError
from odoo.tools import split_every

_logger = logging.getLogger(__name__)
BATCH_SIZE = 50


class AccountEdiProxyClientUser(models.Model):
    _inherit = 'account_edi_proxy_client.user'

    peppol_verification_code = fields.Char(string='SMS verification code')  # TODO remove in master
    proxy_type = fields.Selection(selection_add=[('peppol', 'PEPPOL')], ondelete={'peppol': 'cascade'})

    # -------------------------------------------------------------------------
    # HELPER METHODS
    # -------------------------------------------------------------------------
    def _get_proxy_urls(self):
        urls = super()._get_proxy_urls()
        urls['peppol'] = {
            'prod': 'https://peppol.api.odoo.com',
            'test': 'https://peppol.test.odoo.com',
            'demo': 'demo',
        }
        return urls

    @handle_demo
    def _call_peppol_proxy(self, endpoint, params=None):
        self.ensure_one()
        if self.proxy_type != 'peppol':
            raise UserError(_('EDI user should be of type Peppol'))

        errors = {
            'code_incorrect': _('The verification code is not correct'),
            'code_expired': _('This verification code has expired. Please request a new one.'),
            'too_many_attempts': _('Too many attempts to request an SMS code. Please try again later.'),
        }

        params = params or {}
        try:
            response = self._make_request(
                f"{self._get_server_url()}{endpoint}",
                params=params,
            )
        except AccountEdiProxyError as e:
            if (
                e.code == 'no_such_user'
                and not self.active
                and not self.company_id.account_edi_proxy_client_ids.filtered(lambda u: u.proxy_type == 'peppol')
            ):
                self.company_id.write({
                    'account_peppol_proxy_state': 'not_registered',
                    'account_peppol_migration_key': False,
                })
                # commit the above changes before raising below
                if not modules.module.current_test:
                    self.env.cr.commit()
                raise UserError(_('We could not find a user with this information on our server. Please check your information.'))
            raise UserError(e.message)

        if 'error' in response:
            error_code = response['error'].get('code')
            error_message = response['error'].get('message') or response['error'].get('data', {}).get('message')
            raise UserError(errors.get(error_code) or error_message or _('Connection error, please try again later.'))
        return response

    @api.model
    def _get_can_send_domain(self):
        return ('sender', 'smp_registration', 'receiver')

    @handle_demo
    def _check_company_on_peppol(self, company, edi_identification):
        if (
            not company.account_peppol_migration_key
            and (participant_info := company.partner_id._get_participant_info(edi_identification)) is not None
            and company.partner_id._check_peppol_participant_exists(participant_info, edi_identification, check_company=True)
        ):
            error_msg = _(
                "A participant with these details has already been registered on the network. "
                "If you have previously registered to an alternative Peppol service, please deregister from that service, "
                "or request a migration key before trying again. "
            )

            if isinstance(participant_info, str):
                error_msg += _("The Peppol service that is used is likely to be %s.", participant_info)
            raise UserError(error_msg)

    # -------------------------------------------------------------------------
    # CRONS
    # -------------------------------------------------------------------------

    def _cron_peppol_get_new_documents(self):
        edi_users = self.search([('company_id.account_peppol_proxy_state', '=', 'receiver'), ('proxy_type', '=', 'peppol')])
        edi_users._peppol_get_new_documents()

    def _cron_peppol_get_message_status(self):
        edi_users = self.search([('company_id.account_peppol_proxy_state', 'in', self._get_can_send_domain()), ('proxy_type', '=', 'peppol')])
        edi_users._peppol_get_message_status()

    def _cron_peppol_get_participant_status(self):
        edi_users = self.search([('proxy_type', '=', 'peppol')])
        edi_users._peppol_get_participant_status()

        # throughout the registration process, we need to check the status more frequently
        if self.search_count([('company_id.account_peppol_proxy_state', '=', 'smp_registration')], limit=1):
            self.env.ref('account_peppol.ir_cron_peppol_get_participant_status')._trigger(at=fields.Datetime.now() + timedelta(hours=1))

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    def _get_proxy_identification(self, company, proxy_type):
        if proxy_type == 'peppol':
            if not company.peppol_eas or not company.peppol_endpoint:
                raise UserError(
                    _("Please fill in the EAS code and the Participant ID code."))
            return f'{company.peppol_eas}:{company.peppol_endpoint}'
        return super()._get_proxy_identification(company, proxy_type)

    def _peppol_import_invoice(self, attachment, partner_endpoint, peppol_state, uuid):
        """Save new documents in an accounting journal, when one is specified on the company.

        :param attachment: the new document
        :param partner_endpoint: DEPRECATED - to be removed in master
        :param peppol_state: the state of the received Peppol document
        :param uuid: the UUID of the Peppol document
        :return: `True` if the document was saved, `False` if it was not
        """
        self.ensure_one()
        journal = self.company_id.peppol_purchase_journal_id
        if not journal:
            return False

        move = self.env['account.move'].create({
            'journal_id': journal.id,
            'move_type': 'in_invoice',
            'peppol_move_state': peppol_state,
            'peppol_message_uuid': uuid,
        })
        if 'is_in_extractable_state' in move._fields:
            move.is_in_extractable_state = False

        move._extend_with_attachments(attachment, new=True)
        move._message_log(
            body=_(
                "Peppol document (UUID: %(uuid)s) has been received successfully",
                uuid=uuid,
            ),
            attachment_ids=attachment.ids,
        )
        attachment.write({'res_model': 'account.move', 'res_id': move.id})
        return True

    def _peppol_get_new_documents(self):
        params = {
            'domain': {
                'direction': 'incoming',
                'errors': False,
            }
        }
        for edi_user in self:
            params['domain']['receiver_identifier'] = edi_user.edi_identification
            try:
                # request all messages that haven't been acknowledged
                messages = edi_user._call_peppol_proxy(
                    "/api/peppol/1/get_all_documents",
                    params=params,
                )
            except AccountEdiProxyError as e:
                _logger.error(
                    'Error while receiving the document from Peppol Proxy: %s', e.message)
                continue

            message_uuids = [
                message['uuid']
                for message in messages.get('messages', [])
            ]
            if not message_uuids:
                continue

            for uuids in split_every(BATCH_SIZE, message_uuids):
                proxy_acks = []
                # retrieve attachments for filtered messages
                all_messages = edi_user._call_peppol_proxy(
                    "/api/peppol/1/get_document",
                    params={'message_uuids': uuids},
                )

                for uuid, content in all_messages.items():
                    enc_key = content["enc_key"]
                    document_content = content["document"]
                    filename = content["filename"] or 'attachment'  # default to attachment, which should not usually happen
                    decoded_document = edi_user._decrypt_data(document_content, enc_key)
                    attachment = self.env["ir.attachment"].create(
                        {
                            "name": f"{filename}.xml",
                            "raw": decoded_document,
                            "type": "binary",
                            "mimetype": "application/xml",
                        }
                    )
                    if edi_user._peppol_import_invoice(attachment, None, content["state"], uuid):
                        # Only acknowledge when we saved the document somewhere
                        proxy_acks.append(uuid)

                if not tools.config['test_enable']:
                    self.env.cr.commit()
                if proxy_acks:
                    edi_user._call_peppol_proxy(
                        "/api/peppol/1/ack",
                        params={'message_uuids': proxy_acks},
                    )

    def _peppol_get_message_status(self):
        for edi_user in self:
            edi_user_moves = self.env['account.move'].search([
                ('peppol_move_state', '=', 'processing'),
                ('company_id', '=', edi_user.company_id.id),
            ])
            if not edi_user_moves:
                continue

            message_uuids = {move.peppol_message_uuid: move for move in edi_user_moves}
            for uuids in split_every(BATCH_SIZE, message_uuids.keys()):
                messages_to_process = edi_user._call_peppol_proxy(
                    "/api/peppol/1/get_document",
                    params={'message_uuids': uuids},
                )

                for uuid, content in messages_to_process.items():
                    if uuid == 'error':
                        # this rare edge case can happen if the participant is not active on the proxy side
                        # in this case we can't get information about the invoices
                        edi_user_moves.peppol_move_state = 'error'
                        log_message = _("Peppol error: %s", content['message'])
                        edi_user_moves._message_log_batch(bodies={move.id: log_message for move in edi_user_moves})
                        break

                    move = message_uuids[uuid]
                    if content.get('error'):
                        # "Peppol request not ready" error:
                        # thrown when the IAP is still processing the message
                        if content['error'].get('code') == 702:
                            continue

                        move.peppol_move_state = 'error'
                        move._message_log(body=_("Peppol error: %s", content['error'].get('data', {}).get('message') or content['error']['message']))
                        continue

                    move.peppol_move_state = content['state']
                    move._message_log(body=_('Peppol status update: %s', content['state']))

                edi_user._call_peppol_proxy(
                    "/api/peppol/1/ack",
                    params={'message_uuids': uuids},
                )

    def _peppol_get_participant_status(self):
        for edi_user in self:
            try:
                proxy_user = edi_user._call_peppol_proxy("/api/peppol/2/participant_status")
            except AccountEdiProxyError as e:
                _logger.error('Error while updating Peppol participant status: %s', e)
                continue

            if proxy_user['peppol_state'] in ('sender', 'smp_registration', 'receiver', 'rejected'):
                edi_user.company_id.account_peppol_proxy_state = proxy_user['peppol_state']

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    @handle_demo
    def _peppol_migrate_registration(self):
        """Migrates AWAY from Odoo's SMP."""
        self.ensure_one()
        response = self._call_peppol_proxy(endpoint='/api/peppol/1/migrate_peppol_registration')
        if migration_key := response.get('migration_key'):
            self.company_id.account_peppol_migration_key = migration_key

    def _get_company_details(self):
        self.ensure_one()
        return {
            'peppol_company_name': self.company_id.display_name,
            'peppol_company_vat': self.company_id.vat,
            'peppol_company_street': self.company_id.street,
            'peppol_company_city': self.company_id.city,
            'peppol_company_zip': self.company_id.zip,
            'peppol_country_code': self.company_id.country_id.code,
            'peppol_phone_number': self.company_id.account_peppol_phone_number,
            'peppol_contact_email': self.company_id.account_peppol_contact_email,
            'peppol_migration_key': self.company_id.account_peppol_migration_key,
        }

    def _peppol_register_sender(self):
        self.ensure_one()
        params = {
            'company_details': self._get_company_details(),
        }
        self._call_peppol_proxy(
            endpoint='/api/peppol/1/register_sender',
            params=params,
        )
        self.company_id.account_peppol_proxy_state = 'sender'

    def _peppol_register_receiver(self):
        # remove in master
        self.ensure_one()
        params = {
            'company_details': self._get_company_details(),
            'supported_identifiers': list(self.company_id._peppol_supported_document_types())
        }
        self._call_peppol_proxy(
            endpoint='/api/peppol/1/register_receiver',
            params=params,
        )
        self.company_id.account_peppol_proxy_state = 'smp_registration'

    def _peppol_register_sender_as_receiver(self):
        self.ensure_one()
        company = self.company_id

        if company.account_peppol_proxy_state != 'sender':
            # a participant can only try registering as a receiver if they are currently a sender
            peppol_state_translated = dict(company._fields['account_peppol_proxy_state'].selection)[company.account_peppol_proxy_state]
            raise UserError(
                _('Cannot register a user with a %s application', peppol_state_translated))

        edi_identification = self._get_proxy_identification(company, 'peppol')
        self._check_company_on_peppol(company, edi_identification)

        self._call_peppol_proxy(
            endpoint='/api/peppol/1/register_sender_as_receiver',
            params={
                'migration_key': company.account_peppol_migration_key,
                'supported_identifiers': list(company._peppol_supported_document_types())
            },
        )
        # once we sent the migration key over, we don't need it
        # but we need the field for future in case the user decided to migrate away from Odoo
        company.account_peppol_migration_key = False
        company.account_peppol_proxy_state = 'smp_registration'

        self.env.ref('account_peppol.ir_cron_peppol_get_participant_status')._trigger(at=fields.Datetime.now() + timedelta(hours=1))

    def _peppol_deregister_participant(self):
        self.ensure_one()

        if self.company_id.account_peppol_proxy_state == 'receiver':
            # fetch all documents and message statuses before unlinking the edi user
            # so that the invoices are acknowledged
            self._cron_peppol_get_message_status()
            self._cron_peppol_get_new_documents()
            if not tools.config['test_enable'] and not modules.module.current_test:
                self.env.cr.commit()

        if self.company_id.account_peppol_proxy_state != 'not_registered':
            self._call_peppol_proxy(endpoint='/api/peppol/1/cancel_peppol_registration')

        self.company_id.account_peppol_proxy_state = 'not_registered'
        self.company_id.account_peppol_migration_key = False
        self.unlink()

    @api.model
    def _peppol_auto_register_services(self, module):
        """Register new document types for all recipient users.

        This function should be run in the post init hook of any module that extends the supported
        document types.

        :param module: Module from which this function is being called, allows us to determine which
            document types are now supported.
        """
        receivers = self.search([
            ('proxy_type', '=', 'peppol'),
            ('company_id.account_peppol_proxy_state', '=', 'receiver')
        ])
        supported_identifiers = list(self.env['res.company']._peppol_modules_document_types().get(module, {}))
        for receiver in receivers:
            try:
                receiver._call_peppol_proxy(
                    '/api/peppol/2/add_services',
                    params={'document_identifiers': supported_identifiers},
                )
            # Broad exception case, so as not to block execution of the rest of the _post_init hook.
            except (AccountEdiProxyError, UserError) as exception:
                _logger.error(
                    'Auto registration of peppol services for module: %s failed on the user: %s, with exception: %s',
                    module, receiver.edi_identification, exception,
                )

    @api.model
    def _peppol_auto_deregister_services(self, module):
        """Unregister a set of document types for all recipient users.

        This function should be run in the uninstall hook of any module that extends the supported
        document types.

        :param module: Module from which this function is being called, allows us to determine which
            document types are no longer supported.
        """
        receivers = self.search([
            ('proxy_type', '=', 'peppol'),
            ('company_id.account_peppol_proxy_state', '=', 'receiver')
        ])
        unsupported_identifiers = list(self.env['res.company']._peppol_modules_document_types().get(module, {}))
        for receiver in receivers:
            try:
                receiver._call_peppol_proxy(
                    '/api/peppol/2/remove_services',
                    params={'document_identifiers': unsupported_identifiers},
                )
            except (AccountEdiProxyError, UserError) as exception:
                _logger.error(
                    'Auto deregistration of peppol services for module: %s failed on the user: %s, with exception: %s',
                    module, receiver.edi_identification, exception,
                )

    def _peppol_get_services(self):
        """Get information from the IAP regarding the Peppol services."""
        self.ensure_one()
        return self._call_peppol_proxy("/api/peppol/2/get_services")

```

## File: models\account_journal.py

```python
from odoo import _, fields, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    account_peppol_proxy_state = fields.Selection(related='company_id.account_peppol_proxy_state')
    is_peppol_journal = fields.Boolean(string="Account used for Peppol", default=False)

    def peppol_get_new_documents(self):
        edi_users = self.env['account_edi_proxy_client.user'].search([
            ('company_id.account_peppol_proxy_state', '=', 'receiver'),
            ('company_id', 'in', self.company_id.ids),
            ('proxy_type', '=', 'peppol')
        ])
        edi_users._peppol_get_new_documents()

    def peppol_get_message_status(self):
        can_send = self.env['account_edi_proxy_client.user']._get_can_send_domain()
        edi_users = self.env['account_edi_proxy_client.user'].search([
            ('company_id.account_peppol_proxy_state', 'in', can_send),
            ('company_id', 'in', self.company_id.ids),
            ('proxy_type', '=', 'peppol')
        ])
        edi_users._peppol_get_message_status()

    def action_peppol_ready_moves(self):
        return {
            'name': _("Peppol Ready invoices"),
            'type': 'ir.actions.act_window',
            'view_mode': 'list,form',
            'res_model': 'account.move',
            'context': {
                'search_default_peppol_ready': 1,
            }
        }

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.addons.account.models.company import PEPPOL_DEFAULT_COUNTRIES


class AccountMove(models.Model):
    _inherit = 'account.move'

    peppol_message_uuid = fields.Char(string='PEPPOL message ID')
    peppol_move_state = fields.Selection(
        selection=[
            ('ready', 'Ready to send'),
            ('to_send', 'Queued'),
            ('skipped', 'Skipped'),  # TODO remove this state in master, we now put a regular error.
            ('processing', 'Pending Reception'),
            ('done', 'Done'),
            ('error', 'Error'),
        ],
        compute='_compute_peppol_move_state', store=True,
        string='PEPPOL status',
        copy=False,
    )

    def action_cancel_peppol_documents(self):
        # if the peppol_move_state is processing/done
        # then it means it has been already sent to peppol proxy and we can't cancel
        if any(move.peppol_move_state in {'processing', 'done'} for move in self):
            raise UserError(_("Cannot cancel an entry that has already been sent to PEPPOL"))
        self.peppol_move_state = False
        self.sending_data = False

    @api.depends('state')
    def _compute_peppol_move_state(self):
        can_send = self.env['account_edi_proxy_client.user']._get_can_send_domain()
        for move in self:
            if all([
                move.company_id.account_peppol_proxy_state in can_send,
                move.commercial_partner_id.peppol_verification_state == 'valid',
                move.state == 'posted',
                move.is_sale_document(include_receipts=True),
                not move.peppol_move_state,
            ]):
                move.peppol_move_state = 'ready'
            elif (
                move.state == 'draft'
                and move.is_sale_document(include_receipts=True)
                and move.peppol_move_state not in ('processing', 'done')
            ):
                move.peppol_move_state = False
            else:
                move.peppol_move_state = move.peppol_move_state

    def _notify_by_email_prepare_rendering_context(self, message, **kwargs):
        render_context = super()._notify_by_email_prepare_rendering_context(message, **kwargs)
        invoice = render_context['record']
        invoice_country = invoice.commercial_partner_id.country_code
        if invoice_country in PEPPOL_DEFAULT_COUNTRIES:
            render_context['peppol_info'] = {
                'peppol_country': invoice_country,
                'is_peppol_sent': invoice.peppol_move_state in ('processing', 'done'),
            }
        return render_context

```

## File: models\account_move_send.py

```python
from base64 import b64encode
from datetime import timedelta

from odoo import fields, models, _
from odoo.addons.account.models.company import PEPPOL_LIST
from odoo.addons.account_edi_proxy_client.models.account_edi_proxy_user import AccountEdiProxyError

class AccountMoveSend(models.AbstractModel):
    _inherit = 'account.move.send'

    # -------------------------------------------------------------------------
    # ALERTS
    # -------------------------------------------------------------------------

    def _get_alerts(self, moves, moves_data):
        # EXTENDS 'account'
        def peppol_partner(moves):
            return moves.partner_id.commercial_partner_id

        def filter_peppol_state(moves, states):
            return peppol_partner(moves.filtered(
                lambda m: self.env['res.partner']._get_peppol_verification_state(
                    peppol_partner(m).peppol_endpoint,
                    peppol_partner(m).peppol_eas,
                    moves_data[m]['invoice_edi_format']) in states))

        alerts = super()._get_alerts(moves, moves_data)
        # Check for invalid peppol partners.
        peppol_moves = moves.filtered(lambda m: 'peppol' in moves_data[m]['sending_methods'])
        invalid_partners = filter_peppol_state(peppol_moves, ['not_valid_format'])
        if invalid_partners and not 'account_edi_ubl_cii_configure_partner' in alerts:
            alerts['account_peppol_warning_partner'] = {
                'message': _("Customer is on Peppol but did not enable receiving documents."),
                'action_text': _("View Partner(s)"),
                'action': invalid_partners._get_records_action(name=_("Check Partner(s)")),
            }
        not_peppol_moves = moves.filtered(lambda m: 'peppol' not in moves_data[m]['sending_methods'])
        what_is_peppol_alert = {
            'level': 'info',
            'action_text': _("Why should you use it ?"),
            'action': {
                'name': _("Why should I use PEPPOL ?"),
                'type': 'ir.actions.client',
                'tag': 'account_peppol.what_is_peppol',
                'target': 'new',
                'context': {
                    'footer': False,
                    'dialog_size': 'medium',
                    'action_on_activate': self.action_what_is_peppol_activate(moves),
                },
            },
        }
        info_always_on_countries = {'BE', 'FI', 'LU', 'LV', 'NL', 'NO', 'SE'}
        can_send = self.env['account_edi_proxy_client.user']._get_can_send_domain()
        any_moves_not_sent_peppol = any(move.peppol_move_state not in ('processing', 'done') for move in moves)
        always_on_companies = moves.company_id.filtered(
            lambda c: c.country_code in info_always_on_countries and c.account_peppol_proxy_state not in can_send
        )
        if always_on_companies and any_moves_not_sent_peppol and not filter_peppol_state(moves, ['not_valid', 'not_verified']):
            alerts.pop('account_edi_ubl_cii_configure_company', False)
            alerts['account_peppol_what_is_peppol'] = {
                'message': _("You can send this invoice electronically via Peppol."),
                **what_is_peppol_alert,
            }
        elif (peppol_not_selected_partners := filter_peppol_state(not_peppol_moves, ['valid'])) and any_moves_not_sent_peppol:
            # Check for not peppol partners that are on the network.
            if len(peppol_not_selected_partners) == 1:
                alerts['account_peppol_partner_want_peppol'] = {
                    'message': _(
                        "%s has requested electronic invoices reception on Peppol.",
                        peppol_not_selected_partners.display_name
                    ),
                    **what_is_peppol_alert,
                }
        return alerts

    # -------------------------------------------------------------------------
    # SENDING METHODS
    # -------------------------------------------------------------------------

    def _get_default_invoice_edi_format(self, move, **kwargs) -> str:
        # EXTENDS 'account' - default on bis3 if Peppol is set but no format on the partner
        invoice_edi_format = super()._get_default_invoice_edi_format(move, **kwargs)
        if 'peppol' in kwargs.get('sending_methods', []):
            return move.partner_id.with_company(move.company_id)._get_peppol_edi_format()
        return invoice_edi_format

    def _get_mail_layout(self):
        # EXTENDS 'account'
        # TODO remove the fallback in master
        if self.env.ref('account_peppol.mail_notification_layout_with_responsible_signature_and_peppol',
                        raise_if_not_found=False):
            return 'account_peppol.mail_notification_layout_with_responsible_signature_and_peppol'
        return super()._get_mail_layout()

    def _do_peppol_pre_send(self, moves):
        if len(moves.company_id) == 1:
            can_send = self.env['account_edi_proxy_client.user']._get_can_send_domain()
            if moves.company_id.account_peppol_proxy_state not in can_send:
                registration_wizard = self.env['peppol.registration'].create({'company_id': moves.company_id.id})
                return registration_wizard._action_open_peppol_form(reopen=False)

        for move in moves:
            if move.peppol_move_state in ('ready', False):
                move.peppol_move_state = 'to_send'

    def _is_applicable_to_company(self, method, company):
        # EXTENDS 'account'
        if method == 'peppol':
            return company.country_code in PEPPOL_LIST and company.account_peppol_proxy_state != 'rejected'
        else:
            return super()._is_applicable_to_company(method, company)

    def _is_applicable_to_move(self, method, move, **move_data):
        # EXTENDS 'account'
        if method == 'peppol':
            partner = move.partner_id.commercial_partner_id.with_company(move.company_id)
            invoice_edi_format = move_data.get('invoice_edi_format') or partner._get_peppol_edi_format()
            return all([
                self._is_applicable_to_company(method, move.company_id),
                self.env['res.partner']._get_peppol_verification_state(partner.peppol_endpoint, partner.peppol_eas, invoice_edi_format) == 'valid',
                move.company_id.account_peppol_proxy_state != 'rejected',
                move._need_ubl_cii_xml(invoice_edi_format)
                or move.ubl_cii_xml_id and move.peppol_move_state not in ('processing', 'done'),
            ])
        else:
            return super()._is_applicable_to_move(method, move, **move_data)

    def _hook_if_errors(self, moves_data, allow_raising=True):
        # EXTENDS 'account'
        # to update `peppol_move_state` as `error` to show users that something went wrong
        # because those moves that failed XML/PDF files generation are not sent via Peppol
        moves_failed_file_generation = self.env['account.move']
        for move, move_data in moves_data.items():
            if 'peppol' in move_data['sending_methods'] and move_data.get('blocking_error'):
                moves_failed_file_generation |= move

        moves_failed_file_generation.peppol_move_state = 'error'

        return super()._hook_if_errors(moves_data, allow_raising=allow_raising)

    def _call_web_service_after_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_after_invoice_pdf_render(invoices_data)

        params = {'documents': []}
        invoices_data_peppol = {}
        for invoice, invoice_data in invoices_data.items():
            partner = invoice.partner_id.commercial_partner_id.with_company(invoice.company_id)
            if 'peppol' in invoice_data['sending_methods']:
                if not partner.peppol_eas or not partner.peppol_endpoint:
                    invoice.peppol_move_state = 'error'
                    invoice_data['error'] = _('The partner is missing Peppol EAS and/or Endpoint identifier.')
                    continue

                if self.env['res.partner']._get_peppol_verification_state(partner.peppol_endpoint, partner.peppol_eas, invoice_data['invoice_edi_format']) != 'valid':
                    invoice.peppol_move_state = 'error'
                    invoice_data['error'] = _('Please verify partner configuration in partner settings.')
                    continue

                if not self._is_applicable_to_move('peppol', invoice, **invoice_data):
                    continue

                if invoice_data.get('ubl_cii_xml_attachment_values'):
                    xml_file = invoice_data['ubl_cii_xml_attachment_values']['raw']
                    filename = invoice_data['ubl_cii_xml_attachment_values']['name']
                elif invoice.ubl_cii_xml_id and invoice.peppol_move_state not in ('processing', 'done'):
                    xml_file = invoice.ubl_cii_xml_id.raw
                    filename = invoice.ubl_cii_xml_id.name
                else:
                    invoice.peppol_move_state = 'error'
                    builder = invoice.partner_id.commercial_partner_id._get_edi_builder(invoice_data['invoice_edi_format'])
                    invoice_data['error'] = _(
                        "Errors occurred while creating the EDI document (format: %s):",
                        builder._description
                    )
                    continue

                receiver_identification = f"{partner.peppol_eas}:{partner.peppol_endpoint}"
                params['documents'].append({
                    'filename': filename,
                    'receiver': receiver_identification,
                    'ubl': b64encode(xml_file).decode(),
                })
                invoices_data_peppol[invoice] = invoice_data

        if not params['documents']:
            return

        edi_user = next(iter(invoices_data)).company_id.account_edi_proxy_client_ids.filtered(
            lambda u: u.proxy_type == 'peppol')

        try:
            response = edi_user._call_peppol_proxy(
                "/api/peppol/1/send_document",
                params=params,
            )
        except AccountEdiProxyError as e:
            for invoice, invoice_data in invoices_data_peppol.items():
                invoice.peppol_move_state = 'error'
                invoice_data['error'] = e.message
        else:
            if response.get('error'):
                # at the moment the only error that can happen here is ParticipantNotReady error
                for invoice, invoice_data in invoices_data_peppol.items():
                    invoice.peppol_move_state = 'error'
                    invoice_data['error'] = response['error']['message']
            else:
                # the response only contains message uuids,
                # so we have to rely on the order to connect peppol messages to account.move
                invoices = self.env['account.move']
                for message, (invoice, invoice_data) in zip(response['messages'], invoices_data_peppol.items()):
                    invoice.peppol_message_uuid = message['message_uuid']
                    invoice.peppol_move_state = 'processing'
                    invoices |= invoice
                log_message = _('The document has been sent to the Peppol Access Point for processing')
                invoices._message_log_batch(bodies={invoice.id: log_message for invoice in invoices})
                self.env.ref('account_peppol.ir_cron_peppol_get_message_status')._trigger(at=fields.Datetime.now() + timedelta(minutes=5))

        if self._can_commit():
            self._cr.commit()

    def action_what_is_peppol_activate(self, moves):
        companies = moves.company_id
        can_send = self.env['account_edi_proxy_client.user']._get_can_send_domain()
        if len(companies) == 1 and companies.account_peppol_proxy_state not in can_send:
            action = self.env['peppol.registration']._action_open_peppol_form()
            action['context'].update({
                'active_model': 'account.move',
                'active_ids': moves.ids,
                'dialog_size': 'medium',
            })
            return action
        else:
            return moves.action_send_and_print()

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from stdnum import get_cc_module, ean

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.addons.account.models.company import PEPPOL_LIST

try:
    import phonenumbers
except ImportError:
    phonenumbers = None


def _cc_checker(country_code, code_type):
    return lambda endpoint: get_cc_module(country_code, code_type).is_valid(endpoint)


def _re_sanitizer(expression):
    return lambda endpoint: (res.group(0) if (res := re.search(expression, endpoint)) else endpoint)


PEPPOL_ENDPOINT_RULES = {
    '0007': _cc_checker('se', 'orgnr'),
    '0088': ean.is_valid,
    '0184': _cc_checker('dk', 'cvr'),
    '0192': _cc_checker('no', 'orgnr'),
    '0208': _cc_checker('be', 'vat'),
}

PEPPOL_ENDPOINT_WARNINGS = {
    '0151': _cc_checker('au', 'abn'),
    '0201': lambda endpoint: bool(re.match('[0-9a-zA-Z]{6}$', endpoint)),
    '0210': _cc_checker('it', 'codicefiscale'),
    '0211': _cc_checker('it', 'iva'),
    '9906': _cc_checker('it', 'iva'),
    '9907': _cc_checker('it', 'codicefiscale'),
}

PEPPOL_ENDPOINT_SANITIZERS = {
    '0007': _re_sanitizer(r'\d{10}'),
    '0184': _re_sanitizer(r'\d{8}'),
    '0192': _re_sanitizer(r'\d{9}'),
    '0208': _re_sanitizer(r'\d{10}'),
}


class ResCompany(models.Model):
    _inherit = 'res.company'

    account_peppol_contact_email = fields.Char(
        string='Primary contact email',
        compute='_compute_account_peppol_contact_email', store=True, readonly=False,
        help='Primary contact email for Peppol-related communication',
    )
    account_peppol_migration_key = fields.Char(string="Migration Key")
    account_peppol_phone_number = fields.Char(
        string='Mobile number',
        compute='_compute_account_peppol_phone_number', store=True, readonly=False,
        help='You will receive a verification code to this mobile number',
    )
    account_peppol_proxy_state = fields.Selection(
        selection=[
            ('not_registered', 'Not registered'),
            ('in_verification', 'In verification'),
            ('sender', 'Can send but not receive'),
            ('smp_registration', 'Can send, pending registration to receive'),
            ('receiver', 'Can send and receive'),
            ('rejected', 'Rejected'),
        ],
        string='PEPPOL status', required=True, default='not_registered',
    )
    peppol_eas = fields.Selection(related='partner_id.peppol_eas', readonly=False)
    peppol_endpoint = fields.Char(related='partner_id.peppol_endpoint', readonly=False)
    peppol_purchase_journal_id = fields.Many2one(
        comodel_name='account.journal',
        string='PEPPOL Purchase Journal',
        domain=[('type', '=', 'purchase')],
        compute='_compute_peppol_purchase_journal_id', store=True, readonly=False,
        inverse='_inverse_peppol_purchase_journal_id',
    )

    # -------------------------------------------------------------------------
    # HELPER METHODS
    # -------------------------------------------------------------------------

    def _sanitize_peppol_phone_number(self, phone_number=None):
        self.ensure_one()

        error_message = _(
            "Please enter the mobile number in the correct international format.\n"
            "For example: +32123456789, where +32 is the country code.\n"
            "Currently, only European countries are supported.")

        if not phonenumbers:
            raise ValidationError(_("Please install the phonenumbers library."))

        phone_number = phone_number or self.account_peppol_phone_number
        if not phone_number:
            return

        if not phone_number.startswith('+'):
            phone_number = f'+{phone_number}'

        try:
            phone_nbr = phonenumbers.parse(phone_number)
        except phonenumbers.phonenumberutil.NumberParseException:
            raise ValidationError(error_message)

        country_code = phonenumbers.phonenumberutil.region_code_for_number(phone_nbr)
        if country_code not in PEPPOL_LIST or not phonenumbers.is_valid_number(phone_nbr):
            raise ValidationError(error_message)

    def _check_peppol_endpoint_number(self, warning=False):
        self.ensure_one()
        peppol_dict = PEPPOL_ENDPOINT_WARNINGS if warning else PEPPOL_ENDPOINT_RULES

        return True if (endpoint_rule := peppol_dict.get(self.peppol_eas)) is None else endpoint_rule(self.peppol_endpoint)

    # -------------------------------------------------------------------------
    # CONSTRAINTS
    # -------------------------------------------------------------------------

    @api.constrains('account_peppol_phone_number')
    def _check_account_peppol_phone_number(self):
        for company in self:
            if company.account_peppol_phone_number:
                company._sanitize_peppol_phone_number()

    @api.constrains('peppol_endpoint')
    def _check_peppol_endpoint(self):
        for company in self:
            if not company.peppol_endpoint:
                continue
            if not company._check_peppol_endpoint_number(PEPPOL_ENDPOINT_RULES):
                raise ValidationError(_("The Peppol endpoint identification number is not correct."))

    @api.constrains('peppol_purchase_journal_id')
    def _check_peppol_purchase_journal_id(self):
        for company in self:
            if company.peppol_purchase_journal_id and company.peppol_purchase_journal_id.type != 'purchase':
                raise ValidationError(_("A purchase journal must be used to receive Peppol documents."))

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('account_peppol_proxy_state')
    def _compute_peppol_purchase_journal_id(self):
        for company in self:
            if not company.peppol_purchase_journal_id and company.account_peppol_proxy_state not in ('not_registered', 'rejected'):
                company.peppol_purchase_journal_id = self.env['account.journal'].search([
                    *self.env['account.journal']._check_company_domain(company),
                    ('type', '=', 'purchase'),
                ], limit=1)
                company.peppol_purchase_journal_id.is_peppol_journal = True
            else:
                company.peppol_purchase_journal_id = company.peppol_purchase_journal_id

    def _inverse_peppol_purchase_journal_id(self):
        for company in self:
            # This avoid having 2 or more journals from the same company with
            # `is_peppol_journal` set to True (which could occur after changes).
            journals_to_reset = self.env['account.journal'].search([
                ('company_id', '=', company.id),
                ('is_peppol_journal', '=', True),
            ])
            journals_to_reset.is_peppol_journal = False
            company.peppol_purchase_journal_id.is_peppol_journal = True

    @api.depends('email')
    def _compute_account_peppol_contact_email(self):
        for company in self:
            if not company.account_peppol_contact_email:
                company.account_peppol_contact_email = company.email

    @api.depends('phone')
    def _compute_account_peppol_phone_number(self):
        for company in self:
            if not company.account_peppol_phone_number:
                try:
                    # precompute only if it's a valid phone number
                    company._sanitize_peppol_phone_number(company.phone)
                    company.account_peppol_phone_number = company.phone
                except ValidationError:
                    continue

    # -------------------------------------------------------------------------
    # LOW-LEVEL METHODS
    # -------------------------------------------------------------------------

    @api.model
    def _sanitize_peppol_endpoint(self, vals, eas=False, endpoint=False):
        # TODO: remove in master
        if not (peppol_eas := vals.get('peppol_eas', eas)) or not (peppol_endpoint := vals.get('peppol_endpoint', endpoint)):
            return vals

        if sanitizer := PEPPOL_ENDPOINT_SANITIZERS.get(peppol_eas):
            vals['peppol_endpoint'] = sanitizer(peppol_endpoint)

        return vals

    @api.model
    def _sanitize_peppol_endpoint_in_values(self, values):
        eas = values.get('peppol_eas')
        endpoint = values.get('peppol_endpoint')
        if not eas or not endpoint:
            return
        if sanitizer := PEPPOL_ENDPOINT_SANITIZERS.get(eas):
            new_endpoint = sanitizer(endpoint)
            if new_endpoint:
                values['peppol_endpoint'] = new_endpoint

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            self._sanitize_peppol_endpoint_in_values(vals)

        res = super().create(vals_list)
        if res:
            for company in res:
                self.env['ir.default'].sudo().set(
                    'res.partner',
                    'peppol_verification_state',
                    'not_verified',
                    company_id=company.id,
                )
        return res

    def write(self, vals):
        self._sanitize_peppol_endpoint_in_values(vals)
        return super().write(vals)

    # -------------------------------------------------------------------------
    # PEPPOL PARTICIPANT MANAGEMENT
    # -------------------------------------------------------------------------

    def _peppol_modules_document_types(self):
        """Override this function to add supported document types as modules are installed.

        :returns: dictionary of the form: {module_name: [(document identifier, document_name)]}
        """
        return {
            'default': {
                "urn:oasis:names:specification:ubl:schema:xsd:Invoice-2::Invoice##urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0::2.1":
                    "Peppol BIS Billing UBL Invoice V3",
                "urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2::CreditNote##urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0::2.1":
                    "Peppol BIS Billing UBL CreditNote V3",
                "urn:oasis:names:specification:ubl:schema:xsd:Invoice-2::Invoice##urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0::2.1":
                    "SI-UBL 2.0 Invoice",
                "urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2::CreditNote##urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0::2.1":
                    "SI-UBL 2.0 CreditNote",
                "urn:oasis:names:specification:ubl:schema:xsd:Invoice-2::Invoice##urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:sg:3.0::2.1":
                    "SG Peppol BIS Billing 3.0 Invoice",
                "urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2::CreditNote##urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:sg:3.0::2.1":
                    "SG Peppol BIS Billing 3.0 Credit Note",
                "urn:oasis:names:specification:ubl:schema:xsd:Invoice-2::Invoice##urn:cen.eu:en16931:2017#compliant#urn:xeinkauf.de:kosit:xrechnung_3.0::2.1":
                    "XRechnung UBL Invoice V2.0",
                "urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2::CreditNote##urn:cen.eu:en16931:2017#compliant#urn:xeinkauf.de:kosit:xrechnung_3.0::2.1":
                    "XRechnung UBL CreditNote V2.0",
                "urn:oasis:names:specification:ubl:schema:xsd:Invoice-2::Invoice##urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:aunz:3.0::2.1":
                    "AU-NZ Peppol BIS Billing 3.0 Invoice",
                "urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2::CreditNote##urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:aunz:3.0::2.1":
                    "AU-NZ Peppol BIS Billing 3.0 CreditNote",
            }
        }

    def _peppol_supported_document_types(self):
        """Returns a flattened dictionary of all supported document types."""
        return {
            identifier: document_name
            for module, identifiers in self._peppol_modules_document_types().items()
            for identifier, document_name in identifiers.items()
        }

    def _get_peppol_edi_mode(self):
        self.ensure_one()
        config_param = self.env['ir.config_parameter'].sudo().get_param('account_peppol.edi.mode')
        # by design, we can only have zero or one proxy user per company with type Peppol
        peppol_user = self.sudo().account_edi_proxy_client_ids.filtered(lambda u: u.proxy_type == 'peppol')
        return peppol_user.edi_mode or config_param or 'prod'

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models, modules, tools
from odoo.exceptions import UserError, ValidationError

from odoo.addons.account_peppol.tools.demo_utils import handle_demo


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    account_peppol_edi_user = fields.Many2one(
        comodel_name='account_edi_proxy_client.user',
        string='EDI user',
        compute='_compute_account_peppol_edi_user',
    )
    account_peppol_edi_mode = fields.Selection(related='account_peppol_edi_user.edi_mode')
    account_peppol_contact_email = fields.Char(related='company_id.account_peppol_contact_email', readonly=False)
    account_peppol_eas = fields.Selection(related='company_id.peppol_eas', readonly=False)
    account_peppol_edi_identification = fields.Char(related='account_peppol_edi_user.edi_identification')
    account_peppol_endpoint = fields.Char(related='company_id.peppol_endpoint', readonly=False)
    account_peppol_migration_key = fields.Char(related='company_id.account_peppol_migration_key', readonly=False)
    account_peppol_phone_number = fields.Char(related='company_id.account_peppol_phone_number', readonly=False)
    account_peppol_proxy_state = fields.Selection(related='company_id.account_peppol_proxy_state', readonly=False)
    account_peppol_purchase_journal_id = fields.Many2one(related='company_id.peppol_purchase_journal_id', readonly=False)

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('is_account_peppol_eligible', 'account_peppol_edi_user')
    def _compute_account_peppol_mode_constraint(self):
        mode_constraint = self.env['ir.config_parameter'].sudo().get_param('account_peppol.mode_constraint')
        trial_param = self.env['ir.config_parameter'].sudo().get_param('saas_trial.confirm_token')
        self.account_peppol_mode_constraint = trial_param and 'demo' or mode_constraint or 'prod'

    @api.depends("company_id.account_edi_proxy_client_ids")
    def _compute_account_peppol_edi_user(self):
        for config in self:
            config.account_peppol_edi_user = config.company_id.account_edi_proxy_client_ids.filtered(
                lambda u: u.proxy_type == 'peppol')

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    def action_open_peppol_form(self):
        registration_wizard = self.env['peppol.registration'].create({'company_id': self.company_id.id})
        registration_action = registration_wizard._action_open_peppol_form(reopen=False)
        return registration_action

    @handle_demo
    def button_update_peppol_user_data(self):
        """
        Action for the user to be able to update their contact details any time
        Calls /update_user on the iap server
        """
        self.ensure_one()

        if not self.account_peppol_contact_email or not self.account_peppol_phone_number:
            raise ValidationError(_("Contact email and mobile number are required."))

        params = {
            'update_data': {
                'peppol_phone_number': self.account_peppol_phone_number,
                'peppol_contact_email': self.account_peppol_contact_email,
            }
        }

        self.account_peppol_edi_user._call_peppol_proxy(
            endpoint='/api/peppol/1/update_user',
            params=params,
        )
        return True

    @handle_demo
    def button_peppol_smp_registration(self):
        """
        The second (optional) step in Peppol registration.
        The user can choose to become a Receiver and officially register on the Peppol
        network, i.e. receive documents from other Peppol participants.
        """
        self.ensure_one()
        self.account_peppol_edi_user._peppol_register_sender_as_receiver()
        if self.account_peppol_proxy_state == 'smp_registration':
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'title': _("Registered to receive documents via Peppol."),
                    'type': 'success',
                    'message': _("Your registration on Peppol network should be activated within a day. The updated status will be visible in Settings."),
                    'next': {'type': 'ir.actions.act_window_close'},
                }
            }
        return True

    def button_migrate_peppol_registration(self):
        """
        Migrates AWAY from Odoo's SMP.
        If the user is a receiver, they need to request a migration key, generated on the IAP server.
        The migration key is then displayed in Peppol settings.
        Currently, reopening after migrating away is not supported.
        """
        raise UserError(_("This feature is deprecated. Contact odoo support if you need a migration key."))

    @handle_demo
    def button_deregister_peppol_participant(self):
        """
        Deregister the edi user from Peppol network
        """
        self.ensure_one()

        if self.account_peppol_edi_user:
            self.account_peppol_edi_user._peppol_deregister_participant()
        return True

    def button_account_peppol_configure_services(self):
        wizard = self.env['account_peppol.service.wizard'].create({
            'edi_user_id': self.account_peppol_edi_user.id,
            'service_json': self.account_peppol_edi_user._peppol_get_services().get('services'),
        })
        return {
            'type': 'ir.actions.act_window',
            'name': 'Configure your peppol services',
            'res_model': 'account_peppol.service.wizard',
            'res_id': wizard.id,
            'view_mode': 'form',
            'target': 'new',
        }

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import contextlib
import logging
import requests
from lxml import etree
from markupsafe import Markup
from hashlib import md5
from urllib import parse

from odoo import api, fields, models
from odoo.addons.account_peppol.tools.demo_utils import handle_demo
from odoo.addons.account.models.company import PEPPOL_LIST

TIMEOUT = 10
_logger = logging.getLogger(__name__)



class ResPartner(models.Model):
    _inherit = 'res.partner'

    invoice_sending_method = fields.Selection(
        selection_add=[('peppol', 'by Peppol')],
    )
    available_peppol_sending_methods = fields.Json(compute='_compute_available_peppol_sending_methods')
    available_peppol_edi_formats = fields.Json(compute='_compute_available_peppol_edi_formats')
    peppol_verification_state = fields.Selection(
        selection=[
            ('not_verified', 'Not verified yet'),
            ('not_valid', 'Not on Peppol'),  # does not exist on Peppol at all
            ('not_valid_format', 'Cannot receive this format'),  # registered on Peppol but cannot receive the selected document type
            ('valid', 'Valid'),
        ],
        string='Peppol endpoint verification',
        company_dependent=True,
    )

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends_context('company')
    @api.depends('company_id')
    def _compute_available_peppol_sending_methods(self):
        methods = dict(self._fields['invoice_sending_method'].selection)
        if self.env.company.country_code not in PEPPOL_LIST:
            methods.pop('peppol')
        self.available_peppol_sending_methods = list(methods)

    @api.depends_context('company')
    @api.depends('invoice_sending_method')
    def _compute_available_peppol_edi_formats(self):
        for partner in self:
            if partner.invoice_sending_method == 'peppol':
                partner.available_peppol_edi_formats = self._get_peppol_formats()
            else:
                partner.available_peppol_edi_formats = list(dict(self._fields['invoice_edi_format'].selection))

    # -------------------------------------------------------------------------
    # HELPERS
    # -------------------------------------------------------------------------

    def _log_verification_state_update(self, company, old_value, new_value):
        # log the update of the peppol verification state
        # we do this instead of regular tracking because of the customized message
        # and because we want to log the change for every company in the db
        if old_value == new_value:
            return

        peppol_verification_state_field = self._fields['peppol_verification_state']
        selection_values = dict(peppol_verification_state_field.selection)
        old_label = selection_values[old_value] if old_value else False  # get translated labels
        new_label = selection_values[new_value] if new_value else False

        body = Markup("""
            <ul>
                <li>
                    <span class='o-mail-Message-trackingOld me-1 px-1 text-muted fw-bold'>{old}</span>
                    <i class='o-mail-Message-trackingSeparator fa fa-long-arrow-right mx-1 text-600'/>
                    <span class='o-mail-Message-trackingNew me-1 fw-bold text-info'>{new}</span>
                    <span class='o-mail-Message-trackingField ms-1 fst-italic text-muted'>({field})</span>
                    <span class='o-mail-Message-trackingCompany ms-1 fst-italic text-muted'>({company})</span>
                </li>
            </ul>
        """).format(
            old=old_label,
            new=new_label,
            field=peppol_verification_state_field.string,
            company=company.display_name,
        )
        self._message_log(body=body)

    @api.model
    def _get_participant_info(self, edi_identification):
        hash_participant = md5(edi_identification.lower().encode()).hexdigest()
        endpoint_participant = parse.quote_plus(f"iso6523-actorid-upis::{edi_identification}")
        edi_mode = self.env.company._get_peppol_edi_mode()
        sml_zone = 'acc.edelivery' if edi_mode == 'test' else 'edelivery'
        smp_url = f"http://B-{hash_participant}.iso6523-actorid-upis.{sml_zone}.tech.ec.europa.eu/{endpoint_participant}"

        try:
            response = requests.get(smp_url, timeout=TIMEOUT)
            response.raise_for_status()
        except requests.exceptions.RequestException as e:
            _logger.debug(e)
            return None
        return etree.fromstring(response.content)

    @api.model
    def _check_peppol_participant_exists(self, participant_info, edi_identification, check_company=False):
        participant_identifier = participant_info.findtext('{*}ParticipantIdentifier')
        service_metadata = participant_info.find('.//{*}ServiceMetadataReference')
        service_href = ''
        if service_metadata is not None:
            service_href = service_metadata.attrib.get('href', '')

        if edi_identification != participant_identifier or 'hermes-belgium' in service_href:
            # all Belgian companies are pre-registered on hermes-belgium, so they will
            # technically have an existing SMP url but they are not real Peppol participants
            return False

        if check_company:
            # if we are only checking company's existence on the network, we don't care about what documents they can receive
            if not service_href:
                return True

            access_point_contact = True
            with contextlib.suppress(requests.exceptions.RequestException, etree.XMLSyntaxError):
                response = requests.get(service_href, timeout=TIMEOUT)
                if response.status_code == 200:
                    access_point_info = etree.fromstring(response.content)
                    access_point_contact = access_point_info.findtext('.//{*}TechnicalContactUrl') or access_point_info.findtext('.//{*}TechnicalInformationUrl')
            return access_point_contact

        return True

    def _check_document_type_support(self, participant_info, ubl_cii_format):
        service_references = participant_info.findall(
            '{*}ServiceMetadataReferenceCollection/{*}ServiceMetadataReference'
        )
        document_type = self.env['account.edi.xml.ubl_21']._get_customization_ids()[ubl_cii_format]
        for service in service_references:
            if document_type in parse.unquote_plus(service.attrib.get('href', '')):
                return True
        return False

    def _update_peppol_state_per_company(self, vals=None):
        partners = self.env['res.partner']
        if vals is None:
            partners = self.filtered(lambda p: all([p.peppol_eas, p.peppol_endpoint, p.is_ubl_format, p.country_code in PEPPOL_LIST]))
        elif {'peppol_eas', 'peppol_endpoint', 'invoice_edi_format'}.intersection(vals.keys()):
            partners = self.filtered(lambda p: p.country_code in PEPPOL_LIST)

        all_companies = None
        for partner in partners.sudo():
            if partner.company_id:
                partner.button_account_peppol_check_partner_endpoint(company=partner.company_id)
                continue

            if all_companies is None:
                all_companies = self.env['res.company'].sudo().search([])

            for company in all_companies:
                partner.button_account_peppol_check_partner_endpoint(company=company)

    # -------------------------------------------------------------------------
    # LOW-LEVEL METHODS
    # -------------------------------------------------------------------------

    def write(self, vals):
        res = super().write(vals)
        self._update_peppol_state_per_company(vals=vals)
        return res

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        if res:
            res._update_peppol_state_per_company()
        return res

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    @handle_demo
    def button_account_peppol_check_partner_endpoint(self, company=None):
        """ A basic check for whether a participant is reachable at the given
        Peppol participant ID - peppol_eas:peppol_endpoint (ex: '9999:test')
        The SML (Service Metadata Locator) assigns a DNS name to each peppol participant.
        This DNS name resolves into the SMP (Service Metadata Publisher) of the participant.
        The DNS address is of the following form:
        - "http://B-" + hexstring(md5(lowercase(ID-VALUE))) + "." + ID-SCHEME + "." + SML-ZONE-NAME + "/" + url_encoded(ID-SCHEME + "::" + ID-VALUE)
        (ref:https://peppol.helger.com/public/locale-en_US/menuitem-docs-doc-exchange)
        """
        self.ensure_one()
        if not company:
            company = self.env.company

        self_partner = self.with_company(company)
        old_value = self_partner.peppol_verification_state
        self_partner.peppol_verification_state = self._get_peppol_verification_state(
            self.peppol_endpoint,
            self.peppol_eas,
            self_partner._get_peppol_edi_format(),
        )
        if self_partner.peppol_verification_state == 'valid':
            self_partner.invoice_sending_method = 'peppol'

        self._log_verification_state_update(company, old_value, self_partner.peppol_verification_state)
        return False

    @api.model
    @handle_demo
    def _get_peppol_verification_state(self, peppol_endpoint, peppol_eas, invoice_edi_format):
        if not (peppol_eas and peppol_endpoint) or invoice_edi_format not in self._get_peppol_formats():
            return 'not_verified'

        edi_identification = f"{peppol_eas}:{peppol_endpoint}".lower()
        participant_info = self._get_participant_info(edi_identification)
        if participant_info is None:
            return 'not_valid'
        else:
            is_participant_on_network = self._check_peppol_participant_exists(participant_info, edi_identification)
            if is_participant_on_network:
                is_valid_format = self._check_document_type_support(participant_info, invoice_edi_format)
                if is_valid_format:
                    return 'valid'
                else:
                    return 'not_valid_format'
            else:
                return 'not_valid'

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_edi_proxy_user
from . import account_journal
from . import account_move
from . import account_move_send
from . import res_company
from . import res_config_settings
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_peppol_registration,access.peppol.registration,model_peppol_registration,account.group_account_invoice,1,1,1,1
account_peppol_service_wizard_system,account_peppol.service.wizard,model_account_peppol_service_wizard,account.group_account_invoice,1,1,1,1
account_peppol_service_system,account_peppol.service,model_account_peppol_service,account.group_account_invoice,1,1,1,1

```

## File: static\src\components\peppol_info\peppol_info.js

```javascript
/** @odoo-module **/
import { Component } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import {_t} from "@web/core/l10n/translation";
import { standardActionServiceProps } from "@web/webclient/actions/action_service";


class WhatIsPeppol extends Component {
    static props = { ...standardActionServiceProps };
    static template = "account_peppol.WhatIsPeppol";

    setup() {
        super.setup();
        this.actionService = useService("action");
    }

    closeButtonLabel() {
        if (this.props.action.context.action_on_activate.res_model === "peppol.registration") {
            return _t("Activate")
        } else {
            return _t("Got it !")
        }
    }

    activate() {
        const action = this.props.action.context.action_on_activate;
        this.actionService.doAction({
            name: action.name,
            type: action.type,
            res_model: action.res_model,
            views: [[false, action.view_mode]],
            target: action.target,
            context: action.context,
        });
    }
}

registry.category("actions").add("account_peppol.what_is_peppol", WhatIsPeppol);

```

## File: static\src\components\peppol_info\peppol_info.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="account_peppol.WhatIsPeppol">
        <div class="modal-content shadow">
            <div class="modal-body p-5">
                <h2 class="fw-bold mb-1 text-center">What is Peppol and why it's great ?</h2>
                <ul class="d-grid gap-4 my-4 list-unstyled small">
                    <li class="d-flex gap-4">
                        <div>
                            <h5 class="mb-0">The e-invoicing network</h5>
                            PEPPOL is the secure standard for e-invoices used in EU and across the world.
                        </div>
                    </li>
                    <li class="d-flex gap-4">
                        <div>
                            <h5 class="mb-0">Fully automated</h5>
                            PEPPOL allows for complete automation of sending and receiving e-invoices.
                        </div>
                    </li>
                    <li class="d-flex gap-4">
                        <div>
                            <h5 class="mb-0">Free on Odoo</h5>
                            Create, send and receive e-invoices for free.
                        </div>
                    </li>
                    <li class="d-flex gap-4">
                        <div>
                            <h5 class="mb-0">E-invoices will soon be mandatory in many countries</h5>
                            Odoo keeps you up to date with the new regulation.
                        </div>
                    </li>
                </ul>
                <button class="btn btn-lg btn-primary mt-3 w-100" t-on-click="activate" t-out="closeButtonLabel()"/>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\components\peppol_radio_field\peppol_radio_field.js

```javascript
/** @odoo-module **/

import { evaluateExpr } from "@web/core/py_js/py";
import { RadioField, radioField } from "@web/views/fields/radio/radio_field";
import { registry } from "@web/core/registry";

class PeppolRadioField extends RadioField {
    static template = "account_peppol.PeppolRadioField";
    static props = {
        ...RadioField.props,
        hiddenItems: { type: String, optional: true },
        readonlyItems: { type: String, optional: true },
    };

    setup() {
        super.setup()
        this.initialSelection = this.props.record.data[this.props.name];
        this.readonlyItems = evaluateExpr(this.props.readonlyItems || "[]", this.props.record.evalContext);
        this.hiddenItems = evaluateExpr(this.props.hiddenItems || "[]", this.props.record.evalContext).filter(item => (item != this.initialSelection));
    }

    get items() {
        return super.items.filter(item => !(this.hiddenItems.includes(item[0])))
    }
}

const peppolRadioField = {
    ...radioField,
    component: PeppolRadioField,
    extractProps: ({attrs, options}, dynamicInfo) => {
        return {
            hiddenItems: attrs.hidden_items || "[]",
            readonlyItems: attrs.readonly_items  || "[]",
            ...radioField.extractProps({attrs, options}, dynamicInfo),
        }
    },
}
registry.category("fields").add("account_peppol_radio_field", peppolRadioField);

```

## File: static\src\components\peppol_radio_field\peppol_radio_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="account_peppol.PeppolRadioField" t-inherit-mode="primary" t-inherit="web.RadioField">
        <xpath expr="//input" position="attributes">
            <attribute name="t-att-disabled" add="|| this.readonlyItems.includes(item[0])" separator=" "/>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\res_config_settings_buttons\res_config_settings_buttons.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { escape } from "@web/core/utils/strings";
import { registry } from "@web/core/registry";
import { pick } from "@web/core/utils/objects";
import { useService } from "@web/core/utils/hooks";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

import { Component, markup, useState } from "@odoo/owl";


class PeppolSettingsButtons extends Component {
    static props = {
        ...standardWidgetProps,
    };
    static template = "account_peppol.ActionButtons";

    setup() {
        super.setup();
        this.dialogService = useService("dialog");
        this.notification = useService("notification");
        // we have to pass this via context from python
        // because the wizard has to be reopened whenever a button is clicked
        this.state = useState({
            isSmsButtonDisabled: this.props.record.context.disable_sms_verification || false,  // TODO remove in master
            isSettingsView: this.props.record.resModel === 'res.config.settings',
        });
    }

    get proxyState() {
        return this.props.record.data.account_peppol_proxy_state;
    }

    get migrationPrepared() {
        return this.props.record.data.account_peppol_proxy_state === "receiver" && Boolean(this.props.record.data.account_peppol_migration_key);
    }

    get ediMode() {
        return this.props.record.data.edi_mode || this.props.record.data.account_peppol_edi_mode;
    }

    get modeConstraint() {
        return this.props.record.data.mode_constraint;
    }

    get smpRegistration() {
        return this.props.record.data.smp_registration;
    }

    get createButtonLabel() {
        const modes = {
            demo: _t("Activate Peppol (Demo)"),
            test: _t("Activate Peppol (Test)"),
            prod: _t("Activate Peppol"),
        }
        return modes[this.ediMode];
    }

    get deregisterUserButtonLabel() {
        if (['not_registered', 'in_verification'].includes(this.proxyState)) {
            return _t("Discard");
        }
        return _t("Remove from Peppol");
    }

    async _callConfigMethod(methodName) {
        this.env.onClickViewButton({
            clickParams: {
                name: methodName,
                type: "object",
            },
            getResParams: () =>
                pick(this.env.model.root, "context", "evalContext", "resModel", "resId", "resIds"),
        });
    }

    showConfirmation(warning, methodName) {
        const message = _t(warning);
        const confirmMessage = _t("You will not be able to send or receive Peppol documents in Odoo anymore. Are you sure you want to proceed?");
        this.dialogService.add(ConfirmationDialog, {
            body: markup(
                `<div class="text-danger">${escape(message)}</div>
                <div class="text-danger">${escape(confirmMessage)}</div>`
            ),
            confirm: async () => {
                await this._callConfigMethod(methodName);
            },
            cancel: () => { },
        });
    }

    deregister() {
        if (this.ediMode === 'demo' || !['sender', 'smp_registration', 'receiver'].includes(this.proxyState)) {
            this._callConfigMethod("button_deregister_peppol_participant");
        } else if (['sender', 'smp_registration', 'receiver'].includes(this.proxyState)) {
            this.showConfirmation(
                "This will delete your Peppol registration.",
                "button_deregister_peppol_participant"
            )
        }
    }

    async updateDetails() {
        // avoid making users click save on the settings
        // and then clicking the update button
        // changes on both the client side and the iap side need to be saved within one method
        await this._callConfigMethod("button_update_peppol_user_data", true);
        this.notification.add(
            _t("Contact details were updated."),
            { type: "success" }
        );
    }

    async checkCode() {
        // avoid making users click save on the settings
        // and then clicking the confirm button to check the code
        await this._callConfigMethod("button_peppol_sender_registration");
    }

    async sendCode() {
        await this._callConfigMethod("button_send_peppol_verification_code");
    }

    async createReceiver() {
        await this._callConfigMethod("button_peppol_smp_registration");
    }
}

registry.category("view_widgets").add("peppol_settings_buttons", {
    component: PeppolSettingsButtons,
});

```

## File: static\src\components\res_config_settings_buttons\res_config_settings_buttons.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="account_peppol.ActionButtons">
        <xpath expr="//div[hasclass('action_buttons')]" position="inside">
            <div class="d-flex" colspan="3">
                <div class="mt-3">
                    <button type="button"
                            class="btn btn-primary me-1"
                            t-on-click="createReceiver"
                            t-if="proxyState === 'sender' and (smpRegistration or this.state.isSettingsView)">
                            Allow reception
                    </button>
                </div>
                <div class="mt-3">
                    <button type="button"
                            class="btn btn-secondary me-1"
                            t-on-click="updateDetails"
                            t-if="['sender', 'smp_registration', 'receiver'].includes(proxyState) and this.state.isSettingsView">
                            Update
                    </button>
                </div>
                <div class="mt-3">
                    <button type="button"
                            class="btn btn-primary me-1"
                            t-on-click="checkCode"
                            t-if="['not_registered', 'in_verification'].includes(proxyState)">
                            <t t-out="createButtonLabel"/>
                    </button>
                </div>
                <div class="mt-3">
                    <!-- TODO remove in master -->
                    <button type="button"
                            class="btn btn-secondary me-1"
                            t-on-click="sendCode"
                            t-att-disabled="this.state.isSmsButtonDisabled"
                            t-if="proxyState === 'in_verification'">
                            Send again
                    </button>
                </div>
                <div class="mt-3">
                    <!--
                    we will show Deregister/Discard in two situations:
                    1. The user is currently in the registration wizard, they are not registered/in verification
                       (e.g. we've sent them the SMS code). They will see it as a "Discard" button
                    2. The user is *not* seeing the wizard, but the regular Settings view. We'll show
                       this as a Deregister button, as they are already registered as either a sender
                       or a receiver.
                    -->
                    <button type="button"
                            class="btn btn-secondary me-1"
                            t-on-click="deregister"
                            t-if="migrationPrepared === false or ediMode == 'demo'">
                            <t t-out="deregisterUserButtonLabel"/>
                    </button>
                </div>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\verification_code_widget\verification_code_widget.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

import { Component, useRef } from "@odoo/owl";

class VerificationCodeWidget extends Component {
    static props = {
        ...standardFieldProps,
    };
    static template = "account_peppol.VerificationCodeWidget";

    setup() {
        super.setup();
        this.inputs = [];
        for (let i = 0; i < 6; i++) {
            this.inputs.push(useRef(`input_${i}`));
        }
    }

    /*
    overrides the default paste behaviour, so that if a value is pasted
    into one of the verification code fields, it's split between the input fields,
    so they can easily copy and paste the code they received via SMS.
    */
    onPaste(ev) {
        if (!ev.clipboardData?.items) {
            return;
        }
        ev.preventDefault();

        let pastedData = ev.clipboardData.getData('text').split('');
        let target = ev.target;
        for (let i = target.id; i < this.inputs.length; i++) {
            this.inputs[i].el.value = pastedData.shift() || null;
        }
    }

    // switch focus to the next input box once they enter
    // one digit and switch focus to the previous input box
    // if they press backspace
    onKeyUp(ev) {
        if (ev.target.value.length === 1 && ev.target.id < 5) {
            ev.target.nextElementSibling.focus();
        } else if (ev.key == 'Backspace' && ev.target.value === "" && ev.target.id > 0) {
            ev.target.previousElementSibling.focus();
        }
    }

    _save() {
        let verificationCode = [...this.inputs.map((i) => i.el.value)].join('');
        if (verificationCode.length === 6) {
            this.props.record.update({ verification_code: verificationCode });
        }
    }
}

registry.category("fields").add("verification_code", {
    component: VerificationCodeWidget,
    supportedTypes: ["char"],
});

```

## File: static\src\components\verification_code_widget\verification_code_widget.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>

    <t t-name="account_peppol.VerificationCodeWidget">
        <div class="row w-50 mt-0 pt-0" t-on-focusout="_save">
            <t t-foreach="[...Array(6).keys()]" t-as="i" t-key="i">
                <input class="verification_code col border border-2 rounded m-1 p-0 text-center"
                       type="text"
                       maxlength="1"
                       t-att-id="i"
                       t-ref="input_{{i}}"
                       t-on-keyup="onKeyUp"
                       t-on-paste="onPaste"/>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\js\portal.js

```javascript
/** @odoo-module **/

import portalDetails from "@portal/js/portal";

portalDetails.include({
    events: Object.assign({}, portalDetails.prototype.events, {
        'change select[name="invoice_sending_method"]': "_onSendingMethodChange",
    }),

    start() {
        this._showPeppolConfig();
        this.orm = this.bindService("orm");
        return this._super(...arguments);
    },

    _showPeppolConfig() {
        const method = document.querySelector("select[name='invoice_sending_method']")?.value;
        const divToToggle = document.querySelectorAll(".portal_peppol_toggle");
        for (const peppolDiv of divToToggle) {
            if (method === "peppol") {
                peppolDiv.classList.remove("d-none");
            } else {
                peppolDiv.classList.add("d-none");
            }
        }
    },

    _onSendingMethodChange() {
        this._showPeppolConfig();
    },
});

```

## File: tools\demo_bill

```text
Z0FBQUFBQmxKVDZxTkRmMFBib0VheVp6aEZfSDVqZFdXQVd2UWVrRkRHQUZ5eXhwVFpscUo4VGtXaGFpeE1PLWZnNFJ3c2FTTExjM2dDQ0xLTzNFb3pYbFFpYnpWSlk3RlpOR0NxeXdjb19tcXRnUGlhcXFxdTl1dG9WZ1F2dUhfOFdjWjVjQmxYZ0FlVFBiVy1VY19TS192U203MU1JSzNsUFZwRUVEZUUyR1pzNVhIMm82Qm9oY2VBQ0lpU0tsRWpTb3E2LU8xa0d6ZWMtVVZzcWFURUdqcG8ycWZvTG11YWtKU3RXcVI5RktSTFlGaGZPTmJvd0xKelVtQmZ1YUdwbEdIRnJsd0l6ei1NMHBNQnRXbzZDVHpBU2xaMk9LYW1IWklnZzk5TmlUdHJKaTV3cGM4aWRvd3hiUGYtU0lOZFZTOFh3NVRkcjBRajlzaGRhZm9SNHpHekU2X2Q4bkJiVTVEdjFkTU9iX3lmbm1tU0dQcXNETW9yZWhOTWpDTXh1OW9Bem5seHR4cW4zdlV3SkQtMjEyb3VZalpuSzc3VUQ2VTVrMjVfNU9oZmZRU2FVWW5HSzdWS3dUbzZlRFVEMWwzT0dGWHA5ZGJ0STk3TmJaUExFdkRGLVdMNmstV05tOUUtNFJ0dDVBbnZ4d2xBMHJEeTZRcHVLeHhfZGJkS2lEUXluZl9MN0NyVGpld0JQMl9JeURCZFRuQnVobVFMSTYtelY0U2RfT240SGg0cnhuQUhCX0xnWGVHVXllM2UxOUY0bW1McWVFXzBhRjFPc21ra2IzV2M3TUdqT3VLZ2Flbm1HbERNWmV0d2R2TlQ4aS1uajhyTzNmVnpCWUVzcm5zUW1hOUZBWnB2enlzSEpYUGJ4dWV3NVBBSnd1Yk5sOTlHOWVXWHpjZ0pUN3lQQnpVblFUZ21KUUdoNU5LLXI5d2N3THM1QnlXWWw0ZmxUNFhRaTVzZGtJUk5kS241U0drYU9pYmFKQjNIVDRRTEFmWl9CdGtGa1ZJYkdtYWF1M1hXM3Iwd1FZRXh1SUlaNF8zTHptZDdfMmQ0M0pDNlg0SkVhaElPSC1Semdod2FfTGhNa2hDTU42ZElDSEpOREJNc0pieW9wNnhDcEVFb21vRHh0el9GNEZkbF9LVklTQlpqbFpsUFRaREtTUVFEeDM0alpJaVdZOUxvT2syV2djUS1MeEJQQTBFdXRNQk5mVjR5QVpqLXdFVWxCWmgwSjBBajNoc2tpYWhWU3RITER1N1czQnI2X2d5WEVLVmFlN3Q1TktrZWxxLTk4Uy1wOEZCbWFTZ1dSLXhsRFVSM0xoWHl0Qk1wNHhHN0xyUzF4dzFiVnNlWlVYRFE1UU90Q1hSWHhtOVNERFVSX1ZoMFVBOExLblVnTThWUVY4UnUtWWdLNDZEbUIxN2lYSEtDQ0dYWmVzNzQ5OGhLS2ZXeEFxSVplcHkzbjUxVXdHT08wc0tVV2hnSUwxeW1VRFJYRW1BRGxRYWxIdXdfdUlqOWF5ZDN5NmtHdlhrWEtjd2hBN2xlUkhSZDFEanl6Yzd2SEk0ZkxhT2tobHJxYXRpRDZSeU1wZ0xkZlBBNy1FVWdHRm40d3B5X25HcXQ4MnpOZDdPamw0dFE4SDFDRTRVMVNqWDg0X1FBTlJsczI1NzdON29iZ09EOF93TzhXb1VTRV9ZSkhLcHFyY0g4dzlyZlQzeHVoSXB6U2FoTXFnLVZmLWpCdWF6VmZxU040OEdqaHlvOW9La3pJZWwyaGtTY2VZcFdTVTlwXzNsdm5oRmpMS3J0VTk2ajRlU2JWdktIcWlKZkVUblZBSmE2WXFHbGFrTE43ZU1KaGFFZE4yRkliTWhiN0lWeEJIa0RmSnhWbVpfVmdzTno0bE9JSG5nU2N1VlI3V05uQzNRYm94UVByR0FtLTZyXzZFNVFhSldhTk9qSDUtNldJTmZCYlV6M2p2S1MzRl83UGZCSktRZEp3UjktTUpXX1pYeTBWSE05bXNmQlZaNC1OcTltb3U2b1J2VlpMenJEM1BlWFluRzBNTmJ5LUgzMFhhY0dfWUkxeHE4SkdwbWZDSlNvbHhYUTByMjlKZlZ0MlZuM1pIb1lKQUlsX0wweng5VTlsWnc4aDJqUkJxZ0MwTk4tVVRsaWFocVBWWmhSRklPN24tZlFCQnZOUVlSWXdyZzFMZG5RRXg1UlBXa0tNMUxTdDJ1THlyc05CZVh1dmNwVUVGQld3NHVQNElfRUVXRmJUN2ppdUhLSUxpaHc5TEZpemUwc3ZhRWVzT3VrNW5zaEw1VkRuT0NpZTNITTNNTmJ1Q2N3bUdoa0pXd1hQRGV6OVZNengwbkNqZC1QcW12NTlvLUFYdTQxZUtQSUJxMzFIWTZienNOZ2ZnUUR4UUNRUGRQdnJDd0N1NUN3a1h0VE5jbzdqcnZCY3psTG9tVWRJQTZKV1FFOEE4NFZJM0VkN2ZoSDE3ZmVTVDhxWHl5bHBQb2QyQ0JsTHNoOWhrTnBoR0NqTnc0Z1dmOTd6N0FsLXphU1FzZXNDalNhY0w0VUtZeGRRRmhndTdoaGo1VEVhVlZ5QmVTaVJSUXd0TlBVTzEtblRmS3BNREV1YVNMMUVKSjFmdW5CQ2dZQXNFZjExV29iUVVSWTBhZFk1N2h3OUpEdXVvZHNqYWpyZFBRc3Z1ckZibHp3ZmczREJDYWR5MExITnhRT1U5cDBaSTVvRWlDaTBVOWRKZHJ5YlpDQjE5cmVDcEM3TDhYUUZ2bVNSN0JRUG9kTzBFNzVod2U0UWI2LTR5QlltUXRsWVF1REh0cXYtTHNBOHNFQXB4OWtaTWRubzZWdFdVVHdWWjAxcDFSeVhkOW9YWnhsWTd5RTlfbDdVRzBxbXl4S3dud0pvU2pXZE5WMU1tWTdHLUE0M19EVVNlM2lvcGk0M3BCdDJuMVg1ZF9sNml2VWRDcFBOVlBSZkJpQnVGU3htYzZ6R1NsTWZ5OU9MVVZfNTFiNUJNZGRsMWU0cGtfUFE0bEF1Y0dQZFFvdk5tN2xNbWc3UmFHdkE2R1lCeXFZMDlxRGh0eExTV3NvSXdTWkw0cUdQSnMtMkJUSGVjNGQ3WlNEb2tYeTVvRHRpNGpDZ1EwQV9lS25HOXZHS1V4LUhDU1FfWjg3UnJwYVJ4UWZPdFNyNzBsWWpCdzFYd2FOUXJTdVNHeE1Ua3htOHVwQUhtcEdjR2NFamxRR3VLWjVqeHBta0R4SGdGSFdPT0dQZUthMDdfMHdKMmhKRTBvRXZodDVBS2dvZmxGNC1UT0hfZVoxWWNGXy01aVhPVzlvMGY0cmJJeERLWUlNNFAxZTlEUkFMRjdNN1hZQzJ2OEdlMV95T1NtWDF0aTc5U2kzVG8yM3YxY0d4WHltdU5aOE5JZUdFaFFqaHdFYWhiWklyOVBNWmx4VlhmNjhMeXQ2ZUFzamo4aTNTU3dJZGs1LXVTV1RTckVkLWNUNFNyMkUwUjNIclZraFp0WG0tdG9kdU5aekQ2TlQ0VUZJUDhjamxvNzZxQzhLXzJzV1J2Q0RKSjJaYzFvaVNDM01paHNVZ3JVdUpxS0xfY2pzVG1zUzMwdmphb2FLcjI4Q0JlUGE3eWd1a0pxa2lZQjFtUFVpUGo2dzhwZWwyVTk5Q3dqb2FvZVhxX2kwaEdxcU1QV3Q0VllNalhmdWVCMEVkN0RjU2ItcTloWnVlc1pON0FVbTdaNm95elJEcms1ZTZoS1MtRDNuMHRMUjk0WHNqTnNHNXhCeG9kbm01cVVnZ19EVjJsVEdNVTlTQ29NTmJ3ZDAzZ3hrN1RaVk5TUGpnUnQzQXhzSUd0R0ktaFRXVENNSkU5OENNTjBYbzA2bzdrZ3VxVHpQWEdFcjlweFhaaG5McGc4VUNzUmFJbVBuWWMwd0lzcEZReHI1a1o2OEVQTzJGMTA1dUVmWVFMZTBHdkRMQzJXWndYb09BSk1HNWszemhEdl9BZXY2YnI0MWtBckloU01HM2h4MkpGSTVGSXJ1cmstWldPXzZtajRTZU9iUnYyTzVEcEFUSUt2b3FXcWZ5RmJXX0JwUWhzS2ZTNW9nTWdIT3R3djN2ekJqYXh4akJ6aUdtNEhmYjlkUGdCOERpVjVZdjhTc3NNM0x4LUdpc2F1YllFUklnSUdjclZ4Y1pzcGp6ZXlhQTVNblg3ekp3THFYVnBmZ3BuMkJMZk9aZWg1T1Z3UGprQ253WjJxbWtkODlQX2t4Vm1TQjExSE1WVWF0OERVdi1nc3QwdXo0Q3hoT0JNMS1wNG4zZVkxVFItVFJZV2J6SDg3eVlMNmt5NTdjLTFVNDRhTkljLXlyWkZPZFRNVTZOd0xfemVnS1FnUUlTVTNMeGtzWjNWeEF2TDJ6bmJhNlFiWmFYUzZHc3E5ZG10ZDF1QjVRamdvLVJhT2JzRFBDRm9xVzFHc3pYSmZiNThQWl92UTV4Z1dDUFA0WkVsNDlkOXN6ckcyNEo3RmFDZ292cHNFOVlnUjNITERRc2FseDRMNjJSUHVkeVJ2c1M3NUw0RHNCVmZVenRQWFZwMEdlQ1ZUZHNKbDJJUmR4T0FHSmVaNWNUVjFuRDdvb2NZTHNTbUxpR0pva0ZtWERDZjZLVXpFRlh5UVRPV2NkQUZNUGE0cW5ZOXhxWXFtbHBYUU1mTVBxUzMyNXFCOW5hOHJVaDc5TEJVdUkxMW5vdGFWdzJYdnN0bVFxTGxoUTVsZ1JEc3ZlQTY0WWNyaTF1SE1TOUtfMi1sVXFCSzBwZHlGZEJ4N2tyMkMzRVRWRmZka3UzaXhqbjFVTUkzNVozdWlOcU5XaWJZN2VmNHRCQnZaU3ZmVER6bm4xOXhmWDBjNjFMS3B5WTZxd0lPdjI2dDZxcFRVVzNPaF9jWEZxZUJDOGxNbHFKOElOTUtSQVMzcG0zQXF1UXFRRmE3VnFuVjlWeGlyZUpKMGZpRE81ZUNYcXlNb082RjltOHdqSDNkYktPOFNua1BCM3YtLThncF9DYmpKS255a1NpTS10aWU4aU8wMTF4ZTQwN0REMVMwSTlLNV9YbGxmSjJBWXpENTJjdURWd3dmOURKUWNyWDRZM3RyYnVZMFpmUHhYanNXb1gwYXVzRUJyU0RjdHVNb1Z1TU0zTkJWTUszZ2JpalpKUzVSUFdSSERTemZWNEFNR2tvZGRYNjc3aU9tdzFhcl9UMTBJT1E1N1UxNFp0RVZhLVBpZVFvWjdmSmFfRl9XdzlEZTNtTDZWckt3djlnWXd1MDRIbmtNWTN5VmZhczhjWjZicWs1VTltX2I4VjI5d3hURkcwYzVNaUF2R0FYNnhMYXZTaGlLS2ZYZGVGQUh5V2VwZjg3SkU5LWJYQmNDMjl6Q1dEOVh6YmdyOXZQVEx5YzNBMlB2cWN5aGlkQ2N3RzllRDJuSS1zcUZodW5URTdpOUVQT214RFVlVjIydThlTEYxdVh3SG0tcFVZTjhndWctN1pHa0Y2NmpTdVZmYTNFVlY3djN5b2hqOEl6WGFtZnhPYjJDdjBjMHR1em50UnZ5WmZYQnNOUHRjakw1ZGNIdmFJY0tUQ1ROak83cHJIYzBQNUJFcG9RZnJtcmNmQjJubUVNQTJMRm1idzhubVJtaHFIeTRGZzR0OXQyRWp3RW5JQVc1MndsVjJfc3RaYVlnLVNQSlRlc21tS1FwcmZwV3B5bmxWc21FVEtRenZ6a1lDRDdPNHlwX0RtTzk3UEM3TGtmLUFjUWJQX2JsYjJwZUpFdlJ5cUsyN1Jfemx1anJSSWRvM2VQT3JYcEhwTHVtcWRFUGRhaW1nWGtseWI4a3R1SWlDRUpHY1VZVXpQbEk0cEF5RUo5S2tTWXE3T2Nydkh1VUN6TWF6cnZXNExfdTBPNVFzU05zNTE5QVFEcHV4eWFyWXNSaFZjSVA0ZzJxc1E5WXlhQXc1TXdua0NaZ3lxb3kwV0hFc29mdGdLcVlFa2ZZeVBuUVZId2txRFdOb1RnRFlncGlGYUNQendOUVlfX3FoNUdmRHJ0ZG04T2J4QXJnaGY1Skg0eHV3TFFYb1pHWHpRR0I3SDJkVmpYclFPdlk0Q2c3ODVLTkw4ZmRKNlRpSlJocEJPUFUtQnlWNFZZNHFPRlZNRUtmWGlTSXJkSEc1QXpkRlAxQUkyNE1XcFFmbThpdzBUUG9UdF9xdjNrWmZxLUJjbUJOdHhqa3lQSXpiNElKZjdFTUxfbURONEIzeGkxUWpvUkZGMHlrMDF2R3ZRUEt6NXNmU3R5akVlR0pSUFFoZTUwaUhSMlRyc0h0YlJVdVRtMzRJMkFVTnlZUHdOSHIxSTJzMV9jMEswRnVUUERRc3puVDh0SDkyVFgwYW80ZjRacjhsRVN6M2MwSXRxV3E5dzB5LU0xOENxelU5V25VeWJ4MGhJX01iX0JZSmQ1UVEyZnpQYVRucjNsUjI4Tk43enNFTGtVc194aUdzZjVGWlNxbzJOZTJVVVRreWdlcUtoQmRMZWluaGxIZks0bTMtZGktNXl6cnVUbVhIVEpBc1JDbWc5X3BOYXlJRWx2bzRRN2hJQ1R5elVDZnc2MWtiWEpEVzlTV21VT0c5MFkxU1JCZjU0VWlKS3ZUYVhnLUwyMVU4OWNZRmFBd3pET3hkTlVzRUJULXBrRW03cE15cFJYRjNTcnNubFBzUy0wcHc4Q3ZfRzgyYUlaYTc0ZVVIQzd4V3hORkIzYTRvZWtTYmV4dzRtLXdta2hGQXkzdVpiOUpmTXB5WlJHSGZtMnZxZF93bHdpZUZKZTh0VFphd2FfaXdXeFdHYW00QnVGSldSTzZ0YzBvMUFIZDJFYlgzdjBzcFpWNkZvM3FNelhTZWhCTHM3bjgxaTczNnhybGU0ejhPc0h4SnU1QWhWMXhhNEprUVB6MGphZkJsT2xhbV8wbS1JdndoNGt0SkdKaGZybnB5SVI5VWpCdnVrVElaRmc3cWZWRG84a3ZrRUlpbmVOU096V1VvZkRZU0lMWWhjaDdRSllFaThhcWtTYXNLNVAwMlNFbUNucm96MVB1eFNVcjM0R0x5NWgwYWFrMTBuanA0cWZqS3pySl9rWEpoajFncTRCcVJvZmptblh6cHlzSjQ4N2tBVXdCTWcyc1BxckY0TVN4WlpndzloRk5ZX1RfSW16SjNta3FMbmN1bkVFdU5KOEVrUm5wTllrZmFJajBqenZrcXo0V0pqVjZMX0RWUG1ILURSb0pYSXh0S2cyejFTbGNYRUhUeHRGMlBLOE1TenJoV1JOd2FzempGQjZXdjlacUs1bnp4amhyVm5EV2hWVUxvRHhESld6RVZQdWttV3BtMS1zRHllVWRqeGV0WnVjcHJPU0lycjZrRGI3MUo2VTBKd0tud3lsNUhvWk5LSFlIZlNCdzlQeUNvOUUwM1NqN2FYRThxdzZ3OFk4WE1DTGluNkhOS09yX1V3S2pTNmRDX21OdFF6N1hFOVZyclVXalZwRmZ1bk5yRmRCVW9VZnZ4SVAxWjRhZEdub2hYRF95c2haSnhqX1NJRFpLYml2c2JSRnA0VUdXdnJ3Ym1FNkRrOTlaS1RJcEhwMWE3TmFUWmpYNnRJcTVVOXcyOFk2b3hpYkM4VnEyYXpKZXVHTzM5d2Fyc19TbUdCVGl1YTFaZ25XZnFhRnlLUW1rUzBaMGplRHhSOS0tdWhoM0hXWDJsUWViUUU5akRkaHFmcF9FdWctbXZPYjE4Q0VUemktM3RXSjNJOHNpWVBRTEhscXhYYkQyd1dsU3pVQjV3N25HZFBac05CeWJuc3hodXFOU3k3YTBSLVJqaHZzbllyLVRKV0l5R200eHFhWTR3LUZoQ0ZmanZUa0ZzV2dJeUpIYVowUGdqOUNUTGs3NjBIdTNET3AzR3FyRVJYcnhwREx5RkxFbmpFUEJVSENLUXZhdk92OGNGVDcxRGxkRGxTMkVTd2l1ZUJIX2l2bE1kN0E3ZFlfMzdrdnBDNjFWTWRKSmpSVjhsb2ZPaWs5aWNFTWJ4OHkydERoSzhSUEpTUlJrQjh3OW1GMjBob1ZvTWZ5aGpEbVZ6SlJka2czTklmajNtTTZncklnN25mME9aWkFicE5GMU14YzROSzZ2dWh1a1BHeHpfZWdfZ0hLaXdxbUxoTUI1bm5kaTFGZUk5NnRYSHkyZm9wSklXVFVFRUFDaXhMbXI5YV80djBoc1htRnZfU1VIQUxOa3ZJdjEzMHI4SmUtbDFtZWk2Z0ZYdTg1T0pXZ2k3bDRvMG0tSVJsZXpVMEhFcXJqTVZUWk14RDR4RFAwNVg0VHhvaWNReHhxZ0xlb01RUnNBS3JNQTlDc04yX25PdWh1dXJnYmw3bF9vdTdrTWJLTVlHNFE5MW13TXAzM3lUVjU2c2VvSVdGMWFhOHV3czIwVko3S3RFUzV0ZDZweE44NVY0Nzl2NE1hOW1lRkJhYVd4ZDdMaUJOWGwxQTJ1SEt0VTdXVlhyOFZ1NWF4WmFHaGFIUlJ0cUZrZlJoeERQeDl5V2NXNEtqMnYtQmpGalA0cGluNmR6S1ZIYWtBRFdaenl3ZDZsYVAyUFd3clZuNXB1dW5Zdy1RRzBiX0xxTC0wWF9FUm1CSjFTaXJqc0ZicUNhOG56TXRRT0Z3c29YOWpOUy1pR1BjcjR4dFZrbUN4SGttWFBmZGR4RldBQk5jcXBLblFXckZ6Tms3U1h5M01lSkpQa3JhaklTS2g3ZVlLYlRXak1mWDU1eGN6NjNINXR1VHkwS01FbWdhV0YzU3lwNDFuQkJtaXhZYW0xcF9RdEtlaElVZm9PcDFabW1qaXRPU2Rfa1hwVWFKWHlDVjlPc0liTi0yWmVjZlFvNlNsNlRUUTN1RUd5MmQzLVhSWExscW5ZMC1wbDZvejluTkFTa2lMdnFpdXRVSlBpMWl2QXk3ZjBFakpQV3hNalIyQVNrZWhpMHJTcnpPbW51M1lVU2dPcmJ1clpURVFwN1RMaE1qR2piTmM5ZzRaS2NmMVNSSzlZSTN3RDc4TGVMb09qVnFXUjRVN2RxM3pYZXRzMElSRWtDWVV5R3V6ZDlkSGVVdUZULXVqWTZIdUEzcnR3dENEclhQbHptWnpwNkpnRTlpcE42WllxSlhTb0tmMXRpeWlTXzFsRGhmdHQwY1FmNWdPa0hOM3RzYi1ncURWSlQwMlROYlhkZGtXZlJmeEZELXFybGV1Q20xLWFNSmhsbkVvZXpQTTZDdE90SzQ4VzVOaS1aTlNLZkllRFV3eDE3ZEd0R3VKck5uZnA4b0hnX2Z2SUVJell4YTRHSDFuV2p4TWxsNEhTS05nX3NTNlZ4RVZrV2xZWENVRXAxdDR5cko0V29IVFFoNFVNTThMdVpFRjZBcmNSNXp6Rk9mb1J0bnNiNTM3SkoyX2VoRi1ydzUtcWVjTzQyZkFaeThzQ3h2N2FlSkhwcUdYMFp4enUwZEQyYWV2QVRqc1RZeEFINUxuUFVEdjBGSndyS0txTkZ1ZnZXQmFsdVl0WkxGanljQzB5VERodUZRZEl1QnIxT3daVGhoMzNJNEdUdFdEZGxUVXctalF4Y3E1bVItREs1MFlSRlNtRFdCenFaVWdZVTJxR3paM2h5TWI0VFRkTDAtOUl0dUZBOURiWWZiMXcyYUh0Qkxid2NEWmMwbjJwMHZRaDAzV2NiSHM3T215ZGhlSTlDa2tZWW1lVERld2pDYXR6aF9lRkw0cDEzTHpYY3JWeTlWVUNKalhIWEtxai16ZnVsQWJyV3FwdEdYOVh4b1JmNDlMVHdrenpaWGhUeDRKNjNrQkNRanZuWnNaSGcxWXpuQlU2SDdJdTJxSml0UjFmODNJZTVRdXU3YUZrTU1OTlNucnNXRW43MkFLVVhuM3lZdWhOb0FBV1ZkSURic29Nb2F1SXVnYXVQVXFUb1k4VlhXR1k5bjkxR08wOFYtdU1Nb3Fza19OQzVHdGlBcUtPcHBfazNZeFVsaHNhTGNRcFUyYm9pV3QzaEpHdUpIeHRMNy1KZTIzQlFZSFNOQnNfM0ZDRGZOZElFVV9rZGVGY0FQby1TeUJSX0VZcTVYS1hsQTZRVUlCSk90V0pBUFNqY1JzNEhBOXRleUk2UzR5b1lYbUFsclUwOGNEenFKWFRWWDRRTGREU3EyVnpmUmlQeVdKaG1keGlQY08tcXdhVzgwd05HMU5JTVZZMEdZVjZzZ0I1cjNhaTNWNFU1Z3FrUEd0X08wb0hEWDd5bXVXSzh3bFpBUDZHLU5sX0k3TV9vV3UtT1JCcTIwZHlZUmFuOWFHeGNkQ2w4dm1xYUoyYmpmQnh0ajdEM1NuMkN6UFJ4djFnazZCYksyeW90ZlN3Z0g4VmtlOG1zcy1mQ1JGSUJNQS1jazhjOHlFSk9LdFp4UnBTb2loa2JKdFdPbEJyX3puVWs4Qk9OUVhHSHdvbnhiY1VlbHQwaWxDR0RzeVJUdHc4bHZ1ZjNVT19GMkV4NFhLSm1fMW93UGJyX20xaWQtMjBGc2JlaHJaYU9pWUNwMFctUGt5Q3lVS3BLODBxeHBrRi1KYndjZ1ZUUzRJX0d5eXhBY2liRGp4MUxEOHZ1bTZjNWFzUFR4a1hyZlNTQTNSdXdSck9SdFJPejlpdm9kUlJhcDRnSGpwMFpyUkt2cEgwSDlOb2lIZG9OSVdlTHpYRzFCc2NiT2ZFZmNNTnhnelR0N3VjME1PV2lWYTFvRzhxdmh3ZlFtYzFOcmFWVy0zV0FoUTdDQnlCeTNzTkI0RlhjdHRHd1JYZzdhaERpS2p1Z2dwXzAzYURWN3FWNTJhZUJNSVg3bVk2WHg2WU5LN0Q4SnY4Yms5bWxLWHJQOG1DelZTajFKT2JlOXNVWFZYeUViZXRiTFBmUTFtNXZUMDR3Ny14RXR4SkZjZHVTWldnQnJKMFJ5cFcxMWFjX0JJQlYwcXU4OUxNbmtrSXpsSlV0WFFBRGczVGRsY0FXQUtoSmJvOGxxM3AzTk5iV2tKOHZON3ZGUV83bldrX1JMSGg2WC1YUmZ2MGllWldkdjlyUzRXUHRKZ3djTV92YzNENFVGVFQxMlpPY3VaajlHd3FuZHdMOXVDWGlvY2ZYUmhJanMyZkVSaGc1Tm1Kc1RweFg5enY3YmFoM3oyRE1yZkFwTTVCUWVRbG8xdVdWaTR5Q0t4S3h5ZzB6VTUzb3hPcFUtM3U5aWZQUzZHUzVHUkRiOVNkNHAxTkIwNnNSN0RzWkhxSnlYTVFkeGVrRmtvWnU2dmhMZHBhZGY4cUxFdXVJVGpBeXZYc2owWjVnd1ZrdXhVV2N1SXJxYl9XWUVZSUNiYWhUOXBzVUNXVkh1VEY4RUI2dlZ4b1QtUWZGR3R4STl2SC1aTDVTR0pFSjdKdS1uMWdfYlNWNTg0dG01eU02VHBXS0plVlB2NFpqR2thWlM5RDYyZ2h4Mm9fczlVTmRpN3ZJbHJpWXZJMGJCTkl1akw5aVo4eWlGZ0UzRmpUNU1WSkFqQ1lrUVBjY1o4anN2bS1LT0ZXa3p0a0JhdWR3N3BDajFsSVJkZ3RNMEtPX05SbWVqak14RjVhZ0pDdzRnb3lFX1JXbHg0LXdvNDBjMUN6c092SnQ2UEd6YWRqOTNrQlJDV2xxRFVXWm1QUkdKWUpZYWNZN1l2cUZWSGtvRlFNUUg3UnU4b3d4THB0VzJKN2NBUThSaWM0RVFEM0ZkVjhYQ3Q0SXVKZlFQQmtnUkZmNjdzcXA1aG5IOVZvZXk1bVBJZE16OEFHMmhmY2dXU0RKNmxicUxMU1Z2QVo3Tk5HamRQVUExNlpCbWlBLWY1cFVaeHlwRXA2b1M2dkpXblVyS3ktYzFnTDFEOEg4dDAxRFNUQzhfU3VDdzc4V3VUek5kNHRrQVduSFNGalNDRW5BQ1ZuRTVteXUtMHZ2T3VOQ0ppLXVQbmQtSXZzb0xDQ0ttUkdyY2NjdDNiVkdKS0YxQjBVQW9ZQlktVmtLTFZjQ0RKMGFsaC0xMlcybEdyVWQ2TmRHVW1fS09qMEZ6YmN0aGFSaTh3UWxiNmNpSUI5OFJNRnUyYWkzSE1jYm1jSHRJZUJYZnRNV2UtbHRiX21Bc1lVekNtRER5Vm1PMGFRYUt4TVZQY2tyRWhmVVBvNUJyaGdUWWZISkVNUGxWV2l5U2RaMVVKRzBzWXV6V3dCNEM0WFMwWXNRaHF6QUtVMVpCNHNNUEZmRi1XU0MwcTBjeFB3Tm9MSnpKX2d6LXh1UFNkT1NRd2RrbC1LQmxCUTk4Q19lcEI3UTFzaVRLbHBiT1lZTzJyR2dNSWNlWTROUkhKd3Naakg5Ym1CcGk4QUtaX3ZRQTUxN3dzOVMySzllNGQ3RUZaUFFhSTBQVGV2TWV4LUI0QzZfZ09CcExGNTlFX0c2SnBjZGt4SUZqeUw3U285S3F5TmlHQ0FJMEZuWEpJdVAwRUdsRl9KX0RlVGt5M0RKY0lCbHZyQmJ0Q3l1MHRacmstYkRiaTZXRWVRTVJUenB3V3NJNmpTdl9qaWl2ZGVpSWZCMXJ5SGZ5UUZ3ZXhKMUN4ZExtTnNpQzJ3Sk9YVGVLRVE3VE1OSzA3Vl9qaVo2b3ljWHBoQTBPRnY4RzNHQ2VHTjNhLUYzaloxMnhmb0c3NTNvUXR1MEhDSmVkR0NuMWZxZHlmaDVBU0tKTnZ0T21sb1RLV1NzMFIyVVZTd3l1OUpMZG80Y1plV3BZSk1pXzZ6V3lveXhFM0gxOUpMb0NORWR4UzJseTZfVHliTDVxS3RKcjJ0WUEyN21GbFNRX196M2VZX0Z2eTRvNmRmWmtBLTFSSHo3ZFBBUzJOUXF0NVJRcEw3b2lIakdVbW5jRlVXSDRJSExOdUc0ekRuWHBtQi1aTXhfWG5VOXpNUEJNRjlkamxxcWhSeE40QnIxTVhxTXpqQ2RvSjE0OXBvWGFkX0ZtR2xVWDY2TU13VzhjcDVJSERBYmx4azYzbHNMSHVEdG1lVzMxdk5lWWM4dWZ1eDEwUFRMbDJudFBmTjFCcmV3UHRDY1duTE9GQm1Lck5IVFpHdUhGUENScW9reXFuTzZJSkNDMEQ0dnctOHo1d1c4MS1vN0VBdkZKaGdxME55YXVwSllhMk1RSjJVU1hjQUlwejBTTGQtQW53dXpZVmJZNEZUOU1idnVWckpPeXdKS29vdlk5bUlXMGlCa281c3RneURQemNIUURkMDU4MUM5c1UzU3k1UUY5RVVnOWhQRm5HVktVRXdlekd2dlVLdlFQX0hmVjk5dzduRGF0dGVaejZnV0tsVjZWc1RKVXB1SnhEN0JyMmlUMlZBMFBFZUFkc3pVSDdYX3VTdDdkV2lmQXRYTG9QcFQ4cGJFR2ZUb1BqUS05RndORUhJSnFRamZyMFExNVlnU21vaHZyamNtb2MwZ1ZpMkN3bzM0TTlVQzRpTWN5bFBDVlQ1aWJJNnY2Q283UW5yNVdXMXVpNXN1cXJOZnBUby14dGx5R0ZQcmVxTG9XX0JoaG9TT3RPLUpYbWdQMVVVUFQ4QUFWNmFWMFJtd2VwaDlJYmdhQ29uQm9KUGdGZDFMWDZZamVLcE1JZjQyRVo1MzN0dGxCejBDR1UzRzhUWUh6NVZ0VDRvVFdtOHB6WlNGQ2p2SHoyVVEtWFdUQVN1b01JNGdpYjdYbUV2X3g5TURxUl83TnV2NV9UMFR1U05KWmZQSDcyQm5JMkFXQ19MZzNieEY3eHRCcEN5aFZNSklrVmhLZEZMdGt2SnBSTllXQ2lDNmF2TE54VUE1NUYzeVF6d1F3RzdVaV9OTG02endaTEFRWHpxb19GbnNaSEZ1VlRWb0xqcnYxZEFRWHRNR1ljdDZEbk9YNjBaSHVNVmpaZTFydEtZdEtaNzM4WGUySms0OC1iY0JLa2JEajVhc09lWTNDLXhaVXRLZ3hZSVRHQnh4VEFsX3J1b1FzMWVQZTJGbVBVVXV5SkRKRFl6YVRMZHhmbWI4RXlpMXNLMDJELTBNQzNkM29kdl8xNTJOaXh3TnNlb2VZcEVSQ3o5THNUU0NiMkR5clZBWGVrak1HTlY0dzY5NWozNmV0V1Nkc19sWC1hT0QzN0U5V2t2TlNBVjN3eElwYXFGNGRIcnhGc1YtRC1jYUJMYlZsZll2LWFQTTNybUhJblVRSENiNWFHS1AxcHM5VUdhdmNQdVJFclhzQmt0MHpfUDg3cmZPakVVTU5jZXh4VDRpZXJCSWt2RGxXSDFybGNNcEpFVmh4VjZVZkdEcDVDaEhmd2RSaG9nQjFiQ0JKeHZyYkltYVNJUVZrYVNuZzdleVNZMUV4V1VPc01SdFBFUnNSU3dOcHNnNE4wNldWZm5sc0JQbGtGeWNSSEtJeFVhVzd2MnNOeDFNU1QwanJlZkNwenNiWUVSb2NIZXpuN1hZcmVMcVlETDljeldpUExjeEpzYVAwbWNuMXJxZzdkNDRMd0p4S1UyQTVGa0tFU1BUQmNLeFpJMTlUbVZNUHVENDFyNlJrbldyUi1zNGVnU2lGZ1VQVnFCaHVxWWNta0kyRFRQNFlUMXd5ZnpIQ3NiV1pQYTFEZXptcHJETk45cVJvcTNMa18zV2lsWWV6UDVKNElFaEp0dlRSVV9LYTM0Si04ZU14eDBiN2Y0a2E3QUc5amkwNU04ZVV6bU5VbXZETVdMOHh6OGtOUEdSMDZwZHY3QUk0Rk9KMVdXTzFGQ3JIbDVhaWlOVWtQWFM5MzJNWjB2T0xkNFRVYzhFUTVkUnRwcG1udnpYeWxXekdxQ2FEeXJwNHR6ak9XYm9leG1xWVh6M3hLbE8tampiS3ExMmVHc3c0NVNVZk13dXBDSWl2TC1UU01IcW5nWG1JX1FHeU91S1d5YVZSQmRvTDRKTEtMUFdYQWlydU1GYXJaTEZ5YV9KS3NFNlVnelJWaEtDSXVscUF2TVJ0bUFkdjlheFpkSG5IYjhVS0c0ZGs5SWdhNG1nM2Etd25BZEU2NDVrYzFsa21GbHBpdlNtMGdMRjM4eC1faUtLUDRSRGU0ejd4VC05b29OcVlXU1FrUUtJWmhLbVJNNFBtbHYtX09LWmxMeThsWVZZSHpPOVhnZ0Z1Ymk0UkJSaFVXYW9DQmNXN2NCbWI4cEdPY0xLRWViSFVza2Z2a0M4dWs4dE11NnpleTc0SEpEcHJLQVJpRkY3emlyaWpiQnBvakFPd1pSblo1dTNLNHRoNWRJWlZUT2ZtRkp0U0E4Sl96ekNIOHBEQTk2U0UzWjlfZTNCNDJDbTNBa2FIY09Db0RTblRrYWZyQ0RkS3FWVnFwMDYyTDB4RGNuLXR5RzlhNG1rUVpEbGF3VHJMTzhiRzB1R09icXVNa2x1Nmh1RWJQT0F5ckVFel9nRENONms0WkJSN1VUeUVQVm52ZkJhaGN0MzQ3X201S1lmMGd3T1A5MzVBRVcxMkJUUktfN19nRHJDLUE4dGlmbEtRRU5RUUhIbXJWQm4wWi1VRkFmUXRMYUwwb0lXM21EVE44UEFWeFNKY0RyekFoNG5pM3JKT0RKSF9uMHR0djVqWDZrMmtfUmlIVGNJNVFXT0RvUkVrMkQ2RjJiaTk2OUFsb1EwU2JJSUJzZ3Nmb3NTYXJUdnpfbDFSMXdVcWstTl9ybXp2Zlk2WmgzazlYZzQxMGFPMnp2dVdneVVja25GUFRRdWdMLTV3U1hfWDA5RnBSU0tLeXFZVmZ2cjdKaF9kZjlxeVVjNnZsRENhempMTnNKQVgzQWZwMUlidndTOHNoQThnZDVfMmpKMmhDSm9HVWFKRGRtSWVyQ2JVQ2ZzVkEzUk1DQ1RsRHZzbWRUS2RNalpXZS1OeFp4SjZuc25ISFFrOTdYY0V4b1VvV2VFMXdoVnRaWWJaeGVNRmtjSTh0ZFMxaENSWFg4Y2dMLTZLRTJhbS1wNHJCWDFSYjBzR0QxdnRxOHgxVUxDNjczYmMzbVhvOVdENUJCVUc5OEZsNlpjenlFM01rSE4zdmF4SlRNS0xaWWdlUUJnZVo5MGRMa2U5NTNPZTBkQ0RwOHRLel93VXZXQnZzWmlRZ3hpZTVMR3JmMEhzc2E3NzlDNW8xYjVwcHFWbnpKd2ZxUG41V2dvbHBvck1VbVc2ZGUwWHZrdGtVcHgzRkY5Zm9aQ3BsMm5qOGRsemlrd2lDMXMweDhYTmpyN1Y5akJFclJYUE1MSVpGSkE4Z19wVXlMWkdwWUgwdnhXMi16UEwxcU5EekpLeUhxZ1FkTnNsQ0pkMklxOGJhRk43VFBaSFE0VG9xRDV1V2RyaVBUaWZMbHItSnpaVkxJcVhVX1AzbUdNd05DcWRtUExoR295b0xpR1lvRnNXMHVFT2tKMmN2S2dySDc2akVzRnRHeTdPWWZyMUVSbHdKMm85bGhaTkdYQUxrTDdPM2VZb01iNlJ0Q1FDUnVadF9TN0hLM1R0QUVpbEdVeG04SjhDX2FTMnVwZjdYTkhRNlR5c0lIa2FjU0RuU1ZsWG9JMWUwMkk4SUM3SDByU3M5SHFzdGtrQTNKbDRIRE9VVHNPa0xNTWV1WjQtWlpKSkZYd0R3V19nQ21xVTFYVGE0TExjRVh2Tkh2bVp4U2Q0SF9RNXlMNDR0WjNMRlc3SEpwQXd5dDM5LXRlSG5ZTWFyMmpiVjExNkRMMjlsSHkwQVA0SThabWhVNG11Ui1YNXhZUjdpVUlfeS10blFDbXZSYmhZTEdMR2oxYm9YQVJfN0dyYUZxd0hBYm5iX1NuWmRiTnFIV2xLUjNlSmtKWHNiNmI5ejNVRHVkZXRMN1dnY3owYkFDSm5GZldOQUphZjJfeS0zeEFBU0VxeHFIX2dKWm5OanlFdEdqZmg0V1ZpTlVZeVNoT2tzZDdJeks3MzFLMWM4RmdOZExNNkQ2Y3ZrQ3Z0c2JwOTlDUVk1SG84eFowdlV2R1lVWDVNN2NUYjJpV3NZRTZHamI4X3I5Q0oyZnI3S2RhajBZTUNxb20yekZVYnpuazNDUC0zU3YyRmlxaEFXYlc0U1BIMTJpTnNPNmN6VzlVTDhBNmlSSUZ3d2NGT2drenM3UnhlUEs5cm5rVXBJZ1RCUExvUTlrTnlWWGNudUZ2NV8xWjNzd1RHZERwMXdLQUR1SkdfdThIWHdtLWY3TEFVM2xpTERRMVJScGtQSjJ1SGpOTlN5VjVKRDVGV0Zlck56OHFCSGFlbHpKRDZ2cFBRRHAtOU1zSzQzdS05dkRXTVB2ZVdzWnNqc3BsZ29IVk0zamNmRTluX3A3d1hha1l0YUFyUkVpamdzWVM3NmNqUDMycVBVbm16b0NacGdjbko4QUcycGY3UEpLOE5YVGRTNWlkTUxCVnVtajBwcG5NbmRXQ2o0dWhTcERRREVCN0Q5Y2hlQ0J6Mnpha2QySWE0blFWSWtiUUlnMkVjTFZDdC04SlRWbmdPSUpsQkdqbUpxOV8yUUl4Q2ZndF9vR3Z5OHZGSmo4RjBicVdLbjg1UVhyT29QRUtURFpIU1ktZTJnLVdDeG1qR1V6RVpva25ieUpLRDVINnNpSm1sNHd3MHBZc1U5dkFvemhTVS1wcEpfdkQwaUV3X3M3U2dVMlVXY05ZV2dUb0V5OC1hNkc0dzRsRDZST1Y5aHZLcnRyRFJqcXNsanBoOUhPWll0cWZaT1hNVTNELXFfQ0lxcFRXd0ZzVDNsVVlGRFFoZzVWRGZBNXNNS29QU0hwbWJpYmgxVzlXMGtKM2RKN2VJd2NyVkt4QWN2RjZXOEkwc1dRM09wY3ZZRlcySlYtbFJIZ3lyaGxJWkZBS2lCS1BpMHJma3FJdEJ4c1Jtb0xvUU51MUN5ZDZfTzQyYTRJbXdtUjhzS2FIQ29DMmNINHdtS0JHQ1hQaEtLZGpCRllWSmNmX09HOHktZVE3RmJxTk4tVS1wbWtXN2tDUVRGN3hxSVdOWEhqVEt5RFc1VEU1UWZRdENGLTZfa1d5RVUybmJPdXZmYWoxUjRPdlZhcVQxRVByX3lNWk5uN2p1OTl4aEE2OHkxYUtiY2NQTlpKN29RR2NONjhpNGg4ekQyUXFGdkIxM1pRRWJvZEd0dmRvRzF2MGRrTUpmaXBxMW5MRmEtanVMeDQydWVZUk1ZdmEyWU9tNm9JbWVqV0dGdmhEbnViVjMzdWVvOTAtc1JtUlFLaVMtOWR4dERUazVxaC1XN1E3aVVYMlpYX0hLSmFxQ3oweGthTU4tdUYwVkpfYS16X0lZS1E3QWs4Nkw3Y0MyTDhYeE1oN0hQbmlvTl84YWFwNXZmRUU3eUJqQWVSSW5DNktuUWVnbUg2dVlVVFlvZDl1cVpBNGRyRTlLcTVyVFl3MFVXNy1CUlFoa0U2Q19fLTNQaENyNWV6cWhEcWFwSkoxWnBYcEhqd1Y4M3hGdDc2dWF1THlwQTFVeTVOUGg4UTAzdk9wd1VOZlN3aFpNYktFWEFaWlQ4QjZpUHloT0E5SkVWVmhxQUJCdmJPWm5PQlNJMkdwekJ0UG9qV0VaTW9DcjMza2poUGY0YjNRMm55a1lhbEwxbi0wal85TjFtY1FUYkVjLU85Uy1mX0t6dHdpbFo2d0twRUNUdGI1a0VGa3JyLUs3RXdHd0RUTkpLX05QV1RpN3ZiV3FrUUxrdV9mWmxMcUtjU0ZNVG90RThCa1Z6QWlLX21VcG5LNFZiZWprMkY2cUlZdDZmQXFzcU1ZeE1UdkpRTHhyVE00dW84dzU0NXh2ZzVIcmRUTHFCTE5xZGludXMycXVjLTFWRHBuN1RxZ1lQX0QxdmZGSVV6cEhjVjFjZnJBWUhDdEZBVGhTam9ZUGxTUnJpdVFLYnpuQVFTeGgtd3hSZ0Vnd2xaa0wtaVN4WTJKZ1BiSHlwTE14YVBwbVpob00zQUVQbGhaU0dXTFhyU0pVLUFMYmk3WUFGX1NSWGJQTnNrZXQ4amdoRzFTQ0hzNXRzcXJ6WDFHeG5wZWhPdWdnYnNHN3Z5UHdNYm5EMzFBMU1DWUVNWW9VSnBMVDYzOFFsS2pqUDQ4VmJhUlNIM1VRTHVxUGYxWW1UdWJJemtRTU9YejVIem9KbG5TcmZCOTc5Xzh2Ml80dUVBa2kyYXZTcVRNTkNuQnptajhwX1dXUnBYNUEtamJNMHVHZTRtOUItSXFxcFJKRE1LV0dnWHRvZm1zdERkRFRBN1NVcm5FNEtKS1hBS2g3UWJIZl9jSko2WVhWRnE1ZTZTa3F4RlZHam1kb3ZXUHdzOGQwLVJvRWxsLVNRWkUzRWdSTnFYLU5HcFFRbzFFSW1DUXJmU2VhanRWWXBaR2RkU1F5TGN3NEoyQ1lsZ3JNYy1hODNfRHY1S2NzOXJBZENjdERGUVRvYmlienFsbDJ0cTBRTjhsQ0dxcF9oNDJrdUwxNk9DYmFWemVwSjNCSHp4M3JmMFBJZmRJZXJjb1hRUUVsNkFUaHFWdkZxTkJ2cE9wUkNHblVKdHRMNjROc0ExTkd3NWU2Y2RScVNjcHJoMzlJOV9fWjhCdHhLdVdmR3p5UzdxZlJldm1pY1dlei02UjV0dTdFN0FXSzJYZWRtaWM3UWZzaUxaWVphdVRueXFiYnYyYTgtMUV5R0lQZUMtYi1XcTZiLWhfbDBNVXVsTlBWTUNVaXdCSVRIZlBsbHBOTzhjZTdJekpfTXphQmVVeXNKRGRPZTJGNVJpQWRSNEw4WGRIQjM0TFdLQjZpQmlwNHZEMDAtT1VjNmh1WGN2X01Yc0xWM254VEJXdzFDSXpBZWlkcGZBdVlwSkVQMGpLRzc2VDMwM3ZIREtNeG5lbjhjOXZlNlNweENobi1Jdkl3Zm9zNUprVFRGal9Nb1p0TF9nemFUeU9xNkF6QmtkamVqUWtuamsxU0RXaXQ4QTVGeGJZVXVsazVuazBYbGlNR2hCT0hSVUZ1N012VU1Qdjk1WnBuYWxCdllJNDdzX2owQTlULUVRV1puU3cyaGpuYm9MV3czZlFuTWhIUFJOcWd3OWNXUjBfaDNWaElldXFMeEtkTHBqSU1zUmo4S3ZDUkNkZjFlUkVsZmtQUVM1dTEzUG54YTA1aFZYcncza25BdkZsUEItMzc3SU1Zd0gzbWNRa3YwS2lKTkR3WHU4TVppSEpRYnV3ZVVPYTBILVZVSVVLbWpGZTVhZVI5RGVKTWs1c0E4cmdvWTJVU2dlZW1WY1Fvak95djg5dWxmSHB5LTJxZU1ZVFBma2RCNlBWdXU3NWx3TksxVXpvYkw0eFpLN2pUMjdaTlhGOWJjVGloLUZWM09oS1labDRHejFkZFBnNGNpbm94cE0zVlhCOGoxSzF1RV96bW1oUWFQUVlyRXpUNXdRSFdvd29JOVVReVF6NWZSR2w0TGo5RFhYZHZBYllNNHlYNkpOM21meGIteWJ5dHpQdzNuNzhOY1RITlZBOWEyTUJzc2JuUTUxMlM2bWpOUXFYUWg3N0dtRzFBQ0ZRMjhqVHM3cUFsbm9rMEN6R00zLVZwTVk5WTYxOERQZjB0YVBtck1kcmg2Y01uNjJNalIxa3lORlFzaURrbXBMY1NiY0t4Y2k5aXFOREZHdjZUczRfX0E2emc2c2dNVHNZNFZRQ01yNl81QkRMZVJ2STdxN1pQYzk2WDdkTnVMTzZzeGpwQjZpcGJVMVhBTTktdDlUOUtSc2V3QXl5V3o5MmFkR01zQXh2UXh1Tm1IV043Y0tMZGk1WGMtUkFKc0FBS3JsaDdONEhTRjc3SDhSWlhacFRoeUJYbWhhMXctbkxyVmFyUEtaTmhNUnFYbkNtRHllZnRKQlFOaENveXdRUDF4VUU2WjZYYmVmempYMXNMVWF0MHBOZGZJRzhsaFMtMVA3WXFrWkJsekNQTzcxamZCaWNkbC1PZ0hUdzN6TUIwMEZxR2NVVjFKZldXVk9NMU9LWWFSUlFaLU0zaEV6eEM1Qlg2VkEwYnFNamdxZDF0Y3ZRUjJxTGVPcU90OG1XNUY2OFRjODhTMXBEbDF3S1gydW1zXzdDS3I1MTJraW5fWDVic1g5Y1VjdENnR25VdWQ1Rkw3amQ0MHhuSmFwUkRKSnNqMVU3aUdraVNGb25oZUNRZzNrR1QzU0txN3l3QzdYOGlPUmhnOGh0WWRXZGNjX1duN1pNckM1Y3ZvUVFlOWszUWdfSU5aNlBvQzQ1aHRpUWU4QW8xY0RqbGNjVzk1YWFuUWE4OVpkdzZlWmVrZjFGUURJVE1qLWVBT3U4RWRfQTJpaFFFa1JZMngzTWdlZm5vWnRUVXdsbUVWTUhOQ3BBUFI1ekFPV0hEYVlLSURqT3RQYklSZjNwTzljUTNPZDk2eUROX0tURDU1MDhJOVhuc01fVGo1NzhLdi1Kd3VwYV9lc3NzNFNvVzdBcTBUUVprQU15OFhIaHJzRHdoRjZNWmxpaVNsODY4dkx5N3gwV1hzNzJ6TDFQbG9ieUFfaTM5Mk9CTEZoazlTQWpRRWtiSExLVURhaHU3SVlkX2k4WVNzUmFXekswUGZVQ1c4NFJSTmhweXFHclZBajZSN3hDdU16SlpBNGxCa3ZoSUFId0ZqMzI5T3RwclJiQy1VdUpITVFaSXNBZWxiTmZpaExCOTJ0elBuMHYwZDFiYmxXN3laU0JRV19GQXpsSDFrY0FvenJKcEZsVFRMVkJWcFgxaF9ELTFnWm1sVVgxbE9KUmJzcHJlcDd6WkU0YVM2eHB0UHFmLXVNVEhYcDV5NzNoaG04dFktdEQtbVJyQ0VncktWX18wTFNrNjhkQWNpYXFoRmtKTkJFWlJXTXkzVHlWMDd3Sy1RTnJrY0NBWmRrT1I1TklFU3prRm5aX1VJM0hDZTBETlJPTlIwbVRoT2hSNldtV21YckxKWWQzQ0lPNGVuMG5VcXhvSGgxaDJIQjFmQ0h5Mmt0bmlqazlpZ05wa1UxVGxYcnNBVlNUdmRlR0JMdk56a3h5d1BILUI5X2I4bEtsUDVTOHN2bjN6SEtVZ2FhUGlMWGdjMmY2ZExUaTJJNHViUnNRLTd2UFlJWTBKQk5nR1dxTUNWMkZWaUNLUDdQRVprcmtnSU1ib3hSOXVLWFVCR2g4QktUZVlLRTlzdDRrZXd6RFhyNVlkTzFpLTFIZDBpb1prOHlWT3VYRmVXckdPSDBiTFhlRldtbGJKOUczazNrSW1rdXpaYjZyaXVLV3Q1TW9LeUV5dnFwZTR6SmpSUnVILVdSaXR2Z2VkSG91bFE1NDBvSVktaHRSRS1iTThhMlBjeU4xYUViZGxRRkl1cEtLYXp2UHN0OHdaUDQxdVlBQjY0U2xyMjZoTnJiUG5idklDaExXVVZOYzctSEhJRThNQmxRdS1wYjhVRzZJdnZpRmZDbzdwelB0WThmdnFzejJvd29GSEJkYWZtREJqSzVIdndHVDRjYjVUbWdkdFN1VjRmZExaRUNDaDlxLTNmUzBlQXZSRElGdzc5RGNoMDdBck03czd6TDcxTUlTS3lxMFRWeE9nbTQzUXVZYWNWUjN0STNHSC1fbk5DNVVkVEVlRjNPTW44dHUwNUVxcVZwcDlvR1Ffdktxd1lGbGl6WUx3ZExHb1Z2Wk9LRlB2bkVETHozT1ptTmNpbXN1QmNrOV90RXI5YWRVZFlMRkFLbHZoVHNReHhiRkR1emhTVTJTZjJUNFR5VWQyekFTQnEwcWx6M0t6bWpBdzN5dXdTMmwxOTFLRnVXYVcxWEM4ckR1M0FpU25HZWd6ZDA2Y2ZFMW55d2NwdXNoRkpKZmkxUW43cVU3Ni15UkI2QkNXYXo0M1FwTGdTdENQZHluMlpsZkJQUE5uTEI1eEhRV0Q4WGNqT290S3B0SWV0SEE3TXo4WDIxV1RSckxOUFREMWRNa09VY0VqNENWeTJvb0dSVVpUYkx6OTJwMENwNEwyQ0pvWE40azhCallHRW5NcnNQSjBweTZFV0x1cWdKVERuOGJVa0dHRmFWR3lRMFBUUUlIVTl1VnE4ZmpJZXpjVm82bmNlNHVaYVBHWjZfQkVjVnpNUlNoUFRBOGdBTV9nb2hXMk1BRElvSjdFa1c5Z2NkY0xxNzdCbWZFVzdFNFg3cjQyUk1RX3Y2X0VBVTlrREtkcl9ZcWFCMzg4aDA4clhyenYtTXViTmtlSjZsZUdaTGpvdHVDR1FyaXU0TEI0SkZITklOTVUwZFE5SVg1Y1NBWGhuYllmUzE3eFc0cE9yWjRoNjd2V3lkbHZ1cjlQQkhFZEhlN3JCaHpwcEs0R0cwZmZrbEhJTldTeDVnVWYxaDJ2aGtKUndMODFaVEE5bVpSWkpQeU1oMXMtcFcxaVJENG5iMnlEaXAzdjVsTUZRcUtMUVpEMnJ0MVg2OVRETXZKNlNUdV93YkJlSFA0YlZpbFNock1WLWNYYmg4ZGtjaHZOMzNYbzZ6aUdiYW1tbm85UGNzUGVKTkZjZ0FiX1JMbUVlbFI5R2hONmRaZ1c0bE42bFBMSS03YWJoOE91VkJVOTRDTzljRkRaUEg2STdaalN3Sl81WUp6RHBKZVJEUGt6Sk9ibHRWbFNnTlBxQVBCbldxVWxZZV9uTDg2SkhFT1NnclIyektlem5udGNaaTRqdTI2REZ3M1JoRG9xcklJWnh6ZE41SXQ3c1liUGhPbmFOdWw4d3N6NFlVXzlFbG4wb2dnWjA3dTRhS2pFVG44YlhuUTVsWmNZWEpweVNXLWlsd0tINHZZRjVja2tCQ3dfdG8yamxvRVVrSkpJTVlTcHFGdG5xRG4yNFBkdUZHeTVzak9lZ0JlSjlELVNmWFQzSU00VnlTVGx3U3Y5MFAwazlTVm5TY2VtTEtOYmZUdXhjSXBtSFhjaVBEaTlyMHRSOXdOeXIyczhKRXBDeFYxTWtKWjFRVEF4SHV5MkhsZU00M01HLUIwVjhHOW1HdndhUzl6WHRXdFJBUWlRZzhnRGlGQ3IyY0xRZU5FVGprM1NpQjgxSXBDSW1ESXN1OVRpLXZnTHpmYVA2SGVzMTJuN0daSjRETzMwODdzTDVfQ3NXb2FrUmU3U042M1hTbDltU1BVaEZDeHNTRjZuUVpaYU1uSVlmZk1IcnlsM19JTktNWHlmbDB1X1BCZm9BVjdkVExFUEhLa21Gd0wzUHNSQ2JDRHRqbW1OSmRLdXRLZnJLY3I0SVNBRHZlRDU0SG1sOFBuWVhyN1pBRlVfa1Q0Vm40UzI2emVlb1FLam9Ld0hua1BuTmxQMm1Ha1BwWjRzcTVZcTg5YnYzeW43NUF2alk4LTBIN1p4ZnBQUFVOX2hlRnF2QkY3SEY3SkIwdE1Yd3R2MTNUSzlFMzBrV0E2UEttRW12LVpPemFhSTM1SGh1eVNJODliZUZJQTJYdnZhdndTWmJxd3llLWVyUlZPZHB3cHdrUDd5OVlpanR4Z3lManFFMmpEc1BOTVhFUURIYmZlZWp6cU5vbDlETDhudFEyejZ6eFdDUVlPLTItWFdGbEF2Y0RHeXZVWk52YUl5MUszTk9kbUZ1OC01VGJFRFBZWVl2bkVRWGhNX2EtN0QybTE1cDZQVkxtT3RLWVp3V3NWTEd4eUppS0lkQ3FEdUVpV2Vmd1puc1ZQMnR3a1dFZmlaekI1aFNQcHJrd25tb1NZNEdBYnBhM0tycGwyMXNxUFIxaXRtb3Z5VFB2VmtreEpDcnBram0ydzRQamdyVVRSejRHS1NsVUYyYnRXU2ZrR0gxTmExbXNlNlhiVTdrb1k2MGNaNFlCblg1aHJIUjk3Qk9xS25lNWd3aC10VEpZM2NmZEpzc2dac1RLUlBqS1Jqa0pBTzJELXQwSUNWbU9RX0FfT195Z2U2Tm5YYnBxTkh0aXA3Nzl3WEtZZFVzTEVOM0I1anlLczhhcnI3YXNKNEo4eUFfVVhjdnNUT3dpRVF1YXNweXIwbHlOU0gzZDVmcGJjUVh3RGFnWVBoa0VkUG1WQzZxdVB6WDZxa3plZWJHU2JWZ2VyZFN4X1g3cXRnbGhBVEhPWWUzcEpMSGxXSTV0bWkyTl80TjdOSGpod0RYaWNHWGVFbE1LT05oV285QkRpTmxSamxxUF9BRnlKWkxuVW1vRmpheDRCclhmcnkxN2lYYjNmTFg5ajFRWlZIREdmZkRxLV9aVmJEMjFuaEs2Y1lxel9NUDBsYklxN1YwdDlUYzNhRldGMWgyNThpZlFLcHZXT180QlF4M2pXMUNsY21XempGRlQzRTExX0JiUmgzdWIxWFRhaGxqYUROZ2dSTUxuWjBhYzc2Yzc3SWRZd3d3SkFOTU9aX3I2aU5hWEpuTGx3a2VldTJMV0pibWxEYzIxRHZ6NV9Lbld2WmtCdmxFODBWOXhHVTVjcU1BRjM3cDFCWHAwN2EwV1d5S1lZT0l2S3o1ZTVxWUQzdmUtUTY4SWtHVFliLUR5ejV2dFY3ckNMb3ZSMzJLVnZWa0VmMUtpWFJ1ZG5TSFFnSVR6Zkc1X3JzS3dFYkt2Y0RpR2JZMC1tYnVQS0FkOFl2YzhZdFRNZHJpY1ZDRUVOdjFZcnYxMDdrRHFvZnF6emsxU1JocUZqQUtYMFZIbXF5ZHFEOU1tUDJuZVlXYXp4M09idkFNU3RFczlqU09ld3VXSkw0aDk2THZHWlZweWx0RXlXcWRaOEhhUzl1WV9JTzVWcGZyb05KT3JTR1NxNFM5NUFSSTljM0U3ZzFidEdRRElHbFBVSk9HbGdiMzZ3SWo1QWN1R1lLQWphQlBSTkRMbEhRN09TX2d5ZlRGb0E4WlVfTngzeUVSZG5xMGVVNEV0bkNZR2FFRFpqR0xUb0lQNm5xMUoxVUpQYk5YUll3SjdJWF9GN1BNVHFRZHd1SkxQNUNuUmtkLWpYd3JzNS1EYkw5UE1MXzFWX2F1U1pvWGdWR29RR3B0TlZ4VURtS0FXWTU1SHlLY3ZqSmxPNGxwLVBBSW5KRmp3djdoNGQwZ05RZWYyVmpQM3hvQ3RYaThOZnBWSXFOaDVMUUtweFd4d0JPZXVlbmU0UGUtc3hLajZHSXpIQkNVRnZUY2VESmF1VFNPSVN2alNHQVZCYzd3RkRIdUY3MVB2VDh4cV9JWWdGbm1qNDNCdzZQLVRJUVJGREprd3YyMGdBc0JUMl9lWWZ5ZWZWTWV3b1VaNWcyQmI1cW94SXRpVm5zS280MUFpYlZpOHMxMk9OR1JJcmVCZm9yWldWcGI4Mzdma3JTNWowcEtwTnVkcTllQnZhcm9tVGpFd21MZkVNQWtObjhheDFTUnBDam5HT2VHODB5ZEdvRGFnWUFNSk1saEtxMW9TcjRQWU43STYtWkp5eS1NaGkzMlM2ZFVxRlJFQVEtNkFSVW1kSXdtYTVMZW9NRllmcFA5MHcxVWtUVUJ1UjJkTFJDb2ZvbzBVdlIwNUJqOEctY3NuSXgwU0U1YUNIdnBKaWozMjdwLUw0aHlJZzU5N1F1bWlsTWRtdl82WDV0VzVYSEdvNVVhR3AwdEdyeDBodEpWa0txa2pqbXVBQ3BsY2hvQjUxTnVQNXotcG1PME54U2FHRVpxM1AxX1VrY0RfN1U0a1hqVWhiUlFtSmdma2tUSm1IUlJVYVIzZ3htbUNqaUZJT09GeW5OU2Q2WURPRGxkR1NJbWlpcEFSRnhlRVdWekJKVHlGTFZjWWZ0Zk5JVWcyVnl5dUo1TFVUM2Q3TXZsYmpjQWI0dGROZ0lQNzBsLWJfN09Xdm1UU203WDFtNVdQanB6RFJmUlZRSHc4d3ZWR0tpQWtHZmMyUHNHNFB4WmpqYXktVVJRLUlYVE5vaVFQUkV2b3BwdmFxazRjZV9FQ21tejVfVDRnaEdIcUhHMTVuZ0VnbEw3ZFNram1lb1pacFBrbjRsbWZFTER5OWhvRnFHTVFDdWFxZXRQREJmWjNUVWRqTFZsREVVMVVaTXlFOHhreVZuSFlFb3RZcnZoQl9SZGsyMDlCSGxSZ3Q5Q2x5RGtNS2xOOEptNGlfeXFOdklsbDhScldrODVQWUVFeUVia2l2TTFvSjM0aTB4NmFzeEZScUJqTk8zRXpsb2cxZEM3VDFaUW9La2k0U2hMYXRCT2V1VG0tRXZqdUlpUTdZQW5nZEktNTA1Sms5SFhxMWRtUnd1azA5cklvVW11Y2FoaE4xaDVnekp3b0tVOGdoVFJWVTdNczhvR0xKb09hRmczUDNFaWVWMmtRU1c3TlFNNFFyOFBwbDZxa2thdUlRemFmb3luNEJYX2E5cHFDTmxFZmptcDFPM1FWTVF4Y0xuUF9sYVExNkhLSVoydnltb2JQT01sWGVyblFGdmhpdERQbVBVS3FwRnBmXzgwU3BwOWtsU3d3TFIzckJ1Uzc5NkdmNldTNEc0QUtrcDAtbGdnSE04ZzhyZ0VLdmlCNlVRQllyNFdheV9oQWxIOVpPcm5KdnV6RHpCRjhxY2pwRHJ5UVdTWjFfbk1MeEZELTl2X1V2NUFvX2J5aHgwcGdiMlJfZkczdTc5OXR1LVE0Wlk0TnBUVmhmazJoMWRHdDlQd1RYZE8wNWNFVnQ1VUE4QnVqcGJXT3hOQTBQa0x6elg1amRaZ0pLcEtLVTM4QjItSmtHRnFHWDZQOVJwNnFodl9rQy15NkkxMlh3NkU5UloxNE1SczV6NWNsc044dXFkS044RE5JUm0tQ2pCalRXVGt6VTU4Zldsb3JMbm9kaWVRTmZRT0FZV2ljWkdyZ1NPejJoNUxMNFBITXJfRlNrbGI0ZU1Bb21haWVpUzhUYXIxcVlveHJYZHF0R1VPMWZCOHRqQ3JxRXIyeWxTay1kTVdZSnpsSkRVc214T013ZDYyWm1ZX3FEWWFWMWp2ZnhIN0xXM1htTFR2cmdRazd2NUo1X1JpRVdWRkNhODJNcmRpcllNWmVMSGFqMmg4VWwtSWwwUk4yc0NwMWpTUW1tdlZ2eWowUEJLaURhQzBvXzBuNm1UalRHZzE1NmxJWm9LOFJZMkRaS0xyYVlmUDhfRVk4SElJMUMtNFcwWEhlQlI5dVlRSnR0SHBWdEM5NWlBVDRZcnhiXzdqSFUyYzZyQmczNGI2WXFBeVg0ak41NzdPREFUeWtLRzQ5MVFyMi0wX1g3eUtnN0ZQOGNZcldfeHFGMWtmcTNnY3cxVjRCbmxVMmMtZnZSbldkamsxbGhQdkZDNVg2WXNicDdBUXpBX1ZKdUtNM3pMdkJaYzVocEFMZmpXVWFueHpCcHdCUE10WXZVUGg0bWVGc200aDZZVS0wN3Q5NTZUWk5OV184alZfdUN5eHFza0VfdkQ3aTN4ZDdxMmtCRlE1eHhhb19KQThNekM5SW5GYy0zaFNMXzFRaW1NUEZLZ0xIZlhpUlNTVDByVkpXdkIxVlBOckQxZmNjTUN3YnlGYTVCbXVQSWE5MFg1VGhfeGhmcWptTjVEV1ZKbnRZUDVqTFRtOXZZVk1QTU1pTUFkQmdqcFE4UEYwaXBKdjU1VWtsbjJFbmdvNjBkaFZnWk8xSDBNdmxEVmhBM2JqeGNkUkVKNWZ0YXBQQ0tiREdmbVhvQXBoZnBaZHgxWnVrSEZ1emJwdGRxY2stQzJzM05pUGFTaE9VNTVJRkxuWjByeWFBWk5DUml3aW9vQTZlVTVJWFVqNlFyWlU1MFRBU0lZMXNLS19vcmNqLThJRDFqbmRIdGdMZm5rVy1QTmRoM3RlLWlKR0VNVWJsbFhLVWhxTXZ5VmNieUp6OVpNeUF2VVh3OS1YdWdJOFVEX196UDdoUXp6OVoxWnJuZktOeV90T1J4SlMzdlBoLUZaYXBUTEoyOGh0VWdNSHNRZW9oNUVfYzhxUjNSUG9iaFpzY2pEeEd0UEVKSFhqWFhCY1VicUNia2JmUmNaSXd2LU94cC1YaEprMTNqNk52YnRQZEhQUjdSa05mUGczXzJYQmZFTG9DMDdmTDR6bkJCZUNtSWUwUjVTTkVtdmhVTG15QmhmbFB3VFNOVVJLdW5nbG91OWNDQ1pQc3A5ZFV4LW9wN1FBSjdhTGhNYXA0T0JPTzBxZTJ4eHNITktzYnE2OU1uTlBEUUJUbVhfdUdvVVo3SUhyZW9SX3NzWEkzQVM3SjhxV0RIRHVLbm92QUZfSFItYTI3dGtsbHRjVHlvT01lNWEwZlNsSFFRalpNNVRYNWZleVBadm5CR1A3Rms1ekx5TThNM19kRkF2SjdUdlNHOTF4dFU2YjNvZEpqUHdRU0dOT2R1eFYxNVVBTVJuVzUxbXFhbktmeGxwQUdjdFc4aDBUQ2dnVjBsdFRTcFphTHRXdDFSN3d6NjRmaFZ1STBZRHJEbWVUd3dRbDVlZE1zdUUzdVkyM25uTkFfTGx4Z2REQUJ0ekVONWlXOWpicWVLajRnRWZmM3pPUElya0s2c1RMTUlmekZacGhuQnAtLXFDcWJ4V3RtSzYtU0NETUt3aHhrZDE0QU9mM0NvV1NEbGM5VzhJdGUyV3NzTnJfZXZETVJzTXI3amZRQTFsWEtPV0hfbzVxRTB3TElZd3hhN2JUdF9fZU04MzlDUG1tdDFFY0hENTE2SUZjN3d4eWlOdGhOcW01bTVZck81bXRyUlB0V2lMYnJ3bHYxZy1zZTlic2ZsTzliMlo4UWVDZEdlVkxZcjVTcVFGT2p5NjlxTWs0LWpfU2NuNEw1ZURPbFp3bWRPLVJKdUJ1US1MejJlV3Z4NXpfWjlWZ2Q4aThPOVExNER5Zk85dU1XSi00VTloUlZjR2RQSE12MDVkR2ZqcklGWVR4bGFUWmIzb1ZKbDhwM25YSE1lTFVETUFjWmhVOHlsV2ZvSTZuMEF5NVcyVnJnUkItSHd6cWxJdHRweklybm1PZk5SWUg1U0MzVzV2bkdzLWpkLVdxamx2ZEJXOWJUcHhSSVZsY3R5NEZ4MzRHdTVSemExUjE3a2ZCWDdaaFVSWnZQdWxTMV9iNGxkWmdIRWdSV0FsaWtCWnhMakNubzF2SDhBRkVqUHNqbzBwbUxsRnBaS3RDcjgtSXVrMHhhOTRIb1lzMnFzUVdzRVdidHR4Unh6SXpNRWFYVXZHS2xqOC1WZExGX1lISExWanItUVNid1ZvU3FoaWU4X0lVSGlGSk5LbXJ2cXBvQk5BMmFTTDE1NFNKSHFjczhTLWRPLURUejFmdklfamlfR1ZtU3loNFhIcV9oQWlBNzV6djF3RVpIU2ZEbDBNUnBGTTNrVW8tQnhBY29tT1N3RjFReHMxbG5lbHlNOWZaU3NWWGxRSW5wZDNDUmdlNUhxcGJESjJFLVRlaDNEdUZYcWNvRVhUZmJtNnJBZ0o4d3ctWmVOSmhnR1JLRHN5MmVOMnByMEwxMUJmMDlOWGNFSmVMUHFNVzJPVGt3OWNCc210LUdXTDlLcTBkQ19DYmNXVW9iUHlwbnE5eVBhcEtZa25WdXR4anJOV1BoUnl4TTNvLWpoeEpCTnFhc3pGcDd2eG93SUZRM2xKTTZpUEEzU1FqTnpJdm1SNm1HalByS1BNendCSlBOeFY4dHJ2M1lOR0g3YW5BOUh6REhWbEl1dk9ZM0lYbHFTQ2daV1Y3ZjNWanVrbnhtSHJ3OVAwNUlqZ2JqT2piSG9yRWVUU0lkR1RpUk43Qjc4enRpbHhndHk0SkpmMnJ0OUczdDhyc3F3cF9GNVhIWW1NOFJscFRXMjJwenpBNHgyMEdCbGhTWExkM2JaUndWdGt2c09HcS1kZDB1eWtYYy0waDUwNmJ2cEZ4c05uUzZDRUNmTldpQXdKUTVtQTdwUHU2NjhNSGNQMjhkOXZGYXMwQUM2TzBNU24xdEVENzdBUHRIUW1NQVVQUjNpUTRTajFNdEFZMVE0alhBQy12OFhxWVBnVEI5Rm9GYTdxcjFwMGwtYVBueDNub0o2bERTbWpaOXA1M1hQbE5qdHZnTVhTUHk2X1dtamY5MzRmcW9zT1N3STNIT2h0TXVPd0pwTkNXMHJyWU5fWEVVRXZqZWdmVncxU0VGU1VmVVZIcWQ3RmVWWUQwOUhmZmJUMGVIUWJOZEtrekNKdnJPOWNRZUxrdUZrRGk4a3pVWERHOW5pOGRaMUVnT2VjalpDaTJ1VEQyTWFjdDhla2NjV0tudzd1OHk5RGdGY3dMMmNQWTRqUGlTUVVTa3FsV3dTWEJ5TDQzSnUycWtkNWpZekpDNmRsanpuMzR3S1VrMy0zTFhOQjRISm9VZzF4N1Q4bFl4ekJtOF91TndqcHNpQVNtQmF0ZFQwM05zNFVJNnF2T0owY2tMRnVyaDczMU5aTnhkNGxnZ04tZmJQbGMwMzBBX3RUN0JieWhqdEFoNDVLWWp6cWVnb0I5aEdvMXJ0amtVRGRrM2RlRkhXSFJIYjZRbURuVmF6ZDF6SFhvbEVUNkd2TXk1LXExV2NNYWZxU0I2dWloWEc5WTQwNUJXZnhmeEZCejQ2dXQxRkJMeDRGb29DeWpZbVJYLVFKY1kydVJKcF80MnJYMW1jVU5laUFMdWdER08xX1JhdHNvc04wcHRkZmF5Vm9oSlVzT1VMSVFYay1jQktJN1ZjTTNKNi12NXJvQUQ5ZFlfdTgyYTFtdE5FU1JzMW1NcThwU21xTHBwSk1WWlJSRll1TlVDcHRVekhLQmlzQUR1c0hlNkM2c0Z1Ym1TWFhZUHNwWTU1SlBZSktoQVJoZUVHNE1kSzRLMnc2ZXN4UTVEdU9jbG9OMFJMYU9ja0lLNURHREtjbmUwVHEwTm12UDdlcXlJSWltWExjbGxaaHFzVHRrRnRzb1poUFpHZGlBLTZGdzRkOHUzNmJBczRJOWxEMkpjMzVUd2FlY1dvY1VJcFM4N2IyNTZ2SFNpdnNJWkdITml5c1c4b1Qydzk4LVIydGxhNG5PUjFKOHNaSmdhaFJERkM2NmFYZ1RIVzFILVdCMjRXN1hvenFsTEFpdWNKWXZ3V002TE90LW5NYVN0a2hXUFVUamlBUVlTZm1wdVNNeU1TZExaaDRpYWI5TG5vb3RhbmhBeTN0LS11dGJWMkZjQlJFR0x3MllwUjU3aUFlUlItcnJ2TlNHbVYxb2VKay04SnhibUpBVkVMYmRVeUZGUUprd1ZJcUQzRjJmanJMNjNFRTVEa29RRjBTbkNNdGxqWmdXU0tpd3NkaWFZa0pPTzVZNnlydy1zYmlLMUtUaVVuc241MEZuODlEN3k4OTl0alhzcnNHT0txUWY3M2xjTDlnMXhkTGdlVWxXSHF4Z2NSanlkSGpjYjQyNDBDZjBYeDhCTHFyZWtTYWZXUTAwbmtvVGwwV2ZOcFZKVG1zQ3VVZV9nb1RhWVpIODY2M2hjTjJRQTBHeGJZaTVwajVHOEFaRlpvU25LaGVDWk1meXZDalNjZm5ienJDazdkUFdyUkpNMy1SS2JTU0VBY0N2dFlQZ1ZuUVEzLW1tVU1VWTBia3lvd2RIb1N6YUVsMXlrZzlqY21WMW0zck1yazkzdUtncW1jRnloOXJBN1FiNTZDR0ItZHY0ejZ2UVJYSFRJZjB5UU1HWExtd1BwM09rS0REbTVfWGtzWTZVTmMwVHlMQ1JGazVHWGpQaFo0Qk9CT1NZMDZzWnFYOXZxZXNSNXAtU0t0d2ZYYzY3Y0NCMlJhMlFjZVNmdzlhS2QwN1BnZ19Wc2w4bWFieC05Zm5McTd0LWFvUkxhMllNendWNllqZ2doaUpicmViLUdYMDFsenBidWJ5YzNxOElGbl9MUkxuclZlMlBHTDNEdUtjekJ4Rlp2X0tGMENhNzhrc2RSUExTUXNRU0QyT2k0dWx5eVBXSTlGWlJZbGtEMEp3MzFra0hUSGtvckdON0VMaVl4QXZDWXFZVmpndU4xVVFtb2J2d3UySHhfVDlCVEtra1BkTHpwUF9VMUhqanNRd0RpeUtPa2RYQUtTUWxaWDJtbkdHZnNWbFlEeEZRVnpvczlyVVJCZGgtUnFhNUJrc2drYlFUekVHRW1YLVFudWNSVmoyelgwSXQ3a21nZTZUdHlFRG13ajZPTGNhbFV4b01TVXdjTlF4b0s4SHVDT1NIRS00bDVuV1AyWjlPdjFIdzR4RDJPQ2g2RnRzQ1ZJUDVTMlJQYXg5ZVNGdEloTWkzamN5SC1Xbm41NG9VNW1rQ2I0R3BWWl9GWmtrNmpHWTRQN0hWa3BsZ19nc19pcC1Qc3kzWDNtZjNIbGdoRjc2LWlXaDFiTXE0R2N3dkNuNDdmWUd1b0FxUXdjOUpvblItVUlvVWJIc1NHNGtyY3diek43bUJtYjljQURlVzNUSlpuM2xlUHNway10cWhycWJqbUpZa0hYeGFOS21hVTdZZWJMYWRxSnVaYmpfLU1xdGxyS21IMGdBa3VGXzlsbEdUNXBDWk1iQ05QUnBsT0pOQ0JfNWVTTFhqc0ZmUWZtRmlTSTV0ZXJIVzU0UkVGSTV6VlpLLTg1bl9rNTJRbE9ncE1MN3d0c1ZJY2p4cUU0OXJiMjdBRHVEc2pvRVdlbEVlRTdCeHhEWjVJNkF2SldrXzAxZkNwa1ZhVkh6U2NFNGp6T1BnN0ZNYkEzQXR4NE50VVhrUnZYMTJOZEhQWVlZaHpGY2UxVDVnVFJQZUpPSTlzckZtXzlZbVdBNHpBZGNjWGJXTkszVjlCUFdMMzRuWFZKRURFeWNveG9NczB6ZUtYYTBfY0hvSnZJOVd1Yk5lVmRSNmRxUVhmZkdfWTB1V3A0X0ZoaEhqSlRCd283MGN2WnFIX1Y3R29Gb1dPRXgza3JNeTZvMkNVTHFnUUtTVjNaaVpmeDlQeVdXVVV2OUhRM0hQZHMzOVAtMk5aVTFGSF9acEw4NERYaVRfODN6QW9EdS1VTFppWFl6NnM4N1FSRzhoRnVsZXNXTWFLbWM2ZTFhS0VtZVZTWDNzN2xMSWdjNWtRR0Y1OVhnVDdQd0NIQktJX1VVOEdMMWx5TmUtajBUZTdtT0VBbGFuV28yODBBendKRWdoX2d3OFcyc1A5N0hfRnNGNEF5TDVFTjdkU2VSMENsM0xTR0kxTi0tYmU5ZThsbHNqRHFKV1d4NlNwUzZZQkliQlRhcGhhZW5TaTdGM002QzYwN3pZVTgyVVFtclQ1UXpsdk9EUHlBZU9rN0hhYUNjVG5rXzlZblItaHE2RWRvSGQzZm54RFd3VTFRM3F4Z3g1Rk9kRlJ6TnVNT3NEdEd4U1ZvQW4teEpDSUt5dk5GT0ltNHc1T2hVQzMxWm5OX0VVRXJITnByN2hkcWZ0YlpxZlhsUXQ3bGotMWRTcXJLMHU5TWkyMk9PWDl3MGdpNklad0hGclBzem1QdHVzMTFLNWVTQU5LVXlNNXRzODdCV2N2U3dvX1RoMW83a2I5czhTWkVBMVp6MzZLaE5LZ0hqTkFvelJ5eHQwYlhkeDlBRkozSzZxbkJ6RGNvWklnazVjdEQ0WS11QTNobDd6Y3BocUhYdnBxazdqWDBhU2VSeWdRcXppMUF4VlBWZzNONGc2eTlVRUFwZUdsTnRyMHJCVkdYRTJ5NWdfZ2xGanRjWl9ZUTh2VUdXLUo1QmRpRjF3UlBLbEJYQVQtNHBmV1BET01CS0wzTkNPcGo5SlBoUzZ2OFI5cFh1WVBENUtPYTNjbzZLVjZhSTFaZW9PNUF2dDlQenpXandwLWo5VnVDN3EyRC01SUdOV3Vva0JmcU9QZldHS0ozUkJ2N0lqWldoTnJmZ1hvUXNqNlhqMDE4VUN6clZ4ZWVRdDByRkVsNk5adTNReFVKcWN3NzItcnB0dEdyWnVYMnlHZXVJdFBtQmkwM3BJcXM0dEdpMkxiN29adUM4OEtwSFBaRU1SM05TVjZ3cGZXMUhRYU5FdXB2ZVFjclVvNjVxN05uQjVfMjFUR1ZZWlc1VTNEXzZ1MXFhZVIyaDltNl9MS2x0Vjg3cDhDaW43SHJFN1BqcG50X0NhbWo2NXpFd2tGQzlDSUxkdFB6cFBmTmVNUWhKNzZqcjhEN3h2cHJTNGN2cVpESXRGVktrWXotbE8yWGIyMHRfYWFaTGJURlF3bnV2X0tBYjJ3aDRwQ2xDbTl6V2ZxN0ZFUlVuSnJPN00wbVc5T2V0NG9MUVhlTlNoU0tUTmJmYjFTeC02bWdPZ18xbnNrakdXRmFVVHRTZmZtVTE0dVJlQnpIQkxQQUhFbDBSTVkzZHhrWjBNUWFmdS1RUTN2amRVZnFiQzBQOFRVT3AwUElKVVJKMzl2Z1lDMVZjcjhfNUU1V3Q3R2tMVktNTldESHRxbF9YcExqQmY3bjZCX2x3ak9SNW1VSl90QU1kTTBWQ09mNk84WE54NENRVWNxbW1zXzMtUnpmcEEzZl9Ydk9EbzFUQ2R5YXNSMGFMUnQ0ajB1RGwzX185X0NwcklRVG5McTBORk8wRlczamRHZU93QnBVaHJybjlpTGxTakQ0WE53enRCRklJSkpUVFFENXVudXhZMEFIYzVoSXR1X2xscGp6eFhDSThqZFNGNVhtd3p0eE0yVjM4azVuWDhkVVliUzZOMTRKRkEyVVYyeEdqbDdRMlhILW50a2lxT0p0MFF6Mi01djdsX3pjQV9kV2NBRm4zYkd3SjgtQ05BZlNkS0h1LU83NWFjQ2JZR0c3U1l3TGJwc3FPTTMyQzJ4Rkp4eFRlZ2JJc19JRVQ5dGhETEhVLWNVSTFoeGVjalRWVU5WVkRlNTRTTjRuSEZMMFVOWG55UTlrdWMybG1fOUt1REQxOS1yVk92M3E3Z2lHNW5IcW5xTzF3VExBbVp0Z1RabGFrX1ByN2NxdnNyZ3Faem9ROWV0WmZRenhIWkRSVlgxTWZyR2oxVVlmRmtFWjMxR2hqNU9CNnJrS0V0eVBjNVdJektDNEZIYU5yejZWOV9WdmN1cW1fLTNKUk1ORVlBWjZKZTFQTk1LWktzZ0RHTXQxamFpYVk3X1BiQjBxWTRGYUk4N29hSGVFWWhpTkhST2ZKUFpyQ0FwUlUwN0w1Q1hDNWdjS2NfV0JQX3hqNlRwZHJscEVCSE12T1ROSkZFcENEOHl1LWZyemZ1d2hMNHRHQ2dCVEJkNFI1S1hobnRtOFV0blVSQVdveV9MbkFoZ3JQczhJUTdUam84M00zc2NnMDV6cG9nOE43TzEyM2RpSzBVTkJXdzREWHdZVzdyNWJFT3lUYW12U25xUUtEOG9EWkgwdG1tNFpFOUdmc3dmM0ZaWWR2T3dMRVpfUVozMWdzal9mTFhCUjItQUw2bVItb3I4eWNWU3RKaVlDV0packxOekJfRFYySGFMMVBaRnJzblFzT2JJbkZvUE1mYmZxNWpKMjB6OWJLaldKelZlZXdyMTFZNjhEZElZdlYwUzZwMlBEWWpKV09GQ1kyLXhNOVlmQ0lrVTNHYm10V1dLcDVKNWZWZGlNekduVWRZN2NUcnM3a2Q1NlBCX2ZTVkVSWGJiMTZHTWZqN1ZKZmdrVmtXWWx5YThOTnNuSWtyWkFveDRHZ0NDS2RnanMzd0xRMnZOc3JLVk5YRHMyNS1OaExzeWg5azZtZkpnNkwtSHVMOWJoTEhqcHExaWUxVHc1THMxSzN2b3JMNzlDeG83VXlieWUyR3hsWjQydDVsYVVYbXZXOTlVZ1ZIUmlmN3NXUjJCQnFCNE1iR0pHR1NhSEF5TUpleHB6MGJFWm1tT0NiQVVadXJPcFlHT0Fhc0JuX3B3Q0F0YjhmcDFTN0NmSWF4VjhiM3RQeDJtQWF6cnB0MFlwZ0VocWw4alFDanl4bGFrczZQNlRZWWR3NnhfTExZeGJMcnY1UHJsZ0RIc241Tk1DdWEtTnNJbTI5V29CamU3X0hPbzNjYXgwZ0lkVU5WVFV6bzFxWGlaRTlVRGJzMEM3c0puUXd2cFNodzZjRkwzSWFBY0UtZ0F5UmlXNDZFX3dvV29GOVNJcHZwQ19iMmhHZWpsazZ5UzNsa0ZjRi11LWhzUE1wLXNoazFQaVJjRXNJTnN2cnlkTnRYRmVSV3JCU190MEN5YmFNQ0dQclNnN1RrNmN1Tk5sLUttR09PdU5LbFlKX1J3dXE3QlRuWUM4VFpCYVBBVWx5R01Uams4V0JQaURQclp3dUxsMmZNSzlleFd5cGNLdXhVRlJodVM1QnF3Z3o5NWotUmVUMUtwc2xqZGRPZkk2QmZUVFJKVGVTMGNDT0QwZF90dHpoaVppOEhzaHhmZ1FGQzhud2NXN3g3Y1VZbGJ6YndTdUI2Wkx1VTc5ZG13Mkh4NGVKNDc2WEtIVW1yZzB6aFROYnQ4aWpUN3Axek9vSmdkUWZ5d0wtYzJ3bXpyMzJPRzB0eUEtV0VPTWI3dXNJRUlCY0ZwdlA4UjlHeFV6Rk51RGJZRFI1OVRnbzNRWEJtTS1OblVyZFY3RXplNklFbXVwV3JFRzZVdUNINENkektITjZDY3FNY0tia1VtT3BRQ0syejJNdy16QWVraHMxaFRnN1lSbjVEdjAwakFjS1U1UWNpUGo2OVcta2FSR0RvT3Nfb2c1X01jWGpGM25tSVRsengyTk56ZlJFQjA4Slp3SEk2WGZySVlIODJremR6Z3VwYmQ2M1VqMzJacGJjRWZiZWw0OEVMeU9aMHpTRU9HZ0NzbEVTeXF4U1B3cWRQeWlURm5SNkF6YWRNRHFqdURVYU54WVdTY3MwaEk2YndzYnc1TElRRjBYNnFEeEc1My1ldEJtSndVc2Y3OHlobS1KLThiY3F3SFpMTXZacEtqYlB0TkNabHZ1RVo4NDJoX0FDOV93Ul91VEF6OE9CR2pjTjdtLUpabkZ3Tzc2anZFbzNyQ1NXN0luVWo3Tm5TZGwxV3dBZUdpWUJqM25FaWxFMWd0QWFZQmIwU09JMEdoWXUtV3c4bGRtUzdGYy1SYnNOR1VOMHRQV0dHSlFLZGZET1lOY3lsMXcwTFVRM2w5dXZkV09yZzZvaFk5c0p2TThHUTRvRjVGWkpVT0tKSnpMNG5reUtOTGpRNGFvTU9ybWNZejk4Y05TM2xmZ2RVbU9mZVpWSV9oSUlsZFlUNVBfMElma0FBR0JZNTExSDU2SU5KQTNtN0ViZW9WUDFqeFFIZWF1SWF3WDVEODNkN1dzRTdBcXBaOFFiSEFIUTdXODdkSGZvdWxxTjVTSzZqd3Z0RHZqR3d0LUVwMTZieFNsbEtRYWdrbDZvU3pYUThtMnhsTjFqTDl0N1VQQnJCenZvQ1FQbXRlaXVUQ1l1dUtNdDdUcGo3SUJWTXFmS0VjVEZ2ZDNJcExkLTFwc2tMd1ZlbVo2TThGSXJ0S3NMM3RHaURQLXpZOTh3aHN6dy11RW1BZzN3bXdEcVpyMTRwTlBwcC14cUgyQ09OaVcxN2p3TWFQSzg0Um9tU3RPVXpDMVZKTU9VR0VJdDFuSkJiemtMeWhNYWgtYkoyX1ZsRk1JS0ZIMEdGSWVJaVZxSHA0ZE5qYkRINUtwTXRfM1h1cmtKc1VlUFBqVnBiVW5RbHllUk1nMmtnaVBOdEpTZkluVndpTFFVM1JpZEtNZkRFcjhyZ25zQjB1VXB2Sllwb2NLUzhWXzlnU3NpaVhHSnRDUjJIdEZncGduWEpocWNzcTZBeDgwZTVLTXJoN1BPSkwxRzNJV0FsOVhVRGpJVWR4NWtGUWdURzVsb1FWNTBKWHY4VHJpVWpsMTZ4eFBTYmREeFJvQ1ozT2VFRzRzZzlfTDB5Ym15ODdHSF9UbFRwY1laQTJrNFNleFFkUkZwV01RMnU2cXc3VUFyQ1BnLUl2eVAzcnU2SjVDQUVQem9RNHNIUGhSWGVjSFZrQU1McHVGcHZfM3o3SkNCT091ZGFYTlY3NnZ1dFgzNjBOT2U3R3hQWTBDekNPWFhqeXVzVEFlZ0V6ZlhXZTFQSmVLcVZRVlgyaTVaZVlXQlA0WFJDOFc0TlFGTFpkQzJLQWRkcl9DRGR3ZTZERXl2UkJjanFBdUVGR3g3MXppZ0laZ2UxeVExaGNqWFJUSTZNRU5GSmZ0WklrV293eDBYUEtUQ01jWm8zME01UkpacnpCTkw0UGRMWVV5TTZXemIzTWF5VEtLVEY5QUphOHdhVzdNQ3JmUlJ5S3puXzBMaHNKN0NMUTc2R3VjR0k5MWZmaFVfWWphYkllaWRURWV6X0Y1TzNDc3pWNlM0UDVHcFBnUUJpbURpLTdkT0NZUUFELVo2TU9WNDZEZlNPb1hLRW1VSDh5OE00cEpuZkhLQm9yWEg4V3AyVTFZbzdQbHJkeHZ0aTJQSGhoOC1LZ25NMmlfVHlldEhscEVFWi1yTjF5My0xRzE5UjVHbFlJQWYwZTZtckNkYnJBYUluc1NOZUMxcTgyd2NQSk9QZXU3OFBsX2lqMXBCby1DcXg0WnFUT0xFSnRUUXpYcFNFTjk3SzBUeml4NFJkSk5Lb0hfNkRHZmRkMEF4OFRGcXlzM3FZVVNFWDlEODl2REpvc3MxbzFDTVdSVVd3dmpPeGJjTWNxMzNFR25HZ0ZrSEVlOFdlVkVISW9xNWRjd2o0bzdzZGhmTFY5ckZIN2Y4cElFMFhrT09fc3BnYXpDdlVTWlhlcV83a2FhT0dacGZxQVBTNjZaeTdWcUF1RWZzZnZ2dFoyWWE0OV9KQVhXZDNKV19YN2lvWjlRQllSdzE1X3VNNi1zLWFObFBIQkI3akItZnNkM1ZGcjlTbUtBcHlld2h6cnRjVVlyb1Y2bzdNdUF5MlFqcFVVZElGcDdEWTJJckdKNkstTjZpenlRVkNzRWpEVjNlcDZuTzF5TDByaHZNLVJSbE51TzE0aUFBR3pqQTdIcGM3SEJnNVFEUFBiR2NkLU5JUjFYTnRUZE9fVjlLa2dVS3U2WGtsaXczYUgxeW5CWHg3ZWtzbTJWN3I5dzJLSFNnd3BoOGt5Z0o1Q1daSXhJQWJxNnZFdEJFLUpVc081V2pCR3FDWmRNbDM4RWlYWnJBdjE0eDlxMjQzZHpWUkxSOExYdm9tanVick9NZU56UlNrS0o1WFlaMDVjeFh6TDBvZ24zQ2VBTUs5cGJrUEJlYUZQNTVnTWpJbUE3NzF2NGlORXFxVlo0N3d4elhLQ29rdmZlaDY0cHRxOVl5Tmhmb2Y4b1VaYWl2ZUlYYjd0bkEzd1JfTG5RWW5PQW03TkFFSWpVcE92ZlhSbWRfajljMVdFdzVHZzBxOG5WSU1oLUEybWtIUm50NU5VOS1hV3d0ZXRuRlRxd25vYUlmc3pZbEVHZ2RTRzZhWjhIU2ZveVdrTU5Hbm5LU1VodUJvdFM1b3ByRUJZb0VpSmN1YWF1V0F4MWRDa2VsZEszZEVUV0gzN3BaRnNWZ0FhX3RrTFNLdVkzOHFfR0NnOFBCWFdjbDJJTXZSTW0xQXlvLUYzbVcteXF0dWxtZm1BRWpkMEtrWDZNek9FUTRDNEVTWEI0dVRJS1BqMkpCX2pMeEZQaFdVbFo4OGxHcjQwLTRZTWRXbEk4RndJYnprU0VVdWlyOGtPRnVfUEE0Q3Y2akZxeVZ4RlZRMmRCWEphWDNBZm5iWFlDZmJJZGx1UWdfb0RaZzRxZDgxU2JtU2lsQjhSVjYwVXFLZ05hdGtaaHVJaXBscHQzaXh1RDBnaXhsb2YxbWJST3ZmaHFveXpGaGFKWVNQd0JKQjJUYlRZaURBdlJXYjh6UnVHZVJNQ1JNQ08xRTlfcTlDd3F0UmoyM0lvQ1NCbzJXWE9JSzg2OFZ3elpmeURzRk9CN2prckJPNndtOExGZ0xPbkdNVVlNWE1BczRRWVpWdFlEWmVEamNPbUtXZWtLYmVfTGlHVXI5cFlJNm1TY2hUbXFnTWhkQjJTUUFYYmJuV3B6MnVPNXZDZ2VWOU5hbTBPWTJSZDBGNkw2NUFJTjZjamVJS0RfUXV2VEM2Xy1MMVFWUThaZzB3MEUtd2IwN0htbHl4SFdxb25NckE4a0p3LWlyS0p4eTB4YWZMcGluaTRHbFltSW96dWdDR0VfRDhnVkNfbWlORnc0ZHZBbnc4S0NXaEt5TlRZY0RjVVQ2ZTNGZ3Q1X2ZuNmhacFBzNVhUbVNkMm8zbWFnMTJ6YkZyZXQ0STMyeVJzb0xlM1VFdHQ0WTNPeTJDVlJtNWk2S3loUVNfdWZ3NHFfVXZIc2dDdGhxcmVKLTlINjlqX1g2c0JoNmZLS3Y1WHZsYUxlemtPYUs0aUlvVnBGTHZTTGpsZzczMTlkZnhxLWdaNUhiSUQzOWgtUEpFRTF5Wjg3cXdWRFAzblVtRVRYQ3Jza3lEZWYtWjRBaDdWRFM2c3c3Q3N0Q29WaGJLNEdWRTZrUTFwTVNvam9ZQy0zWkNfbWNRMGlZc0lQZ0ctY2w1aXhwUXB6aVByTnJaNmt6VzNqU2gwNjZ2RlB4M09kWkZaMmRQcWVEQ0lXTGZaU3JiMTZIT01iczJZc01JNlo4QlMzMDBGeVhfYUdlenExb29oMlRCdVRreXZJWHhCSWdBdTNZWE14by1QZkl2V1dzR0dXZUNFTHNXTlpMRk9ncUllZERMT3JPUW84U21xZnBoMjE5aWUxc3QwemJDLXFXSHpHMFRURmdvNHNydTNMNndVQ3duSGd1amFHeDRPcTJxQk1YTFNYY29zZ25sUng4VkxNR2RpNGtidW5iTUhyeUdWb1psM19mUVl4ZG1rQVBLV2YtbGlHZExBYWlWTGxMdVpvYWVVWEQxUmhDVGVlalIxcUxpdmtUSmdWNFNEdTRubDJCaUpjTVJJYW5iT1FlOVJCa2JUMGd2aXlBRjZPVmJyUGpQYTF6VXBFYmNjdTVNd05jRU1oa010cURMYVNPYlgwRWE4aE1ZOGkteTNTVmlxRG5lMWMwRzFzTTRlV0tqNE5FLVYyN2NsdUwtMjkwX0ZJWjZHdWFYZVNvZklUT3B2ZllaazBNRkJTYmQwMUEzU3gwQWZMY2VURHJaN1BrNjhFdUhBTmJjbHJLS25VZDcwX1VKc0l3emhDU05IOFVhamlMSDR4c0Fkd2owS2VPdEdMWTRVZjIwQVFNV09tOE9RYjU1SDRDWndBejB5Q01YM3RBbVgxcmlRNW15QjBBNzJUZ1pRbnlGV0JkTUlyM1RsZ2NSbTV0aDJ0ZkdYY1Y3WGMzLVRyMEdVdllSenNMTHk0WnNuLVpBNFFFXzVMUlNHM0xjMXJLVmxsWUlmWE1aNHRhTFQ4S1l1NVpQYVEtN0ppNm1BMGpSY3ozMkpJb1JSZUx1alVCb0tQbTRmYnhzZHVCc09rOEhJUVpoelFkT2lpU0RhdzNURFcwNXNSd2tLakh4cjh6ZXpjYzdPRF90Q0ZHTm1TRmdiVmxDR19VLWJxZjM2bTZPQWdKb2o5UEhOaFZXcXFiWFVDa1kzdWlycTZTZ1YwWmE0NVp1UUZsSUdPU0IxaWU2WXA3NFduWDZHeW9sSGJvUmtSS3Y2cFdsbGZOQUpyV3h3Z210S3NqSG1YeHd5NXhsTy1aYTlFT21GejlGcGV1Wk5yZWJzazg5UUJJODNfSUpycW9ZSk9mNzRFWWRRZ1JKejNrNkVxUEpqT0NTWi05MkF2Y1BlZ1JvejZDNUExOUFuM1hZcGtoSHlGSk01ejljem1STjlkSWFVNVQ3YUxkNWNaT1VxbzVaNTdoV3RZU29RNy1sLTlkR0hTNWdlVmJxTURFaG45NzUzLTVobDFWcmdCTGJWemJ1S0xMaGxBWnFvakFsUnlKYTI3ZFJxTnVEcFBsTmdLR0ZlbDlvQUQtSTRuc3BKbGFmYl9JcndacS1UQ3hqM2E5M3czWFJ3Y2hydmM2aV9McTAzVk16RGd5Nm5zWDN6SXpjSUZGTTlqbGxNeHc4S3dfWDE1WkNNMzZqQmRrZXgycUxFbTA4WXNRbjFhTGZGOWdycnZRSEdkeEFtWTJzSWNWRWpzbmh4eDFYT29WdjVUTXR6dmw0RGlNejhWQjgyZ1FHTFhvQTJyTmY1NHN6eVRib3l5UC1oUVVmYTNxYURuLUl4RVBhZGtPM3JVMFhpMFMyQ2o3ZGZ4VDNOd0pYTTVoYjBUQU1TN2VaR0pKWG1CeWxzcS1YRjNFb2FWY1ZxenUxU3p3b1hEekp2R1JrVFV3dlJiTzd5aEZDREVfdTBNbFhiTlFqbTIwTlA0aFFPVVpzdEVNbTk5OEJuRDdRQjMwRzBzMmFJeW9MZEFvcklTYk4wWnMwOGFXMnI1bGV0NHhQV21yc1BtWTY2ZEo2SGdpME1TaHVRX0FFNEVrSmtqMTZmaWFzbmJMYkVPQWt3dE55Tzg3NmxDSC0yem5JdlFPRFczWnlYcnRMaDRJcFljcnZBSTVTTHFUVUZzSmpBWmZBYlhZQlJ2eDFobzMxNmFGNW5sbHptclIxenJ4YU1qZDFvcU5YMm11RWtyWDM3SHM2V0pRVUNtVGhKb29lV3dDOHNQNjNLam5RUm5MZWRQNS03b0ZvRllCT3BmaVEwUGtlN2hoZk5zZkRwOTZkQmJyTUNCcEFBckxoQ3kyVGJtclloR2hJYnIwRGtBVUFBcUpMWldFWk8wWWdmdnU3YzQ1MzN1WlBaZzNHdzNpbkROUVlrbGZ2b201ZmQ3NDlMMFo0Tmd4dmhKUWNhMy1ySmliYWJzRkNZVzJHS0lzdndBV2VzX0dwQlQ4S1BJc0dKSFZQREwxM3lGQ0I0QUkza2xwZElRQUJmRFN0ckxSUTd5X2hLOVBTT2RMWW85a1BIZTFRN3hYS0Q3dVA2MC0tdXRiZnBZTHd0S2Q0WlY1Nll6c3A4YTgyZERKdHJYMEhHTXRMVnRBRm1fTUJxUkxwa25ld0pWX2lBdGE5c0NkRVlWM2ZIZ0doT2JjTjRUUFhoZzMyM2tldm16WmhNM25KNk04Q1l4YnZvY1NfNFRRTlRXczJTaklpR2laei11bkFNSjR5bTRqUmZ4UzRiNVdYZGtza1BlMUR3UWNOcURQMkM0WW5pd1hPZHlmR3hXVUNkU2IzYjR2UkRJWXg0akVRTDRfNTFVc1NWQ2lVY2hIajZ1b1JudnEwVHUwa0RCaE1YRVZiXzl0MWluS0s5MFFhOHE2azYxai02V0pEZzlWcEVzUkRPUlJ0LXlEdDZMR2hOSzlOTWN3ZmwyVWN1Tm5yLUR0V0wxN3FmNHhZRzdLVEhUQkNFc2htdUFmaEV0RXJOMy0wYklabE1TNFNUbGRqUjd3bDN4Q2VfajR5bGhuRzg0SG5kNVJPV1lnM2swVWVaUERMSmhxN2kxVEluWjBGNlJ2TDhrNmlrWTZtS3JNNDRaeW9qeEExdF9QUVBQSGR2bDVJcmFLd3B0bmZzWVdRaFBZT2JqeUh6TTJCUkhEZmhEQW8tbXhna3l5dU5IcHpsTElBZVNiLW5zb1dLTkRMWi1uQ21UTkN2RGw1SVFrX3lGSTFuU0lEQXpRQlEzYU1SQWthQjloNHY2SkVCYjV6SDhlcExrbEdocjJQOGVOQ2ZrMF9kak9oaGlkYktPWE5zbUhXV2YydG9vLWhWQXpfNDgzWDJKRU9vVlNBNm1nay1lUzBRRjBPQ1ZzaThlbXU3a1NNbERjSGIydjN2bzFrMTlsaDZQNWVnajdmOHc5SnhCcUI1Vm9EdDJCNFJ2bEhybnJ2b090R2JFLW5SRjFieEpTYUsyMGxVSllRMEx2ZDE3RDRPVHRLWENoalpUTlZuYTVpMWRrRVJmZjVGdVRnS0VFWEJRT1JVRmxqVjBXR2RfTWZqMDE1b1FkaWRvUzlFWFZudjJBWjJTRTNSS2tMMDJHLXJ4Y0daX0xsWWYzdnJFOEVFOWV0WGI1VktDajNsNVppdnZvWkxTYnZkS0tpNzJ5QWR5TnV1STJZbTdULUdYSHRISW5hbFpKWXZhT1Y4ZktLcnRuaEcyMVBHWU5xQ29wRWJWTS12LVdNLS05OXJNZVo0QnBfZ3ZNVDVYdEg4Y0tKMHBIRTM0S2w5Q0NWQlhOdW01Y3pEcktMUTdwYzZKeUwtcG9SdmdhQ1pBYVh1YXNIT0xXSGdnU1pJTzFmZjZRVkpDa3RTMFd1Wl9fbFJhUUlxSFJIYXVYRWFNUzc4a0ZrZFZXTGtSUWtPaTZHeHhXaHFlT29qT3BCY01kclBDcVQ4MG90VURHTUxkYmpLRDVRdlpmSWtESkQ0am1RYnF4b1lQdDBmanAxZEowbGF6Tm5Eay1PT1NhZVdrdjQ3cGtoTGhQLVRMSUdmZ0VpaGZZU1NfaFg3N256TF96UWVPdlpkM2dBMDlETURvNC1QYVVZajVlUnRqZWN6RURXVmtQM19NV0lyaF9qWHJ3UUNMd0N3N29sTFJoZENFcGZaUW82RzVhWHV4aG5oSTdVYU1jT2pYcFRCZy1fNUJKTFZXa1JNUV90cFR2bjNucXM3eFBoXzdfdnVfaHBJaGUyTTBGaFNtcEVlOXNGOFdFZjNCSWdqY3pkUi1vQnVMdnphQVFzMmVCNHBRVzYzYTVEczhBYy1KSkZHSFpPY2tzY0N5LWJwenF2OGJzOGltOFdNcUUzWVRTZktzOGx3Y2dzSlJPcldTVzJCR3hyYVlzTzRrQkExU0tfUUY5eEsyU3FRUzlrYm9OTWl6Y05oUFlzbVd0elB4NXl0ZlpOX3NlSVZjemI1WGZVWDFwQnRpeHdDaVRCSGV4RW5vYmdXcnNid2h1eUI2cTI5ZndDT2ZYVzlNX1htZDdXTVZuS0hNOG5XRndBTDl2UTVWOEVDN0JseUhwanVnU3dGeG1BYVBpUFpETlhCRFZQNWZBNmNBS0FxSGJ6SXJTWWZqYVBPUE9mV05RLUY5YkkyOG9aLWxPcGpTZHVzTTZXMHBSdnMzUXpHa25QWWU5VGw5VE1kMVBNa0FOTlhEZ1MwT2pnQ3BEc1FNYl9ZSVhoOFFNLWs3RXg5aXFvTldOdDJtR0YzY21yRXdIcUNzaHotbGpLaS01Njh3ZmhhbnJkWnJ1U1ptek1pTG1hcnRRMkVJUjB4amoyTXo4NXNXb1FLbkU3UjFKMkl4UWFnUDdHNndZMjUzYlZ6ekNqS3RVcDB3SFNtMkRNQVROTGk0amhteTZaeHJLLVNleHZwRVl4Z3d5aDR2aGJGRlNjNlJOZWN0Y0Z6ZUZqV3RpSDNwYlduanU4aDd1Yk16RFBIOWtheTcyZ25vZS1uTkdSSkh5cTRHdjYzRER0SVNoMWlnbmZJa1Y3NEF4enBDVDFPVTl5NGQtV2NnakRWSmhPbXRDZXc0RXdaZnV0eXh0cmJqS2NiZkpTcnd3Z1IzOWJ1X3lZbERQRnhUTDBXdmZHZEtDSXJPUVJTNkxDb0pRMkFFTTVWMGJZYWtvZTNEd2swZ1pvdXRhNzI2bkNHYWllTkhpSzJGQ3RHOUtDY0RXN2NqSWNOZFRLQlVVLVlNeVdoaXAtdTlick1xMUVkOWpXbVRWWVdZNG9pZlhJYmNYWmJvbG1JTUhqbGwwN2drMXFwMzNCa3Bua2Y1VmtDRFpqOE92WWxITHM2MHNIX21nX1RDNVNCVHJ0VkEzRjdaejMxQVRUSXBnUDJBQnhab0xJUXR0cGZhbmM0Wko1YUlmdGFJcEFHY0EyRTlkbVdwR2pOLUVONkRFZmlKd3huUXA2aFI0M2lpeE1iYUxWdjhzd1NYcDduVHJBOE9TMWMySzJwWkxpc2NuTkJWMEF3MTRGdmZOZjhFYVpNVENlUHpjemxsV0h3Q20tdzRQT2w1MWJYaG8tcEdCZUNsbm1NTFc1aEN1WjBBbGxVcGxRRHFWWGVNLUJHR3NsTm9tSGhiQ0xpOVdyRkFfNGIzUkFWa09pM1lhZkRsSkw5NVA2azZQbllETEdVOF9WcHZyUS1xM2xjRWNpRmV6cXpNUkNUcHZ5ZWU1ZkFzM3ZYMVIzbW5QTlk5RXktQVdtd3JmYzZsNVcwRlBnNnpfY2FuTktSU2hOUmdMMjZLa2ZtMUdta3l5SU9VUDJuR2RWSVBXZ0NFaFJnOEhFQUtKWmtWU25YMXllYnY0ejl3S080N1M2Q0hFTFNTaENaaGt2STZLdFJvaGc1cTNXdkNpVnUwdVRKalN4WGxaNTE0WjNHckxhVDI5LUJiMDBWMDNBeVVlVVA3elpNV29PV0hwbW5Ed09IYzJYbFFJN09FMnpOUTJuenZEb3ZxcUpXa3NRRjdKMHhXZjRtd0FPUTVtLVZBNldTZm5PUTZveFJ6X0VlekFMWWQybjAxMHRWSXFWR0tTTlhsWFpSNF9PcHo3OGZFTFlocEVOVUdWNmZkUmFGb2pYWWoyOGNkT2dldV9yLS1tbXlUSEJEMDFWQ1RnVmlHNFNGLUNDSnZMVWRDMm5wU3Rlbzl6R0U5a3U2Mmh2TG1VTGx6b1RVb2x4blQ1VWxaR3B6VVJCekVYX3BTVVB5dkdZajIxLU5NcUxtUFpLQlJZbGxFb2h3dHViQlQtVnpBVm9NdWdVdXFVQ1RZbGdCSmhncFlTSGNLblhrUkhCZzM3ZFFpVmR2amo3U3kwMFR6OWNVbUdremY3cHFxMjhmR09WTGRQdkk2bEFDUVo4RUo4a2pPZWg4WTV1U0xWS2VxRC1YblF5RUY3ZTFubm1oX25ZVXROVlUtZ05IcWViXzNKaW81MnplSy1tMkwtakNTeU5mZVZYTGxlT1JLMjlXTlpIQ1pwQ2M3cjNRbUZQMTZpSjgxMXBmV0R1amhqaWZlZHJVZ1B5blVsTG8yVW94bWduTHd0U3pJQk1TM2tCdlpPcHZQOFpfbi1FZjk1WmlCRGxwWk52Zm9vV1VHQTdUODlqcnYtYlVGYV85ZTYwUlphSWU0V0xpc3l4UkFEeEwtY2FDQWZQYlJNVkVtc0U2SmhHQXB3QUlMa2ZxeWdOcnVRaHFmcjd5blkzWDQ1eHhqbzloUDhacG4xNnlZcFFWbnllNURKODFlZ0JWd2V4UUhjWGhuM2ozLWFLQzJaZDdqbXljWnpDUTlEdDVHY1JJZUsyWHRCY2lWNEdiZDRRZGRQS3Jwa3ZqQkR5WHd6VWRkZWl0TUM3b21SNGpJVFF1ZURoSzJVeFM3REhjTW5MNW00czVmX3RiQ0JvTlJ3ZC1VejBtZmI1bHUtc044dFR6V1FfVXlkZFBEczNYQWcwSUViVjlrVkFVZEhhelhpV2J3ZDUzdEwyc0FINUxocDVLU2t2QUZIdEI1TUk2ZjZ3WG9BdWtRSHpqelhIS096WXlIM0EtTDNRVUhlbV83enpUeGZoZkFnaFZJc3ZkcV9HSW5DUW1PMzNmR2VwMFFLeGVQVTBMWTR6VFZxMmExTTFTdVhhQm5mREtjS0gxZ0c1VHdnbjdWejBIVllQQU02VnZnSER6Tkd4UVNVV3ZSQU1JRVl3ZkFqUkh1Z2NGN2I4SFBLTXY0RU0tQzlvXzR0ZWdFajhST2Rvaks4WXE2Q3dhRDRnalh5a2RGM05xN0lMUnBfRjBKQlBfaDBmazc1RlBidXhOZ0RlTU1OalMxanpqYTdUQUhHTUpraTk1eEMtYUpqSnZvc0ZmcFNxODl0UHRXUmNZaFhDNWludEFnNk5EWkhvNGg4TW13MHZJUGhYNXBSZGFaMXlmenQ3ZHlwZ0twZzVlVFdscExNQ1JJWEtHdlBnQ3BHWnJTc3A2eUY0eUoxODE2Rmg3MTBuSUtFa1lWajgwMjNjOHdCaGkwY0N4TDZRakFaWHJjaGxoRUV4ZXlWTjFPU0hzeHJyM0VQSUZ4NkV2UkdLejdGTUxUZXlCTm84VFdfOFN6YUJvdEVmT3NRTXE0RkFsTTVQa3RfQ3hxVWM5SlltUzQyYkxJQklGU3B4cDVVWE9Cck5PRkxkdFp2U0NhblU3YUR1clY4NFBtb1JZSUt3WnhvaDVsUmhkT2FDRXQ1dnNLcmNnaVVSdWhGckFZYXN1bDNySzNtb2U5Yi1pZzZJSjFrQ0g3Vlh5NHdnRzlfT2RzdDZTTGxtc0pPaVBBZTVfZ1JpWTZOMUh3WUVvdlM0d1ZJbGxJYUFLWVFHdkEzVGZ3Rm50RFlGVG83WmJrYXRjRUZWTlp6SEx0b2tpbGk5a3d6Wm1DN2xrMXlPM1pPdWx2SEQyX2ZSMVlOZWFiRDM2YjYzUERYV2UybWJWUG1HUGJac055TjlLYVBBb1l4WEE3V3NWdUt2OXNuTkR0c19ocEZjZVotVlkxSVg1SlAyYkFYWGJtZW1zYzZQZ2VQcXA1eE1paWktSENmTE9zYXUwNVpDU05pS3lmQzY5RVdHeDE4S0tMN3psWl9FTGttODNnSDBDLW1ObGJhemZod3g4YWtSOTE1OEIzY1BBUjhpT0xobl9MbmNpdXlwVWZ0ZTBRd2tGVjc1M2VxMDRhMjNVOHVmNlVaS0R2aWE4LS15dFluVEdfS3JMRUxmNjJzWm94MW5hazhCZm5KeTBydlZNc2N2aHRia2tSaWxUclduSnRFMEttV3J0TktOT19MMDhneVhLZDNXdmN6T1dDT1prQnA3RDlrSjhGbU8wb2R0MVp1VnZwX3B0QzYtQzA3cG1fUmVVTEZDOWtYSnhJZUxFbG9laUd1bzlCSTZHTGdMN1FpMkI4aWlnd2kwUzlFLWp6LWduQmt3Q0RyYzFtSjdZTWdFNnhPSl9Qb214X1h3VVBGXzFPWW1NdFhXX2JpYmlyZTdaQW5CckptSzNSbEUzU29ERUhoSV91NEtral9HbE1LeHZJd29iS3RnLXdCTFQ0WVhWaGlVbTUxU0ppMEhpVGxkVmxsdDQyUW4zSGw0aUtxaEZGdklEV2NDZWt3cWUyWjNfU3kzLUxoY1pMRkctLUhTbWJjUHdqMVUxakZfcGpVWUduUlRkZTJXV08tdld6RjVFdWVZeUQwNFc5UTY3ZUw3SnVSa3A5NjdoUVZnM0xRN0paVEdCbmcycGtyazlyUkhKM21NMFYybUY5REZQcnNCOHR5aTJfNWhhZC1HeklGd0k0OGllWG1WSXRDZldJcG9IcEZ4OUMtQy1fNDRXdFJwTDFKemFCUjdsZHk2b0M3T0RyamtDUWtVdEhWcENmb0FDNlNlWmN5SGlUMVNva1lxMlc5WVdVTC1IZ1hsbnhfQTVoWC1EQkFOUTU4SURrZ0NzSEZNTXVGQkg1M1VhQVlYY0JKbUh0VjhGbzlVUkhOS01SQS13LUgzNFFHSTF1SmFuQ2JHZ3dkMURyTU9nQ1dTRzVDWjBWdkdjanJUdW5DX1BjeFRBWDB5bXhVamlLVElUMWNnejZCeXA5UUYwdnIydXkxVnM0c1pYbThyVFpvRTVOck1qQ3V5V0pvckJWUlM3czZ6VGZBREdYTnVBNDh6ZFVXaXp0czJ6TmpfRmtjOFlvRHBBODFZZFpPWFVsSmIzZ2dqeEo0NDZ0cG5DN0VWWDJuMmotZnJTXzN6TDE5NUswZGVqSW5SOTZrLXFhWUc2UHI3OG1QZHNwNjZJcFlUWWpqZlBvNXhPdldQWTRhOExsT2hqMXJvZHZ2N19TdDRKOTdwRXpvZ0ZzTDhhamxZeTNOUzdBYy1MSlhBdFcxcTVhdjBQcXh5UEdOUmhMM3RVbHE1SlFZRE5oQjMwWVNCZFY4QUV3T1c2VHBpbmptWWdiY0lTV25vdTJla2UyVS03U2k0U1Y0U1dhVzJrSUVUSnYyX2hGcmw1UmFUeWxMcTI2V1dUQi1URjFfYklDNjVzVDRPX2dYTndvZjdsbzl6QkkyZ2F5d1plYWVTalNSSzkzeVh6cDBHNGVEXzNYYWVUNjRWeU5GNEdFM2FTNE5YcVhGV2FqeEVDS1h4dExkOUtMcmdnRGo3OWxwNUM0ZGVSdjBhak40SlhnTGNSVUw4QllGQWNYTDdkV3hFZkZmWkRwVERfVmZYTU9BTG5BRUVFNmV2RHlFenBMRUUwY29kNkVuRDlJODJJd2hFZGRqTU44cHdpbkg4TlMwNzhvbkNlc2FwV3MzS1pEc3V5LWJPZ2huczVtTzRTRS0zYzdzUkFsV3Y4VzJHZEhsUmNYdTdOUEVySmsxaE9uMWk3eWFiMmtraVZjTlhxNnRHZE1JNWg3WGQzYWJkQVhLcEQtZlFYeVYzT0IzSXBWQnJFMXM0elV5U042MGJpSTdhckc1WXpQX25iVnhFd1JxNjNYSVFDZE9YZ09rYm1DZEc3WlRXX2JjTkh4NUMzY2FkMVlwX2loMXU0MDZaWHVSRnBaS0Vmb29PRXV6bDJ6V2QtbWV3cVJhbS1ZOGdkZ0JLWjA5XzYtRm5KTXBqc3JzUFRGMUZENGRpUG91OGo1enVaXzVEc2VDS3REYUExM1Q5UG9WVk1vSngzTWphRnNtNEwwcWpUNHdzUGhOVEZvcFdINS1RdElONFAtQTBjamZ5dENOejNUUkpxWk1ENGtwQzJaYVVBVkxEOVBEU1YxUjh0X3JYNng0bkFIc2NzRTEtSUJnTUg5cTJuemNXdWx5MXdDaDZfNnp0MHRUUUszMW5IbWI3elVBZ1BBZkJuQkU4c0dEVDM4VndRRjh3ak1lZ1RBeGJHelcxSTI1c2tJaUVVMjUyOGNGOW0wZ19DUzlhNVNLRk12ZXBIY2Y4UmNDMEJsczlMTmF6ei1fNXZWYmltOVhaUE00ZVBPNFpQa1gwRjQ3UHdrWXRHMm00T1Z6Y0JNdnNFRkQwektObWEtTXZncGdWc3lOUG5jT190QTlIVF90R0pSMEd5TFpFQ2cwQi1RYUg3MDVKMi15T1RFV0UtT01DMERUekRTQUQxQnVORkpJZTBUcUxaOEZVTjJoaGJwTl9KWlVqeXd5eE5lVzVUR0xmVjZEazNuMnpJNUlyN1plUERlRnBOSEtrdjN1YlloT3Z6bW9XYUUwdmN6NElTZTh6RFpEZ0p2LXBsckt4bjJwOTQ4YndYbi1vNDdGdkNSZHBrWlMtbzIyWnhpVnpqa25kLUVVV2sxa3VKSm14dXNKbXlpU1FNQUlkR1pTQk44NjhfQmZZVHVlYy1rZllOeEFaOWFHVERVSU94anhMNm5qY1N0ZWstUUVKQjkzYVk0YU51ZGRBel9tT2JzdlEyOUF1NUNYVTFZRy1hYXNMTVRJTnhIWVdvVDdWS1RCZkFVeXRxZVZwUTRfekZGS19pSTNFTm1KVnVsZXRoZy15Xy1ORlFaMk9LM0hFUnRrZzRMMTZxS3JMZ0ZGOXVxR0VSSG1lRFlteVpNU1R6aGpheEdHRXJpWVVaOGM0VE5wMTBXS2t3SGhac2gzQVB0SEVxVGJwQTBENjRETzdua1h4QVRfTEhseVZGRjN5ODVjS3lHSEhQZldsNHRZdFpLQ0F4QTRkNWFLTXpoUmNZa1NwTmxEdXFyYlBlSGRKRmJrVGZLeC1GeXlDX1NmbklNTURIVG1YRTRtemVfVTQ0Q09CVTV0NVZ6U3RfYnpOVXdTTkVWb0hLc194dWhkMzY0b3pVLWoxMHRNNDh6bXdsUGlxMFJISzc2MUxhb0FzNXE3UXhQb3VOZ0ZSWF84RU04SVI0cVU0MUt1THdxNEVYR291bFcxVEhMNlZjeXAwa3VKd0lnRk5aQ2xzTFV5UmpyczM1OUJOc09taGxMTUpjLUlfcUJvRF9ndzFMcVBtWk9xeEFEM3FPVDR6X1JSUnRiZGxXUFpPdUNVNEZBcmYxdmpwdFJpVlBwRklGSDNLRURnd2NSYzRnbWN2V2NYMWR6Yl9Lb1JaaHo2MEpodnFsSENGMVVoQlVuR1c5SlNKa1dpa0xRa18yTUZJdU05TU5oZ2dvS1JIUlN0UEV6YllJWHZGZWxKdjF4UjdWZThUSHdJLU4wZ20yVjZ3ZS1fcDBzam15VHRLcGQwQ1g3Q2l0bmx0S3NHWHE3NDROdXFKbU96Q3QtZjI4Nlh1cjJIT3hzZTEwaG01akUyb3R3cXE2VDZrRmIwVEhPMVdVNzJpOFdIYi1TMzN6UFVXRGxfZzZCV0ZmTml0WW03YU9LSEstYklVWDU3UzVmODBGSDVQSGg5VGtnaVlWSWJyTWx2Vk5nTnF3Z1VnX0NKSnhUUkVtTUpGWDNHcGh3TmsyNzJGTU9nMVMyTHFIZS1RYmJUeEl6WjRiaElzMEtMR2JLR2ozWjdxeVJ4SGJHX3c0dzdiZXVBRzA0enNOTGFUTEhyTWhDOTNrSWFVZTRHc25WNTFpelllOFA2c2lOZTNwaTVDVVFNVElHdHhhMEhlZ2FzVk5pMnkyU0N6eWsyMVEyalliLWdST2tXaDQ5c29YWDZGbTJTaEVJaDNYSEpqY2tNOFRMb2F1ZU5nWWY5Tk1mcVRYX01uNUpMOXhwRlpvLUkwMGppRTRCOG9wT0E0bHhRMjlocHR6cEZVOWotTDBRbExrVWJZWHRGZVNrX1V1SmF5bFNmcUpHUkNjRmk3cDV4aVNtZDYxREdFYzZvcDEyMGtWcm1KRVRjekpGOEw3ZlN6dmd3STM2d2UteVlvUW5Qc0NtTFpTWVVZZGRJR0dyczFIXzJ3SU5uSlhJMS13cVMxUzFDTzFCamw0el8tT2hmNXAtTlNYUnYwME5FZE13ZE9yY2RhalhvWFJMN2x3eTIyeGptWHBnaEFlS1BxNHdYZmYxQ0t2cWhrakFjdGNyYmJWLWI4U0V1bnRVTUVPM2wxUThxQi10SF80MzA2WE9EMkJrRklBNXZfcW1scEFmR18wQ0VsYm52V0loOWUxc2ltaG11am0wSG9BZWNKUDV3NmlmY0JpRGJaVlJFdVAzUnRfR2x5MVo5UVpWZHJUZ1UwNzdDU0JJWW5FWWs3ZlBHSVphMnNXQUl1bW9GaXFPaEc0cnFxNEhRZWhEdTZtZ2ZWLWF0cG1HVEhHVHY2OW5aUzRLUEw5dkJmSVY4aEF0UjQ2a05aMy1GTnY4LWZnQ2dmdHFnbmRsSTBsZC0wN29FdFR4YUlXbjVWMkJpbk5WUklEVnlZT2hVT2RsZjl2X3dGWjFjcGZpd3RrdU1wVEREWjA1S2FHS01mbFV0TElwNjhmblphZ3NXNGl3dzlJTGZpM1ZTU3JZTzVpVVBmOHlMaldhb1A5RXlnWDdCWmVSdWFqSDdhYV8xRVJPUUY5TWJFRm1TaFJYRHpRa1hsNFFSQ182ejQtOV9mQ2REQXcxWWZhaHhreVpCaHBwYnliSzFDVHBLQnc3QkQxcEhvanNKU01PTHJwZU5IWWpXRk1HU0xxOUlQR0hhSXVEZXYxYVlZakRqM3ptNV9VVmlrU0tsaWlQbDFyMWhQUFhXMXY0X01CcU9Wdl9PWVBydVZDSEpFeTFGbnNEWnJtd2NGVWlxSlpJaTd5S181NDFXemZab0ZhMjBxaHhod0tYd0c2TkpENE1zZHJ2VjJ3Y3lkcDVsOVF4anQxMFRCNlFzNFo3QWtZbkNFRnVEYWlyem1zYVpidGlxODVINnBGS2U0Y29GRlRpWFFkWXFmTGFaMDYyR21RbThtU0VGa25TcU0wRk4wbEwtanFiRFZKUThsWWVZN3B3OGZmUm9LaEJBbVRIaDFvSzZPSFN4eUE3UGVZcF9WWjRUT3Q2ZFlUbTUzNmV6TV9GNEVzTk1XUDNLNkhHMGpkX3p1WEVtQ1FkM1c3MEh2UzRhZjVnWGtNdHREd3BySGZOODlNeWN3UlJxN1pIc3dZbXBSa3FlMEFfTkVlWHQ4QzBHenFLTEZwV3Q1RkNqdWxYVnJzNWdycjNRUXZ4UGg5UkRhYVFrVTJRalpLeG9aTHNqVS1yUkV4SW52SlMwUHFNYUhUY1UtZmw2a2R3MFBnbFdiWVFtQWtPa2Vqb2lSVnBJTy1CcUozeDFhaGpjdGY0aWdxQVpsdDFMQkFRZ25QbWRCWktZZ096U0tGdzhUN19oeUNSb0M1UktkRnVkSkItWG1DNEh0LS1oSDZYZ2JLQXRJTTNHQlNDcU1VdGUyZm4zTGZXNk5SS2U4d3lrbW1LdnRnbUZxTnM1Q29zcGktZGVYNkFyelRMQUpubTdBUVlRQzZOcmkzQ0xjckZXeElhV0huUmtNLS03MnpOeVV1YVJ6RG13bkNTOFEzSW81SkdjOVRZNUhZaE5Ca1A4WGU4QS15Y2VraEZNc2FvVHo2R1BQZFpudWFxMFA0NkVTdURibTZndjNJVDluZFB4X1RPMkx0aEQyY2Fva09zUEo0NTM5QUxRUFFfSG92VTFPcjMzNkNXY0Rvd3M0a3lqRDNJMHJFTERuRGNMZVdNcG84eUdjVm10U2VoYmw5Q0dlR2J2dGhfeWNNMmR3RVFtQ3RXUjRHam5wekU5MUZveWhWT2ZxdDE2MjVvbVhEaTZ5TXE4U01tWDAzM1Z1YnVubFZQVDJoZDBEdHJLanZkazRpSUUwVEpraURQN2l0OGhKREpvU3ZPLW5YeDkwVTJjNkZMLXNnOExQRnNhdmxnNXp2Rzg0T0Rvb0V0OHc5M0l0MFU1VG1KNUhZZUM4dWE0UVJYSWZ4WmxkelB3alNpb1U2Vl9vdnFzYWFBaU9OTTQ5M2M0aWV2V1ZDSHhtTVZGVTJOV1ZIU1dCRDZSYnpKSXQ3TG1pdUszN0g4QktuSE9SckJrMk1aNmtqU3ZRWXE1aDRNQV9aSWFReHZHZVNZb3dJdFRMNml3REdhSDM4UlZuSE9XZ3FrRUYxUUpTSVJuSnFmV28yaXVTTmVlU25qMzZsdUFDTzJyNlBpOXpYeE1EcmlYS0k2bWREOHhKVjRqWFJDNkVSQnE5SnJSNGs3OV9zQzg0WU9WN0VObngwMTlHaExfWTF1Z0VINHBZamZTT2pQWTFVZThLUTc2VFVjaUVuZjFnekdqMHRIWjJIcXd4RGltWGRLTE9vYzJsbDlQVVM5SmZLQjVwZjllMlFmOGVncWZ2QkcwZmFfM3hyT3pnaTJfT3RzUk91eEdtcVl4RFM3QnNOalRWLXNIUGYwcXliZ0NvQTFoWEVJUWlUNndkTm4zVkpocnlVVFBfTUJiV3RNcWtFbEJmMWF3bnpNOUVMNGJ4LWNUVUI2TTdmLXY2M1MwZEVLejVGeWFlTVd3cU5keXEwZkJsZkVRSHdqVk95MGhvWnNfaWJDdWh4RE5XZm5GVWhyTU1sUW5mVktOOWhCRXRNdFZWU2pUeGdZSnRyNzVKdHU5bHRqNFpRSXJrWDBhN0c0Z1ktS2xtbWRBV2d3TTctRmpDRmxIdDRoQWFLX3ZPTndKYl9EX0U3QXNTYVpaT1lPTGRXeVF3V3pnQXJ5WUVhaFZHdVI2NHhHelZqeW9VdUthRHdZaVlkbVF5M0N3YlJZUnZxUzFWWWRiT2pkQkM5QndpeHZ5dVZycmgySkNhTHVhaTlOR09Ib3ZQcWVYbXg5WHpoaV9iVjFQMnhHQjNYWmZJQ29sclBXUzkwOFdWUmwzanJ0cGZqQXo3bVVSSEdhR0EyV3V5M0VvSnBLRnhaNzRXMHA0djBWZU9xRVVyOVcxT1BKUkY3YlRfaXRBR1VBZWllNzdzMGFfekluOU9VcHZEc1VQSmIzd2NDa0s4dkhqOVIzMTdNY29nRWl0OWFYREQ5elZvMGRUZ1V2ZGpOSHN0NElDd3NFU3JSemNIMHFKeHlnVGc1TUMtZnFOUmhYS0ZFLThJbnZGdk9WRzB5akkxV3pXWkE1NUR6S1RPTWZZNTFveFFVUjdzdnF0SUVhODlSOEpRaExIU2VPV3J3MDhTTlo3R3JOMGR6WXhjbGtnY19VNGlEakx5dE1KcWN5V1FOOGhTVzRORVY1QTR1ZDdkVndJMHNLN01Tc0NVSXhzSUFvOGIyTlpPNlFoYTRSU2NHZFFhZUVvYXZST1dTQWhRNURNLVN6MmxYUXFtY1NjQkNVRGwtQU9RZE1ZUG84ejNZMWpQa3pjRTJMZTQxRzZ2S2lOTmxKS3dLZ1d0YkdLWHRTSmZ4eUJxTS1oZERqc2NVQkxvc3p0OTd3aXR3em90N1dEMS01VjBCaTlXZV90Y042TmNacmVxQnJuOGhHYjZTWTlvX0xfOEk0M0lhWjBMUEM3WWJSaGUyMnJWLUhJd2NPZmV0d29vYThMaGlSaW42RWExd0xLTXZab3lIMHhyZGM4S1V5bUdnUWwzNjltWXhJa3lGeGQyT2U1Q3JOa2lvTmNNekdhTWdpNnhoWGRMZ1VoTjIzNHo3OVN6ZVhucnBJOFpMSnlqUHRiYnhfRWc3OHNVMEdiVHBlMnhhRWZGTU96VkNIS2NNcERLUzdKZ184czFSeHNlWlJBcGhGR3RIXzc2UmdEV04xYUlLYVJyRDNNVG4xQWY4NVdLMDhxbVh6RnI5V252S19iNEQ1ajJMUXBVelYxMHRsUDFDOXEyTU85RzE2VDlXVkNYWVFrUVdSdjRwMERmb3dsWk1nOGFua2x0ckUxWU5KRDBBM2hFek1Rd0hOc1hwTFY4YkVoamVpZXd2cG9JcmZJdGUyTTRPQ25MR3lmTTV2a3VMdlN1RGlIVzZpWnl0SkZXZ2swU0czNmR3Rm9nSFBkNW1UbGFfX1A5ZEJGLWkyMFd0MzR6NlU1X3BfNUxFZHJKVHlQSzMza0piTk1DVkxldkc3UjN6U01lNTVpcDVWLXpqVGx5UmJHUzRsN2tqVTNJT2pVWVBjYnNEMmt5S2d4ZGg1b1dsQ3NJSzB2NFk1dVpQWThRR05pT2lnZFIzaDJJTF9iemctS2RrbWhMNHBQVU5xMjZwaGZCUHl0YzFUd2ZaR05wXzdZVTRtQXFMSE1yMXRhWVRRdmhmUDBMUmJaRm5ieGVhUjNSQ05hNm1jYmNyQk9JcmFCcmNJZTBPSmdjX2xHclBGS194bDVTaklhUlZtY0N3M0J6cExBNEFGVzVKNzYwcExTMEpoZU1naVFkc2w2NjJKNFlyWW9vNGkyLXFtS3ZtdXhoRDZRRjVBV2YyelZRNHVwMWxiN1RfdE5vR0pBU3FicWp2UmV3OHlfUGdpaG9mR08tYk9Gekh4c1RFcmJNbWdFY2Y1VXJ1STBIUElVcmFOMkU3Q0J3elBoV3MzQkpvRnZyQm1TeFBrSnAzLVBIaGtNbU1ldkZqTS1EdUsyd3dnc2tsTm5saXB5czUxVmFVRjBTa2RhdjA1ZmZJdnlYenlXOHIyY0tleDk3Nl9XWm9SN0I2TnJsNnJMUERyQWJTZGpLelBRMGxCVGViSmsyVE50OV9rMnA2bS1wX1pQVWNwRXBzMS12VjF3TkhudThxRXZoOVpoeF83QnpSR3ozY010ZXpNMXpiREJZNlNURHpLRVJsdWlxQ3RLYU1JVW5fWkhJNDh0Q3BlZm9UYnZQVHFHQlRBZDJOWDFYMEt2LUx2RzdqTTdIQ1MySUFqRDM5SzZGcnViSEc1RXJSLWV0UHROVGN2VXdBc3cwcVZrTkd3QXliVFpiREQ2Z0dHVVE3dDRlTUZNLVV4djBfaWc2dnNlWXc0Ti1XQlgwSmxpNWZRamZYM2JWeE1TRXpiby1fSDZuTU9SbnN4eE16eGhLR2JnbGtCbS1abVFIcHlVQkRxSnhvMzNnelFVYkRmQ0RnUW51V3hxTTNvQTRGd0tiX3RuM2VTYUwwUFpISXZWcVhSQXZ4aGEwdkJ0RzU0cHRfZ21keG1rN1JwR1BqVGtsMzRXUllfZmFyaEN2LWpIdWZiX3dPRXE4c29UWVBmR2V3NUJYM3hHRExSSlhNVDJ4UUtCR2J4eVB5TkEzVElHdlZNbzUxdU92a0hmeUtWTk13ckJXTmFwb2JYLUZVUF9VY2d3QThIWWFIVlpVVU1jSGFBbFZEU1pqTUxLRTJfRDFWX2dWUnU1OXNyS2VNVnREQ2VYNGNOWnRuNkFpejNHMHRSQ0FEdTEyZ2R4c3dqZ1BqdUVzU0FlTmtTLTFiMXkzVVI0VHNLOU01MVluYU5SX2xlcHA5ZVo1U1ViMFNPcEc4cDhVeWVmQTFHTV8xUm1BUDU2MVcxUW0xU2ZXaHhlMEM0elQwVU9mWjAyQ2UwMkVWcF80aE81bzVlMzJPc0trMFppRnhIUXhwZzIyOS1HSFluakZJcE4xdEFfS0hjRVMwSTd5NEZmbWpsSU5scnRISHJ1Z1JhdllBdklSTGJGa1ZRRW1hbVhodHVDX2VmeFdNUU5YeXc1SzA3SkNhLVpuOUl3NTY1czM1Tm1pUnE2TDZNT1Y2R195blpDZmptd0cxdkdNeElBcUtyeTBPdTlKbm5CWEc0SzdZb2h1YUN2VDVOOUtwMzVkX0VyN1JtaWJ6VXFNMzQ2YW1xX2dJQjVQWDhGbTI2NExfTm1NS2dGRjZyWFpLTjU2UE8yQnlrTWxJTlhEZFRhZVA3UUwxaVJQWXFmUnFfUG9BbjVsajY0dmRBM1owbnVzLW05Q09FVlhoQk5VMkY0TENveEI2QUpQZlA0UGlycFNNT216U1ZObWNMaG5ubUExbHItUG5nc2k2MjY4Z3R3SGVSZzFfZ194MVpnRFVydzk0T2pVckpFanF6UHd2UDhsOUtYdE8wY0ozSXA0SlFNejlSc3hLeWNEVTI2eFV2bHVEblhaSHhGYy00YnFnejJyYzlmSnRKZ2J3MEE0Nm5QZmhzRWgxRTJBVW5zSHh3b012M29JaTd1eUlEb1JBUWZsbnNFRUZEc0cxSWtZdHV4dHBBWGN0dndsQVZ5ZTBFN1JNSDh1cmp0Sk1zVUwyT08xWXYyVWo0NVFwTnFQNkdpc1dLdTFaNGhaQUJnc1JZdVJPZzZMUFpZSmlreHpSTUZxYzRDMzRKc1BnM2JxN21NUkNhNEtsMVlrOGEwNEVhaTNlU2F1QUJOcldGY2hKQWluV2ZnS2w1N1FOVjIwRUNETGFtbVJBdEp0MF9LenFsLTVxb0F6N21kVGdUNjdVaXU4NHM5bFc4ZmVra24wWllUNjI4V1AzWjNXeFZ2bUFKdlpTQzBxR1A5dFlVVnNQLTdWbzZKRDExS1pKay1ieWM5YUdRZW1nWUFjNzk4UEwtREVsMjRiSmVIWXEzZGF2blVXdXpkM0tKdy1rZXJneG9KS1VMekJ4bEw5QUprSFprWHJpb0xkeEFvaHpNNm1OMmRxa3VuZHVvUm9xcGwzcldtOV95RWh2MFJlNjJjUllFemlVU3Q2U0w4R2RtdTdpMm5mU2VhdG1XWnliMHUxSl9WQkQtX3NqR0lKME9lTE5XUGJzODNRMTZjdkRWeVJOMG1ZSjM0eXJCV1BOajZZWjdoQmZEVzZEcTdWZnFMaFZkSFZjUW1qVndydGhrLWhjTFFCR1dLZTQ1RE44RWZ2MmR3Wk94UHc5bkhaQ3V4ZnNqQ09qM2x4X2dReFVwbDgzZk9HZjFvUHRueXR3WkNIeThrQmFNbVRRZ3B1NGVvYnF1NUZQczhQRUJuTUtMZ0dVOVBmdmVzeWFYVl8ydW0xb0xiVG9NeTA4Ml9BZ21NNlJKQ05NV0ZOSjl6c095Mk5zLWMwcGx0NlVTSjVzQTNnLTFmamRMNkN3bS1ZUXlyb0tMUlU1MlZvNXVHTFk1N1l3M2ZmU1pzOTRNTTlMUFEtOUREd3NOeE4zd1JLU1NudnV6anBzbnhHZ2FqOEZ6S29TenMteWpETWM4MjJYcW1DQXdKTmJRc1JKT0dadnNDdVBEU1dpUk5VWHdXM2FYQUpfWnNtMy1RQVVMTzV3UmZZS0lzRXJ4OTdEdldFWnZyN1ozUkp4VEZKOTFWbU1aUEFoT3U2cjhuMktxai1OMnBVeWRKVTQ0OW51N0FHeVRwNHUwWWt6UDZNZzJLYmhSUDZWZlZuODlYc2U0bENPMk10ME9kT1d6MkcxeTBJRzdaZmpGa0FTVjkydUNzcmFfTTQ4dGV3Z1dQeVJVVThkc0JMczVoMERKbVcxbTMybnp6dXphMHVyVXpSRUNSWDFBVmtubmVHOF9pd0dXcHREVU1IS2ZidHhJYkNfc2JUNUtUaDlhdjc5c0psN1FBUDBmX3RnYWNsZ0NJV3M2bnl0SHZwU0ZaLXFBSGtTR0tRN0xrMURZMnlLSkFzam83elVsRHZXUHBoc1RHU1NMTjFzRVRGNHNiN2FIMkpoa0JiTnVGNkZNR2puelduQ0pRZ3RtRjk4SHhPTS1vS2FJT2pvVG5NSU5lOWIxNlNqU21QYVc0SWZFeTkyekRiajdVT0JyX0FqdlhHYTRacWhKR1hiaVk0ZnJuUEoyVGdTUGxWUWV5VU00ZWpTV21IRGQtaHlQQ21xbnpjRE1yNHpBQmZnN2g0X29wb1N3d3NOUmV1bDlNZzFjd2V6VERBQTNYZGMtQ21NOUtPVnZ6ZV82QXpWMlR6TjNmWlBCQnFPckZDekZMUkJiZVk2SlpmRnZzWWw0T21Vci1ueWVzeWlhRW85X01aTENULXh0bmhRcTdYSXpBZTUwN1RzSHI0dmtyTVBYUER5Q3dCVndUWThSOGtraEt6aUhTeVNpOGZQa1hTY0lHYVhRdm1pVE1TYXNCbE5oelpRX3hnRm1XMzFZeXRqeEhzOG1UR1oyTlVrUjFRRFQyTl9mR015MGFTVlpvbHRmUUFWZHNicVJvR2ZrSHBzSllKbElSeDJ4c0xvbTR6Z1hGTDFXNWxOSWVyVm5Ub182NVdkbzk0eXloUnhPcENpdjBIcERFY2ZCaDNkMW9qcExXQktRS1FuMDhlc2hxaTJOdTNsV0FoQ1N0MlJwOTFLejBaWUNGcnloQUVUWENKTzJkTUp6cy1TMkpzVFFhejhCcThvYnlDS3Exam9pZWJPLThPTVRrRGFnQkZ1TW9ZQ3FIZll0R2RIMm9JVUhWdWRYMDVKcGRiV1FwNzBPTzhLVW5FOU5LNVliOW5oWmxlNFc1MmNVbnN4MHpyNFVELWZJLUNMVE1LOWhCaENMRmlkR09tWmRCQ2daN2ZEbnlEdDF3RGZrZ1psb085cmpDWGtCQm1FUk05aXRqZ2JkV1dCY2RNVFRmNTBEMzNHeTZNaTJ5VEpLdWtsUGk4cVlETi1OUXAyd2gtM2J6RVpfU0hYbUF6Q3hVekF5MUFPTXZ4Tl9NQkY1LXNJejFlWDBDdUJRSXB2T1JEbW5vcnBQbktzM2VGTWJMeGdkVzFvNDg2dE4xV01xeHEwY3JGd0JjQmpHZUs2VUlVcnhFbzdaajBJQ0ZQd202SW1lRkhvTVR5aUlGYXRNU044YUZqRDF6RlNNa09KY2FfX0FOVkdoNThBWVNFUzJzRnNPQmsyS2ppLUY1Zm80NEdHQXllNG9BSDNpTWE2alh6WjQ2UU51cUpSdUVMeGp3WlQwSVFEcUplNm5sNFdGYmZvVjItckVLZ3hBMzJPOHhHRFZNRFRycWpkVDBIbFVLVGJMdjVCVDdBWXpULWRDcEo2dE9VSnllZHNlQi1xY25YcDlQRVJ6T3lLcjhRcWRBWEU4bmY3VnRLRVpaRnNJY1BPelBrcWtBSE92TVFsdzRVYUl5T25YSk5rSFg3dkgzc2JCOGxrSkd1MTAza2RWZE1fMGEzWnp6TUZ1NjRrb2VidnhtNE50SFhEYTV1X3BEUDFJa3RQLUN0Q3gyclBiczh5RWZkN0szNFpGOTlrSWx3TGlRblJWZkRPQlYzU3ZDb2tWNFdhUlRXSEs1cERXdEtkR2FlbHltbmlqc1Q3SGhmYl9YTVpWMVlXZWN5eWdscmQtenJ3M0xPWG9NYkhzd0RvcF9NNks4Rkxxbjg3M3pfbUFhcFpzZmpyTFJPQkhIMUFQRTRmT2E2d2pDVEE3c3NzRzZnMk9zUnY0bXJJQlFIRzhrTU5Db3ZpQ2hHZHpHU0F2b3dqY3BTb3lWaVRWZXU3WWx0eEIwMFYxMnhiUC1VQmJuN0NidHdQSWV4dWxiMFhwWFBfdmQzRG5qTktXcmlWUFZoTWxnbXozU1BLNDZMU1A1YzV4Z050TTI4X0ctU0FnRmVpNjJNLS1JQlNLTUFGOGctSzdqRWZ4T3Y3aWY2eEU4a3hULUtkZHJfTHh5bFo0dVItM19zZm9tUWFQM1A3Nkt3cHdNLVFPN3JXbzJWaWpVcjlPZzFVT2g2VVE4QWJ5bmFJNEFOTTdsSkIzeHctX3BOMThRYl9PTU0zTnNjZDZ3YWk1blpYZWxiRWdEaGpWMER4SlQzeGRtRERGVGd4RzQ1R0F2Wmh1TFhOaHhhbGFRb1l4NlBaYzlZUklTZmtmamZOV184aTJsc29uelVxZWJ5eUc1dF9fS2U2c3FBMHlwYk1nbE9KQTNGdkRPVmxOS1pzSklQT2h4TGpGWkNyMExCX3hvZ1FuWElVTEEwVlB2WkN4aTdRVWlMN2xqVklpSjY3VDdDcVBOdldLeGdzaTVjNkh0d1dnbnJmc0JsUXl1V25hRzZITURoZnpPc1gyZnNrR3FXdXBHZ044b1J3cVZPQnFnMXJrUlNjdWNqRWpLQ1lITVZrQTExS3dfRW8ybGFYdkdqTEw4YUJKRVk0Rnh3Y0MxUkU0U29sMG1CdGRXdVVLQk1XaTUwbUJaNU14OGdpMUk0VVpfZ1BTb3N5T0d3eGpLMXBwUF9ma0RVUWl2M2Frb2lmMi0xNnpoSS16Z2kyYzJjN0pkakNtVlJ4d3dOQUFoX3gyUjZXV29kUHUyeHZHQ2ZMNHNKQmtkZ0MzaGlRUmVKWnNXbFVmVThJNG5XRkU2MlBTLWVXMDJVRVBCYnFwVjFVN3VJUDJtVmdpTXY3Q2MzcjZDZ0kwX1lmZDY1Z2c2ZmtwWXpneVlkeHVCcU9PUGNVU3BaRVc3UmNBZ1dVbEF3R2hQV0o1dlYwYkRaSmx0YVc5eDdnekFnVG1rRjBjc2xQUGMzUFlUU1JMYlJHQy1WdHFwVTRWbGdxMkNFTkZpRVZrQ0FLanBnMUhPVVBwc0ozTWFra0tCTkRkT0pQWWlrakFQQjRzSzBEclVMdzU4b1pkZnhEZlhKNGJZa2ZEYWZyNjd4QU9sTjN2OG1OMkhiaVl5bkpySVNaUGlEcTc4akg4X2F1TDl5VHRNcVd5aTYtWTFJQmVyZHQxZXM2eUhldG5hakFpRlE3eUpEd0VvTHRseDlVRGJQdVd1RkVIOXBtWWo0Y2FPWUQxQ0F5ekNoeUtNV05NRjRQWHU3QVNzcHZJVXlsZDZOOXJfaVFEYnVVb3A0eC1Gc2E1Y3Q2VkJzcDRXMFBKTWlOMXZtWTZQZVNRd0xYVFlxbUNJMXRib2xFUWx1X2FTM1FlVUFtTGptX1FPVjhwaTBwSGtZLU5Nc3FqM1Y3M0hMOWZ2UDJTaTVJQ0NlT2VheXY3bV95TG00a0tVejRkZnRHY2JacGFscG9VSmxNTW8zY2JQOTZtdUpJaFY2TnlpLW5ZSG44YUFnRWJiQ2pKNGVqNlE0cE9fQWZLUFFWc3AzRmlEWk12TXY5RmEtWEYzTzJOeXI4eS1GWkdYaktHcnl2U2lDVEYySFVpSDZUQ1NYeGF0Ri1DUmxHU3lseldCdU5KUFBfQ211eUVRLVFuQTRFU3JxRFVjMEM1WmFldWY4cFlWYWQyN3dKNWNwQ2JXSnFDZmZJOHlmUC1zNnFvVFhsd0o1REdsUjlTWmdTTDlQOGJnbm04SHQ3enpMOXE3LTQ2eDhXWW1XTGhfalhIMzRnYl9uTkVBWEVneEVoeGw1Ukk0MHVTT3JwaUlsTGVqY1FjM3g0RFlRQUhSRExQOG94dEhKWUt0WWZCaXdRT212SlI3SklFNmhpVVBacU9jc3hyaXNXd3NoUl92N092SEcxVXRieHZSTjNQUThaZHk3NTI2TVZvLS1vSjZRYjNMUlhrLXpGczE5bzVMOEhlMDJEa090UUJQYVFaMUNweW1QT200YnYwWnYxUFhJVmg4TGpuenpXWmt5YUFqbUNtWUxXXzNnUXY2SnpGSTZKSDA4Q1JTVGZCU0I4cW8ySG16TmNQUm5JcGUzeVRVX1dHQ3pQOGJ3ZF9Db3RjS25yWnpST3hrOXctRHVqUW8xZlM2UmxuVEJuQ2laMVFFLVZoOHc3TGtkTkprRzU4enRxNjM3YjRSTXBETWJ1M1RhTzVpS0FuaVA2Wk5fNExsc3ZIcmJ5RndTajU3dzFIWjZNbTRDRXYwZG1oRFJmVmd5dktWcVlsX2hWWU4zQmFTOFdpWks1bEduZmo5QmZHX3E5LWt5SVVTN0tHTFpoV0pXQ1kzaDd3ZTV1ZjJGOERhR2tsZERLTTR2UWFCNmhmcGoxbVNfQVNrcF92dURsdmlybGs4NmNQby1ZUVdDYnpPOWs1Z0tOaW55X0h3YWRDU3JldUdQcmdIT2NBTEpNd21BUFYxZmg0ZTNXRHFfSWJEX2o2Y1g0SzhGSXFuRzVUMGxucXdXY3hhREpENGdZWlk5Qzc3LXZzSGtMZjNscGV4S0c4S2JmTTlFUDRZaXE4M1VuU0g5MDItLWVWcldBVW1NUGtKQXdtOHF5cFFFaWM3MEU0cU1nbmtER19yNFF3UHVzWEtoZTJtUDlvX2ZiWXRZbmxLUjZFUGlKUGp0cTF4MGZDM2l1U3V5QWs1N0hCazgwOFVLQUNhOFhpNVVEQjBMZnFNZ2M0SHg1eVNJY1Y4dnRRYjBoU3RXRGc4TnVOWFFSOExoalIzZVlhenBCci05MU1hN1Y3SWQzcUg0Mm1DSDBFdURVVDdHMFpHb1Y2Y0VUY1JOODIzYXBhMFFqbWdRMmlQYWdqaFdIV2JZbEV5YzhaN1BNRDJlS2xvWUtrRkpnVmwwOTdBdnI4azRIeDl0aURGOFh5bXFqcVZVa0VzQXRRTDJIc2R5ZElwaDh3Z1l3RlY2N3ZaZEpUYzBrdHdMNWMycEZIbl9OcVRaOFFzaWs5ZXdIZWg2SXRoOTU5a1NvWGx1MTRKeTRqM1Nla2NyQUFQbk9YWEUtTW1oNU1uUUh5anhqQUtYdm9aZHd2NzZTSk80eUhfVXh2a21QTWpFa0lUWVFCaUY5eGk0Nkpoc0VINTJtNVJUc2lRU3hsc2FmQ2ZyYWM0cnhwUWFoSHAwWi1xdnp1b2t1WnNMLWFvUFlCYWNKcEZIWjBGN3dETE0zQUc5azU3bEx3WllwTGJZRmJlenhuZGtHSmN1eFdtSnQwNk5Gd0ZpWkJCcXFxTEpSTFZXZWtTeWhNbU5oczlGZm00S3FyWEdJNl95ZUpZYjBHUm9HQ2JqQzlmb3FvV25vYlFDMVBTUUg3d04tZ2pJZUJRbVBuNDFsX2M5WEZXTEhHUzdBRmVYeklYMFRnb0FkZ0w4QWlEdWROTUJwOEVJMXRPa1J1RWc4dTA5WE9wSE5IZ3FydE12WU82NnpyMjNrNWw5a2VEVk1KRVFnSU5PcUk5R3I2M001UWFaQmdmZERQX25JakhaY3BRLUg4SGZ3c3I1S2pQRzl3eGE0QzczNTA3RmlJclZSaFpqeUR0Sk91clNJT2xUMUxQYjBHMWdldmVZcHl4V3hRaEZRWjRRbkxrMUpmRmt1Ykt5czJ0NXhBekgzM0IzRmxTdzEtU3NiYmZiQzc3M29lYThMTlRFcWl3dWtvZWttVm1TNTB6RVk2ZHNwaHI1SGxzZnAxMW83RURoTG9IS0ZKQnlVYkRNMDRRRWoyNjIyTXpibUxkYmRHYjdqY2FmTEtKOVhlZHVMWG42UnpJdUFZQnZPbHo2ZHMwaEJiNHRrYjA0OVRwS3N1c3pOUDJHcmIxOENXYzRZV2VSNmlZQVBVQlRFdzBGWWs2NWNTMVNMbnRCdDN6VVM1Q3cwR2NILWhsWlZyZ1ZVN3pvZW1fRzgxeVRNUnNTZ0VtZS1YenlyanF2ZllFOUROZmFSYWFzeElOZDVFX2hSQ201RDdFc2xfY3l1dTEtNVF6VTBGOEk4WE5aZkV0TWV6YmFaZ1JEU19OOEh0d2pjY1RCSldBX2NuZldvOFhGTFBVNFpscjg1WmI4aVVLVHZoc1o2XzQ3WUNZRzFCdUxCcG1hYnZDOFQwUG15VEQtaUhKX1VHUFkxWTRDbmZIMkdMZzAxeXBLTnluci1lOGJFYU9rZ3BHSkJILUdqekpKSlpYLUdHbVVBZU5ocWxneGtrSEh2OWExak1hQ3R5NFlyT2RpMjI0YU5NMEs2WElseGhVR25GUFZJbnZwRHk0YW9aR3BrekhFQTl0NTB2ZVRvajJ2WFg2LTFVbExxblZhVVM4UFU3bTRoSXoxeTNYcUJVMG5Ta3ppcllBZGwxcnpDb1hmZEVoVVlVU0lxOEJ0TndadDY5OWhlRW04ZC14ZlgxQjhNTExqdDNvdjVBcG1HUEpjNTJqNmJFalFSWDZ1eEhzakMxeUp1Q0VmUTRFR1gzeGY1S0JIN3dnd1RqR0gzMTRJZnkwR1JDdi1TRTFBV1JJWFYtOVpRY0Y1ZUtRWTg3ZllyZkwyZ3lZMkI1UWgxX1JqMzlmNjJGTkFCMUE0X1FGQkotYkFvM21tMnBjbHVVRXdFV2dsemJHaUZFSDEzamt6am5rSHRjLTUxRUVVd3RBOGI0UF9BcUEtNGhZdU9jZkhWOHZIZ0MyNDRBY05sV29qTk5PbDQzVHgxWWRvQ04tRjBYUlBkN1dEVlhSTWVnOWhPVTFLLS0zbFlPeDhDeHR3cThrd0VjeFBOTlFoNjA4ZEQwYk1KaldDN0pFTXdyS1BQd1lNYTN1Q2hHQkdXRGdnSDVZQ2FMbTBrRUxGeHFtUlU3SV9YN2R3S2RxeEdUNktEZzROT0RCcy02cUtodGlweC1DbndTZVN1eWluaTZNeVNzdXBlTS1mY3dwQzQ1bmE4ajJnZ3R3WmYxTnZ3c016M2tyR09zNmRDR25NYU1LUnpxN2tsUnpDSlg0S05NTkNGd0FHc0J6RURMa0hJeGVBekhMTTV1ai1nRUJTTGJZT05IWng2THlQZUloVF9mWjZoRVFhcUplei1HX3NsQnhlTkNUVnlmQi0ybXhVZkV2V3BTVkZtYV9jelhTV2tjNXpvcWo4aFNzazQ0SXpMUUlDb1paNzZKZnQ2dW1YQ21jSDVWdHBWck9ZX0NUMnhiN2FLOGtIVTJZVFBMUWxYaUs0ZXdjNlVHcnFLR3lMUDQ2VjdreHU1SWlhU21jbmMtaEZtWXJMZVpieGVzelZ0bWZ0aGZjbTFyeWFWcUlwckhoTUd2dU5KaldMbTNoYmRDQ2Y0S08tNHhIeXNFbVMtLWVJZnpXbDNQRHBjVGRuSjFHT1kwenQ5Z1U0S1lSVzZlS2hQZjVYX2hnNXdFbW1ZbXgyb01NckpQVUg0T2tMYlY2MEdNckxlX3Q0ek4xcTZxSjJvSi05ZmdDR2dlN1hXbWpPWXhwSW5jcmRQNndxYXNEaG1aRFV5WjJ1Mnp6QnZ5T1JoR3BTd1g1eVlheTAwTEtkV1R5Z0NRZUxYZzNzODNqd0d2d1cxRHg5ZjViRms3c2ZYMHVRNmlQSkRWbzRLS2hhYkh1RGN4X2xfWHc5cEY3N2o2VmcteXBCZTdGUTdpcHNOS2NQV0haYURGTXZiamhaUU5QZUJrNFNma3B3b2UzbHVZN2xpZ3dqS0FyQWZpWWtDd0J2WHdOUW9lbnNQZy1PWUJZU3dBWXZPLVk1QkFYdXFLSHhjTW1TQzZHSGJ4SjNScllfVUpBNTBibDdobk1BdkluSWNnS2VIRVdSZlR1eGFJdHo1RUctTnZneTNEVExfWHZsUTJNYk93QThueVEtTllLcU5Ldks4RjZ0WUlaNUhWbmRNUldsTDYteDJwcEQxcUJNSTE2ZDZfdU1qSVhSZHlDc2xrRVlhbmRMY2pjWVFiSmhqejNPVW8xWDl1VzJVMHgyVGhWajFKNUR5U0xETEhaR2ZEMkd5UWJpX0tpbnB5UmtVdmlvNTdQQjBjWXVJbFloSWZVcm5uX213WWVWS2ZfWDQ0T0loajNxR2V4R19JeVB5QW15S3NsYkwxZkgyTTFQb3UyZ3lrRXJXSkZzZlpQYy0wRm5lcGp5VG1LcjlmdG1GV2VfWkl3empzQnRLdDBhNERidHAwUm1xNlZUZDN5NEZ4RF9DUlhObm9RWTBBRzdUVDRkajJvRl9WZmthT195Qkpmc29OTnVLT2huVTBCRFd3ZllSM1RNdjgzWm5rdUJvNHpGVWJEUFktQW5pLTdRR0hDZjh3aTdHTHNWR2hLYXloRXlQWkNjUUY1ai13ZjNpVzlDekdOMzZQZ3h3NDJmRExBcHByc211TUl0TGtNNy1NWTRpRUtQLUJuUWdRZWtmbGVWNHFIRDdkYk5leTdOTkpyeXFmWWJ0dXN6eUVyeFJkdnVTZlZ4MS1MMG9LVl9KS3hQeWVPZTdmNTRObWppMHhoZy1VR0t0NHlrMDF1TzRVNGI5OWtwb2VlanB6QzJ1eC1wNDBRQy1FaG1WbURQOEc0YnktaVFUQjJZVnJsYTctQjExM3Q1ZlR1RzBXazRvR3AwSGh2YkZlQldMOXdwOEMzVGdxOTZNNjJ4TkFLZWNsM05NR2ZqWTZ2OXU1N3ZINDB4S2FzaEhHeGRLWm1XLVNYVTFNOGtnRlFCbDZ6TDlCZGJJM2pDZkNHeWtqSjZKV1B5TThlTmxQdmJuQUlzMk5MNW9XQkJlYUY5SEF3Q2FPbTlPWklwUEFlNGkxT0hBaVFybmVSTXcxS3VORDNwb01QeDFGRy1iQjZDb0FGS0dmWndyRWJHZzR2OTdNSUxxUFZ0NmQ4aV9HSHozNFQ1ZFhrSDlycVg4Y1RUM3R2SXpIZ3VmY0dLckJwdHhZZjhPZVVMVXhGNWZXeTRqQl9jdGxuQWM3MTl2a05EVDlUMDJrRkhHS0pXVXJCc0JlajhUcUR6UDRSQ0dDX0s3eXBqNUl6aWF2LS00N1hUbzU2NFFURTFVM1pjUWppc3BNNTFiUkpmZnVkV0RqNmxkM2RnRkFPRmJKdkwxQlBFT2V0TkNkTTlsb0M3RWRFWUZROEdkZUlDODZkVTFyTW1SVldPdEpyaVZCYkE3aG16VG1pS2dpeHBSUy1ITFRxd0FCREFpRGZ2SDFFQ0RsVWNJN2ZCOTdvd0VGYmZoVlNKb01EbWdQM2JPeE9PdHRhNVo4c1Vfc2ZDODNMZ3RBZEgtWDZ3YkVKMWFDMzlqWXBYQ1RsYVg0YlByLUN2NFVrY1l1THJXNzhXVlBIOWpvdGNHWjkyRXRveVJ0RWhqYUhvZVh3Qlh3NzRXVGNxWXVkajVibC1uZVN4dFhDdkRPUDl5UHNvU1pMbHZRSjc1U3lvRWxvS2dfSkh2N3IzVjRQWWQ2azhtaDZBT2xCNVFnUWdxVUU2aHVISVdoaGJrT3NVWTdrSlQ4ZWEwbk1sN2VYVXIxM2d5S0V0QmdQV01rc2l6djF6MFBxVWZhS3RTX0VqeU5hdlM0WG5rUzZjRHFsckpPRzlENzZpbll3bXl4c3A5clRodmN5V01zaDhabklNRlY3MXNKWDlWQnR1UEhES0FlUFpoalpIUjJERTVQOFdTTTBzVGVFQ1RnSXByRTc0WjZYVm5OUkpCTHFKbm9LZ2tsWlM4V0ZEU2hWdjZjcEE2Y3NhcjhkdTllc2RCYUczczhJclhrSnFpQXNoOTNHVldNei1TTk91Z1pVOEZRNThyOXRxTmxVdmJlcFBjMkJuQk9RMGxVN0ZjVHMyQlJFRzRENURlSnZrQ25fczdjMEpGQ0ZGNDh2VTh5bTd5Y21LQXpJUFlUN1QtVTdsd2Vxc05GUHRlRHVFU0hKcVNzcnBmY0ZDSmc3MXZIcVBDNE5NWVZIR3lPMC0teEEzQTlNVVFhMk93Ml9rNno2bm5lWHlHeHVMVEtwSDVFNzM4WmtfcmZZUURZdkdBYmtTM3dGOEpTQ0ozdS12cmIzSTRQZmxNVXo0ZF9RLUNaM0F6dDJSd2VUTW9hVWFiVHR0UzRPVy0zRGtRMFZXZ0pKU05pMWlPUXBuSWFnYXE4QlczRWNHVDVHTzRsRnd2bl9EOU0wQy16d2pnS0FoYWNwXy1Tcm9oRTdKdmFhajFLaUZJOFhha3ZRQmJvZlFfV1JhaEtoRWlhSmc4STFXdHFYd2Z5SUdWc2hLSTJDM0hUY1lvdFV3UHdLWEJ6XzZfWkdaaFQ4amxZVzY1S3N5UzJLMUtWeDE3dnU0bUFpWG41b0tnZE5sVlFKbXZ6XzJ0U1N3T1dvZFNvRTZ6ekRKZmFpNERUcS1UNm1GdXE2Y2k2UHBFSUFRSC1HckdBVkgtUVhkdWZRZFBCU0x0TDZFUEx1eW5DMkQtc19pNVJodWl5RWZkZ1Bhd3JDMlVFVGV3T3pSSXdCS1J3bVVRejVyeFowcjR4S0Y0cURBMk5rYmxYMXlibUhVXzA3S0ZETlJvNjUzeVVTLW92TDNiRHFnai13TVJrZUFSbnNJUXZEVTk4OE1KUnZXcU5FYnFoLVhwUVQ1eFZFa25rSi1VdjFBeXZVX25vcXNDclo5MEN5ZW95S0JiUXFwMUhwdTBqLTFpcl84TXdIV2tielpUTWR5UXo2a3YyQXg1Z0F6aUotdXBKMUdqVldfdDNvRWdUcndjaFZBa1hJMGUybEF1VzRfUjRyVmRubXU1Uzd1VGRKSWcxeWY4cUEtekJ5R09ISnVnUU9heG1YblozU3QyM1hqV2l5bE10V3VleFUzQjdydFFpRFU1NzgxSnJZT3JCVTJ0cWxNdllZTEpDYXB0eDZheG9aVWw1bkw0NXgwQTJtdTRjYjJpNW1BR1M5V0Z6ZGJvc2NCUEc3NmU3b2Nha0xWMHlDMm5ONV8zTmxzR2Zsc2pyR3czaktyYkpEWHF1YXpWbEVUeHRSU1hMV1FEY3lqd2pNb0pXYUtjaEtZajhVTmhjSEVXZ3NfRGlJNEstUzExT0FjWXZ5c0pRSFBnS29XLV81YVFvTUxWRG9hTWx1NGxteVRyakZXQldZYVhrWFNFLWgtTWtxeWJtVFFSSnR5VzJRUXczeEpMUjFEVG9fcWxPUEhqejBTZU1OR2M3ZkhjLUxBeUEwN3hUeUJMUTUxM29rekp3bERqM3hodlFUX2JtY0xZUVRqcW1pWE9QemZaaHI5aXZzNE9qOG5TZkpIalRDTHIxNW5FT2I3LXM0SWtfVnIxTXBsVFBaQTB1alFoMDhXTmEtNGFLbmNfMEgtcUthMXB6Nm9DUzlaeHViODY0Z1o3QVkteVBpdjhWZUlkbU42djhMT0VGTF83WHlUbVdick5qc2tGcmxwdnRKZHVYbWdHZDZ5cVRicy01c1FUYS1MUlBLMGZJSnZTWEZRV2lkVGg4MlZrWnY2V3pBMW9ISkxlQ0hlU2JYN0lSck5xZDZqZG5Qc3czVFFtTkoyOFR3dDJEOURWZURPTHVQR1NYbDBaWHlmTlBTMUI1T2tWc0t2YVNUXzUtOFdQVXI1XzBGVHItRDhjbkcxTURtY1puZjRyTjRRTkhvU2otZDl2ZGdYZXJ3dkdlbG1ITGpwZmJRRGRFQld1cXJjYjZmUnh6YW5HOFpwbDJGM3Y2LWNBNTRiVnk1YXZmY25JNnRMamRycFpxN3RYZ3FkUXNub0UxUnNjSmpuWkhBRVZsU0NDbjFOZlpQY1RDRlRKclZXb2Uybi1WWFZyb05UbHlqaGtYanl2M1F0dEJXbmE3bUJpcnpOdkgxeHFtLWxJcTQ4bThVTmdzWnF0TlZKTFpGbkJJMEU2N0dZdXlZd2tSTjlrWkZmUjROSFRZWUlmUnBLcU1aTEVLODJGMjBtRURfUjdpVEVzTzExZ0FlMGZ1eEExRkJJNjcyaVFQcWVtbmlyYzBwZnN6VWhLVjFfakRfVW1sZFJ4OFdETG02WXl0Qm1GaGRjNi1VLTlWY3Q3TjZYWFJXVmdBQk1JX3dvempDdEprY1BHOTRVQ1FDREJpbDhmb2VlNGdlS0FZa0lIb2JNSVhUa0VNNW1La3pKaTk2bEVEX29BVUY5NU9uTVNuQXNtUkNjQ29oUUpYQXJTNVpQdXpxVFh4VWYyQ1dSSE5wQ2NlNC1jY3kwdGZlSlpjd0ppbm40UnNOQUpBMi1BTmpTQzU3MEJwV0FMaHNGX3VMV3RkODZYWUhyQ0NtX0xUOHUzdmlWbERWRGM5UXBLVW1KU1hWNGY2c2w4RU1LekxPQlZfOGFnM3JHWUNKamc0ODNiUjczMWx4aFVERDZqUnRHZGdzQ3VtZ0EwYTZOZkhOTjBUOV83ekRmQXFNVG5lVWZsYVF0NldtVjFYSWcyYnZWZk5LM3Nkb0RMdVBEMkJjeXd2b2l0SWdSaEFKZHVpU0s2U3VrZGhPNTJVUXctaC14bEFCQURFbWxQYzdsM2FQWHl1MHFQaWE1bGh6WGFOUWtucGNjSE5NajdNOVhDbnVnaVpzVXRZcU9zZGpQdG5KLU4tYWNPTHZGZGZtZ3NtQjZTZmRqWFl2VkdyYjV4aVoyVUxrU3VuclhOa2MydUhHcTA2QmF2amt3SzAyaW85dml3cFZiMTRzeUI2SF9iUi1yVEtMYnlxdXJRWldCRU42NTc3eUhSUkNlS1ZmZ01nWnI3UjRockxpZWZGQlVraGlEU2ZxUW1DeGNLNFVmQlduRkJkYmRoVnpUYzkxckVQcDVXT191UE5wdjBESGJsYkViLTVPdzQ4MlpWZEdoS0J3Q0FNTWxYN1BBRklLSVhjSjhSRThacHlGTXFZUDAwT0x4ZmR1bHdrMkFBYng3c0NuTGk3VjVHMy1iRmNMX25taGdiaFJGNjhKRERvUHV0R0dSNGdDamNtMHNkQlk1dUxiZ2g1dldIOU5BXzM2ZVNXRDkzTWl2SGhTczBySWlWUVctM3ktbjVRMk15TFptZkt3a3ZhN0ZYTS04V3YxMzMzYVVNcUU4WTgtUkZRb29TemlteVVUWW4zS3Z6Mlo3eDJoZ3BDME9rdnNBZ1JiaDlwakMxNE5VYjF3VHNidDVNZnh1dUw0b1dZcFF6ZEVmYnp4a0ZJYU16SnBxTVVhcDhOaHFuQmpzZlBZbWplaWs2aGl2ZHdlV0FuVGRTY2ZzeEFjQUNTVTQ1ZzlITENqczhqQkh3X2M2Ykt6RTU4SnRoT0dkakZTQnRNcnpDcDdNOEswZ3A3RHFHSU03VDFIMWhsY21nNUY4NENnZXZtWExJbk9lMi1yRVBPdW9EZldRNzIyVFUwS3JjUVdIb0xlcVBlQkxPRTF6LWZFb2FCMWNDcW1aeE9ianpmanZrRUFHdVhjYlNib2UwdHJpNkJkUnRlVXRpRllWU3dqRnFyeEo0S1h4YndKdE1EV1c5cGtuYS1BeWJobDZDTENTMGpEcENVa1o2NzhWOTdoTVhza2Y5U2dnZnBzWUFOR0xQaS1TTEdpa3ljUkVaZ2N1OEhyeGNEemxkbU42TDNyd1JjUXVXWTUtOXllTl9QLTQ5RUtJaGJyVTNYVmJSeThVbDludWRHT2d6ZHlFRHRZTVpmWVU2SkFQajRLMFotMTE0S3RXSmR5Ykg1SEpsbnFXRVV1X3BLY3pVZ2R2cE5vWVBkSFhpZncxb1lHdF80b1dKaFhxR1d3OEcxbExIOXpXTFlkY1RnQ29aTWhVbzY5SjBKT0ZFbFVlREpJaExjV1JwTEdMWDJMLU1IekUxd1hheG5wdmZyMVl6aTQ0ejlQN05rZGVUM241RjJnZEx1bkhaUFNIUERPenJlLTUxN21FY0J4d3JYSzYtdVlNZnJnc3cxNGdhSzFsN1ZhY21zWUw1ZDZkeVJtWktoa3pmbk5nMGFNdVBaWnBXVUd4NjNFUHJ4V3lVRk9RUDVPYUg2VGNYejVDdHk4VXNEQWg4UjNkNzZUejBDR21ubWZjQld1R0d3WS0xQ2U5WHZFSF9TTWJSOWd6SC0yTVNYN21LYVlJV0tINmJxVmNSVUdDMDVOV2t3b29pbUg3czgwV3hoYXFhS3ByaWtndm1HRzBHdXpoc2NXRHNPUDFWM3o1a2lrLUpYd2J3VzcwS2FNalVucDMzQnFDa0RKWkhhRXZfZ1VrM01nSFdTNUhPMG9vY2RBZENkcWlxZnh0bkZaR3Q4LVBla0ZUeEx1RUdvSzR2RlE2UnVtOXBVa2dtTi1TY0RYT1ozUFdBN0ktTmVtT2RXZDZSNjdPcHVDYWhtbzFpZ1VZMUFPVmtya0ZrWW9BeDdJLU9Ha0x4VTdDcjNSQkYzVS1nX2ZfVS1rUzY1QTJnNFJCODVMWWlOLVMwcHdneHJhRDNsREx1eUlmVnBrWGpCckZZNFFRM1Q0Z3JXWXZ3QXM4bVgxaTFra1hvR3E2ZFBaa284VlBCTEZzdHpaZmstRm5FcmdTTlJkb05qdVhVN3hOcldFODRBdENrZHNsaTQ2Y2U4Q1Y4MWhWRXBjTEh4RS1NMUJUVXVVM0hsNlBIWVdNZDlyNVQwZE0wazdEejNLamZicm9DbmdXQkhVMWtZWWl5M1Y4ZzRJSXByUzhFU1FxdEtPVXNINjduOXZweDZJZ1U3RDRxVEF1SFh5WWg0dnRRWktHTmotSVlTMGMwTHNHSFA0UER2QmI1SzV2NWIxZUt3REFMS3N5VTdyWnc0QjZuTUJjejE1U1hDYzU0ZVBwZHpXMThTRndKMEJUSDQteGwzZlJOdFZzTXpBVjA1TVUxOXgzY0VCcmZLUmZJd1pubTZGdEVOV0haRkhtRzZtVkhOVFlsSXJKcWZzN0xFamFxWVE3aUZOc01fcm1zSXJVQ3EwWHdlVDFOa3QwUXRVU2dBa3F3M0Z4eUZaYUo3RGVEWWhBQkVWZmxUeURZSHV1aTR5dG4zMlFHYTAydlJHbENBXy1BUFBFNVBpdkFQeGNkUmtTdU92M2w5U09paUR2dlJtX3RVdzFHbl9jWWRfVlU5SlE0aUlybHdWZTZkOXNWQnctLVF5bDliazVXVVNBelE4dmk5S2c4SkZJUmN5Z2hudS1QY1lwRkxJcm14X1JuM2pGcjNDUGxhYmJSdzRPNW1TX1YyOGx5dExDMmJsUVV4YzBRYWpUXzZfRWZ0ZGpiOUczSmEwbTNBTUtIZjBITE02ZENEdTNRVUY4dVYtRmhacFlQMEZGeERpbDJFTmMyRUR5U0NXUHZnM2FBTzFHZmFJc1Q2cmRmYzl2WVB2cnFLWGNYM1Mxc2h2aWVocnNXYzBGcXNfemhEaG1RYnEzMUVlV3VBUHVRd1BUb19mNW5rVWRpZFplR2xzbkNnbWFERF82TjhkZ0lCOEVuNW55MXBOTHQ5NGtmMHlBcGRDYjNtN25TbXJWZHBwSGN2T3BURndUeXRLRjBpWEdSanFpY2NjVHYyZlB6dHZBVERLbUdsM3NvdXg3dmV0R0V4VU92Q2N3bEpvZUFOakdOZ3hoem1VMGJIcGhfN2wta19BQjRybXh6bGRRYWxJZm5KcDRoUUVacDNFZi02T1hOUzRGSGlzSTI2SktXaEk5Ulhmb1NSQWNVdTMwNGtiWEhtOF9JcFZEUFhIajZFSXpjY2NmUmxDN0tfV0QtSjlQMnk4UlBuaC1xeWF5SklRaDZEMlJUNzBXempUQUstdGZoU1l5akp0NW5sQk0ycUVuRkxCX2ZidEdULXFqTE51bHdrT3BBajQtVm5zZU8zNGx3cjZaaVRFZlJMZm5GUUVFLTJJc3RlcmxrR1pPcGFLQy1nOWg1ZEVKT0d1MXc1cXAxVDFBNVhNNkZWc3VzVEo3QXk4S1B2UllvMVN3YXp0MVdwZzRyYlI3dHdfYjlHUlNWRUN4c3MwOUQ5ZzlXQ21nTXRmYURMekVyNExfQV90OXZUVERwV1ZseU94UExaM2VOZ2o1WVRuZ2x3SUlWLVdET1VHY2RLaUhVbHdvaGU1ZkFfVXRNZ0FrTWw5Z3RQXzBiVkNXekNQY3FFM0NZWkdBN3hNZVVSYXZ4ci05ZVBTN2gtVTdGRmYwS2FiaEl2NFVSSDBhUGEzT2xfaFVHRVU1MWdwSlFTTUUySGR3NTE1clNxYkotNnczX1k3NEItNFhzY0RJREtieHV4VWl5N3hRTjdKVXo2c19vYldDd0FOdnJjbEljTUV1SzVibzFiN0xXTVBZWW1KWjJab1diaTl5NFVDRzNfeGJNaEFsNlNyMTA3RWVkR2xadDNGME9udmxCNUhrcjRTVWsteXBHYkdETjNzeS1hcElmUlFyVUVUZGRoaFB2MklxbGtpVk1DQThXUWdGUmgyWE9YX2Z2WER0cVpGQXBhVXg5cnAzQlhoMHcxTVRTcTlFMEo0TThPTXFJOERwa0hNcUtTanpKdFhRc3oxZXE2ckZLZkVIeG9fa1FrVWlXT29kR042WDE0UGhFSUk0Tk5Sbk5HNDFjMTRVdjFUWTFlM0NnQjR5aFVPV1pXNG9oMFZCVHB0czMxOHZveUJYZHVaQlpodWZJekpQaVBoYkctMnBDa1hKYndLcGoxQ3hveUlYWHU4WGFvMjZnZ1RqZnV2WGpTT3IySmNaSTBTaU1MUDV3cTd1X0FISnRZcURDOGJPRUM4R2swZktJMGFIREdqS0VWRVhFdU56MEhuZ29BVXowVGh2UjJwdi1MWkZsUTEtY1p5SlBoeWRobW85UkxWNEo3THNnZXJDcG5OdmZXZmNPZVhJaTExeTQ2T2VVaVJhb0xsTXY0X3UwQVR6RFI1a19OemFyVnJsSmNhWDRFWXUxVTI1STFvN0tvQXVhUDI0V2ZJTWd2UGZhX1pMMl80VmN0YTZETlpWbWowVjRnUnNBdXRMcC1fYnZocFpUX053OXhLQ3RjSEFEWldDM0phVGpCR0dhR0ZEaklsbm15ZEQ4NDFEVDBQa2lpQnA4azN2cnJRR2U5eEtONUhRV2tDOVYtZU9CSEdFSVhBM0UxUFh5djVMVDc5bWk2TUIwRnlDaW9ibEVlUkUzMUdPTU5HX25wOW91c1hINzhMSmZiS3lxNTRKOHB1NVVHWmpVN0ZoOS1tOF95SWpPYkRRMXpiOW9nYkI4Z1U0VnF3aG5RVndzTTRKZ0VUd09HekgwNUQ2WXlITkd0bWY4OGtUN0M1dkQ3MFpZX3dMNnd2VVlyM3VFYzVPb1pScXlnM0x5azlRZ1RxR1Vna2tZR0NxSDYzMUM3M21MMk1LVDFkOXNsUENicTNud0pvTmdMaWlHSG9xUHlNR1BkajgtRTFMNW1ocUNORmlIMHAwVGxzSGYzUlp3MnVWVHpxMzVVYTlDcVBNYWpqVG4zaUE4d1pZMHo2MUgxSFJoaE1Vb2xqU2dKdU1Xdy1HMWlnUHpVdnZYZTMwSHAxQjZkZXRJdmVsZmlWMzFkYWlnU3VqdDQyQ0hwNWl5alB3VmRJUS16SDdELW5wR21BTUFsLUpKYjFvQ3l4SGdxdHFld0JCMENxOWJQbmRmR0tlazd4d2c2dVVVbjNLSHpCajBUMTM2OGcxU3Zyc2dfY1NRN1ZZdS1mT1IzMnpWS1dHdmh1QlZ4bUtHTnJSbk0wWFlEc3ltTnB5RVdmM1Zycm1sNElYZm9GN09wTTV2dzJiT3pWMUQzVnZNc196SV9wcld1WkhoTmRuZ3JCdkE5VGpTU3pCdG1adWNJZmNSQkJib3ZVemQ0YklZZGlVRHd4dTJBNm95MXY0d3RrZnJPYmhfcXg5aGdRQ253Wl8yZVBjTnZ4ckxfRWVkZkpaMUZhSjRob2R3S3lQdzB2SkliOGRJQm42allycVNlNFdVTGIyWGx5eUFCOEdnVldEaUJuMGJreXl4WUtUX3VzZDkwdXpDTVRmNHowVTlxa3l6LWhtUGYyVlZHX2FVYTY3d0VIcTJ3MVVWYzFwM0hnbDRwWDZEZUtFeFBjMXFqczNvZkNpdWdVdjMxQ3hiR3V3b2N4aXRINkNxWGxfeVNxX0ZJY1hHY2ZoNFMzNzUwOTNnazNHWVNaRE81TVhHcjQ1MnZnSWw1QWlnNTk1Y2I5cU92RWh2ZC1LdkF6Qlg2ZDV5TjJ3UmxhRC1aUGtqWWZuT1lMTkVab3pkWlVFclBxalJ3MVZuZ01jVHZhczZlejg4ZUYzelFoQkZvQlBybjJwZEpsMzBVOFh5MWl4Z1diTUhjdE0wb3JCelNub1VwdWJ2YlBUMXdfbTYzZ05jTEcydy1QZG02ODQ0ZFlEbWp4a29Mei1uNG92Q19jb2RIbnNNMEZiYjJxRlF5c0pqNEN5TlROaFd0Wm5lQXhBZl9BWTV2MEViYU1Tb3lxVmVJNEhIRl9EX19RdnQ1SUhQRGh1c0FVTjlUOXVFYVZkWnZoZzd5bXN2c09wQjBXS21rMGwxYm1yaUhPLTI5Rm1PMGZ0YnJ4VTBrOW5RVkROSjdwS0pGaGV4RFFHbmpwcGw5OXp2WFpGOXJFYVpjS1lzcDhXSUxRSXlQWjg0ZGdwT243VW9FZG9QTG9FOTdsbnA1aUhlajVkcW1mTlRmNXREbUhMUXktaW9pd1FmOUxqUEhSVXpWNXdDOThrazl1T1dFQWJyVFBpNW8wNEl1MVZJUjY0TnVkTHhvS0g0ZExkdk1mVERyXzdtTTJaLWFFTmM1ek9qaDkzNTFqSVY4azUyZ3VfWHlMNDBDV21BUHpZQVRoQUlUX0F6TndaZXRRUUFDWDhFM05qeDVwZlNsU2tsM0oyTXVPMDJmRXYtRjBnQUI3SUw5OWZCN1lYMHREY1JRdUk2WWRIcW1kVWluTkwtOUJEQUZKbUE4YVkwekpkMXpPVmRuTXllb3JncFpyOFpJbmZDX3ZTazRYMmowUHd2cU1jckxzN0FCSDBUVkhiamlFd05IdHZCV25sTzJGenBLNE41TWNsTE03LV9mUk5zaWw4bGxmZUd4WE9jc1RYaTNaa2l0QUl1MklkcFNxWEZTT3NhNU1NQjE0LWVNWm5oMDQyVDA5bUZrVzBIR2RFNlVZa0tsaEhQU2dFLVhiRTdpQi10OHNiaWpQYm1hNTh2VE5NXzhKVVg1TGZZWlNwa2RVVS1CZDRtaURpak5oeGpZY3A0cC0tZTVUWjNwZmRCSTNnS2tydjdIcklFdEVsLXdXdlRER3kwODRkWXFTNDl4eFNuZEJrYzJiUHBwckRaVFpwamNBRTdLQ3o2Rllncjh2aXdOT0Fpblg1UUhIRFBnREJxcnNZNDl3aFdJbHVSOEUzcEJpanZRMVd6cDhfUk43Q2pXTEdHMEp1MmdRcjVFRF9WNHhIU0I3cVpiOXpWdXdrNzhNa1ZaNVQ2VUFMSnVlb2F4RTJyWjdlWXZPbTRnUmotSVV5LXZWNTEycnhybXhNU3l3YWxlNE5IbFd6MDAzcW9qSFNXVmxxQV9wWGVJRVcwTGhHZ3ZTWFZQemxJZkg0NVFpYklmSExfMHFwanJWVUJDQUNEdksxVktHZTVWeW9xdnEyRjBxT3lRc3Z2cFJ0MUQ4a1lraHktYkpZYlNCc2ZyeWVvb3lyZXhoNnFzUzdVNGFGZExKY2gtcW9fVW0zNXBNMDkzTTVtemZWY3lwQzlBN0lnS2toQ2JkVFBOX2Jnd3prdjFqdEdPRG5iaTl2SjVJQU8zb3k2UGZtdkZQNHN6cENkN2MzU01NY0hkZE9ncVJkZTQzTFNRSWdzRVBMWWdtdFZ4WE1TY1V5ZzRqb1QwUk5KMGI4TmxZelZTeW9pWlZla1ZEb3RqdDdyM0t6N0lvcnhWQ3NjSU1NakFEbUUzSUIyWklRdzF4ZlR6UE42TGZjaUtkSzBKVHZoWnJMMVl2bngtR3Z3MjJVMnlQZ3NFQnlTbEN4TGdaV2pwa0lzZXgyemhkd0t0MVRMNGFBWkhDSGwwSmdkMXgxVTBfV1g5d2hTYnQtd01fVFJLY2MxNUZPV3BIOWpwNlFxLXRxZkl2Q0dEQ1FtNFVxQ2FQTTRrb3JHa2lEc0Rmc3BnX2J1R1hWbEtZdm5ZSTFJTUlWNmQzdVRuWEhQdk1xNDlLMk9sd0lDRVg1eWl3Y1l5VV9ZWDNTN1k0aGdSRl9JSXkxYzNaMFc1dDJOblo5cUVpY1FYVVNSUjVZTXh2VmE5UTlGdTR1VF9xMjQ4RHUzUmNVQUJUM0tqRW5mRmgzb2xUUDhKRVpTZTlOTUNVRGNZSnB5TjNueTZneE5CWnVsa3R1SEhTcVFwRHFfMGhaTDJjMGxtOGVleGRnd085WlZqQ3NSOU1UQ0RoSm85SVdJMVN2MHhoOHg1V3B4VU12Vjl1Zm1jRTh1ZmpkQ0FSd0JSS0VJWmZEUzBNb3BzYVhhQ3ZSaHJqLWRZUVd5Vjg1UzdSYjU1VWlVMUdqejVJOG5jTnRzY2dlTWc3VFZhZUk4RU1jQXp3UkthT3BSakhYT0JqUXpTRkhRWU01Y2JHdU5qc2ljVVpDNnh2V0JPQ3FrUk9tNDFGVTU3aXJUT1htQW84OWNIRDUxWGxWQldCeHZvNHBGQzdaYy1idVdQOXVsYTZ4WWgyeEd6QUlHbzI5Y2xQMV9aTnlER0NrbVhCalJMbFlEeXRlalI0ZldEcC1xRTVwTHR4c0hBQktSNmV5T0c0MFFxbnV3ZWZZN2VPeGdqUzV0eVd5UUI2bkd6a0I5X3ByTVQ5dF9qdkQxZnFGNUpVYVFQb1FqbWtVaWp6YW1KMWpocTFrcHg3MktFMG9rX3liUWFrOG0za2d6bVJyYXlTUm51TGJkV2c0X3ItMThvVzFGejM4NnQ2c05pMzBxeUZxcmJVc3laeUY1M1BKSVJ6WUF1T0RZajh2bVBoNDJvUjg2ajAwakItWkZnZzd6SUF3eTBfQWM4RUlWSDFfWUJtMWFHeEJ2NkVfWHNLVVN0RDJrMlN5X09FVi1tSmt2R0tsZ3FnY2lUX19UM3VkSVQ5UDZjbGluYmk0WW1Ga1BBWXdjUXZOUDBwX3p6U2p4NkhldUx5eGtsS2lIYkJhVFY5a29QLWFZcXllX0gtOEJoVkNlUmhZYnBOU1V1OVlfcEpfRWdrODZRS2Jtd09YdTl3X3FuWHQ1c3FvUjNlRE04TGl6WmNLQlZ1blNyNkhNdDBiYS16MEJFWHFXMlpwZDd2QnQ5V0ZyZjFtLWpLS2FPc1cwMTBGY2ZrclhtS3NXTkVEYVRqdHlDdjlJOXhGUm9vSkFNdWt5aS1PZ3BPaDd3ZGM3RE0zNmZObnZGZnBXMG5FTUZQYU9WUUVOb000ZkFWU0VDVEZtSmptSEJLaVp0QUNWaGR4bVV3WmRES2JKYWRDS3dnc2stdGt6QWc2c2haTnNKdnhhX3J3cS1KcG5EX19vbXZPOWRNeko0ZmFPQzZQbnA2Q08zbWRHNnJyLUF3OXVRY0owanRZVXNEeWNZOUhXY0VPQ0pqdjFHbF9VV3pQbENxdkkyRndZQmhxN0RST09JWHYwc2RsRmlFRDF0SWI0OERfNDluMDdqYTJEbm45UldqTWJwaVp3LVE2U0syYkp5eEszZU56U2I2ODZRTFJXVTFIZ2RrbWFENWt1ZWplajZtNk1ObjBmVUlCYUdDaFhmYWVEdGU0QTV4WnlsWEZFY0Q1WmJzV0RYZ2hzTzlQM0ZxUWl1MDlPNGo0cjB6TkpucWZRWGRkM0FBNW4xTkh2RXo5UE5hYlMyZHBTaG5RYWpTWVg0V0U2dGtMNTVmeDNaNlNUVGlHZnRMWWNwZmlmSjI1dXZ5UjgxY0QyRFRrbzRhLVBtRi1CLXY5czFfMDFRV3d3eUo4Ty1kM1MxOWNnWkNoa243YmZiUk1NZlNyMGpyYUw5Qk9QaWZwVlVWdGxuR21Kdk5UdEcyVmhpc1BUYl9PMzd3X0o1OFNWTTc3RjdHRFE2dFZaeG5sbzR5aVRaMVkybWlVVGJFeHc2SWxWeDc3TXZkTHpPeE04NEdZYUlJQzVyV3NPaU1uNTc3QkhScDFFVnp6S01fdWw5eHR1MFJ4RERCS1VZamoxM1dadEhrczJIWWFkVjhBa212aGpQUzBBMlVYZ0pMWXE0eWxMT1NlcmkxQzhlMERzQzdHbUpTMHRfMS0yakN5b24xTXhlX2RRb3JFSlpiOFFQazZKR1Z2SGJVVDd0T0YtMW9sQWVFaWR2TlpjX08zSHg2T3p1X21uYUFkWldCOV9jZWxnOTJZZkFicTl6VldxaTQxYzRnT09BOVp3Qnk1bDViUDgzcUNVNENod0F3T3BrMTBWX3ZueHBId0t2UHJVVlRBYkl5RVE4RGptOHVYaTBvNW9XQzl3RTRNWDV0MzNmaGdsX3dzZkZ0b0pQNGY0bExSc1ItM2Z5UzZzOFVzVjMxeXAzSWR1WUw2dkc2NUU0NFhCdEJPTUhTX3VsSDVpYl9OMmRULXlLU0lUNTJxYlMtaHJaS19ycmw0WVR6Rk51dFdiTV95dzBMMERCSFQ1OFJEdzZyZ0NYSmdTaVRRZXUybmdTdVkySFh2QS1uNnFKa1JQdzdIUERnRzM5amJJc3hCaTFTYi1zMzZwOVI3NTd3Q0toZ3F5WlUtTEZEZFdHVmVRYUUxUjY1U2RzSG9iZ0pmdFZiLUJrZEUyZHQydjFFZXp1RnNoanp0WWI4c0VUa25PcTN6SEh6aEZIUE05WmhFRjZ3X1pwTHRmT0FkYmhJZDJiUU1CN3lCc1VHUmhCdkJYYi1NZ21jUW14Ykt3dHR4ZW9jbXYyRFN0LWgxbG15VWlWTFoybWllUXVsM0pBQWVNWENJa01lR3doQWkzUGJMZmhsZkFsUGZkRVRVYmZFSWEteHQ5WHdvV1hzUnRmNHBrZWlxUWlhbWdGV09xeGkzWG9DcTd1ZDh3c3NieXk0LWRvUGRSTVRSNDE0VVNyblRLOTBfdEdTVkJuWTRMVEdkdFp1OWlWc3ZESnFwRFBwa0Y2STV2Rm9rOXFjNnczSlJtSlRjMWk4S0Jhb2NkYTNpajJXVU90WnphZUU4aUplOFpxU0x6V3dKaHFhcnNmc2k0My1iZkhpbTA1MUd5MkRfT2NUT0N6czdWZXl4WHc4RkUwS001cWhmWkxXdjJBOXNwaTE2N1BXVTRpN1JBa3NIUkhlMUpwQTNRWGpWZ0VNQ3NGN0lNLTdtUkVrYW9kYVFWSXc4MGpQUFI4SXFzcmJTbV9fQUZYaVpWWVlRVXE1SXRzRjBYLXU5SUxteGI1MGVSZ3V5c2pTanM0d2VaQ1dUUnFZTXBuUmpKTW5RdFJDY0c5Q2JiX2dCMkpheFZBVFBrM0Z6ZTZwaThuQ1NCeWpKSTlDa1V5SnpIX0JyTzJENXZrQU5oUkxja2wydHI1VGcwNkQ2Qm41d2lHTjZlX2ZhdkJFWkhUZ3pYZ0dKeGI5RC1fME5nOUpmNXVGbkxWMHBzSl8yczZ0WlRlUmZVMDUxOThTRThFTTVjRlhoNndTaXRNY0FJbHlBaHdPY3JGenFUQlBVbUpWLXJuTTluSUtBazJfTXVzb0RVTXBoTXYydnBNa3FRdFoxVWhJRk9YdkxWa2pKX21ia04xU3Z2MkdxX1ZJQ091YzlEQmpyWl8tR3ZTZlVNR1RvVnRuT2ZKUXBjbzU0anI3UW11NkJtS0EyN09SM3AtcjJMNjJDZ0tVeG1UNkNmN0FoYjltRkVwWXl4QmJsR180ejNaZEdBMjczVExKaXdKQmlVU3lFdWZ6VWZfQm8yN3BLX2tOQ1NfTXhaNlhfMGx0UU9ETnVPSW84Wjg0S3hkWlJKaVhSUTBZU20xMzJqbDVvMXNEVzRJakRVdWhJNzhkM0IzazJUUi1KODRIYnkxV3Q3WkVraTU4Sk5adzRGU29mQzlFSXlnTzBFRUh0ODlyaVhSN0pPaEMwS2tUcVpINzNYUzA1N29WaU16WlYtdWlra0plUXJHcEZXWGJMV2MwTjd3eWZPcXdfNW1rcXRzZ1RQaHd5aTQyb3NxUTRDM2tNSllFbjg1ejU4c01YZDNrenkzRjVrUDhmUjl5UG5RZ2V4czBIa2NoZUNCSGZaMF9PZ0hhbGd3QjN4U2VpdGpKY2RXeWlWTW5kVVN1SWFsSS1RQnpFd2RQMGtWcENqUDFsUHZTOE1KOG9uc2FaUmJKWTBaQzlNUUpwUERWbm9jdmVHNGZtV3BuWFhuU01EejA4UzRuRWNabXpPUlJYUUJ0TWV0alJRT3B1eE5WYzBxRFNsWkNHbk5xS1dFM1hkdl92MXFaT2tuazRvZUJfUzBUOFJOcW53YXh5WVNNNVZ6M1JHcXcyYUE2aDN6b1FiWFA3UnFOUlZac0hPRTViNVdlV2dMN3VTVnZhTE1zRUdJVWNDMVpBTGd0b0NDLTFSdVhob0dYcHhvajhIUGUyYTJWTERuVzlUYlNHZWtaWFgzXzZyc1BtbTJEM3R1MXRtaE1sRm5TdFI3bXd1dXJ4NEsydXIxamJ5NzFrLW5odE9yTmFWMkt0cGhudUlJa2V2UFhrNzhhdXpTS1VaYmFDTUFoVmdnY3czX0xSbXY1d0FRQ0NNaXFidFpCNFgxbFRGZm45MTJqN2piXzZUXzRaN2hyYU9ZRkJvaWJZX2ItdnFTdUJNTTItRm52cEoxNEQtcjlHZUZ0SnpwNWNzOFZ2MmJtLTgxbng0Vk93Vy1HM0lrWktuVmN6RVItSzJHM3VKM0FhN1YtLVVseGR3S2U0V1pYa0tHam40dWtMOWdwNFByT0JmZnM3dy1hTVViQ3FxcG9UUzVCT1RKQ0tZbGcya184c3lmRlUxOHVDTEt6Y1dxbVdBdGt0TGkwUldiOFRjRlhETV9VTWE5dTB4eThkWFZkMDY0YXhaSExJeVdvUUtDcGdyYkU5RWZPQnFzUzJBSjVCdzhXSUhCS3lJYmN5clZ4bnlPNjczbEJac0cwRXR4MEV3RGdpcFlVLTFOZ213TGJvQk1Wc3hEWjhYTkhyTm9FNTJIekZQLWZUMFlQNnVNLUQxOHozU0lXUk51dUJpYzJyX0RwOENEbU5rbmRiWTlaeGRNZG1vOGxQUE03M0ViRzQ4aUQ4YnhOa2Z2QzZqaExYWHQ1U1RfSTc3MWE5TG80QXhoY0c0TlRjSjctM20yemdhUTlHWTZSMG9wbHo1U2VtSjhHQ1NFc1hCT0dIUE1EbnJlNlp0R0FRM19saGF6WFhoZkx6eWRQRUUtTGFZSzdtaTJSTV9ER3FkaWkxQi01OHQtemJRYXUzazAwWTRJTmVzd3FPY0JJMHVoY1ZJX0F4ZGpEbUE3WFpIV3NlOE0xenlIRklZVHFDNDlXNGI1VHg4V21maWtZWjk1dTUxT0tldk9QOTc0WGNhUUh2TmU0QlVTd0twU2gyb2h1TVJubXJKbXBwQ0F3dlBOWC1yTkhZTEx5cVp1LXNEOWlrOXNxVG1hTUdWMEFNSjIzcnpUOGYyU2p2RFpmbWppWmwzaFdnaGpJRzFacDRIMUZxWFNFaGdkSjE1MmY1T1pkZU9UR3pOMmozNUg2T2RIVS1UeHdQdDVwTENWOU1mZElvSFlLbGRWR09faU02X0NKMW5PSUlIekVJSXVnTjJ0RWVKd01ic2lnb2dCUk13cHF4Zm1UamNtemZRSjhQWTA5TEpVSjBSSzhPbjNha1RWd3hwcDEtRHBMbVNjUTBYVFhPbmZwcW4xUjhHcmFSdUs0YTAxOHVMYXMybld4TmJ5Wno3UlBnZWs4b1FJbE0zWGllY2hSRGk1QWljTUpuSV9PMktzYkJJaklheFFDQS1TelhDcDlFOTZFZnZWeEtHelRwSzl4MWRSZXc5STZ1cFdVUWc2VFZvUzNRNlB6LUJxRDlqN2NmSWdGQWpRd3h0eDVVZ2xUc3phRjZFUk1EeGlIWENMOFd2MmJfbE1PblJxU0RCSUh6b3lZMnp6Q3ZpaDBYazFCRGE5ZHRnVU41Q1duYzRFbzVkbFpOYmxXNVY2MWw0LTREVmM1TkNuU2N3NjAyU2ZfcXItcWp3N0tSU3hRSEt6YmVMMUVzYXdYMTJ3RjNtYlc2X2RJVWY3eGRHNm81VjZGWmdlWmtCUzluTTdGZWJiTXNIZ0lZTVRwY3poYXRrV3dqdW01OEYtbXJfMUVBdkFHR1Mxank5RG1XZG5sNW9VN083cklTYkNaVmZNUVN4STJMQ0ZkN1pPRXFDTC03VlRjUm9LS3BRSXpPbXNaQzlWSjVkdF9oc0syS05Lc2l5a0NDcEx6c2NSY2JkSUVxSTNoZWxUU2hvVzdINlV2eXY3b3Q2c3pGWElWTGJVMENmVmFmeVBHdTlneU5VZjVzeU53amVhNVY0LUk3VzBERkF0SUdBM1hhU0JkcndFaWtMWkp6RlZoZnVIWFJUT0dHUjRJeGs4YlJXVlNWdDZac1hiYlRiazVraEhfel80U2JEUU5GekNiZ0RXXzhqbFRQVlZwa3pIaDhjaVYwNUYyYm81ODZqTzVMOFNZdFNCcFpXSWpUUnRkOExOV3NVRHVGTDJjU2tiNlJ3a2Zuall6OTRYVGhKQ052WFIxd0c1Q0h2WkFHYXNmOVlqRkZqZVFySnhqdkgxZWdLeWV5MnA0TlFEZ1dnYlA4MGpDa18teVI2Y3U1bXkxdmpsbWtnX2pLSF95eHhkdElOY2lvRXlFNWxIQ0paa2RTR2RIc25pNjQxYmV0ZGJ2dThuMFlnaVltWXlRRVNpOWNXTGZGaVpBYnpGYWMtOFVIeTE4N1lhb2llZzRXd0V3TlR5RzB4ek11RDJvN192MHpVaGZTdThrbnRZSlVLVGl3OUZsS0RRSXROcDFRdTM1WmE2WTRGY00tekk5Y20yVkxMNlVISVRZWDY2bmN5SWZUTVBVdHMtOG1hUXYwdVg4RFNrWmJreHVLTmhFV1JwRDNIT0VsT0lMeDVWdWdDSjFOdVM3ZDVyenBtV1FIU01TbXpvMHJYM1NoM3pwem93TV9SVFYtVWw0SHpvelVSM2ZhYURUOEczbnNJQzJDYWlqQi0taXNQYVgxb3lzczg4dTBqX0ZpTF9ncGVtWVVYQ3l4V1hZUjFRYWRlSk5HdDJxYW9maE9tRG5Ud3doVi1YWkdBRTYzTTZteHRKQzNXV0pNUUFsYXVwNlZQRXVkZTN3cVk2ZFUyNGpqNEN2VGNTVW85dkpFb1RMaEZtUFFiOG0zQzdtallJNVM0cVluc01DN2VFbVlUd3h2a241SXViQT09
```

## File: tools\demo_utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64encode
from decorator import decorator
import uuid

from odoo import _, fields, modules
from odoo.tools.misc import file_open

DEMO_BILL_PATH = 'account_peppol/tools/demo_bill'
DEMO_ENC_KEY = 'account_peppol/tools/enc_key'
DEMO_PRIVATE_KEY = 'account_peppol/tools/private_key.pem'

# -------------------------------------------------------------------------
# HELPERS
# -------------------------------------------------------------------------

def get_demo_vendor_bill(user):
    return {
        'direction': 'incoming',
        'receiver': user.edi_identification,
        'uuid': f'{user.company_id.id}_demo_vendor_bill',
        'accounting_supplier_party': '0208:2718281828',
        'state': 'done',
        'filename': f'{user.company_id.id}_demo_vendor_bill',
        'enc_key': file_open(DEMO_ENC_KEY, mode='rb').read(),
        'document': file_open(DEMO_BILL_PATH, mode='rb').read(),
    }


def _get_notification_message(proxy_state):
    if proxy_state == 'receiver':
        title = _("Registered to receive documents via Peppol (demo).")
        message = _("You can now receive demo vendor bills.")
    else:
        title = _("Registered as a sender (demo).")
        message = _("You can now send invoices in demo mode.")
    return title, message

# -------------------------------------------------------------------------
# MOCKED FUNCTIONS
# -------------------------------------------------------------------------

def _mock_call_peppol_proxy(func, self, *args, **kwargs):

    def _mock_get_all_documents(user, args, kwargs):
        if not user.env['account.move'].search_count([
            ('peppol_message_uuid', '=', f'{user.company_id.id}_demo_vendor_bill')
        ]):
            return {'messages': [get_demo_vendor_bill(user)]}
        return {'messages': []}

    def _mock_get_document(user, args, kwargs):
        message_uuid = args[1]['message_uuids'][0]
        if message_uuid.endswith('_demo_vendor_bill'):
            return {message_uuid: get_demo_vendor_bill(user)}
        return {message_uuid: {'state': 'done'}}

    def _mock_send_document(user, args, kwargs):
        # Trigger the reception of vendor bills
        get_messages_cron = user.env['ir.cron'].sudo().env.ref(
            'account_peppol.ir_cron_peppol_get_new_documents',
            raise_if_not_found=False,
        )
        if get_messages_cron:
            get_messages_cron._trigger()
        return {
            'messages': [{
                'message_uuid': 'demo_%s' % uuid.uuid4(),
            } for i in args[1]['documents']],
        }

    endpoint = args[0].split('/')[-1]
    return {
        'ack': lambda _user, _args, _kwargs: {},
        'activate_participant': lambda _user, _args, _kwargs: {},
        'register_sender': lambda _user, _args, _kwargs: {},
        'register_receiver': lambda _user, _args, _kwargs: {},
        'register_sender_as_receiver': lambda _user, _args, _kwargs: {},
        'get_all_documents': _mock_get_all_documents,
        'get_document': _mock_get_document,
        'participant_status': lambda _user, _args, _kwargs: {'peppol_state': 'receiver'},
        'send_document': _mock_send_document,
        'add_services': lambda _user, _args, _kwargs: {},
        'remove_services': lambda _user, _args, _kwargs: {},
    }[endpoint](self, args, kwargs)


def _mock_button_verify_partner_endpoint(func, self, *args, **kwargs):
    self.ensure_one()
    old_value = self.peppol_verification_state
    company = kwargs.get('company') or self.env.company
    endpoint, eas, edi_format = self.peppol_endpoint, self.peppol_eas, self._get_peppol_edi_format()
    state = _mock_get_peppol_verification_state(func, self, endpoint, eas, edi_format)
    self.with_company(company).peppol_verification_state = state
    self._log_verification_state_update(company, old_value, state)
    if state == 'valid':
        self.with_company(company).invoice_sending_method = 'peppol'


def _mock_get_peppol_verification_state(func, self, *args, **kwargs):
    (endpoint, eas, xml_format) = args
    if not (eas and endpoint):
        return 'not_verified'
    if not xml_format:
        return 'not_valid'
    if xml_format not in self._get_peppol_formats():
        return 'not_valid_format'
    return 'valid'


def _mock_user_creation(func, self, *args, **kwargs):
    func(self, *args, **kwargs)
    self.account_peppol_proxy_state = 'receiver' if self.smp_registration else 'sender'
    content = b64encode(file_open(DEMO_PRIVATE_KEY, 'rb').read())

    attachments = self.env['ir.attachment'].search([
        ('res_model', '=', 'certificate.key'),
        ('res_field', '=', 'content'),
        ('company_id', '=', self.edi_user_id.company_id.id)
    ])
    content_to_key_id = {attachment.datas: attachment.res_id for attachment in attachments}
    pkey_id = content_to_key_id.get(content)
    if not pkey_id:
        pkey_id = self.env['certificate.key'].create({
            'content': content,
            'company_id': self.edi_user_id.company_id.id,
        })
    self.edi_user_id.private_key_id = pkey_id
    return self._action_send_notification(
        *_get_notification_message(self.account_peppol_proxy_state)
    )


def _mock_receiver_registration(func, self, *args, **kwargs):
    self.account_peppol_proxy_state = 'receiver'
    return self.env['peppol.registration']._action_send_notification(
        *_get_notification_message(self.account_peppol_proxy_state)
    )


def _mock_check_verification_code(func, self, *args, **kwargs):
    self.button_peppol_sender_registration()
    self.verification_code = False
    return self.env['peppol.registration']._action_send_notification(
        *_get_notification_message(self.account_peppol_proxy_state)
    )


def _mock_deregister_participant(func, self, *args, **kwargs):
    # Set documents sent in demo to a state where they can be re-sent
    demo_moves = self.env['account.move'].search([
        ('company_id', '=', self.company_id.id),
        ('peppol_message_uuid', '=like', 'demo_%'),
    ])
    demo_moves.write({
        'peppol_message_uuid': None,
        'peppol_move_state': None,
    })
    demo_moves.message_main_attachment_id.unlink()
    demo_moves.ubl_cii_xml_id.unlink()
    log_message = _('The peppol status of the documents has been reset when switching from Demo to Live.')
    demo_moves._message_log_batch(bodies=dict((move.id, log_message) for move in demo_moves))

    # also unlink the demo vendor bill
    self.env['account.move'].search([
        ('company_id', '=', self.company_id.id),
        ('peppol_message_uuid', '=', f'{self.company_id.id}_demo_vendor_bill'),
    ]).unlink()

    mode_constraint = self.env['ir.config_parameter'].get_param('account_peppol.mode_constraint')
    if 'account_peppol_edi_user' in self._fields:
        self.account_peppol_edi_user.unlink()
    else:
        self.edi_user_id.unlink()
    self.account_peppol_proxy_state = 'not_registered'
    if 'account_peppol_edi_mode' in self._fields:
        self.account_peppol_edi_mode = mode_constraint


def _mock_update_user_data(func, self, *args, **kwargs):
    pass


def _mock_migrate_participant(func, self, *args, **kwargs):
    self.company_id.account_peppol_migration_key = 'demo_migration_key'


def _mock_check_company_on_peppol(func, self, *args, **kwargs):
    pass


_demo_behaviour = {
    '_call_peppol_proxy': _mock_call_peppol_proxy,
    'button_account_peppol_check_partner_endpoint': _mock_button_verify_partner_endpoint,
    '_get_peppol_verification_state': _mock_get_peppol_verification_state,
    '_peppol_migrate_registration': _mock_migrate_participant,
    'button_peppol_sender_registration': _mock_user_creation,
    'button_deregister_peppol_participant': _mock_deregister_participant,
    'button_update_peppol_user_data': _mock_update_user_data,
    'button_peppol_smp_registration': _mock_receiver_registration,
    'button_check_peppol_verification_code': _mock_check_verification_code,
    'button_register_peppol_participant': _mock_user_creation,
    '_check_company_on_peppol': _mock_check_company_on_peppol,
}

# -------------------------------------------------------------------------
# DECORATORS
# -------------------------------------------------------------------------

@decorator
def handle_demo(func, self, *args, **kwargs):
    """ This decorator is used on methods that should be mocked in demo mode.

    First handle the decision: "Are we in demo mode?", and conditionally decide which function to
    execute.
    """
    if self.env.company._get_peppol_edi_mode() == 'demo':
        return _demo_behaviour[func.__name__](func, self, *args, **kwargs)
    return func(self, *args, **kwargs)

```

## File: tools\enc_key

```text
HjlTVuGbHXeCBWAhlIDfR3xZcCtykbMmvRoeGLARDWN+AhOdTffFb0hR+3GkMpwCvTshTkIx4nJwdFyIx/g3OuKpGPQJLi1d/4IhwYyhhSFAuHJRWE8rm2Z++aZ4Qd69h4bQjr5ymy/icqiYZ6Atyu1EXfunZBk5LsctoTp99htyFD4EzX6IH+9sBzvb32tkdU7p4LBYalTonJr8cK7IzTHyoDhg7UIwArztHwCrnf2SMLDJB/BdiihN0alf+vNN8/FOhGmJY3qH2nqrDTKk96dgUoapNi6RRWrE6rzmcBL2Lr/dBYfDXqJKqC178+m89ezaPySSToj8MNKNpqXFUw==
```

## File: tools\private_key.pem

```pem
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCj4tVufRBH9j1Q
1cag4xHHgYhNOWSTPtkZbB6ZVfxng/MLGgUF+InNRn2ZWFKpTsQucSa1JKWaYWt+
gmq+7NMUu1smgdhyZxbGXubVEm8bp1AQO7432V4pOU06NW8i6YqRb79ru+wY32RN
yWU1GnDyoGFfndM94ewlAkP9THwg6RHl5uYPcy+4PLRixhnMi+CoPhJ53yA8stHZ
rsTKpYXvMLgbOVjI5+7mRq4ixlbmtIwifXdv0yExcj6Q0VJ+iq8aRxcxIRlNV6/6
ifp7+cZUNpw/KzWVOQ+DcMgzMir7nn2YatLSPTcqv7YjQqpS/D2ZUu8S/SNxmQ50
nF4/xoknAgMBAAECggEAen9gfReqBb/+kN9ZeoR/k5o0oVRW4uDFMYVpUb+9zDoK
fq/SNWZSykb4NpiYIMkpRnV5M0jTJ5PN31/oHhGyrPpl5WCEwu5fTaM98uG2fvsO
kzO0uNYW1cVo/itWiMf7tT3L3OE4VlcUCDiTF6BN8G0Em43CiazG79rDqx9yYL3j
Vnj/NLiY0OnxXL6SkjxZHvjXPl8jjTrf8oZl+twpwskiC5cqIpfsTGSsDq5WS2rh
eYdGhH4EcUAfK5PHTTIwBGFReekrTXv1PXm3EK36T8himQVX5sSd2F79QsVXsnfM
PR5W85W7MhEKYD4mZSQsKThj+CCE8G7U5uJ2Ah+LYQKBgQDOPNQYdfrY2Ivcy1X7
umVniEyoJ5fs3GLRwXTH+nPNxIa5/YtzLqd17GU/Ec6nyEDGcDqEpuKfopcEEqqw
w6Z9JPudOPJoVRugTHkpAzv/A9Sq5IC3P2SJUDV11u80x9qe3j5oKj/woEE2HsUI
uCn8HF9NUO7r8G/eg5+8vFcHiQKBgQDLbfnt2xFRFgF06+E2GIK/GAKbkJHhYO1L
2obZQnGj7+irMlt+uaDbHk+jFCHk+q3xd48x+o1aliNwXSDQ3QJfxCSIfcP9xG2W
3l6kCO5irwgt589E5rEUEegQM2F6oS3OtcUr9zjeW1rDMjHHLD87Tb9rB2GAweQf
yLqsoZovLwKBgERjX2GNHdVyWU6qDqUetimSxPityG8+1XYA1JzLrEL7fEGIlgln
2xf7f8dePEze1rv20zDRtiyBWdp75iYfesHc1aLZE2kNb8/EDBlRfT+fIZJZm2Uo
nEn8Uv30e/Xgn9o2kDMyb2l3eqhbo7K0fxeewOt+fvu2CyKaOwn22lUhAoGBAKQz
YfAmykR8EbLxjnhesnJSjBBLUiTsWr3GZuBI7HdZ96Dv5cBVT0xum/NTFcTAvtRQ
IApEZgJ/e51/3jQYoIjyRlbRxPg5rAeB+DxJZTnMdDqxiLDh0H8VsQ4amw0juljG
iZ9iTsnUTV+PTXSp92QD7oUSkRYf6uXo3Rzo2A5LAoGAI4o8zbghCrqJvkuk0hfh
UP5X/eu06EMYVLamZit64kJMdumKcJuVDo4jyz+DYhz6T3bMPbAzRBqsZA4M8l73
XWp9z1bKS/AHsZSXLLRKk1C/OR7z5zLz/j8BgbqKpjNtRCp1DFu1IgPC2+3rVr89
mQa1nuu+xYOkgxcs7F5qM7g=
-----END PRIVATE KEY-----

```

## File: tools\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import demo_utils

```

## File: views\account_journal_dashboard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_journal_dashboard_kanban_view" model="ir.ui.view">
        <field name="name">account.journal.dashboard.kanban</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account.account_journal_dashboard_kanban_view" />
        <field name="arch" type="xml">
            <data>
                <xpath expr="//kanban" position="inside">
                    <field name="is_peppol_journal"/>
                    <field name="account_peppol_proxy_state"/>
                </xpath>

                <xpath expr="//t[@id='account.JournalBodySalePurchase']" position="inside">
                    <t t-if="['sender', 'smp_registration', 'receiver'].includes(record.account_peppol_proxy_state.raw_value)">
                        <t t-if="journal_type == 'sale'">
                            <div class="w-100">
                                <a type="object" name="peppol_get_message_status" groups="account.group_account_invoice">Fetch Peppol invoice status</a>
                            </div>
                            <div class="w-100">
                                <a type="object" name="action_peppol_ready_moves" groups="account.group_account_invoice">Peppol ready invoices</a>
                            </div>
                        </t>
                        <t t-elif="journal_type == 'purchase'">
                            <t t-if="record.is_peppol_journal.raw_value and record.account_peppol_proxy_state.raw_value === 'receiver'">
                                <div class="w-100">
                                    <a type="object" name="peppol_get_new_documents" groups="account.group_account_invoice">Fetch from Peppol</a>
                                </div>
                            </t>
                        </t>
                    </t>
                </xpath>
            </data>
        </field>
    </record>
</odoo>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_peppol_view_move_form" model="ir.ui.view">
        <field name="name">account.peppol.view.move.form</field>
        <field name="model">account.move</field>
        <field name="priority">30</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <header position="inside">
                <button name="action_cancel_peppol_documents"
                        type="object"
                        class="btn btn-secondary"
                        string="Cancel PEPPOL"
                        invisible="peppol_move_state != 'to_send' or state == 'draft'"/>
            </header>

            <xpath expr="//div[@name='journal_div']" position="after">
                <label for="peppol_move_state"
                       invisible="not peppol_move_state or state == 'draft' or move_type in ('in_invoice', 'in_refund')"/>
                <div name="peppol_div"
                     class="d-flex"
                     invisible="not peppol_move_state or state == 'draft' or move_type in ('in_invoice', 'in_refund')">
                    <field name="peppol_move_state" class="oe_inline"/>
                    <span class="mx-1" invisible="'demo_' not in peppol_message_uuid"> (Demo)</span>
                    <span class="text-muted mx-3"
                          invisible="peppol_move_state != 'to_send'">
                        The invoice will be sent automatically via Peppol
                    </span>
                </div>
            </xpath>
        </field>
    </record>

    <record id="account_peppol_view_out_invoice_tree_inherit" model="ir.ui.view">
        <field name="name">account.move.out.invoice.list.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_out_invoice_tree"/>
        <field name="arch" type="xml">
            <field name="status_in_payment" position="before">
                <field name="peppol_move_state" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="account_peppol_view_out_credit_note_tree_inherit" model="ir.ui.view">
        <field name="name">account.move.credit.note.list.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_out_credit_note_tree"/>
        <field name="arch" type="xml">
            <field name="status_in_payment" position="before">
                <field name="peppol_move_state" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="account_peppol_view_account_invoice_filter" model="ir.ui.view">
        <field name="name">account.invoice.select.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//search/group/filter[@name='status']" position="after">
                <filter string="Peppol status" name="peppol_move_state" context="{'group_by': 'peppol_move_state'}"/>
            </xpath>
            <xpath expr="//filter[@name='to_check']" position='after'>
                <separator/>
                <filter name="peppol_ready"
                        string="Peppol Ready"
                        domain="[('state', '=', 'posted'), ('peppol_move_state', '=', 'ready'), ('move_type', 'in', ('out_invoice', 'out_refund', 'out_receipt'))]"/>
                <separator/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\account_portal_templates.xml

```xml
<odoo>
    <template id="portal_my_details_fields" inherit_id="account.portal_my_details_fields">
        <xpath expr="//select[@name='invoice_sending_method']/../.." position="after">
            <div t-if="peppol_eas_list" class="col-xl-6 portal_peppol_toggle">
                <label class="col-form-label" for="peppol_eas">Peppol e-Address (EAS)</label>
                <select name="peppol_eas" t-attf-class="form-select #{error.get('invalid_peppol_config') and 'is-invalid' or ''}">
                    <t t-foreach="peppol_eas_list" t-as="eas">
                        <option t-att-value="eas" t-att-selected="(peppol_eas or partner.peppol_eas) == eas">
                            <t t-esc="eas_value"/>
                        </option>
                    </t>
                </select>
                <label class="col-form-label" for="peppol_endpoint">Peppol Endpoint</label>
                <input type="text"
                       name="peppol_endpoint"
                       t-attf-class="form-control #{error.get('invalid_peppol_endpoint') and 'is-invalid' or ''}"
                       t-att-value="peppol_endpoint or partner.peppol_endpoint"/>
            </div>
        </xpath>
    </template>
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
            <xpath expr="//div[@id='account_peppol_install']" position="replace">
                <div id="account_peppol" class="col-12 o_setting_box">
                    <div class="o_setting_right_pane border-0">
                        <div invisible="account_peppol_proxy_state not in ('sender', 'receiver', 'smp_registration')">
                            Your Peppol ID <field name="account_peppol_edi_identification" class="oe_inline o_form_label" readonly="1"/> <field name="account_peppol_proxy_state" class="oe_inline o_form_label text-lowercase" readonly="1"/> invoices and credit notes.
                        </div>
                        <div invisible="account_peppol_proxy_state != 'rejected'">
                            You registration has been rejected, the reason has been sent to you via email.
                            Please contact our support if you need further assistance.
                        </div>

                        <!-- The participant registered as a sender: display register as a receiver button, email, migration key field -->
                        <div class="row" invisible="account_peppol_proxy_state not in ('sender', 'smp_registration', 'receiver')">
                            <label string="Primary contact email"
                                   for="account_peppol_contact_email"
                                   class="col-lg-3 o_light_label"/>
                            <field name="account_peppol_contact_email" required="account_peppol_proxy_state in ('sender', 'smp_registration', 'receiver')"/>
                        </div>

                        <!-- The participant is waiting to be registered on the SMP -->
                        <div invisible="account_peppol_proxy_state not in ('smp_registration', 'receiver')">
                            <div class="row">
                                <label string="Incoming Invoices Journal"
                                       for="account_peppol_purchase_journal_id"
                                       class="col-lg-3 o_light_label"/>
                                <field name="account_peppol_purchase_journal_id"
                                       required="account_peppol_proxy_state in ('smp_registration', 'receiver')"/>
                            </div>
                            <div invisible="account_peppol_proxy_state != 'smp_registration'">
                                <p class="mt-4">
                                    Your registration should be activated within a day.
                                </p>
                            </div>
                        </div>

                        <!-- Optional: field related to registering as a receiver -->
                        <div invisible="account_peppol_proxy_state != 'sender'" groups="base.group_no_one">
                            <p colspan="2" class="mt-4 mb-2">
                                I want to migrate my existing Peppol connection to Odoo (optional):
                            </p>
                            <field name="account_peppol_migration_key"/>
                        </div>

                        <div class="mt-4"
                             invisible="account_peppol_proxy_state != 'receiver'"
                             groups="base.group_no_one">
                            <div invisible="not account_peppol_migration_key">
                                Your migration key is:
                                <field name="account_peppol_migration_key"
                                       nolabel="1"
                                       readonly="account_peppol_proxy_state == 'receiver' and account_peppol_migration_key"/>
                            </div>
                        </div>


                        <!-- All action buttons -->
                        <div invisible="account_peppol_proxy_state != 'receiver' or account_peppol_edi_mode == 'demo'">
                            <button string="Configure Peppol Services"
                                    name="button_account_peppol_configure_services"
                                    type="object"
                                    class="btn-link mt-3 mb-3"
                                    icon="oi-arrow-right"/>
                        </div>

                        <div class="d-flex gap-1 action_buttons" colspan="3">
                            <div invisible="account_peppol_proxy_state not in ('not_registered', 'in_verification')">
                                <div class="text-muted">
                                    Allow sending and receiving invoices through the PEPPOL network
                                </div>
                                <button name="action_open_peppol_form"
                                        type="object"
                                        string="Activate Electronic Invoicing"
                                        class="oe_highlight mt-2"/>
                            </div>
                            <widget name="peppol_settings_buttons"
                                    invisible="account_peppol_proxy_state not in ('sender', 'smp_registration', 'receiver')"/>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_partner_form_account_peppol" model="ir.ui.view">
        <field name="name">res.partner.form.account.peppol</field>
            <field name="model">res.partner</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
            <data>
                <xpath expr="//field[@name='invoice_sending_method']" position="before">
                    <field name="available_peppol_sending_methods" invisible="1"/>
                    <field name="available_peppol_edi_formats" invisible="1"/>
                </xpath>
                <xpath expr="//field[@name='invoice_sending_method']" position="attributes">
                    <attribute name="widget">filterable_selection</attribute>
                    <attribute name="options">{'whitelist_fname': 'available_peppol_sending_methods'}</attribute>
                </xpath>
                <xpath expr="//field[@name='invoice_edi_format']" position="attributes">
                    <attribute name="widget">filterable_selection</attribute>
                    <attribute name="options">{'whitelist_fname': 'available_peppol_edi_formats'}</attribute>
                </xpath>
                <xpath expr="//field[@name='invoice_edi_format']" position="after">
                    <field name="bank_account_count" invisible="1"/>
                    <div class="alert alert-warning mb-0"
                         colspan="2"
                         role="alert"
                         invisible="country_code != 'BE' or not peppol_endpoint or peppol_eas in (False, '0208')">
                         The recommended identification method for Belgium is your Company Registry Number.
                    </div>
                    <div class="alert alert-warning"
                         colspan="2"
                         role="alert"
                         invisible="peppol_verification_state != 'valid' or country_code">
                         To generate complete electronic invoices, also set a country for this partner.
                    </div>
                </xpath>
                <xpath expr="//field[@name='peppol_endpoint']" position="after">
                    <field name="peppol_verification_state" invisible="1" force_save="1"/>
                    <label for="peppol_verification_state"
                           groups="base.group_no_one"
                           invisible="not peppol_endpoint"/>
                    <div class="row"
                         groups="base.group_no_one"
                         invisible="not peppol_endpoint">
                        <div class="col-6">
                            <field name="peppol_verification_state" readonly="1"/>
                        </div>
                        <div class="col-6 pt-0">
                            <button name="button_account_peppol_check_partner_endpoint"
                                    class="btn btn-secondary"
                                    type="object"
                                    string="Verify"
                                    help="Verify partner's PEPPOL endpoint"/>
                        </div>
                    </div>
                </xpath>
            </data>
        </field>
    </record>

    <record id="partner_action_verify_peppol" model="ir.actions.server">
        <field name="name">Verify Peppol</field>
        <field name="model_id" ref="account_peppol.model_res_partner"/>
        <field name="binding_model_id" ref="base.model_res_partner"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
            for record in records:
                record.button_account_peppol_check_partner_endpoint()
        </field>
    </record>
</odoo>

```

## File: wizard\account_move_send_batch_wizard.py

```python
from odoo import models


class AccountMoveSendBatchWizard(models.TransientModel):
    _inherit = 'account.move.send.batch.wizard'

    def _compute_summary_data(self):
        # EXTENDS 'account' - add checking of partner's validity
        for wizard in self:
            if peppol_moves := wizard.move_ids.filtered(lambda m: wizard._get_default_sending_method(m) == 'peppol'):
                for move in peppol_moves:
                    move.commercial_partner_id.button_account_peppol_check_partner_endpoint(company=move.company_id)
        super()._compute_summary_data()

    def action_send_and_print(self, force_synchronous=False, allow_fallback_pdf=False):
        # EXTENDS 'account'
        self.ensure_one()
        if peppol_moves := self.move_ids.filtered(lambda m: self._get_default_sending_method(m) == 'peppol'):
            if registration_action := self._do_peppol_pre_send(peppol_moves):
                return registration_action
        return super().action_send_and_print(force_synchronous=force_synchronous, allow_fallback_pdf=allow_fallback_pdf)

```

## File: wizard\account_move_send_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, _
from odoo.exceptions import UserError

class AccountMoveSendWizard(models.TransientModel):
    _inherit = 'account.move.send.wizard'

    # -------------------------------------------------------------------------
    # DEFAULTS
    # -------------------------------------------------------------------------

    def _compute_sending_method_checkboxes(self):
        """ EXTENDS 'account'
        If Customer is not valid on Peppol, we disable the checkbox. Also add the proxy mode if not in prod.
        """
        for wizard in self:
            peppol_partner = wizard.move_id.partner_id.commercial_partner_id.with_company(wizard.company_id)
            peppol_partner.button_account_peppol_check_partner_endpoint(company=wizard.company_id)
        super()._compute_sending_method_checkboxes()
        for wizard in self:
            if peppol_checkbox := wizard.sending_method_checkboxes.get('peppol'):
                peppol_partner = wizard.move_id.partner_id.commercial_partner_id.with_company(wizard.company_id)
                peppol_proxy_mode = wizard.company_id._get_peppol_edi_mode()
                if peppol_partner.peppol_verification_state == 'not_valid':
                    addendum_disable_reason = _(' (Customer not on Peppol)')
                elif peppol_partner.peppol_verification_state == 'not_verified':
                    addendum_disable_reason = _(' (no VAT)')
                else:
                    addendum_disable_reason = ''
                vals_not_valid = {'readonly': True, 'checked': False} if addendum_disable_reason else {}
                addendum_mode = ''
                if peppol_proxy_mode == 'test':
                    addendum_mode = _(' (Test)')
                elif peppol_proxy_mode == 'demo':
                    addendum_mode = _(' (Demo)')
                if addendum_disable_reason or addendum_mode:
                    wizard.sending_method_checkboxes = {
                        **wizard.sending_method_checkboxes,
                        'peppol': {
                            **peppol_checkbox,
                            **vals_not_valid,
                            'label': _(
                                '%(peppol_label)s%(disable_reason)s%(peppol_proxy_mode)s',
                                peppol_label=peppol_checkbox['label'],
                                disable_reason=addendum_disable_reason,
                                peppol_proxy_mode=addendum_mode,
                            ),
                        }
                    }

    def action_send_and_print(self, allow_fallback_pdf=False):
        # EXTENDS 'account'
        self.ensure_one()
        if self.sending_methods and 'peppol' in self.sending_methods:
            if self.move_id.partner_id.commercial_partner_id.peppol_verification_state != 'valid':
                raise UserError(_("Partner doesn't have a valid Peppol configuration."))
            if registration_action := self._do_peppol_pre_send(self.move_id):
                return registration_action
        return super().action_send_and_print(allow_fallback_pdf=allow_fallback_pdf)

```

## File: wizard\peppol_registration.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import contextlib
try:
    import phonenumbers
except ImportError:
    phonenumbers = None

from odoo import _, api, fields, models, modules
from odoo.exceptions import UserError, ValidationError

from odoo.addons.account_edi_proxy_client.models.account_edi_proxy_user import AccountEdiProxyError
from odoo.addons.account_peppol.tools.demo_utils import handle_demo


class PeppolRegistration(models.TransientModel):
    _name = 'peppol.registration'
    _description = "Peppol Registration"

    company_id = fields.Many2one(
        comodel_name='res.company',
        required=True,
        default=lambda self: self.env.company,
    )
    contact_email = fields.Char(
        related='company_id.account_peppol_contact_email',
        readonly=False,
        required=True,
    )
    edi_mode = fields.Selection(
        string='EDI mode',
        selection=[('demo', 'Demo'), ('test', 'Test'), ('prod', 'Live')],
        compute='_compute_edi_mode',
        inverse='_inverse_edi_mode',
        readonly=False,
    )
    edi_user_id = fields.Many2one(
        comodel_name='account_edi_proxy_client.user',
        string='EDI user',
        compute='_compute_edi_user_id',
    )
    account_peppol_migration_key = fields.Char(related='company_id.account_peppol_migration_key', readonly=False)  # TODO remove in master
    edi_mode_constraint = fields.Selection(
        selection=[('demo', 'Demo'), ('test', 'Test'), ('prod', 'Live')],
        compute='_compute_edi_mode_constraint',
        help="Using the config params, this field specifies which edi modes may be selected from the UI"
    )
    phone_number = fields.Char(related='company_id.account_peppol_phone_number', readonly=False)
    account_peppol_proxy_state = fields.Selection(related='company_id.account_peppol_proxy_state', readonly=False)
    verification_code = fields.Char(related='edi_user_id.peppol_verification_code', readonly=False)  # TODO remove in master
    peppol_eas = fields.Selection(related='company_id.peppol_eas', readonly=False, required=True)
    peppol_endpoint = fields.Char(related='company_id.peppol_endpoint', readonly=False, required=True)
    peppol_warnings = fields.Json(
        string="Peppol warnings",
        compute="_compute_peppol_warnings",
    )
    smp_registration = fields.Boolean(  # TODO switch to computed non-stored in master
        string='Register as a receiver',
        help="If not check, you will only be able to send invoices but not receive them.",
        compute='_compute_smp_registration',
        store=True,
        readonly=False,
    )

    # -------------------------------------------------------------------------
    # ONCHANGE METHODS
    # -------------------------------------------------------------------------

    @api.onchange('peppol_endpoint')
    def _onchange_peppol_endpoint(self):
        for wizard in self:
            if wizard.peppol_endpoint:
                wizard.peppol_endpoint = ''.join(char for char in wizard.peppol_endpoint if char.isalnum())

    @api.onchange('phone_number')
    def _onchange_phone_number(self):
        for wizard in self:
            if wizard.phone_number:
                wizard.company_id._sanitize_peppol_phone_number(wizard.phone_number)
                with contextlib.suppress(phonenumbers.NumberParseException):
                    parsed_phone_number = phonenumbers.parse(
                        wizard.phone_number,
                        region=self.company_id.country_code,
                    )
                    wizard.phone_number = phonenumbers.format_number(
                        parsed_phone_number,
                        phonenumbers.PhoneNumberFormat.E164,
                    )

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('company_id.account_edi_proxy_client_ids')
    def _compute_edi_user_id(self):
        for wizard in self:
            wizard.edi_user_id = wizard.company_id.account_edi_proxy_client_ids.filtered(lambda u: u.proxy_type == 'peppol')[:1]

    @api.depends('peppol_eas', 'peppol_endpoint', 'smp_registration')
    def _compute_peppol_warnings(self):
        for wizard in self:
            peppol_warnings = {}
            if (
                wizard.peppol_eas
                and wizard.peppol_endpoint
                and not wizard.company_id._check_peppol_endpoint_number(warning=True)
            ):
                peppol_warnings['company_peppol_endpoint_warning'] = {
                    'message': _("The endpoint number might not be correct. "
                                "Please check if you entered the right identification number."),
                }
            if not wizard.smp_registration:
                peppol_warnings['company_on_another_smp'] = {
                    'message': _("Your company is already registered on another Access Point for receiving invoices."
                                 "We will register you as a sender only.")
                }
            wizard.peppol_warnings = peppol_warnings or False

    @api.depends('peppol_eas', 'peppol_endpoint')
    def _compute_smp_registration(self):
        for wizard in self:
            wizard.smp_registration = False
            if wizard.peppol_eas and wizard.peppol_endpoint:
                try:
                    edi_identification = f'{wizard.peppol_eas}:{wizard.peppol_endpoint}'
                    wizard.edi_user_id._check_company_on_peppol(wizard.company_id, edi_identification)
                    wizard.smp_registration = True
                except UserError:
                    pass

    @api.depends('edi_user_id')
    def _compute_edi_mode_constraint(self):
        mode_constraint = self.env['ir.config_parameter'].sudo().get_param('account_peppol.mode_constraint')
        trial_param = self.env['ir.config_parameter'].sudo().get_param('saas_trial.confirm_token')
        self.edi_mode_constraint = trial_param and 'demo' or mode_constraint or 'prod'

    @api.depends('edi_user_id')
    def _compute_edi_mode(self):
        edi_mode = self.env['ir.config_parameter'].sudo().get_param('account_peppol.edi.mode')
        for wizard in self:
            if wizard.edi_user_id:
                wizard.edi_mode = wizard.edi_user_id.edi_mode
            else:
                wizard.edi_mode = edi_mode or 'prod'

    def _inverse_edi_mode(self):
        for wizard in self:
            if not wizard.edi_user_id and wizard.edi_mode:
                self.env['ir.config_parameter'].sudo().set_param('account_peppol.edi.mode', wizard.edi_mode)
                return

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    def _ensure_mandatory_fields(self):
        if not self.contact_email or not self.phone_number:
            raise ValidationError(_("Contact email and phone number are required."))

    def _action_open_peppol_form(self, reopen=True):
        action_dict = {
            'name': _("Activate Electronic Invoicing (via Peppol)"),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'peppol.registration',
            'target': 'new',
            'context': {
                'dialog_size': 'medium',
                **self.env.context,
            },
        }

        if reopen:
            action_dict.update({
                'res_id': self.id,
                'context': {**self.env.context, 'disable_sms_verification': True},
            })
        return action_dict

    def _action_send_notification(self, title, message):
        move_ids = self.env.context.get('active_ids')
        if move_ids and self.env.context.get('active_model') == 'account.move':
            next_action = self.env['account.move'].browse(move_ids).action_send_and_print()
            next_action['views'] = [(False, 'form')]
        else:
            next_action = {'type': 'ir.actions.act_window_close'}

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'title': title,
                'type': 'success',
                'message': message,
                'next': next_action,
            }
        }

    @handle_demo
    def button_peppol_sender_registration(self):
        """ TODO remove in master
        The first step of the Peppol onboarding.
        - Creates an EDI proxy user on the iap side, then the client side
        - Calls /activate_participant to mark the EDI user as peppol user
        - Allows the user to become a sender but not a receiver on the Peppol network.
        Basically, a Sender does not exist on the peppol network. They use our
        Access Point to send invoices to Peppol participants without having to register
        themselves.
        """
        self.button_register_peppol_participant()

    @handle_demo
    def button_peppol_smp_registration(self):
        """ TODO remove in master
        The second (optional) step in Peppol registration.
        The user can choose to become a Receiver and officially register on the Peppol
        network, i.e. receive documents from other Peppol participants.
        """
        self.button_register_peppol_participant()

    @handle_demo
    def button_update_peppol_user_data(self):
        """
        Action for the user to be able to update their contact details any time
        Calls /update_user on the iap server
        """
        self.ensure_one()
        self._ensure_mandatory_fields()

        edi_identification = f'{self.peppol_eas}:{self.peppol_endpoint}'
        if self.smp_registration:
            self.edi_user_id._check_company_on_peppol(self.company_id, edi_identification)

        params = {
            'update_data': {
                'edi_identification': edi_identification,
                'peppol_phone_number': self.phone_number,
                'peppol_contact_email': self.contact_email,
            }
        }

        self.edi_user_id._call_peppol_proxy(
            endpoint='/api/peppol/1/update_user',
            params=params,
        )
        return True

    def button_send_peppol_verification_code(self):
        """ TODO remove in master
        Request user verification via SMS
        Calls the /send_verification_code to send the 6-digit verification code
        """
        pass

    @handle_demo
    def button_check_peppol_verification_code(self):
        """ TODO remove in master
        Calls /verify_phone_number to compare user's input and the
        code generated on the IAP server
        """
        pass

    @handle_demo
    def button_register_peppol_participant(self):
        self.ensure_one()
        self._ensure_mandatory_fields()

        if self.account_peppol_proxy_state in ('smp_registration', 'receiver', 'rejected'):
            raise UserError(
                _('Cannot register a user with a %s application', self.account_peppol_proxy_state))

        edi_user = self.edi_user_id or self.env['account_edi_proxy_client.user']._register_proxy_user(self.company_id, 'peppol', self.edi_mode)

        # if there is an error when activating the participant below,
        # the client side is rolled back and the edi user is deleted on the client side
        # but remains on the proxy side.
        # it is important to keep these two in sync, so commit before activating.
        if not modules.module.current_test:
            self.env.cr.commit()

        edi_user._peppol_register_sender()

        if self.smp_registration:
            try:
                edi_user._peppol_register_sender_as_receiver()
            except (UserError, AccountEdiProxyError) as e:
                self.button_deregister_peppol_participant()
                raise

        # success
        notifications = {
            'sender': {
                'message': _('You can now send electronic invoices via Peppol.'),
            },
            'smp_registration': {  # TODO remove in master
                'message': _('Your Peppol registration will be activated soon. You can already send invoices.'),
            },
            'receiver': {
                'message': _('You can now send and receive electronic invoices via Peppol'),
            },
        }
        state = self.company_id.account_peppol_proxy_state
        return self._action_send_notification(
            title=None,
            message=notifications[state]['message'],
        )

    @handle_demo
    def button_deregister_peppol_participant(self):
        """
        Deregister the edi user from Peppol network
        """
        self.ensure_one()

        if self.edi_user_id:
            self.edi_user_id._peppol_deregister_participant()
        return True

```

## File: wizard\peppol_registration_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="peppol_registration_form" model="ir.ui.view">
        <field name="name">peppol.registration.form</field>
        <field name="model">peppol.registration</field>
        <field name="groups_id" eval="[Command.link(ref('account.group_account_manager'))]"/>
        <field name="arch" type="xml">
            <form class="form_peppol_registration">
                <field name="company_id" invisible="1"/>
                <sheet>
                    <div class="m-0" name="warnings" invisible="not peppol_warnings or account_peppol_proxy_state != 'not_registered'">
                        <field name="peppol_warnings" class="o_field_html" widget="actionable_errors"/>
                    </div>

                    <div name="peppol_registration" invisible="account_peppol_proxy_state not in ('not_registered', 'in_verification')">
                        <p class="text-muted">Send electronic invoices, and receive bills automatically via Peppol</p>

                            <!-- Optional step: receiver registration fields -->

                        <group col="1">
                            <group>
                                <field name="smp_registration" string="Allow incoming invoices" invisible="1"/>
                                <field name="peppol_eas"
                                       placeholder="Peppol ID"
                                       nolabel="1"
                                       class="o_field_peppol_eas_selection"/>
                                <field name="peppol_endpoint" nolabel="1" placeholder="Your endpoint"/>
                                <field name="contact_email" string="Email"/>
                                <field name="phone_number" required="edi_mode not in ('demo', 'test')" string="Phone"/>
                            </group>

                            <!-- edi mode info -->
                            <field name="edi_mode_constraint" invisible="1"/>
                            <group invisible="account_peppol_proxy_state != 'not_registered'">
                                <field name="edi_mode" invisible="1"/>
                                <div class="text-muted col" invisible="edi_mode != 'prod'">
                                    By clicking the button below I accept that Odoo may process my e-invoices.
                                </div>
                                <div class="text-muted col" invisible="edi_mode != 'test'">
                                    Test mode allows sending e-invoices through the test Peppol network.
                                    By clicking the button below I accept that Odoo may process my e-invoices.
                                </div>
                            </group>
                        </group>
                    </div>
                </sheet>
                <footer>
                    <widget name="peppol_settings_buttons"
                            invisible="account_peppol_proxy_state in ('rejected', False)"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\service_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from markupsafe import Markup

from odoo import api, fields, models, _

_logger = logging.getLogger(__name__)


class PeppolService(models.TransientModel):
    _name = 'account_peppol.service'
    _order = 'document_name, id'
    _description = 'Peppol Service'

    wizard_id = fields.Many2one(comodel_name='account_peppol.service.wizard')
    document_identifier = fields.Char()
    document_name = fields.Char()
    enabled = fields.Boolean()


class PeppolServiceConfig(models.TransientModel):
    _name = 'account_peppol.service.wizard'
    _description = 'Peppol Services Wizard'

    edi_user_id = fields.Many2one(comodel_name='account_edi_proxy_client.user', string='EDI user')
    service_json = fields.Json(help="JSON representation of peppol services as retrieved from the peppol server.")
    service_info = fields.Html(compute='_compute_service_info')
    service_ids = fields.One2many(
        comodel_name='account_peppol.service',
        inverse_name='wizard_id',
        readonly=False,
    )

    # -------------------------------------------------------------------------
    # COMPUTES
    # -------------------------------------------------------------------------

    def _compute_service_info(self):
        for wizard in self:
            message = ''
            supported_doctypes = self.env['res.company']._peppol_supported_document_types()
            if (non_configurable := [
                identifier
                for identifier in (wizard.service_json or {})
                if identifier not in supported_doctypes
            ]):
                message = Markup('%s<ul>%s</ul>') % (
                    _(
                        "The following services are listed on your participant but cannot be configured here. "
                        "If you wish to configure them differently, please contact support."
                    ),
                    Markup().join(
                        Markup('<li>%s</li>') % (wizard.service_json[identifier]['document_name'])
                        for identifier in non_configurable
                    ),
                )
            wizard.service_info = message

    # -------------------------------------------------------------------------
    # OVERRIDES
    # -------------------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        """Get the selectable document types.

        Synthesize a combination of locally available document types and those added to the user on
        the IAP, add the relevant services.
        """
        wizards = super().create(vals_list)
        supported_doctypes = self.env['res.company']._peppol_supported_document_types()
        for wizard in wizards:
            wizard.service_ids.create([
                {
                    'document_identifier': identifier,
                    'document_name': document_name,
                    'enabled': identifier in (wizard.service_json or {}),
                    'wizard_id': wizard.id,
                }
                for identifier, document_name in supported_doctypes.items()
            ])
        return wizards

    def confirm(self):
        """Interpret changes to the services, and add or remove them on the IAP accordingly."""
        services = self.service_ids.read(['document_identifier', 'enabled'])
        service_json = self.service_json or {}
        to_add, to_remove = [], []

        for service in services:
            if service['document_identifier'] in service_json and not service['enabled']:
                to_remove.append(service['document_identifier'])

            if service['document_identifier'] not in service_json and service['enabled']:
                to_add.append(service['document_identifier'])

        if to_add:
            self.edi_user_id._call_peppol_proxy(
                "/api/peppol/2/add_services", {
                    'document_identifiers': to_add,
                },
            )
        if to_remove:
            self.edi_user_id._call_peppol_proxy(
                "/api/peppol/2/remove_services", {
                    'document_identifiers': to_remove,
                },
            )

```

## File: wizard\service_wizard.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="peppol_service_configuration">
            <field name="name">account.peppol.service.configuration.form</field>
            <field name="model">account_peppol.service.wizard</field>
            <field name="arch" type="xml">
                <form string="">
                    <group>
                        <div colspan="2">
                            <field name="service_info" role="status" class="o_field_html alert alert-info" invisible="not service_info"/>
                            <field name="service_ids"
                                   nolabel="1"
                                   widget="one2many"
                                   columns="2">
                                <list editable="bottom" create="false" delete="false" edit="true">
                                    <field name="wizard_id" column_invisible="1"/>
                                    <field name="document_name" readonly="1"/>
                                    <field name="document_identifier" groups="base.group_no_one" readonly="1"/>
                                    <field name="enabled" widget="boolean_toggle"/>
                                </list>
                            </field>
                        </div>
                        <footer>
                            <button name="confirm" string="Confirm" type="object" class="btn-primary" data-hotkey="q"/>
                            <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x" />
                        </footer>
                    </group>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_send_wizard
from . import account_move_send_batch_wizard
from . import peppol_registration
from . import service_wizard

```

