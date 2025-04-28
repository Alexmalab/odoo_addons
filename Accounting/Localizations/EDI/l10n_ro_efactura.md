# Odoo Module: l10n_ro_efactura

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'author': 'Odoo',
    'name': 'Romania - Send E-Factura',
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'summary': "Bridge module for sending Romanian E-Factura to the SPV",
    'countries': ['ro'],
    'depends': ['l10n_ro_edi'],
    'data': [
        'data/ir_cron.xml',
        'security/ir.model.access.csv',
        'views/account_move_views.xml',
        'views/res_config_settings_views.xml',
        'wizard/account_move_send_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
import binascii
import requests

from werkzeug.urls import url_encode, url_join

from odoo import _, http
from odoo.exceptions import UserError, ValidationError
from odoo.http import request


URL_ANAF_AUTHORIZE = 'https://logincert.anaf.ro/anaf-oauth2/v1/authorize'
URL_ANAF_TOKEN = 'https://logincert.anaf.ro/anaf-oauth2/v1/token'


class L10nRoEdiController(http.Controller):

    @http.route('/l10n_ro_edi/authorize/<int:company_id>', auth="user")
    def authorize(self, company_id, **kw):
        """ Generate Authorization Token to acquire access_key for requesting Access Token """
        company = http.request.env['res.company'].browse(company_id)
        if not company.l10n_ro_edi_client_id or not company.l10n_ro_edi_client_secret:
            raise UserError(_("Client ID and Client Secret field must be filled."))

        auth_url_params = url_encode({
            'response_type': 'code',
            'client_id': company.l10n_ro_edi_client_id,
            'redirect_uri': company.l10n_ro_edi_callback_url,
            'token_content_type': 'jwt',
        })
        auth_url = f'{URL_ANAF_AUTHORIZE}?{auth_url_params}'
        return request.redirect(auth_url, code=302, local=False)

    @http.route('/l10n_ro_edi/callback/<int:company_id>', type='http', auth="user")
    def callback(self, company_id, **kw):
        """ Use the acquired access_key to request access & refresh token from ANAF """
        company = http.request.env['res.company'].browse(company_id)
        access_key = kw.get('code')

        def log_and_raise_error(message: str):
            message += '\n' + _("Received access key: %s", access_key)
            company._l10n_ro_edi_log_message(message, 'callback')
            raise UserError(message)

        # Without certificate, ANAF won't give any access key in the callback URL's "code" parameter
        if not access_key:
            log_and_raise_error(_("Access key not found. Please try again.\nResponse: %s", kw))

        try:
            response = requests.post(
                url=URL_ANAF_TOKEN,
                data={
                    'grant_type': 'authorization_code',
                    'client_id': company.l10n_ro_edi_client_id,
                    'client_secret': company.l10n_ro_edi_client_secret,
                    'code': access_key,
                    'access_key': access_key,
                    'redirect_uri': company.l10n_ro_edi_callback_url,
                    'token_content_type': 'jwt',
                },
                headers={
                    'accept': 'application/json',
                    'user-agent': 'Odoo (http://www.odoo.com/contactus)',
                },
                timeout=10,
            )
            response.raise_for_status()
        except requests.exceptions.RequestException as e:
            log_and_raise_error(f"Request to {URL_ANAF_TOKEN} failed: {e}")

        response_to_log = _("Response (code=%(status_code)s) to %(url)s failed:\n%(text)s",
                            status_code=response.status_code,
                            url=response.url,
                            text=response.text)
        try:
            response_json = response.json()
        except requests.exceptions.RequestException as e:
            error_cause = _("Error when converting response to json: %s", e)
            log_and_raise_error(f"{error_cause}\n{response_to_log}")

        try:
            company._l10n_ro_edi_process_token_response(response_json)
        except ValidationError as e:
            log_and_raise_error(f"{e}\n{response_to_log}")
        except binascii.Error as e:
            error_cause = _("Error when decoding the access token payload: %s", e)
            log_and_raise_error(f"{error_cause}\n{response_to_log}")
        except Exception as e:
            error_cause = _("Error when processing the response: %s", e)
            log_and_raise_error(f"{error_cause}\n{response_to_log}")

        return request.redirect(url_join(request.httprequest.url_root, 'web'))

```

## File: controllers\__init__.py

```python
from . import main

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <record id="ir_cron_l10n_ro_edi_refresh_access_token" model="ir.cron">
            <field name="name">E-Factura: Refresh Access Token</field>
            <field name="model_id" ref="account.model_account_move"/>
            <field name="state">code</field>
            <field name="code">env['res.company']._cron_l10n_ro_edi_refresh_access_token()</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="interval_number">30</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False"/>
            <field name="nextcall" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 22:00:00')"/>
        </record>
    </data>
</odoo>
```

## File: data\neutralize.sql

```sql
UPDATE res_company
    SET l10n_ro_edi_test_env = true,
        l10n_ro_edi_client_id = NULL,
        l10n_ro_edi_client_secret = NULL,
        l10n_ro_edi_access_token = NULL,
        l10n_ro_edi_refresh_token = NULL;


```

## File: models\account_move.py

```python
import requests

from odoo import models, fields, _, api, modules, tools


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_ro_edi_document_ids = fields.One2many(
        comodel_name='l10n_ro_edi.document',
        inverse_name='invoice_id',
    )
    l10n_ro_edi_state = fields.Selection(
        selection=[
            ('invoice_sending', 'Sent'),
            ('invoice_sent', 'Validated'),
        ],
        string='E-Factura Status',
        compute='_compute_l10n_ro_edi_state',
        store=True,
        help="""- Sent: Successfully sent to the SPV, waiting for validation
                - Validated: Sent & validated by the SPV
                - Error: Sending error or validation error from the SPV""",
    )
    l10n_ro_edi_attachment_id = fields.Many2one(comodel_name='ir.attachment')

    ################################################################################
    # Compute Methods
    ################################################################################

    @api.depends('l10n_ro_edi_document_ids')
    def _compute_l10n_ro_edi_state(self):
        self.l10n_ro_edi_state = False
        for move in self:
            for document in move.l10n_ro_edi_document_ids.sorted():
                if document.state in ('invoice_sending', 'invoice_sent'):
                    move.l10n_ro_edi_state = document.state
                    break

    @api.depends('l10n_ro_edi_state')
    def _compute_show_reset_to_draft_button(self):
        """ Prevent user to reset move to draft when there's an
            active sending document or a successful response has been received """
        # EXTENDS 'account'
        super()._compute_show_reset_to_draft_button()
        for move in self:
            if move.l10n_ro_edi_state in ('invoice_sending', 'invoice_sent'):
                move.show_reset_to_draft_button = False

    ################################################################################
    # Romanian Document Shorthands & Helpers
    ################################################################################

    def _l10n_ro_edi_create_attachment_values(self, raw, res_model=None, res_id=None):
        """ Shorthand for creating the attachment_id values on the invoice's document """
        self.ensure_one()
        res_model = res_model or self._name
        res_id = res_id or self.id
        return {
            'name': f"ciusro_signature_{self.name.replace('/', '_')}.xml",
            'res_model': res_model,
            'res_id': res_id,
            'raw': raw,
            'type': 'binary',
            'mimetype': 'application/xml',
        }

    def _l10n_ro_edi_create_document_invoice_sending(self, key_loading, attachment_raw):
        # TODO in master: use 1 dictionary "values" as the parameter
        """ Shorthand for creating a ``l10n_ro_edi.document`` of state ``invoice_sending``.

        :param key_loading: string of the e-factura index
        :param attachment_raw: bytes, from xml_data
        :return: ``l10n_ro_edi.document`` object """
        self.ensure_one()
        document = self.env['l10n_ro_edi.document'].sudo().create({
            'invoice_id': self.id,
            'state': 'invoice_sending',
            'key_loading': key_loading,
        })
        attachment_values = self._l10n_ro_edi_create_attachment_values(
            raw=attachment_raw,
            res_model=document._name,
            res_id=document.id,
        )
        document.attachment_id = self.env['ir.attachment'].sudo().create(attachment_values)
        return document

    def _l10n_ro_edi_create_document_invoice_sending_failed(self, message, attachment_raw=None, key_loading=None):
        # TODO in master: use 1 dictionary "values" as the parameter
        """ Shorthand for creating a ``l10n_ro_edi.document`` of state ``invoice_sending_failed``.
        The ``attachment_raw`` and ``key_loading`` dictionary values is optional in case the error is from pre_send.

        :param message: string of the error message
        :param attachment_raw: <optional> bytes from xml_data
        :param key_loading: <optional> string of the e-factura index
        :return: ``l10n_ro_edi.document`` object """
        self.ensure_one()
        document = self.env['l10n_ro_edi.document'].sudo().create({
            'invoice_id': self.id,
            'state': 'invoice_sending_failed',
            'message': message,
        })
        if key_loading:
            document.key_loading = key_loading
        if attachment_raw:
            attachment_values = self._l10n_ro_edi_create_attachment_values(
                raw=attachment_raw,
                res_model=document._name,
                res_id=document.id,
            )
            document.attachment_id = self.env['ir.attachment'].sudo().create(attachment_values)
        return document

    def _l10n_ro_edi_create_document_invoice_sent(self, values: dict):
        """ Shorthand for creating a ``l10n_ro_edi.document`` of state `invoice_sent`.
        The created attachment are saved on both the document and on the invoice.

        :param values: dictionary containing 'key_loading', 'key_signature', 'key_certificate', and 'attachment_raw'
        :return: ``l10n_ro_edi.document`` object """
        self.ensure_one()
        document = self.env['l10n_ro_edi.document'].sudo().create({
            'invoice_id': self.id,
            'state': 'invoice_sent',
            'key_loading': values['key_loading'],
            'key_signature': values['key_signature'],
            'key_certificate': values['key_certificate'],
        })
        attachment = self.env['ir.attachment'].sudo().create(self._l10n_ro_edi_create_attachment_values(values['attachment_raw']))
        document.attachment_id = self.l10n_ro_edi_attachment_id = attachment
        return document

    def _l10n_ro_edi_get_attachment_file_name(self):
        """ Returns the signature file attachment's name from ``l10n_ro_edi.document``/``invoice_sent`` """
        self.ensure_one()
        return f"ciusro_{self.name.replace('/', '_')}.xml"

    def _l10n_ro_edi_get_failed_documents(self):
        """ Shorthand for getting all l10n_ro_edi.document in invoice_sending_failed state """
        self.ensure_one()
        return self.l10n_ro_edi_document_ids.filtered(lambda d: d.state == 'invoice_sending_failed')

    def _l10n_ro_edi_get_sending_and_failed_documents(self):
        """ Shorthand for getting all l10n_ro_edi.document in invoice_sending and invoice_sending_failed state """
        self.ensure_one()
        return self.l10n_ro_edi_document_ids.filtered(lambda d: d.state in ('invoice_sending', 'invoice_sending_failed'))

    ################################################################################
    # Send Logics
    ################################################################################

    def _l10n_ro_edi_get_pre_send_errors(self, xml_data='', assert_xml=False):
        """ Compute all possible common errors before sending the XML to the SPV """
        self.ensure_one()
        errors = []
        if not self.company_id.l10n_ro_edi_access_token:
            errors.append(_('Romanian access token not found. Please generate or fill it in the settings.'))
        if not xml_data and assert_xml:
            errors.append(_('CIUS-RO XML attachment not found.'))
        return errors

    def _l10n_ro_edi_send_invoice(self, xml_data):
        """
        This method send xml_data to the Romanian SPV using the single invoice's (self) data.
        The invoice's company and move_type will be used to calculate the required params in the send request.
        The state of the document deletion/creation are as follows:

         - Pre-check any errors from the invoice's pre_send check before sending

            - if error -> delete all error documents, create a new error document
            - else -> continue to the next step

         - Send to E-Factura, and based on the result:

            - if error -> delete all error documents, create a new error document
            - if success -> delete all error & sending documents, create a new sending document

        :param xml_data: string of the xml data to be sent
        """
        self.ensure_one()
        if errors := self._l10n_ro_edi_get_pre_send_errors(xml_data, True):
            self._l10n_ro_edi_get_failed_documents().unlink()
            self._l10n_ro_edi_create_document_invoice_sending_failed(message='\n'.join(errors))
            return

        self.env['res.company']._with_locked_records(self)
        result = self.env['l10n_ro_edi.document']\
                     .with_context(is_b2b=self.partner_id.commercial_partner_id.is_company)\
                     ._request_ciusro_send_invoice(
            company=self.company_id,
            xml_data=xml_data,
            move_type=self.move_type,
        )
        result['attachment_raw'] = xml_data
        if 'error' in result:  # result == {'error': <str>, 'attachment_raw': <bytes>}
            self._l10n_ro_edi_get_failed_documents().unlink()
            self._l10n_ro_edi_create_document_invoice_sending_failed(
                message=result['error'],
                attachment_raw=result['attachment_raw'],
            )
            self.message_post(body=_("Error when trying to send the E-Factura to the SPV: %s",
                                     result['error']))
        else:  # result == {'key_loading': <str>, 'attachment_raw': <bytes>}; initial sending successful
            self._l10n_ro_edi_get_sending_and_failed_documents().unlink()
            self._l10n_ro_edi_create_document_invoice_sending(result['key_loading'], result['attachment_raw'])
            self.message_post(body=_("E-Factura has been sent and is now being validated by the SPV with index key: %s",
                                     result['key_loading']))

    def _l10n_ro_edi_fetch_invoice_sending_documents(self):
        """
        This method loops over all invoice with sending document in `self`. For each of them,
        it pre-checks error and make a fetch request for the invoice. Based on the answer, it will then:

         - if no answer is received, it will do nothing on the selected invoice
         - if error -> delete all errors, create a new error document
         - else (receives `key_download`) -> immediately make a download request and process it:

            - if error -> delete all sending and error documents, create a new error document
            - if success -> delete all sending and error documents, create success document
        """
        session = requests.Session()
        to_delete_documents = self.env['l10n_ro_edi.document']
        invoices_to_fetch = self.filtered(lambda inv: inv.l10n_ro_edi_state == 'invoice_sending')

        for invoice in invoices_to_fetch:
            if errors := invoice._l10n_ro_edi_get_pre_send_errors():
                to_delete_documents |= invoice._l10n_ro_edi_get_failed_documents()
                invoice._l10n_ro_edi_create_document_invoice_sending_failed(message='\n'.join(errors))
                continue

            active_sending_document = invoice.l10n_ro_edi_document_ids.filtered(lambda d: d.state == 'invoice_sending')[0]
            previous_raw = active_sending_document.attachment_id.sudo().raw
            self.env['res.company']._with_locked_records(invoices_to_fetch)
            result = self.env['l10n_ro_edi.document']._request_ciusro_fetch_status(
                company=invoice.company_id,
                key_loading=active_sending_document.key_loading,
                session=session,
            )

            if result == {}:  # SPV is still processing the XML (no answer yet); do nothing
                continue
            elif 'error' in result:  # Fetch error / SPV finished validating the XML and sends back a disapproval answer
                to_delete_documents |= invoice._l10n_ro_edi_get_sending_and_failed_documents()
                result['key_loading'] = active_sending_document.key_loading
                result['attachment_raw'] = previous_raw
                invoice._l10n_ro_edi_create_document_invoice_sending_failed(
                    message=result['error'],
                    attachment_raw=result['attachment_raw'],
                    key_loading=result['key_loading'],
                )
                invoice.message_post(body=_("Error when trying to fetch the E-Factura from the SPV: %s",
                                            result['error']))
            else:  # result == {'key_download': <str>}; SPV finished validation and sends us an approval answer
                # use the obtained key_download to immediately make a download request and process them
                final_result = self.env['l10n_ro_edi.document']._request_ciusro_download_answer(
                    company=invoice.company_id,
                    key_download=result['key_download'],
                    session=session,
                    status=result['state_status'],
                )
                to_delete_documents |= invoice._l10n_ro_edi_get_sending_and_failed_documents()
                final_result['key_loading'] = active_sending_document.key_loading
                if final_result.get('error'):
                    final_error_message = final_result['error'].replace('\t', '')
                    final_result['attachment_raw'] = previous_raw
                    invoice._l10n_ro_edi_create_document_invoice_sending_failed(
                        message=final_error_message,
                        attachment_raw=final_result['attachment_raw'],
                        key_loading=final_result['key_loading'],
                    )
                    invoice.message_post(body=_("Error when trying to download the E-Factura answer from the SPV: %s",
                                                final_error_message))
                else:
                    invoice._l10n_ro_edi_create_document_invoice_sent(final_result)

            if not tools.config['test_enable'] and not modules.module.current_test:
                self._cr.commit()

        # Delete outdated documents in batches
        to_delete_documents.unlink()

```

## File: models\ciusro_document.py

```python
import io
import requests
import zipfile

from lxml import etree
from odoo import models, fields, api, _

NS_UPLOAD = {"ns": "mfp:anaf:dgti:spv:respUploadFisier:v1"}
NS_STATUS = {"ns": "mfp:anaf:dgti:efactura:stareMesajFactura:v1"}
NS_HEADER = {"ns": "mfp:anaf:dgti:efactura:mesajEroriFactuta:v1"}
NS_SIGNATURE = {"ns": "http://www.w3.org/2000/09/xmldsig#"}


def make_efactura_request(session, company, endpoint, method, params, data=None) -> dict[str, str | bytes]:
    """
    Make an API request to the Romanian SPV, handle the response, and return a ``result`` dictionary.

    :param session: ``requests`` or ``requests.Session()`` object
    :param company: ``res.company`` object containing l10n_ro_edi_test_env, l10n_ro_edi_access_token
    :param endpoint: ``upload`` (for sending) | ``stareMesaj`` (for fetching status) | ``descarcare`` (for downloading answer)
    :param method: ``post`` (for `upload`) | ``get`` (for `stareMesaj` | `descarcare`)
    :param params: Dictionary of query parameters
    :param data: XML data for ``upload`` request
    :return: Dictionary of {'error': <str>} or {'content': <response.content>} from E-Factura
    """
    send_mode = 'test' if company.l10n_ro_edi_test_env else 'prod'
    url = f"https://api.anaf.ro/{send_mode}/FCTEL/rest/{endpoint}"
    headers = {'Content-Type': 'application/xml',
               'Authorization': f'Bearer {company.l10n_ro_edi_access_token}'}

    try:
        response = session.request(method=method, url=url, params=params, data=data, headers=headers, timeout=60)
    except requests.HTTPError as e:
        return {'error': str(e)}
    if response.status_code == 204:
        return {'error': _('You reached the limit of requests. Please try again later.')}
    if response.status_code == 400:
        error_json = response.json()
        return {'error': error_json['message']}
    if response.status_code == 401:
        return {'error': _('Access token is unauthorized.')}
    if response.status_code == 403:
        return {'error': _('Access token is forbidden.')}
    if response.status_code == 500:
        return {'error': _('There is something wrong with the SPV. Please try again later.')}

    return {'content': response.content}


class L10nRoEdiDocument(models.Model):
    _name = 'l10n_ro_edi.document'
    _description = "Document object for tracking CIUS-RO XML sent to E-Factura"
    _order = 'datetime DESC, id DESC'

    invoice_id = fields.Many2one(comodel_name='account.move', required=True)
    state = fields.Selection(
        selection=[
            ('invoice_sending', 'Sent'),
            ('invoice_sending_failed', 'Error'),
            ('invoice_sent', 'Validated'),
        ],
        string='E-Factura Status',
        required=True,
        help="""Sent -> Successfully sent to the SPV, waiting for validation.
                Validated -> Sent & validated by the SPV.
                Error -> Sending error or validation error from the SPV.""",
    )
    datetime = fields.Datetime(default=fields.Datetime.now, required=True)
    attachment_id = fields.Many2one(comodel_name='ir.attachment')
    message = fields.Char()
    key_loading = fields.Char(string="E-Factura Index")  # To be used to fetch the status of previously sent XML
    key_signature = fields.Char()    # Received from a successful response: to be saved for government purposes
    key_certificate = fields.Char()  # Received from a successful response: to be saved for government purposes

    @api.model
    def _request_ciusro_send_invoice(self, company, xml_data, move_type='out_invoice'):
        """
        This method makes an 'upload' request to send xml_data to Romanian SPV.Based on the result, it will then process
        the answer and return a dictionary, which may consist of either an 'error' or a 'key_loading' string.

        :param company: ``res.company`` object
        :param xml_data: String of XML data to be sent
        :param move_type: ``move_type`` field from ``account.move`` object, used for the request parameter
        :return: Result dictionary -> {'error': <str>} | {'key_loading': <str>}
        """
        result = make_efactura_request(
            session=requests,
            company=company,
            endpoint='upload' if self.env.context.get('is_b2b') else 'uploadb2c',  # TODO: change the context value into a method parameter in master 
            method='POST',
            params={'standard': 'UBL' if move_type == 'out_invoice' else 'CN',
                    'cif': company.vat.replace('RO', '')},
            data=xml_data,
        )
        if 'error' in result:
            return result

        root = etree.fromstring(result['content'])
        res_status = root.get('ExecutionStatus')
        if res_status == '1':
            error_elements = root.findall('.//ns:Errors', namespaces=NS_UPLOAD)
            error_messages = [error_element.get('errorMessage') for error_element in error_elements]
            return {'error': '\n'.join(error_messages)}
        else:
            return {'key_loading': root.get('index_incarcare')}

    @api.model
    def _request_ciusro_fetch_status(self, company, key_loading, session):
        """
        This method makes a "Fetch Status" (GET/stareMesaj) request to the Romanian SPV. After processing the response,
        it will return one of the following three possible objects:

        - {'error': <str>} ~ failing response from a bad request
        - {'key_download': <str>} ~ The response was successful, and we can use this key to download the answer
        - {} ~ (empty dict) The response was successful but the SPV haven't finished processing the XML yet.

        :param company: ``res.company`` object
        :param key_loading: Content of ``key_loading`` received from ``_request_ciusro_send_invoice``
        :param session: ``requests.Session()`` object
        :return: {'error': <str>} | {'key_download': <str>} | {}
        """
        result = make_efactura_request(
            session=session,
            company=company,
            endpoint='stareMesaj',
            method='GET',
            params={'id_incarcare': key_loading},
        )
        if 'error' in result:
            return result

        root = etree.fromstring(result['content'])
        error_elements = root.findall('.//ns:Errors', namespaces=NS_STATUS)
        if error_elements:
            return {'error': '\n'.join(error_element.get('errorMessage') for error_element in error_elements)}

        state_status = root.get('stare')
        if state_status in ('nok', 'ok'):
            return {'key_download': root.get('id_descarcare'), 'state_status': state_status}
        else:
            return {}

    @api.model
    def _request_ciusro_download_answer(self, company, key_download, session, status=None):
        """
        This method makes a "Download Answer" (GET/descarcare) request to the Romanian SPV. It then processes the
        response by opening the received zip file and returns either:

        - {'error': <str>} ~ failing response from a bad request / unaccepted XML answer from the SPV
        - <successful response dictionary> ~ contains the necessary information to be stored from the SPV

        :param company: ``res.company`` object
        :param key_download: Content of `key_download` received from `_request_ciusro_send_invoice`
        :param session: ``requests.Session()`` object
        :return: {'attachment_raw': <str>, 'key_signature': <str>, 'key_certificate': <str>, 'error': <str>}
        """
        result = make_efactura_request(
            session=session,
            company=company,
            endpoint='descarcare',
            method='GET',
            params={'id': key_download},
        )
        if 'error' in result:
            return result

        # E-Factura gives download response in ZIP format
        zip_ref = zipfile.ZipFile(io.BytesIO(result['content']))
        # The ZIP will contain two files, one with the original invoice or with the identified errors (as the case may be)
        # and the other with the electronic signature (containing 'semnatura').
        # If there is an error (status == 'nok') we want to provide the file with the errors.
        if status == 'nok':
            signature_file = next(file for file in zip_ref.namelist() if 'semnatura' not in file)
        else:
            signature_file = next(file for file in zip_ref.namelist() if 'semnatura' in file)
        xml_bytes = zip_ref.open(signature_file)
        root = etree.parse(xml_bytes)
        error_elements = root.findall('.//ns:Error', namespaces=NS_HEADER)
        if error_elements:
            error_message = ('\n\n').join(error.get('errorMessage') for error in error_elements)

        # Pretty-print the XML content of the signature file to be saved as attachment
        attachment_raw = etree.tostring(root, pretty_print=True, xml_declaration=True, encoding='UTF-8')
        return {
            'attachment_raw': attachment_raw,
            'key_signature': root.findtext('.//ns:SignatureValue', namespaces=NS_SIGNATURE),
            'key_certificate': root.findtext('.//ns:X509Certificate', namespaces=NS_SIGNATURE),
            'error': error_message if error_elements else False,
        }

    def action_l10n_ro_edi_fetch_status(self):
        """ Fetch the latest response from E-Factura about the XML sent """
        self.ensure_one()
        # Do the batch fetch process on a single invoice/document
        self.invoice_id._l10n_ro_edi_fetch_invoice_sending_documents()

    def action_l10n_ro_edi_download_signature(self):
        """ Download the received successful signature XML file from E-Factura """
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'url': f'/web/content/{self.attachment_id.id}?download=true',
        }

```

## File: models\res_company.py

```python
import base64
import binascii
import requests

from datetime import datetime
from dateutil.relativedelta import relativedelta
from werkzeug.urls import url_join

from odoo import fields, models, api, _
from odoo.exceptions import UserError, ValidationError
from odoo.http import request
from odoo.tools.safe_eval import json


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_ro_edi_client_id = fields.Char(string='eFactura Client ID')
    l10n_ro_edi_client_secret = fields.Char(string='Client Secret')
    l10n_ro_edi_access_token = fields.Char(string='Access Token')
    l10n_ro_edi_refresh_token = fields.Char(string='Refresh Token')
    l10n_ro_edi_access_expiry_date = fields.Date(string='Access Token Expiry Date')
    l10n_ro_edi_refresh_expiry_date = fields.Date(string='Refresh Token Expiry Date')
    l10n_ro_edi_callback_url = fields.Char(compute='_compute_l10n_ro_edi_callback_url')
    l10n_ro_edi_test_env = fields.Boolean(string='Use Test Environment', default=True)
    l10n_ro_edi_oauth_error = fields.Char()  # Error field to be shown in case of error from the authentication process

    @api.depends('country_code')
    def _compute_l10n_ro_edi_callback_url(self):
        """ Callback URLs are used for generating client_id and client_secret from l10n_ro_edi's setting. """
        for company in self:
            if company.country_code == 'RO':
                company.l10n_ro_edi_callback_url = url_join(request.httprequest.url_root, 'l10n_ro_edi/callback/%s' % company.id)
            else:
                company.l10n_ro_edi_callback_url = False

    def _l10n_ro_edi_log_message(self, message: str, func: str):
        with self.pool.cursor() as cr:
            self = self.with_env(self.env(cr=cr))
            self.env['ir.logging'].sudo().create({
                'name': 'l10n_ro_edi_log',
                'type': 'server',
                'level': 'INFO',
                'dbname': self.env.cr.dbname,
                'message': message,
                'func': func,
                'path': '',
                'line': '1',
            })
            self.env.cr.commit()

    def _l10n_ro_edi_process_token_response(self, response_json):
        """
        To be called just after processing the json response from https://logincert.anaf.ro/anaf-oauth2/v1/token
        This method reads and process the json, and writes the token fields on the company.
        """
        self.ensure_one()
        if 'access_token' not in response_json or 'refresh_token' not in response_json:
            raise ValidationError(_("Token not found.\nResponse: %s", response_json))

        # The access_token is in JWT format, which consists of 3 parts separated by '.':
        # Header, Payload, and Signature. We only need the Payload part to decode the token
        # and get the access expiry date
        payload = response_json['access_token'].split('.')[1]
        payload += '=' * (-len(payload) % 4)
        decoded_payload = base64.b64decode(payload, altchars=b'-_', validate=True)
        access_token_obj = json.loads(decoded_payload)
        access_expiry_date = datetime.fromtimestamp(access_token_obj['exp'])
        refresh_expiry_date = datetime.now() + relativedelta(years=3)
        self.write({
            'l10n_ro_edi_access_token': response_json['access_token'],
            'l10n_ro_edi_refresh_token': response_json['refresh_token'],
            'l10n_ro_edi_access_expiry_date': access_expiry_date,
            'l10n_ro_edi_refresh_expiry_date': refresh_expiry_date,
            'l10n_ro_edi_oauth_error': False,
        })

    def _l10n_ro_edi_refresh_access_token(self, session):
        """
        Uses the saved client_id, client_secret, and refresh_token on the company (self)
        to make request to the SPV and renew the company's token fields.
        """
        self.ensure_one()
        if not self.l10n_ro_edi_client_id or not self.l10n_ro_edi_client_secret:
            raise UserError(_("Client ID and Client Secret field must be filled."))
        if not self.l10n_ro_edi_refresh_token:
            raise UserError(_("Refresh token not found"))

        response = session.post(
            url='https://logincert.anaf.ro/anaf-oauth2/v1/token',
            headers={'Content-Type': 'application/x-www-form-urlencoded'},
            timeout=10,
            data={
                'grant_type': 'refresh_token',
                'refresh_token': self.l10n_ro_edi_refresh_token,
                'client_id': self.l10n_ro_edi_client_id,
                'client_secret': self.l10n_ro_edi_client_secret,
            },
        )
        response_json = response.json()
        self._l10n_ro_edi_process_token_response(response_json)

    def _cron_l10n_ro_edi_refresh_access_token(self):
        """
        This CRON method will be run every 30 days to refresh the following fields on the company:

         - ``l10n_ro_edi_access_token``
         - ``l10n_ro_edi_refresh_token``
         - ``l10n_ro_edi_access_expiry_date``
         - ``l10n_ro_edi_refresh_expiry_date``
        """
        ro_companies = self.env['res.company'].sudo().search([
            ('l10n_ro_edi_refresh_token', '!=', False),
            ('l10n_ro_edi_client_id', '!=', False),
            ('l10n_ro_edi_client_secret', '!=', False),
        ])
        session = requests.Session()
        for company in ro_companies:
            error_cause = ''
            try:
                company._l10n_ro_edi_refresh_access_token(session)
            except ValidationError as e:
                # From access/refresh token not found after sending request
                error_cause = e
            except requests.exceptions.RequestException as e:
                error_cause = _("Error when converting response to json: %s", e)
            except binascii.Error as e:
                error_cause = _("Error when decoding the access token payload: %s", e)
            except Exception as e:
                error_cause = _("Error when refreshing the access token: %s", e)

            if error_cause:
                error_header = _("Refresh token failed [company=%(company_id)s]", company_id=company.id)
                self._l10n_ro_edi_log_message(
                    message=f'{error_header}\n{error_cause}',
                    func='_cron_l10n_ro_edi_refresh_access_token',
                )

```

## File: models\res_config_settings.py

```python
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_ro_edi_client_id = fields.Char(related='company_id.l10n_ro_edi_client_id', readonly=False)
    l10n_ro_edi_client_secret = fields.Char(related='company_id.l10n_ro_edi_client_secret', readonly=False)
    l10n_ro_edi_access_token = fields.Char(related='company_id.l10n_ro_edi_access_token', readonly=False)  # TODO remove readonly=False
    l10n_ro_edi_refresh_token = fields.Char(related='company_id.l10n_ro_edi_refresh_token', readonly=False)
    l10n_ro_edi_access_expiry_date = fields.Date(related='company_id.l10n_ro_edi_access_expiry_date', readonly=False)
    l10n_ro_edi_refresh_expiry_date = fields.Date(related='company_id.l10n_ro_edi_refresh_expiry_date', readonly=False)
    l10n_ro_edi_callback_url = fields.Char(related='company_id.l10n_ro_edi_callback_url')
    l10n_ro_edi_test_env = fields.Boolean(related='company_id.l10n_ro_edi_test_env', readonly=False)
    l10n_ro_edi_oauth_error = fields.Char(related='company_id.l10n_ro_edi_oauth_error')

    def button_l10n_ro_edi_generate_token(self):
        """ Redirects to controllers/main.py ~ `authorize` method """
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'url': '/l10n_ro_edi/authorize/%s' % self.company_id.id,
            'target': 'new',
        }

```

## File: models\__init__.py

```python
from . import account_move
from . import ciusro_document
from . import res_company
from . import res_config_settings

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
l10n_ro_efactura.access_l10n_ro_edi_document_readonly,access_l10n_ro_edi_document_readonly,l10n_ro_efactura.model_l10n_ro_edi_document,base.group_user,1,0,0,0
l10n_ro_efactura.access_l10n_ro_edi_document_group_invoice,access_l10n_ro_edi_document_group_invoice,l10n_ro_efactura.model_l10n_ro_edi_document,account.group_account_invoice,1,1,1,1

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_move_form_inherit_l10n_ro_edi" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n_ro_edi</field>
        <field name="model">account.move</field>
        <field name="priority">30</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='journal_div']" position="after">
                <label for="l10n_ro_edi_state"
                       invisible="not l10n_ro_edi_state or state == 'draft' or move_type in ('in_invoice', 'in_refund')"/>
                <div name="l10n_ro_edi_div"
                     class="d-flex"
                     invisible="not l10n_ro_edi_state or state == 'draft' or move_type in ('in_invoice', 'in_refund')">
                    <field name="l10n_ro_edi_state" class="oe_inline"/>
                </div>
            </xpath>

            <!-- CIUS-RO Documents Tab -->
            <xpath expr="//page[@id='other_tab_entry']" position="after">
                <page id="l10n_ro_edi_documents"
                      string="E-Factura"
                      invisible="not l10n_ro_edi_document_ids">
                    <field name="l10n_ro_edi_document_ids">
                        <tree create="false" delete="false" edit="false" no_open="1"
                              decoration-danger="state == 'invoice_sending_failed'"
                              decoration-warning="state == 'invoice_sending'"
                              decoration-success="state == 'invoice_sent'">
                            <field name="message" column_invisible="1"/>
                            <field name="attachment_id" column_invisible="1"/>
                            <field name="datetime"/>
                            <field name="state" widget="account_document_state"/>
                            <field name="key_loading" optional="hide"/>

                            <button name="action_l10n_ro_edi_fetch_status"
                                    type="object"
                                    string="Fetch status"
                                    invisible="state != 'invoice_sending'"/>
                            <button name="action_l10n_ro_edi_download_signature"
                                    type="object"
                                    string="Download"
                                    invisible="not attachment_id"/>
                        </tree>
                    </field>
                </page>
            </xpath>
        </field>
    </record>

    <record id="out_invoice_tree_inherit_l10n_ro_edi" model="ir.ui.view">
        <field name="name">out.invoice.tree.inherit.l10n_ro_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_out_invoice_tree"/>
        <field name="arch" type="xml">
            <field name="state" position="before">
                <field name="l10n_ro_edi_state" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="out_credit_note_tree_inherit_l10n_ro_edi" model="ir.ui.view">
        <field name="name">out.credit.note.tree.inherit.l10n_ro_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_out_credit_note_tree"/>
        <field name="arch" type="xml">
            <field name="state" position="before">
                <field name="l10n_ro_edi_state" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="in_invoice_tree_inherit_l10n_ro_edi" model="ir.ui.view">
        <field name="name">in.invoice.tree.inherit.l10n_ro_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_in_invoice_tree"/>
        <field name="arch" type="xml">
            <field name="state" position="before">
                <field name="l10n_ro_edi_state" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="l10n_ro_edi_view_account_invoice_filter" model="ir.ui.view">
        <field name="name">account.invoice.select.inherit.l10n.ro.edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//search/field[@name='journal_id']" position="after">
                <field name="l10n_ro_edi_state"/>
            </xpath>
            <xpath expr="//filter[@name='to_check']" position="after">
                <filter string="Sending E-Factura" name="l10n_ro_edi_state_invoice_sending"
                        domain="[('l10n_ro_edi_state', '=', 'invoice_sending')]"/>
                <filter string="Error E-Factura" name="l10n_ro_edi_state_invoice_sending_failed"
                        domain="[('l10n_ro_edi_document_ids.state', '=', 'invoice_sending_failed')]"/>
                <filter string="Sent E-Factura" name="l10n_ro_edi_state_invoice_sent"
                        domain="[('l10n_ro_edi_state', '=', 'invoice_sent')]"/>
            </xpath>
            <xpath expr="//group" position="inside">
                <filter string="E-Factura Status" name="l10n_ro_edi_state_group"
                        domain="" context="{'group_by':'l10n_ro_edi_state'}"/>
            </xpath>
        </field>
    </record>

    <record id="l10n_ro_edi_action_fetch_ciusro_status" model="ir.actions.server">
        <field name="name">Fetch E-Factura Status</field>
        <field name="model_id" ref="account.model_account_move"/>
        <field name="binding_model_id" ref="account.model_account_move"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
            if records:
                records._l10n_ro_edi_fetch_invoice_sending_documents()
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_form_inherit_l10n_ro_edi" model="ir.ui.view">
        <field name="name">res.config.settings.form.inherit.l10n.ro.edi</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@id='invoicing_settings']" position="after">
                <block title="Romanian E-Factura" id="l10n_ro_edi_settings" invisible="country_code != 'RO'">
                    <div class="col-12 col-lg-12 o_setting_box">
                        <div class="o_setting_right_pane border-0">
                            <div class="mb-2">
                                <span class="o_form_label">
                                    E-Factura Details
                                </span>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                            </div>
                            <div class="text-muted oe_inline mb-3">
                                <div class="mb-1">
                                    <div>Data needed to generate token for sending RO-CIUS to Romanian government</div>
                                    <div>
                                        To benefit from this feature, you must have a digital signature USB token from
                                        Romania connected with an ANAF account
                                    </div>
                                </div>
                                <ul>
                                    <li>
                                        Go to <a href="https://pfinternet.anaf.ro/" target="_blank">https://pfinternet.anaf.ro/</a>.
                                        Login or Sign Up if you don't have an account yet
                                    </li>
                                    <li>Go to "Editare profil Oauth"</li>
                                    <li>Generate a new Client data
                                        <ul>
                                            <li>Fill "Denumire Aplicatie" with any desired name for your registration data</li>
                                            <li>
                                                Fill "Callback URL 1" with the following URL:
                                                <br/><field name="l10n_ro_edi_callback_url" widget="text" class="w-100 text-success"/>
                                            </li>
                                            <li>In "Serviciu", select the option "E-Factura"</li>
                                            <li>Submit the form by clicking on "Generare Client ID"</li>
                                        </ul>
                                    </li>
                                    <li>Copy the generated Client ID and Client Secret to the fields below</li>
                                    <li>With the digital signature USB token inserted, generate the token using the button below</li>
                                </ul>
                            </div>
                            <div class="row text-warning" invisible="not l10n_ro_edi_oauth_error">
                                Error when generating token:
                                <field name="l10n_ro_edi_oauth_error" widget="text" class="w-100"/>
                            </div>
                            <div class="row">
                                <label for="l10n_ro_edi_client_id" string="Client ID" class="col-lg-3 o_light_label"/>
                                <field name="l10n_ro_edi_client_id"/>
                            </div>
                            <div class="row">
                                <label for="l10n_ro_edi_client_secret" class="col-lg-3 o_light_label"/>
                                <field name="l10n_ro_edi_client_secret"/>
                            </div>
                            <div class="row">
                                <label for="l10n_ro_edi_access_token" class="col-lg-3 o_light_label"/>
                                <field name="l10n_ro_edi_access_token"/>
                            </div>
                            <div class="row">
                                <label for="l10n_ro_edi_refresh_token" class="col-lg-3 o_light_label"/>
                                <field name="l10n_ro_edi_refresh_token"/>
                            </div>
                            <div class="row">
                                <label for="l10n_ro_edi_access_expiry_date" class="col-lg-3 o_light_label"/>
                                <field name="l10n_ro_edi_access_expiry_date"/>
                            </div>
                            <div class="row">
                                <label for="l10n_ro_edi_refresh_expiry_date" class="col-lg-3 o_light_label"/>
                                <field name="l10n_ro_edi_refresh_expiry_date"/>
                            </div>
                            <div class="my-3" invisible="l10n_ro_edi_access_token">
                                <button name="button_l10n_ro_edi_generate_token"
                                        type="object"
                                        string="Generate Token"
                                        class="btn btn-primary"/>
                            </div>
                            <setting id="l10n_ro_edi_test_env_setting" class="mt-3"
                                     help="Activate test environment for sending E-Factura to SPV">
                                <field name="l10n_ro_edi_test_env"/>
                            </setting>
                        </div>
                    </div>
                </block>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\account_move_send.py

```python
from odoo import api, fields, models, _


class AccountMoveSend(models.TransientModel):
    _inherit = 'account.move.send'

    l10n_ro_edi_send_enable = fields.Boolean(compute='_compute_l10n_ro_edi_send_enable')
    l10n_ro_edi_send_readonly = fields.Boolean(compute='_compute_l10n_ro_edi_send_readonly')
    l10n_ro_edi_send_checkbox = fields.Boolean(
        string='Send E-Factura to SPV',
        compute='_compute_l10n_ro_edi_send_checkbox', store=True, readonly=False,
        help='Send the CIUS-RO XML to the Romanian Government via the ANAF platform')
    l10n_ro_edi_warnings = fields.Char(compute='_compute_l10n_ro_edi_warnings')  # To be removed in master

    def _get_wizard_values(self):
        # EXTENDS 'account'
        values = super()._get_wizard_values()
        values['l10n_ro_edi_send'] = self.l10n_ro_edi_send_checkbox
        return values

    @api.depends('move_ids.l10n_ro_edi_state', 'enable_ubl_cii_xml')
    def _compute_l10n_ro_edi_send_enable(self):
        """ Enable send to SPV if we can create the XML, or
            if the XML is already created and the move already have a l10n_ro_edi.document in error """
        for wizard in self:
            wizard.l10n_ro_edi_send_enable = any(
                (move._need_ubl_cii_xml() or move.ubl_cii_xml_id) and
                move.country_code == 'RO' and
                move.l10n_ro_edi_state in (False, 'invoice_sending')
                for move in wizard.move_ids
            )

    @api.depends('move_ids.l10n_ro_edi_state', 'l10n_ro_edi_send_enable')
    def _compute_l10n_ro_edi_send_readonly(self):
        """ We shouldn't allow the user to send a new request if any move is currently waiting for an answer. """
        for wizard in self:
            wizard.l10n_ro_edi_send_readonly = (
                not wizard.l10n_ro_edi_send_enable
                or 'invoice_sending' in wizard.move_ids.mapped('l10n_ro_edi_state')
            )

    @api.depends('l10n_ro_edi_send_readonly')
    def _compute_l10n_ro_edi_send_checkbox(self):
        for wizard in self:
            wizard.l10n_ro_edi_send_checkbox = not wizard.l10n_ro_edi_send_readonly

    @api.depends('l10n_ro_edi_send_readonly')
    def _compute_l10n_ro_edi_warnings(self):
        """ TODO in master (saas-17.4): merge it with `warnings` field using `_compute_warnings`. """
        for wizard in self:
            waiting_moves = wizard.move_ids.filtered(lambda m: m.l10n_ro_edi_state == 'invoice_sending')
            wizard.l10n_ro_edi_warnings = _(
                "The following move(s) are waiting for answer from the Romanian SPV: %s",
                ', '.join(waiting_moves.mapped('name'))
            ) if waiting_moves else False

    @api.depends('l10n_ro_edi_send_checkbox')
    def _compute_checkbox_ubl_cii_xml(self):
        # EXTENDS 'account_edi_ubl_cii'
        super()._compute_checkbox_ubl_cii_xml()
        for wizard in self:
            if wizard.l10n_ro_edi_send_checkbox and wizard.enable_ubl_cii_xml and not wizard.checkbox_ubl_cii_xml:
                wizard.checkbox_ubl_cii_xml = True

    @api.model
    def _call_web_service_after_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_after_invoice_pdf_render(invoices_data)

        for invoice, invoice_data in invoices_data.items():
            if invoice_data.get('l10n_ro_edi_send') and not invoice.l10n_ro_edi_state:
                build_errors = None
                if invoice_data.get('ubl_cii_xml_attachment_values'):
                    xml_data = invoice_data['ubl_cii_xml_attachment_values']['raw']
                elif invoice.l10n_ro_edi_document_ids:
                    # If a document is on the invoice but the invoice's l10n_ro_edi_state is False,
                    # this means that the previously sent XML are invalid and have to be rebuilt
                    xml_data, build_errors = self.env['account.edi.xml.ubl_ro']._export_invoice(invoice)
                elif invoice.ubl_cii_xml_id:
                    xml_data = invoice.ubl_cii_xml_id.raw
                else:
                    xml_data, build_errors = self.env['account.edi.xml.ubl_ro']._export_invoice(invoice)

                if build_errors:
                    invoice_data['error'] = {
                        'error_title': _("Error when building the CIUS-RO E-Factura XML"),
                        'errors': build_errors,
                    }
                    continue

                invoice._l10n_ro_edi_send_invoice(xml_data)

                if self._can_commit():
                    self.env.cr.commit()

                active_document = invoice.l10n_ro_edi_document_ids.sorted()[0]

                if active_document.state == 'invoice_sending_failed':
                    invoice_data['error'] = {
                        'error_title': _("Error when sending CIUS-RO E-Factura to the SPV"),
                        'errors': active_document.message.split('\n'),
                    }

```

## File: wizard\account_move_send_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="account_move_send_inherit_l10n_ro_edi">
            <field name="name">account.move.send.form.inherit.l10n_ro_edi</field>
            <field name="model">account.move.send</field>
            <field name="inherit_id" ref="account.account_move_send_form"/>
            <field name="arch" type="xml">

                <!--TODO: to be removed in master-->
                <xpath expr="//div[@name='warnings']" position="inside">
                    <div class="alert alert-warning" role="alert" invisible="not l10n_ro_edi_warnings">
                        <field name="l10n_ro_edi_warnings"/>
                    </div>
                </xpath>

                <xpath expr="//div[@name='advanced_options']" position="inside">
                    <field name="l10n_ro_edi_send_enable" invisible="1"/>
                    <field name="l10n_ro_edi_send_readonly" invisible="1"/>
                    <div name="option_l10n_ro_edi" invisible="not l10n_ro_edi_send_enable">
                        <field name="l10n_ro_edi_send_checkbox" readonly="l10n_ro_edi_send_readonly"/>
                        <b><label for="l10n_ro_edi_send_checkbox"/></b>
                        <i class="fa fa-question-circle"
                           role="img"
                           aria-label="Warning"
                           title="You can't send now. Some move(s) are waiting for an answer."
                           invisible="not l10n_ro_edi_send_readonly"/>
                    </div>
                </xpath>

            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
from . import account_move_send

```

