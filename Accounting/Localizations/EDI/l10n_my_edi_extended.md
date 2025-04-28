# Odoo Module: l10n_my_edi_extended

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
    'name': 'Malaysia - E-invoicing Extended Features',
    'countries': ['my'],
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'icon': '/account/static/description/l10n.png',
    "summary": "Extended features for the E-invoicing using MyInvois",
    'description': """
    This module improves the MyInvois E-invoicing feature by adding proper support for self billing, rendering the MyInvois
    QR code in the invoice PDF file and allows better management of foreign customer TIN.
    """,
    'depends': ['l10n_my_edi'],
    'data': [
        'views/account_move_view.xml',
        'views/report_invoice.xml',
        'views/res_partner_view.xml',
    ],
    'installable': True,
    'auto_install': ['l10n_my_edi'],
    'license': 'LGPL-3'
}

```

## File: models\account_edi_xml_ubl_my.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class AccountEdiXmlUBLMyInvoisMY(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_myinvois_my"

    def _export_invoice_vals(self, invoice):
        # EXTENDS 'l10n_my_edi'
        vals = super()._export_invoice_vals(invoice)

        # For self billed documents (when sending in_xxx entries to the platform) the supplier and customer are reversed.
        if self._is_self_billed(vals['vals']['document_type_code']):
            vals['vals']['accounting_supplier_party_vals']['party_vals'] = self._get_partner_party_vals(invoice.partner_id, role='supplier')
            vals['vals']['accounting_customer_party_vals']['party_vals'] = self._get_partner_party_vals(invoice.company_id.partner_id, role='customer')
            # /!\ For the company (regular invoices) it is the field on res.company that is used, and not the one on res.partner.
            # In master the behavior will be aligned and the classification information will be retrieved in _get_partner_party_vals
            vals['vals']['accounting_supplier_party_vals']['party_vals'].update({
                'industry_classification_code_attrs': {'name': invoice.partner_id.commercial_partner_id.l10n_my_edi_industrial_classification.name},
                'industry_classification_code': invoice.partner_id.commercial_partner_id.l10n_my_edi_industrial_classification.code,
            })
        # Sometimes, a foreign customer is also a supplier.
        # To avoid needing to change the Generic TIN depending on what you do with your commercial partner, we will automatically switch
        # depending on the context.
        if self._is_self_billed(vals['vals']['document_type_code']):
            other_party = vals["vals"]["accounting_supplier_party_vals"]["party_vals"]
            opposite_generic_tin = 'EI00000000020'
            expected_generic_tin = 'EI00000000030'
        else:
            other_party = vals["vals"]["accounting_customer_party_vals"]["party_vals"]
            opposite_generic_tin = 'EI00000000030'
            expected_generic_tin = 'EI00000000020'
        # Switch the generic tin to the correct one when it makes sense (For example when a supplier has the buyer generic tin set)
        for identification_val in other_party['party_identification_vals']:
            if identification_val.get('id_attrs', {}).get('schemeID') == 'TIN' and identification_val.get('id') == opposite_generic_tin:
                identification_val['id'] = expected_generic_tin
        return vals

    def _get_delivery_vals_list(self, invoice):
        # OVERRIDE 'l10n_my_edi'
        customer = invoice.company_id.partner_id if invoice.is_purchase_document() else invoice.partner_id
        return [{
            'accounting_delivery_party_vals': self._l10n_my_edi_get_delivery_party_vals(customer),
        }]

    def _export_invoice_constraints(self, invoice, vals):
        # EXTENDS 'l10n_my_edi'
        constraints = super()._export_invoice_constraints(invoice, vals)
        # The credit/debit note error would trigger for self billed invoice, we check if it's the case and remove it if needed.
        document_type_code, original_document = self._l10n_my_edi_get_document_type_code(invoice)
        if document_type_code == '11' and f"myinvois_{invoice.id}_adjustment_origin" in constraints:
            del constraints[f'myinvois_{invoice.id}_adjustment_origin']
        # The classification check was only looking at the product, we also want to validate lines without product
        for line in invoice.invoice_line_ids.filtered(lambda line: line.display_type not in ('line_note', 'line_section')):
            # If there are no products, we still expect a classification to be manually set.
            if not line.product_id and not line.l10n_my_edi_classification_code:
                self._l10n_my_edi_make_validation_error(constraints, 'class_code_required_line', line.id, line.display_name)
            # We allow invoicing a product with no classification when the classification has been manually provided.
            if f"myinvois_{line.product_id.id}_class_code_required" in constraints and line.l10n_my_edi_classification_code:
                del constraints[f"myinvois_{line.product_id.id}_class_code_required"]

        return constraints

    @api.model
    def _l10n_my_edi_get_document_type_code(self, invoice):
        """ Override the super method to include self billed documents. """
        # OVERRIDE 'l10n_my_edi'
        super()._l10n_my_edi_get_document_type_code(invoice)

        if 'debit_origin_id' in self.env['account.move']._fields and invoice.debit_origin_id:
            code = '03' if invoice.move_type == 'out_invoice' else '13'
            return code, invoice.debit_origin_id
        elif invoice.move_type in ('out_refund', 'in_refund'):
            # We consider a credit note a refund if it is paid and fully reconciled with a payment or bank transaction.
            payment_terms = invoice.line_ids.filtered(lambda aml: aml.display_type == 'payment_term')
            counterpart_amls = payment_terms.matched_debit_ids.debit_move_id + payment_terms.matched_credit_ids.credit_move_id
            counterpart_move_type = 'out_invoice' if invoice.move_type == 'out_refund' else 'out_refund'
            has_payments = bool(counterpart_amls.move_id.filtered(lambda move: move.move_type != counterpart_move_type))
            is_paid = invoice.payment_state == invoice._get_invoice_in_payment_state()
            if is_paid and has_payments:
                code = '04' if invoice.move_type == 'out_refund' else '14'
            else:
                code = '02' if invoice.move_type == 'out_refund' else '12'

            return code, invoice.reversed_entry_id
        else:
            code = '01' if invoice.move_type == 'out_invoice' else '11'
            return code, None

    def _get_invoice_line_item_vals(self, line, taxes_vals):
        # EXTENDS 'l10n_my_edi' to use the new field
        vals = super()._get_invoice_line_item_vals(line, taxes_vals)
        # Replace the code to get it from the line instead
        vals['commodity_classification_vals'] = [{
            'item_classification_code': line.l10n_my_edi_classification_code,
            'item_classification_attrs': {'listID': 'CLASS'},
        }]
        return vals

    @api.model
    def _is_self_billed(self, document_code):
        """ Small helper which returns True if a document code is for self billing.
        To avoid repeating the check multiple time, risking to forget to update one or the other.
        """
        return document_code in {"11", "12", "13", "14"}

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import time
from collections import defaultdict

import werkzeug

from odoo import fields, models, api, _, SUPERUSER_ID
from odoo.exceptions import UserError
from odoo.tools import image_data_uri


class AccountMove(models.Model):
    _inherit = "account.move"

    # ------------------
    # Fields declaration
    # ------------------

    l10n_my_edi_invoice_long_id = fields.Char(
        string="MyInvois Long ID",
        copy=False,
        readonly=True,
    )
    l10n_my_invoice_need_edi = fields.Boolean(
        compute='_compute_l10n_my_invoice_need_edi',
        export_string_translation=False,
    )

    # --------------------------------
    # Compute, inverse, search methods
    # --------------------------------

    @api.depends('move_type', 'state', 'country_code', 'company_id')
    def _compute_l10n_my_invoice_need_edi(self):
        for move in self:
            # We return true for malaysian invoices which are not sent yet, sent but awaiting validation or valid.
            move.l10n_my_invoice_need_edi = self.env['account.move.send']._l10n_my_edi_need_edi(move, ['in_progress', 'valid'])

    def _get_name_invoice_report(self):
        # EXTENDS 'account'
        if self.l10n_my_edi_external_uuid:  # Meaning we are a myinvois invoice, meaning we need to embed the qr code.
            # As we add the view in stable, we need to check that it exists.
            if self.env.ref('l10n_my_edi_extended.report_invoice_document', raise_if_not_found=False):
                return 'l10n_my_edi_extended.report_invoice_document'
        return super()._get_name_invoice_report()

    # --------------
    # Action methods
    # --------------

    def action_invoice_sent(self):
        """ The wizard should not be available for invoices sent to MyInvois but not yet validated.
        This is because before validation the ID used for the QR code is not available and the user should NOT send the invoice yet.
        """
        self.ensure_one()

        if self.l10n_my_edi_state == 'in_progress':
            raise UserError(_('You cannot send invoices that are currently being validated.\nPlease wait for the validation to complete.'))

        return super().action_invoice_sent()

    # ----------------
    # Business methods
    # ----------------

    def _update_validation_fields(self, validation_result):
        """ Extended to update the long id as well. """
        # EXTENDS 'l10n_my_edi'
        super()._update_validation_fields(validation_result)
        self.l10n_my_edi_invoice_long_id = validation_result['long_id']

    def _generate_myinvois_qr_code(self):
        """ Generate the qr code which should be embedded into the invoices PDF """
        self.ensure_one()

        if not self.l10n_my_edi_invoice_long_id:  # Only valid invoices have a long id
            return None

        # We need to add the portal url to the qr
        proxy_user = self._l10n_my_edi_ensure_proxy_user()
        if proxy_user.edi_mode == 'prod':
            portal_url = "myinvois.hasil.gov.my"
        else:
            portal_url = "preprod.myinvois.hasil.gov.my"

        try:
            qr_code = self.env['ir.actions.report'].barcode(
                barcode_type='QR',
                width=128,
                height=128,
                humanreadable=1,
                value=f'https://{portal_url}/{self.l10n_my_edi_external_uuid}/share/{self.l10n_my_edi_invoice_long_id}',
            )
        except (ValueError, AttributeError):
            raise werkzeug.exceptions.HTTPException(description='Cannot convert into QR Code.')

        return image_data_uri(base64.b64encode(qr_code))

    def action_l10n_my_edi_send_invoice(self):
        """ Create the xml file (if needed) to be sent to the platform.
        This will replace what is done in send & print.
        """
        # Gather the moves that have to be sent and the xml for each of them.
        moves, xml_contents = self._l10n_my_edi_prepare_moves_to_send()
        # We then push the moves to myinvois.
        self._l10n_my_edi_send_to_myinvois(moves, xml_contents)
        # We need to see if the validation status is already available; otherwise it will be fetched via a cron.
        self._l10n_my_edi_get_status(moves)
        # Finally, we update the move attachments
        for move, xml_content in xml_contents.items():
            if xml_content:
                self.env['ir.attachment'].with_user(SUPERUSER_ID).create({
                    'name': f'{move.name.replace("/", "_")}_myinvois.xml',
                    'raw': xml_content,
                    'mimetype': 'application/xml',
                    'res_model': move._name,
                    'res_id': move.id,
                    'res_field': 'l10n_my_edi_file',  # Binary field
                })
                move.invalidate_recordset(fnames=['l10n_my_edi_file_id', 'l10n_my_edi_file'])

    def _l10n_my_edi_prepare_moves_to_send(self):
        AccountMoveSend = self.env['account.move.send']
        xml_contents = defaultdict(list)
        moves = self.env['account.move']
        for move in self:
            if not move.l10n_my_invoice_need_edi or move.l10n_my_edi_state:
                continue

            moves |= move

            if move.l10n_my_edi_file:
                xml_content = base64.b64decode(move.l10n_my_edi_file).decode('utf-8')
            else:
                xml_content, errors = move._l10n_my_edi_generate_invoice_xml()
                if errors:
                    raise UserError(AccountMoveSend._format_error_text({
                        'error_title': _('Error when generating MyInvois file:'),
                        'errors': errors,
                    }))
                xml_content = xml_content.decode('utf-8')
            xml_contents[move] = xml_content
        return moves, xml_contents

    def _l10n_my_edi_send_to_myinvois(self, moves, xml_contents):
        AccountMoveSend = self.env['account.move.send']
        if moves and xml_contents:
            errors = moves._l10n_my_edi_submit_documents(xml_contents)

            for move in moves.filtered(lambda m: m in errors):
                move.message_post(body=AccountMoveSend._format_error_html({
                    'error_title': _('Error when sending the invoices to the E-invoicing service.'),
                    'errors': errors[move],
                }))

            # At this point we will need to commit as we reached the api, and we could have a mix of failed and valid invoice.
            if moves._can_commit():
                self._cr.commit()

            # We already logged the details on the invoice(s) and saved the api results. If we send a single invoice, we can safely raise now.
            if errors and len(moves) == 1:
                raise UserError(AccountMoveSend._format_error_text({
                    'error_title': _('Error when sending the invoices to the E-invoicing service.'),
                    'errors': errors[moves],
                }))

    def _l10n_my_edi_get_status(self, moves):
        AccountMoveSend = self.env['account.move.send']
        retry = 0
        errors, any_in_progress = moves._l10n_my_edi_fetch_updated_statuses()
        while any_in_progress and retry < 2:
            time.sleep(1)  # We wait a second before retrying.
            errors, any_in_progress = moves._l10n_my_edi_fetch_updated_statuses()
            retry += 1
        # While technically an in_progress status is not an error, it won't hurt much to display it as such.
        # The "error" message in this case should be clear enough.
        for move in moves.filtered(lambda m: m in errors):
            move.message_post(body=AccountMoveSend._format_error_html({
                'error_title': _('Error when sending the invoices to the E-invoicing service.'),
                'errors': errors[move],
            }))
        # We commit again if possible, to ensure that the invoice status is set in the database in case of errors later.
        if self._can_commit():
            self._cr.commit()

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api
from odoo.addons.l10n_my_edi.models.product_template import CLASSIFICATION_CODES_LIST


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    # ------------------
    # Fields declaration
    # ------------------

    l10n_my_edi_classification_code = fields.Selection(
        string="Malaysian classification code",
        selection=CLASSIFICATION_CODES_LIST,
        compute="_compute_l10n_my_edi_classification_code",
        store=True,
        readonly=False,
        copy=False,
    )

    # --------------------------------
    # Compute, inverse, search methods
    # --------------------------------

    @api.depends("product_id.product_tmpl_id")
    def _compute_l10n_my_edi_classification_code(self):
        """ Default to the product classification if any """
        for line in self:
            # We don't want to automatically update it on invoices that were sent to MyInvois
            if not line.move_id.l10n_my_edi_external_uuid:
                line.l10n_my_edi_classification_code = line.product_id.product_tmpl_id.l10n_my_edi_classification_code or line.l10n_my_edi_classification_code

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    # ------------------
    # Fields declaration
    # ------------------

    # Note: When merging with the base module in master, the company's industrial classification should become a related to this field.
    l10n_my_edi_industrial_classification = fields.Many2one(
        comodel_name='l10n_my_edi.industry_classification',
        string="Ind. Classification",
        compute='_compute_l10n_my_edi_industrial_classification',
        store=True,
        readonly=False,
    )
    l10n_my_edi_malaysian_tin = fields.Char(
        string="Malaysian TIN",
        help="The value set in this field will be used as TIN for the customer/supplier.\n"
             "If left empty, the Tax ID field will be used.",
    )

    # --------------------------------
    # Compute, inverse, search methods
    # --------------------------------

    @api.depends('l10n_my_edi_malaysian_tin')
    def _compute_l10n_my_tin_validation_state(self):
        # EXTEND 'l10n_my_edi' to add the depends
        super()._compute_l10n_my_tin_validation_state()

    def _compute_l10n_my_edi_industrial_classification(self):
        default_classification = self.env.ref('l10n_my_edi.class_00000', raise_if_not_found=False)
        self.filtered(lambda p: not p.l10n_my_edi_industrial_classification).l10n_my_edi_industrial_classification = default_classification

    # ----------------
    # Business methods
    # ----------------

    def _l10n_my_edi_get_tin_for_myinvois(self):
        # EXTEND 'l10n_my_edi'
        # When l10n_my_edi_malaysian_tin is set, it will be used instead of the VAT.
        # A user may want to keep the correct VAT on a foreign contact while also use myinvois with a malaysia TIN/Generic TIN
        # Using the Tax ID field also causes issue when base_vat is enabled, which block setting foreign VAT numbers.
        return self.l10n_my_edi_malaysian_tin or super()._l10n_my_edi_get_tin_for_myinvois()

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_my_edi_industrial_classification', 'l10n_my_edi_malaysian_tin']

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_edi_xml_ubl_my
from . import account_move
from . import account_move_line
from . import res_partner

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_move_form_inherit_l10n_my_myinvois_extended" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n_my_myinvois_extended</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="l10n_my_edi.view_move_form_inherit_l10n_my_myinvois"/>
        <field name="arch" type="xml">
            <!-- Hide the original send & print button, as it is difficult to update its invisible condition -->
            <button name="action_invoice_sent" class="oe_highlight" position="attributes">
                <attribute name="invisible" add="l10n_my_invoice_need_edi" separator=" or "/>
            </button>
            <!-- Instead, we display our own buttons so that we can write the invisible properly for our use case. -->
            <xpath expr="//button[@name='action_invoice_sent' and not(@class)]" position="after">
                <button name="action_invoice_sent"
                        type="object"
                        string="Send &amp; Print"
                        invisible="(not l10n_my_invoice_need_edi or l10n_my_edi_state != 'valid') or (state != 'posted' or is_being_sent or invoice_pdf_report_id or move_type not in ('out_invoice', 'out_refund'))"
                        class="oe_highlight"
                        data-hotkey="y"/>
                <button name="action_invoice_sent"
                        type="object"
                        string="Send &amp; Print"
                        invisible="(not l10n_my_invoice_need_edi or l10n_my_edi_state == 'valid') or (state != 'posted' or is_being_sent or invoice_pdf_report_id or move_type not in ('out_invoice', 'out_refund'))"
                        data-hotkey="y"/>
            </xpath>
            <!-- We want the CTA to be primary only on invoices, and secondary on vendor bills. -->
            <button name="action_invoice_sent" position="before">
                <button name="action_l10n_my_edi_send_invoice" string="Send To MyInvois" type="object"
                        groups="account.group_account_invoice"
                        class="oe_highlight"
                        invisible="not l10n_my_invoice_need_edi or l10n_my_edi_state or move_type not in ('out_invoice', 'out_refund')"/>
            </button>
            <button name="action_register_payment" position="after">
                <button name="action_l10n_my_edi_send_invoice" string="Send To MyInvois" type="object"
                        groups="account.group_account_invoice"
                        invisible="not l10n_my_invoice_need_edi or l10n_my_edi_state or move_type not in ('in_invoice', 'in_refund')"/>
            </button>
            <!-- The rejection button is only intended for received bills, as we don't support that at the moment, and we now send bills, it will be confusing to keep it. -->
            <button name="action_l10n_my_edi_reject_bill" position="replace">
            </button>
            <field name="l10n_my_edi_display_tax_exemption_reason" position="after">
                    <field name="l10n_my_invoice_need_edi" invisible="1"/>
            </field>
            <!-- Add the classification code to the invoice lines -->
            <xpath expr="//field[@name='invoice_line_ids']/tree/field[@name='name']" position="after">
                <field name="l10n_my_edi_classification_code" optional="hide"/>
            </xpath>
            <field name="l10n_my_edi_external_uuid" position="after">
                <field name="l10n_my_edi_invoice_long_id" invisible="not l10n_my_edi_external_uuid"/>
            </field>
        </field>
    </record>

    <record id="invoice_send_to_myinvois" model="ir.actions.server">
        <field name="name">Send To MyInvois</field>
        <field name="state">code</field>
        <field name="model_id" ref="model_account_move"/>
        <field name="binding_model_id" ref="model_account_move"/>
        <field name="binding_view_types">list</field>
        <field name="code">
            if records:
                action = records.action_l10n_my_edi_send_invoice()
        </field>
    </record>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
        <div id="qrcode" position="after">
            <div id="myinvois_qrcode" class="d-flex mb-3 avoid-page-break-inside" t-if="o.l10n_my_edi_external_uuid">
                <div class="qrcode me-3" id="myinvois_qrcode_image">
                    <t t-set="qr_code_url" t-value="o._generate_myinvois_qr_code()"/>
                    <p t-if="qr_code_url" class="position-relative mb-0">
                        <img t-att-src="qr_code_url"/>
                        <img src="/account/static/src/img/Odoo_logo_O.svg"
                             id="qrcode_odoo_logo"
                             class="top-50 start-50 position-absolute bg-white border border-white border-3 rounded-circle"
                        />
                    </p>
                </div>
                <div class="d-inline text-muted lh-sm fst-italic" id="qrcode_info" t-if="qr_code_url">
                    <p>Scan this QR Code to<br/>access your invoice
                    </p>
                </div>
            </div>
        </div>
    </template>

    <!-- Workaround for Studio reports, see odoo/odoo#60660 -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_my_edi_extended.report_invoice_document'"
               t-call="l10n_my_edi_extended.report_invoice_document"
               t-lang="lang"/>
        </xpath>
    </template>
</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_my_myinvois_extended" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_my_myinvois_extended</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="l10n_my_edi.view_partner_form_inherit_l10n_my_myinvois"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='l10n_my_edi']/group" position="inside">
                <field name="l10n_my_edi_industrial_classification" readonly="parent_id"/>
                <field name="l10n_my_edi_malaysian_tin" placeholder="EI00000000020" readonly="parent_id"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\account_move_send.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.tools import str2bool


class AccountMoveSend(models.TransientModel):
    _inherit = 'account.move.send'

    @api.depends('move_ids')
    def _compute_l10n_my_edi_enable(self):
        """ Override to disable the usage of MyInvois in the Send & Print wizard.
        It is not fully compatible with the QR flow and thus, we intend to send the file to MyInvois separately.
        """
        super()._compute_l10n_my_edi_enable()
        for wizard in self:
            # In master, the send & print sending flow will be fully removed and this won't be needed anymore.
            # For now, this is kept so that runbot won't fail the base module tests, which we still want to run atm.
            disabled = str2bool(self.env['ir.config_parameter'].sudo().get_param('l10n_my_edi.disable.send_and_print.first', 'True'))
            wizard.l10n_my_edi_enable = not disabled and wizard.l10n_my_edi_enable

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move_send

```

