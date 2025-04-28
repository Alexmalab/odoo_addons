# Odoo Module: l10n_rs_edi

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import wizard
from . import models

```

## File: __manifest__.py

```python
{
    'author': 'Odoo',
    'name': 'Serbia - eFaktura E-invoicing',
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'description': """
eFaktura E-invoice implementation for Serbia
    """,
    'summary': "E-Invoice implementation for Serbia",
    'countries': ['rs'],
    'depends': [
        'account_edi_ubl_cii',
        'l10n_rs',
    ],
    'data': [
        'views/res_config_settings_views.xml',
        'views/account_move.xml',
        'views/res_partner_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
UPDATE res_company
    SET l10n_rs_edi_api_key = 'dummy_key',
        l10n_rs_edi_demo_env = TRUE

```

## File: models\account_edi_xml_ubl_21_rs.py

```python
from odoo import api, models


class AccountEdiXmlUBL21RS(models.AbstractModel):
    _name = "account.edi.xml.ubl.rs"
    _inherit = 'account.edi.xml.ubl_21'
    _description = "UBL 2.1 (RS eFaktura)"

    @api.model
    def _get_customization_ids(self):
        vals = super()._get_customization_ids()
        vals['efaktura_rs'] = 'urn:cen.eu:en16931:2017#compliant#urn:mfin.gov.rs:srbdt:2022#conformant#urn:mfin.gov.rs:srbdtext:2022'
        return vals

    def _export_invoice_vals(self, invoice):
        # EXTENDS 'account_edi_ubl_cii'
        vals = super()._export_invoice_vals(invoice)

        vals['vals'].update({
            'customization_id': self._get_customization_ids()['efaktura_rs'],
            'billing_reference_vals': self._l10n_rs_get_billing_reference(invoice),
        })
        return vals

    def _l10n_rs_get_billing_reference(self, invoice):
        # Billing Reference values for Credit Note
        if invoice.move_type == 'out_refund' and invoice.reversed_entry_id:
            return {
                'id': invoice.reversed_entry_id.name,
                'issue_date': invoice.reversed_entry_id.invoice_date,
            }
        return {}

    def _get_invoice_period_vals_list(self, invoice):
        # EXTENDS account_edi_ubl_cii
        vals_list = super()._get_invoice_period_vals_list(invoice)
        vals_list.append({
            'description_code': '0' if invoice.move_type == 'out_refund' else invoice.l10n_rs_tax_date_obligations_code,
        })
        return vals_list

    def _get_partner_party_vals(self, partner, role):
        vals = super()._get_partner_party_vals(partner, role)
        vat_country, vat_number = partner._split_vat(partner.vat)
        if vat_country.isnumeric():
            vat_number = partner.vat
        vals.update({
            'endpoint_id': vat_number,
            'endpoint_id_attrs': {
                'schemeID': '9948',
            },
        })
        return vals

    def _get_partner_party_legal_entity_vals_list(self, partner):
        # EXTENDS 'account_edi_ubl_cii'
        vals_list = super()._get_partner_party_legal_entity_vals_list(partner)
        for vals in vals_list:
            vals['company_id'] = partner.l10n_rs_edi_registration_number
        return vals_list

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        # EXTENDS 'account_edi_ubl_cii'
        vals_list = super()._get_partner_party_tax_scheme_vals_list(partner, role)

        for vals in vals_list:
            vat_country, vat_number = partner._split_vat(partner.vat)
            if vat_country.isnumeric():
                vat_country = 'RS'
                vat_number = partner.vat
            if vat_country == 'RS' and partner.simple_vat_check(vat_country, vat_number):
                vals['company_id'] = vat_country + vat_number
        return vals_list

    def _get_partner_party_identification_vals_list(self, partner):
        vals_list = super()._get_partner_party_identification_vals_list(partner)
        if partner.country_code == 'RS' and partner.l10n_rs_edi_public_funds:
            vals_list.append({
                'id': f'JBKJS: {partner.l10n_rs_edi_public_funds}',
            })
        return vals_list

```

## File: models\account_move.py

```python
import uuid
import requests

from odoo import _, api, fields, models
from requests.exceptions import Timeout, ConnectionError, HTTPError

DEMO_EFAKTURA_URL = 'https://demoefaktura.mfin.gov.rs/api/publicApi/sales-invoice/ubl'
EFAKTURA_URL = 'https://efakturadev.mfin.gov.rs/api/publicApi/sales-invoice/ubl'


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_rs_edi_uuid = fields.Char(
        string="RS Invoice UUID",
        compute="_compute_l10n_rs_edi_uuid",
        copy=False,
        store=True,
        help="Unique Identifier for an invoice used as request id",
    )

    l10n_rs_edi_is_eligible = fields.Boolean(
        compute="_compute_l10n_rs_edi_is_eligible",
        store=True,
        help="Technical field to determine if this invoice is eligible to be e-invoiced.",
    )

    l10n_rs_edi_attachment_file = fields.Binary(
        string="Serbian E-Invoice XML File",
        copy=False,
        attachment=True,
        help="Serbia: technical field holding the e-invoice XML data.",
    )

    l10n_rs_edi_attachment_id = fields.Many2one(
        comodel_name='ir.attachment',
        string="eFaktura XML Attachment",
        compute=lambda self: self._compute_linked_attachment_id('l10n_rs_edi_attachment_id', 'l10n_rs_edi_attachment_file'),
        depends=['l10n_rs_edi_attachment_file'],
    )

    l10n_rs_edi_state = fields.Selection(
        string="Serbia E-Invoice state",
        selection=[
            ('sent', 'Sent'),
            ('sending_failed', 'Error'),
        ],
        tracking=True,
        readonly=True,
        copy=False,
    )

    l10n_rs_edi_error = fields.Text(
        string="Serbia E-Invoice error",
        copy=False,
        readonly=True,
    )

    l10n_rs_tax_date_obligations_code = fields.Selection(
        string="Tax Date Obligations",
        selection=[
            ('35', 'By Delivery Date'),
            ('3', 'By Issuance Date'),
            ('432', 'By Billing System'),
        ],
        store=True,
        readonly=False,
        compute='_compute_l10n_rs_tax_date_obligations_code',
    )
    l10n_rs_edi_invoice = fields.Char(string="Invoice Id", copy=False)
    l10n_rs_edi_sales_invoice = fields.Char(string="Sales Invoice Id", copy=False)
    l10n_rs_edi_purchase_invoice = fields.Char(string="Purchase Invoice Id", copy=False)

    @api.depends('country_code', 'move_type')
    def _compute_show_delivery_date(self):
        # EXTENDS 'account'
        super()._compute_show_delivery_date()
        for move in self:
            if move.l10n_rs_edi_is_eligible:
                move.show_delivery_date = True

    @api.depends("country_code", "move_type")
    def _compute_l10n_rs_edi_is_eligible(self):
        for move in self:
            move.l10n_rs_edi_is_eligible = move.country_code == 'RS' and move.is_sale_document() and move.l10n_rs_edi_state in (False, 'sending_failed')

    @api.depends('country_code')
    def _compute_l10n_rs_tax_date_obligations_code(self):
        for move in self:
            if move.country_code == 'RS':
                move.l10n_rs_tax_date_obligations_code = '3'

    @api.depends('l10n_rs_edi_state')
    def _compute_show_reset_to_draft_button(self):
        # EXTENDS 'account'
        super()._compute_show_reset_to_draft_button()
        for move in self:
            if move.show_reset_to_draft_button and move.l10n_rs_edi_state == 'sent':
                move.show_reset_to_draft_button = False

    @api.depends('l10n_rs_edi_is_eligible')
    def _compute_l10n_rs_edi_uuid(self):
        for move in self:
            if move.l10n_rs_edi_is_eligible and not move.l10n_rs_edi_uuid:
                move.l10n_rs_edi_uuid = uuid.uuid4()

    def button_draft(self):
        # EXTENDS 'account'
        self.write({
            "l10n_rs_edi_error": False,
            "l10n_rs_edi_state": False,
        })
        return super().button_draft()

    def _l10n_rs_edi_send(self, send_to_cir):
        self.ensure_one()
        self.env['res.company']._with_locked_records(self)
        xml, errors = self.env['account.edi.xml.ubl.rs']._export_invoice(self)
        if errors:
            return xml, errors
        params = {
            'requestId': self.l10n_rs_edi_uuid,
            'sendToCir': 'Yes' if send_to_cir else 'No'
        }
        url = DEMO_EFAKTURA_URL if self.company_id.l10n_rs_edi_demo_env else EFAKTURA_URL
        headers = {
            'Content-Type': 'application/xml',
            'ApiKey': self.company_id.l10n_rs_edi_api_key,
        }
        error_message = False
        try:
            response = requests.post(url=url, params=params, headers=headers, data=xml, timeout=30)
            response.raise_for_status()
        except (Timeout, ConnectionError, HTTPError) as exception:
            error_message = _("There was a problem with the connection with eFaktura: %s", exception)
            self.message_post(body=error_message)
            return xml, error_message
        dict_response = {}
        try:
            dict_response = response.json()
        except requests.exceptions.JSONDecodeError as e:
            error_message = _("Invalid response from eFaktura: %s", str(e))
        self.l10n_rs_edi_state = 'sending_failed' if error_message else 'sent'
        self.l10n_rs_edi_error = error_message
        self.l10n_rs_edi_invoice = dict_response.get('InvoiceId')
        self.l10n_rs_edi_purchase_invoice = dict_response.get('PurchaseInvoiceId')
        self.l10n_rs_edi_sales_invoice = dict_response.get('SalesInvoiceId')
        return xml, error_message

    def _l10n_rs_edi_get_attachment_values(self, xml):
        self.ensure_one()
        return {
            'name': self._l10n_rs_edi_get_xml_attachment_name(),
            'mimetype': 'application/xml',
            'description': _('RS E-Invoice: %s', self.move_type),
            'company_id': self.company_id.id,
            'res_id': self.id,
            'res_model': self._name,
            'res_field': 'l10n_rs_edi_attachment_file',
            'raw': xml,
            'type': 'binary',
        }

    def _l10n_rs_edi_get_xml_attachment_name(self):
        return f"{self.name.replace('/', '_')}_edi.xml"

```

## File: models\account_move_send.py

```python
from odoo import _, api, models, SUPERUSER_ID


class AccountMoveSend(models.AbstractModel):
    _inherit = "account.move.send"

    @api.model
    def _is_rs_edi_applicable(self, move):
        return move.l10n_rs_edi_is_eligible

    def _get_all_extra_edis(self) -> dict:
        # EXTENDS 'account'
        res = super()._get_all_extra_edis()
        res.update({'rs_edi': {'label': 'eFaktura', 'is_applicable': self._is_rs_edi_applicable, 'help': 'Send the E-Invoice to Government via eFaktura'}})
        res.update({'rs_cir_checkbox': {'is_applicable': self._is_rs_edi_applicable, 'label': _("Send to CIR"), 'help': _("Send to Central Invoice Register(For B2G and the public sector)")}})
        return res

    @api.model
    def _get_invoice_extra_attachments(self, invoice):
        # EXTENDS 'account'
        return super()._get_invoice_extra_attachments(invoice) + invoice.l10n_rs_edi_attachment_id

    @api.model
    def _call_web_service_after_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_after_invoice_pdf_render(invoices_data)
        for invoice, invoice_data in invoices_data.items():
            # Not all invoices may need EDI.
            if 'rs_edi' not in invoice_data['extra_edis']:
                continue
            if not invoice.company_id.l10n_rs_edi_api_key:
                invoice_data["error"] = {
                    "error_title": _("eFaktura API Key is missing."),
                    "errors": [_("Please configure the eFaktura API Key in the company settings.")],
                }
                continue
            send_to_cir = 'rs_cir_checkbox' in invoice_data['extra_edis']
            xml, error = invoice._l10n_rs_edi_send(send_to_cir)
            if error:
                invoice_data["error"] = {
                    "error_title": _("Errors when submitting the e-invoice to eFaktura:"),
                    "errors": [error],
                }
                continue
            invoice_data['l10n_rs_edi_attachment_values'] = invoice._l10n_rs_edi_get_attachment_values(xml)

            if self._can_commit():
                self._cr.commit()

    @api.model
    def _link_invoice_documents(self, invoices_data):
        # EXTENDS 'account'
        super()._link_invoice_documents(invoices_data)
        attachments_vals = [
            invoice_data.get('l10n_rs_edi_attachment_values')
            for invoice_data in invoices_data.values()
            if invoice_data.get('l10n_rs_edi_attachment_values')
        ]
        if attachments_vals:
            attachments = self.env['ir.attachment'].with_user(SUPERUSER_ID).create(attachments_vals)
            res_ids = [attachment.res_id for attachment in attachments]
            self.env['account.move'].browse(res_ids).invalidate_recordset(fnames=['l10n_rs_edi_attachment_id', 'l10n_rs_edi_attachment_file'])

```

## File: models\res_company.py

```python
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_rs_edi_api_key = fields.Char(string="eFaktura API Key")
    l10n_rs_edi_demo_env = fields.Boolean(string='Use Demo Environment', default=True)

```

## File: models\res_config_settings.py

```python
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_rs_edi_api_key = fields.Char(related="company_id.l10n_rs_edi_api_key", readonly=False)
    l10n_rs_edi_demo_env = fields.Boolean(related='company_id.l10n_rs_edi_demo_env', readonly=False)

```

## File: models\res_partner.py

```python
from odoo import api, models, fields, _
from odoo.exceptions import ValidationError


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_rs_edi_registration_number = fields.Char(
        string="Registration Number",
        help="Company ID ( Matični Broj ) assigned by the Serbian Business Registers Agency (APR) ",
        size=13,
    )
    l10n_rs_edi_public_funds = fields.Char(
        string="JBKJS",
        help="Unique Identifier of Public Funds Users such as Government agencies, public institutions and state-owned enterprises.",
        size=5,
    )

    @api.constrains('l10n_rs_edi_public_funds')
    def _check_l10n_rs_edi_public_funds(self):
        for record in self:
            if record.l10n_rs_edi_public_funds and \
                (len(record.l10n_rs_edi_public_funds) < 5 or not record.l10n_rs_edi_public_funds.isdigit()):
                raise ValidationError(_('Public Funds ID(JBKJS) must be exactly five digits'))

    @api.constrains('l10n_rs_edi_registration_number')
    def _check_l10n_rs_edi_registration_number(self):
        for record in self:
            if record.l10n_rs_edi_registration_number and \
                (len(record.l10n_rs_edi_registration_number) not in [8, 13] or not record.l10n_rs_edi_registration_number.isdigit()):
                raise ValidationError(_('Customer identification number should be 8 or 13 digits'))

```

## File: models\__init__.py

```python
from . import res_company
from . import res_config_settings
from . import account_move
from . import account_edi_xml_ubl_21_rs
from . import res_partner
from . import account_move_send

```

## File: views\account_move.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_move_form" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n_rs_edi</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='delivery_date']" position="after">
                <field name="l10n_rs_tax_date_obligations_code" 
                    invisible="country_code != 'RS' and move_type != 'out_invoice'"
                    required="country_code == 'RS' and move_type == 'out_invoice'"/>
            </xpath>

            <xpath expr="//field[@name='delivery_date']" position="attributes">
                <attribute name="required">country_code == 'RS' and l10n_rs_tax_date_obligations_code == '35'</attribute>
            </xpath>

            <xpath expr="//sheet" position="before">
                <div class="alert alert-warning" role="alert" invisible="not l10n_rs_edi_error">
                        <div class="p-0 m-0">
                            <i class="fa fa-warning" role="img" title="Serbian eFaktura"/>
                            <span class="mx-1">Serbian E-invoice Error:</span>
                        </div>
                        <field name="l10n_rs_edi_error"/>
                    </div>
            </xpath>

            <xpath expr="//page[@id='other_tab_entry']" position="before">
                <page id="l10n_rs_edi"
                      string="eFaktura"
                      invisible="not l10n_rs_edi_state">
                    <group>
                        <group>
                            <field name="l10n_rs_edi_state" readonly="1"/>
                        </group>
                        <group>
                            <field name="l10n_rs_edi_invoice" readonly="1"/>
                            <field name="l10n_rs_edi_sales_invoice" readonly="1"/>
                            <field name="l10n_rs_edi_purchase_invoice" readonly="1"/>
                        </group>
                    </group>
                    <group>
                        <field name="l10n_rs_edi_attachment_file" widget="binary" filename="l10n_rs_edi_attachment_file" readonly="1"/>
                    </group>
                </page>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n_rs_edi</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@id='invoicing_settings']" position="after">
                <block title="eFaktura (Serbia)" id="l10n_rs_edi_settings" invisible="country_code != 'RS'">
                    <setting string="eFaktura Credentials" help="Configure your eFaktura credentials here" company_dependent="1">
                        <div class="content-group">
                            <div class="row">
                                <label for="l10n_rs_edi_api_key" class="col-lg-3 o_light_label" string="API Key"/>
                                <field name="l10n_rs_edi_api_key"/>
                            </div>
                        </div>
                    </setting>
                    <setting id="l10n_rs_edi_demo_env_setting"
                        class="mt-3"
                        help="Activate demo environment for sending e-invoice to eFaktura">
                        <field name="l10n_rs_edi_demo_env"/>
                    </setting>
                </block>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_partner_view_form" model="ir.ui.view">
        <field name="name">res.partner.view.form.inherit.l10n_rs_edi</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_rs_edi_registration_number" invisible="'RS' not in fiscal_country_codes"/>
                <field name="l10n_rs_edi_public_funds" invisible="'RS' not in fiscal_country_codes"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\account_move_send_wizard.py

```python
from odoo import _, api, models


class AccountMoveSendWizard(models.TransientModel):
    _inherit = 'account.move.send.wizard'

    @api.onchange('extra_edi_checkboxes')
    def _onchange_extra_edi_checkboxes(self):
        checkboxes = self.extra_edi_checkboxes or {}
        if 'rs_edi' in checkboxes:
            if checkboxes['rs_edi']['checked']:
                checkboxes['rs_cir_checkbox']['readonly'] = False
            else:
                checkboxes['rs_cir_checkbox']['checked'] = False
                checkboxes['rs_cir_checkbox']['readonly'] = True
            self.extra_edi_checkboxes = {**checkboxes}

```

## File: wizard\__init__.py

```python
from . import account_move_send_wizard

```

