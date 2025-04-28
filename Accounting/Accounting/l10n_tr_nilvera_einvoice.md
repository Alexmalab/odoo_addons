# Odoo Module: l10n_tr_nilvera_einvoice

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import wizard

```

## File: __manifest__.py

```python
{
    'name': 'Türkiye - Nilvera E-Invoice',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'description': """
For sending and receiving electronic invoices to Nilvera.
    """,
    'depends': ['l10n_tr_nilvera', 'account_edi_ubl_cii'],
    'data': [
        'data/cron.xml',
        'views/account_journal_dashboard_views.xml',
        'views/account_move_views.xml',
        'wizard/account_move_send_views.xml',
    ],
    'auto_install': ['l10n_tr_nilvera'],
    'license': 'LGPL-3',
}

```

## File: data\cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_nilvera_get_new_documents" model="ir.cron">
        <field name="name">Nilvera: retrieve new documents</field>
        <field name="model_id" ref="model_account_move"/>
        <field name="state">code</field>
        <field name="code">model._cron_nilvera_get_new_documents()</field>
        <field name="interval_number">12</field>
        <field name="interval_type">hours</field>
        <field name="numbercall">-1</field>
        <field name="nextcall" eval="(DateTime.now() + timedelta(hours=12)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="active" eval="True"/>
    </record>

    <record id="ir_cron_nilvera_get_invoice_status" model="ir.cron">
        <field name="name">Nilvera: retrieve invoice status</field>
        <field name="model_id" ref="model_account_move"/>
        <field name="state">code</field>
        <field name="code">model._cron_nilvera_get_invoice_status()</field>
        <field name="interval_number">12</field>
        <field name="interval_type">hours</field>
        <field name="numbercall">-1</field>
        <field name="nextcall" eval="(DateTime.now() + timedelta(hours=12)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="active" eval="True"/>
    </record>
</odoo>

```

## File: models\account_edi_xml_ubl_tr.py

```python
from odoo import models


class AccountEdiXmlUblTr(models.AbstractModel):
    _name = "account.edi.xml.ubl.tr"
    _inherit = 'account.edi.xml.ubl_21'
    _description = "UBL-TR 1.2"

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        # EXTENDS account_edi_ubl_cii
        return '%s_einvoice.xml' % invoice.name.replace("/", "_")

    def _export_invoice_vals(self, invoice):
        def _get_formatted_id(invoice):
            # For now, we assume that the sequence is going to be in the format {prefix}/{year}/{invoice_number}.
            # To send an invoice to Nlvera, the format needs to follow ABC2009123456789.
            parts = invoice.name.split('/')
            prefix, year, number = parts[0], parts[1], parts[2].zfill(9)
            return f"{prefix}{year}{number}"

        # EXTENDS account.edi.xml.ubl_21
        vals = super()._export_invoice_vals(invoice)

        # Check the customer status if it hasn't been done before as it's needed for profile_id
        if invoice.partner_id.l10n_tr_nilvera_customer_status == 'not_checked':
            invoice.partner_id.check_nilvera_customer()

        vals['vals'].update({
            'id': _get_formatted_id(invoice),
            'customization_id': 'TR1.2',
            'profile_id': 'TEMELFATURA' if invoice.partner_id.l10n_tr_nilvera_customer_status == 'einvoice' else 'EARSIVFATURA',
            'copy_indicator': 'false',
            'uuid': invoice.l10n_tr_nilvera_uuid,
            'document_type_code': 'SATIS' if invoice.move_type == 'out_invoice' else 'IADE',
            'due_date': False,
            'line_count_numeric': len(invoice.line_ids),
            'order_issue_date': invoice.invoice_date,
        })
        return vals

    def _get_country_vals(self, country):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_country_vals(country)
        vals['name'] = country.with_context(lang='tr_TR').name
        return vals

    def _get_partner_party_identification_vals_list(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_party_identification_vals_list(partner)
        vals.append({
            'id_attrs': {
                'schemeID': 'VKN' if partner.is_company else 'TCKN',
            },
            'id': partner.vat,
        })
        return vals

    def _get_partner_address_vals(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_address_vals(partner)
        vals.update({
            'city_subdivision_name ': partner.city,
            'city_name': partner.state_id.name,
            'country_subentity': False,
            'country_subentity_code': False,
        })
        return vals

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_partner_party_tax_scheme_vals_list(partner, role)
        for vals in vals_list:
            vals.pop('registration_address_vals', None)
        return vals_list

    def _get_partner_party_legal_entity_vals_list(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_partner_party_legal_entity_vals_list(partner)
        for vals in vals_list:
            vals.pop('registration_address_vals', None)
        return vals_list

    def _get_partner_person_vals(self, partner):
        if not partner.is_company:
            name_parts = partner.name.split(' ', 1)
            return {
                'first_name': name_parts[0],
                # If no family name is present, use a zero-width space (U+200B) to ensure the XML tag is rendered. This is required by Nilvera.
                'family_name': name_parts[1] if len(name_parts) > 1 else '\u200B',
            }
        return super()._get_partner_person_vals(partner)

    def _get_delivery_vals_list(self, invoice):
        # EXTENDS account.edi.xml.ubl_21
        delivery_vals = super()._get_delivery_vals_list(invoice)
        if 'picking_ids' in invoice._fields and invoice.picking_ids:
            delivery_vals[0]['delivery_id'] = invoice.picking_ids[0].name
            return delivery_vals
        return []

    def _get_invoice_payment_means_vals_list(self, invoice):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_invoice_payment_means_vals_list(invoice)
        for vals in vals_list:
            vals.pop('instruction_id', None)
            vals.pop('payment_id_vals', None)
        return vals_list

    def _get_tax_category_list(self, invoice, taxes):
        # OVERRIDES account.edi.common
        res = []
        for tax in taxes:
            is_withholding = invoice.currency_id.compare_amounts(tax.amount, 0) == -1
            tax_type_code = '9015' if is_withholding else '0015'
            tax_scheme_name = 'KDV Tevkifatı' if is_withholding else 'Gerçek Usulde KDV'
            res.append({
                'id': tax_type_code,
                'percent': tax.amount if tax.amount_type == 'percent' else False,
                'tax_scheme_vals': {'name': tax_scheme_name, 'tax_type_code': tax_type_code},
            })
        return res

    def _get_invoice_tax_totals_vals_list(self, invoice, taxes_vals):
        # EXTENDS account.edi.xml.ubl_21
        tax_totals_vals = super()._get_invoice_tax_totals_vals_list(invoice, taxes_vals)

        for vals in tax_totals_vals:
            for subtotal_vals in vals.get('tax_subtotal_vals', []):
                subtotal_vals.get('tax_category_vals', {})['id'] = False
                subtotal_vals.get('tax_category_vals', {})['percent'] = False

        return tax_totals_vals

    def _get_invoice_monetary_total_vals(self, invoice, taxes_vals, line_extension_amount, allowance_total_amount, charge_total_amount):
        # EXTENDS account.edi.xml.ubl_20
        vals = super()._get_invoice_monetary_total_vals(invoice, taxes_vals, line_extension_amount, allowance_total_amount, charge_total_amount)
        # allowance_total_amount needs to have a value even if 0.0 otherwise it's blank in the Nilvera PDF.
        vals['allowance_total_amount'] = allowance_total_amount
        if invoice.currency_id.is_zero(vals.get('prepaid_amount', 1)):
            del vals['prepaid_amount']
        return vals

    def _get_invoice_line_item_vals(self, line, taxes_vals):
        # EXTENDS account.edi.xml.ubl_21
        line_item_vals = super()._get_invoice_line_item_vals(line, taxes_vals)
        line_item_vals['classified_tax_category_vals'] = False
        return line_item_vals

    def _get_additional_document_reference_list(self, invoice):
        # EXTENDS account.edi.xml.ubl_20
        additional_document_reference_list = super()._get_additional_document_reference_list(invoice)
        if invoice.partner_id.l10n_tr_nilvera_customer_status == 'earchive':
            additional_document_reference_list.append({
                'id': "ELEKTRONIK",
                'issue_date': invoice.invoice_date,
                'document_type_code': "SEND_TYPE",
            })
        return additional_document_reference_list

    def _get_invoice_line_price_vals(self, line):
        # EXTEND 'account.edi.common'
        invoice_line_price_vals = super()._get_invoice_line_price_vals(line)
        invoice_line_price_vals['base_quantity_attrs'] = {'unitCode': line.product_uom_id._get_unece_code()}
        return invoice_line_price_vals

    def _get_invoice_line_vals(self, line, line_id, taxes_vals):
        invoice_line_vals = super()._get_invoice_line_vals(line, line_id, taxes_vals)
        invoice_line_vals['line_quantity_attrs'] = {'unitCode': line.product_uom_id._get_unece_code()}
        return invoice_line_vals

    # -------------------------------------------------------------------------
    # IMPORT
    # -------------------------------------------------------------------------

    def _import_retrieve_partner_vals(self, tree, role):
        # EXTENDS account.edi.xml.ubl_20
        partner_vals = super()._import_retrieve_partner_vals(tree, role)
        partner_vals.update({
            'vat': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cac:PartyIdentification//cbc:ID[string-length(text()) > 5]', tree),
        })
        return partner_vals

    def _import_fill_invoice_form(self, invoice, tree, qty_factor):
        # EXTENDS account.edi.xml.ubl_20
        logs = super()._import_fill_invoice_form(invoice, tree, qty_factor)

        # ==== Nilvera UUID ====
        if uuid_node := tree.findtext('./{*}UUID'):
            invoice.l10n_tr_nilvera_uuid = uuid_node

        return logs

```

## File: models\account_journal.py

```python
from odoo import models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    def l10n_tr_nilvera_get_documents(self):
        self.env['account.move']._l10n_tr_nilvera_get_documents()

    def l10n_tr_nilvera_get_message_status(self):
        """ Gets the status from Nilvera for all processing invoices in this journal. """
        invoices_to_update = self.env['account.move'].search([
            ('journal_id', '=', self.id),
            ('l10n_tr_nilvera_send_status', 'in', ['waiting', 'sent']),
        ])
        invoices_to_update._l10n_tr_nilvera_get_submitted_document_status()

```

## File: models\account_move.py

```python
import uuid
from markupsafe import Markup
from urllib.parse import quote, urlencode, urlparse

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.addons.l10n_tr_nilvera.lib.nilvera_client import _get_nilvera_client


class AccountMove(models.Model):
    _name = 'account.move'
    _inherit = ['account.move']

    l10n_tr_nilvera_uuid = fields.Char(
        string="Nilvera Document UUID",
        copy=False,
        readonly=True,
        default=lambda self: str(uuid.uuid4()),
        help="Universally unique identifier of the Invoice",
    )
    l10n_tr_nilvera_send_status = fields.Selection(
        selection=[
            ('error', "Error (check chatter)"),
            ('not_sent', "Not sent"),
            ('sent', "Sent and waiting response"),
            ('succeed', "Successful"),
            ('waiting', "Waiting"),
            ('unknown', "Unknown"),
        ],
        string="Nilvera Status",
        readonly=True,
        copy=False,
        default='not_sent',
    )

    @api.model
    def _get_ubl_cii_builder_from_xml_tree(self, tree):
        customization_id = tree.find('{*}CustomizationID')
        if customization_id is not None and 'TR1.2' in customization_id.text:
            return self.env['account.edi.xml.ubl.tr']
        return super()._get_ubl_cii_builder_from_xml_tree(tree)

    def button_draft(self):
        # EXTENDS account
        for move in self:
            if move.l10n_tr_nilvera_uuid and move.l10n_tr_nilvera_send_status != 'not_sent':
                raise UserError(_("You cannot reset to draft an entry that has been sent to Nilvera."))
        super().button_draft()

    def _l10n_tr_nilvera_submit_einvoice(self, xml_file, customer_alias):
        self._l10n_tr_nilvera_submit_document(
            xml_file=xml_file,
            endpoint=f"/einvoice/Send/Xml?{urlencode({'Alias': customer_alias})}",
        )

    def _l10n_tr_nilvera_submit_earchive(self, xml_file):
        self._l10n_tr_nilvera_submit_document(
            xml_file=xml_file,
            endpoint="/earchive/Send/Xml",
        )

    def _l10n_tr_nilvera_submit_document(self, xml_file, endpoint, post_series=True):
        """
        Submits an e-invoice or e-archive document to Nilvera for processing.

        :param xml_file: The XML file to be submitted.
        :type xml_file: file-like object
        :param endpoint: The Nilvera API endpoint for submission.
        :type endpoint: str
        :param post_series: Whether to attempt posting the series/sequence to Nilvera if it is missing.
                            Defaults to True. Useful for avoiding an infinite loop.
        :type post_series: bool
        :raises UserError: If the API key lacks necessary rights (401 or 403 responses), if the response
                            indicates a client error (4xx), or if a server error occurs (500).
        :return: None
        """
        with _get_nilvera_client(self.env.company) as client:
            response = client.request(
                "POST",
                endpoint,
                files={'file': (xml_file.name, xml_file, 'application/xml')},
                handle_response=False,
            )

            if response.status_code == 200:
                self.is_move_sent = True
                self.l10n_tr_nilvera_send_status = 'sent'
            elif response.status_code in {401, 403}:
                raise UserError(_("Oops, seems like you're unauthorised to do this. Try another API key with more rights or contact Nilvera."))
            elif 400 <= response.status_code < 500:
                error_message, error_codes = self._l10n_tr_nilvera_einvoice_get_error_messages_from_response(response)

                # If the sequence/series is not found on Nilvera, add it then retry.
                if 3009 in error_codes and post_series:
                    self._l10n_tr_nilvera_post_series(endpoint, client)
                    return self._l10n_tr_nilvera_submit_document(xml_file, endpoint, post_series=False)
                raise UserError(error_message)
            elif response.status_code == 500:
                raise UserError(_("Server error from Nilvera, please try again later."))

            self.message_post(body=_("The invoice has been successfully sent to Nilvera."))

    def _l10n_tr_nilvera_post_series(self, endpoint, client):
        """Post the series to Nilvera based on the endpoint."""
        path = urlparse(endpoint).path  # Remove query params from te endpoint.
        if path == "/einvoice/Send/Xml":
            series_endpoint = "/einvoice/Series"
        elif path == "/earchive/Send/Xml":
            series_endpoint = "/earchive/Series"
        else:
            # Return early if endpoint couldn't be matched.
            return

        if not self.sequence_prefix:
            return

        series = self.sequence_prefix.split('/', 1)[0]
        client.request(
            "POST",
            series_endpoint,
            json={
                'Name': series,
                'IsActive': True,
                'IsDefault': False,
            },
        )

    def _l10n_tr_nilvera_get_submitted_document_status(self):
        with _get_nilvera_client(self.env.company) as client:
            for invoice in self:
                response = client.request(
                    "GET",
                    f"/einvoice/sale/{invoice.l10n_tr_nilvera_uuid}/Status",
                )

                nilvera_status = response.get('InvoiceStatus', {}).get('Code')
                if nilvera_status in dict(invoice._fields['l10n_tr_nilvera_send_status'].selection):
                    invoice.l10n_tr_nilvera_send_status = nilvera_status
                    if nilvera_status == 'error':
                        invoice.message_post(
                            body=Markup(
                                "%s<br/>%s - %s<br/>"
                            ) % (
                                _("The invoice couldn't be sent to the recipient."),
                                response['InvoiceStatus'].get('Description'),
                                response['InvoiceStatus'].get('DetailDescription'),
                            )
                        )
                else:
                    invoice.message_post(body=_("The invoice status couldn't be retrieved from Nilvera."))

    def _l10n_tr_nilvera_get_documents(self):
        with _get_nilvera_client(self.env.company) as client:
            response = client.request(
                "GET",
                "/einvoice/Purchase",
            )

            if not response.get('Content'):
                return

            journal = self.env.company.l10n_tr_nilvera_purchase_journal_id
            if not journal:
                journal = self.env['account.journal'].search([
                    *self.env['account.journal']._check_company_domain(self.env.company),
                    ('type', '=', 'purchase'),
                ], limit=1)

            document_uuids = [content.get('UUID') for content in response.get('Content')]
            document_uuids_count = dict(self.env['account.move']._read_group(
                [('l10n_tr_nilvera_uuid', 'in', document_uuids)],
                groupby=['l10n_tr_nilvera_uuid'],
                aggregates=['__count'],
            ))
            for document_uuid in document_uuids:
                # Skip invoices that have already been downloaded.
                if document_uuid in document_uuids_count:
                    continue
                move = self._l10n_tr_nilvera_get_invoice_from_uuid(client, journal, document_uuid)
                self._l10n_tr_nilvera_add_pdf_to_invoice(client, move, document_uuid)
                self._cr.commit()

    def _l10n_tr_nilvera_get_invoice_from_uuid(self, client, journal, document_uuid):
        response = client.request(
            "GET",
            f"/einvoice/Purchase/{quote(document_uuid)}/xml",
        )

        attachment_vals = {
            'name': 'attachment.xml',
            'raw': response,
            'type': 'binary',
            'mimetype': 'application/xml',
        }

        attachment = self.env['ir.attachment'].create(attachment_vals)
        try:
            move = journal.with_context(
                default_move_type='in_invoice',
                default_l10n_tr_nilvera_uuid=document_uuid,
            )._create_document_from_attachment(attachment.id)

            # If move creation was successful, update the attachment name with the bill reference.
            if move.ref:
                attachment.name = f'{move.ref}.xml'

            move._message_log(body=_("Nilvera document has been received successfully"))
        except Exception:   # noqa: BLE001
            # If the invoice creation fails, create an empty invoice with the attachment. The PDF will be
            # added in a later step as well.
            move = self.env['account.move'].create({
                'move_type': 'in_invoice',
                'company_id': self.env.company.id,
                'l10n_tr_nilvera_uuid': document_uuid,
            })
            attachment.write({
                'res_model': 'account.move',
                'res_id': move.id,
            })

        return move

    def _l10n_tr_nilvera_add_pdf_to_invoice(self, client, invoice, document_uuid):
        response = client.request(
            "GET",
            f"/einvoice/Purchase/{quote(document_uuid)}/pdf",
        )

        filename = f'{invoice.ref}.pdf' if invoice.ref else 'Nilvera PDF.pdf'

        attachment = self.env['ir.attachment'].create({
            'name': filename,
            'res_id': invoice.id,
            'res_model': 'account.move',
            'datas': response,
            'type': 'binary',
            'mimetype': 'application/pdf',
        })

        if (invoice.message_main_attachment_id
                and invoice.message_main_attachment_id.name.endswith('.xml')
                and 'pdf' not in invoice.message_main_attachment_id.mimetype):
            invoice.message_main_attachment_id = attachment
        invoice.with_context(no_new_invoice=True).message_post(attachment_ids=attachment.ids)

    def _l10n_tr_nilvera_einvoice_get_error_messages_from_response(self, response):
        msg = ""
        error_codes = []

        response_json = response.json()
        if errors := response_json.get('Errors'):
            msg += _("The invoice couldn't be sent due to the following errors:\n")
            for error in errors:
                msg += "%s - %s: %s\n" % (error.get('Code'), error.get('Description'), error.get('Detail'))
                error_codes.append(error.get('Code'))

        return msg, error_codes

    # -------------------------------------------------------------------------
    # CRONS
    # -------------------------------------------------------------------------

    def _cron_nilvera_get_new_documents(self):
        self._l10n_tr_nilvera_get_documents()

    def _cron_nilvera_get_invoice_status(self):
        invoices_to_update = self.env['account.move'].search([
            ('l10n_tr_nilvera_send_status', 'in', ['waiting', 'sent']),
        ])
        invoices_to_update._l10n_tr_nilvera_get_submitted_document_status()

```

## File: models\__init__.py

```python
from . import account_edi_xml_ubl_tr
from . import account_journal
from . import account_move

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
                    <field name="is_nilvera_journal" invisible="1"/>
                    <field name="l10n_tr_nilvera_api_key" invisible="1"/>
                </xpath>

                <xpath expr="//t[@id='account.JournalBodySalePurchase']//div" position="inside">
                    <t t-if="record.l10n_tr_nilvera_api_key and record.l10n_tr_nilvera_api_key.raw_value">
                        <t t-if="journal_type == 'sale'">
                            <div class="w-100">
                                <a type="object" name="l10n_tr_nilvera_get_message_status" groups="account.group_account_invoice">Fetch Nilvera invoice status</a>
                            </div>
                        </t>
                        <t t-elif="journal_type == 'purchase' and record.is_nilvera_journal.raw_value">
                            <div class="w-100">
                                <a type="object" name="l10n_tr_nilvera_get_documents" groups="account.group_account_invoice">Fetch from Nilvera</a>
                            </div>
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
    <record id="account_nilvera_view_move_form" model="ir.ui.view">
        <field name="name">account.nilvera.view.move.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">

            <xpath expr="//div[@name='journal_div']" position="after">
                <label for="l10n_tr_nilvera_send_status"
                       invisible="country_code != 'TR' or state == 'draft' or move_type in ['in_invoice', 'in_refund']"/>
                <div name="nilvera_div"
                     class="d-flex"
                     invisible="country_code != 'TR' or state == 'draft' or move_type in ['in_invoice', 'in_refund']">
                    <field name="l10n_tr_nilvera_send_status" class="oe_inline"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\account_move_send.py

```python
from io import BytesIO
import logging

from odoo import _, api, fields, models

_logger = logging.getLogger(__name__)


class AccountMoveSend(models.TransientModel):
    _inherit = 'account.move.send'

    l10n_tr_nilvera_einvoice_enable_xml = fields.Boolean(compute='_compute_l10n_tr_nilvera_einvoice_enable_xml')
    l10n_tr_nilvera_einvoice_checkbox_xml = fields.Boolean(
        string="Send E-Invoice to Nilvera",
        compute='_compute_l10n_tr_nilvera_einvoice_checkbox_xml',
        store=True,
        readonly=False,
    )
    l10n_tr_nilvera_warnings = fields.Json(compute='_compute_l10n_tr_nilvera_warnings')

    def _get_wizard_values(self):
        # EXTENDS 'account'
        values = super()._get_wizard_values()
        values['l10n_tr_nilvera_einvoice_xml'] = self.l10n_tr_nilvera_einvoice_checkbox_xml
        return values

    @api.model
    def _get_wizard_vals_restrict_to(self, only_options):
        # EXTENDS 'account'
        values = super()._get_wizard_vals_restrict_to(only_options)
        return {
            'l10n_tr_nilvera_einvoice_checkbox_xml': False,
            **values,
        }

    @api.model
    def _get_default_l10n_tr_nilvera_einvoice_enable_einvoice(self, move):
        return move.l10n_tr_nilvera_send_status == 'not_sent' and move.is_invoice(include_receipts=True) and move.country_code == 'TR'

    def _l10n_tr_nilvera_check_invoices(self):
        moves_to_check = self.move_ids.filtered(self._get_default_l10n_tr_nilvera_einvoice_enable_einvoice)
        invalid_records = moves_to_check.partner_id.filtered(
            lambda p: p.country_code != 'TR' or not p.city or not p.state_id or not p.street
        )
        if invalid_records:
            return {
                "partner_data_missing": {
                    "message": _("The following partner(s) are either not Turkish or are missing one of those fields: city, state and street."),
                    "action_text": _("View Partner(s)"),
                    "action": invalid_records._get_records_action(
                        name=_("Check data on Partner(s)"),
                    ),
                }
            }

        return {}

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('l10n_tr_nilvera_einvoice_checkbox_xml')
    def _compute_checkbox_ubl_cii_xml(self):
        # EXTENDS 'account_edi_ubl_cii'
        super()._compute_checkbox_ubl_cii_xml()
        for wizard in self:
            if wizard.l10n_tr_nilvera_einvoice_checkbox_xml and wizard.enable_ubl_cii_xml and not wizard.checkbox_ubl_cii_xml:
                wizard.checkbox_ubl_cii_xml = True

    @api.depends('move_ids')
    def _compute_l10n_tr_nilvera_einvoice_enable_xml(self):
        for wizard in self:
            wizard.l10n_tr_nilvera_einvoice_enable_xml = any(self._get_default_l10n_tr_nilvera_einvoice_enable_einvoice(move) for move in wizard.move_ids)

    @api.depends('l10n_tr_nilvera_einvoice_enable_xml')
    def _compute_l10n_tr_nilvera_einvoice_checkbox_xml(self):
        for wizard in self:
            wizard.l10n_tr_nilvera_einvoice_checkbox_xml = wizard.l10n_tr_nilvera_einvoice_enable_xml

    @api.depends('l10n_tr_nilvera_einvoice_checkbox_xml')
    def _compute_mail_attachments_widget(self):
        # EXTENDS 'account' - add depends
        super()._compute_mail_attachments_widget()

    @api.depends('l10n_tr_nilvera_einvoice_checkbox_xml')
    def _compute_l10n_tr_nilvera_warnings(self):
        for wizard in self:
            if wizard.l10n_tr_nilvera_einvoice_checkbox_xml:
                wizard.l10n_tr_nilvera_warnings = wizard._l10n_tr_nilvera_check_invoices()
            else:
                wizard.l10n_tr_nilvera_warnings = False

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------
    def _link_invoice_documents(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._link_invoice_documents(invoice, invoice_data)
        # The move needs to be put as sent only if sent by Nilvera
        if invoice.company_id.country_code == 'TR':
            invoice.is_move_sent = invoice.l10n_tr_nilvera_send_status == 'sent'

    @api.model
    def _call_web_service_before_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_before_invoice_pdf_render(invoices_data)

        for invoice, invoice_data in invoices_data.items():
            if invoice_data.get('l10n_tr_nilvera_einvoice_xml'):
                if attachment_values := invoice_data.get('ubl_cii_xml_attachment_values'):
                    xml_file = BytesIO(attachment_values.get('raw'))
                    xml_file.name = attachment_values['name']
                else:
                    xml_file = BytesIO(invoice.ubl_cii_xml_id.raw or b'')
                    xml_file.name = invoice.ubl_cii_xml_id.name or ''

                if not invoice.partner_id.l10n_tr_nilvera_customer_alias_id:
                    # If no alias is saved, the user is either an E-Archive user or we haven't checked before. Check again
                    # just in case.
                    invoice.partner_id.check_nilvera_customer()
                customer_alias = invoice.partner_id.l10n_tr_nilvera_customer_alias_id.name
                if customer_alias:  # E-Invoice
                    invoice._l10n_tr_nilvera_submit_einvoice(xml_file, customer_alias)
                else:   # E-Archive
                    invoice._l10n_tr_nilvera_submit_earchive(xml_file)

```

## File: wizard\account_move_send_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="account_move_send_inherit_l10n_tr_nilvera_einvoice">
            <field name="name">account.move.send.form.inherit.l10n.tr.nilvera.einvoice</field>
            <field name="model">account.move.send</field>
            <field name="inherit_id" ref="account.account_move_send_form"/>
            <field name="arch" type="xml">
            <xpath expr="//div[@name='warnings']" position="inside">
                <field name="l10n_tr_nilvera_warnings" class="o_field_html" widget="actionable_errors"/>
            </xpath>
                <xpath expr="//div[@name='advanced_options']" position='after'>
                    <field name="l10n_tr_nilvera_einvoice_enable_xml" invisible="1"/>
                    <div name="option_send_nilvera"
                         invisible="not l10n_tr_nilvera_einvoice_enable_xml">
                        <field name="l10n_tr_nilvera_einvoice_checkbox_xml"/>
                        <span class="fw-bold">
                            <label for="l10n_tr_nilvera_einvoice_checkbox_xml"/>
                        </span>
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

